# 01: Survey

Run on Linux (CachyOS, Python 3.14) with `scripts/build_ipa_from_nsp.py <dump> --survey-only`,
from a `.nup` made by `nximport_cli` from the base NSP and the v393216 update.

## Layout note

The importer stored the update as a complete snapshot: the dump has `updates/393216/` only, no `base/`.
Tools that take base paths must be given the update's paths instead.

## Code modules

| module | what | build ID | text / ro / data (MB) |
|---|---|---|---|
| main | the game | `41d74a97c43ac72ca58f1173a618081a` | 5.4 / 1.5 / 0.4 (bss 19.0) |
| rtld | rtld | `2f1967113a281280a48ce3fc6dc23049` | small |
| sdk | nnSdk | `3053b22935e32ddf15ef81997603e5c2` | 5.9 / 7.0 / 0.4 |
| subsdk0 | GLSLC | `e0d8adcc446cf14d6af1a51193f72e30` | 6.5 / 3.0 / 1.0 |

- 64-bit. SDK 16.2.0 (as Super Mario Odyssey).
- SDK components: NVN, libz, nnSdk. No middleware (no CRIWARE, Unity, Unreal).
- Engine markers: NVN only.

## RomFS

10,894 files, 1.51 GB. Images in `Image2K/` and `Image720p/` (`.dig`), data in `DAT/` (`.dat`),
sound as Ogg / WAV / Opus, movies as `.ogv` and `.mp4`. A few PC leftovers (`.exe`, `.lnk`, `.vdf`).

## Imports

153 imports from main: libc, nn::os, nn::audio, nn::fs, nn::hid, nn::vi, nn::time, nn::oe,
nn::account, nn::diag, `nvnBootstrapLoader`, and GLSLC.

Spot checks against the host: account, hid, oe (CPU boost), `nvnBootstrapLoader` and `glslcCompile`
have direct hooks. Audio renderer calls go through the host's audio service (`hle_audren.c`).
Nothing found missing, not proven until it boots.

## Shaders

- No BNSH / BAHS in the RomFS.
- `shader_split.py` on the update: `shaders: 0 (); sources ;` and no `[RED]` line.
- main imports `glslcInitialize`, `glslcCompile`, `glslcFinalize`, `glslcSetAllocator`.
- `grep` for `gl_Position` / `#version` in the RomFS finds nothing: the GLSL is not stored as
  plain text (packed, scrambled or built in code; not known).

Conclusion: the game compiles its shaders at run time. This is the "game compiles its shaders while
it runs" case in `peanuts/docs/NEW-GAME.md`. It needs a booting provisional pack and a run on the
device with `DM_GLSLC_CAPTURE` before shader tables can be written.

## Provisional profile keys (guessed, no heap check on Linux)

```
modules       main subsdk0 sdk
mem_reserve   0x100000000
mem_alias     0x10000000
mem_stack     0x10000000
app_pool      0xCD500000
sys_resource  0x0
```

## Size estimate

Medium to large. Small, conventional game on a well-tested SDK, but a custom engine (no profile to
copy) and run-time shaders (the guide's hard case; no game in the repo has finished this path).
