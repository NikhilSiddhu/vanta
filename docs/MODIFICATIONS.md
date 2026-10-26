# Modifications vs upstream IINA

Upstream: https://github.com/iina/iina @ `96f605f6d8c4a3ed7d5d0afac644d8681f766757` (2026-10-25)
License: GPLv3 (unchanged)

## 1. Identity / branding

| File | Change |
|---|---|
| `iina/Configs/iina.xcconfig` | `PRODUCT_NAME = IINA` → `Vanta`; `PRODUCT_BUNDLE_IDENTIFIER = com.colliderli.$(TARGET_NAME)` → `com.vanta.player`; entitlements path pinned to `iina/IINA.entitlements` (the `$(PRODUCT_NAME)` form would break once the product was renamed) |
| `iina/iina.xcodeproj/project.pbxproj` | product reference `IINA.app` → `Vanta.app` |
| `iina/Configs/OpenInIINA.xcconfig` | extension bundle id → `com.vanta.player.$(TARGET_NAME)` (Safari App Extension ids must be prefixed by the host app id) |
| `iina/iina/SettingsPageUtilities.swift` | hard-coded `com.colliderli.iina.OpenInIINA` → `com.vanta.player.OpenInIINA` |
| `iina/iina/Base.lproj/Localizable.strings` | 21 user-facing strings, `IINA` → `Vanta` (English only; other 54 localizations untouched) |
| `iina/iina/Base.lproj/MainMenu.xib` | app menu title, `IINA Help` → `Vanta Help` |
| `iina/iina/Base.lproj/AboutWindowController.xib` | window title `IINA` → `Vanta`; credits line replaced with "A dark, minimal player. Built on IINA." |

Deliberately **not** renamed: the `iina://` URL scheme, `IINAPH*` history keys, `com.colliderli.iina`
dispatch-queue labels and/or pasteboard types, and internal Swift symbols. These are plumbing, not UI;
renaming them buys nothing and risks breaking plugin/history compatibility.

## 2. Icon — black

`iina/iina/iina.icon` (Apple **Icon Composer** format, i.e. the Liquid Glass icon system) —

* background gradient: blue-black `#2E2F32 → #090C16` → near-black graphite `#121214 → #000000`
* play-glyph layer: cyan `#6CD8FF → #3B8DFF` → polished silver `#F5F5FA → #B2B2BA`
* accent layers: indigo/violet → graphite, so the icon reads as a single dark mass with one bright edge

Because the `.icon` bundle is what Xcode feeds to `actool`, the icon still gets the native macOS 26/27
Liquid Glass specular highlight and dark/light/tinted variants — it is not a flattened PNG pasted on top.

## 3. Auto-update neutralised

| File | Change |
|---|---|
| `iina/iina/AppDelegate.swift` | `feedURLString(for:)` now returns `nil` instead of the IINA appcast, so Sparkle makes no update checks |
| `iina/iina/Info.plist` | removed `SUFeedURL` + `SUPublicDSAKeyFile` |

Rationale: a rebranded, ad-hoc-signed build must never be silently replaced by an upstream release.
The "Check for Updates" menu item remains but is inert.

## 4. Single audio control (macOS owns the volume)

IINA/mpv has its own software gain, so the player's slider and the Mac's volume keys were two
independent controllers that could disagree.

| File | Change |
|---|---|
| `iina/VolumeSlider.swift` | new `SystemVolume` helper (CoreAudio): `current()`, `set(_:)`, `startMonitoring(_:)` against the default output device's `VirtualMainVolume` |
| `iina/PlayerCore.swift` | `setVolume` now writes the **output device** volume and pins mpv at unity gain (`max(volume, 100)`), so 0–100 % maps 1:1 to the system; >100 % stays available as boost on top. `.softVolume` is stored as 100 so mpv never restores a stale gain |
| `iina/PlayerCore.swift` | `init` registers a CoreAudio property listener so the slider updates live while the volume keys are held |
| `iina/PlayerWindowController.swift` | `updateVolume()` reads `SystemVolume.current()` instead of `player.info.volume` |

Result: one number. Keys, Control Center and the in-player slider are the same control.

### 4b. No in-player volume UI

With the volume owned by macOS, the player's own slider + mute button became a second, redundant
control surface, so they are gone:

| File | Change |
|---|---|
| `iina/MainWindowController.swift` | `oscVolumeView` is created hidden with zero width/height and no slider or mute button attached; all five OSC mount points still reference it, so they now contribute nothing to the layout |
| `iina/MiniPlayerWindowController.swift` | mini player volume popover container hidden for the same reason |

Consequence: adjusting volume is now done with the Mac's volume keys, Control Center, or
**Player ▸ Volume Up/Down** (⌘↑ / ⌘↓), all of which change the same value. Scrolling over where the
slider used to be no longer changes volume — the target view is gone.

**Not possible:** hiding the macOS volume HUD while the player is focused. That HUD is drawn by the
system in response to a hardware key, before the app is involved; suppressing it needs a global event
tap and Accessibility permission, which is more invasive than the problem. Because both controls are
now the same value, the HUD no longer contradicts the slider — and dragging the in-app slider does
**not** raise the HUD at all.

## 5. Not changed

* No network endpoints added or removed — the app can reach exactly the hosts upstream IINA can
  (`iina.io`, `opensubtitles.com`, `assrt.net`, `shooter.cn`, `github.com`, Apple docs URLs used only
  for help links).
* No telemetry, analytics or crash reporting exists in upstream IINA and none was added.
* Playback engine (mpv/FFmpeg dylibs) is the official prebuilt set from `iina.io/dylibs/arm64`.
