# Artoo STM32 controller firmware

## Current local modernization candidate — October29

The [Artoo modernization implementation](artoo-modernization-result-20261029.md) is the latest local software candidate. Artoo `d3cc937` adds retained fault/runtime diagnostics, bounded startup, a verified modern MAVLink1 export and CMake provenance. The selected isolated Arm15.3/Pillow13 candidate passes all six profiles and production/bench DVT each pass13 Renode scenarios. Linux graph `e0dafae6…` includes the diagnostic bridge, forwarding fix and prior permanent power-hold BSP; all four builds and16 baseline checks pass. Exact artifacts and source/checker identities are preserved. No hardware changes, flashing or pushes occurred; deployed Linux/MCU identities remain unchanged. Flash-stall recovery, persistence/watchdog design and physical acceptance remain open.

Use the [qualified reproduction instructions](/home/jresp/solo/.integration/artoo-modernization-20261030T031656Z/tooling8/reproduction.md) for the isolated newer compiler/Pillow environment and required MAVLink export. CMake/Ninja is now the maintained build graph. Plain wrapper defaults still use baseline GCC13.2/Pillow10.2; selecting the qualified newer tools is explicit. The source and artifact identities in the linked result supersede the historical build instructions below.

## Earlier bring-up and deployment records

**Latest local BSP candidate:** [permanent power-hold and updater startup fix](controller-power-fix-20261029.md), graph7bac1220…, passes four builds and16 baseline checks. Controller U-Boot asserts holds; Linux owns them until final power-off. Startup verification resynchronizes within a deadline and programming requires the bound driver. This candidate is not installed; current physical Linux/MCU remain unchanged.

**Latest physical MCU state:** [OpenSolo5 bench firmware installed](artoo-flash-result-20261029.md) after explicit authorization using temporary GPIO19/21 holds. Full readback/page127 preservation and settled exact-version/calibration/bridge checks pass; controls are released. Linux remains be98 plus its Chrony correction. Permanent BSP/updater power handling remains pending.

**Latest boot-path test:** [automatic-update preflight](artoo-boot-preflight-20261029.md) reproduces the reset in actual rcS/S60 context. The one-shot guard prevented replay, the hook is restored, and MCU firmware/calibration are unchanged.

**Latest reset work:** [October29 reset diagnosis](artoo-reset-diagnosis-20261029.md) isolates an active-reset-path failure even outside ROM mode. The controller recovered with its original firmware/calibration; no programming occurred. J2 console/electrical evidence is needed before another programming attempt.

**Latest physical readiness:** [October29 serial/flash review](artoo-flash-readiness-20261029.md) passes the live application UART and candidate/tool/calibration checks. Reset/ROM entry remains unqualified; no STM32 flash has occurred. Battery-only power is not established as the cause of the prior restart.

Updated October 28, 2026. Native builds and software regression checks work in
WSL. The original recovery research is retained below as a historical snapshot.
No MCU programming or physical debugger connection has occurred.

## Historical working build before CMake migration

Primary checkout: `/home/jresp/Active_GH/artoo-firmware`, branch `codex/artoo-wsl`,
revision `48034572145e9e62dff881141277564e1ed1d3a6`. The recovered
`/home/jresp/solo/apps/artoo-firmware` checkout remains preserved. Canonical
upstream is `/home/jresp/Active_GH/OpenSolo`; `UPSTREAM.json` records its
OpenSolo 4 lineage and owner patch. Local OpenSolo 5 branding and reliability
changes are not an upstream release. No remote history was rewritten or pushed.

Tup 0.7.11, ARM GCC 13.2/binutils/newlib and GDB multiarch are installed. A project Python 3.12 environment uses pinned Pillow 10.2.0. The build repairs Pillow/font metrics, explicit version generation, packed checksum accesses and intentional LCD fallthrough while retaining compiler warnings as errors. A UBSan host regression verifies an independent heartbeat wire CRC and unaligned checksum operations.

Run:

```bash
cd /home/jresp/Active_GH/artoo-firmware
./scripts/build-wsl.sh production dvt
```

See [WSL_BUILD.md](/home/jresp/Active_GH/artoo-firmware/WSL_BUILD.md) for dependencies and workflow. The wrapper cleans only known ignored outputs before regenerating Tup's standalone script, so subsequent builds work too.

The preserved calibration-correction production DVT candidate is
`5.0.0-dev+g89fc1d44680c`: **190,956 bytes**, **69,140 bytes** before the
reserved parameter page, **6,840 bytes** of static RAM, 11 constructors and
zero preinitializers. Its bounded heap ends below a minimum 4 KiB stack
reservation. The largest compiler-reported individual frame is 288 bytes;
this is not a complete call-chain/interrupt stack or physical high-water proof.
[Final build receipt](/home/jresp/solo/.integration/artoo-builds/linear-calibration-20261028/production-dvt/verification.json)
binds the source, ELF/BIN hashes, native tests and exact board/profile arguments.
All 800 generated images/glyphs render the exact baseline RGB565 pixels. Tamp
compression saved 59,472 bytes in its comparison build and adds 312 bytes of
static workspace; palettes and descriptor sizes are unchanged. See the
[Tamp implementation report](artoo-tamp-integration-20261028.md).
Newlib syscall-stub warnings remain visible.

Hardware preflight found a valid zero trim on the linear rate dial. The current
firmware and SoloLink updater `46d677a` preserve that calibration; native tests
and all 32 Renode checks passed. See [the calibration correction](artoo-linear-calibration-20261028.md).
The controller still runs `OpenSolo-4dev`; no MCU programming has started.
The current local candidate adds an explicit bench-only idle-countdown option.
All six board/profile builds and native regressions passed; normal and bench
DVT each passed all 32 canonical Renode checks. Saved manifests identify
`5.0.0-dev+g48034572145e` and
`5.0.0-dev+g48034572145e-bench-noidle`. Neither was selected for flashing or
promoted into the Yocto image. See [bench qualification](artoo-bench-idle-20261028.md).

The Wi-Fi fault was traced to the controller BSP's PCIe MSI/legacy-interrupt
configuration. The permanent controller-only kernel fix and actual deployment
state are recorded in [Wi-Fi kernel qualification](controller-wifi-pci-20261028.md).
The separate radio-name fix resolves the PHY dynamically through sysfs.
Keys, credentials, calibration, and the existing manual Artoo package are preserved.

The four production/factory build profiles pass: `production dvt`, `production bb03`,
`factory dvt`, `factory bb03`; `bench dvt` and `bench bb03` also pass. Production excludes destructive factory commands
while retaining raw input/calibration diagnostics. Factory builds remain separate
artifacts. DVT is hardware ID 3 and the default; BB03 is ID 2, with a digital
LCD backlight instead of DVT PWM. A compiled ID does not identify a physical PCB.

The [audit resolution ledger](artoo-modernization-resolution-20261028.md)
tracks transport, persistence, input validation, clock/ISR/DMA ownership,
factory isolation, renderer bounds and runtime helper fixes, plus remaining
hardware/design gates. [Renode software tests](artoo-renode.md) provide finite
reset, poisoned-memory, constructor and task-loop checks; native sanitizer
fixtures cover additional protocol, input, command and graphics behavior.

The exact final candidate also passed **three Renode boot scenarios, two
negative controls, 26 real UART receive regressions and a compressed splash
render check**. The render check observes successful Tamp decoding followed by
splash update and main-loop continuation. Those UART cases
consumed 636 bytes with 15 expected rejections and no overruns, drops or line
errors. [Final Renode evidence](/home/jresp/solo/.integration/artoo-builds/renode-tamp-6eddd33-qualification-r2-20261028/result.json)
retains peripheral-model warnings and scope limits; physical UART/DMA, ADC,
charging, flash and display behavior remain hardware acceptance work.

This candidate has not replaced the separately pinned Yocto MCU artifact.
The earlier branding-only payload is packaged under `/firmware/candidates/`,
outside the boot updater's automatic discovery pattern. The current Linux
rebuild remains set aside for final integration/validation. The Python updater
and loader were hardened separately; see [the Python report](python3-compatibility-20261028.md).
No update was enabled or flashed. Earlier
[recovery artifacts](/home/jresp/solo/.integration/artoo-builds/rebase-20261028/verification.json)
and [branding artifacts](/home/jresp/solo/.integration/artoo-builds/opensolo5-branding)
are preserved independently.

## Role and hardware evidence

The handheld controller contains two separate software targets. The i.MX6 runs the Yocto Linux image, networking and video. The Artoo STM32 handles joystick/camera-control sampling, buttons, the LCD, LEDs/vibration, battery/charging and power sequencing. See the [OpenSolo architecture description](https://github.com/OpenSolo/OpenSolo/wiki/Artoo-STM32).

The firmware targets Cortex-M3 with 256 KiB flash and 48 KiB SRAM; its last 2 KiB flash page stores parameters. The [architecture flags](/home/jresp/solo/apps/artoo-firmware/Tupfile:67), [linker memory map](/home/jresp/solo/apps/artoo-firmware/src/ldscript.ld:5) and [parameter layout](/home/jresp/solo/apps/artoo-firmware/src/params.h:89) establish this software target. Canonical [flashing documentation](https://github.com/OpenSolo/OpenSolo/blob/master/documentation/artoo/flash-custom-firmware.md) records bootloader chip ID `0x414`, a high-density STM32F1 target.

STM32F103ZC is consistent with those requirements, but the exact package marking and installed PCB revision are unverified. The local Artoo HDTC/HDMI schematic is the i.MX6 board, not an STM32 mainboard schematic/BOM. The [pinout PDF](</home/jresp/solo/hardware-docs/Artoo pinout cheat sheet.pdf>) describes J3 as the connection to the STM32 board. The source defaults to [BOARD_DVT](/home/jresp/solo/apps/artoo-firmware/src/board.h:4); its reported hardware version is a compiled constant and cannot independently identify a physical PCB.

The LCD code uses an ILI9341 parallel/FSMC driver with Tianma initialization data and 320×240 assets. [USART1 PA9/PA10](/home/jresp/solo/apps/artoo-firmware/src/board_dvt.h:117) connects to i.MX6 UART2, exposed as `/dev/ttymxc1`, at 115200 baud with SLIP framing. See [host protocol initialization](/home/jresp/solo/apps/artoo-firmware/src/hostprotocol.cpp:35).

## Repositories and workspace tracking

**Historical recovery snapshot:** repository revisions, package inventories,
identified updater defects and planning steps in the following research sections
describe the initial recovery state. Use the current build section above and
linked resolution/Python/Renode guides for implemented changes and acceptance.

| Repository | Role and checked state |
|---|---|
| [Decryptortuning/artoo-firmware, PORT](https://github.com/Decryptortuning/artoo-firmware/tree/PORT) | Owner's working MCU port at `/home/jresp/solo/apps/artoo-firmware`, commit `31901c3500eee205534fade61e5c19b0e3622e19`. Public standalone import, not a GitHub-linked fork. Default `main` is `5788cb060e56748659c9bc0f5627018fb4b70ae0`. |
| [Decryptortuning/stm32loader, PORT](https://github.com/Decryptortuning/stm32loader/tree/PORT) | Separate Python 3 uploader at `/home/jresp/solo/apps/stm32loader`, commit `9d33c637fa46ac251acc9a060d4e541084dc3e4c`. This is not the joystick/LCD application firmware. |
| [OpenSolo/OpenSolo/artoo](https://github.com/OpenSolo/OpenSolo/tree/master/artoo) | Canonical consolidated source; local reference is `/home/jresp/solo/OPENSOLO_REFERENCE/artoo`. Repository `master` is `5b745243a2d88691ccd045e7238742431c668a5a`; latest repository push was in 2022. |
| [OpenSolo/artoo](https://github.com/OpenSolo/artoo) | Archived standalone repository, deprecated in favor of the consolidated source. Last push was in 2018. |
| [OpenSolo/stm32loader](https://github.com/OpenSolo/stm32loader) | Archived upstream loader source. |

`OPenSolo_2` has no `.gitmodules` and no Git submodule entries. Its `workspace-lock.json` records the separate Artoo and STM32Loader repositories and their exact revisions. Cloning the integration repository alone does not recursively fetch these workspace checkouts.

The owner's initial firmware import matches old OpenSolo `3.0.0` core code; the PORT commit updates 14 Python utilities. Later canonical Artoo changes, including the January 2020 SmartRTL long-home-button action, are absent from that baseline. Compare and deliberately choose the source baseline before carrying changes forward. Historical provenance remains in [SOURCE](/home/jresp/solo/apps/artoo-firmware/SOURCE:1).

## Historical build pipeline and image integration at recovery

The firmware uses [Tup](/home/jresp/solo/apps/artoo-firmware/Tupfile:75). It generates resources, compiles C++11 with `arm-none-eabi` GCC, links custom startup/vector/register code and emits ELF, HEX and BIN outputs. It uses newlib/nano, libm, libgcc and libnosys; production disables exceptions, RTTI and semihosting. This is separate from the Linux cross compiler used by Yocto.

The current [Yocto recipe](/home/jresp/solo/workspace/layers/meta-3dr/recipes-firmware/artoo/artoo-firmware_3.0.0.bb:8) packages checked-in `files/artoo.bin` as `/firmware/artoo_3.0.0.bin`. It does not fetch or compile `apps/artoo-firmware`; editing that checkout alone does not change the SD image. Historical [solo-builder/build_artoo.sh](/home/jresp/solo/OPENSOLO_REFERENCE/solo-builder/build_artoo.sh:5) built with Tup and copied the result into the recipe.

The packaged binary is 241,857 bytes, contains `v3.0.0`, and has SHA-256 `70024792f4b1d7f903397cc91e91a79341149c63c4d3431fcac7f4d899179135`. Its initial stack vector is `0x2000c000`, consistent with 48 KiB SRAM. The linked application limit is `0x3f800` bytes, leaving 18,239 bytes of size headroom relative to this payload; a new compiler's output needs its own memory checks.

The Linux image invokes `checkArtooAndUpdate.py` during startup, but firmware writes are opt-in in the owner's port. The loader operates on the controller's internal UART and GPIOs; it is not a generic PC USB flasher. Its software still needs review before enabling updates: subprocess output is bytes but [strip uses a string](/home/jresp/solo/apps/sololink/STM32Loader/checkArtooAndUpdate.py:213), and the candidate is [moved to loaded even when updates are disabled](/home/jresp/solo/apps/sololink/STM32Loader/checkArtooAndUpdate.py:323).

## Original build qualification proposal

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

## Original recovery work list

1. Choose the source baseline and explicitly account for missing canonical firmware changes.
2. Make Python interpreter/dependency selection reproducible. Tup still invokes literal `python`; activating a project virtual environment can supply it without a global Python 2 alias.
3. Port asset generation's [FreeTypeFont.getsize call](/home/jresp/solo/apps/artoo-firmware/tools/assetgen.py:189) to current Pillow while preserving glyph dimensions and baseline placement. Pillow removed this API in [version 10](https://pillow.readthedocs.io/en/stable/releasenotes/10.0.0.html). A Python 3.11/Pillow 9.5 environment is a temporary compatibility option, not a demonstrated baseline. Font files already exist; the renamed fonts are documented Roboto copies.
4. Fix explicit version provenance. `git describe --tags` fails for the owner's disconnected import history; the [shell pipeline](/home/jresp/solo/apps/artoo-firmware/Tupfile:88) can otherwise embed an empty version. Preserve meaningful source identity rather than silently accepting an empty string.
5. Build ELF/BIN in an isolated native checkout and check vectors, constructors, binary size below `0x3f800`, SRAM/stack headroom, glyph output and protocol behavior. Preserve the supplied v3.0.0 binary as the comparison artifact.
6. Define how the accepted binary and its true version enter Yocto. Review the enabled-update path before any MCU programming.

Hardware work later needs the actual MCU marking, board revision/pinout, target voltage/reference/reset handling and an explicitly selected probe or controller-side loader procedure. The sample `.gdbinit` attaches and writes a debug register, so do not treat it as a read-only inspection script. WSL USB transport is described by [Microsoft](https://learn.microsoft.com/en-us/windows/wsl/connect-usb).

## Local layout and rebuild plan — October 28, 2026

This is the proposed execution sequence. The checkouts have not been moved, rebased or built by this planning step.

### Target layout

| Location | Ownership |
|---|---|
| `/home/jresp/Active_GH/artoo-firmware` | Primary working checkout of Decryptortuning's MCU firmware. |
| `/home/jresp/Active_GH/stm32loader` | Primary working checkout of the controller-specific uploader. |
| `/home/jresp/Active_GH/OpenSolo` | Canonical upstream checkout used to compare and extract component history. |
| `/home/jresp/Active_GH/OPenSolo_2` | Integration manifest, build/validation wrappers and source-to-Yocto handoff. |
| `/home/jresp/solo/.integration/artoo-builds` | Versioned build reports and candidate artifacts, separate from accepted images. |
| `/home/jresp/solo/workspace/docs` | Central procedures, findings and acceptance evidence. |

Use WSL Git and native Linux tooling. Keep source repositories independent and record them in the integration manifest; this plan does not introduce submodules. Python environments and Tup-generated files belong to ignored per-repository locations. Pin external compiler/dependency versions and preserve accepted ELF/MAP/BIN artifacts with their source/tool identities.

### 1. Preserve and establish the working checkouts

Record existing heads, branches, tags and local changes. Preserve the current source refs and supplied v3.0.0 binary. Create the new repository-named working checkouts from the existing owner repositories, retaining history and any local-only refs; verify them before retiring the old app locations. Do not remove the recovered/reference sources or change remote main/PORT branches.

Update workspace discovery, the source inventory and active documentation together to support the separate development root. Keep archived locks as evidence of their original locations. Leave the current fetched-source Yocto baseline in place while qualifying the MCU build.

Acceptance: one documented primary working checkout per component, complete recoverable history and a consistent source inventory; both existing Yocto configuration guards still pass.

### 2. Reconcile with canonical upstream

Pin the canonical OpenSolo revision, compare its `artoo/` component against the owner import and inventory all differences. Extract the Artoo subtree history into a standalone upstream baseline. On a new `codex/` branch, replay the owner's actual Python-port and documentation changes onto that component baseline. Review conflicts and intentional differences; do not blindly rebase the standalone root onto the full monorepo root.

Do the equivalent provenance/delta review for STM32Loader, retaining its controller-specific UART/GPIO behavior. Choose the matching canonical component baseline before replaying the Python 3 changes; do not substitute an unrelated generic PC uploader.

Acceptance: reviewed owner-versus-upstream delta, retained later canonical features, recorded source-to-subtree mapping and preserved original refs. No force-push or remote branch replacement is part of this local phase.

### 3. Restore a reproducible software-only build

Install and pin the qualified candidate tools: Tup, ARM bare-metal GCC/binutils/newlib and a project Python virtual environment with Pillow. First address interpreter selection, font metric compatibility and explicit version provenance. Retain the Tup pipeline and begin with its generated build-script path. Resolve compiler/runtime issues surfaced by the actual build without disabling diagnostic checks globally.

Build firmware and validate the uploader's Python behavior without any serial device. Check ELF vectors/constructors, FLASH/parameter-page boundaries, SRAM/stack headroom, asset geometry and protocol fixtures. Preserve the exact compiler/package versions, source revisions and generated ELF/MAP/BIN/hash reports.

Acceptance: clean, repeatable firmware builds with meaningful embedded versions, validated memory/assets and hardware-free loader/updater checks. This is the first milestone before feature updates; it does not establish board runtime correctness.

### 4. Connect the accepted firmware to Yocto

Define an explicit artifact contract: MCU source revision, real firmware version, toolchain identity, binary hash and board selection. Replace the manual stale-binary handoff with a documented build/package step or an appropriately scoped recipe that consumes the accepted artifact. Decide this implementation after the standalone build works.

Repair the Linux-side updater's bytes handling and disabled-update candidate handling, with failure/skip tests. Rebuild and inspect the controller image to prove it carries the intended binary and metadata. Keep MCU programming opt-in.

Acceptance: the packaged bytes/version match the accepted standalone artifact, and disabled/failed updates have checked behavior. Source edits alone must no longer be mistaken for a newly packaged MCU build.

### 5. Apply scoped updates, then validate hardware

After the reconciled baseline builds, prioritize updater/calibration preservation, joystick/button behavior, LCD assets/UI and any desired control features as separate changes. Keep feature changes separate from toolchain migration so regressions can be attributed. Record memory and asset impact for each change.

Verify the actual MCU marking/PCB revision and debug connection before selecting
a programming procedure. Follow the user's authorized hardware scope and current
workspace instructions. MCU writes remain outside the present local-only work.
Hardware acceptance covers joystick calibration, buttons, display, battery/power
behavior and the i.MX link; an SD-card build does not qualify a new STM32 binary.

## Python updater hardening — October 28, 2026

The locally committed updater/loader now fail on subprocess/verification failures,
retain skipped/failed candidates, verify calibration and the running version, and
never automatically escalate ordinary retries to readout-unprotect or mass erase.
See [Python compatibility review](python3-compatibility-20261028.md) for exact
behavior, mocked regression coverage and source handoff. These changes have not
been deployed or used to flash the STM32; the physical controller still runs
`OpenSolo-4dev`.


## Local OpenSolo 5 branding — October 28, 2026

The local Artoo candidate at `60a2707305e671137ed11f2e406964a8e36cd2b0` now
uses an **OPENSOLO 5** startup/shutdown logo, `5.0.0-dev` development version,
and OpenSolo 5 support prompts. The upstream source lineage remains OpenSolo 4;
this is not a claim of an upstream release or hardware qualification.

The original 212 × 48 pixel canvas and shared RGB565 palette are retained.
A complete native WSL firmware build passed, including the existing MAVLink
CRC/unaligned-access regression. Decoding the compiled RLE asset produced an
exact RGB565 match to the new PNG. The binary is 244,744 bytes, leaving 15,352
bytes before the reserved parameter page. The embedded version is
`5.0.0-dev+g60a2707305e6`. Pixel-accurate physical LCD appearance remains untested.

Source artwork: `/home/jresp/Active_GH/artoo-firmware/resources/images/logo_startup/Solo_Logo.png`.
Branding source notes: `/home/jresp/Active_GH/artoo-firmware/resources/BRANDING.md`.
Local artifacts and verification: `/home/jresp/solo/.integration/artoo-builds/opensolo5-branding`.
The searchable gallery at `/home/jresp/solo/.integration/artoo-builds/lcd-assets/gallery.html`
reflects the new local artwork.

No target changes, automatic-update enablement, flashing, pushing, or replacement
of the accepted Yocto MCU payload were performed. The controller still runs
`OpenSolo-4dev`. The separate Python updater hardening is described in
[the compatibility report](python3-compatibility-20261028.md).

## Asset compression experiment — October 28, 2026

An isolated comparison covered all 123 images and 677 generated font glyphs.
Tamp extended with a 256-byte window gave the best measured size: approximately
58.4 KiB projected net savings on linked assets after stream metadata and an
ARM decoder/wrapper allowance. All tested streams passed bounded-chunk
sanitizer checks; three Cortex-M3 emulation runs verified final RGB565 pixels
for every asset. Compression is not integrated or flashed, and actual LCD
timing remains unmeasured. See [the complete experiment](artoo-compression-20261028.md)
for corpus identities, comparisons, limits and reproduction evidence.

## Bench idle-shutdown override — October 28, 2026

The camera paddle is disconnected on the open controller, so the existing
A/Pause/paddle idle override cannot be used. A temporary updater-display hold
was tried through the normal STM32 bridge. It successfully prevented idle
shutdown, but source review found that `PowerManager::canShutDown()` also
rejects normal software shutdown while the updater UI is active. Earlier
claims that ordinary shutdown protection was unchanged were incorrect.

The owner dismissed the maintenance display with A, returning to the normal
screen. The marker, runlevel link and boot helper were removed. No firmware
programming was involved. Do not use the updater UI as a standing bench
keep-awake feature; implement an idle-only development setting for that purpose.
