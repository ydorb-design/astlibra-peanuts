# 02: Plan for phase 1

Goal: a test build that boots on the AYN Thor far enough to compile its shaders, then capture them.

1. Install the Android build tools on the PC: Java 21, Android SDK (platform 36, build-tools 36.0.0,
   CMake 3.22.1), NDK 28.2.13676358, adb. Packer Python packages: capstone<6, lz4, zbic==1.0.1.post1,
   fonttools. No .NET or spirv-val needed for a `--no-shaders` pack.
2. Copy the provisional profile into the local Peanuts clone, `dm/games/0100F8D01A06E000/`
   (never committed upstream until the game plays, per NEW-GAME.md).
3. Pack the code alone: `scripts/pack_android.py <the .nup> --no-shaders`.
4. Build a test app beside the release app:
   `./gradlew apk -PnxApp=peanuts -PnxAppId=peanutstest -PnxLabel="Peanuts test" -PnxGames=0100F8D01A06E000 -PnxPackEngine=fake`.
5. Install with adb, push the pack and `.nup`, add keys and firmware in the test app.
6. Set `DM_GLSLC_CAPTURE=<folder>` in the game's Extra environment and start it from the library.
7. Read the log, pull the capture folder, run `shader_split.py` again with it.

Open questions:
- Does the game boot with the guessed memory keys? (No heap check on Linux.)
- Does it reach its GLSLC compile calls before anything else fails?
