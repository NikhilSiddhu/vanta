# Vanta

A dark, minimal macOS media player — a fork of [IINA](https://github.com/iina/iina) (GPLv3).

Vanta is not a new engine. It is IINA at a pinned upstream commit with a small, deliberate set of
changes: a real rebrand, a black icon built in Apple's Icon Composer format, a single system-owned
volume, and defaults that make the player open as a clean canvas instead of a wall of buttons.

```
Upstream: https://github.com/iina/iina
Commit:   96f605f6d8c4a3ed7d5d0afac644d8681f766757  (2026-10-25)
License:  GNU GPLv3 (inherited, unchanged)
```

## What is different

| Change | Detail |
|---|---|
| Name / identity | `Vanta`, bundle id `com.vanta.player` |
| Icon | near-black graphite body, silver play glyph, native Liquid Glass variants |
| Volume | the output device is the only volume: mpv is pinned at unity gain, so the Mac's keys, Control Center and the player cannot disagree. The in-player volume UI was removed |
| Defaults | one toolbar button instead of four, floating auto-hiding control bar, dark theme, `←`/`→` = ±10 s |
| Auto-update | upstream Sparkle feed disabled — a fork must never be silently replaced by an IINA release |
| Handlers | claims video types only; audio stays with Music, `.m4v` stays with TV |

Full details, file by file, in [`docs/MODIFICATIONS.md`](docs/MODIFICATIONS.md).
Security review of the upstream source in [`docs/AUDIT.md`](docs/AUDIT.md).

## Build

Requires Xcode (built and tested with Xcode 27.1) and the prebuilt mpv/FFmpeg libraries:

```bash
git clone https://github.com/NikhilSiddhu/vanta.git
cd vanta
./other/download_libs.sh --arch arm64 --skip-plugins   # or universal
xcodebuild -project iina.xcodeproj -scheme iina -configuration Release \
  CODE_SIGN_STYLE=Manual CODE_SIGN_IDENTITY="-" DEVELOPMENT_TEAM=""
```

The built app is `Vanta.app`. It is ad-hoc signed; it is a local/personal build, not notarized and not
distributed as a binary.

## Licence and attribution

Vanta is a **modified version** of IINA and is therefore licensed under the **GNU GPLv3** (see
`LICENSE`). If you redistribute it, you must ship the corresponding source — this repository is that
source.

* IINA is © its authors and contributors; see the copyright headers retained in every file.
* The player engine is [mpv](https://mpv.io) and FFmpeg, fetched at build time, not vendored here.
* The application icon is a recolour of IINA's Icon Composer asset, and therefore also GPLv3.
* **Vanta is not affiliated with, endorsed by, or supported by the IINA project.** Please do not report
  Vanta bugs upstream. "IINA" is used here only to describe the origin of the code.
