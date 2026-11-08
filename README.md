# MacPad+

MacPad+ turns the edges of a Mac trackpad into brightness and volume controls. It is a menu bar app written in Swift. It is open source under the MIT license.

Status: working prototype. Edge sliders, the HUD, haptics, pointer freeze, the typing guard, the swapped-side icon, outside-click dismissal, quick mute, media strip, and the menu bar panel work on this Mac.

## What it does

- Slide the left edge up or down to change display brightness.
- Slide the right edge up or down to change volume.
- Double-tap the top-right corner to mute.
- Swipe the bottom edge to skip tracks. Double-tap it to play or pause.
- See a HUD message for Next, Previous, Play/Pause, Muted, and Unmuted.
- Hold Shift-Option for fine steps.
- See a native-style HUD on the screen with the pointer.
- Feel a haptic tick at each step.
- Keep the pointer still while you slide. It hides during the slide and returns when you lift.
- Click outside the menu panel to close it.
- Keep normal scrolling and clicks. Only a one-finger slide that starts at the edge counts.

The app lives in the menu bar. It has no Dock icon.

## Requirements

- macOS 13 or later.
- A MacBook trackpad or a Magic Trackpad.
- Accessibility permission for media keys and for ignoring gestures while you type.
- Xcode 16 or later to build. The current build was tested with Xcode 27.2 on macOS 27.2.

Intel Macs are not tested. The build is universal, but no Intel test has run.

## Build

```sh
./Scripts/build.sh --run
```

The script builds a release binary, creates `.build/MacPadPlus.app`, and signs it ad hoc. To use a Developer ID, set `CODESIGN_IDENTITY`:

```sh
CODESIGN_IDENTITY="Developer ID Application: Your Name (TEAMID)" ./Scripts/build.sh
```

## Run

```sh
open .build/MacPadPlus.app
```

The first run opens a setup window. It explains the Accessibility permission, then lets you test both edges.

The Feedback settings page has a “Test haptics” button. It fires one tick with the current strength.

## Test

```sh
swift test --disable-sandbox
```

`--disable-sandbox` is needed in this session because the outer sandbox blocks SwiftPM's own manifest sandbox. On a normal machine, `swift test` should work.

The package has 23 unit tests for the gesture state machine, value mapping, and settings.

There is also a runtime self-test:

```sh
.build/MacPadPlus.app/Contents/MacOS/MacPadPlus --self-test
```

It checks brightness, volume, haptics, and that the trackpad source starts. It restores brightness and volume after the check.

## Permissions

macOS does not expose raw trackpad contacts to normal apps. MacPad+ uses a private framework to read them. This is why the app is not on the Mac App Store.

Accessibility is used to:

- read when you type, so a resting palm does not change a level;
- send media keys.

Pointer freeze has a timer fallback and works without Accessibility.

The app does not read which keys you press. It does not look at your screen. It has no analytics and makes no network requests.

## Architecture

```
Sources/MacPadCore/   pure Swift gesture engine, edge zones, value mapping, settings
Sources/MacPadApp/    AppKit shell, SwiftUI views, private framework bridges
Tests/                 unit tests for the core
Scripts/               build, icon, and spike scripts
```

The hot path is event driven. It reads a contact frame, updates a small state machine, writes the system level only when the quantized value changes, and sends at most one HUD update per 1/16 step. The HUD uses Core Animation layers, not SwiftUI layout.

## Known limits

- Private frameworks can change in any macOS update. The app degrades if a symbol is missing, but a future OS may still break it.
- The app is not notarized yet. Ad-hoc builds may show a Gatekeeper warning.
- Per-app profiles are not implemented.
- Keyboard backlight is not implemented.
- External display brightness is not implemented.
- Quick mute and media strip are tested with a real player. The HUD shows that the command was sent. Some players may ignore next or previous.
- Magic Trackpad haptics are not tested.
- Haptics work while MacPad+ is front. Background haptics were not clearly verified.
- Cursor freeze has a 1.2 second watchdog. If the app misses the end of a slide, it releases the pointer and shows it again.
- Rebuilding and reinstalling the ad-hoc app can reset Accessibility. Grant the installed copy again after a reinstall. The typing guard does not work until you do.
- The typing guard waits 0.8 seconds after the last key press. A slide must also move at least 5 percent of the edge while held for at least 60 ms. Taps, press-drumming, and palm drift never reach that gate.
- Palm-heel contacts are rejected at touch-down: narrow and long ellipses (short axis under 6.5 and long axis over 9.2 sensor units), as calibrated on this trackpad.
- Only the main trackpad device is read. The Touch Bar device is ignored, so typing near it cannot change a level.
- The repository URL in `AppInfo.swift` is a placeholder. Change it before publishing.

## Roadmap

1. Test the gesture path on real hardware and fix feel issues.
2. Test media keys, mute, and the menu panel.
3. Add per-app profiles with inheritance.
4. Add a keyboard backlight spike.
5. Add external display support only after hardware testing.
6. Add a notarized release workflow.

## License

MIT. See `LICENSE`.
