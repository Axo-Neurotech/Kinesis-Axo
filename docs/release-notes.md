# local 0.5.0, build 6.22

handwriting settings and configuration reads now share one generic control service. live EMG uses that service as well, rather than opening a separate configuration service. EMG changes cannot interrupt a model operation. model failures name the failed step, and matched service rejections start restoration immediately instead of waiting for a timeout.

54 focused regression tests passed across legacy and modern encryption profiles. on updated band 005M, live EMG configured at 2,048 Hz and delivered samples, and the wearer confirmed correct handwriting text in Kinesis. normal-mode restoration and enabled Mac controls were verified afterward. simultaneous EMG and handwriting under sustained load was not verified.

# local 0.5.0, build 6.20

opens the input channel advertised by the band on each connection, rather than requiring channel 255. after its software update, band 005M advertised channel 129, which the old check rejected before enrollment could begin. the parser accepts only a two-byte little-endian value in the Bluetooth LE dynamic range and retains support for legacy channel 255.

supports the newer band's verified AirShield scheme 26, including its key derivation, authenticated packet prefix, full-block padding and tagged service channels. a peer that offers 27 may reply with 26 or 27 while the app continues to send its verified Enable 26. the peer-27 reply was checked against a live authenticated response from updated band 005M. legacy scheme 3 retains its original packet bytes; mismatched keys, unknown schemes and unauthenticated packets remain rejected.

# local 0.5.0, build 6.17

uses Axo's bundled teal-and-white mark for the macOS app icon.

finds handwriting controls by their exact names instead of assuming the same setting numbers on every firmware. bounded discovery stops for missing, ambiguous or incomplete control descriptions. the band's original typed values are checked before changing its model, and restoration uses the discovered controls and verified readback.

handwriting failures identify the operation that failed. diagnostic logs describe only response field numbers and wire types, without raw payloads, setting values or text.

reads bounded model identifiers from the existing device-info reply. optional or malformed metadata cannot interrupt the normal connection. the new band advertised gesture and wake models but no handwriting model; a band software update must be checked before the alternative handwriting API can be validated.

44 focused handwriting tests passed. on the new band, live discovery found controls 19/20, but reading the model selector returned a different value representation from the supported enum schema. handwriting remains unavailable on that firmware; the app refuses the unsupported operation before changing model settings.

# local 0.5.0, build 6.13

fixes Meta sign-in pop-ups by preserving WebKit's supplied configuration and relationship to the opening page. sign-in windows handle validated return links directly and close when sign-in completes or is cancelled.

rediscovers an unclaimed band before enrollment and after Meta sign-in, so a changed Bluetooth identifier does not leave setup waiting on the old identifier. discovery stays targeted to the remembered band, preserves the saved Meta session, and stops if the target is absent or ambiguous. existing pairing keys are reused when available.

sign-in failures identify the exchange, HTTP status and validated numeric error codes without exposing response bodies, credentials or contact information. the sanitized failure remains visible after closing sign-in. retry starts a fresh session, and cancelled attempts cannot save a session later.

109 focused tests passed, followed by all seven rediscovery tests after the final recovery-message adjustment. live Meta sign-in, new-band enrollment, saved-key reconnect, motion input, gesture reception and Accessibility approval were verified on macOS.

# local 0.5.0, build 6.8

adds Axo Neurotech's original mark and shared brand palette across the sidebar, setup, navigation, controls, hand feedback and sensor plots. light mode uses snow and sage; dark mode uses teal and mint with rose alerts. branding is bundled and static, with no network requests or new animation timers. gesture mappings and connection behavior are unchanged.

# local 0.5.0, build 6.7

adds per-gesture custom keyboard shortcuts, configurable macOS/ChatGPT Dictation, a saved dot-chat destination, and a separate **Send in ChatGPT** action. Dictation and Send have independent shortcuts; Send only acts while ChatGPT is in front and remains an explicit user gesture. shortcuts and chat destinations are local preferences, not bundled account data.

also improves stalled-subscription recovery and air-mouse switching. provisional single taps wait when a double tap or hold has its own assignment, and duplicate raw/classified contact reports cannot immediately undo a completed air-mouse off gesture.

63 focused tests passed with fake connections, input adapters and session stores. live dictation into the selected dot chat was user-verified. the separate Send action has isolated coverage; no automated live message was submitted. the packaged app is the tested Apple Silicon build with a local ad-hoc signature, without Apple notarization.

`scripts/build.sh --local` supports rebuilding on the current Mac with Command Line Tools. `scripts/package-local.sh` archives the built app plus allowlisted sources without rebuilding, re-signing, or including recordings, preferences, pairing material or account sessions.

# kinesis 0.5.0

adds experimental handwriting. write on any surface with your band hand, and kinesis types it into the text field you're in. assign "handwriting on / off" to a gesture to start and stop it. you can find it in dev mode.

also fixes high cpu use while live EMG is on.
