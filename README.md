<picture>
  <source media="(prefers-color-scheme: dark)" srcset="design/kinesis-links-dark.svg">
  <img src="design/kinesis-links.svg" width="80" alt="kinesis">
</picture>

# kinesis

use your meta neural band to control your mac




https://github.com/user-attachments/assets/43ed6094-5c66-4992-aacd-e010dae3f4fa





swipe between desktops, open mission control, control your music, and pinch + turn for volume or brightness. pick your own mappings, practice in setup, then keep it in the menu bar.

this extended build adds custom keyboard shortcuts, configurable Dictation, a saved ChatGPT/dot conversation destination, a separate **Send in ChatGPT** action, connection-stall recovery, and fixes for air-mouse gesture switching. [usage and configuration](docs/usage.md).

the Axo edition uses Axo Neurotech's teal, sage and snow palette, original Axo mark, and matching light/dark controls and sensor highlights. the app keeps its Kinesis name and menu-bar mark. branding is bundled locally; the interface does not load assets from the web.

built on some tinkering i did with astra in [neural-band-poc](https://github.com/callbacked/neural-band-poc) (feel free to go through that repo to do your own thing), so naturally i wanted to harness it to make it do something useful

some things may act quirky or not work at all, if that is the case i would love to know why, but this is all highly experimental

## run it
needs macos 14+ and a neural band. the packaged local build is for Apple Silicon; source can also be built for Intel. preserve an existing working pairing when updating the app. resetting or forgetting a band changes its pairing and is not an update step.

**hackathon download:** [Kinesis 0.5.0 build 6.22 for Apple Silicon](https://github.com/Axo-Neurotech/Kinesis-Axo/releases/download/v0.5.0-axo.6.22/Kinesis-0.5.0-build6.22-arm64-local.zip). unzip it, move Kinesis.app into Applications, and open it; developer tools are not required. read the [setup guide](https://github.com/Axo-Neurotech/Kinesis-Axo/releases/download/v0.5.0-axo.6.22/START-HERE.md) for permissions, Meta sign-in, band pairing, EMG, and handwriting. [release details and checksums](https://github.com/Axo-Neurotech/Kinesis-Axo/releases/tag/v0.5.0-axo.6.22).

this experimental build is ad-hoc signed and not Apple-notarized; macOS may require manual approval on first launch. this repository is private, so its release downloads require repository access.

local app and source archives are produced with the commands below. the [original upstream releases](https://github.com/callbacked/kinesis/releases/latest) are maintained by callbacked.

set your band in pairing mode, hit pair band, and sign in with meta once. allow bluetooth + accessibility when prompted, and accept the bluetooth pairing request from macos. enable controls and you're in.

## why the meta sign-in

the band only works with an owner that meta has enrolled. the meta ai app does that enrollment when you set up a band, and kinesis does the same enrollment for your mac. it opens meta's own sign-in page once and keeps the session token in your keychain. it only gets that token, not your password. once the band is enrolled, connecting doesn't need meta. forget this band clears the key and the sign-in.

## build it

swift 6 and macOS developer tools. for a local build on this Mac, Command Line Tools are sufficient:

```sh
./scripts/build.sh --local
```

this builds for the current Mac and signs locally without requesting a signing identity. it writes `dist/Kinesis.app` and does not replace the installed app. macOS may require fresh Bluetooth, Accessibility or Keychain approval for a locally rebuilt app. the dot conversation link, gesture assignments, band pairing and account session are runtime settings and are not included in the source or app archives.

for a universal Apple Silicon + Intel build, use full Xcode with Swift 6:

```sh
git clone https://github.com/Axo-Neurotech/Kinesis-Axo.git
cd Kinesis-Axo
./scripts/build.sh
open dist/Kinesis.app
```

quit a copy running from `dist/` before rebuilding it. add `--dev` to either build command to include developer tools.

## experimental iPhone controller

The [iPhone controller work](docs/iphone-controller.md) starts with a physical ESP32 Bluetooth keyboard/mouse adapter for scrolling and configurable shortcut triggers. Build its firmware with `./scripts/build-phone-bridge.sh`; this compiles without writing to a board. The [adapter setup](firmware/axo-phone-bridge/README.md) describes pairing and deliberate single-action phone tests. Shared Swift command handling and a Mac developer diagnostic are included; the native iPhone companion and live band-to-phone operation still require device validation and full Xcode.

## package a local build

```sh
./scripts/package-local.sh
```

this packages the already-built app and an allowlisted source archive under `dist/release-local/`, with release information and verified SHA-256 checksums. it does not rebuild, re-sign, read account settings, or change the installed app. generated archives stay outside the source commit; attach them as GitHub Release assets when publishing. the local app is ad-hoc signed and not notarized. public notarized distribution requires your own Developer ID signing and Apple notarization; see [release instructions](docs/usage.md#release-files).

## license

source code: [mit](LICENSE), retaining the upstream copyright notice. bundled assets retain their own notices in `Sources/Kinesis/Resources/`: the hand mesh has an MIT notice; the Meta/Gizmodo band image and Axo brand marks are separate from Kinesis's MIT license.
