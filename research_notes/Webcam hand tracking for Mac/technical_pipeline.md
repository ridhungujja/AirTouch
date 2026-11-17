# Technical Pipeline: Webcam Hand Tracking Cursor/Touch/Drawing on macOS Apple Silicon

Research scope: perception engine choice, camera capture, smoothing, mapping, gesture detection, robustness, performance. Target hardware: MacBook Air M4 (16 GB), built-in 1080p camera, optional iPhone 13 Pro via Continuity Camera. Researched November 2026.

**Note on sources:** This session's network sandbox blocked several primary sources: ai.google.dev, developers.google.com (the MediaPipe docs), gery.casiez.net (1€ filter page), arxiv.org, docs.ultraleap.com, huggingface.co, cdn.jsdelivr.net and cloud.flect.co.jp. As a result, some MediaPipe and 1€ facts below come from search-result summaries of those pages, not from full-page reads. Apple WWDC pages were read in full. Claims I could not source are labelled as inferences or listed under gaps.

## 1. Perception engine: Apple Vision vs MediaPipe vs others on Apple Silicon

### Takeaway
Both Apple Vision hand pose and MediaPipe Hand Landmarker output the same 21-keypoint hand skeleton. Vision is the natural choice for a native Swift Mac app:
- It is built into the OS (no dependencies) and gives chirality and per-point confidence.
- Apple says Vision runs on the Neural Engine when available, but publishes no hand-pose latency figures.

MediaPipe has two advantages over Vision:
- It is the only one of the two with published latency numbers (about 12–17 ms on a Pixel 6).
- It adds relative depth (z) and metric "world landmarks".

No published head-to-head benchmark exists for the two on a Mac. Benchmark both on the M4 before committing.

### Cited Findings
**Apple Vision (VNDetectHumanHandPoseRequest, macOS 11+)**
- It returns 21 landmarks: four per finger and thumb (TIP, DIP, PIP, MCP for fingers; TIP, IP, MP, CMC for the thumb) plus the wrist. Coordinates are normalized with a lower-left origin, so the sample flips y with `1 - y` to get AVFoundation coordinates. Each point has its own confidence — [WWDC20 "Detect Body and Hand Pose with Vision"](https://developer.apple.com/videos/play/wwdc2020/10653/)
- `maximumHandCount` defaults to 2. Pose is computed "for every detected hand", so a higher count adds latency. Apple's sample sets it to 1 for performance — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Apple lists these known failure cases:
  - hands near the image edges (partly cut off);
  - hands parallel to the camera's viewing direction (a "karate chop");
  - gloves;
  - feet sometimes detected as hands.

  — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Apple suggests detecting the hand with the pose request, then following it with `VNTrackObjectRequest`, which "is good at maintaining object identifiers as things move off screen or become occluded temporarily" — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Chirality was added in 2021. `chirality` is `.left`, `.right` or `.unknown`, and it is predicted for each hand independently — [WWDC21 "Classify hand poses and actions with Create ML"](https://developer.apple.com/videos/play/wwdc2021/10039/)
- WWDC21 guidance on conditions and revisions:
  - Best results come with the hand under about 3.5 m (11 ft) from the camera and without extreme dark or bright lighting.
  - When more hands are visible than `maximumHandCount` allows, Vision returns the "most prominent and central" hands.
  - Pin `revision` to avoid behaviour changes when the SDK updates.

  — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- A newer Swift-native API exists: the `DetectHumanHandPoseRequest` struct (macOS 15 / iOS 18 and later), which returns `HumanHandPoseObservation` with an optional `chirality` — [Apple docs: DetectHumanHandPoseRequest](https://developer.apple.com/documentation/vision/detecthumanhandposerequest.md); [HumanHandPoseObservation](https://developer.apple.com/documentation/vision/humanhandposeobservation.md)
- Vision gives no z or depth output. Points are 2D normalized positions plus confidence; for example, the hand-action input is shaped 45×3×21 (x, y, confidence) — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- Compute placement: one secondary blog says Vision uses the Neural Engine when available, then the GPU, then the CPU. Apple does not document this for hand pose — [blakecrosley.com](https://blakecrosley.com/blog/vision-framework-built-in)
- The nearest published timing is for **body** pose, not hands: about 40 ms per `VNDetectHumanBodyPoseRequest` on an iPhone 11, capping it near 45 fps. Parallel queues and lower input resolution did not raise throughput — [Apple Developer Forums thread 679909](https://developer.apple.com/forums/thread/679909)
- A 2021 developer write-up found that Vision's hand *recognition* is accurate, but individual joint coordinates "sometimes ... can be very wrong", so the author added filters — [FLECT blog (search summary; page fetch blocked)](https://cloud.flect.co.jp/entry/2021/09/21/084902)

**MediaPipe Hand Landmarker**
- Official benchmark: HandLandmarker (full) averages 17.12 ms on CPU and 12.27 ms on GPU on a Pixel 6 — [MediaPipe Hand Landmarker guide (search summary)](https://ai.google.dev/mediapipe/solutions/vision/hand_landmarker)
- The pipeline is a palm detector followed by a landmark model, per the 2020 MediaPipe Hands paper. Tracking reuses the previous frame's hand region, so the palm detector only re-runs when tracking is lost. This is MediaPipe's documented design; I could not re-fetch the docs to quote the exact default thresholds — [search summary of MediaPipe paper/docs](https://www.emergentmind.com/topics/mediapipe-hands)
- On Apple Silicon (M1 Pro, M2), Python logs show the Metal GPU delegate initializing ("Created TensorFlow Lite delegate for Metal"). In one 2024 report the GPU path failed on macOS while `Delegate.CPU` worked — [mediapipe issue #6041](https://github.com/google-ai-edge/mediapipe/issues/6041); [issue #5568](https://github.com/google-ai-edge/mediapipe/issues/5568)
- Google lists Python 3.9–3.14 on macOS Apple Silicon as supported, with GPU opt-in per task through `BaseOptions.delegate`. One GitHub project said MediaPipe did not yet support Python 3.13, so check the version — [MediaPipe docs (search summary)](https://developers.google.com/edge/mediapipe/solutions/vision/hand_landmarker?tab=Web); [GitHub project README (search summary)](https://github.com/Pythonpreran/Aircursor)
- A Core ML conversion of the MediaPipe hand models exists on Hugging Face (`hugging-mac/mediapipe-hand-coreml`), which could run on the ANE. Its benchmarks could not be fetched — [Hugging Face](https://huggingface.co/hugging-mac/mediapipe-hand-coreml)

**Accuracy comparisons**
- A third-party 2021 comparison measured MediaPipe Hands at PCK@0.2 = 0.703 on STB and 0.693 on RHD. That is below InterHand (0.8245 on STB) and the authors' own model (0.895 on STB, 0.866 on RHD) — [Fast Monocular Hand Pose Estimation on Embedded Systems, arXiv 2102.07067](https://arxiv.org/pdf/2102.07067)
- For body pose (not hands), a yoga study scored MediaPipe at 93.4% accuracy and Apple Vision at 89.6%, with lower variance for MediaPipe. One pose (headstand) reversed the result — [eight-angle blog](https://eight-angle.pages.dev/en/blog/vision-vs-mediapipe-yoga-pose-detection)
- RTMPose-Hand: the only evidence found is one forum anecdote that it handles gloves better than MediaPipe but is "too flickery" — [Google AI forum](https://discuss.ai.google.dev/t/keypoint-hand-pose-model-that-works-on-gloved-hands/143513)

### Inferences
- On an M4 both engines should comfortably run one hand at 30–60 fps. A Pixel 6 manages about 12–17 ms per frame, and the M4 is a much stronger chip. Even a 40 ms Vision call on an iPhone 11 would likely drop well under 16 ms on M4-class hardware. This must be measured.
- For a native Swift app, Vision has the lowest integration cost: no Python, no TFLite, chirality built in, and OS-managed ANE/GPU scheduling. MediaPipe's advantages are relative z (useful for "push to click") and published latency figures.
- Practical choice: prototype with Vision (setting `maximumHandCount = 1` or `2` and pinning `revision`). Build the pipeline so the engine can be swapped behind a "21 points + confidence + handedness" interface.

### Gaps
- No published fps or latency for Vision hand pose on any M-series Mac. Apple does not say whether ANE is used for this request.
- No head-to-head Vision vs MediaPipe *hand* accuracy or jitter study.
- No RTMPose-Hand latency on Apple Silicon, and no measured fps for MediaPipe Python on M-series.
- The exact current MediaPipe defaults (`min_hand_detection_confidence`, `min_hand_presence_confidence`, `min_tracking_confidence`, all commonly 0.5) could not be confirmed, because the docs host was blocked.

## 2. Camera pipeline (AVFoundation, frame rate, exposure, Center Stage, Continuity Camera)

### Takeaway
Pick an `activeFormat` whose frame-rate ranges include the rate you want, then set the min/max frame duration. Whether the MacBook Air's built-in camera exposes 60 fps is unconfirmed; enumerate `device.formats` on the actual machine. Center Stage and automatic exposure can both move or change the image under the tracker. Detect them, and where the API allows, turn them off or lock them.

### Cited Findings
- A device's `formats` lists the formats it supports. Each format advertises its supported frame-rate ranges. To get higher frame rates, choose a suitable `AVCaptureDevice.Format` as `activeFormat` — [Apple: capture device formats](https://developer.apple.com/documentation/avfoundation/capture-device-formats.md); [Apple: AVCaptureDevice.Format](https://developer.apple.com/documentation/avfoundation/avcapturedevice/format.md)
- Configuration changes need `lockForConfiguration()` first and `unlockForConfiguration()` afterwards. Holding the lock longer than needed "may degrade capture quality in other apps sharing the device" — [Apple: lockForConfiguration](https://developer.apple.com/tutorials/data/documentation/avfoundation/avcapturedevice/lockforconfiguration().md)
- `.autoExpose` adjusts exposure once and then locks it. However, developers report brightness still changing with scene content under locked or custom exposure (on iPhone), and the mode changing within the first second after locking — [Apple: autoExpose](https://developer.apple.com/tutorials/data/documentation/avfoundation/avcapturedevice/exposuremode-swift.enum/autoexpose.md); [Forums 780406](https://developer.apple.com/forums/thread/780406); [Forums 749574](https://developer.apple.com/forums/thread/749574)
- Face-driven auto exposure (`isFaceDrivenAutoExposureEnabled`) can be controlled, which matters when a hand passes in front of the face — [Apple docs](https://developer.apple.com/documentation/avfoundation/avcapturedevice/isfacedrivenautoexposureenabled.md)
- Center Stage:
  - It is controlled with `AVCaptureDevice.centerStageControlMode` (`.user`, `.app` or `.cooperative`) together with `isCenterStageEnabled`. Changing related properties such as `centerStageRectOfInterest` requires the configuration lock.
  - KVO on `isCenterStageEnabled` reportedly does not fire when the user toggles it from Control Center.
  - The RecordKit team toggles Center Stage from code (`.cooperative` mode, then flipping `isCenterStageEnabled`) as a workaround for a Studio Display bug.

  — [Apple: centerStageRectOfInterest](https://developer.apple.com/documentation/avfoundation/avcapturedevice/centerstagerectofinterest.md); [Forums 692163](https://developer.apple.com/forums/thread/692163); [Nonstrict blog](https://nonstrict.eu/blog/2025/studio-display-camera-fails)
- Users report Center Stage being hard to turn off in some apps on macOS Tahoe — [Apple Discussions](https://origin-discussions2-us-dr-prz.apple.com/thread/256186991)
- Continuity Camera: a competing vendor, Reincubate, says Apple does not let apps change Continuity Camera's frame rate. This is a vendor claim, not Apple documentation — [Digital Trends (Camo 18)](https://digitaltrends.com/computing/camo-18-takes-on-continuity-camera)
- The WWDC21 "What's new in camera capture" session covers system video effects (Center Stage, Portrait, mic modes) that are user-controlled through Control Center — [WWDC21 10047](https://developer.apple.com/wwdc21/10047)
- In Apple's hand-action demo, ARKit delivered 60 fps but the model was trained at 30 fps, so every other frame was dropped. The frame rate fed to temporal classifiers must match training — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)

### Inferences
- With Center Stage on, the crop pans and zooms. An absolute hand-to-screen mapping then drifts even when the hand is still. On startup, set `centerStageControlMode = .app` (or `.cooperative`) and disable it, warn the user if that fails, and watch for unexpected zoom changes in the frame.
- Motion blur during fast motion grows with exposure time. In low light, auto exposure lengthens exposure (often to 1/30 s or more, which also caps fps), and landmarks smear. Prefer a format that keeps 30–60 fps, and accept a darker image rather than a lower frame rate. Tell users to light the hand from the front.
- Ask for BGRA or 420v pixel buffers at around 720p rather than 1080p. Hand models resize the input to a small tensor anyway, and less data means less copying. Set `alwaysDiscardsLateVideoFrames = true` so stale frames never queue (standard AVFoundation practice).
- Desk View is an ultra-wide-derived, top-down Continuity Camera feed. It could support a "draw on the desk" mode, but I found no technical details on accessing it for tracking.

### Gaps
- Whether the M4 MacBook Air's 12 MP Center Stage camera exposes 60 fps formats through AVFoundation.
- What fps Continuity Camera (iPhone 13 Pro) delivers to a Mac app.
- Whether exposure lock (`isExposureModeSupported(.locked)`) is supported on the built-in Mac camera.
- Desk View API details.

## 3. Smoothing and jitter: 1€ filter, Kalman, prediction

### Takeaway
Use a 1€ filter on each landmark, or at least on the cursor point. Start with `min_cutoff ≈ 1 Hz` and `beta = 0`. Raise `beta` during fast movements until lag disappears, scaling it by factors of 10 to match your units. Pass real timestamps so the filter adapts to frame-rate changes. Kalman filters or prediction can recover a little latency but can overshoot.

### Cited Findings
- The authors' tuning procedure:
  1. Set `beta = 0` and `mincutoff` to a reasonable middle value such as 1 Hz.
  2. Move quickly and increase `beta` until the lag goes away.

  Beta's order of magnitude depends on the data's units, so try values such as 0.001 or 0.0001 and adjust in factors of 10 — [Casiez 1€ filter page (search summary)](https://cristal.univ-lille.fr/~casiez/1euro)
- Jitter while still means lowering `mincutoff`; lag during fast movement means raising `beta` — [DFKI multisensor-pipeline docs](https://dfki-interactive-machine-learning.github.io/multisensor-pipeline/Documentation/modules/signal/filtering); [libossia docs](https://ossia.io/libossia/html/structossia_1_1one__euro__filter.html)
- `mincutoff` is a cutoff frequency in Hz (lower removes more jitter). `beta` is the speed coefficient (higher reacts faster) — [Needle Engine docs](https://engine.needle.tools/docs/api/OneEuroFilter)
- One hand-pose wrapper defaults to `min_cutoff = 1.0` and `beta = 0.02`. Raise beta if fast clenches feel laggy; lower it if the hand twitches at rest — [MyoGestic docs](https://nsquaredlab.github.io/MyoGestic/how-to/post-process-output/)
- Monado/Collabora found the 1€ filter added too much latency for XR hand tracking and changed approach — [Collabora blog](https://test.www.collabora.com/news-and-blog/blog/2022/05/31/monado-hand-tracking-hand-waving-our-way-towards-a-first-attempt/)
- Apple's sample code uses no smoothing filter. It gates points on confidence (> 0.3 in WWDC20, > 0.2 in WWDC21) and requires 3 consecutive frames of evidence before changing state — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/); [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- Some GitHub hand-mouse projects use a Kalman filter for "high accuracy, low latency" — [awesome.ecosyste.ms index of hand-mouse projects](https://awesome.ecosyste.ms/projects/github.com%2Fmark0011astra%2Fhand_mouse)

### Inferences
- **Units matter for beta.** With normalized coordinates (0–1), beta will be much larger than with screen pixels. Filter in one fixed space, either normalized image coordinates or screen points, and tune there. The common `dcutoff = 1 Hz` default for the speed derivative comes from the reference implementation, which I could not re-verify here.
- **Suggested starting values** (screen-pixel units): `min_cutoff` 0.5–1.5 Hz and `beta` around 0.005–0.05. For drawing, use more smoothing (a lower min_cutoff); for pointing, a higher beta. These are inferences to tune, not sourced numbers.
- **Two filters.** Use one heavily smoothed signal for the cursor and a separate, lightly smoothed one for gesture distances such as pinch. Heavy smoothing on the pinch distance delays clicks.
- **Dead zone.** Ignore movement under about 0.2–0.5% of the active region when the hand is near still, which lets the user hold the cursor steady for a click. This is an inference.
- **Latency budget** (inference): capture exposure plus readout (about 17–33 ms), inference (about 5–15 ms), filter lag (about 10–30 ms when slow, close to zero when fast with a good beta), plus display. The end-to-end target is about 60–90 ms at 30 fps, lower at 60 fps. A short linear extrapolation (velocity × about 1 frame) can hide around 16–33 ms, but it overshoots when the hand stops; fade it out at low speed.

### Gaps
- Could not fetch the original 1€ paper (CHI 2012) for its measured comparisons against Kalman, moving average and similar filters.
- No sourced end-to-end latency figure for webcam hand cursors on a Mac.

## 4. Mapping: absolute vs relative, active region, stable landmarks

### Takeaway
Map a reduced *active region* (the central part of the camera frame) to the screen, so the user never has to reach the frame edges, where Vision's tracking is worse. Point with a stable landmark such as the index MCP or the palm centroid rather than the fingertip, because pinching moves the fingertip. For fine work, offer a relative ("clutch") mode.

### Cited Findings
- Apple warns that hand pose is unreliable near the image edges because of partial occlusion. This supports mapping a smaller inner region of the frame to the full screen — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Typical open-source hand mice map the index fingertip to the cursor directly (absolute), which is the approach prone to jitter. No popular repository was found that implements clutch or relative mapping — [search of GitHub hand-mouse projects](https://github.com/petehsu/LyraPointer); [hamoco on PyPI](https://pypi.org/project/hamoco/1.0.0/)
- An SFU study found un-instrumented in-air pointing reached less than 75% of mouse throughput. The authors suggested latency, click detection and tracking quality as causes — [SFU in-air pointing paper](https://vvise.iat.sfu.ca/user/data/papers/inairpointing.pdf)

### Inferences
- **Active region.** Map about the central 50–70% of the frame (with margins of roughly 15–25% on each side) to the screen. Recalibrate the region from the hand's position when tracking starts.
- **Mirror the image.** Flip x for a front camera so movement feels natural. Note that `chirality` refers to the subject's real hand, so verify whether it flips under your mirroring.
- **Avoiding cursor shift when pinching.** The thumb and index tips converge during a pinch, so a fingertip-driven cursor jumps at the moment of the click. Options:
  - drive the cursor from the index MCP, the palm centroid (wrist plus the MCPs), or the midpoint of the thumb and index MCPs;
  - freeze or heavily damp cursor movement for about 100–200 ms around the pinch onset;
  - use the cursor position recorded just before the pinch began (keep a small history buffer).
- **Relative/clutch mode.** Move the cursor by the hand's delta while "engaged", for example while the hand is open, and let the user re-centre with a fist or by dropping the hand. Add trackpad-style acceleration: gain rises with speed, about 1× when slow up to about 3–4× when fast. This trades absolute predictability for precision and less arm fatigue.
- **Drawing mode.** Use the fingertip itself, since the stroke should follow the tip, with light smoothing. Treat pinch-and-hold as "pen down".

### Gaps
- No sourced HCI numbers on the best active-region size or gain functions for webcam hand cursors. No study of fingertip vs MCP pointing stability was found.

## 5. Click and gesture detection

### Takeaway
Detect pinches from the thumb-tip to index-tip distance, *normalized by hand size*, using two thresholds (hysteresis) and a few frames of debounce in a small state machine. Apple's own sample uses a fixed 40-pixel threshold with 3 frames of evidence. Normalizing by hand size makes the threshold independent of distance from the camera.

### Cited Findings
- Apple's WWDC20 pinch-to-draw sample:
  - `pinchMaxDistance = 40` (pixels between the thumb and index tips) and `evidenceCounterStateTrigger = 3` frames.
  - States: `unknown → possiblePinch → pinched → possibleApart → apart`.
  - Points collected during "possible" states are buffered, then drawn when the state becomes `.pinched` or discarded when it becomes `.apart`. This keeps the delay from cutting off the start of strokes.

  — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Ultraleap's Pinch Detector computes a pinch value from the fingertip-to-thumb distance and uses different pinch and unpinch thresholds (hysteresis) to prevent flicker — [Ultraleap docs (search summary)](https://docs.ultraleap.com/xr-and-tabletop/xr/unity/plugin/features/pinch-detector.html)
- Handsfree.js (MediaPipe on a webcam) scales the thumb-to-finger distance by frame height, compares it to a threshold, and requires a number of frames (tolerance) before triggering — [Handsfree.js pinchers plugin](https://handsfreejs.netlify.app/ref/plugin/pinchers)
- Apple's hand-action demo uses a high confidence threshold (pose confidence > 0.9) plus a "Background" class to avoid false triggers (the Midas-touch problem). Running the classifier at intervals reduces flicker — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- Temporal gestures (swipes and similar) in Apple's pipeline use a 1.5 s × 30 fps = 45-frame window of 21 joints. Create ML hand pose and action classifiers are optimized for the ANE — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)

### Inferences
- **Normalized pinch.** `d = |thumbTip − indexTip| / handScale`, where `handScale = |wrist − middleMCP|` (or the mean palm width from index MCP to pinky MCP). Starting thresholds (inferences to tune): pinch on at `d < 0.25–0.35`, off at `d > 0.45–0.55`. That gap of about 0.15–0.2 is the hysteresis band. Require 2–3 frames (about 33–100 ms at 30–60 fps) to enter and 2–3 frames to exit.
- **Click vs drag.** Pinch down then release within about 300 ms, with movement under a small radius, counts as a click. A pinch held longer than that, or moved beyond the radius, becomes a drag (mouse down, move, mouse up on release).
- **Right-click.** Thumb-to-middle-finger pinch, or a pinch held about 600–800 ms without moving.
- **Scroll.** With index and middle fingers extended together (or a two-finger pinch), map vertical hand velocity to scroll deltas. Add a dead zone and momentum.
- **Two-hand zoom.** When both hands pinch, zoom by the ratio of current to initial distance between the pinch points. Use `chirality` to keep track of which hand is which.
- **Dwell click** (an accessibility option). Click when the cursor stays within about 1–2% of the screen for about 0.8–1.5 s, and show a progress ring.
- **Midas touch.** Require an "engaged" state: the hand raised into the active region, palm facing the camera, confidence above a threshold. Ignore gestures for about 200–300 ms after the hand is acquired, and add a gesture to pause or resume tracking.
- **Palm orientation and z.** Vision has no z. The palm normal can be estimated in 2D from the cross product of the wrist→index-MCP and wrist→pinky-MCP vectors (with sign from chirality). The change in apparent hand size (`handScale`) is a cheap proxy for moving toward or away from the camera, which could support "push to tap". MediaPipe's relative z or world landmarks give this more directly.

### Gaps
- No sourced published values for Ultraleap's activate and deactivate distances (the page fetch was blocked), and no peer-reviewed optimal pinch thresholds for webcam tracking.

## 6. Robustness: lost hands, confidence gating, multiple people, clutter, lighting

### Takeaway
Gate on per-point confidence (Apple uses 0.2–0.3), hold the last good state through brief dropouts, and lock onto one hand by position and chirality. If the hand is lost for more than about 300–500 ms, release any held button so no drag gets stuck.

### Cited Findings
- Per-point confidence gating: > 0.3 in WWDC20, > 0.2 in WWDC21. High-level actions are gated on > 0.9 confidence — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/); [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- When more hands are visible than `maximumHandCount` allows, Vision returns "the most prominent and central" ones. `VNTrackObjectRequest` keeps hand identity through temporary occlusion and off-screen movement — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/); [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)
- Lighting: avoid extremes. Gloves degrade results. Distance under about 3.5 m works best — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)
- False positives: feet detected as hands — [WWDC20 10653](https://developer.apple.com/videos/play/wwdc2020/10653/)

### Inferences
- **Lost-hand policy.** Coast for 1–3 missing frames by holding the last position with velocity decay. After about 300 ms of loss, end any drag (mouse up) and hide or dim the overlay. On reacquisition, reset the 1€ filter state so the cursor does not swoop across the screen, and do not allow a click for about 200 ms.
- **Multiple people.** Keep the hand whose wrist is closest to the last tracked position and whose chirality matches. Reject hands whose size is far below the typical size at desk distance (likely a person in the background).
- **Plausibility checks.** Reject frames where bone lengths change by more than about 30% between frames, or where the hand jumps more than about 25% of the frame in one frame. Both indicate "very wrong" joint outputs.

### Gaps
- No sourced quantitative low-light performance figures for either engine.

## 7. Performance: 30–60 fps, CPU and thermals on a fanless MacBook Air, battery

### Takeaway
Limit the work to one hand, a capture size of about 720p, no queued frames and no per-frame allocations. Measure with Instruments, because no published M-series power or thermal figures exist for hand tracking.

### Cited Findings
- A high `maximumHandCount` is a common cause of low fps in camera feeds. Time `handler.perform([request])` to profile — [Axiom Vision diagnostics (third-party skill doc)](https://openskillindex.com/skills/comeonoliver-skillshub-axiom-vision-diag)
- Lowering input resolution or parallelizing Vision requests did not raise body-pose throughput on iPhone 11 (about 40 ms per request) — [Forums 679909](https://developer.apple.com/forums/thread/679909)
- Pixel 6 reference: about 12 ms (GPU) or 17 ms (CPU) per MediaPipe hand frame — [MediaPipe guide (search summary)](https://ai.google.dev/mediapipe/solutions/vision/hand_landmarker)
- Running classifiers at intervals rather than on every frame reduces load and jitter. In Apple's demo, a hand pose and a hand action classifier ran concurrently — [WWDC21 10039](https://developer.apple.com/videos/play/wwdc2021/10039/)

### Inferences
- The fanless Air throttles only under sustained heavy load. One hand-pose inference per frame at 30–60 fps should be a light load, but this must be verified.
- **Adaptive rate.** Run at 60 fps while the hand is active. Drop to about 10–15 fps (or detection only) when no hand has been seen for a few seconds, and stop the capture session when tracking is paused. This is likely the biggest battery win.
- Keep processing on a single serial queue and drop late frames. Avoid converting to `CGImage` or `UIImage`; hand the `CVPixelBuffer` straight to Vision. Pass the correct `orientation` instead of rotating pixels.
- In Python with MediaPipe, OpenCV capture plus GIL overhead adds latency. Use `LIVE_STREAM` mode with monotonically increasing timestamps (in ms), or move to a native Swift pipeline.

### Gaps
- No measured CPU, GPU or ANE utilization, power draw or thermal data for Vision or MediaPipe hand tracking on M-series Macs.
