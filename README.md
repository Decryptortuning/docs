# OpenSolo modern port project documentation

The OpenSolo port builds controller and drone images on Yocto 5 Scarthgap, Linux 6.6.52 and U-Boot 2024.04 in Ubuntu WSL2. Both development and release profiles pass offline image/initramfs/ARM checks, with repeatable packaging. See the [Scarthgap refactor result](scarthgap-refactor-20261028.md) for exact pins, 82 relevant passing tests and remaining hardware limits. The working controller card has not been replaced by this refactor.

Last updated: October 28, 2026. The current local refactor stays on Scarthgap; Yocto 6 and license work are deferred. [WSL relocation verification](wsl-relocation-20261028.md) remains historical evidence.

## Last offline-qualified baseline (not flashed)

Source9 graph `36f841398e89201e2e462391cfa2779188349999090295c8b91c967f29380138`
has four builds, sixteen image/initramfs/ARM checks, 82 relevant tests, matching
cached-rootfs packaging repeats and 34 independently restored Git stores.
This is not a clean full rebuild, comprehensive CVE review or hardware acceptance.
The physical controller still uses its older rootfs/overlay and no-MSI kernel.

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

Use [Build and validation](build-and-test.md) to resume work on this machine. Use [Project status and evidence](status-and-evidence.md) to identify the verified image before a hardware session.

## Local entry points

The active source/build workspace is `/home/jresp/solo`. Build wrappers, templates, tests and the source lock are in the [WSL recovery integration repository](/home/jresp/Active_GH/OPenSolo_2/README.md). This documentation checkout is `/home/jresp/solo/workspace/docs`; its configured Git origin is `https://github.com/Decryptortuning/docs.git`.

The [original board support audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md) records the recovered December 2025 snapshot. Consult the current guides for the status of findings addressed since that audit.

## Historical material

[Recovered build environment notes](build-env.md) retain the original local edits. Their active-layer list, setup procedure and build wrapper describe the earlier workspace; the current procedure is [Build and validation](build-and-test.md).

[WSL recovery notes before consolidation](history/wsl-recovery-20261008.md) preserve the former integration README as a dated snapshot. New project status and procedures belong in the guides above.
