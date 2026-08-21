ISL Translate — Real-Time Sign Language Translator

A working hackathon prototype: translates hand gestures to speech, and speech to sign cues, entirely in the browser. No backend, no build step, no training data to collect in advance.

How it works
Hand tracking — MediaPipe Hands reads the webcam feed and returns 21 3D landmark points per hand, live, in the browser.
Built-in gesture model (works immediately, zero training) — a geometric rule-based classifier looks at which fingers are extended vs curled (comparing fingertip distance from the wrist to the knuckle joint) and matches the pose to a small starter vocabulary:
Hand shape	Recognized as
Open palm, all fingers extended	hello
Closed fist	no
Thumbs up	yes
Pointing (index only)	help
Peace sign (index + middle)	please
Thumb-to-fingers pinch	thank you
Four fingers extended, thumb tucked	how much
Thumb + pinky extended	stop
This means the demo recognizes signs the moment the camera turns on — no setup, no data collection.
Custom gesture training (optional) — landmarks are also normalized (relative to the wrist, scale-invariant) and can be fed into a TensorFlow.js KNN classifier. Add a new gesture name, hold the sign, click "capture sample" 15-30 times, and the classifier starts recognizing it — custom-trained signs take priority over the built-in model once trained.
Sign → speech — once a gesture is classified with reasonable confidence, click "speak translation" to have the browser read it aloud via the Web Speech API (SpeechSynthesisUtterance).
Speech → sign — switch modes to use the microphone. Speech is transcribed live via the Web Speech API (SpeechRecognition) and matched against your vocabulary, showing a text cue card for that word. (Swap in real sign photos/video clips here for a stronger demo — see "Next steps" below.)
Running it

No install needed for the demo itself — it's static HTML/JS pulling libraries from a CDN.

bash
cd isl-translator
python3 -m http.server 8000

Then open http://localhost:8000 in Chrome (best Web Speech API + camera support). Camera and microphone access require either localhost or HTTPS — it will not work opened directly as a file:// URL in most browsers.

If you want to deploy it for judges to try on their own phones, any static host works (GitHub Pages, Netlify, Vercel) — HTTPS is provided automatically by all three.

Using it in your demo

Fastest path — no setup: click Start camera, allow access, and just hold up one of the built-in shapes (open palm, fist, thumbs up, pointing, peace sign, pinch). The right panel shows the live prediction immediately, labeled "(built-in model)". Click Speak translation to hear it read aloud.

To add your own signs on top of that:

Pick "Add new sign" and type a word (e.g. "toilet", "water").
Hold the sign steady in frame and click Capture sample 15-30 times. Vary hand position slightly between captures (distance from camera, slight rotation) so it generalizes.
Once trained, that sign is recognized live and labeled "(custom-trained)" — custom signs take priority over the built-in shapes.
Switch to Speech → sign mode, click Start listening, and speak one of your vocabulary words — a cue card appears.

For judges: lead with the built-in shapes since they work instantly with zero setup, then optionally train one custom sign live to show the system is extensible, not just a fixed demo.

Recent bug fixes
Overlay not drawing on first camera start — the canvas-sizing listener was attached after the camera had already started, so on many browsers loadedmetadata had already fired and the landmark overlay stayed 0×0 (hand tracking worked, but nothing was drawn). Fixed by attaching the listener first and sizing immediately as a fallback.
Resource leak on repeat Start/Stop — clicking Stop never closed the MediaPipe Hands instance, so each Start/Stop cycle stacked another model instance and frame loop in the background. Stop now closes and clears it.
Stale prediction after the hand leaves frame — the last detected gesture (and an enabled "Speak translation" button) used to stay on screen indefinitely once your hand left the camera view. It now resets to "—" immediately.
False-positive word matching in Speech → sign — matching used plain substring search, so saying "I know" or "snow" would wrongly trigger the "no" sign cue. Matching is now whole-word.
Newly added gesture not auto-selected — adding a custom sign left the dropdown on whatever was previously selected, so your first captured samples could get filed under the wrong label. The new sign is now selected automatically.
Misleading "live" badge — the red "live" badge on the video panel was shown even before the camera started. It now only appears while the camera is actually running.
Files
index.html — page structure and styling
app.js — camera pipeline, landmark normalization, KNN training/inference, speech I/O
Known limitations (be upfront about these to judges)
The KNN classifier is a static-pose recognizer — it classifies a held handshape, not a moving gesture. Real ISL includes motion and non-manual markers (facial expression, mouth patterns) that this prototype doesn't capture.
Recognition quality depends entirely on how many/varied samples you capture per sign during setup — more samples and consistent lighting significantly improve accuracy.
SpeechRecognition is a Chrome/Edge feature (WebKit Speech API) — it isn't supported in Firefox. Demo in Chrome.
The "sign cue" side is currently a text label, not an actual ISL video/image — swap in real sign media for a production version (see below).
Next steps if you keep building
Real sign cues: replace the text-label cue cards with short video loops or illustrated hand-pose diagrams for each vocabulary word (a folder of .mp4/.gif per word, referenced by a lookup table, is enough for a stronger demo).
Two-hand and motion gestures: MediaPipe Hands supports maxNumHands: 2 — track sequences of frames (not just a single pose) and classify short motion trajectories for signs that require movement.
Offline support: MediaPipe Hands and TF.js both run fully client-side already — for true offline use, self-host the CDN scripts (hands.js, camera_utils.js, tf.min.js, knn-classifier.min.js) instead of loading from jsdelivr, and cache them via a service worker.
Persisting trained gestures: currently the KNN classifier resets on page reload. Serialize knn.getClassifierDataset() to localStorage or a backend so trained vocabularies persist across sessions.
