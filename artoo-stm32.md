# Artoo STM32 controller firmware

Research date: October 28, 2026. This identifies the firmware sources and a candidate native WSL build setup. No STM32 firmware build, install, debugger connection or MCU flash was performed in this research.

## Role and hardware evidence

The handheld controller contains two separate software targets. The i.MX6 runs the Yocto Linux image, networking and video. The Artoo STM32 handles joystick/camera-control sampling, buttons, the LCD, LEDs/vibration, battery/charging and power sequencing. See the [OpenSolo architecture description](https://github.com/OpenSolo/OpenSolo/wiki/Artoo-STM32).

The firmware targets Cortex-M3 with 256 KiB flash and 48 KiB SRAM; its last 2 KiB flash page stores parameters. The [architecture flags](/home/jresp/solo/apps/artoo-firmware/Tupfile:67), [linker memory map](/home/jresp/solo/apps/artoo-firmware/src/ldscript.ld:5) and [parameter layout](/home/jresp/solo/apps/artoo-firmware/src/params.h:89) establish this software target. Canonical [flashing documentation](https://github.com/OpenSolo/OpenSolo/blob/master/documentation/artoo/flash-custom-firmware.md) records bootloader chip ID `0x414`, a high-density STM32F1 target.

STM32F103ZC is consistent with those requirements, but the exact package marking and installed PCB revision are unverified. The local Artoo HDTC/HDMI schematic is the i.MX6 board, not an STM32 mainboard schematic/BOM. The [pinout PDF](</home/jresp/solo/hardware-docs/Artoo pinout cheat sheet.pdf>) describes J3 as the connection to the STM32 board. The source defaults to [BOARD_DVT](/home/jresp/solo/apps/artoo-firmware/src/board.h:4); its reported hardware version is a compiled constant and cannot independently identify a physical PCB.

The LCD code uses an ILI9341 parallel/FSMC driver with Tianma initialization data and 320×240 assets. [USART1 PA9/PA10](/home/jresp/solo/apps/artoo-firmware/src/board_dvt.h:117) connects to i.MX6 UART2, exposed as `/dev/ttymxc1`, at 115200 baud with SLIP framing. See [host protocol initialization](/home/jresp/solo/apps/artoo-firmware/src/hostprotocol.cpp:35).

## Repositories and workspace tracking

| Repository | Role and checked state |
|---|---|
| [Decryptortuning/artoo-firmware, PORT](https://github.com/Decryptortuning/artoo-firmware/tree/PORT) | Owner's working MCU port at `/home/jresp/solo/apps/artoo-firmware`, commit `31901c3500eee205534fade61e5c19b0e3622e19`. Public standalone import, not a GitHub-linked fork. Default `main` is `5788cb060e56748659c9bc0f5627018fb4b70ae0`. |
| [Decryptortuning/stm32loader, PORT](https://github.com/Decryptortuning/stm32loader/tree/PORT) | Separate Python 3 uploader at `/home/jresp/solo/apps/stm32loader`, commit `9d33c637fa46ac251acc9a060d4e541084dc3e4c`. This is not the joystick/LCD application firmware. |
| [OpenSolo/OpenSolo/artoo](https://github.com/OpenSolo/OpenSolo/tree/master/artoo) | Canonical consolidated source; local reference is `/home/jresp/solo/OPENSOLO_REFERENCE/artoo`. Repository `master` is `5b745243a2d88691ccd045e7238742431c668a5a`; latest repository push was in 2022. |
| [OpenSolo/artoo](https://github.com/OpenSolo/artoo) | Archived standalone repository, deprecated in favor of the consolidated source. Last push was in 2018. |
| [OpenSolo/stm32loader](https://github.com/OpenSolo/stm32loader) | Archived upstream loader source. |

`OPenSolo_2` has no `.gitmodules` and no Git submodule entries. Its `workspace-lock.json` records the separate Artoo and STM32Loader repositories and their exact revisions. Cloning the integration repository alone does not recursively fetch these workspace checkouts.

The owner's initial firmware import matches old OpenSolo `3.0.0` core code; the PORT commit updates 14 Python utilities. Later canonical Artoo changes, including the January 2020 SmartRTL long-home-button action, are absent from that baseline. Compare and deliberately choose the source baseline before carrying changes forward. Historical provenance remains in [SOURCE](/home/jresp/solo/apps/artoo-firmware/SOURCE:1).

## Existing build pipeline and image integration

The firmware uses [Tup](/home/jresp/solo/apps/artoo-firmware/Tupfile:75). It generates resources, compiles C++11 with `arm-none-eabi` GCC, links custom startup/vector/register code and emits ELF, HEX and BIN outputs. It uses newlib/nano, libm, libgcc and libnosys; production disables exceptions, RTTI and semihosting. This is separate from the Linux cross compiler used by Yocto.

The current [Yocto recipe](/home/jresp/solo/workspace/layers/meta-3dr/recipes-firmware/artoo/artoo-firmware_3.0.0.bb:8) packages checked-in `files/artoo.bin` as `/firmware/artoo_3.0.0.bin`. It does not fetch or compile `apps/artoo-firmware`; editing that checkout alone does not change the SD image. Historical [solo-builder/build_artoo.sh](/home/jresp/solo/OPENSOLO_REFERENCE/solo-builder/build_artoo.sh:5) built with Tup and copied the result into the recipe.

The packaged binary is 241,857 bytes, contains `v3.0.0`, and has SHA-256 `70024792f4b1d7f903397cc91e91a79341149c63c4d3431fcac7f4d899179135`. Its initial stack vector is `0x2000c000`, consistent with 48 KiB SRAM. The linked application limit is `0x3f800` bytes, leaving 18,239 bytes of size headroom relative to this payload; a new compiler's output needs its own memory checks.

The Linux image invokes `checkArtooAndUpdate.py` during startup, but firmware writes are opt-in in the owner's port. The loader operates on the controller's internal UART and GPIOs; it is not a generic PC USB flasher. Its software still needs review before enabling updates: subprocess output is bytes but [strip uses a string](/home/jresp/solo/apps/sololink/STM32Loader/checkArtooAndUpdate.py:213), and the candidate is [moved to loaded even when updates are disabled](/home/jresp/solo/apps/sololink/STM32Loader/checkArtooAndUpdate.py:323).

## Recommended build qualification setup

Keep the existing Tup pipeline and use native Ubuntu-24.04 WSL tooling. The following are available package candidates on this host, not a qualified firmware build result:

| Tool | Candidate and purpose |
|---|---|
| Tup | Noble 0.7.11; source build orchestration. |
| ARM bare-metal GCC | `gcc-arm-none-eabi` 13.2.Rel1, matching binutils/newlib, including `nano.specs`. |
| Python environment | Project Python 3.12 virtual environment with a pinned Pillow 10.2 baseline after the font API port. The virtual environment supplies the literal `python` executable used by Tup. |
| ELF/debug tools | `gdb-multiarch` 15.1 and ARM binutils for symbol/vector/size inspection. |
| Hardware debugger, later | OpenOCD 0.12 with a verified suitable probe, or Black Magic Probe's own GDB server. Hardware connection is a separate confirmed step. |

The WSL host has Python 3.12, make/CMake/Ninja and FUSE support. Tup, bare-metal ARM GCC, GDB multiarch and OpenOCD are absent from PATH/package inventory, and system Python has no Pillow. No packages were installed during this research. See [Noble Tup](https://packages.ubuntu.com/noble/tup), [Noble ARM GCC](https://packages.ubuntu.com/noble/gcc-arm-none-eabi), [Tup's manual](https://gittup.org/tup/manual.html) and [Black Magic GDB documentation](https://black-magic.org/docs/usage/gdb-commands/).

For an initial isolated build, use `tup generate build-once.sh` and then run the generated script. This is the existing [CI pattern](/home/jresp/solo/apps/artoo-firmware/.travis.yml:19) and avoids FUSE tracking for that first build. It should be qualified against the actual source and pinned tools before becoming the documented normal build command.

Historical documentation used Ubuntu 14.04 Vagrant, Python 2 and unpinned PPA/pip tooling. It does not establish an exact reproducible legacy compiler baseline. Modern native tools are the recommended qualification candidate. The build enables `-Werror`, so new compiler diagnostics, runtime/link differences and code size need qualification. Retain the existing pipeline while establishing that baseline.

## Work required before a trusted new binary

1. Choose the source baseline and explicitly account for missing canonical firmware changes.
2. Make Python interpreter/dependency selection reproducible. Tup still invokes literal `python`; activating a project virtual environment can supply it without a global Python 2 alias.
3. Port asset generation's [FreeTypeFont.getsize call](/home/jresp/solo/apps/artoo-firmware/tools/assetgen.py:189) to current Pillow while preserving glyph dimensions and baseline placement. Pillow removed this API in [version 10](https://pillow.readthedocs.io/en/stable/releasenotes/10.0.0.html). A Python 3.11/Pillow 9.5 environment is a temporary compatibility option, not a demonstrated baseline. Font files already exist; the renamed fonts are documented Roboto copies.
4. Fix explicit version provenance. `git describe --tags` fails for the owner's disconnected import history; the [shell pipeline](/home/jresp/solo/apps/artoo-firmware/Tupfile:88) can otherwise embed an empty version. Preserve meaningful source identity rather than silently accepting an empty string.
5. Build ELF/BIN in an isolated native checkout and check vectors, constructors, binary size below `0x3f800`, SRAM/stack headroom, glyph output and protocol behavior. Preserve the supplied v3.0.0 binary as the comparison artifact.
6. Define how the accepted binary and its true version enter Yocto. Review the enabled-update path before any MCU programming.

Hardware work later needs the actual MCU marking, board revision/pinout, target voltage/reference/reset handling and an explicitly selected probe or controller-side loader procedure. The sample `.gdbinit` attaches and writes a debug register, so do not treat it as a read-only inspection script. WSL USB transport is described by [Microsoft](https://learn.microsoft.com/en-us/windows/wsl/connect-usb).
