# Vulcan's Gift — native portrait animation

A reusable animation framework for Total War: THREE KINGDOMS. The Workshop mod provides the Lua player; the companion native DLL adds direct APNG and WebP decoding. Existing PNG-frame animation remains supported.

## Distribution

- **Steam Workshop:** the integrated `Vulcans_Gift.pack`.
- **Native component:** `VulcansGift.dll`, supplied separately or inside the TK Mod Manager language-patch/harness package.
- **Source:** this repository. The native WebP decoder is statically linked, so no additional WebP DLL is required. No standalone launcher or artwork-conversion program is needed.

The DLL is distributed separately through the TK Mod Manager language-patch/harness package. This repository currently publishes source code only.

## Source download

Download [VulcansGift_Source.zip](VulcansGift_Source.zip?raw=true), extract it, and run `build.cmd` using the requirements below. The archive preserves the complete source tree, including Rust, Lua, tests, dependency notices and vendored libwebp sources.

## Install with TK Mod Manager

1. Keep the original official **Script Extender 0.46.0** installed.
2. Open Settings → Script extender → **Import local DLL**, and choose `VulcansGift.dll`.
3. Use local version **`0.46.0-vulcansgift.4`**, select the new entry, and choose **Use this version**.
4. Enable Script extender for the profile, enable Vulcan's Gift, and start the game normally.

The local-DLL import selects one extender version; it is not an additive plugin list. This DLL loads the original installed 0.46.0 core by its full path, waits for readiness, then installs the animation hooks. The original DLL and its configuration remain in place. Core `se_status` / `se_version` queries are forwarded. Do not overwrite the official `0.46.0` folder or the manager's language-patch DLL.

## For mod authors

### Existing PNG sequences

Keep the established convention:

```text
Source: ui/characters/example/large_panel/norm/face.png
Frames: ui/vulcans_gift/animations/characters/example/large_panel/norm/ani/face_000.png
        ui/vulcans_gift/animations/characters/example/large_panel/norm/ani/face_001.png
        ...
```

Frames start at zero with consecutive numbering. Legacy portrait playback remains 24 fps; advisor playback remains 30 fps.

### APNG

Place the animated PNG at the normal illustration `.png` path. Animation is detected from its `acTL` chunk. Preserve the game's expected canvas, transparent margins and `Comment` placement metadata. The APNG itself is decoded directly; no PNG sequence is produced.

### WebP

Place a valid engine-readable PNG and a WebP sibling in the same folder and preferably the same pack:

```text
face.png   # coordinate/cover image, including the expected placement metadata
face.webp  # animation decoded directly by the DLL
```

PNG and WebP dimensions may differ. Animation is fitted into the PNG canvas, centered with its aspect ratio preserved; transparent margins fill the remaining area. Resized frames are cached when RAM permits. A matching WebP takes precedence over APNG for that source; a lower-priority WebP cannot replace a higher-priority PNG override. Arbitrary cropped artwork is not automatically aligned. The PNG is one cover/layout image, not an animation frame sequence.

### UI coverage

The same native path is connected to character details, starting-faction selection (leader and noteworthy-character portraits), and the UI advisor portrait. Text, buttons and campaign map pins are outside the player traversal. Within one portrait part, animated norm/happy/angry alternatives are preferred over static alternatives.

## Performance and supported inputs

Encoded files are read directly from the active packs identified by the running game's actual command line. Excluded packs are respected; the tool does not guess an unrelated saved manager profile. PNG sequence lists are indexed too.

There is no animation-count cap. Composed native frames are prepared in the background and retained in RAM when possible. The initial cache budget is 85% of available physical RAM after a 2 GiB reserve. Other sources use streaming playback. Native frame timings and loop counts are read from the file. Existing atlas descriptors are retained and the source region is bounds-checked before upload.

Current compatibility: Windows x64; verified Steam game PE build `0x69ce4c84 / 0x4836000` (1.7.2.0); TKMM 0.6.8; official extender 0.46.0. A UTF-8 or UTF-16LE mod-list `.txt` argument is required. Currently uncompressed PFH5 image entries only. There is no fixed 64 MiB encoded-file limit. Pack bounds, address-space conversion and allocation failure are checked. Per-image dimension/frame limits remain 4096×4096 pixels and 10,000 frames. Native output is 8-bit/channel RGBA.

## Validation

Development testing observed APNG and WebP in character details, animation in faction selection, and APNG in the advisor. Advisor atlas uploads have runtime log evidence. These observations are not a claim that all nine format/UI combinations were separately benchmarked.

Offline checks cover decoding/composition, restart and timings, cached-frame equality, atlas bounds/overflow, active-pack selection, PNG-to-native handover, UI scope, and legacy 24/30 fps timing. Public native unit tests generate their own fixture and do not require private mod artwork. No general FPS or turn-speed improvement is claimed.

## Build

Use Windows x64 with Visual Studio C++ Build Tools, CMake, and Rust's MSVC toolchain. From an x64 Developer Command Prompt, run:

```bat
build.cmd
```

The script builds vendored libwebp 1.6.0, builds the adapter, and builds `native/`. Output is `VulcansGift.dll` at the repository root. For unit tests after building, set `WEBP_STATIC_DIR` to the absolute `webp_build` directory and run `cargo test --manifest-path native/Cargo.toml`.

Lua source is under `lua/script/`. Put those files in the corresponding paths of the existing Workshop pack. No test portraits or locally generated animation samples are supplied.

## Logs and rollback

Animation log: `VulcansGift.log` in the imported version folder. The official extender keeps its original 0.46.0 log and configuration. The automatically generated request/registry handshake is under `%LOCALAPPDATA%/VulcansGiftNative/` and requires no user editing.

Close the game, select official 0.46.0 again in TKMM, and restore the previous Workshop pack to roll back.

See `THIRD_PARTY_NOTICES.txt` and `licenses/` for dependency notices. The official extender binary and third-party portrait artwork are not included.
