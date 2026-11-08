# Star Wars Trilogy VR (fan Dreamcast game) on the Quest

This fork's "vr" build, set up for the fan-made Dreamcast game `IND-155164` ("SWTA DC").
Unofficial fan project; nothing of the game or of a Dreamcast BIOS is in this repository.

## Build the APK (Windows)

1. Install JDK 17, the Android SDK with NDK 29 and CMake 3.22.1 (see the top of `build-quest.cmd`).
2. `git checkout swta-profile` (and `git submodule update --init --recursive`).
3. Run `hotd2-vr\build-quest.cmd`.
4. The APK is `shell\android-studio\flycast\build\outputs\apk\vr\flycast-vr.apk`.

The app installs as **SW Trilogy VR** (`com.flycast.emulator.swta.vr`), next to HOTD2 VR if you have it.

## Put it on the headset

1. Sideload the APK (SideQuest, or `adb install -r flycast-vr.apk`).
2. Copy the disc image (`swta_demo.cdi`) to the headset's `Download` folder.
3. Copy a Dreamcast BIOS, `dc_boot.bin` (and `dc_flash.bin` if you have it), to the same `Download`
   folder. (Since Android 11 file managers can't write into `Android/data/<app>/`; if you prefer that
   folder, `adb push` can, as `.../files/data/dc_boot.bin`.) The app looks in its own data folder
   first, then in `Download`.
   **The BIOS is needed**: with Flycast's built-in replacement the demo stops after a few seconds.
4. Start the app from the Unknown Sources library, allow file access when asked, and press Play.

## What the profile does (`core/rend/vr_reproject.cpp`)

The game chain-loads a program per part and each keeps its GL state somewhere else in RAM, so the
profile for `IND-155164` has `scanMatrices`: twice a second it looks in RAM offsets
`0x340000..0x400000` for a GL-style perspective matrix with the viewport matrix (scale 320, 240,
centre 320, 240) `0xA0` bytes after it, and uses what it finds. On a PC it logs
`VR: projection matrix at RAM ...` when it does. Seen so far:

| Part | Projection | Viewport |
| --- | --- | --- |
| menus | `0x8C349CC0` | `0x8C349D60` |
| a level | `0x8C397E20` | `0x8C397EC0` |

The focal length comes out at 492.1 px in x and y.

## Not done / not tested

- Not run on a headset.
- No view widening (`fovScale` 1) and no hands model; the aim is the generic arcade gun.
- Only the menus and the start of a level were checked; other parts may keep the matrices elsewhere.

## Finding the matrices in another part (PC)

Build the Windows version (`build-win.cmd`), then `swta-ram-dump` has `SWTA_DUMP=N` to write
`ram_<s>.bin` and `vram_<s>.bin` to the data folder every N seconds of game time. In `emu.cfg`
the options are under `[config]`, e.g. `vr.Reproject = yes`, `vr.DebugShots = 5`.
