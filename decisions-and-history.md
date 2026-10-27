# OpenSolo integration decisions and build history

The recovery keeps the original project layout and NXP Scarthgap baseline while making source selection, image identity and software recovery behavior explicit. This preserves a usable comparison point for the remaining BSP and application work.

## Current decisions

| Decision | Reason and consequence |
|---|---|
| Keep source/builds/caches under native WSL storage | Yocto work stays on `/home/jresp/solo`; the Windows integration checkout supplies small templates and commands. |
| Keep separate app repositories and two active custom layers | `meta-3dr` owns product policy; `meta-solo-bsp` owns boards. A wholesale monorepo conversion is unnecessary for the current recovery. |
| Retain the pinned NXP vendor layer set | Complete the port against a known Linux 6.6.52/U-Boot 2024.04 baseline before independently changing upstream versions. |
| Use an exact source lock for the baseline | Recipe pins and clean vendor/layer checks prevent accidental source drift. Local app development requires a separate explicit profile. |
| Remove ineffective recovered masks/local overrides | The masks matched no recipes and externalsrc was inactive. Required provider removal needs a coherent dependency change, not only corrected escaping. |
| Stage custom image outputs under `IMGDEPLOYDIR` | Yocto owns the SD/update outputs and links; the squashfs must be the exact producer output. |
| Preserve the existing complete-image cache policy | This release skips `do_image_complete` cache creation. Regeneration evidence concerns reused package state and rebuilt final images. |
| Preserve compatible native tar in the wrapper | Pinned pseudo cannot track the host tar's direct `openat2` calls; the native tar path/hash is checked and recorded. |
| Use progress records and verify maintenance completion | Resume only known provisioning intentions; keep UPDATE after installation failures. Avoid claiming power-loss atomicity for FAT or raw U-Boot. |
| Keep docs in the existing docs Git repository | Central procedures/status live here; the integration README is an entry point and audit findings retain their historical context. |

## Recovery milestones on October 8 2026

| Milestone | Build result | Preserved evidence |
|---|---|---|
| Controller comparison baseline | 7,039 tasks passed; 6,935 already current | `/home/jresp/solo/recovery/new-baseline-20261008` |
| Controller DHCP and staged image output | 7,039 tasks passed; 7,012 already current | `/home/jresp/solo/recovery/controller-fixed-20261008` |
| Cleanup and image regeneration | 7,039 tasks passed; 7,028 already current | `.integration/controller-regenerated-validation.json` |
| Initial provisioning failure guards | 7,039 tasks passed; 7,004 already current | `.integration/controller-provisioning-guards-validation.json` |
| Restartable provisioning and verified updates | 7,039 tasks passed; 7,004 already current; 26 fault tests and ARM checks passed | `/home/jresp/solo/recovery/controller-maintenance-20261008` |

The maintenance work required actual ARM utility probes: minimal BusyBox lacks the advanced `od`/`head` options assumed by host tools. The final implementation uses supported header/readback operations and enables only required `dd` conversion/flush support. This is why source syntax or host-only tests are insufficient evidence for target maintenance code.

The original board audit's controller DHCP, image staging, input selection, layer organization and provisioning/update software findings have been addressed to the recorded software-test level. Physical behavior, drone media, an explicit development profile, recipe/package cleanup and broader rollback/reproducibility work remain in the [roadmap](roadmap.md).

## Documentation ownership

| Change in the project | Update these records |
|---|---|
| New accepted image or hardware result | `status-and-evidence.md`, the relevant roadmap milestone and preserved artifact/report set. |
| Build command, host workaround, source/profile behavior or configuration policy | `build-and-test.md` and `workspace-and-sources.md`; integration scripts/templates and lock as applicable. |
| Device tree, kernel/U-Boot or board sequencing finding | `board-support.md` and its roadmap acceptance criterion. |
| Provisioning/update/reset policy | `boot-and-recovery.md`, owning source/tests and relevant validation status. |
| Change in integration policy | This decision record with its reason and concrete effect. |

Date validation results and preserve artifact identity. Keep build/emulation evidence distinct from physical observations. Update the current guide when behavior changes, and preserve earlier evidence as dated history rather than leaving competing active procedures.

The [original audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md), [recovered build notes](build-env.md) and [former recovery README](history/wsl-recovery-20261008.md) provide historical context. New material belongs in the current guides; the recovered `build-env.md` retains its existing local edits.
