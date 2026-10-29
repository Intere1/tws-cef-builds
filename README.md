# tws-cef-builds

Patched [Chromium Embedded Framework](https://github.com/chromiumembedded/cef)
binary distributions used by TWS's browser engine. Each GitHub release holds
one CEF minimal distribution, built from an official CEF release branch plus
the patches in [`patches/`](patches).

## Releases

| Release | CEF base | Chromium | Platforms | Patches |
| --- | --- | --- | --- | --- |
| `cef-154.0.8037.17-tws.1` | branch 8037 at `062ebe4` (154.0.23) | 154.0.8037.17 | macosarm64 | [`patches/154.0.8037.17`](patches/154.0.8037.17) |

## Patches

- `0001` adds `AudioOutputTest.AudioStateChangeTest` and
  `AudioOutputTest.AudioStateChangeMutedTest`.
- `0002` fixes the hermetic Xcode path CEF passes to `ibtool`
  (`use_system_xcode=false`).
- `0003` adds the experimental `CefDisplayHandler::OnAudioStateChange(browser, audible)`,
  called when a browser starts or stops producing audible sound
  (`WebContentsObserver::OnAudioStateChanged`).

`0002` and `0003` are proposed upstream; once a CEF release carries them, these
builds are retired.

## Build

Release branch checkout through CEF's `automate-git.py`, patches applied with
`git am`/`git apply`, then:

`build/mac_files/xcode_binaries` points at an Xcode whose only macOS SDK is
26.5 (Chromium 154's SDK; newer SDKs fail to link), and ANGLE runs the Metal
toolchain installed by `xcodebuild -downloadComponent MetalToolchain`.

```sh
GN_DEFINES='is_official_build=true use_system_xcode=false angle_metal_toolchain_dir="<Metal.xctoolchain>/usr/bin"' ./cef_create_projects.sh
autoninja -C out/Release_GN_arm64 cefclient cefsimple ceftests
ceftests --gtest_filter='AudioOutputTest.*:DisplayTest.*'
CEF_ARCHIVE_FORMAT=tar.bz2 python3 tools/make_distrib.py --output-dir=../binary_distrib \
  --ninja-build --arm64-build --minimal --no-docs --no-symbols
```

Each release lists the archive's SHA-1.

## License

CEF is BSD-licensed ([LICENSE.txt](LICENSE.txt)); Chromium's licenses are in
each distribution's `CREDITS.html`.
