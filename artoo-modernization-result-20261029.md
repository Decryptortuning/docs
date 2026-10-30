# Artoo modernization implementation and qualification

October 29, 2026 local time; receipts use October 30 UTC. The authorized local
implementation pass is complete. The selected software candidate uses CMake,
Arm GNU 15.3.Rel2, Pillow 13.0.0 and the owned PyMAVLink 2.4.50 C/MAVLink1 export.
Production and bench DVT each pass all 13 Renode scenarios. Nothing was flashed,
deployed, pushed or changed on the controller/router during this pass.

## Candidate identity

| Input | Qualified revision or identity |
|---|---|
| Artoo | `d3cc9379437ff8ca51667f2b723d40c297da970b` |
| SoloLink | `6d16834cea2a558a3560968bd10b28e78f08616d` |
| Product layer | `17da8f828c7f790135c4b0461e72c5accc8e1eff` |
| BSP | `2819766` (unchanged permanent power-hold fix) |
| Integration/checkers | `0ac58d5692efe7881d4ffcd620d8a5497a888a35` |
| Linux source graph, all four profiles | `e0dafae6a4e5a6eadbefba7f944944334b790246d237ac32fded33d543e6b4de` |

[Receipt directory](/home/jresp/solo/.integration/artoo-modernization-20261030T031656Z)
contains immutable freezes, failed attempts, exact manifests, logs and preserved
artifacts. `freeze8.json` binds final firmware/checker sources; Linux build inputs
have not changed since freeze5. Later MCU-only fixes do not relabel older tests.
Documentation commits following qualification are not firmware input revisions.

The physical controller remains Linux be98 plus the Chrony init overlay and
`5.0.0-dev+g48034572145e-bench-noidle`, installed in the earlier authorized session.
The new Linux images include the permanent power-hold BSP, but it remains
uninstalled. The Yocto-staged MCU payload was not replaced or enabled for update;
its byte authority is separate from these newly built MCU candidates. The source
lock's `build_input:false` Artoo entry remains historical provenance inventory.

## Implemented changes

**Diagnostics:** faults and unexpected interrupts now capture a checksummed,
64-byte retained RAM record and halt. The entry routine captures MSP/PSP before
the C prologue, checks frame bounds and switches to a dedicated 512-byte emergency
stack. Actual final compiler disassembly shows eight-byte capture frames calling
only terminal halt, without libc. Records commit validity last. They do not imply
retention across loss of power, automatically reset, or write flash. Existing
latched RCC reset flags are reused; backup-register retention is not assumed.

Bench builds add stack/heap high-water observations, OOM counts, task execution
counts/durations, foreground gaps and heartbeat coalescing. Timing values use the
actual nanosecond SysTime units. These are measurements, not complete worst-case
stack/timing proofs; production omits the bench instrumentation.

Host message 27 provides a versioned, explicitly encoded 204-byte diagnostic
reply. Linux uses its existing sole UART owner and exposes a bounded loopback
UDP interface with token validation, one pending request and rate limiting.
The controller-only `artoo-diagnostics` CLI reads it without opening the UART.
Existing version/UID/HW and 172-byte calibration interfaces remain unchanged.
Diagnostics alone do not activate firmware startup. Busy traffic can delay or
expire a diagnostic request; the interface does not guarantee service latency.

**Forwarding:** the Linux bridge now excludes only LOG_DATA message ID120.
The old comparison incorrectly also excluded IDs97 and134 by using payload-length
and CRC constants as message IDs. Actual bridge fixtures exercise forwarding,
framing, truncation and diagnostic request/reply handling on host and ARM Linux.

**Build organization:** CMake/Ninja is the sole maintained graph with six isolated
profiles, explicit source inventory and out-of-tree generation. Before selecting
it, the unchanged-toolchain comparison achieved exact Tup/CMake BIN, HEX and asset
parity at `9191566`. Tup remains in history. Manifests bind tools, matched runtimes,
Python/rendering dependencies, assets, compile/link commands, external headers and
source state. Unreviewed compilation inputs and incompatible reused caches are
rejected. Export destinations cannot overwrite prior evidence.

**Bounded startup:** pre-runtime HSI/HSE/PLL and clock-switch polls use finite
iteration budgets and stage-specific retained failure records. ADC/DMA/calibration
initialization returns typed failure stages through its actual callers. Initial
battery sampling has a five-second deadline. Failure halts rather than continuing
with invalid timing or control state. The poll budget is not a measured physical
deadline. Existing charge, shutdown and button policy is preserved.

**Embedded MAVLink:** firmware now consumes a verified immutable export from the
product-owned Solo XML, using the pinned 2.4.50 generator and explicit MAVLink1.
Owned generator patches preserve aligned checksum access and fix signed-byte
undefined behavior on CRC recovery. Standard EFI_STATUS225 and Solo
GPS_ACCURACY225 remain separate dialect definitions.

The old 2015 MCU definitions were compared independently. Reviewed differences
are confined to unused remote logging: message IDs141/142 become184/185 with
associated layouts/CRCs, MAV_COMP_ID_LOG50 becomes155, and four remote-log enums
are added. No current MCU application consumer uses those definitions. This is
not a claim that the entire old generated library is identical. All 166 current
Solo golden frames parse and reserialize byte for byte. MAVLink2 and automatic
command retries were not introduced.

## Dependency evaluation

| Comparison | Result |
|---|---|
| Pillow13 / FreeType2.14.3 vs Pillow10.2 / FreeType2.13.2 | All 800 assets and 405,717 RGB565 pixels identical; production/bench BIN and HEX identical with the same compiler. |
| Arm15.3.Rel2 vs GCC13.2 | Production DVT 191,108 vs192,788 bytes, saving1,680. Static RAM7,456 vs7,448 bytes. Largest individual compiled frame296 vs288 bytes. |
| Combined newer tools vs Arm-only | Production/bench BIN and HEX identical. All six combined profiles pass linked-image checks. |
| Tamp | Existing seven relevant C/header files already match2.5.0; no replacement needed. |
| CMSIS6.4 / CubeF1v1.8.7 | Evaluated as references; selective adoption remains a separate later design task. No HAL/RTOS replacement. |

Arm archive SHA256 is
`463df9665c246fa13bab2102115bbf83440661b6eb70b44f3dfd4712ca30037a`;
the pinned release branch revision is
`94aede75fefec048383b47f71b35f14a35db8338`. The archive matches the official published
checksum; its signature was not independently verified. The exact Python3.12
Pillow13 wheel SHA256 is
`42d63be3af4f9b951b199b0a9d23365d37ab14687e6d77b74940432c8515c4ca`.
Raw upstream metadata, downloads and installed identities are retained locally.

The selected candidate uses isolated newer tools. System compiler selection and
the original `.venv` are preserved, so an unconfigured wrapper still selects
baseline tools; this is not a global default-tool replacement. Use the receipt's
`recommended-tool-environment.sh` and a fresh `ARTOO_BUILD_ROOT` to reproduce the
selected variant. The historical Pillow10.2 environment remains the independent
raster oracle. Build commands never fetch dependencies or promote artifacts.

## Qualification and artifacts

| Scope | Final result and limits |
|---|---|
| MCU native suites | All12 pass: startup, diagnostics, transport, reliability, concurrency, runtime, graphics, MAVLink, calibration, idle, backlight and moving average. Host sanitizers are not Cortex-M3 execution. |
| Dependency/build matrix | 16 builds and16 linked-image checks pass: six baseline, two Pillow-only, two Arm-only, six combined. Includes vectors, constructors, memory bounds and factory/profile distinctions. |
| Final combined Renode | Production and bench DVT each pass13 scenarios: reset/startup, power wait, Running, five clock failures, poisoned-memory negative control, UART, Tamp rendering, actual MAVLink consumers and fault/retention. No input changes occurred during either run. |
| Protocol | 3,333 retained and3,335 new contract assertions pass; all166 golden frames and bad-CRC recovery pass. Independent ARM Linux wire suite passes all166 using the same export. |
| Actual consumer paths | Renode exercises ten UART/MAVLink cases including VFR_HUD, sender rejection, CRC rejection, fragmentation, concatenation, radio status and gimbal heartbeat. Command/ACK policy has native coverage, not full successful command lifecycle or flight acceptance. |
| Linux | Four incremental builds and16 image/initramfs/ARM/maintenance checks pass. Controller CLI source hashes match both installed roots and it is absent from both drone roots. |
| Cross-component bridge | Actual Yocto-built ARM Diagnostics/Telem/PacketHandler/SLIP fixture passes under qemu-arm with current rootfs, real headers, local UDP and pipes. No fake MAVLink constants or hardware UART. |
| Integration/SoloLink | Final108 integration tests and44 SoloLink tests pass; pinned workspace verification passes. |

Renode retains visible modeled/unimplemented-peripheral warnings. It does not
qualify electrical power holds, physical clock/ADC/DMA behavior, LCD timing,
UART electrical integrity, charger/shutdown behavior, pairing or flight. The
largest compiled frame is not a call-chain/interrupt stack bound. Numerical
behavior has regression and exercised target-path coverage, not exhaustive
equivalence under the retained fast-math options.

Final MCU artifacts are in `cmake8-combined/{production,bench,factory}-{dvt,bb03}`
under the receipt. DVT identities:

| Profile | BIN SHA256 |
|---|---|
| Production,191108 bytes | `a8868ddc850dc31f849b9f8514af40cbef78576787066935cfc6a4b60040f75a` |
| Bench,191488 bytes | `71289170097c21b1879a3dc878c63a4a082648fb07cfb03c59e3bc10970a184a` |

`linux-artifacts` preserves32 independent files, eight per profile, with source
paths and hashes. These are separate from mutable Yocto deploy symlinks.

| Linux SD profile | SHA256 |
|---|---|
| Controller development | `443cb9ccb3974e38e760d132bdf19a272a978c0be93694c3a05804f044520371` |
| Controller release | `c0d42d425fdc5c1f77372def48e4c8badb150fe038a207960cfbe450f5f612cc` |
| Drone development | `98cd6424211b4c613fe263e17e27be14768e41964eb7a41b0738ecea2aac12ec` |
| Drone release | `f8b03fcdf631f499e92947fdeeecb1e2ebcd6049f6d3eb8dca1d7593e0988cde` |

Earlier failed freezes remain evidence: signed timestamp comparison, an outdated
idle fixture, IRQ attributes, object-order parity, packed-checksum alignment,
signed CRC bytes and an obsolete ADC calibration stub were corrected before
new freezes and reruns. No failed result was rewritten into a pass.

## Remaining gates and next work

1. In a separately authorized hardware session, qualify the permanent i.MX
   power-hold lifecycle and updater startup behavior, then the exact new MCU
   artifact: application/ROM/reset return, calibration preservation, diagnostics,
   shutdown, controls, ADC and LCD behavior. The present pass changes no device.
2. Flash BSY recovery needs an executable RAM-resident path and a reviewed
   single-bank failure policy. Its existing wait remains unbounded; a SysTime
   deadline alone would not establish recoverability while flash stalls.
3. A two-page parameter journal needs coordinated linker/loader/updater changes,
   migration/rollback and interruption tests. Page127 preservation remains intact.
4. Active watchdog recovery requires reset-cause, power/output and reset-loop
   policy plus physical acceptance. It remains disabled/design-only.
5. Selective CMSIS adoption needs exact device/register/IRQ comparisons; it is
   not necessary to replace the working startup with a complete vendor HAL.

These are explicit deferred design or physical gates from the
[plan](artoo-modernization-plan-20261029.md), not silently completed work.
Yocto6, licensing, publication and automatic firmware promotion remain deferred.
Unrelated owner edits to `build-env.md` are preserved.
