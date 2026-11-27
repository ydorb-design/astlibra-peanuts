# ASTLIBRA Revision on Peanuts Native

Notes for bringing up ASTLIBRA Revision (Switch 1, KEIZO / WhisperGames) in
[Peanuts Native](https://github.com/igawa6/peanuts-native). The game is not on Peanuts' supported list.

This repo holds notes and, later, draft profile files. It never holds game files, keys, firmware,
`.nup` files, packs or logs with game data.

## Status

Phase 0, survey: done. See [notes/01-survey.md](notes/01-survey.md).

Phase 1, shaderless test build and GLSLC capture: done, 30 shaders captured. See [notes/03-first-boot.md](notes/03-first-boot.md).

Phase 1b, real shaders and the texture pool fix: plays with correct graphics on the Thor. See [notes/04-shaders-and-texture-pools.md](notes/04-shaders-and-texture-pools.md).

## Game

| | |
|---|---|
| Title ID | `0100F8D01A06E000` |
| Update | `0100F8D01A06E800`, version 393216 |
| main build ID | `41d74a97c43ac72ca58f1173a618081a` |
| SDK | 16.2.0 |
| Engine | none detected (KEIZO's own) |
| Graphics | NVN, shaders compiled at run time with GLSLC |
