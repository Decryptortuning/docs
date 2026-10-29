# OpenSolo project status and validation evidence

The current local candidate is [Maintenance A](scarthgap-maintenance-a-20261028.md), graph `7055fbe5738126848417be71d94a1e4fb6b83a7c507287527bce2ae2e3babf48`. All four controller/drone development/release builds pass sixteen baseline image/initramfs/userspace/maintenance checks and four expanded ARM runtime suites. Packaging repeats match byte for byte, the independent SD-only cache regression passes, and a fresh 34-store restore plus both normal offline source-fetch closures and Pixhawk checksum controls pass. Exact copies are in [software-qualification-7055fbe57381](/home/jresp/solo/recovery/scarthgap-maintenance-a-20261028/software-qualification-7055fbe57381).

The CVE reports retain unresolved findings and mapping limits; the ledger distinguishes them from software test results. This is not a clean full rebuild, security clearance or hardware acceptance. One final ARM certificate fixture initially rejected a certificate after a backward WSL clock step; an unchanged strict retry passed, and both attempts are retained. After qualification, the user explicitly authorized the identified 7.95 GB controller card: the development image was written and all 180,356,096 bytes verified by cache-invalidated readback. Production provisioning functions created fresh GOLDEN/LATEST/RWFS/LOG, both boot slots and raw U-Boot match the qualified payloads, and all four read-only filesystem checks passed. Fresh station-only LAN/admin settings and an on-card SSH identity are provisioned; the workstation pins its public host key with strict checking. The first physical boot confirms the expected graph, `pci=nomsi`, Wi-Fi/DHCP at `192.168.2.207`, authenticated SSH through the normal alias, matching 1 MiB memory-only transfers each way and zero ath9k reset counters. The serial bridge opens `/dev/ttymxc1` and identifies hardware3 / Artoo `OpenSolo-4dev`. Reboot, longer soak and broader serial behavior remain unqualified. No STM32 firmware was flashed and no card backup was taken.

The [card receipt](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/card-complete.json) records the exact target and result. Offline provisioning changes the partition table and settings after the initial full-image readback; the final card is not claimed to retain the source SD image's whole-file hash. It also means this session has not tested the controller's blank-card first-boot provisioning path. No old overlay programs or settings were restored.

The subsequent [October29 live Linux audit](controller-optimization-audit-20261028.md#october-29-fresh-image-runtime-audit) confirms active dual-stack LAN filtering, working real-time bridge threads and useful standby CPU/RAM/thermal headroom. It identifies a proven custom-logger/glibc fortify boundary defect, ineffective requested rotation substitutions, absent external clock synchronization, console session ownership errors and an inactive hardware watchdog. GPU pool and thread-stack reductions require loaded measurements. The user confirmed idle power-down after an orderly DHCP release at14:39:40 UTC; the last live ath9k reset counters are zero. The read-only audit changed no live configuration and does not qualify paired/video behavior.

[Source9](scarthgap-refactor-20261028.md) and the relocation tables below are preserved historical baselines. Use the maintenance ledger for current sources/artifacts, and the Wi-Fi/serial reports for observed physical-controller behavior.

Current October 28 Wi-Fi/Artoo acceptance: [permanent PCIe kernel fix and two
warm reboot results](controller-wifi-pci-20261028.md), plus [local bench/normal
Artoo qualification](artoo-bench-idle-20261028.md). The kernel is installed;
STM32 firmware remains unflashed. Earlier image receipts below retain their
original scope and identity.


As of October 28, 2026, both controller and drone images build and pass image/initramfs validation on this WSL host. The controller also passes 26 maintenance fault tests and ARM userspace/utility checks. The drone media pipeline and current physical board behavior remain unverified. See the [WSL relocation verification](wsl-relocation-20261028.md) for that historical build scope and source fix.

## Earlier relocation validation coverage

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

## Historical verified controller baseline

At the preserved relocation baseline, the `meta-3dr` revision was `2f5bee6a16480dcea72e00d733d423f13d19ac82`; the BSP revision is `cfae891da533653a1a82d69499370d968934a035`. The controller artifacts were regenerated at `a4aa5bc54758835216dfe57b33d1771bc208483b` and verified unchanged by a successful build at the final product revision. The intervening change enables the requested drone KMS package and isolates its architecture. Each preserved artifact set has its exact build inputs and source lock.

The preserved relocation controller copy is [wsl-relocation-20261028/controller](/home/jresp/solo/recovery/wsl-relocation-20261028/controller/verification.json). It contains eight artifacts, validation reports, original build inputs and the final-source recheck. Stable filenames select the copied versioned outputs. The [October 8 baseline](/home/jresp/solo/recovery/controller-maintenance-20261008/verification.json) remains preserved as earlier evidence.

| Artifact | SHA-256 |
|---|---|
| Controller SD image | `bc56f806b3d9f0241ce613d979a044385c24f3836946adf08816dccf40958632` |
| Squashfs rootfs | `47e21310db66908ef24c97b847d3d8a89513dd2410042e35b11e10c6564d8965` |
| Update archive | `b157c690103c6b901f2430fd2577ce713c38e7a309e8b4d7b78a93eba09378ce` |

The SD filename is `3dr-controller-imx_controller.rootfs.sdcard`; the update archive is `3dr-controller.tar.gz` with its `.md5` sidecar. Full artifact hashes and versioned names are in the snapshot's `verification.json`. DTB and U-Boot hashes remain unchanged from the comparison baseline; the bundled kernel/initramfs also remains unchanged from the October 8 maintenance build.

## Earlier relocation reports

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

## Subsequent LAN management setup

The development card now has a local LAN Wi-Fi/SSH profile; see [build and validation](build-and-test.md#local-controller-lan-and-ssh-profile). During configuration, normal journal replay and clean unmount resolved the earlier RWFS check differences; a read-only e2fsck then passed. The earlier first-boot journal observation is preserved above as historical evidence. Actual LAN association and SSH login remain pending a boot check.

The first LAN boot reached “Waiting for Solo” and started sshd on port 22, but
OpenWrt recorded only brief association, with no WPA completion or DHCP lease.
The prepared PSK matches the router. AP auto-channel startup overlapped station
connection attempts; a channel conflict is suspected, not yet established. At
the owner’s request, the card now disables the SoloLink AP while retaining LAN
station mode and controller services, and enables WPA syslog for the next boot.
Selected logs and results: `/home/jresp/solo/.integration/controller-lan/station-only-20261028`.
No full card backup was taken. Station-only boot and SSH verification are pending.

The station-only boot completed WPA key negotiation on the controller and router,
then repeatedly reported beacon loss and locally initiated reason-4 disconnects.
Subsequent `WRONG_KEY` warnings followed beacon loss; they do not establish a
credential mismatch. No DHCP lease or SSH session was obtained. Removing the AP
therefore did not resolve the failure. Wi-Fi power save is disabled by default
in the current ath9k driver; no speculative power-save change was installed.

The development card now has a temporary `options ath9k use_msi=1` override to
test PCI interrupt delivery, plus a bounded interface-up diagnostic hook writing
interrupt counts, MSI vector allocation, station statistics and link/IP state to
`/log/wifi-bringup.log`. The kernel and driver support this option. This is a
diagnostic hypothesis, not a proven fix; both files can be removed to restore
legacy interrupt mode. Credentials and radio timing settings remain unchanged.
RWFS passed its check after clean unmount. Evidence and rollback paths are in
`/home/jresp/solo/.integration/controller-lan/msi-test-20261028/result.json`.
The MSI boot, DHCP and SSH checks remain pending.

The MSI test boot now has verified DHCP and pinned-key SSH access at
`192.168.2.207`; the workstation hostname alias resolves and connects as well.
MSI is active on IRQ 295 and interrupt counters increment. Gateway and DNS are
`192.168.2.1`, station mode is active, and Wi-Fi power save is off. Recurring
beacon loss and reconnects still interrupt SSH, so stable LAN acceptance remains
open. Signal improved to approximately -31 dBm; this confounds attributing the
connectivity improvement solely to MSI. Live diagnostics and result are saved
under `/home/jresp/solo/.integration/controller-lan/msi-boot-success-20261028`.
