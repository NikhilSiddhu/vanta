# Vanta — source audit

**Subject:** https://github.com/iina/iina @ `96f605f6d8c4a3ed7d5d0afac644d8681f766757` (2026‑10‑25)
**Method:** full source read of `iina/` (218 Swift files), entitlements, `Info.plist`, every build-phase
script in `iina.xcodeproj`, `other/`, `deps/`; dependency resolution; runtime host enumeration.
**Verdict:** ✅ Safe to build and run locally. No telemetry, no analytics, no hidden networking, no
obfuscation. It behaves like a media player because it is one.

---

## 1. Telemetry / analytics — none

`grep -i` across all Swift sources for `sentry|crashlytics|firebase|amplitude|mixpanel|telemetry|analytics|google-analytics|umami` → **0 matches**.

Nothing phones home about what you watch. Crash logs go only to Apple's local Console (`os.Logger`
subsystems such as `iina`, `video0`, `mpvgl`) — local, not network.

## 2. Network — only what you'd expect

| Host | Why | When it is contacted |
|---|---|---|
| `iina.io` / `www.iina.io` | appcast (auto-update), docs links | **now disabled** — see MODIFICATIONS §3 |
| `api.github.com`, `raw.githubusercontent.com` | plugin store index | only when you open the plugin store |
| `www.opensubtitles.com` | subtitle search | only when you ask for subtitles |
| `api.assrt.net`, `secure.assrt.net`, `www.shooter.cn` | subtitle providers | only when you ask for subtitles |
| `github.com`, `addons.mozilla.org`, `crowdin.com` | extension/help links | only when clicked |
| arbitrary URLs | Open Network Stream, yt-dlp | only when you open a URL |

`URLSession` appears in **1** file (`PluginStorePanel.swift`); `URLRequest`/`URLComponents` in 8
(subtitle clients, plugin store, open-URL dialog). No background beacons, no periodic polling.

Not delivered: no `NSTask`, no `posix_spawn`, no `popen`, no `system()` anywhere in the app's Swift code.

## 3. Entitlements — what they actually mean

`iina/IINA.entitlements`

```xml
com.apple.security.cs.allow-unsigned-executable-memory  → mpv's JIT/scripting
com.apple.security.cs.disable-library-validation        → load mpv/FFmpeg dylibs signed by another team
```

Both are unavoidable for an mpv-based player. Consequence to be aware of: the app is **not sandboxed**,
so it can read any file your user can read. That's normal for VLC/IINA/mpv and is why it can open a
video from anywhere; it also means it is only as trustworthy as its source — which is why we built it
from audited source instead of downloading a binary.

The `OpenInIINA` Safari extension is separate and **is** sandboxed (`files.user-selected.read-only`).

## 4. Build phases — three, all local

1. copy `deps/plugins/` into the bundle (skipped in our build),
2. copy + relink the `.dylib` set,
3. `iina/write_info_plist_header.sh` — writes a C header of build metadata.

No `curl | bash`, no remote code execution at build time.

## 5. Supply chain

| Item | Assessment |
|---|---|
| SPM packages | **Sparkle**, **Just**, **PromiseKit**, **GRMustache.swift** — long-lived, well-known, pinned by version |
| `deps/lib/*.dylib` (mpv + FFmpeg stack) | prebuilt from `https://iina.io/dylibs/arm64`, fetched over HTTPS |
| ↳ integrity | the script itself has **no checksums**, but every dylib is **code-signed by upstream**, so each download is verifiable: `codesign --verify <dylib>`. This check caught 3 truncated downloads during install — they were deleted and re-fetched |
| `deps/executable/youtube-dl` (yt-dlp) | fetched from GitHub **`releases/latest`** — unpinned. Only used for online-video support; irrelevant for local files. Treat as the weakest link in the chain |

## 6. License

GNU **GPLv3**. Personal/local modification is unrestricted. Redistribution requires publishing the
modified source — which is why `the source tree at ` stays on disk and `MODIFICATIONS.md` lists every change.

## 7. Residual risk

1. **Not sandboxed** — by design for a player.
2. **Plugins** are executable JavaScript/extension code. Don't install plugins you don't trust; we
   shipped with the official plugin set disabled.
3. Subtitle search sends the video's filename/hash to the subtitle provider you choose. Skip it for
   anything private.
4. Upstream auto-update is disabled, so **security fixes must be pulled manually** (`git pull` +
   `./build.sh`).
