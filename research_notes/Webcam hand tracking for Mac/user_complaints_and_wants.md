# User complaints and wishes: webcam hand-gesture cursor control, touchless and mid-air interfaces, air-drawing apps

> **Method and source caveats (read first).** This session's network blocked direct access to reddit.com, news.ycombinator.com (and the HN Algolia API), producthunt.com and technologyreview.com. The GitHub issue API also needs repo-level access. As a result:
> - **Reddit, HN and Product Hunt comment threads could not be read directly.** Where they show up below, the evidence comes from search-engine snippets, and it is labelled that way.
> - GitHub issue lists *were* readable through the web pages, but only the titles were visible, not the bodies.
> - Several Leap Motion-era press quotes come from search-result snippets, not full articles. Treat their wording as approximate.
> - "Frequency" rankings are a judgement drawn from how often a theme appeared across the sources I reached. They are not a count of a representative sample.

## What do users complain about?

### Takeaway
For DIY and open-source webcam virtual-mouse projects, the most common complaint by far is that **the thing will not install or run**: mediapipe and OpenCV version errors, and "no output" after launch. Next come complaints about how it feels to use: jitter and low precision, false or accidental triggers (the Midas touch problem), arm fatigue, sensitivity to lighting and camera angle, missing actions (right-click, scroll, drag), and macOS permission and camera-FPS problems. Leap-era reviewers had already named fatigue, unreliable tracking and weak use cases as the reasons the category failed.

### Cited Findings

**Rank 1: Installation, dependency and "won't run" failures (GitHub issues dominate)**
- On the most-starred webcam virtual-mouse repo (Viral-Doshi/Gesture-Controlled-Virtual-Mouse, about 883 stars), most visible issues are about install or launch. Titles include "error installing mediapipe…", "OPENCV error, what is correct version of opencv we need to use?", "AttributeError: partially initialized module 'cv2'", "No module named 'screen_brightness_control'", "speech_recognition module is not installing", and "i ran the proton.py code but the application never opened". At least six separate "No output" issues (#19, #25, #28, #29, #41, #44) were filed, including "Proton not responding". — [GitHub issues: Gesture-Controlled-Virtual-Mouse](https://github.com/Viral-Doshi/Gesture-Controlled-Virtual-Mouse/issues?q=is%3Aissue)
- NonMouse (about 203 stars, supports macOS) shows the same pattern: "mediapipe dependency preventing installation using pip" (#28), "pip install cannot find mediapipe==0.810.1" (#27), "error: PyObjC requires macOS to build" (#15), a PyObjC dependency error (#12), "I cannot run newest version on win 11" (#24), and a ZeroDivisionError crash (#9). — [GitHub issues: NonMouse](https://github.com/takeyamayuki/NonMouse/issues?q=is%3Aissue)

**Rank 2: Jitter, instability and low precision**
- An academic paper on a webcam gesture system reports **59% accuracy for cursor control, against 99% for a traditional mouse**. (This figure comes from a search-result summary of a paper on academia.edu. I could not verify which paper it was.) — [Search summary; e.g. Mouse Control Using Hand Gestures (academia.edu)](https://www.academia.edu/86561308/Mouse_Control_Using_Hand_Gestures)
- Open-source READMEs present smoothing as a forced trade-off: "higher smoothing values give a smoother but slower cursor, while lower values are snappier but jittery". Papers recommend Kalman filtering for "instability and jitter" inherent in hand-based control. — [labellerr gesture controller blog](https://www.labellerr.com/blog/ai-powered-hand-gesture-controller/); [Virtual Mouse Operation Using Webcam (SciTePress 2025)](https://scitepress.org/publishedPapers/2025/139155/pdf/index.html)
- In a 2025-26 macOS webcam pointer project, the author admits: "Precision is below a trackpad's; very small targets are harder to hit." — [Bhavyashah20/gesture-control README](https://github.com/Bhavyashah20/gesture-control)
- NonMouse issue #13 (translated): tilting the hand or camera makes cursor movement stop matching fingertip movement. #7: "Sometimes the orientation of the hand does not match the orientation of the image." — [NonMouse issues](https://github.com/takeyamayuki/NonMouse/issues?q=is%3Aissue)
- *(Leap era, 2013)* MIT Technology Review reported that Leap Motion's forums were full of complaints like "control is spastic to say the least". (Quote taken from a search snippet; the full article was not reachable.) — [MIT Technology Review, "Leap Motion's struggles reveal problems with 3-D interfaces" (2013)](https://www.technologyreview.com/2013/08/29/176613/leap-motions-struggles-reveal-problems-with-3-d-interfaces/amp/)
- *(Leap era)* A Tom's Hardware VR hands-on reported the device "losing [the reviewer's] left arm constantly". (Search snippet.) — [Tom's Hardware Leap Motion VR hands-on](https://www.tomshardware.com/uk/news/leap-motion-vr-hands-on,29438.html)

**Rank 3: False or accidental activation (the Midas touch problem) and the need for a clutch**
- HCI literature names the Midas touch problem "a well-known problem for gesture interaction, where normal body movements are mistakenly treated as input to a system". — [Rhythmic Micro-Gestures, Univ. of Glasgow](http://eprints.gla.ac.uk/147499/8/147499.pdf)
- The gesture-control README treats false triggers as the **main risk**. Its acceptance test requires recordings of ordinary hand movement near the camera to produce "zero intents of any kind". It also adds an arming gesture plus a "clutch" so that the system "stays silent when you are not addressing it". — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- GestuReMote (macOS) uses a closed fist to enter and exit control mode, which is an explicit clutch. — [samvoisin/gesture-control](https://github.com/samvoisin/gesture-control)
- A Product Hunt commenter on Pawvis (a local, open-source macOS camera gesture app) asked about **accidental triggers during video calls**. (From a search-engine summary; the PH page itself was unreachable.) — [Pawvis on Product Hunt](https://www.producthunt.com/products/pawvis)
- The gesture-control README also reports gesture-shape collisions: scrolling and double-click share a finger shape, so a "tucked thumb" is needed to tell them apart. — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)

**Rank 4: Gorilla-arm fatigue**
- *(Leap era)* Alphr wrote that even with perfect hardware, "the sheer arm-aching awkwardness of navigating your PC by gesture" would still kill the idea. (Search snippet.) — [Alphr Leap Motion Controller review](https://www.alphr.com/peripherals/32310/leap-motion-controller-review/)
- *(Leap era)* Leap Motion's own ergonomics blog conceded that new users "often get so excited that they elevate their entire arm in non-ergonomic ways". — [Leap Motion blog, "Taking motion control ergonomics beyond Minority Report"](https://www.leapmotiontechnology.com/blog-sub/taking-motion-control-ergonomics-beyond-minority-report)
- Modern macOS webcam project: "Sustained use is tiring; the clutch lets your hand rest between movements." — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- Academic HCI has a dedicated metric for this, Consumed Endurance (Hincapié-Ramos et al., 2014), for "the degree of arm fatigue during mid-air interactions". Clutching at 1:1 mapping is noted to add fatigue. — [ACM UIST'24 Eye-Hand Movement paper citing CE](https://dl.acm.org/doi/fullHtml/10.1145/3654777.3676446)

**Rank 5: Lighting, background and camera conditions**
- "Detection degrades in dim or strongly backlit rooms." — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- NonMouse #11 (translated): camera FPS drops to about **5 Hz on macOS Monterey**. #20: "USB cameras not working?" — [NonMouse issues](https://github.com/takeyamayuki/NonMouse/issues?q=is%3Aissue)
- Projects confine gestures to an inner "active zone" and ignore the frame edges, which is a workaround for hands leaving the frame or edge-of-frame tracking loss. — [nexustech101/aircontrol (via ecosyste.ms)](https://awesome.ecosyste.ms/projects/github.com%2Fnexustech101%2Faircontrol)

**Rank 6: Missing actions (right-click, scroll, drag, text, zoom) and broken gestures**
- Viral-Doshi issues: "Scrolling control not working" (#23); "after proton getting open it is not recognising command" (#22); "Cursor is invisible" (#34). — [GitHub issues](https://github.com/Viral-Doshi/Gesture-Controlled-Virtual-Mouse/issues?q=is%3Aissue)
- The gesture-control README lists "No right-click, text entry, or zoom", "Primary display only", and a scroll direction fixed to natural that ignores the system preference. It dropped timing-based double-click, displacement scrolling and pinch-to-move "after measurements showed they failed", and redesigned scrolling as a joystick, not a swipe. — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- Stuck-button hazard: "A hard crash mid-drag can leave macOS with the button held". This needed a watchdog that releases the button when the hand disappears, plus an Esc failsafe. — [same](https://github.com/Bhavyashah20/gesture-control)

**Rank 7: macOS permission and OS-integration hassles**
- Camera and Accessibility permissions are both required. "macOS caches the decision at launch", so granting permission while the app is running requires a restart. Switching Spaces can trigger an extra prompt to control System Events. Synthetic events are ignored in places: "Mission Control ignores synthetic Ctrl+arrow", so the author fell back to AppleScript. — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- Newer Mac apps (Conductor, Gstrl) require **macOS 14+** because they use Apple Vision hand tracking. That cuts out older Macs. — [dev-talix/conductor](https://github.com/dev-talix/conductor); [Gstrl](https://tomyang-tz.github.io/Gstrl/)
- Platform gaps: requests for Linux/Ubuntu/Debian support (#21, #33); "PyObjC requires macOS to build" when installing on other OSes. — [Viral-Doshi issues](https://github.com/Viral-Doshi/Gesture-Controlled-Virtual-Mouse/issues?q=is%3Aissue); [NonMouse issues](https://github.com/takeyamayuki/NonMouse/issues?q=is%3Aissue)

**Rank 8: "Gimmick" and use-case doubt**
- *(Leap era)* Notebookcheck said the promo videos showed "overly optimistic" scenarios, and that the device "can't be used for complex movement patterns yet". TechCrunch (2013) ran "3D Gesture Control Is An Area Of Focus On Innovation We Likely Don't Need Or Want". (Snippets.) — [Notebookcheck Leap review](https://www.notebookcheck.net/Review-Leap-Motion-Motion-Control-Technology.98821.0.html); [TechCrunch 2013](https://techcrunch.com/2013/08/18/3d-gesture-control-is-an-area-of-focus-on-innovation-we-likely-dont-need-or-want/amp/)
- Computerworld's Mike Elgan argued that in-air gestures failed because they were applied to "the unnatural act of controlling on-screen action". — [Computerworld](https://www.computerworld.com/article/1373748/why-in-air-gestures-failed-and-why-theyll-soon-win.html)

### Inferences
- For a polished, packaged Mac app, "it doesn't install" is already solved, and that alone sets it apart from the DIY Python projects that make up most of what people search for.
- The design problems that matter most for retention are, in order: (a) false triggers, (b) jitter versus lag tuning, (c) fatigue, and (d) gaps in actions such as drag, scroll and right-click. The most mature recent project's main architectural choices (an arming gesture or clutch, a deadzone, a watchdog) are direct answers to (a), (b) and (c).
- CPU, battery and heat, and privacy over an always-on camera, were on the question list, but I found **no direct user complaints** about them in the sources I reached (see Gaps). They may still matter; the evidence simply was not reachable.

### Gaps
- I could not reach Reddit (r/macapps, r/computervision, r/accessibility), HN comment threads, Product Hunt comments or App Store reviews from this session, because the network blocked those hosts. Verbatim end-user quotes about CPU, heat, battery drain and camera-privacy unease are therefore missing.
- I only had GitHub issue titles, not the bodies, so I cannot quote the details of issues.
- I found no air-canvas-specific complaint data. infoaryan/Air-Canvas-project has only one open issue, and its title was not retrieved.

## What happened with Leap Motion, Kinect, Myo, Google Soli and Apple Vision Pro, and what lessons do reviewers draw?

### Takeaway
Each earlier attempt died or was shelved for some mix of four reasons: unreliable tracking, fatigue, too few compelling apps or use cases, and a "gimmick" reputation. Vision Pro's gaze-plus-pinch is the counter-example people cite. It works because the hands rest low and the pinch is only a confirmation (a clutch), not a pointer.

### Cited Findings
- **Leap Motion (2013, classic lesson):** reviewers cited fatigue ("arm-aching awkwardness"), spastic or unreliable control, and over-promising demos (see above). — [Alphr](https://www.alphr.com/peripherals/32310/leap-motion-controller-review/); [MIT TR](https://www.technologyreview.com/2013/08/29/176613/leap-motions-struggles-reveal-problems-with-3-d-interfaces/amp/); [Notebookcheck](https://www.notebookcheck.net/Review-Leap-Motion-Motion-Control-Technology.98821.0.html). A Meta/Oculus forum thread is titled "Why Leap did not succeed / what Oculus should learn from it" (not read in full). — [Meta community forums](https://communityforums.atmeta.com/discussions/dev-pcvr/why-leap-did-not-succeed--what-oculus-should-learn-from-it/254640)
- **Kinect:** Microsoft discontinued the Kinect sensors in fall 2017 and the PC USB adapter in early 2018. One cited reason was that "there were not enough good games". Azure Kinect was discontinued in October 2023. — [ZKM Net Art Extinction Timeline](https://extinction.zkm.de/events/kinect-discontinued/); [Wikipedia: Azure Kinect](https://en.wikipedia.org/wiki/Azure_Kinect)
- **Kinect for disabled users (older study):** gesture operation was slower than a mouse and users made more errors. Users complained about the discomfort of slower operation compared with the mouse they were used to. — [ResearchGate: Gesture controlled human–computer interface for the disabled](https://www.researchgate.net/publication/314030938_Gesture_controlled_human-computer_interface_for_the_disabled)
- **Myo armband:** Thalmic Labs ended sales in October 2018 and dropped software support. Forum posts called the gesture recognition unreliable (anecdotal, low-quality sources). — [Synthiam community](https://synthiam.com/Community/Questions/Myo-Gesture-Armband-Discontinued-4542)
- **Google Soli / Pixel 4 Motion Sense (2019):** BGR called it "a super sophisticated feature… but it's still a gimmick". Android Central said "there isn't any compelling reason to use the feature" and later called it "pretty much a glorified gimmick". Gizmodo AU said it was "still kinda gimmicky — but I'll take a gimmick that actually works". Tom's Guide said the gestures "work well enough" but worried about the short list of use cases. It was dropped from the Pixel 5, which put it on the list of "Dead Google features". — [BGR](https://bgr.com/tech/pixel-4-vs-iphone-11-motion-sense-soli-radar-gestures-explained); [Android Central](https://www.androidcentral.com/google-failed-soli-pixel-4a); [Tom's Guide](https://tomsguide.com/opinion/pixel-4-motion-sense); [TNW](https://thenextweb.com/plugged/2020/05/18/google-pixel-5-motion-sense-soli-radar-chip-analysis/). There was a dissenting view too: Android Central's "I expected to hate the Pixel 4's Motion Sense gestures, but I love them". — [Android Central](https://www.androidcentral.com/i-expected-hate-pixel-4s-motion-sense-gestures-i-love-them)
- **Apple Vision Pro gaze + pinch (2024–):** design analyses note that the system ignores gaze until a pinch is registered, which prevents Midas-touch activation "simply by looking". Users report that eye tracking struggles with small or far targets ("more problems with eye tracking when I'm sitting down and looking up at a big window that's small or far away"). Taps "didn't always register as expected" when typing. A MacRumors forum view holds that "gesture fatigue shouldn't be an issue" because the hands rest in the lap. — [Gaze+Pinch design principles (Medium/Lancaster)](https://medium.com/antaeus-ar/design-principles-issues-for-gaze-and-pinch-interaction-a95e251169ae); [Hans Gerwitz, The visionOS Experience](https://hans.gerwitz.com/2024/04/16/visionos-ux.html); [MacRumors forum](https://forums.macrumors.com/threads/these-gestures-are-how-you-control-apple-vision-pro.2392259/page-4)

### Inferences
- The lesson that keeps coming back: **pointing with a raised arm fails; small confirmations with the hands at rest succeed.** For a webcam Mac app, this argues for (a) relative or "clutch" pointer movement with small hand motions near the desk, (b) a pinch as click, and (c) a clear engage/disengage state.
- The other recurring killer was the lack of a "why". Products that tried to replace the mouse for general use failed, while narrow, situational wins (media skip, presentations, hands-busy moments) are where reviewers came around.

### Gaps
- I did not reach The Verge or Ars Technica reviews of Vision Pro gestures or Leap Motion. I also have no Ultraleap-on-Mac specifics (for example the dropped macOS support in Gemini/Hyperion), and that should be confirmed separately.

## What do people actually want (use cases, gestures, precision)?

### Takeaway
The evidence points to use where the hands are busy or for short bursts, not to replacing the mouse. That means presentations and media control, accessibility for people with motor impairments (with customizable sensitivity), and drawing or annotation. On gestures, users want pinch to click or drag, scroll, right-click, zoom and Spaces switching, plus a reliable on/off clutch, multi-display support, and on-device, local processing.

### Cited Findings
- **Presentations and media:** Zesture (Show HN) was positioned for media and presentation control rather than general mouse use. — [Show HN: Zesture](https://news.ycombinator.com/item?id=24098874)
- **Full mouse replacement, with pinch semantics:** Conductor (macOS 14+, on-device Apple Vision) turns "pinches into clicks, drags, scrolls, and zooms". Gstrl adds swipes, speech-to-text and an AI-agent hookup. Pawvis lets users **train custom gestures** and runs locally. — [Conductor](https://github.com/dev-talix/conductor); [Gstrl](https://tomyang-tz.github.io/Gstrl/); [Pawvis on Product Hunt](https://www.producthunt.com/products/pawvis)
- **System controls:** volume and brightness gestures are common features, and users request them (Viral-Doshi #26 "change in brightness"). Gesturix covers cursor, clicks, scrolling, volume and brightness. — [Viral-Doshi issues](https://github.com/Viral-Doshi/Gesture-Controlled-Virtual-Mouse/issues?q=is%3Aissue); [Gesturix](https://awesome.ecosyste.ms/projects/github.com%2Fanshikarawat14%2Fgesturix)
- **Configuration:** "How to set screen size?" (NonMouse #16). This shows users want calibration of the mapping between camera area and screen. — [NonMouse issues](https://github.com/takeyamayuki/NonMouse/issues?q=is%3Aissue)
- **Accessibility:** Google's Chromebook "Face control" builds on Project Gameface, which was inspired by gamer Lance Carr, who has muscular dystrophy, as an alternative input for people with motor impairments. In a hands-free game-controller study with 8 severely motor-impaired participants, users rated the system easy to learn and said **"sensitivity settings may vary across users and thus need to be customizable"**. The authors note that people who cannot control the required muscles are excluded, so input must adapt to each person's abilities. — [Neowin](https://www.neowin.net/news/google-rolls-out-face-control-for-chromebook-for-a-hands-free-experience/); [Frontiers in Computer Science 2021](https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2021.751455/full)
- **Precision expectation:** even builders concede trackpad-level precision is out of reach ("very small targets are harder to hit"). Users therefore implicitly need mitigations such as target snapping, slow-down near targets, or a precision mode. — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- **Trust and safety features users need:** a preview, dry-run and recording mode so users can see what the system "thinks" before it acts, plus an Esc failsafe. — [Bhavyashah20/gesture-control](https://github.com/Bhavyashah20/gesture-control)
- **Media control use case validated by the Soli era:** reviewers who liked Motion Sense liked it for skipping songs and silencing alarms, which are narrow, hands-free actions. Critics' main point was "the short list of use cases". — [Tom's Guide](https://tomsguide.com/opinion/pixel-4-motion-sense); [Android Central](https://www.androidcentral.com/i-expected-hate-pixel-4s-motion-sense-gestures-i-love-them)

### Inferences
- A credible Mac product should lead with situational modes (Presentation, Media, Draw/Annotate, Hands-dirty/Kitchen, Accessibility) rather than "replace your mouse". It should ship pinch-click/drag, scroll, right-click and zoom from day one, and offer per-user sensitivity, deadzone and smoothing sliders.
- "Runs fully on-device" is a feature that recent Mac apps (Conductor, Pawvis) promote. That suggests privacy is a buying criterion, even though I could not capture direct user quotes on it.

### Gaps
- I found no direct user evidence (Reddit, App Store) for cooking or hands-dirty use, teaching or whiteboarding, or air-drawing demand. These are plausible but unverified here.
- I could not run a YouTube comment sentiment analysis from this session.
- I found no quantitative data on desired precision (for example, the target size users expect to hit).
