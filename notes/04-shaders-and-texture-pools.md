# 04: Real shaders, the Prepare loop and the texture pool fix (2026-11-27)

## Play kept running Prepare

- On Play the launcher checks that the pack's shader library was built from the same shader tables as the APK
  (`Pack.libraryProblem`: the pack's `shaders/v-<chain>/inputs.tsv` "inputs" digest against the one gradle bakes in from
  `dm/gfx/tools/dmsh_chain.py --inputs`, which hashes the profile's `shader_map.tsv` and friends and its `shader_*` / `nvn_*` keys).
- Any change to the profile's tables (a new capture split in) changes that digest. The pack and the APK must both be rebuilt from it.
- The test APK uses `-PnxPackEngine=fake`, whose Prepare is a simulation ("22,852 shaders" is a placeholder). It never writes a library,
  so a mismatch loops for ever.

## Capture folder pitfall

`adb pull <device>/glslc_capture ~/astlibra-work/glslc_capture` into a folder that already exists makes
`glslc_capture/glslc_capture/`. `shader_split.py` only lists the top folder. Pull into a fresh folder and `cp -n *.glslc` across.

## Rebuild routine (after a session whose capture count went up)

1. `cp -n` the new captures into `~/astlibra-work/glslc_capture/`
2. `rm -rf ~/astlibra-work/split`, run `shader_split.py` from `~/peanuts-native` (DM_GAME and DM_GLSLC_CAPTURE set)
3. `scripts/pack_android.py <nup> --shader-pkg ~/astlibra-work/split/pkg ...`
4. `./gradlew apk ... -PnxPackEngine=fake` from `android/app`
5. `adb install -r` the APK, `peanuts/scripts/push-pack.py` the pack

31 GLSLC shaders captured so far (22 + 1 pixel, 8 vertex).

## Wrong textures: descriptor pools

Symptom: geometry right, textures wrong (button icons over the title screen, scene art in the dialogue box, the HUD drawn from the font page).

- The game makes many small texture and sampler pools: 272 or 257 entries, which is NVN's 256 reserved descriptors plus 16 or 1 of its own.
  During play it keeps creating new ones over fresh memory and never finalizes the old ones; hundreds stay in use.
- The host gave each pool a 512-entry range in a 16-slot table (`dm/gfx/nvn/nvn_dpool.h`, `NVN_NDPOOL 16`). From the 17th pool on, every pool
  read the first pool's numbering. The sampler table also stops at 16,384 IDs (`mtl_reg_samp`), so raising the slot count alone drops samplers.
- Fix: [patches/peanuts-texture-pools.patch](../patches/peanuts-texture-pools.patch), against Brody's local Peanuts clone (older than upstream
  `dbbdb85`: no `vka_treg.h`). 1,024 slots; a small pool takes 16 host entries past the reserved IDs (base = start - 256); per-table limits
  (textures 1 << 20, samplers 16,384); two pool objects over the same memory share a range; when full, the least recently used pool gives up its
  slot. Log lines are rate limited (1, 10, 100, ...).
- Tested on the Thor: title, dialogue, HUD and gameplay correct over a 15 minute session; 512 texture and 256 sampler pools live, nothing evicted.
- Unit test: `dm/host/tests/t_nvn_dpool.c` (in the patch).
- Upstream candidate once it has had more play time.
