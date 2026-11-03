# Axo phone bridge for ESP32

This experimental firmware presents a Bluetooth HID keyboard and relative mouse to an iPhone. A custom service on the same Bluetooth connection accepts bounded navigation commands from a future iPhone app. The firmware does not connect to the Neural Band itself. USB supplies power and the local UART setup console; native USB HID, wiring changes, and a board-specific button are not required.

The initial target is the connected **ESP32-D0WD-V3, 4 MB flash**, using the bundled Bluedroid BLE stack in Arduino-ESP32 **3.3.11**. A remembered board name is insufficient to select a flash target; this target came from the chip's ROM identification. Do not flash this classic-ESP32 build onto an ESP32-S3/C6. No flash/upload script is provided.

```sh
arduino-cli compile --fqbn 'esp32:esp32:esp32:PartitionScheme=no_fs' firmware/axo-phone-bridge
c++ -std=c++17 -Wall -Wextra -Werror firmware/axo-phone-bridge/tests/protocol_test.cpp -o /tmp/axo-phone-protocol-test
/tmp/axo-phone-protocol-test
```

Compile success proves API compatibility, not pairing, successful phone input, or correct board upload settings. Preserve the board's existing flash contents before any separately authorized upload.

## Pair the iPhone and test the hardware

Keep the board attached to the Mac for power. Open its Serial Monitor at 115200 and enter `PAIR` followed by Enter. This explicit local command disconnects the current peer and replaces only this ESP32’s previous Bluetooth pairings and their cached notification subscriptions. It waits until those bond removals have completed, generates a fresh passkey, and opens enrollment for 120 seconds. It does not erase NVS, reset the Neural Band, or change Kinesis’s band pairing. In the iPhone's Bluetooth settings, pair **Axo Phone Bridge** and enter the displayed code in the system dialog. Do not paste that code into chat. Successful authenticated bonding closes the window.

The new passkey and bonds persist across reboot; this firmware never erases NVS or logs identity keys. Input remains unprovisioned until a fresh authenticated pairing in this setup window succeeds and the bridge saves its own provisioning marker. Previous firmware’s Bluetooth bonds cannot authorize the first setup. Normal startup does not print the passkey. Keep the Arduino Core Debug Level at None because the vendor BLE implementation can independently print security diagnostics when debugging is enabled. An unrecognized peer may establish a BLE link, but input remains blocked until encrypted, authenticated, bonded pairing completes; passkey negotiation outside the local enrollment window is rejected. Saved bonds can re-encrypt without displaying a new passkey.

Enable the iPhone's Full Keyboard Access for focus navigation and, if needed for mouse/wheel input, AssistiveTouch. The actual iPhone HID subscriptions and behavior must be checked on the phone; successfully pairing alone does not prove navigation works.

For one local test, enter `ARM` in Serial Monitor, then one of these commands within 30 seconds:

| Command | Single test action |
|---|---|
| `TAB` | Press and release Tab to move keyboard focus |
| `UP` | One upward wheel step |
| `DOWN` | One downward wheel step |
| `F13` | Press and release F13, for an explicitly configured phone shortcut |
| `DISARM` | Stop input immediately |

Each test consumes that local authorization and disarms after completion. The firmware requires notification subscriptions for both reports before declaring output ready; the phone may need its accessibility features enabled. No text, PIN entry, arbitrary keyboard input, or held state is available through this console.

Only one BLE central is admitted. During a phone HID test the phone owns the connection, so a separate Mac BLE sender cannot simultaneously command the bridge. The planned iPhone app must use the custom service alongside iOS's HID connection. Whether that shared access and its background behavior work on the actual iPhone remains a device-test requirement; the source does not establish it.

## BLE protocol, version 1

Service: `A10A0001-5D53-4A18-8F6A-1C173E442001`. Command characteristic: `A10A0002-5D53-4A18-8F6A-1C173E442001`, writes with response requiring encryption and MITM authentication. Status characteristic: `A10A0003-5D53-4A18-8F6A-1C173E442001`, authenticated encrypted read/notify. Reading status can trigger the OS pairing flow. Authentication completion changes the session, so read fresh status afterward before arming. The session nonce is a replay boundary, not an authentication secret; BLE authenticated bonding provides access control.

Every command is exactly 12 bytes: version `1`, opcode, unsigned 16-bit sequence (little endian), unsigned 32-bit session (little endian), two argument bytes, and two zero reserved bytes. Sequence and session must be nonzero. Sequences strictly increase within a session; reconnect before sequence 65535 is exhausted. Retransmitting old commands does not repeat their actions.

| Opcode | Meaning | Arguments |
|---|---|---|
| 1 | Arm | Both zero |
| 2 | Disarm | Both zero |
| 3 | Heartbeat | Both zero |
| 4 | Momentary navigation key | Keyboard/Keypad usage; HID modifier byte |
| 5 | Relative wheel | Signed vertical and horizontal steps, each −127…127; at least one nonzero |

Allowed keyboard usages are Enter `28`, Escape `29`, Tab `2b`, Space `2c`, Home `4a`, Page Up `4b`, End `4d`, Page Down `4e`, arrows `4f`…`52`, and F13/F14 `68`/`69` as explicit user-configured shortcut triggers (hexadecimal). HID modifier bits select the left-hand keys: Control=1, Shift=2, Alt/Option=4, GUI/Command=8. The API accepts no text, letter keys, arbitrary reports, mouse-button holds, or key holds. Space can activate the focused item when assigned that function in Full Keyboard Access; this is not direct touchscreen injection. Scroll direction and navigation depend on iPhone settings.

Status is 12 bytes: version, state (`0` disarmed, `1` armed), last accepted sequence, current session, reason byte, output-ready flag (`0`/`1`), and two zero reserved bytes. Integer byte order matches commands. Protocol acceptance is **not confirmation that the iPhone executed an action**. Reasons are defined in `AxoPhoneProtocol.h`; clients should tolerate future reason values. Output ready means authenticated HID subscriptions, supported report protocol, no HID suspend, and no reported congestion; it is not a promise that a phone action succeeded.

The queue holds at most eight commands and rejects work older than 150 milliseconds. BLE reconnection, authentication changes, HID subscription/readiness changes, HID suspend, malformed input, or queue overflow disarms output and changes the session. A 1.5-second command/heartbeat idle timeout also disarms output. No boundary automatically rearms it. Send heartbeats more frequently than that only while controls are intentionally enabled.

Keyboard press and all-keys-released reports use separate serialized transmissions. The pending release is set before attempting a press and is cleared only after the successful ESP-IDF transmit-completion callback for the sole in-flight report, matching its interface, connection, and characteristic. Actual Bluedroid success callbacks omit the payload despite the header documentation, so payload matching is not used. The callback is a stack/transmit result, **not a GATT peer acknowledgement**: HID uses notifications. Any submission failure, transmit failure, or completion timeout leaves neutral release pending and disconnects the link; the ambiguous slot is never reused on that live connection. A new connection must complete neutral cleanup before another action is allowed. Relative wheel movements are never retried after ambiguous failure. A disconnected phone cannot receive a release; reconnection requires neutral reports and explicit rearming.

`AxoPhoneReportMap.h` supplies standard keyboard (report-reference ID 1, eight-byte payload) and relative mouse (ID 2, five-byte payload) descriptors. BLE payloads do not prepend those IDs. The mouse's button, x, and y fields remain zero; only its two wheel fields vary. The device uses Axo branding and experimental Espressif-based metadata, without an Apple vendor identity.

## Verification boundary

The portable C++ fixture checks command bounds, key allowlisting, integer byte order, and exact status bytes. Before real gestures are enabled, test phone pairing, Tab focus movement, activation, both scroll directions, F13 only after assigning it, disconnect/reconnect, suspend/resume, heartbeat expiry, and interrupted key-output cleanup. The future iPhone application's direct band connection and simultaneous HID/custom-service access remain separate work; this firmware alone is not a finished band-to-phone controller.
