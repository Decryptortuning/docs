# OpenSolo project status and validation evidence

As of October 28, 2026, both controller and drone images build and pass image/initramfs validation on this WSL host. The controller also passes 26 maintenance fault tests and ARM userspace/utility checks. The drone media pipeline and current physical board behavior remain unverified. See the [WSL relocation verification](wsl-relocation-20261028.md) for the latest build scope and source fix.

## Current validation coverage

| Area | Status | Evidence or practical limit |
|---|---|---|
| Controller build | Passed | Fresh rootfs/image regeneration passed all 7,039 tasks; 7,030 were current. Final-source recheck passed with all 7,039 current. |
| Image payload and deployment | Passed | GOLDEN and the update archive match the deployed kernel, DTB, squashfs and U-Boot; versioned SD link and controller DHCP checks passed. |
| Deployed initramfs | Passed | `init` and `maintenance.sh` match committed source; machine identity is `imx_controller`; BusyBox matches the tested ARM binary. |
| Provisioning/update fault tests | Passed | 26 tests with simulated partition/format/mount commands and temporary regular-file raw writes. |
| ARM maintenance utilities | Passed | Production archive preflight, hash manifest, hardlink rejection, raw readback, unchanged bootloader skip and 511/512/513-byte boundary cases. |
| ARM controller userspace | Passed | DNSMasq syntax, Python 3.12.6, SSL/PySerial imports, SLIP roundtrip/timeout recovery and pairing configuration. |
| Earlier static startup review | Passed with follow-ups | All 126 startup links and inittab commands present; syntax checks passed for 82 Python and 64 shell scripts. Optional tools have unresolved imports; imported `usb.py` has the wrong shebang. |
| Drone build and payloads | Passed | 7,026 tasks succeeded; 6,985 were current on the final build. SD/update payloads and embedded machine/source identity match; the KMS package and ARM plugin are present. |
| Controller SD preparation | Passed | The confirmed 7.95 GB card was flashed; all 197,133,312 written bytes passed SHA-256 readback. Backup skipped at the user's request. |
| Controller first boot and STM32 link | Observed | Current card provisioned and logs show sustained STM32 traffic; see filesystem qualification below. |
| Real DHCP leases and aircraft pairing | Pending hardware | Daemon startup/range configuration is observed; no client lease or aircraft pairing established. |
| Drone capture and ADV7610 reset/routing | Open | Source/effective-config audit found incompatible bridge selection and reset semantics. |
| Physical reset gesture and early board GPIO sequencing | Open | Legacy behavior was omitted during the port. |
| Clean rebuild and bit-for-bit reproducibility | Open | Successful incremental/cache-based builds do not establish a full clean rebuild; `/VERSION` still contains wall-clock build content. |

## Verified controller baseline

The current `meta-3dr` revision is `2f5bee6a16480dcea72e00d733d423f13d19ac82`; the BSP revision is `cfae891da533653a1a82d69499370d968934a035`. The controller artifacts were regenerated at `a4aa5bc54758835216dfe57b33d1771bc208483b` and verified unchanged by a successful build at the final product revision. The intervening change enables the requested drone KMS package and isolates its architecture. Each preserved artifact set has its exact build inputs and source lock.

The current verified controller copy is [wsl-relocation-20261028/controller](/home/jresp/solo/recovery/wsl-relocation-20261028/controller/verification.json). It contains eight artifacts, validation reports, original build inputs and the final-source recheck. Stable filenames select the copied versioned outputs. The [October 8 baseline](/home/jresp/solo/recovery/controller-maintenance-20261008/verification.json) remains preserved as earlier evidence.

| Artifact | SHA-256 |
|---|---|
| Controller SD image | `bc56f806b3d9f0241ce613d979a044385c24f3836946adf08816dccf40958632` |
| Squashfs rootfs | `47e21310db66908ef24c97b847d3d8a89513dd2410042e35b11e10c6564d8965` |
| Update archive | `b157c690103c6b901f2430fd2577ce713c38e7a309e8b4d7b78a93eba09378ce` |

The SD filename is `3dr-controller-imx_controller.rootfs.sdcard`; the update archive is `3dr-controller.tar.gz` with its `.md5` sidecar. Full artifact hashes and versioned names are in the snapshot's `verification.json`. DTB and U-Boot hashes remain unchanged from the comparison baseline; the bundled kernel/initramfs also remains unchanged from the October 8 maintenance build.

## Current reports

| Report | Contents |
|---|---|
| [Relocation summary](/home/jresp/solo/.integration/relocation-20261028/summary.json) | Source revision, build log, test count and verified artifact copy. |
| [Image validation](/home/jresp/solo/.integration/relocation-20261028/controller-image-validation.json) | SD image and producer payload hashes, DHCP and deployment checks. |
| [Initramfs validation](/home/jresp/solo/.integration/relocation-20261028/controller-initramfs-validation.json) | Embedded source and BusyBox hashes. |
| [ARM maintenance checks](/home/jresp/solo/.integration/relocation-20261028/controller-maintenance-arm-smoke.json) | Archive and library identity plus utility/readback results. |
| [ARM userspace checks](/home/jresp/solo/.integration/relocation-20261028/controller-userspace-smoke.json) | DNSMasq and Python/SLIP/configuration results. |
| [Drone image validation](/home/jresp/solo/.integration/relocation-20261028/drone-image-validation.json) | New SD/update payloads and deployment identity. |
| [Drone initramfs](/home/jresp/solo/.integration/relocation-20261028/drone-initramfs-validation.json) | Committed maintenance source and embedded drone identity. |
| [Drone KMS package](/home/jresp/solo/.integration/relocation-20261028/drone-kms-package-validation.json) | Installed package architecture and ARM plugin identity. |
| [Earlier static startup review](/home/jresp/solo/.integration/controller-readiness-static.json) | Startup presence/syntax checks and optional-tool follow-ups. |
| [Earlier effective metadata check](/home/jresp/solo/.integration/effective-metadata-validation.json) | Both targets' source pins, 32-thread settings and controller-only DHCP scope at that earlier build. |

The controller regeneration log is [controller-20261028-091908.log](/home/jresp/solo/.integration/logs/controller-20261028-091908.log); the final drone log is [drone-20261028-093540.log](/home/jresp/solo/.integration/logs/drone-20261028-093540.log). The [controller final-source recheck](/home/jresp/solo/.integration/relocation-20261028/controller-final-recheck.json) records the later successful controller build. The [fault-test log](/home/jresp/solo/.integration/relocation-20261028/maintenance-tests.log) records 26 passing tests. Reports and exact inputs are copied with the verified artifacts; working report paths may be overwritten by later checks.

## Controller card preparation on October 28

The confirmed controller card was flashed with the verified October 28 image. Readback of all 197,133,312 written bytes matched SHA-256 `bc56f806b3d9f0241ce613d979a044385c24f3836946adf08816dccf40958632` after flushing the block-device cache. The new partition table contains GOLDEN only, starting at 4 MiB with a 184 MiB partition; the initramfs will create LATEST/RWFS/LOG on first boot.

No original card backup was made, following the user's explicit instruction. The temporary WSL USB attachment/sharing was removed and the reader returned to Windows. The [flash receipt](/home/jresp/solo/recovery/controller-card-20261028-3136cfec/complete.json) and [readback evidence](/home/jresp/solo/recovery/controller-card-20261028-3136cfec/readback-verified.json) record the exact source, target identity and checks. First controller boot has now been observed; see the first-boot evidence below for results and remaining checks.

## First controller boot evidence — October 28

Read-only inspection after the observed “Waiting for Solo” boot confirms all four intended partitions and matching build payload hashes on both GOLDEN and LATEST. Configuration templates produced hostapd/wpa_supplicant/SoloLink settings; dnsmasq started with the intended DHCP range. Logs show sustained STM32 traffic, running firmware `OpenSolo-4dev`, explicitly disabled MCU updates, and a subsequent STM32-requested shutdown. The provisioning marker is gone and the update READY marker exists.

This is partial hardware acceptance. RWFS has `needs_recovery`; read-only e2fsck skipped journal recovery and reported free-block/inode allocation-count differences. Do not interpret its exit code alone as a clean result. LOG passed its read-only check. No repair, journal replay or card backup was performed. Other follow-ups are a redundant legacy golden-to-system script abort, fallback 2018 clock, HDMI-audio probe ordering and NFC daemon initialization. Real DHCP leases, aircraft pairing, joystick calibration, video/audio and a second boot from LATEST remain unverified.

[Inspection summary](/home/jresp/solo/recovery/controller-firstboot-20261028-4b853a54/summary.json) and logs are preserved with the read-only filesystem and payload reports. The reader was detached from WSL and temporary USB sharing removed.

## Boundaries for using these results

ARM probes run userspace utilities with QEMU and a host shell harness; they do not boot the board kernel or test electrical signals, radios, STM32 communication or actual DHCP clients. Fault tests simulate filesystem maintenance and use regular files for raw writes.

The recovery behavior requires the new baseline's GOLDEN initramfs. An ordinary update leaves GOLDEN in place, so older cards need a deliberate migration or fresh flash before relying on it. FAT progress records and changed single-copy raw U-Boot writes retain power-loss risks. See [Boot and recovery](boot-and-recovery.md).

The archived drone SD image embeds a different rootfs from its archived deployed squashfs. The new [October 28 drone snapshot](/home/jresp/solo/recovery/wsl-relocation-20261028/drone/verification.json) passes exact-producer validation; use it when assessing new artifacts. Saved older images/logs remain historical evidence. Successful new image construction does not close the drone capture/reset/routing work.
