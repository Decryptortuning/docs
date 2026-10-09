# OpenSolo project status and validation evidence

As of October 8, 2026, the controller builds successfully on this WSL host and passes the software checks below. The new drone target has passed metadata parsing; its image and media pipeline still need work. Current hardware behavior has not been established by these build or emulation results.

## Current validation coverage

| Area | Status | Evidence or practical limit |
|---|---|---|
| Controller build | Passed | All 7,039 tasks succeeded; 7,004 were already current. |
| Image payload and deployment | Passed | GOLDEN and the update archive match the deployed kernel, DTB, squashfs and U-Boot; versioned SD link and controller DHCP checks passed. |
| Deployed initramfs | Passed | `init` and `maintenance.sh` match committed source; machine identity is `imx_controller`; BusyBox matches the tested ARM binary. |
| Provisioning/update fault tests | Passed | 26 tests with simulated partition/format/mount commands and temporary regular-file raw writes. |
| ARM maintenance utilities | Passed | Production archive preflight, hash manifest, hardlink rejection, raw readback, unchanged bootloader skip and 511/512/513-byte boundary cases. |
| ARM controller userspace | Passed | DNSMasq syntax, Python 3.12.6, SSL/PySerial imports, SLIP roundtrip/timeout recovery and pairing configuration. |
| Earlier static startup review | Passed with follow-ups | All 126 startup links and inittab commands present; syntax checks passed for 82 Python and 64 shell scripts. Optional tools have unresolved imports; imported `usb.py` has the wrong shebang. |
| Drone metadata | Passed | Shared maintenance changes parse for `imx_drone`; no new drone image built in this recovery run. |
| Controller boot, DHCP leases and pairing | Pending hardware | Earlier recovered logs describe an older image and do not validate the current baseline. |
| Drone capture and ADV7610 reset/routing | Open | Source/effective-config audit found incompatible bridge selection and reset semantics. |
| Physical reset gesture and early board GPIO sequencing | Open | Legacy behavior was omitted during the port. |
| Clean rebuild and bit-for-bit reproducibility | Open | Successful incremental/cache-based builds do not establish a full clean rebuild; `/VERSION` still contains wall-clock build content. |

## Verified controller baseline

The source revision for `meta-3dr` is `a4aa5bc54758835216dfe57b33d1771bc208483b`. The BSP revision is `cfae891da533653a1a82d69499370d968934a035`. The preserved baseline has its own source lock; future working-lock updates do not alter that snapshot.

The verified deployment copy is [controller-maintenance-20261008](/home/jresp/solo/recovery/controller-maintenance-20261008/verification.json). It contains eight artifacts, the validation reports and build source record. Stable filenames select the copied versioned outputs.

| Artifact | SHA-256 |
|---|---|
| Controller SD image | `145a146175fa47982c91a606fead5c85e80ced0734b73d55db6de42a96422052` |
| Squashfs rootfs | `975681d178c7480f4f23da5d3d21192c72481660e46179e911e5b9155a31ebfa` |
| Update archive | `f07a53c485b7a28287a851b24430f9dedfec053b449d4ac345316444504c9b41` |

The SD filename is `3dr-controller-imx_controller.rootfs.sdcard`; the update archive is `3dr-controller.tar.gz` with its `.md5` sidecar. Full artifact hashes and versioned names are in the snapshot's `verification.json`. DTB and U-Boot hashes remain unchanged from the comparison baseline; the bundled kernel includes the new initramfs.

## Current reports

| Report | Contents |
|---|---|
| [Maintenance summary](/home/jresp/solo/.integration/controller-maintenance-summary.json) | Source revision, build log, test count and verified artifact copy. |
| [Image validation](/home/jresp/solo/.integration/controller-maintenance-validation.json) | SD image and producer payload hashes, DHCP and deployment checks. |
| [Initramfs validation](/home/jresp/solo/.integration/controller-initramfs-source-validation.json) | Embedded source and BusyBox hashes. |
| [ARM maintenance checks](/home/jresp/solo/.integration/controller-maintenance-arm-smoke.json) | Archive and library identity plus utility/readback results. |
| [ARM userspace checks](/home/jresp/solo/.integration/controller-userspace-smoke.json) | DNSMasq and Python/SLIP/configuration results. |
| [Earlier static startup review](/home/jresp/solo/.integration/controller-readiness-static.json) | Startup presence/syntax checks and optional-tool follow-ups. |
| [Earlier effective metadata check](/home/jresp/solo/.integration/effective-metadata-validation.json) | Both targets' source pins, 32-thread settings and controller-only DHCP scope at that earlier build. |

The final build log is [controller-20261008-174358.log](/home/jresp/solo/.integration/logs/controller-20261008-174358.log). The fault-test log is [controller-maintenance-tests.log](/home/jresp/solo/.integration/logs/controller-maintenance-tests.log). Copies of the current validation reports are preserved with the verified baseline because working report paths may be overwritten by later checks.

## Boundaries for using these results

ARM probes run userspace utilities with QEMU and a host shell harness; they do not boot the board kernel or test electrical signals, radios, STM32 communication or actual DHCP clients. Fault tests simulate filesystem maintenance and use regular files for raw writes.

The recovery behavior requires the new baseline's GOLDEN initramfs. An ordinary update leaves GOLDEN in place, so older cards need a deliberate migration or fresh flash before relying on it. FAT progress records and changed single-copy raw U-Boot writes retain power-loss risks. See [Boot and recovery](boot-and-recovery.md).

The archived drone SD image embeds a different rootfs from its archived deployed squashfs. Regenerate and validate a drone image before bring-up. Saved controller/drone images and boot logs represent different earlier development snapshots.
