# local 0.5.0, build 6.7

adds per-gesture custom keyboard shortcuts, configurable macOS/ChatGPT Dictation, a saved dot-chat destination, and a separate **Send in ChatGPT** action. Dictation and Send have independent shortcuts; Send only acts while ChatGPT is in front and remains an explicit user gesture. shortcuts and chat destinations are local preferences, not bundled account data.

also improves stalled-subscription recovery and air-mouse switching. provisional single taps wait when a double tap or hold has its own assignment, and duplicate raw/classified contact reports cannot immediately undo a completed air-mouse off gesture.

63 focused tests passed with fake connections, input adapters and session stores. live dictation into the selected dot chat was user-verified. the separate Send action has isolated coverage; no automated live message was submitted. the packaged app is the tested Apple Silicon build with a local ad-hoc signature, without Apple notarization.

`scripts/build.sh --local` supports rebuilding on the current Mac with Command Line Tools. `scripts/package-local.sh` archives the built app plus allowlisted sources without rebuilding, re-signing, or including recordings, preferences, pairing material or account sessions.

# kinesis 0.5.0

adds experimental handwriting. write on any surface with your band hand, and kinesis types it into the text field you're in. assign "handwriting on / off" to a gesture to start and stop it. you can find it in dev mode.

also fixes high cpu use while live EMG is on.
