# Artoo modernization: second research and implementation plan

**Implementation status:** see the [local qualification result](artoo-modernization-result-20261029.md). Waves A–D and F are implemented/evaluated; wave E bounds clock/ADC/battery initialization, with flash-BSY recovery explicitly deferred pending RAM execution/failure-policy design. CMSIS, persistence, watchdog and physical gates remain open. The planning baseline and unresolved archive note below are historical; the result records the subsequently resolved Arm archive and final candidates.

## Original planning snapshot

October 29, 2026. Three fresh research agents reviewed runtime reliability,
tooling/vendor updates and protocol integration. The parent reviewed the actual
startup, flash, task, build and Linux forwarding code and consolidated this plan.
This is research and planning, not a new firmware qualification. No builds,
tests, installations, hardware access, flashing or pushes occurred.

## Baseline and decisions

Artoo source is clean at `48034572145e9e62dff881141277564e1ed1d3a6` in
`/home/jresp/Active_GH/artoo-firmware`. SoloLink is
`045b2138494e6544b5a24378bd6fc2dd8175817c`; integration is
`72731fa095fee046785790bf1891b245a8959f68`. The physical MCU runs the
previously verified OpenSolo5 bench image from this Artoo source. Linux still
runs be98 plus the Chrony correction. The permanent power-hold graph `7bac1220…`
is locally qualified but not installed or electrically accepted.

The previous reliability waves, Tamp integration, calibration correction and
bench profile remain the foundation. Do not repeat their completed packet,
IRQ ownership, DMA, parameter semantics, heap-bound or renderer fixes. Current
production DVT is 190,956 bytes, leaving 69,140 bytes before the reserved page;
these are prior artifact measurements, not measurements from this review.

Recommended order: diagnostics; focused forwarding correction; build provenance
and CMake parity; independent dependency upgrades; bounded hardware waits;
embedded MAVLink regeneration; selective CMSIS adoption. Persistence redesign
and active watchdog recovery require separate cross-component decisions and
physical gates. Keep Yocto 6, license work, full HAL/RTOS migration and automatic
firmware promotion outside this round.

## Source-confirmed findings

Paths below are relative to Artoo unless explicitly marked SoloLink.

| Finding | Evidence and consequence |
|---|---|
| Silent faults and unexpected IRQs | `src/stm32/vectors.cpp`: fault handlers loop forever; default ordinary IRQ returns without addressing its source, allowing repeated entry. |
| Existing reset capture, conflicting retained storage | `src/stm32/rcc.cpp` already latches/clears RCC flags. `src/main.cpp` calls `Bkp::reset()`, resetting the backup domain. Reuse the latch; do not assume backup-register crash data survives startup. |
| Pre-runtime unbounded clock waits | `src/stm32/setup.cpp` polls HSI/HSE/PLL/switch status before data/BSS initialization. Runtime timeouts or ordinary initialized globals cannot simply be called there. |
| Other unbounded waits | `src/stm32/mcuflash.cpp` waits indefinitely on BSY; ADC calibration/conversion and battery initialization also need explicit failure paths. |
| Limited runtime measurements | `src/tasks.cpp` has atomic handoff; `.su` reports and bounded `_sbrk` already exist. Runtime stack/heap high-water, dispatch delay and fault telemetry are absent. |
| Build input gaps | `scripts/build-wsl.sh` uses ambient compiler PATH and a fixed venv, shared output names and source-tree generated assets. Its tracked-diff dirty flag misses untracked compile inputs selected by source globs. |
| Destructive helper cleanup | `scripts/run-checker.sh` uses broad ignored-file cleanup; `scripts/versioned-binaries.sh` removes its output directory. Replace with explicit owned paths and immutable destinations. |
| Broken regeneration helper | `scripts/regen-mavlink-headers.sh` has literal `ARTOODIR` in its cleanup path, `/bin/sh` with `pushd/popd`, ambient Python and an unpinned sibling source. Do not run it as-is. |
| Separate old MCU MAVLink | Embedded generated headers report November 2015/MAVLink1. Linux's recent generator modernization did not update these files. |
| Linux forwarding defect | SoloLink `flightcode/stm32/Telem.cpp:94` compares message ID with LOG_DATA ID120, payload length97 and CRC-extra134. Only120 is the intended log message;134 is TERRAIN_DATA. This is a confirmed incorrect filter, not proof of an observed flight feature failure. |
| MAVLink2 transport incompatibility | That same bridge requires magic0xFE, payload+8 length and message ID at byte5. Updating only the MCU cannot establish end-to-end MAVLink2. |

## Wave A — diagnostics and runtime evidence

Owned changes: Artoo exception entry, linker/startup, reset snapshot, task/heap
instrumentation and host protocol; SoloLink packet enum/dispatcher and a local
read-only diagnostic interface; integration native and Renode fixtures.

1. Add a fixed-size retained RAM record in a linker-owned `.noinit` section:
   magic, schema/length, checksum, fault kind, IPSR, EXC_RETURN, raw stack pointer,
   validated stacked PC/LR/xPSR, fault status/address registers and task identity.
   Capture MSP/PSP before a C prologue changes context. Validate the frame address
   before reading it; tolerate bad stacks and nested faults. Commit validity
   last. No heap, formatting, flash writes, UART access or ordinary locks in the
   handler. Initial behavior captures then halts; no automatic reset.
2. Report existing latched reset flags. Distinguish cold/random RAM from valid
   retained data. Do not promise retention across power loss. Review emergency
   charge/balance/motor output policy separately from i.MX power holds.
3. Add bench-gated stack/heap high-water and OOM counters, task count/duration,
   foreground gaps and heartbeat coalescing indicators. Paint only a linker-owned
   safe stack range, never the active stack/heap/retained record. SysTime cannot
   recover multiple collapsed ticks; counters are observations, not a complete
   worst-case timing proof.
4. Define one bounded, explicitly encoded, read-only diagnostics request/reply
   with schema and coherent snapshots. Existing packet IDs run through26 and
   Linux MAX27 is a sentinel: audit every dispatcher/array before allocating an
   ID. Preserve SysInfo UID/HW/version layout and the172-byte calibration API.
   Do not append binary fields to the version string. Reads are nondestructive.
   A CLI uses the sole Linux bridge owner, not a second UART open. Keep requests
   local, rate limited and outside vehicle-command routing.

Acceptance: malformed/partial record and stack-pointer fixtures; exact request
and reply bytes, maximum SLIP escaping and invalid-request side-effect checks;
actual exception entry disassembly; Renode fault/reset/retained-record scenarios;
all six board/profile builds with existing regressions and memory/asset checks.
Run canonical production and bench DVT Renode checks on preserved exact binaries.
Measure instrumentation costs before production enablement. Physical retention,
control/display timing and power-output behavior remain unproved by emulation.

## Wave B — correct the forwarding filter independently

Fix SoloLink `Telem.cpp` to compare only intended message IDs. Exercise the actual
forwarding path:120 excluded,134 forwarded,97 not excluded merely because it is
a payload-length constant; include truncation and concatenated frames. Preserve
the current MAVLink1 contract. Freeze and qualify the affected Linux candidate
through existing ARM/image checks. Do not fold this defect into a compiler change
or claim it is fixed by regenerated headers. Hardware installation is separate.

## Wave C — reproducible build and CMake/Ninja parity

First extend provenance: compiler/runtime package or archive identities,
selected multilib/newlib/nano/libgcc, host native compiler, Python/Pillow/FreeType,
input font/image/vendor/config hashes, full compile/link argv and source identity.
Reject or explicitly inventory untracked compilation inputs. Artifact export
must refuse collisions and retain previous receipts. Narrow helper cleanup to
known owned outputs; separate building from immutable artifact publication.

Then add `CMakeLists.txt`, presets and an explicit cross-toolchain file. Keep
GCC13.2, Pillow10.2, Tamp, C99/C++11, optimization/ABI flags, link driver/library
ordering and board/profile defines unchanged. Use Generic system, static-library
try-compiles, compile_commands.json and six isolated output directories for
production/factory/bench × DVT/BB03. Host compression/tests need native tools,
not the cross compiler. Generate version/resources in the build directory with
complete dependencies. Guard against stale `src/resources-gen.h` shadowing the
generated header through quoted includes.

Acceptance: full existing native suite and six profiles; compare binary/sections,
map/symbols, vectors, constructor order, version suffixes, factory exclusion and
all800 asset pixels. Explain any differences rather than silently accepting them.
Exercise repeated/incremental builds and input-change invalidation without source
pollution. Qualify exact production/bench DVT artifacts in Renode. After parity,
choose CMake as the single maintained graph; retain Tup in history/receipts.

## Wave D — individual dependency candidates

| Component | Research result | Decision and gate |
|---|---|---|
| Tamp | Local5279e7c sources match all seven relevant C/header files in release v2.5.0, peeled70706ba834d3601888b8292a87154a8ce4e97287. | No byte upgrade needed. Record equivalence without replacing qualified code. |
| Pillow |13.0.0 is listed in official release notes; current pin10.2.0. | Separate host environment, hashed wheels and FreeType identity; compare every glyph/image pixel and geometry, palettes and compressed size. Never regenerate expected baselines to hide differences. |
| Arm GNU |15.3.Rel2 is supported by official Arm support discussion; exact archive/checksum could not be resolved in this pass. | Candidate only, not an accepted pin. Obtain verified immutable archive before use; isolated prefix, complete matched runtime, no system compiler replacement. |
| CMSIS |6.4.0 current official release. No existing CMSIS dependency to bump. | Introduce selectively after layout/IRQ/intrinsic comparisons. |
| STM32CubeF1 |v1.8.7 tag peeled d12e75247d5bcedc734f829b394517ab4c2726e3. | Reference exact device/core gitlinks; do not consume floating component branches or assume newest CMSIS is automatically compatible. |

Compiler acceptance includes numeric behavior under existing fast-math flags,
unaligned checksum handling, atomics/ISR code, constructors, memory/stack output,
all profiles and Renode. Keep compiler, C++17, LTO, math flags and libc replacement
as separate candidates. Installed CMake3.28/Ninja1.11 suffice for the migration;
their upgrade is not a prerequisite. Optional `_printf_float` removal requires
actual call-site/size evidence and is not bundled with toolchain changes.

## Wave E — bounded initialization and peripheral failure policy

Clock waits need a pre-runtime polling budget and stage-specific failure record;
do not silently run at HSI while UART/SysTick/ADC still assume72MHz. Propagate ADC
failure through actual consumers, preserving invalid control state and reviewed
charge/balance behavior. Bound battery initialization without treating missing
samples as valid or changing button/charger semantics incidentally.

Flash needs more than a SysTime deadline. Review single-bank instruction/data
stall behavior and RAM-resident execution before claiming recovery from BSY.
Do not resume programming/cleanup on a still-busy controller. Preserve dirty
generation, relock, final completion and readback semantics. Separate detecting
failure from guaranteeing that code can execute during a physical flash stall.

Acceptance: actual producer/consumer failure propagation fixtures; independent
HSI/HSE/PLL/switch failure scenarios; ADC/no-sample/error tests; parameter failure
tests. Change Renode's existing expected clock-stall negative control to assert
the new deliberate failure state. Its ADC/flash models cannot establish physical
timeout correctness. No changes to option bytes or empirical battery thresholds.

## Wave F — modern embedded MAVLink generation, same wire contract

Keep one XML authority in product layer
`recipes-mavlink/mavlink/files/solo-dialect/{solo.xml,solo_common.xml}` and its
`generate-solo.py`. Consume a pinned export with generator/hash metadata in Artoo;
do not maintain another independent dialect. Use current owned PyMAVLink2.4.50,
explicit C/MAVLink1 output and SOURCE_DATE_EPOCH. Generate in a fresh directory
and select complete validated output; retire the broken regeneration helper.

Compare the actual2015 MCU definitions against the independent integration
`tests/data/solo-mavlink-v1-contract.json` before assuming equivalence. Review
IDs, CRC extras, wire lengths, offsets, enum values, producer and consumer includes.
Use named Solo includes; current standard EFI_STATUS225 and Solo GPS_ACCURACY225
must remain distinct dialects. Treat old unaligned-checksum patches as behavioral
requirements, not blind patches against newly generated internals.

Acceptance: golden frame parse/reserialization on native and Cortex-M3 paths;
actual MCU sender filtering/command/ACK/heartbeat/mission/expiry behavior; SLIP
fragmentation, escaping, malformed CRC and maximum payload. Preserve current
no-automatic-command-retry policy. Six profiles, memory/assets and canonical
Renode UART coverage remain required. Real telemetry/command acceptance is later.

MAVLink2 is a separate feature decision involving Linux routers/bridges and
vehicle/app peers. A signed maximum frame is280bytes: with host ID281 and worst
SLIP escaping564, versus current raw264/encoded530. Account for every buffer,
stack frame,24-bit ID, extension/truncation and any signing policy. A generator
refresh alone neither upgrades these paths nor fixes late same-command ACK
ambiguity. ROS2/RAS-A is not required for this MCU work.

## Later design gates

Selective CMSIS: compare owned `src/stm32/hw.h`, linker peripheral symbols and
IRQ indices against the selected high-density F103 definitions, then replace
one core/peripheral adapter at a time. Preserve owned startup, SysTick reference,
PRIMASK nesting, DMA channels/shared interrupts and clock ownership. Exact package
marking/silicon revision remains unresolved; ROM geometry is not the full marking.

Persistence: current page127 erase/rewrite is not power-loss atomic. Append-only
records improve ordinary writes but cannot make compaction atomic. A two-page
journal requires linker and loader/updater coordination: the loader currently
erases0–126 and preserves only127. Specify migration, generation/CRC/commit order,
legacy factory import, corrupt-record behavior and rollback before reserving126.
Test interruption at each erase/program/commit boundary. Preserve original
calibration evidence and preferably the external172-byte API.

Watchdog: retain as design-only until diagnostics, waits and physical permanent
power-hold boot/shutdown/reset behavior pass. Define reset-cause precedence:
current `Rcc::pinReset()` ignores watchdog flags in its mask. Feed from actual
foreground/subsystem progress, not SysTick alone. Cover startup, charger idle,
flash, shutdown and ROM transitions; add reset-loop policy without replaying
commands. An STM32 reset interacts with `PowerManager::init()` disabling i.MX
enable, so automatic recovery is not merely a timer setting.

## Execution and release rules

For each wave: settle related firmware/Linux/harness changes, commit and pin the
combined candidate, review actual runtime producers/consumers and record a dated
`.integration` source freeze before builds/tests. Source fixes restart the freeze.
Store exact ELF/BIN/map, build-input manifest, source and checker identities,
test logs and deltas from the previous accepted candidate. Keep factory/bench
artifacts distinct. BB03 compilation does not qualify a physical BB03 board.

Do not update the staged Yocto MCU payload or enable automatic flashing merely
because a local build passes. Physical controller currently runs an independently
identified bench artifact; deployment and later production promotion are separate
decisions. Preserve old receipts, original calibration and owner `build-env.md`.

## Primary research sources

- [Tamp releases](https://github.com/BrianPugh/tamp/releases): release/source comparison.
- [Pillow release notes](https://pillow.readthedocs.io/en/stable/releasenotes/index.html): current candidate.
- [CMSIS6.4.0](https://github.com/ARM-software/CMSIS_6/releases/tag/v6.4.0).
- [STM32CubeF1v1.8.7](https://github.com/STMicroelectronics/STM32CubeF1/tree/v1.8.7) and [F1 device component](https://github.com/STMicroelectronics/cmsis-device-f1).
- [Arm download guidance](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain) and [15.3.Rel2 support discussion](https://community.arm.com/forums/f/compilers-and-libraries-forum/57882/arm-gnu-toolchain-15-3-rel2-arm-none-eabi-gcc---version-reports-15-3-rel1/187076): archive identity remains unresolved.
- [CMake cross-compile try-compile behavior](https://cmake.org/cmake/help/latest/variable/CMAKE_TRY_COMPILE_TARGET_TYPE.html).
- [MAVLink generation](https://mavlink.io/en/getting_started/generate_libraries.html), [MAVLink2](https://mavlink.io/en/guide/mavlink_2.html) and [serialization](https://mavlink.io/en/guide/serialization.html).

Prior context: [implemented reliability waves](artoo-modernization-resolution-20261028.md),
[Linux protocol qualification](mavlink-modernization-20261029.md),
[physical MCU installation](artoo-flash-result-20261029.md),
[permanent power-hold candidate](controller-power-fix-20261029.md).
