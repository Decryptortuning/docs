# OpenSolo modern port project documentation

The OpenSolo port builds controller and drone images on Yocto 5 Scarthgap, Linux 6.6.52 and U-Boot 2024.04 in native Ubuntu WSL2. The latest local candidate is [controller Linux maintenance](controller-linux-maintenance-20261029.md), graph `be98b385…`, with R01–R04 fixes on top of the MAVLink/Python updates. All four profiles pass their software checks; this candidate is now prepared on the controller SD, with first boot pending and publication deferred. Maintenance A resumed after the WSL restart and completed local software qualification on October 29, 2026: four builds, sixteen image/initramfs/userspace/maintenance checks, four expanded ARM runtime suites, matching cached-rootfs packaging repeats and independent offline source recovery. The [maintenance ledger](scarthgap-maintenance-a-20261028.md) records exact sources, results and limits.

The deployed controller graph is `7055fbe5738126848417be71d94a1e4fb6b83a7c507287527bce2ae2e3babf48`. The CVE task closures completed against the recorded current NVD snapshot, but the security review retains findings and coverage gaps. OpenSSL 3.5.9 follow-up, retained kernel/media updates and remaining applicability work are documented in the ledger. Passing software checks does not establish security clearance, a clean full rebuild or hardware acceptance. On October 29, the explicitly authorized controller SD card received the development image, verified readback and fresh offline Wi-Fi/SSH provisioning. Its first controller boot now passes Wi-Fi association, DHCP and authenticated SSH; STM32 firmware is unchanged. The short check includes matching 1 MiB memory-only SSH transfers in both directions and zero ath9k reset counters; reboot/long-soak acceptance remains open. New resume-fix commits are local and unpublished; the earlier pause checkpoint was pushed separately. Yocto 6 and license work remain deferred.

## Previous qualification and deployed controller baseline

Copies, validation reports and source/checker receipts are preserved in [software-qualification-7055fbe57381](/home/jresp/solo/recovery/scarthgap-maintenance-a-20261028/software-qualification-7055fbe57381). Ordinary build/deploy links select newer outputs when rebuilt; use these copies when identifying this candidate.

| Profile | Preserved SD SHA-256 |
|---|---|
| Controller Development | `d5f8c4c6e4fde132852081d06cb0b4212cea94622e4515d31771337ea20ec2ec` |
| Controller Release | `994d67c5cfec9a1d1c93e9fbcfd3b82e80b9ec46660ff036971b798cc06b6489` |
| Drone Development | `524cd3d8ed1c68dd8bf80c3489027b8271060ae776079c0345e9b2667ce81291` |
| Drone Release | `d1bf4b579b889c94de5eaa849184343ba096a6fc76414e5e249701afd8d7b686` |

## Historical source9 baseline (not flashed)

Source9 graph `36f841398e89201e2e462391cfa2779188349999090295c8b91c967f29380138`
has four builds, sixteen image/initramfs/ARM checks, 82 relevant tests, matching
cached-rootfs packaging repeats and 34 independently restored Git stores.
This is not a clean full rebuild, comprehensive CVE review or hardware acceptance.
Those software results did not install a new physical-controller baseline.

| Profile | Preserved SD SHA-256 |
|---|---|
| Controller development | `a9c23f81afa8a1d412b08d791d055601d74b831a925f69e2ea2249f197cb0660` |
| Controller release | `0d57c450cac9165ba34ad913b14870176194ebc8776ad4ce6b12e0fffb5861f3` |
| Drone development | `f95be255ba4bb63da5908d6bea2ba9e385b98b9becdc1be5bd01a422c6926e1e` |
| Drone release | `b106bb4e2cc2d29a3bdf851c1a8d932d1a4bbd1ce7855dad4497cb0a951491f7` |

[Source9 result](scarthgap-refactor-20261028.md), [implementation plan](scarthgap-consolidated-improvement-plan-20261028.md)
and [current maintenance ledger](scarthgap-maintenance-a-20261028.md) are separate authorities.
OpenSolo names the project; OPenSolo_2 is its integration repository. meta-solo-bsp
is active; meta-solo is inactive inventory. `build_input:false` lock entries are
unchecked provenance inventory, including docs, scripts and Artoo source; the
selected staged Artoo artifact has its own byte authority.

The preceding [MAVLink modernization candidate](mavlink-modernization-20261029.md), graph `28fd826a52ca4accf0743f3ef3492ae2fe87e608bcb9adc8f4509abc635db244`, now passes four builds, sixteen baseline checks, four ARM runtime suites and four C wire suites. Its preserved artifacts are local and unflashed. The [ROS2/RAS-A investigation](ros2-rasa-investigation-20261029.md) records a separate offboard-first research path. The new local candidate and the deployed controller baseline have separate artifact identities.

## Project guides

| Guide | Purpose |
|---|---|
| [Scarthgap refactor result](scarthgap-refactor-20261028.md) | Current source organization, profile/security policy, four-image qualification and packaging/source-recovery evidence. |
| [Project status and evidence](status-and-evidence.md) | Current controller baseline, artifact identity, validation coverage and remaining limits. |
| [Wrynose migration plan](yocto-wrynose-migration-plan.md) | Concrete kas workspace, ordered implementation waves, acceptance gates and rollback for the NXP Wrynose port. |
| [Scarthgap package maintenance audit](scarthgap-package-update-audit-20261028.md) | Complete historical image inventory, verified maintenance candidates and next-round package update priorities without moving to Yocto 6. |
| [Yocto modernization audit](yocto-modernization-audit-20261028.md) | Astra review of current layers/BSP, source organization, release options and a gated Wrynose migration plan. |
| [Workspace and source ownership](workspace-and-sources.md) | Repository locations, layer responsibilities, source pins and where changes belong. |
| [Build and validation](build-and-test.md) | Repeatable WSL build commands, CPU settings, input verification and checks. |
| [Board support](board-support.md) | Machine definitions, hardware format migration and unresolved controller/drone BSP work. |
| [Controller Linux maintenance](controller-linux-maintenance-20261029.md) | Latest local R01–R04 fixes, four-profile software qualification and remaining physical gates. |
| [Live controller Linux audit](controller-optimization-audit-20261028.md#october-29-fresh-image-runtime-audit) | October29 fresh-image measurements, logger/rotation/time/console/watchdog findings and the next bounded improvement batch. |
| [Python 3 compatibility](python3-compatibility-20261028.md) | Reviewed source scope, updater safety fixes, Python 3.12 regression coverage and deployment limits. |
| [Controller Wi-Fi kernel fix](controller-wifi-pci-20261028.md) | PCIe interrupt root cause, permanent kernel policy, module compatibility and reboot acceptance. |
| [Artoo bench idle option](artoo-bench-idle-20261028.md) | Local idle-only development profile, all-six builds and normal/bench Renode qualification. |
| [Artoo STM32 firmware](artoo-stm32.md) | MCU responsibilities, source/tooling, build profiles and software/hardware acceptance. |
| [Artoo asset compression](artoo-compression-20261028.md) | Heatshrink/Tamp results on all images/fonts, Cortex-M3 checks and projected flash savings. |
| [Tamp implementation](artoo-tamp-integration-20261028.md) | Integrated image/font compression, measured flash/RAM costs and native/Renode qualification. |
| [Artoo Renode software tests](artoo-renode.md) | Repeatable reset, memory/constructor, startup task and Running-state checks using a small F103 test setup. |
| [Boot and recovery](boot-and-recovery.md) | Disk layout, provisioning, updates, reset behavior and recovery limits. |
| [Project roadmap](roadmap.md) | Next software work, hardware milestones and acceptance criteria. |
| [Decisions and build history](decisions-and-history.md) | Integration decisions, completed milestones and how to maintain these guides. |

For current maintenance results and follow-up work, start with the [maintenance ledger](scarthgap-maintenance-a-20261028.md). The [pause checkpoint](scarthgap-maintenance-a-pause-20261028.md) preserves the earlier stop. Use [Build and validation](build-and-test.md) for general commands. Use [Project status and evidence](status-and-evidence.md) to identify the verified image before a hardware session.

## Local entry points

The active source/build workspace is `/home/jresp/solo`. Build wrappers, templates, tests and the source lock are in the [WSL recovery integration repository](/home/jresp/Active_GH/OPenSolo_2/README.md). This documentation checkout is `/home/jresp/solo/workspace/docs`; its configured Git origin is `https://github.com/Decryptortuning/docs.git`.

The [original board support audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md) records the recovered December 2025 snapshot. Consult the current guides for the status of findings addressed since that audit.

## Historical material

[Recovered build environment notes](build-env.md) retain the original local edits. Their active-layer list, setup procedure and build wrapper describe the earlier workspace; the current procedure is [Build and validation](build-and-test.md).

[WSL recovery notes before consolidation](history/wsl-recovery-20261008.md) preserve the former integration README as a dated snapshot. New project status and procedures belong in the guides above.

## October29 controller Linux maintenance — latest local candidate

The [controller Linux maintenance result](controller-linux-maintenance-20261029.md) supersedes28fd as the latest locally qualified software candidate. Graph `be98b385025c2eff8569d94ab732079ac332ddacfd139c960140f660c5dc7888` includes the completed MAVLink/Python updates plus R01 explicit nonblocking logger ownership, R02 effective BusyBox rotation, R03 bounded network-time/flight GPS ownership and R04 UART0 console-session repair. Four builds, sixteen baseline checks, four expanded ARM suites, four C wire suites and four new Linux suites pass. Preserved artifacts and source/checker lineage are linked from that result.

The physical controller remains7055 with OpenSolo-4dev; no card, controller/router settings or STM32 changes occurred. Controller development now needs a188MiB GOLDEN allocation (deployed card168MiB), so verify layout/capacity before a later authorized installation. Watchdog, memory/stack tuning, real-network clock/runlevel behavior, reboot/loaded pairing/video/flight acceptance and existing security findings remain open. Work is locally committed and unpublished; Yocto6 and license work remain deferred.

## Controller card ready for be98 boot

The explicitly approved card now contains the latest development image and fresh enlarged layout, with verified readback, both boot payloads, four filesystem checks and offline Wi-Fi/SSH provisioning. It is unmounted and detached from WSL; STM32 is unchanged and no backup was made. The [maintenance ledger](scarthgap-maintenance-a-20261028.md#october29-be98-controller-card-prepared--boot-pending) records the new operation. Last observed running7055 and prepared-cardbe98 remain separate until boot is verified.
