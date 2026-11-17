# Existing webcam hand-gesture cursor/touch and air-drawing software (macOS focus, as of Nov 2026)

Research constraints: airtouchapp.com, neural-lab.com and stardance.hackclub.com could not be fetched directly from this session (DNS blocked by the sandbox network policy), and the GitHub API was not available. Details for those come from search-engine snippets of the pages, and GitHub details come from rendered repo pages. Star counts and last-commit dates were mostly not visible; treat maturity judgements as approximate.

## 1. What is "AirTouch"? (all candidates, macOS install/run, cost, last update)

### Takeaway
At least five different things are called "AirTouch". The one that matches the user's description ("translates webcam hand gestures into cursor and touch-like events") **and** runs on a Mac is most likely the commercial **AirTouch at airtouchapp.com** (paid subscription, macOS 14+, Apple Silicon only). **Neural Lab's AirTouch**, the one with the blog, explicitly emulates mouse, keyboard and touch, but it is Windows, Linux and Android only, with macOS "on the way". Two free open-source macOS repos also use the name: NathN07/AirTouch (Swift/Vision) and KartBala07/AirTouch (Python/MediaPipe).

### Cited Findings
**A. AirTouch: "Control your Mac with a wave" (airtouchapp.com), commercial macOS app**
- Turns the Mac's existing webcam into a hands-free gesture controller for the whole Mac: move the cursor with your hand, pinch to click, and 10 built-in gestures — [airtouchapp.com (search snippet)](https://airtouchapp.com/)
- Requires macOS 14 Sonoma or later on Apple Silicon — [airtouchapp.com (search snippet)](https://airtouchapp.com/)
- Gestures include pinch-to-click, two-finger scroll, and a fist that opens Spotlight. Custom gestures can be bound to shortcuts, scripts or apps — [airtouchapp.com (search snippet)](https://airtouchapp.com/)
- Paid subscription. An account is used "only to manage your subscription and billing". No price or trial length surfaced in search — [airtouchapp.com (search snippet)](https://airtouchapp.com/)
- Also listed on a vibe-coding showcase site, which suggests a recent indie, AI-assisted build — [vibecode.net](https://vibecode.net/project/airtouch-airtouchapp)

**B. Neural Lab AirTouch (neural-lab.com), commercial, cross-platform except Mac**
- Webcam gesture control that emulates mouse, keyboard and touch inputs, so host apps (Unity, Unreal, TouchDesigner) need no code changes — [Sony Spatial Reality Display dev docs](https://sony.net/Products/Developer-Spatial-Reality-display/en/tips/air-touch)
- Runs on Android 11+, Windows 10+ and Linux. Shown at CES 2025 — [Engadget](https://www.engadget.com/computing/accessories/neural-labs-airtouch-brings-gesture-control-to-windows-and-android-devices-with-just-a-webcam-180031750.html)
- Neural Lab's 2026 blog posts and homepage still say macOS is "on the way", with no release date — [Neural Lab blog: Best Gesture Control Software in 2026](https://neural-lab.com/blog/best-gesture-control-software); [neural-lab.com](https://neural-lab.com/)
- Pricing: per device per month, free trial (7 days, card required), 17% saving on annual billing. Search snippets give the Individual plan as $29.99/mo or $299/yr (Personal works out to $24.92/mo when billed annually). Linux and Android need the Commercial plan — [Neural Lab pricing](https://neural-lab.com/pricing)
- At CES 2025 the price was about $30/mo for individuals and $300/mo for businesses — [Cybernews](https://cybernews.com/tech/neural-lab-hand-gesturing-feature-ces-2025/); [Engadget](https://www.engadget.com/computing/accessories/neural-labs-airtouch-brings-gesture-control-to-windows-and-android-devices-with-just-a-webcam-180031750.html)
- The "Neural Lab blog" the user mentioned is neural-lab.com/blog. Relevant posts include "AirTouch vs Zoom and Teams built-in gestures", "Apple Vision Pro-like gestures on any screen", "Best hand tracking software for Windows" and "Kinect alternatives 2026" — [AirTouch vs Zoom/Teams](https://neural-lab.com/blog/airtouch-vs-zoom-teams-gestures); [Vision Pro-like gestures on any screen](https://neural-lab.com/blog/vision-pro-gestures-any-screen); [Kinect alternatives](https://neural-lab.com/blog/kinect-alternatives-gesture-control)
- The company is based in El Dorado Hills, CA. Its first target market is healthcare (operating rooms). The CEO says "It is not as good as a mouse or touch screen yet, but much better than existing solutions" — [Comstocks](https://comstocksmag.com/web-only/startup-month-neural-lab)
- It has a partnership with Orbbec depth cameras (Gemini 336) for robustness to lighting — [Orbbec](https://www.orbbec.com/?p=45162)

**C. NathN07/AirTouch, open source, native macOS**
- A "virtual air trackpad" built on Apple's Vision framework, running on the Apple Silicon Neural Engine with no external dependencies. It has a floating HUD and passes clicks and keystrokes through to the active app — [GitHub NathN07/AirTouch](https://github.com/NathN07/AirTouch)
- Gestures: point to move, index-thumb pinch for left click, middle-thumb pinch for right click, pinch-hold to drag, two fingers for scroll and back/forward, 3–4 finger flicks for Spaces, Mission Control and App Exposé (Ctrl+arrows), peace sign to pause, open palm for a temporary pause — [GitHub](https://github.com/NathN07/AirTouch)
- Install: release zip, `brew install --cask airtouch` (per its README; whether this is in the official cask repo was not verified), or build with `./build.sh` using Xcode CLT. Needs Accessibility permission, re-granted after each rebuild — [GitHub](https://github.com/NathN07/AirTouch)
- 0 stars, 7 commits. A very new, tiny project — [GitHub](https://github.com/NathN07/AirTouch)

**D. KartBala07/AirTouch, open source, macOS Python**
- "Virtual touchscreen for macOS: track your fingertip with the MacBook camera and click with a poke." Stack: Python, MediaPipe HandLandmarker, OpenCV (calibration), AVFoundation, AppKit overlay and menu bar, CGEvent, packaged with PyInstaller — [GitHub KartBala07/AirTouch](https://github.com/KartBala07/AirTouch)
- Gestures: poke forward to click, pinch to click, double pinch to double-click, pinch-hold 0.35 s for right click/drag, two-finger scroll, fist for screenshot then open hand to paste, whole-hand swipes for windows and Mission Control — [GitHub](https://github.com/KartBala07/AirTouch)
- Install: `./setup.sh && ./run.sh`, or an arm64-only DMG. MIT license, 0 stars, 13 commits. Needs Camera, Accessibility and Screen Recording permissions — [GitHub](https://github.com/KartBala07/AirTouch)

**E. Other unrelated same-name projects**
- billodkeshwar/AirTouch (webcam touchless mouse), emaanali-cs/AirTouchOS, shravanihaj74/AirTouch (gesture plus voice). All are student-style Python/MediaPipe virtual mice — [billodkeshwar](https://github.com/billodkeshwar/AirTouch); [AirTouchOS](https://github.com/emaanali-cs/AirTouchOS); [shravanihaj74](https://github.com/shravanihaj74/AirTouch-Gesture-and-Voice-Controlled-Virtual-Mouse)
- airtouch.app is a phone app ("Control Your Phone with Hand Gestures", Personal tier $2.99/mo). There is also air-touch.ir (PC). Neither is the Mac product — [airtouch.app](https://airtouch.app/); [air-touch.ir](https://www.air-touch.ir/)
- Neonode's product is branded "Touchless"/"AirBar" sensor hardware, not a webcam AirTouch. No source in this research tied Neonode to the "AirTouch" name (not verified)

### Inferences
- For "try AirTouch first" on a MacBook Air M4, the realistic choices are (A) airtouchapp.com (paid, notarized commercial app, closest to a polished product) or the free repos (C) and (D). Neural Lab's AirTouch cannot be run natively on the Mac today. It would need a Windows VM or another device, and webcam passthrough in a VM is impractical.
- The user's description ("cursor and touch-like events") combined with the "Neural Lab blog" suggests they may have conflated Neural Lab AirTouch (which does touch emulation) with the Mac app. They should confirm which one they mean.
- All Mac candidates need Camera plus Accessibility permission (for CGEvent posting). A Continuity Camera iPhone 13 Pro should work as the camera source for Vision/AVFoundation-based apps.

### Gaps
- airtouchapp.com: exact price, trial, developer identity, version and release date could not be retrieved (site blocked from this sandbox).
- Last-commit dates and star counts over time for repos C and D (GitHub API unavailable). Both show 0 stars.
- Whether the Neural Lab blog has a specific "touch emulation" post. Only adjacent posts were found.

## 2. "A-Canvas" (MediaPipe + OpenCV + PyTorch air drawing with AI styles)

### Takeaway
No project named exactly "A-Canvas" could be confirmed. The closest match is **SkyCanvas** (SteveTM-git/skycanvas): webcam air-drawing whose sketches are turned into images with Stable Diffusion plus ControlNet on PyTorch, behind a FastAPI backend. A Hack Club "Stardance" project page (#19169) also came up for this query but could not be opened. There are dozens of generic MediaPipe/OpenCV "Air Canvas" repos.

### Cited Findings
- SkyCanvas: hand-gesture drawing canvas, webcam, FastAPI backend, sketch to photoreal image via Stable Diffusion plus ControlNet (PyTorch) — [GitHub SteveTM-git/skycanvas](https://github.com/SteveTM-git/skycanvas) (via search summary)
- A Hack Club Stardance project (#19169) was returned for the "A-Canvas … PyTorch AI styles" query, but its content could not be fetched — [stardance.hackclub.com/projects/19169](https://stardance.hackclub.com/projects/19169)
- Hackster "AI gesture art": MediaPipe tracking, OpenCV drawing, and SketchRNN converting strokes to sketches — [Hackster](https://hackster.io/hackathoners/ai-gesture-art-d36827)
- Generic air-canvas repos: antara33/AirCanvas-AI (shape recognition, undo/redo, gallery; modular hand_tracker/gesture_controller/shape_recognizer); d4min/aircanvas (pinch to draw); robert-mcdermott/aircanvas (browser-based, MediaPipe Hands plus HTML5 canvas, aimed at video meetings, dwell or second-hand fist to draw); 1112005-mb, Bhanu1294, Sahilsingh0808 and others — [antara33](https://github.com/antara33/AirCanvas-AI); [d4min](https://github.com/d4min/aircanvas); [robert-mcdermott](https://github.com/robert-mcdermott/aircanvas); [GitHub topic air-canvas](https://github.com/topics/air-canvas?o=desc&s=updated)
- darthdaenerys/AI-Virtual-Painter is a classic OpenCV/MediaPipe virtual painter — [GitHub](https://github.com/darthdaenerys/AI-Virtual-Painter)

### Inferences
- These Python repos generally run on Apple Silicon (MediaPipe ships macOS arm64 wheels), but they draw in an OpenCV window rather than system-wide. SD/ControlNet style transfer on an M4 with 16 GB would run via PyTorch MPS, which is slow (tens of seconds per image) but feasible.
- Note: recent mediapipe releases removed the legacy `mp.solutions.hands` API, so many older tutorials and repos break on current installs and must use the Tasks API (HandLandmarker) — [so-you-knock repo note](https://github.com/so-you-knock/gesture-controlled-virtual-mouse)

### Gaps
- The identity of "A-Canvas" is unconfirmed. The user should supply a URL. The Stardance page needs network access to verify.

## 3. Other commercial / free apps and platform features

### Takeaway
On macOS specifically, the most credible free options are **Pawvis** (MIT, Swift and Apple Vision, the most mature open-source Mac app found, with 324 commits) and **Conductor** (Swift and Vision menu-bar app). Apple ships **Head Pointer** (head and face, not hands) and **FaceTime/video Reactions** (hand gestures trigger effects only, no cursor). **Ultraleap** now supports Apple Silicon but needs its own hardware. Google **Project Gameface** is face-based and Windows/Android only.

### Cited Findings
**Pawvis (pawvis.app, alexandriax/pawvis)**
- Free and open source (MIT), macOS 14+. Uses Apple Vision for 21-joint hand pose, on-device. Works with built-in, USB or Continuity Camera — [pawvis.app](https://pawvis.app/); [GitHub](https://github.com/alexandriax/pawvis); [Product Hunt](https://www.producthunt.com/products/pawvis)
- Gestures: open hand moves the cursor, index dip clicks, pinky dip right-clicks, finger held down drags, quick taps for double/triple click, optional dwell click, two folded fingers to scroll, a two-hand wave to stop. Has a custom gesture library plus user-trained gestures. Optional voice "autopilot" (macOS 26 with Apple Intelligence) can hand computer-use tasks to Codex or Claude Code — [GitHub](https://github.com/alexandriax/pawvis)
- 26 stars, 324 commits, Swift Package. Distributed as a zip; the brew cask is still pending — [GitHub](https://github.com/alexandriax/pawvis)
- Known weakness: Vision needs a well-lit hand, so it gets flaky in dim or backlit rooms. Continuous per-frame inference costs CPU and power — [pawvis.app](https://pawvis.app/)

**Conductor (NikAtNight/conductor, also dev-talix/conductor)**
- Menu-bar app on Apple Vision, macOS 14+, Apple Silicon and Intel. Gestures: open hand held still to take control, pinch to click, pinch-hold to drag, thumb-middle for right click, two-hand pinch to zoom, two-finger flick for back/forward, display switching. Gestures are remappable — [GitHub NikAtNight/conductor](https://github.com/NikAtNight/conductor); [dev-talix fork](https://github.com/dev-talix/conductor)
- 55 commits, 0 stars, zip release or `build-app.sh` — [GitHub](https://github.com/dev-talix/conductor)

**Apple built-ins**
- Head Pointer (Accessibility → Pointer Control → Alternate Control Methods, since macOS 10.15.4) moves the pointer by head movement via the camera. Facial expressions (smile, open mouth, raise eyebrows, tongue) can be mapped to clicks. It has no hand tracking — [thoughtbot](https://thoughtbot.com/blog/an-introduction-to-macos-head-pointer); [How-To Geek](https://www.howtogeek.com/788217/how-to-use-your-head-and-face-to-control-your-mac/); [Apple support](https://support.apple.com/guide/mac-help/unac899/mac)
- Sonoma video Reactions: thumbs up/down, two-hand heart, "rock on" and similar trigger 3D effects in any camera app. Requires Apple Silicon or Continuity Camera (iPhone 12+). This is proof that Apple's on-device hand-gesture recognition runs system-wide, but it is not exposed as cursor control — [MacRumors](https://macrumors.com/how-to/trigger-reaction-effects-facetime-mac); [Tom's Guide](https://www.tomsguide.com/how-to/use-reactions-in-video-calls-on-macos-sonoma); [critter.blog](https://critter.blog/2023/10/25/hand-gestures-in-macos-sonoma/)

**Google Project Gameface**
- Open-source (I/O 2023) "hands-free gaming mouse" driven by head movement and facial gestures (MediaPipe face landmarks). Windows and Android, not macOS, and not hands — [Google blog](https://blog.google/technology/ai/google-project-gameface/); [Google Developers blog](https://developers.googleblog.com/2023/06/project-gameface.html)

**Ultraleap (Leap Motion)**
- Leap Motion Controller 2 tracking software v6.2.0 (22 Sep 2025) has a native Apple Silicon build. The minimum is macOS 11 on Apple Silicon. It needs the Ultraleap IR hardware, not a webcam — [Ultraleap downloads](https://www.ultraleap.com/downloads/leap-motion-controller-2)
- Older Mac SDK and plugin ecosystems lagged. For example, Isadora's Leap Motion Watcher still needs Rosetta — [Troikatronix](https://support.troikatronix.com/support/solutions/articles/13000090672-where-is-the-leap-motion-watcher); [Troikatronix forum](https://community.troikatronix.com/post/50971)

**Other**
- Handsfree.js (browser face, hand and pose tracking library, npm `handsfree` latest 8.5.1). Archive status was not confirmed, and it appears unmaintained (no recent release dates found) — [npm](https://npmjs.com/package/handsfree); [jsDelivr README](https://cdn.jsdelivr.net/npm/handsfree@8.5.1/README.md)
- UCL MotionInput: academic and industry "touchless computing" suite (Windows) — [UCL](https://www.ucl.ac.uk/engineering/computer-science/collaborate/ucl-industry-exchange-network-ucl-ixn-industry-partners/touchless-computing-ucl-motioninput-3)
- Intuiface lists gesture-control and touch-emulation partners (Neural Lab is in this category) — [Intuiface](https://www.intuiface.com/tech-ecosystem/gesture-control-touch-emulation)
- CoolTouch is a Mac driver giving full gesture support for USB touchscreens (physical touchscreens, not webcam). It is relevant as a target for "touch-like events" on macOS — [cooltouch.app](https://cooltouch.app/)

### Inferences
- macOS has no public API for injecting real multitouch or NSTouch trackpad events. All webcam Mac apps synthesize mouse, scroll and keyboard CGEvents and map gestures to shortcuts (Mission Control and similar). "Touch emulation" on Mac therefore means pointer emulation. (Inference from every Mac app's documented gesture-to-shortcut mapping. No direct source was found stating the API limit.)
- Pawvis is the strongest free baseline to compare against any custom build. The commercial AirTouch is the polished paid baseline.

### Gaps
- ManoMotion, Clay AIR and Kinemic: no 2025–26 information surfaced in this pass. Their status is unknown; they are historically SDKs for AR/mobile and Kinemic uses a wristband, so they are not consumer Mac apps.
- Mac App Store webcam gesture apps were not found in search. None were confirmed.

## 4. Popular GitHub projects (virtual mouse / air canvas) and macOS-native vs Python

### Takeaway
The open-source landscape is dominated by Python + MediaPipe + OpenCV + PyAutoGUI "AI virtual mouse" projects, which are cross-platform but prototype-grade. The best known is Viral-Doshi/Gesture-Controlled-Virtual-Mouse (about 700 stars in an undated snapshot). The newer, higher-quality Mac tools (Pawvis, Conductor, NathN07/AirTouch) are native **Swift + Apple Vision (VNDetectHumanHandPoseRequest) + CGEvent** menu-bar apps. Hybrids also exist: Python + MediaPipe + PyObjC/AppKit (KartBala07) and Electron + MediaPipe Tasks + nut-js.

### Cited Findings
- Viral-Doshi/Gesture-Controlled-Virtual-Mouse (also xenon-19): MediaPipe, OpenCV, Python, gestures plus voice assistant. About 707 stars and 233 forks per an undated cached page — [GitHub](https://github.com/Viral-Doshi/Gesture-Controlled-Virtual-Mouse)
- Many derivatives: Ishansourav (MediaPipe plus CNN plus voice, Windows-oriented), ayushcodes13 (OpenCV, MediaPipe, PyAutoGUI prototype), so-you-knock (moved to the MediaPipe Tasks API because legacy hands was removed), BhoomiBliss ("interaction prototype, not accessibility-certified") — [Ishansourav](https://github.com/Ishansourav/gesture-virtual-mouse); [ayushcodes13](https://github.com/ayushcodes13/gesture-controlled-virtual-mouse); [so-you-knock](https://github.com/so-you-knock/gesture-controlled-virtual-mouse); [BhoomiBliss](https://github.com/BhoomiBliss/VirtualMouse)
- Mac-targeted Python/JS repos: wanderermonk/GestureControl (MediaPipe; cursor, click, drag, scroll, desktop switching, window snapping, volume/media); Nithillen/Mac_Gesture_Control (MediaPipe plus CoreGraphics CGEvent; volume, Mission Control); bhavyavashisth/HandTrack-MacControl; sachin-perumandla/gesture--mouse (Electron plus MediaPipe Hand Landmarker plus nut-js, offline); GestuReMote (MediaPipe, "only macOS supported so far") — [wanderermonk](https://github.com/wanderermonk/GestureControl); [Nithillen](https://github.com/Nithillen/Mac_Gesture_Control); [bhavyavashisth](https://github.com/bhavyavashisth/HandTrack-MacControl); [sachin-perumandla](https://github.com/sachin-perumandla/gesture--mouse); [samvoisin/gesture-control](https://github.com/samvoisin/gesture-control)
- Native Swift/Vision Mac apps: Pawvis, Conductor and NathN07/AirTouch (all macOS 14+, Accessibility plus Camera permission, zip or brew distribution) — [Pawvis](https://github.com/alexandriax/pawvis); [Conductor](https://github.com/dev-talix/conductor); [NathN07/AirTouch](https://github.com/NathN07/AirTouch)
- Common requirements for all Mac projects: Camera permission plus Accessibility permission for the app or terminal posting mouse events — [search summary of Mac repos; e.g. sachin-perumandla](https://github.com/sachin-perumandla/gesture--mouse)

### Inferences
- Typical Python architecture: OpenCV capture → MediaPipe 21-landmark hand model → finger-up/pinch-distance heuristics → coordinate mapping into a reduced "active box" with smoothing (EMA) → PyAutoGUI/pynput/CGEvent. Weaknesses: jitter, no click debouncing, the PyAutoGUI 0.1 s default pause, Python packaging and permissions friction on macOS, and frequent breakage from MediaPipe API changes.
- Typical native architecture: AVCaptureSession (supports Continuity Camera) → VNDetectHumanHandPoseRequest per frame (ANE-accelerated) → gesture state machine → CGEvent post, delivered as a menu-bar or HUD app. Advantages: no Python runtime, low latency, signed app bundle. Weakness: Vision hand pose is less robust in low light (per Pawvis).
- Maturity ranking (Mac): Pawvis (most commits, documented, MIT) > Conductor > NathN07/KartBala07 AirTouch (very early, 0 stars). The commercial airtouchapp.com is likely the most polished but is closed and paid.

### Gaps
- Precise star counts and last-commit dates for all repos (GitHub API blocked). The ~707-star figure is undated.
- No Hacker News "Show HN" thread for these Mac apps was located. Pawvis was on Product Hunt.
