# iPhone scrolling and ChatGPT prompts

The intended first version uses the Neural Band to scroll and open voice input in ChatGPT on an iPhone Air. It does not require handwriting or a newly trained EMG decoder: it consumes the band's existing classified gestures.

The standalone design to validate is:

```text
Neural Band → iPhone companion → Bluetooth command → ESP32 adapter
                                                      ↓
                             iPhone ← Bluetooth mouse/keyboard input
```

The adapter presents a real Bluetooth keyboard and mouse. The companion communicates with a custom service on that same accessory; iOS handles its input service. USB supplies power and programming, rather than phone input. This particular combination of system HID and companion access on the same phone connection must be tested on the device.

The connected board was identified by a read-only ROM query as an original ESP32-D0WD-V3 revision 3.1 with 4 MB flash, using a CP210x USB-to-serial port. It was initially described as an S3, but it requires an original-ESP32 build. Bluetooth HID avoids needing a native USB socket or additional USB wiring. Recheck the actual chip before flashing if the board is swapped.

## First controls

The initial operations are a bounded mouse-wheel step and a momentary keyboard chord. Scrolling sends no pointer movement or button presses. Every keyboard press is followed by an all-keys-released report. Output starts disarmed, and a new connection requires a new armed session. Gesture assignments belong to a separate phone profile; the current Mac assignments are not defaults for the phone. F13 and F14 are available as configurable shortcut triggers, with no predefined action on the phone.

ChatGPT documents an **Open your dot** Shortcuts action with Messaging, Dictation, and Call modes. Dictation is the intended prompt entry point when the action is available on the user's account. The phone must first demonstrate that a physical keyboard binding can launch the saved shortcut; the published Apple Full Keyboard Access guide does not establish that particular Shortcuts binding. Do not substitute the Mac's Command-6 shortcut or assume that rerunning the iPhone action stops dictation. Stopping dictation and sending a reviewed prompt need handset tests before receiving gesture mappings.

## Validation gates

1. Build for the detected ESP32 chip. After reviewing and authorizing replacement of the board's current program, install the firmware and complete authenticated Bluetooth pairing directly on the phone. Enable AssistiveTouch for the mouse path and Full Keyboard Access for keyboard navigation. Use the board's local serial test controls to verify one wheel step and one configured chord. Verify that the phone scrolls, releases all keys, and can open the intended Dictation shortcut before connecting band gestures.
2. Build the native companion using full Xcode and the iOS SDK. Reuse the existing Swift band session and enrollment code, with phone-specific lifecycle, sign-in presentation, and saved identity handling. A fresh phone enrollment can replace the band's Mac ownership, so it must be a deliberate migration after the adapter test rather than an automatic launch action.
3. Verify classified gestures while ChatGPT is in front, then after switching apps, reconnecting, and locking/unlocking. Core Bluetooth background support does not prove that this band's L2CAP stream will continue under those conditions. If it cannot, the standalone design must move band reception to the adapter or another always-running host.

The current Mac has Command Line Tools but no full Xcode/iOS SDK. Shared Swift report generation and input gating passed their focused tests, and the adapter firmware was built and installed after preserving the board's original flash. The adapter starts disarmed. Phone pairing, an iPhone companion build, and end-to-end band control remain unverified; no phone enrollment has been performed.

`PhoneBridgeCheck` is a developer check for the custom command service, built with `swift build --product PhoneBridgeCheck`. Its `--help` path does not initialize Bluetooth. Its `status` command sends no input; `scroll` and `key` deliberately arm and send one input action before disarming. The Mac becomes the Bluetooth input host for those checks. It cannot command a different iPhone while that phone owns the adapter's single connection. Initial phone validation instead uses the board's local serial controls while the phone is the paired Bluetooth host.

## Primary references

- [Apple: iPhone Air specifications](https://www.apple.com/iphone-air/specs/).
- [Apple: external pointer devices and AssistiveTouch](https://support.apple.com/en-us/111775).
- [Apple: Full Keyboard Access](https://support.apple.com/en-ie/guide/iphone/ipha4375873f/26/ios/26).
- [OpenAI: iOS shortcuts for your dot](https://learn.chatgpt.com/docs/dots/channels).
- [Espressif: USB device support](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/usb.html) and [keyboard/mouse example](https://github.com/espressif/esp-idf/tree/master/examples/peripherals/usb/device/tusb_hid).
- [Nordic: nRF52840 features](https://docs.nordicsemi.com/r/bundle/ps_nrf52840/page/keyfeatures_html5.html) and [nRF Desktop reference](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/applications/nrf_desktop/description.html).
- [Apple: Core Bluetooth](https://developer.apple.com/documentation/corebluetooth).
