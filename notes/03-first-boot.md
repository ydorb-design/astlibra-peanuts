# 03: First boot on the AYN Thor (2026-11-27)

## How it was started

- Test app `dev.nxnative.android.peanutstest` (`-PnxPackEngine=fake`), beside the release app.
- The launcher will not start a pack with no shader library: Play runs Prepare instead, and with the
  fake engine that is a simulated build (its counts, e.g. "22,852 shaders", are placeholders).
- Fix used: `shader_note  no NVN shaders yet ...` in the local profile's `game.tsv`, so
  `pack_android.py` accepts an empty split and builds a valid empty library (`vk_android_build.py`,
  needs .NET 10 and spirv-val). Then rebuild the APK (the note is a shader chain input).
- Extra environment: `DM_GLSLC_CAPTURE=/sdcard/Android/data/dev.nxnative.android.peanutstest/files/glslc_capture`.
- adb over Wi-Fi (wireless debugging; the USB cable was not seen by the PC).

## Result

- Boots, reaches the frame loop, runs at 60 fps. Audio plays. Input works: A gets into the game.
- Black screen: 100 % of draws skipped (no shaders in the library), as expected.
- Turnip (custom driver `Turnip_Gen8_V39-patched`), Adreno 740, readiness PASS.
- 30 GLSLC outputs captured (2 at boot, more at scene changes).
- Memory footprint about 1.6 GB.

## Found

- **Descriptor pools.** The game creates many small texture and sampler pools (257 to 272 entries
  each). The host's table (`dm/gfx/nvn/nvn_dpool.h`, `NVN_NDPOOL 16`) holds 16; a released pool's
  slot is cleared but never reused, so from the 17th on every pool gets `[RED] texture/sampler pools
  exceed 16: later ones share the first pool's numbering` (about 290,000 lines in 90 s). Once shaders
  draw, textures will likely read through the wrong pool's numbering. First fix candidate: reuse
  released slots in `nvn_dpool_add`, or raise the limit. Not changed yet.
