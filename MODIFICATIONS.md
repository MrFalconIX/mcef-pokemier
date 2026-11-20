# MCEF, Pokémier build: modifications

This is a modified version of [MCEF](https://github.com/CinemaMod/mcef) by the CinemaMod Group, version 2.1.6 for Minecraft
1.21.1 (branch `1.21.1`, commit `4cecd7b`). It is distributed under the same license, the GNU Lesser General Public License
version 2.1 (`LICENSE`). This build was made for the Pokémier Azalea mod, by MrFalconIX, starting on 2026-11-20.

Every modified file says so in its header, with the date and what changed.

## Why

MCEF draws its pages with Chromium "without a window" (off-screen rendering). Chromium draws those at 30 frames a second
unless the browser is created with a different `windowless_frame_rate`. MCEF 2.1.6 does not expose that setting: its java-cef
has neither `CefBrowserSettings` nor `setWindowlessFrameRate`. The command-line switch `--off-screen-frame-rate` is ignored.

## What changed

### java-cef (submodule `common/java-cef`)

From CinemaMod's java-cef, branch `6478` (commit `81dba0e`, CEF 126). That branch has `CefBrowserSettings.windowless_frame_rate`
and `CefBrowser.setWindowlessFrameRate`. On a new branch, `pokemier`:

- **`native/CefBrowser_N.h`:** the declaration of `N_CreateBrowser` now matches its definition in `CefBrowser_N.cpp`.
  - It had one argument too many, the `java.awt.Component` that MCEF removed.
  - Because of that, the definition got C++ linkage and was exported under a mangled name.
  - Java could not find it (`UnsatisfiedLinkError`), so CinemaMod's build of `81dba0e` cannot create browsers.
- **`.github/workflows/build-jcef.yml`:** builds for Windows, Linux and macOS (amd64 and arm64) and publishes the archives as
  a GitHub Release whose tag is the commit hash. The original uploaded them to CinemaMod's server.

### MCEF (`common/src/main/java/com/cinemamod/mcef/`)

| File | What changed |
|---|---|
| `MCEFBrowser.java` | The browser is created with `CefBrowserSettings.windowless_frame_rate` from the settings (default 60). A new `setFrameRate(int)` changes it while the browser lives. |
| `MCEFSettings.java` | New `windowless-frame-rate` (1 to 60, default 60). The download address is read from `pokemier-download-mirror` (default: the java-cef fork's GitHub Releases), so a mirror saved by stock MCEF is left alone. |
| `MCEFDownloader.java` | A GitHub Releases address (`.../releases/download`) is used as `<address>/<commit>/<platform>.tar.gz`. |
| `MCEFClient.java` | Adapted to the java-cef of branch 6478: `onAudioStreamPacket` takes `float[]`, and `onFullscreenModeChange` is passed on to the display handlers. |
| `mixins/CefDownloadMixin.java` | A `mcef.libraries.path` given at launch (`-Dmcef.libraries.path=...`) is kept instead of the default folder. Two builds of Chromium can then live side by side. |

`gradle.properties`: version `2.1.6-pokemier.1-1.21.1`, and the mod's name says it is the Pokémier build. The mod id is still
`mcef`, so mods that need MCEF work with it, and it replaces stock MCEF: do not install both.

## Building

1. Push `common/java-cef` (branch `pokemier`) to your fork. Its URL is in `.gitmodules`. The workflow builds and publishes
   Chromium for that commit, which takes about an hour.
2. Check that `MCEFSettings.DEFAULT_MIRROR` points at that fork's releases.
3. Build MCEF:
   ```
   ./gradlew :fabric:build
   ```
   The jar is in `fabric/build/libs/`. Its manifest says which java-cef commit it downloads (`java-cef-commit`).
