# OpenSolo modern port project documentation

The recovered OpenSolo port builds controller and drone images on Yocto Scarthgap, Linux 6.6.52 and U-Boot 2024.04 in Ubuntu WSL2. The current controller baseline passes image validation, 26 maintenance fault tests and ARM userspace checks. Physical controller acceptance and the drone media port remain open.

Last updated: October 28, 2026. Both image targets were rebuilt and validated after the integration checkout moved to native WSL storage; see [WSL relocation verification](wsl-relocation-20261028.md).

## Project guides

| Guide | Purpose |
|---|---|
| [Project status and evidence](status-and-evidence.md) | Current controller baseline, artifact identity, validation coverage and remaining limits. |
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
