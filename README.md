# IYS AutoClicker

A native macOS auto-clicker written in Swift with `CGEvent`. No third-party dependencies.

The app is a menu-bar utility with a small control window. It can click at a fixed position, follow the mouse pointer, or cycle through a list of points. Optional member features add random jitter, text input, and key repeat.

![macOS 13+](https://img.shields.io/badge/macOS-13%2B-blue) ![Swift 5](https://img.shields.io/badge/Swift-5-orange) ![License: MIT](https://img.shields.io/badge/License-MIT-green)

## Features

| Feature | Description |
|---|---|
| Interval | Milliseconds between clicks, minimum 1 ms |
| Mouse button | Left, right, or middle |
| Double-click | Sends a down/up pair on every trigger |
| Follow mouse pointer | Clicks wherever the pointer is |
| Fixed position | Capture the target with a 3-second countdown, then click that coordinate |
| Multi-point loop | Add points, then cycle through them in order. Up to 5 points free, 10 with membership. Saved automatically. |
| Position jitter (member) | Random offset within a 0–50 px radius. On by default at 3 px; set to 0 for a fixed position. Works in every mode. |
| Text input (member) | Repeat a string at each interval, including Chinese. Optionally press Return after each round. |
| Key repeat (member) | Record a sequence of keys (e.g. arrow keys) and cycle through them. Can hold a mouse button at the same time. |
| Membership | Signed license codes unlock the member features. Basic mouse clicking works without activation. |

## Requirements

- macOS 13 or later
- Xcode Command Line Tools (`xcode-select --install`)

## Build

```bash
./build.sh
```

This produces `AutoClicker.app`, a universal binary for Apple Silicon and Intel Macs. Drag it to `/Applications` to install.

### Optional: stable code-signing identity

macOS ties the Accessibility permission to the app's code-signing identity. If you rebuild the app frequently, create a local self-signed identity so the permission survives rebuilds:

```bash
./setup-signing.sh
```

The script creates a dedicated keychain and a self-signed certificate named `AutoClicker Local Signing`. It only affects the current Mac and is not suitable for distribution.

## Accessibility permission

macOS does not allow an app to synthesize mouse clicks or key presses without permission. On first use, the app asks for it:

**System Settings → Privacy & Security → Accessibility → enable AutoClicker**

Restart the app after granting permission. The orange banner at the bottom of the window disappears once permission is in place.

## Usage

1. Choose the interval and mouse button.
2. Choose a click position: follow the pointer, capture a fixed position, or add multiple points.
3. Press **Start**. For text input and key repeat, a 3-second countdown lets you switch to the target window.

### Keyboard shortcuts

| Shortcut | Action |
|---|---|
| **⌃⌥⌘C** | Start / stop from any app |
| **⌃⌥⌘A** | Show / hide the main window |

To change them, edit the constants at the top of `Sources/main.swift` and rebuild:

```swift
let hotKeyCode: UInt16 = 8          // ⌃⌥⌘C start / stop; 'c' = 8, F6 = 97, F7 = 98
let windowHotKeyCode: UInt16 = 0    // ⌃⌥⌘A show / hide; 'a' = 0
let hotKeyFlags: NSEvent.ModifierFlags = [.control, .option, .command]
let windowHotKeyFlags: NSEvent.ModifierFlags = [.control, .option, .command]
```

### Menu bar and window behavior

The app keeps running when the window is closed. Use the menu-bar icon or **⌘Q** to quit.

### Language

The UI is available in Simplified Chinese and English. On first launch the app follows the system language: Chinese systems start in Simplified Chinese, and every other system starts in English. Change it in **Settings… (⌘,)**. Restart the app for the change to take full effect.

## Safe mode

Very small intervals can flood the system event queue and make the whole Mac unresponsive. Safe mode is enabled by default and applies these protections:

| Protection | Description |
|---|---|
| Local detection | Shows chip model, core count, and current CPU usage, refreshed every second |
| Recommended interval | Estimates a safe value from chip, core count, load, and double-click mode |
| Use recommended | Sets the interval to the current recommendation in one click |
| Safety floor | Apple Silicon: 5 ms. Intel with 4 cores or fewer: 15 ms. Other Intel: 10 ms |
| Overload auto-stop | Stops when event-send time exceeds 75% of elapsed time for 3 consecutive seconds |
| CPU cap | Limits this process's own CPU usage (default 30%), slowing clicks when exceeded and speeding up after 5 seconds below 60% of the cap |

Disabling safe mode allows a 1 ms interval, but overload auto-stop still applies.

## Project structure

```
Sources/
  main.swift              Click engine, system monitor, and UI
  Localization.swift      UI strings (Chinese source keys + English translations)
  LicenseCore.swift       AC2 license-code format and Ed25519 verification
  LicenseManager.swift    Activation record storage and expiry checks
  IssuerPublicKey.swift   Embedded public key (generated; do not edit)
Tests/
  LicenseTests.swift      Standalone tests, no XCTest dependency
Tools/
  cardgen/main.swift      License-code generator and verifier (not bundled with the app)
AppIcon.icns              App icon
build.sh                  Build the universal app
build-cardgen.sh          Build the code generator
setup-issuer.sh           Generate the Ed25519 signing key pair and embed the public key
setup-signing.sh          Create a local self-signed code-signing identity
test.sh                   Build and run the license tests
```

## Tests

```bash
./test.sh
```

The tests use an ephemeral Ed25519 key pair and do not touch the local signing key.

## Membership and license codes

Member features are unlocked with an `AC2-` license code. Codes are signed with Ed25519; the app embeds only the public key, so it can verify a code but cannot issue one. Activation records are stored locally, mirrored in `UserDefaults`, and protected by an HMAC.

If you run your own build, generate a signing key pair and embed its public key before building:

```bash
./setup-issuer.sh
./build.sh
```

The private key is stored at `~/Library/Application Support/AutoClicker/issuer-ed25519.key` by default. Back it up offline. Losing it means you can no longer issue codes; leaking it means anyone can.

## Known limitations

- Multi-point mode uses one shared interval; points cannot have individual intervals, and there is no recording/playback.
- Point coordinates are absolute screen coordinates. Changing displays, resolution, or scaling may shift them; capture the points again.
- The app consumes a full CPU core at very low intervals and may cause noticeable heat and battery drain.

## Author
- Maxmax67 (Max Qiu)
- Ian-Yanziqing (Ian Yan)

## License

MIT. See [LICENSE](LICENSE).
