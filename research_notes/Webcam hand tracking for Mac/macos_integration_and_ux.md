# macOS System Integration and Interaction/UX Design for a Webcam Hand-Gesture App

Scope: (A) macOS plumbing (input injection, permissions, overlays, language choice) and (B) HCI/UX principles for mid-air input (cursor, click, scroll, trackpad-like gestures, air drawing). Researched Nov 2026. Note: several primary hosts (docs.ultraleap.com, developers.meta.com, a key TCC blog) could not be fetched from this session; findings from them are taken from search-result summaries and flagged as such.

## A1. How to inject input on macOS (CGEvent, scroll phases, trackpad gestures, AX)

### Takeaway
Mouse move/click/drag/scroll injection is fully supported via public Quartz Event Services (CGEventCreateMouseEvent / CGEventCreateScrollWheelEvent + CGEventPost). Smooth, trackpad-like scrolling is achievable with pixel-unit scroll events plus the continuous flag and phase fields. True pinch-zoom/rotate/swipe trackpad gestures have no public injection API; they require undocumented CGEvent types (gesture type 29) that WebKit uses only in its test harness — fragile and version-dependent. Safer fallback: map "zoom" to Cmd+scroll / Cmd+= keyboard shortcuts and "swipe between spaces" to Ctrl+Arrow.

### Cited Findings
- `CGEventCreateScrollWheelEvent` is Apple's recommended replacement for the deprecated `CGPostScrollWheelEvent`; it lets you customize the event before posting — [Apple: CGPostScrollWheelEvent](https://developer.apple.com/documentation/coregraphics/cgpostscrollwheelevent.md)
- Scroll events created with the pixel unit (`kCGScrollEventUnitPixel`) are interpreted by most apps as smooth scrolling events — [Apple: CGScrollEventUnit](https://developer.apple.com/documentation/coregraphics/cgscrolleventunit.md)
- `CGEventSetIntegerValueField` sets integer fields on an event; momentum phases are defined by `CGMomentumScrollPhase` (none/begin/continue/end) and the field `kCGScrollWheelEventMomentumPhase` — [Apple: CGMomentumScrollPhase](https://developer.apple.com/documentation/coregraphics/cgmomentumscrollphase.md); [FreePascal MacOSAll header mirror](https://freepascal.org/daily/packages/univint/macosall/kcgscrollwheeleventmomentumphase.html)
- WebKit's test tooling marks synthetic trackpad-style scroll by `CGEventSetIntegerValueField(cgScrollEvent, kCGScrollWheelEventIsContinuous, 1)` — [WebKit Bugzilla 232805](https://bugs.webkit.org/show_bug.cgi?id=232805)
- WebKit has its own "momentum event synthesizer" (i.e., apps may generate momentum themselves rather than relying on OS) and notes momentum events are distinguishable by phase (filtering "all but PhaseNone") — [WebKit bug 233653](https://bugs.webkit.org/show_bug.cgi?id=233653); [WebKit bug 225788](https://bugs.webkit.org/show_bug.cgi?id=225788)
- Chromium review warns true smooth scrolling in receiving apps may depend on app-side handling, not just the synthesized event — [Chromium codereview 42607](https://codereview.chromium.org/42607)
- Quartz Event Services overview (event creation/posting/taps) — [Apple: Quartz Event Services](https://developer.apple.com/documentation/coregraphics/quartz-event-services.md)
- Synthesizing pinch/magnify: a 2010 Apple cocoa-dev answer said these gestures cannot be simulated — [cocoa-dev 2010](https://lists.apple.com/archives/cocoa-dev/2010/Aug/msg01090.html); however WebKit's private SPI header defines `kCGSEventGesture = 29` and `kCGSEventZoom = 28`, and WebKit's EventSenderProxy builds gesture events by setting a CGEvent's type to the private gesture value and setting HID type/phase fields — [WebKit CoreGraphicsTestSPI.h](https://webkit.googlesource.com/WebKit/+/54591b58a50e66aaad868f829cfce9d38eafed7b/Tools/TestRunnerShared/spi/CoreGraphicsTestSPI.h); [WebKit EventSenderProxy.mm](https://webkit.googlesource.com/WebKit/+/0c2ad854b02baf7dcfb76f96705de638b478e115/Tools/WebKitTestRunner/mac/EventSenderProxy.mm)
- AXElements' CGEventAdditions.h documents private `kCGEventGesture = 29` and gesture sub-types (pinch/magnify, smart magnify, rotate, swipe) — [AXElements CGEventAdditions.h](https://codeclimate.com/github/AXElements/mouse/ext/mouse/CGEventAdditions.h/source)
- AppKit receives trackpad gestures as NSEvent magnify/rotate/swipe types that apps handle (`magnifyWithEvent:` etc.) — [Apple: Handling Trackpad Events](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/EventOverview/HandlingTouchEvents/HandlingTouchEvents.html)
- macOS standard gesture semantics to map to: scroll = two-finger slide, zoom = pinch, rotate = two fingers around each other, context menu = secondary click, drag = click-and-drag (HIG table updated Oct 28, 2026 for macOS touch displays) — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)

### Inferences
- Recommended implementation: posting at `kCGHIDEventTap` (or `kCGSessionEventTap`) with a dedicated `CGEventSource`; for drag, emit `leftMouseDown` → `leftMouseDragged` (not `mouseMoved`) → `leftMouseUp`; set click-state field (`kCGMouseEventClickState`=2) for double click. These are standard Quartz patterns, consistent with the docs above.
- For "two-finger scroll" gesture: emit pixel-unit, continuous scroll events with `kCGScrollWheelEventScrollPhase` began/changed/ended, then optionally synthesize a momentum tail (began/continue/end) using your own exponential decay. Test per-app (Safari, Chrome, Electron) since behavior differs.
- Treat private gesture-type events (pinch/rotate/swipe) as an optional, feature-flagged "experimental" path; ship robust fallbacks (Cmd+scroll for zoom in many apps, Cmd +/- , Ctrl+Left/Right for Spaces, Mission Control key).
- AXUIElement (`AXPress`, `AXScrollToVisible`, `AXIncrement`) offers semantic actions and hit-testing (`AXUIElementCopyElementAtPosition`) useful for target snapping/"magnetic" cursor, but requires the Accessibility privilege and is slower/less reliable than raw events (see A2 forum reports of sporadic AX failures).

### Gaps
- Could not fetch full WebKit EventSenderProxy source to confirm exact field IDs for magnification amount; no authoritative source confirms private gesture events still work on macOS 26.
- No primary doc found for exact scroll-phase field ordering expected by AppKit on macOS 15/26.

## A2. Permissions (TCC), code signing, sandbox/App Store, notarization

### Takeaway
Needed: Camera (NSCameraUsageDescription + `com.apple.security.device.camera` entitlement under hardened runtime) and Accessibility (for posting events / AX; TCC services `PostEvent` and `Accessibility` both surface as "Accessibility" in Settings). Input Monitoring (`ListenEvent`) only if you tap/listen to real input; Screen Recording only if you capture screen content (e.g., snapshot-with-ink export). Grants are tied to the code signature's designated requirement — ad-hoc signing changes cdhash every build, silently invalidating the grant. Use a stable Developer ID / Apple Development certificate (or a stable self-signed cert) for dev builds; ship outside the Mac App Store (Developer ID + notarization) because App Review has rejected CGEvent.post use.

### Cited Findings
- Apple DTS (Quinn): "Both `PostEvent` and `Accessibility` map to Accessibility. `ListenEvent` maps to Input Monitoring." APIs gated by `Accessibility` are not compatible with App Sandbox; `PostEvent`/`ListenEvent` are sandbox-compatible. DTS does not speak for App Review; a developer reported a Mac App Store rejection under Guideline 2.4.5 for using CGEvent.post, with no resolution posted through Sept 2026 — [Apple Dev Forums 820594](https://developer.apple.com/forums/thread/820594)
- `NSEvent.addGlobalMonitorForEvents` requires the Accessibility privilege (per DTS footnote) — [Apple Dev Forums 820594](https://developer.apple.com/forums/thread/820594)
- `tccutil reset PostEvent` resets the PostEvent privilege separately; resetting Accessibility does not clear it — [Apple Dev Forums 724603](https://developer.apple.com/forums/thread/724603) (per search summary)
- Code signing doesn't change approved privileges beyond requiring a stable signing identity; a development certificate is fine for testing — [Apple Dev Forums 724603](https://developer.apple.com/forums/thread/724603) (per search summary)
- Ad-hoc signing pins the TCC requirement to the binary's cdhash, which changes every rebuild; the Settings list still shows the app "enabled" while the stored csreq rejects the new binary; fix = sign every build with the same (even self-signed Keychain) certificate — [nickboy blog: TCC cdhash trap](https://nickboy-github-io.pages.dev/posts/tcc-cdhash-trap/) (search summary only; host not fetchable)
- OpenClaw docs: an Accessibility row can remain visibly enabled while its code requirement is pinned to an older build; some permissions reappear only after a full restart — [OpenClaw mac permissions](https://docs.openclaw.ai/platforms/mac/permissions)
- Developer report: editing/re-saving an app after granting Accessibility makes TCC treat it as a new app — [StackOverflow mirror 69058238](https://kaweah.freedombox.rocks/kiwix/content/stackoverflow.com_en_all_2023-11/questions/69058238/why-do-i-need-to-keep-re-enabled-accessibility-for-my-mac-whenever-i-compile-a); [Apple Dev Forums 750905](https://developer.apple.com/forums/thread/750905)
- Revoking Accessibility while a CGEventTap is active can make system input unresponsive (2026 forum thread) — [Apple Dev Forums 844416](https://developer.apple.com/forums/thread/844416)
- `AXIsProcessTrusted` returning stale/wrong values reported on Ventura+; AX APIs sporadically return no data despite grant — [Apple Dev Forums 727984](https://developer.apple.com/forums/thread/727984); [Apple Dev Forums 703188](https://developer.apple.com/forums/thread/703188)
- Camera: need `NSCameraUsageDescription` in Info.plist AND `com.apple.security.device.camera` = true in entitlements when hardened runtime (required for notarization) is on; missing pieces cause crashes, often from build-process bugs — verify with `codesign -d --entitlements` — [Apple Dev Forums 700714](https://developer.apple.com/forums/thread/700714); [Apple Dev Forums 127420](https://developer.apple.com/forums/thread/127420); [Apple: Hardened Runtime](https://developer.apple.com/documentation/security/hardened-runtime.md). (Note: one search summary garbled this as the entitlement key being in Info.plist; the usage-description key is NSCameraUsageDescription.)
- The camera entitlement applies with or without App Sandbox — [Apple Dev Forums 698926](https://developer.apple.com/forums/thread/698926)
- Macs prompt for camera permission for Apple's own Head Pointer as well — [AbilityNet](https://mcmw.abilitynet.org.uk/how-to-control-your-computer-with-head-movement-in-macos-12-monterey)

### Inferences
- Permission matrix: Camera = required; Accessibility (PostEvent) = required for cursor/click/scroll; Accessibility (full AX) = only if using AXUIElement snapping/actions; Input Monitoring = only if listening to keyboard (e.g., global hotkey via CGEventTap listen-only — prefer `RegisterEventHotKey`/MASShortcut-style Carbon hotkeys which need no permission); Screen Recording = only for screen-capture-based export (ScreenCaptureKit), not needed to draw an overlay.
- Onboarding: request one permission at a time, just-in-time, with a pre-prompt explaining why; use `CGPreflightPostEventAccess()`/`CGRequestPostEventAccess()` and `AXIsProcessTrustedWithOptions(prompt)`; poll status (TCC changes don't notify) and deep-link to `x-apple.systempreferences:com.apple.preference.security?Privacy_Accessibility`. Detect the "toggle shown on but not effective" state (preflight false while user claims enabled) and show remove-and-re-add instructions.
- Distribution: Developer ID + hardened runtime + notarization outside the App Store is the pragmatic path.
- Always keep a hardware kill switch (keyboard shortcut) and stop posting events immediately when trust is revoked, given the event-tap lock-up report.

### Gaps
- No official Apple doc enumerating TCC service names; DTS forum is the best available source.
- Could not confirm any macOS 26-specific TCC changes for PostEvent.

## A3. Overlay UI: click-through cursor/ink window, menu bar app, export

### Takeaway
Use a borderless, transparent, non-activating NSPanel/NSWindow with `ignoresMouseEvents = true`, `collectionBehavior = [.canJoinAllSpaces, .fullScreenAuxiliary, .stationary]`, and a high level (`.screenSaver` or `.statusBar`+) per display, rendered with Core Animation/Metal. Some developers report it still fails over other apps' full-screen Spaces, so test. Run as an LSUIElement menu-bar app.

### Cited Findings
- Click-through overlays use a borderless clear NSWindow with `ignoresMouseEvents = true` so clicks/drags/scrolls reach windows underneath — [SuperDimmer blog](https://superdimmer.pages.dev/blog/building-crash-proof-overlay-windows-macos-screen-dimming.html)
- `canJoinAllSpaces` + `fullScreenAuxiliary` is the usual combination; forum posts state `.screenSaver` level (1000) is needed since `.floating` (3) and `.statusBar` (25) sit below full-screen content; others report the window still hidden over another app's full-screen Space — [Apple Dev Forums 826308](https://developer.apple.com/forums/thread/826308); [Apple Dev Forums 800091](https://developer.apple.com/forums/thread/800091); [Apple Dev Forums 26677](https://developer.apple.com/forums/thread/26677)
- Alternative reported working: non-activating NSPanel at `.mainMenu` level with `canJoinAllSpaces` + `fullScreenAuxiliary` — [StackOverflow mirror 36205834](https://kiwix.lema.org/stackoverflow.com_en_all_2023-11/questions/36205834/allow-an-nswindow-nspanel-to-float-above-full-screen-apps)
- Electron's `setIgnoreMouseEvents(true,{forward:true})` limitation on macOS noted (forwarding not available) — [Peek click-through research](https://tangled.org/burrito.space/peek/blob/main/notes/click-through-research.md)

### Inferences
- Because the app posts synthetic events, the overlay never needs to receive mouse events itself — keep it permanently click-through; toggle to interactive only for an explicit "drawing mode toolbar."
- One overlay per NSScreen; handle `didChangeScreenParametersNotification`.
- Ink export: keep strokes as vectors (points + timestamps + pressure-proxy) and export SVG/PDF/PNG of ink alone (no permission); compositing with screen content requires Screen Recording permission via ScreenCaptureKit.
- Menu bar (`MenuBarExtra` / NSStatusItem) holds: enable/pause, mode (pointer / scroll / draw), calibration, permission status.

### Gaps
- No definitive source for reliable overlay behavior over full-screen apps on macOS 26.

## A4. Swift vs Python (PyObjC / pyautogui / pynput)

### Takeaway
Python works for prototyping (MediaPipe Python + PyObjC Quartz calls), but TCC grants attach to the launching host (Terminal/IDE/python binary), making permissions confusing and non-shippable; a signed Swift app bundle gets its own stable identity, native overlay, and lower latency. A hybrid (Swift app hosting Vision/MediaPipe or a bundled Python helper) is viable but the process that posts events must be the signed, permissioned one.

### Cited Findings
- pynput listeners expose `IS_TRUSTED`; "This process is not trusted!" warning occurs without a prompt — [pynput platform limitations](https://pynput.readthedocs.io/en/stable/limitations.html); [StackOverflow mirror 69620702](https://kiwix.lema.org/stackoverflow.com_en_all_2023-11/questions/69620702/this-process-is-not-trusted-when-running-code-in-pycharm)
- Fix commonly reported: grant Accessibility to the terminal/IDE app that launches Python; macOS 15.4 broke pyautogui for users until Terminal was enabled — [Reddit r/learnpython thread mirror](https://red.ks32.dev/r/learnpython/comments/1jtxb1p/macos_154_breaks_pyautogui); [pynput issue 424](https://github.com/moses-palmer/pynput/issues/424)
- Command-line clients have trouble with AX/TCC identity — [Apple Dev Forums 750905](https://developer.apple.com/forums/thread/750905); camera permission for command-line tools similarly attributed to the responsible process — [Apple Dev Forums 111100](https://developer.apple.com/forums/thread/111100)

### Inferences
- pyautogui adds fixed pauses (PAUSE default) and isn't designed for 60–120 Hz streams; direct Quartz via PyObjC avoids this. Swift avoids GIL/bridging overhead and allows Vision's `VNDetectHumanHandPoseRequest` on-device (unverified latency numbers — not researched here).
- Recommend: prototype in Python, ship in Swift (or Swift shell + embedded model).

### Gaps
- No measured latency comparisons found between PyObjC and Swift CGEvent posting.

## B1. Lessons from Vision Pro, Meta Quest, Leap Motion/Ultraleap, HoloLens

### Takeaway
The consistent lesson: separate targeting from commitment (look/point to target, then a discrete, low-effort "pinch" to commit), give continuous visual state feedback because hands have no haptics, keep hands low and near the body with small amplified motions, make targets forgiving, and never make a custom gesture the only way to do something.

### Cited Findings
- visionOS: indirect gestures (look to target, pinch thumb+finger to select) are "comfortable to perform at any distance" with "minimal movement"; direct gestures with raised arms are tiring and "best for infrequent use" — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)
- HIG: "Prioritize comfort... requires people to keep their arms raised for even a little while can be physically tiring, and repeating very similar movements... can stress muscles and joints"; avoid custom gestures requiring a specific hand; offer less-movement alternatives to multi-finger/two-hand gestures — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)
- HIG: add custom gestures only when necessary; they should be discoverable, straightforward, distinct, and not the only way; "Indicate when a gesture isn't available"; provide feedback that "communicates the extent and type of movement required" — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)
- HIG (macOS touch, Oct 2026): make hit targets bigger rather than controls; consider expandable controls for dense UI — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)
- visionOS hover effect is the cue that an element responds to indirect tap; rounder shapes easier to target — [Apple HIG: Eyes](https://developer.apple.com/design/human-interface-guidelines/eyes.md)
- WWDC24: indirect input lets people keep hands close to the body with small movements amplified — [WWDC24 10094 index](https://nonstrict.eu/wwdcindex/wwdc2024/10094/)
- Meta: "Midas touch" = accidental selections from hand tremor, tracking variance, and occlusion; recommends min target ~2.5–3° angular size and ≥12 mm spacing; index finger only for direct pokes; strong audio+visual feedback for poke/pinch/grab since no haptics; index-thumb pinch most accurate; forgiving direction-based gestures; pinch for selection, thumb-tap microgestures for contextual actions; avoid combining microgestures with ray targeting (fatiguing) — [Meta: Hands UI best practices](https://developers.meta.com/horizon/design/hands-ui-best-practices/); [Meta: Microgestures](https://prod.developers.meta.com/horizon/design/design-microgestures); [Meta: Hands gesture design](https://developers.meta.com/vr/design/hands-gesture-design/) (search summaries; pages not fetchable)
- Ultraleap: clear visual + audio cues for interactivity; visual change at contact moment; proximity "intersection glow" tinting fingertips as they near a target; extra visual feedback needed without haptics; signal tracking loss by fading hand visuals near edge of interaction zone; avoid interactions requiring stretching to tracking-zone edges — [Ultraleap XR design principles](https://docs.ultraleap.com/xr-guidelines/Getting%20started/design-principles.html); [Ultraleap 6 VR design principles](https://docs.ultraleap.com/ultralab/hand-tracking-vr-design.html); [Ultraleap automotive gesture HMI guide](https://ultraleap.com/company/news/blog/gesture-control-automotive-hmi) (search summaries)

### Inferences
- For a webcam Mac app: cursor = index-tip or palm-center position (relative, with acceleration, from a resting hand); "click" = thumb–index pinch (closest analog to Vision Pro tap); hold pinch = drag; two-finger/peace pose + vertical motion = scroll; pinch-with-both-hands or thumb–index spread = zoom (fallbacks per A1).
- Engagement/clutch: require an explicit "armed" pose (e.g., hand raised into zone with open palm for ~200–300 ms) before the cursor moves; dropping the hand or a fist = disengage ("lift the mouse"). This is the main anti-Midas-touch mechanism and also enables repositioning (clutching) like lifting a mouse.
- Apply pinch hysteresis (different thresholds for pinch-on vs pinch-off, normalized by hand size) and freeze the cursor position during the pinch onset (cursor tends to jump as fingers close — "Heisenberg effect").

### Gaps
- HoloLens (air tap, "hand ready" state, gaze-and-commit) guidance not fetched; no sourced specifics here.
- Leap Motion legacy guideline specifics (e.g., interaction box dimensions) not retrieved.

## B2. Fatigue (gorilla arm), Consumed Endurance, low-effort zones

### Takeaway
Arm fatigue is the dominant failure mode of mid-air input; Consumed Endurance (CE, CHI 2014) quantifies it from arm biomechanics, and newer work (NICER, SIGGRAPH 2024) adds recovery. Design for elbow-supported, hand-low interaction with small motions amplified by a transfer function.

### Cited Findings
- CE: metric derived from upper-arm biomechanics to quantify gorilla-arm fatigue (Hincapié-Ramos, Guo, Moghadasian, Irani, CHI 2014) — [UManitoba CE project](https://hci.cs.umanitoba.ca/projects-and-research/details/ce); [CE paper PDF](https://www.lri.fr/~anab/teaching/evaluation/papers/consumed-endurance.pdf)
- CE Workbench: open-source real-time CE tool using off-the-shelf skeleton tracking; used to design SEATO, an endurance-efficient mid-air keyboard layout — [CE Workbench](https://hci.cs.umanitoba.ca/Publications/details/the-consumed-endurance-workbench-a-tool-to-assess-arm-fatigue-during-mid-ai); [workbench paper](https://s159.ok.ubc.ca/uploads/8d3a3004-852a-4a6c-bf01-e04674aa450f.pdf)
- NICER (SIGGRAPH 2024) improves the metric with a recovery component — [SIGGRAPH history](https://history.siggraph.org/?p=164282)
- Purdue study on prolonged mid-air use ("Dispelling the Gorilla Arm Syndrome") — [Purdue news](https://research.purdue.edu/news/study-researches-gorilla-arm-fatigue-in-mid-air-computer-usage/); [AMiner entry](https://www.aminer.cn/pub/599c7ed3601a182cd28cc4e5/dispelling-the-gorilla-arm-syndrome-the-viability-of-prolonged-gesture-interactions)
- Apple: prefer indirect gestures; raised arms are tiring — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)

### Inferences
- CE's core finding (from training knowledge; verify in PDF): fatigue is driven by shoulder torque, so keeping the arm bent with hand near the body/lower plane and elbow resting dramatically lowers CE. For a laptop webcam, aim the interaction zone low (hand hovering just above keyboard/desk) — may require tilting view or an external camera.
- Use relative mapping + pointer acceleration so a ~10–15 cm hand travel covers the screen; offer "rest-to-pause" (hand down = pause, no penalty).
- Log engagement time and suggest breaks; consider a CE-like estimate from MediaPipe pose landmarks.

### Gaps
- Did not extract exact CE formula and numeric results from the paper.

## B3. Feedback: cursor states, hover, sound substitutes, confidence

### Takeaway
Because there is no physical contact, the system must expose its internal state continuously: tracking/not tracking, engaged, pinch "armed" (approaching threshold), pressed, dragging, and low confidence — via cursor glyph changes, a proximity/progress ring, and short audio clicks.

### Cited Findings
- Ultraleap: proximity glow as fingers near targets; visual change at contact; audio reinforcement of a successful press; fade hands as they approach tracking limits — [Ultraleap design principles](https://docs.ultraleap.com/xr-guidelines/Getting%20started/design-principles.html)
- Meta: strong audio + visual feedback for successful pokes/pinches/grabs since hands lack haptics — [Meta hands UI best practices](https://developers.meta.com/horizon/design/hands-ui-best-practices/)
- Apple: hover effect signals interactivity; feedback should help predict results and convey extent of movement required — [Apple HIG: Eyes](https://developer.apple.com/design/human-interface-guidelines/eyes.md); [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)

### Inferences
- Suggested overlay cursor states: hidden (no hand) → dim ring (hand seen, not engaged) → solid ring (engaged) → ring shrinking proportional to thumb–index distance (pinch progress) → filled dot + click sound (pressed) → trail (dragging) → scroll arrows (scroll mode) → pen nib (draw mode) → amber/flicker when landmark confidence low. Optional small camera-preview thumbnail with skeleton during onboarding.
- Use AXUIElementCopyElementAtPosition to highlight the hovered control (hover preview) and optionally snap to it — sticky targets compensate for webcam jitter.

### Gaps
- No controlled studies found specifically on cursor-state feedback for webcam desktop control.

## B4. Air drawing: smoothing, pen-up/pen-down, touch plane, beautification

### Takeaway
Pen-up/down segmentation is the hard problem; webcam research uses explicit delimiters (velocity-based end detection) or gestures, and an LSTM hobby attempt failed in real time. Use an explicit, reliable pen-down signal (pinch held = ink) and the 1€ filter for jitter/lag trade-off; add optional post-hoc beautification.

### Cited Findings
- 1€ filter (Casiez, Roussel, Vogel, CHI 2012): speed-adaptive low-pass — low cutoff at low speed reduces jitter, higher cutoff at high speed reduces lag; tuning: set beta=0, tune mincutoff while still, then raise beta to reduce lag; implementations in Python (PyPI OneEuroFilter), JS, TS, etc. — [Casiez 1€ page](https://cristal.univ-lille.fr/~casiez/1euro); [paper PDF](https://core.ac.uk/download/49919969.pdf); [PyPI OneEuroFilter](https://libraries.io/pypi/OneEuroFilter/0.2.0)
- Webcam air-writing: fingertip-velocity-based termination criterion as delimiter; 73.1% mean fingertip detection precision at 18.5 fps — [Mukherjee et al., arXiv 1809.03016](https://arxiv.org/pdf/1809.03016)
- Leap Motion air-writing (CHI 2013 EA): most handwriting recognition depends on pen-up/pen-down windowing; they aim to avoid explicit gestures — [Vikram et al.](https://www.cs.ucsb.edu/~leili/pubs/vikram2013handwriting.pdf)
- MediaPipe + BiLSTM pen-up/down classifier: author reports the task "too difficult" with poor real-time results; works offline only — [air-drawing GitHub](https://github.com/Haixing-Hu/air-drawing)

### Inferences
- Pen state options ranked for webcam: (1) pinch-to-ink (thumb+index touching = pen down) — most reliable, matches Vision Pro idiom; (2) pose switch (index extended = draw, open palm = hover); (3) depth "touch plane" — monocular webcams give weak absolute depth (MediaPipe z is relative), so a virtual plane based on hand scale is noisy; use only with hysteresis and calibration; (4) dwell to start/stop.
- Trim the first/last ~50–100 ms of each stroke to remove pinch-onset/offset hooks; resample by arc length; apply Catmull-Rom or Chaikin smoothing post-stroke; offer shape beautification (line/circle/rectangle snapping, à la $1 recognizer / "hold to snap") as opt-in.
- Map speed to stroke width for a natural ink look.

### Gaps
- No peer-reviewed webcam-specific comparison of pen-down methods found; depth-based touch-plane accuracy with monocular webcam not quantified.

## B5. Accessibility users and Apple built-in features as reference

### Takeaway
macOS already ships camera-based Head Pointer and Alternate Pointer Actions (facial expressions → click/right-click/double-click/drag), plus Dwell Control; these are the reference designs for pause/recalibrate actions, expression sensitivity sliders, and multiple action mappings. The app should be configurable and coexist with these.

### Cited Findings
- Alternate Pointer Actions lets users perform left/right/double-click and drag via keyboard shortcuts, assistive switches, or facial expressions (smile, open mouth); expressiveness level adjustable — [Apple Support: Pointer Control](https://support.apple.com/en-in/guide/mac-help/unac899/27/mac/27); [AbilityNet: pointer actions](https://mcmw.abilitynet.org.uk/how-to-set-up-pointer-actions-in-macos)
- Head Pointer moves pointer from head/face movement via camera; modes "Relative to Head" and "When Facing Screen Edges"; camera selectable; actions can pause and recalibrate pointer; Option-Command-F5 toggles — [thoughtbot intro](https://thoughtbot.com/blog/an-introduction-to-macos-head-pointer); [AbilityNet head movement](https://mcmw.abilitynet.org.uk/how-to-control-your-computer-with-head-movement-in-macos-12-monterey); [How-To Geek](https://www.howtogeek.com/788217/how-to-use-your-head-and-face-to-control-your-mac/)
- HIG: give more than one way to interact; avoid requiring specific body movements/positions; avoid hand-specific gestures — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)

### Inferences
- Provide: left/right-hand and either-hand mode, adjustable pinch sensitivity, dwell-click option (for users who can't pinch), keyboard/switch alternatives for click, "pause/recalibrate" gesture, large-cursor mode, and avoid conflicts if Head Pointer is also using the camera.
- Position mappings: offer both "relative" (trackpad-like, with clutch) and "absolute/edge" (like Head Pointer's screen-edge mode) to suit different users.

### Gaps
- Dwell Control (macOS Accessibility Keyboard dwell) specifics not fetched; Mac App Store guideline 2.4.5 concern means accessibility-positioned marketing may matter but no sourced policy text found.

## B6. Calibration and onboarding

### Takeaway
Onboarding should (1) request camera, then Accessibility just-in-time with clear rationale and live status, (2) show a camera preview with tracked skeleton to establish the interaction zone and lighting, (3) teach the engage/clutch, pinch-click, drag, scroll gestures with interactive practice targets, and (4) calibrate per-user pinch thresholds and the hand-to-screen mapping.

### Cited Findings
- Apple: make custom gestures easy to learn; offer moments to learn; if hard to describe in simple language/graphics, gesture is likely hard to perform — [Apple HIG: Gestures](https://developer.apple.com/design/human-interface-guidelines/gestures.md)
- Head Pointer's built-in "recalibrate" action is a precedent for in-session recalibration — [AbilityNet](https://mcmw.abilitynet.org.uk/how-to-control-your-computer-with-head-movement-in-macos-12-monterey)
- Ultraleap: avoid requiring stretches to the edge of tracking zone; fade feedback near edges — [Ultraleap design principles](https://docs.ultraleap.com/xr-guidelines/Getting%20started/design-principles.html)
- Permission UX pitfalls: grants silently stale after rebuild; restart sometimes needed — [OpenClaw mac permissions](https://docs.openclaw.ai/platforms/mac/permissions)

### Inferences
- Calibration steps: hold open hand at rest position (defines origin & hand scale); pinch 3 times (records closed/open thumb–index distance normalized by palm width → per-user thresholds with hysteresis); trace screen corners comfortably (defines gain so the comfortable box maps to full screen); test click on shrinking targets (tunes 1€ params / snapping).
- Persist calibration per camera + lighting profile; offer quick recalibrate gesture/menu item.
- Include a "practice sandbox" window where synthetic events are contained before enabling system-wide control, and a prominent global hotkey to disable instantly.

### Gaps
- No published onboarding study specific to webcam desktop hand control found.
