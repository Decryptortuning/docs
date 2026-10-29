# Controller optimization and reliability audit — 2026-10-28

**Current follow-up:** the [October 29 fresh-image runtime audit](#october-29-fresh-image-runtime-audit) below records the deployed Maintenance A image. The October 28 observations are historical; several of their defects are now fixed.

This is a read-only assessment of the current Scarthgap controller port. The best next work is to correct several integration defects and make the development/flight profiles explicit, then measure memory and media behavior. There is no evidence supporting a blanket performance governor, a PREEMPT_RT conversion, aggressive VM tuning, or removing media support now.

**The auditing agent implemented no changes.** No services were restarted, no device was flashed, no router configuration was changed, and no build, stress test, benchmark, reboot, card backup, UART open, GPIO change, or package installation was performed. The only new persistent local artifact is this report. Existing source, generated build inputs, lock pins and the dirty `build-env.md` were preserved.

Subsequent local work is tracked in [the Python 3 compatibility report](python3-compatibility-20261028.md) and [the Artoo development notes](artoo-stm32.md). Some source-level findings have since been corrected locally; those changes have not been deployed to this controller. The observations below describe the audited running image.

## Scope and evidence quality

Instructions were read from `/home/jresp/Active_GH/AGENTS.md` and `/home/jresp/solo/AGENTS.md`; no additional AGENTS.md was found in the inspected integration, docs, layer or application trees. This report belongs to the canonical docs checkout, currently `codex/project-docs`.

The audit combines source inspection, existing build outputs, historical saved evidence, and short serialized SSH reads using the pinned `opensolo-controller` host alias. One later SSH connection timed out; a direct-IP override retaining the alias and host-key policy succeeded. No scanning or active radio tests were performed. SSH reads generate a small amount of normal management traffic and authentication logging.

Live observations below cover roughly uptime 244–509 seconds of the same observed boot; boot ID was `fe152c7a-ecb6-4e00-9861-e4002dad22d0` at the checks that read it. Target wall time was in March 2018 and is unsuitable for correlating events against workstation time. “Confirmed” means demonstrated in these observations or the effective image/source; it does not imply a completed hardware acceptance test. “Candidate” means a proposed change requiring measurement. Historical first-boot delays are not current normal-boot measurements.

Key local evidence:

- [Current status](status-and-evidence.md), [build procedure](build-and-test.md), [board support](board-support.md), [boot/recovery](boot-and-recovery.md), and [source ownership](workspace-and-sources.md).
- [Integration source lock](/home/jresp/Active_GH/OPenSolo_2/workspace-lock.json).
- [Effective controller image metadata](/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller/3dr-controller-imx_controller.rootfs-20261028161926.testdata.json) and [997-package manifest](/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller/3dr-controller-imx_controller.rootfs-20261028161926.manifest).
- [Effective kernel configuration](/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/work-shared/imx_controller/kernel-build-artifacts/.config). These generated files were inspected, not edited.
- [Saved MSI connection result](/home/jresp/solo/.integration/controller-lan/msi-boot-success-20261028/result.json) and its bounded `wifi-bringup.log`. Private LAN staging must not be published wholesale.
- [Historical first-boot evidence](/home/jresp/solo/recovery/controller-firstboot-20261028-4b853a54/summary.json), with adjacent boot/kernel logs and payload verification.

## Measured baseline and important limits

| Area | Observed baseline | Interpretation |
|---|---|---|
| Product state | `3dr_controller`, SysV runlevel 3, display reported Waiting for Solo | Standby only; runlevel 4 video, telemetry and pairing workloads are unqualified here. |
| Kernel | `6.6.52-lts-next-ge0f9e2afd4cf`, ARMv7, SMP/PREEMPT build, `nosmp` command line | One active CPU is expected on this i.MX6 Solo; multicore affinity tricks offer no benefit. |
| CPU | `ondemand`; supported sampled frequencies 396/792/996 MHz; time-in-state 30369/830/2539 ticks | About 90% of that sample was at 396 MHz. A single 996 MHz read during SSH is not an idle-power problem. |
| Thermal | 50.5°C at first sample; exposed trip values 85/90°C | No demonstrated overheating. Battery/enclosure/video thermal behavior remains unmeasured. |
| Memory | 498108 KiB MemTotal; 183584 KiB MemAvailable at uptime ~340s; no swap | Roughly 486 MiB visible and 179 MiB available in standby. Do not infer flight headroom. |
| CMA | 327680 KiB total, 93056 KiB free | DT reserves 320 MiB; its free portion is not all permanently wasted RAM. Attribute consumers before resizing. |
| GPU pool | 134217728 bytes total, 18448 bytes used, no listed clients in standby | A measured 128 MiB GPU reservation with negligible standby use is a strong optimization candidate after media qualification. |
| STM32 Linux process | PID 522, seven threads; VmSize 54740 KiB, VmLck 54728 KiB, RSS 52864 KiB | Significant locked memory. Up/down threads actually use FIFO priority 60/56, matching source. |
| Scheduling | PREEMPT, HZ=100, high-resolution timers, NO_HZ_IDLE; RT budget 950000/1000000 μs | Existing low-latency mechanisms are active. No reason yet to remove RT throttling or change priorities. |
| Storage | SquashFS lower, ext3 RWFS and LOG, overlay root; RWFS ~88 MiB filesystem with ~82 MiB available; LOG ~6.75 GiB filesystem | Current usage is tiny. Mount-path visibility and bounded write policy matter more than capacity. |
| I/O policy | `mq-deadline`, 128 KiB readahead, dirty background/foreground ratios 10/20 | No I/O bottleneck demonstrated. Ratios alone do not justify tuning. |
| Wi-Fi | AR9300/ath9k, station only, channel 11/2462 MHz, signal about −31 dBm, MSI IRQ295 | AP is intentionally disabled. This is not the normal flight network profile. |
| Radio stability | Beacon-loss count 28 at uptime ~245s; repeated locally generated reason-4 disconnects later; one SSH timeout | Strong signal and MSI have not produced stable connectivity. |
| Radio settings | `use_msi=1`, `ps_enable=0`, `nohwcrypt=0`, power save off, PCI runtime control `on`, ASPM `default` | Existing settings already exclude several often-suggested “fixes”; do not reapply them or infer effective ASPM state from policy alone. |
| RF survey | At ~509s, channel 11 busy 114339/466795 ms (~24.5%), noise −93 dBm | Cumulative counters, not a controlled interval. Other channels have only ~78–88 ms dwell samples; these do not rank alternate channels reliably. |
| Entropy | Current boot `crng init done` at 11.09s; entropy counter later 256 | Historical first-boot CRNG completion at 73s is not the current result. No basis for weakening entropy requirements. |
| Firewall | IPv4 filter table unavailable; effective config lacks IP_NF_FILTER, NF_TABLES and IPv6 iptables; NAT table exists | Installing iptables and enabling NAT did not provide a host filtering policy. |

`/sys/class/watchdog/watchdog0/*` and `/proc/PID/task/TID/sched` were unavailable because WATCHDOG_SYSFS and SCHED_DEBUG are disabled. This is not proof the watchdog or scheduler is absent: watchdog device nodes exist, and scheduler fields were read from task `stat`. No watchdog device was opened. BusyBox `ps` lacks the requested full procps options; plain `ps` and `/proc` supplied the replacement observations.

## Priority and approval matrix

Priority is the recommended order, not permission to execute. Impact is qualitative because no before/after experiments were authorized. Risk is implementation/qualification risk.

| ID | Recommendation | Evidence/confidence | Expected impact | Risk and prerequisites |
|---|---|---|---|---|
| A01 | Move psplash FIFO storage away from `/mnt` | Confirmed/high | Restore boot/lower/upper mount visibility and maintenance assumptions | Low–medium; fix mount lifecycle, validate cold boot and recovery paths. |
| A02 | Correct timestamp format and persistence policy together | Confirmed/high | Reliable time floor, less overlay churn | Low–medium; test invalid RTC, power loss and time-source jumps. |
| A03 | Correct decimal `iw` parsing and AP/station readiness | Confirmed parser defect/high | Avoid incorrect ACS fallback in concurrent mode | Medium; radio/profile tests and channel-concurrency checks required. |
| A04 | Add explicit standby/development/flight service policy | Confirmed profile mismatch/high | Remove pairing respawn noise and unused dev services | Medium; preserve STM32 and runlevel transitions. |
| A05 | Diagnose remaining beacon loss with controlled evidence | Confirmed symptom/high; cause unknown | Highest current availability benefit | Medium–high for experiments; approval and recovery path before any setting change. |
| A06 | Define host filtering and interface exposure with required kernel support | Confirmed gap/high | Reduce unintended LAN exposure | Medium–high; retain management, pairing, telemetry and update access. |
| A07 | Replace image-wide shared access credentials with device provisioning | Confirmed source policy/high | Security and maintainable recovery access | High; migrate updater/drone identities together and retain recovery. |
| A08 | Connect product syslog routing to the active daemon and bound all logs | Confirmed image wiring/high | Better evidence and predictable SD writes | Low–medium; test all facilities, long uptime and disk-full behavior. |
| A09 | Qualify HDMI detection and GStreamer elements before media trimming | Confirmed obsolete path/high; plugin compatibility incomplete | Restore a runlevel-4 prerequisite | Medium–high; HDMI display/decoder fixtures required. |
| A10 | Attribute and right-size GPU/CMA allocation | Measured standby/high; proposed size unknown | Potentially large RAM headroom | High; video/HDMI, fragmentation and restart acceptance required. |
| A11 | Measure STM32 stack usage and bound thread stacks | Measured locked footprint/high; savings uncertain | Potentially tens of MiB | High; preserve mlock and FIFO behavior, prove worst-case stacks. |
| A12 | Remove inherited unsupported features and packages in tiers | Confirmed package inheritance/high | Smaller image, fewer services, simpler maintenance | Medium; package/dependency and full role acceptance. |
| A13 | Replace unconditional missing-radio reboot with bounded recovery policy | Confirmed code/high | Prevent persistent reboot loops and preserve diagnostics | Medium; no-radio and transient-probe tests. |
| A14 | Reconcile obsolete userspace GOLDEN migration/cleanup scripts | Confirmed source/high | One authoritative recovery path | Medium–high; GOLDEN/LATEST/reset regression coverage. |
| A15 | Resolve USB gadget ownership and LOG export semantics | Confirmed config mismatch/high | Functional, safe USB maintenance | High; never expose an actively writable LOG filesystem as writable raw storage. |
| A16 | Qualify watchdog, shutdown and button recovery end to end | Known port gaps/high | Predictable fault handling and safe power-down | High; dedicated bench approval, no autonomous reboot/flash. |
| A17 | Establish authenticated releases, patch review and runtime-overlay inventory | Confirmed process gaps/high | Security, reproducibility and recoverability | Medium–high; coordinated release/recovery design. |
| A18 | Tune CPU, VM, kernel size or boot parallelism only from measured deficits | No demonstrated deficit/high | Avoid regressions; possible later power/latency gain | Medium–high; meaningful workload baseline first. |

## Findings and proposed changes

### A01 — psplash obscures the retained root mounts

The initramfs deliberately moves the boot, lower and upper mounts into `/mnt/boot`, `/mnt/rootfs.ro` and `/mnt/rootfs.rw`. The effective `/etc/default/rcS` sets `PSPLASH_FIFO_DIR=/mnt`; `S00psplash.sh` mounts a 40 KiB tmpfs there when `/mnt` is not itself a mountpoint. The existing child mounts do not make the parent a mountpoint. Live `/proc/mounts` contains both the original mounts and the overlaid `/mnt` tmpfs; `df` reports the three child paths missing.

Use a dedicated FIFO location with explicit early-boot lifecycle, without covering retained mounts. Simply naming `/run/psplash` is not sufficient unless `/run` is mounted before the splash is started; a later `/run` mount must not hide that FIFO. Check configinit's lower-root access, diagnostics, clean shutdown and the remaining maintenance consumers after the change. The configinit fallback masks part of this issue; successful config generation does not disprove it.

Owners: [initramfs init](/home/jresp/solo/workspace/layers/meta-3dr/recipes-core/initrdscripts/files/init:249), product initscripts/psplash policy. Do not patch the generated rootfs.

### A02 — timestamp producer and consumer disagree

[clock_sync](/home/jresp/solo/apps/sololink/init/clock_sync:25) writes `YYYYMMDDHHMM` every ten seconds. The effective modern `bootmisc.sh` expects `YYYYMMDDHHMMSS`, compares it numerically with a 14-digit current date, then slices seconds/year accordingly. Live `/etc/timestamp` is 12 digits. A 12-digit saved date cannot provide the intended floor against a 14-digit RTC-derived date; a complete power-loss/reboot reproduction was not performed.

The generated image starts with `20180309123456`, despite the image recipe attempting to write build time: Poky's later `rootfs_update_timestamp` uses the reproducible timestamp. Treat this as two distinct policies: reproducible artifact content and trustworthy runtime time. Correct the saved format, validate contents, retain atomic replacement and define the required durability. Ten-second replacements mean 8640 timestamp updates/day of continuous operation, even though the stored value changes only once/minute. A lower-frequency checkpoint plus orderly-shutdown update is a candidate; power-loss tolerance determines the interval.

The init wrapper has no stop implementation, PID ownership or duplicate-start guard. Give the clock process a managed lifecycle. For online development, nominate one network time owner; for offline flight, document the RTC/GPS/peer time authority. Do not let wall-clock adjustments drive control deadlines; STM32's core timing already uses CLOCK_MONOTONIC. Any future trusted-clock correction must be coordinated with logging, certificates and telemetry behavior.

### A03–A05 — network parsing, profiles and radio stability

[iw.py:56](/home/jresp/solo/apps/sololink/net/usr/bin/iw.py:56) searches for integer frequency followed immediately by whitespace. Installed iw 6.7 prints `freq: 2462.0`; that cannot match. [hostapdconfig.py](/home/jresp/solo/apps/sololink/net/usr/bin/hostapdconfig.py:53) turns the resulting None into channel 0/ACS. This explains a concrete route to the previous automatic-channel fallback; it does not prove the parser caused station-only beacon loss. Accept current frequency formats with explicit units/ranges, and distinguish disconnected/not-ready from malformed output. Add fixtures for old integer output, current decimal output, no connection and invalid text when implementation is approved.

The shared PHY must satisfy its actual interface-combination/channel limits. AP startup should wait for the chosen station channel when concurrent operation is requested; a one-second sleep is not a readiness protocol. Avoid silently launching ACS after association/parser failure. Station-only, flight AP-only and deliberate AP+station should be named policies, with idempotent startup/stop and visible readiness failures. Preserve system hostname `3dr_controller`; DHCP's `opensolo-controller` name is a separate concern.

The inittab always respawns `pair_server.py` in levels 3/4/5. Pairing requires hostapd, which the current development profile intentionally disables. Gate pairing and AP-specific DHCP/forwarding on the profile or make unavailable dependencies a bounded, quiet wait. Keep STM32 communication and power-button handling enabled. DNSMasq wildcard sockets alone do not prove it answers on every interface: the controller configuration specifies `interface=wlan0-ap`, and DNSMasq can filter after wildcard binding. Verify interface policy before alleging a rogue DHCP/DNS server.

Current beacon loss survives station-only operation, strong signal, MSI and disabled power saving. MSI materially improved the observed connection sequence, but prior INTx measurements also had a different signal level. Do not promote `use_msi=1` to a production conclusion from this comparison alone. Current logs show repeated reason 4 with `locally_generated=1`; this identifies the local disconnect path, not the ultimate RF/driver/PCI cause.

Next read-only evidence should correlate boot ID/uptime, association epochs, cumulative IRQ deltas, station beacon-loss/retry/failure deltas, channel survey deltas, CPU/softirq pressure and kernel MAC/PCI messages. Avoid interpreting counters across reassociation resets. Existing ath9k debugfs counters are unavailable in this build; if the ordinary evidence is insufficient, consider narrowly enabling driver diagnostics in a later candidate. Do not run continuous scans while measuring the connection.

Only after approval, use one-variable experiments with fixed AP configuration, position, power source and workload: MSI versus INTx; driver/radio reset behavior; channel width or channel; and only then PCI power-management or hardware-crypto hypotheses supported by evidence. Keep the current RF settings until then. `iw phy0 set distance 1800` in the interface file is inherited long-range ACK timing, not an obviously erroneous value; any indoor/flight profile separation needs loss/range measurements. Do not increase transmit power or override the regulatory database to mask the fault. Global US and per-PHY country 99 were both reported; per-device EEPROM/regulatory restrictions need to be respected.

The AP post-up script appends an unscoped MASQUERADE rule every bring-up and enables IPv4 forwarding without symmetric removal. Make any required forwarding explicit, interface-scoped and idempotent. First establish whether the intended production controller should route between networks at all; its DHCP policy deliberately does not advertise gateway/DNS to clients.

### A06–A07 — access policy is incomplete outside SSH

The running card has dedicated key authentication and a password-disabled SSH drop-in, as established by the saved connection evidence. The image recipe still installs a fixed legacy root password and shared authorized public keys. No credential values are reproduced here. Reflashing/resetting can therefore restore a different access policy from this card. Model access as device provisioning and migrate any legacy updater/drone users deliberately, rather than silently deleting their keys during optimization.

Live listeners include SSH22, rpcbind111, DNS53, controller TCP5016/5021 and several UDP control sockets. [TcpServer.cpp](/home/jresp/solo/apps/sololink/flightcode/stm32/TcpServer.cpp:255) binds INADDR_ANY. With the controller on the home LAN, flight-side endpoints require explicit scope. The effective kernel has NAT but no IPv4 filter table, no nftables and no IPv6 iptables; a read-only `iptables -S` query fails accordingly. Define the full IPv4/IPv6 and interface policy, add its kernel capabilities, then validate reachability before applying restrictive defaults. Disabling IPv6 or stripping all network services is not a substitute for that design.

`debug-tweaks`, writable root overlay, debugfs, serial getty, root-run applications and package management suit bring-up but need an explicit release policy. Stack protector and seccomp are present; FORTIFY_SOURCE, HARDENED_USERCOPY and the main SECURITY option are absent. Treat additional hardening as a measured candidate compatible with the vendor drivers, not a promise of a secure image. Review exposed command/control protocols and root privileges before connecting a production profile to untrusted networks.

### A08 — logging configuration is installed but not selected

The generated `/etc/syslog.conf` contains only a comment. Product routing is in `/etc/syslog.conf.busybox`, but the active daemon command has no `-f` selecting it and the files are not linked. BusyBox has configuration-file support enabled. Live logs are consequently concentrated in `/log/messages`, while expected STM32/temperature facility files are absent from the listing. Preserve evidence while connecting the intended configuration to its actual consumer.

The default messages log is bounded at approximately 10 MB plus one generation. Product logs are rotated at boot with twenty generations; that alone does not bound a long-running session. `proc_top` appends/flushed data every ten seconds, `log_temp` logs every minute, and runlevel-4 Wi-Fi telemetry has its own cadence. The card's diagnostic hook is bounded to 36 samples per invocation but appends to a file across boots. Specify per-file budgets, continuous rotation, rate limiting for repeated failures, and a reserved space budget for update staging. Retain fault transitions and last-known health; do not globally suppress the messages needed to solve beacon loss.

At ~509s, disk counters showed 1414 sectors written to RWFS and 5064 to LOG since boot. These include startup, ordinary services and management logging; they are not a steady-state endurance benchmark. Measure deltas over a stable interval after failure loops are fixed. Keep journaling and required update flushes; `noatime`, longer writeback intervals, RAM-only logs or filesystem changes need an explicit durability tradeoff and power-loss tests.

### A09–A11 — media and RAM should be qualified together

Runlevel 4 starts `hdmiout`, `app_streamer`, telemetry and tlog. Standby neither exercises their memory nor proves they work. [hdmiout.c:111](/home/jresp/solo/apps/sololink/flightcode/video/hdmi/hdmiout.c:111) reads an obsolete `/sys/devices/soc0/soc.1/.../cable_state` path; that path is absent live, while `/sys/devices/platform/soc/20e0000.hdmi_video/cable_state` exists. Its failure branch assumes a cable is connected. Use a stable device-discovery/property owner rather than replacing one brittle path string with another unchecked constant.

The same application requests `vpudec` and `mfw_isink`. Their presence/properties and successful pipeline creation under the installed GStreamer 1.x stack were not verified; compile/link success is insufficient. Inventory elements and caps in a later authorized media check, then validate pipeline error handling, clean restart, hotplug, resolution changes and network loss. A registry-building `gst-inspect` was not run on target because it can create caches and the present scope was read-only. HDMI-audio probe ordering errors remain live; determine whether the supported controller HDMI experience needs audio before disabling the node or merely suppressing messages. The MCU's own display is not evidence that Linux HDMI video/audio works.

The kernel inherits 320 MiB CMA from NXP `imx6dl.dtsi`, overriding the apparent 16 MiB Kconfig default. Its source DT declares 1 GiB RAM while U-Boot's board definition is 512 MiB and the live kernel reports a 512 MiB physical memory basis. Correct the description against the actual board and verify the final handed-off DT; do not infer an actual current out-of-bounds allocation from the source mismatch alone.

Live GPU debugfs shows a 128 MiB system pool with only 18448 bytes used in standby. The module's exposed `contiguousSize=4294967295` is a sentinel, not a 4 GiB allocation; the measured pool and vendor fallback logic are authoritative. Attribute all CMA use, including framebuffer, GPU, VPU and movable memory, then test a smaller controller-specific GPU pool and CMA budget. Do not select a final number from this idle sample. Require video decode/output, buffer peaks, reconnection, memory pressure and long-run fragmentation coverage. Removing the GPU entirely before checking its relation to the chosen sink is high risk.

STM32 locks almost its full ~53 MiB address space; seven threads and default pthread stacks make stack reservation a plausible contributor, not a measured allocation breakdown. Inspect maps/stack high-water and failure paths before reducing thread stack sizes. Preserve prefaulting, mlock and actual FIFO 60/56 priorities. The current mlock helper ignores its return value; add error reporting and a defined degraded/failure behavior rather than silently assuming memory is locked. Parent audit reports healthy current serial packet flow, but no worst-case timing or calibration qualification was performed.

### A12 — image and kernel trimming in dependency order

The effective machine inherits `bluetooth`, `bcm4339`, `bcm43455`, `nxpwifi-all-sdio`, touchscreen and other SABRE traits. Effective distro features include 3g, nfc, nfs and zeroconf. The older `3dr.conf` feature-removal list is not the active `3dr-imx-fb` policy. The manifest includes 997 packages, including 338 kernel-module packages, 65 python3-prefixed packages and 103 gstreamer-prefixed packages.

| Tier | Candidate | Evidence and dependency guard |
|---|---|---|
| 1 | ofono/3g, neard/nfc, rpcbind/NFS, unused Bluetooth and unrelated WLAN firmware | ofonod/rpcbind run live; neard historically fails because the NFC family is absent. Confirm no supported cellular/Bluetooth/NFS accessory or maintenance contract. Preserve ath9k, regdb, SDMA and VPU firmware actually used. |
| 2 | RPM alongside DEB/apt/dpkg; pip and multiple editors/help/locales | Build uses `package_deb`, yet explicitly installs RPM. Choose a supported update model; do not remove recovery tools or a documented maintenance workflow by assumption. |
| 3 | Python modules/numpy, GStreamer subplugins, generic util-linux tools | Replace blanket package sets only after import/runtime coverage, pipeline element inventory and update/recovery dependency checks. No direct numpy import found in the scoped SoloLink search does not prove its full dependency closure is unnecessary. |
| 4 | Unused kernel drivers/debug facilities and excess vendor layers | Verify hardware/accessory scope and merged effective config. Built-in Bluetooth/PCI endpoint/demo drivers cost differently from unused modules. Fewer layers mainly improve maintenance/parse scope, not runtime performance. |

Logical uncompressed directory sizes in the generated rootfs are approximately: Python 95.2 MiB, Vim data 36.5 MiB, firmware 17.5 MiB, modules 13.0 MiB, GStreamer plugins 7.6 MiB. These are not compressed image savings and exclude libraries elsewhere. The dpkg status data did not provide usable Installed-Size values. Compare rebuilt manifest, compressed image size and live memory separately when changes are authorized. Rsync has an actual dataflash-log transfer consumer and should not be removed as an apparently unused utility.

Retain native Cortex-A9/NEON hard-float tuning already selected. Avoid broad `-Ofast`, unsafe floating-point transformations or whole-system LTO changes without application/ABI qualification. Do not switch init systems solely for startup speed; these SysV runlevels encode product state and must be preserved or migrated as a complete contract.

### A13–A16 — boot, recovery, USB and power are product contracts

[checknet](/home/jresp/solo/apps/sololink/config/checknet:24) invokes reboot if wlan0 is absent and then sleeps 60s. That legacy workaround can loop indefinitely on a persistent hardware/configuration fault. Replace it with bounded retries only where the driver supports safe recovery, then a diagnosable maintenance state. A time-limited recovery policy must not restart a functioning controller in a flight-critical state merely because management networking disappeared.

The current initramfs already owns validated GOLDEN→LATEST seeding. The still-enabled `golden_to_system.sh` checks old DTB names and tries to construct an older update path; on current GOLDEN it can fail to select a filename. Its `S100` name sorts before `S12`, not after S99. Remove or reconcile this redundant mechanism after verifying all recovery entry paths. `cleanLibs.sh` assumes an old RWFS directory layout rather than overlay `upper`, and `/mnt` is additionally hidden. Do not repair this by introducing a broad library-deletion step; explicitly define any real upgrade migration and preserve user data.

The image requests `g_acm_ms file=/dev/mmcblk0p4` in `/etc/modules`, but effective CONFIG_USB_G_ACM_MS is disabled. This mismatch is confirmed; USB export is not demonstrated. Enabling the missing module blindly would expose the same LOG block device mounted locally read-write. Design exclusive ownership, an offline snapshot/export, or a file-oriented transfer instead. Linux's [mass-storage gadget documentation](https://docs.kernel.org/usb/mass-storage.html) explains why backing storage must not be concurrently modified by other processes. USB serial/network maintenance is a separate possible feature and still needs role, power, security and host compatibility qualification.

The watchdog driver/device exists, but source only starts the user watchdog in the reboot path (`S80startwd`); this does not establish a runtime health watchdog. Define which failures justify recovery, who owns feeding, the required timeout and how fail-safe behavior coordinates with the MCU and vehicle. Never probe health by opening `/dev/watchdog` casually. GPIO power-off code, the MCU's hard timeout and filesystem shutdown ordering must be verified as one sequence. Parent read-only work confirms the currently used UART and loader GPIO bank mapping, but not safe reset/power sequencing.

The physical three-second factory-reset gesture is still absent from the rebased U-Boot path. Boot delay is zero. U-Boot tries GOLDEN if LATEST loading/bootm returns, but this is not rollback after Linux starts and fails. Both slots share RWFS; a bad overlay can affect both. Raw U-Boot remains a single-copy update risk. Keep these limitations visible before optimizing away diagnostic access or claiming an A/B update system. No card backup is required or requested by this audit.

**MCU update gate from the parent's concurrent read-only/source audit:** updater subprocess output uses text operations on bytes after loader execution; the updater ignores the final unprotect-attempt result and can report completion after failures; the loader can print verification failure without failing its exit status. The updater also archives a candidate when updates are disabled. Correct output handling, failure propagation, verification, candidate disposition and calibration preservation together before any future flash approval. Current UART communication with installed OpenSolo-4dev is not authorization or proof for ROM-loader programming. The current `/firmware` directory has no Artoo candidate and update/force flags are absent, according to the parent's live snapshot.

### A17–A18 — build organization, maintenance and restrained tuning

All 10 project and 17 vendor repository HEADs inspected match their recorded lock entries. Tracked dirty state is limited to the preserved scripts/docs inventories among those entries; application and active layer checkouts are clean. Eight generated recipe-source overrides match the lock. The historical `meta-solo` layer is omitted from active BBLAYERS. These are useful controls; retain them.

The lock describes fetched image inputs, not every current card-overlay change. Track a sanitized runtime manifest for the station profile, SSH policy, MSI diagnostic override, hook and config checksums separately from secrets. Never commit WLAN credentials. Test configinit preservation through approved updates/reset because its checksum/base/merge scheme can overwrite unmanaged modifications.

The SoloLink recipe modifies `${S}/config/sololink` during installation; remove that source-tree mutation before adding an externalsrc/development workflow. BSP/product initramfs coupling and overlapping generic appends should be consolidated by ownership. Extra vendor/demo/ML/Matter/Qt/virtualization layers can be reduced after dependency review; merely appearing in BBLAYERS does not prove their software is installed.

The documented build wrapper and pin guards are preferable to ad hoc vendor setup. Host parallelism/pressure settings already reflect native WSL CPU/memory limits; no new build tuning is supported by this target audit. `/VERSION` embeds wall-clock build time, so pin equality alone is not bit-for-bit reproducibility. Define release identity from immutable inputs and preserve artifact hashes/validation receipts; compare two authorized reproducibility builds only after sources settle.

The legacy MD5 archive sidecar and SHA-256 payload/readback checks support integrity checks, not authorization of a release. Plan authenticated manifests, trusted signing keys, key rotation and explicit downgrade policy together with recovery. Existing fault tests are valuable but do not establish authentication or physical power-loss acceptance.

Produce and review an image SBOM/CVE assessment against the pinned recipes and actual backported patches. Version numbers alone cannot establish vulnerability status. Yocto's [Scarthgap vulnerability workflow](https://docs.yoctoproject.org/5.0.12/dev-manual/vulnerabilities.html) provides recipe/image CVE reports; evaluate mappings and exceptions rather than claiming every version match is exploitable. No CVE scan/build was run here and no patch-status assurance is asserted.

Keep ondemand and the existing thermal safeguards until a latency or throughput deficit is measured. CPU frequency is a power/performance tradeoff, as described in the [kernel CPUFreq documentation](https://docs.kernel.org/admin-guide/pm/cpufreq.html); a permanent maximum frequency can cost battery/temperature without improving the observed standby workload. Similarly, do not disable RT throttling, raise HZ, add PREEMPT_RT, enable SD swap, drop caches, force IRQ affinity, disable journaling, or enlarge buffers indiscriminately. Any queue/buffer change must be judged by latency and loss, not throughput alone.

## Staged validation and evidence still needed

1. **Approve a scoped source-fix batch first:** A01 mount visibility, A02 clock format/lifecycle, A03 parser/readiness, A04 profile/service gating and A08 log routing are the clearest integration fixes. Review actual callers, ordering and product-state transitions. Preserve the current card until a reviewed candidate and an authorized deployment plan exist.
2. **Validate offline:** test parser variants; boot-script behavior with retained mounts; timestamp format, invalid values and time-floor cases; configinit upgrade preservation; profile-driven startup; logging routes/rotation; and maintenance regressions. Re-run the established image, initramfs and ARM userspace checks for a deliberately pinned candidate. Do not substitute syntax checks for these behaviors.
3. **Authorize a bench deployment/reboot separately:** verify exact artifact identity and sanitized overlay state. Record monotonic boot milestones: kernel start, root switch, wlan0 creation, WPA completion, DHCP, SSH availability, STM32 readiness and runlevel readiness. Compare normal boots separately from first boot, key generation and filesystem repair. No serial capture was initiated in this audit.
4. **Radio acceptance before general tuning:** collect a stable idle observation, then a representative management/control/video workload under an approved bench setup. Proposed initial gate is zero unexpected reconnects over at least 30 minutes, plus packet-loss/latency and recovery behavior accepted against the control requirements. This is a proposed engineering gate, not a completed test or a flight certification.
5. **Memory/media batch:** establish working HDMI and the actual decoder/sink; record MemAvailable, CMA free/peak allocations, GPU pool peaks, STM32 stack/locked-memory use, major faults and allocation failures. Compare candidate pool/stack reductions one at a time; retain margin for resolution changes, packet bursts and restart. Require no memory failures and no degraded control/video latency.
6. **Service/package/security batch:** remove Tier-1 unsupported features, then optional tooling. Compare installed package graphs, image sizes, open sockets, background CPU/writes and every supported accessory/profile. Apply interface restrictions only with a verified recovery path and both management and flight-path tests. Security provisioning and update authentication deserve coordinated design, not a cleanup side effect.
7. **Power/recovery acceptance:** with explicit hardware approval, validate button shutdown, filesystem clean state, watchdog ownership, lost-radio handling, bad-LATEST fallback, shared-overlay failure behavior and physical reset. MCU flashing remains separately gated by the updater defects and calibration preservation.

For each experiment retain input hashes, profile, boot ID, uptime range, position/AP conditions, baseline/candidate measurements, failures and rollback outcome. Minimum performance metrics are control-loop/packet interval p50/p95/p99/max and loss, video startup/restart latency and drops, time to usable controller, memory/CMA/GPU peaks, temperature/frequency residency, write-sector deltas and battery consumption where measurable. A faster boot is not an improvement if control readiness, recovery or diagnostics regress.

Still unmeasured: representative paired/flight runlevel behavior; HDMI element availability and successful pipeline; long-term radio stability; control timing under video/CPU/I/O load; battery impact; stack high-water; worst-case CMA fragmentation; effective PCI link power states; runtime watchdog ownership; filesystem durability under power loss; physical reset gesture; fully authenticated update/recovery design; current CVE/backport status; and fresh-image access policy acceptance. Those gaps limit tuning confidence but do not weaken the concrete source/runtime defects above.

## Report preservation

The pre-existing `build-env.md` remained untouched; its inspected SHA-256 was `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`. This standalone report is uncommitted. No source lock was advanced. Recommendations require user approval before implementation on source/build inputs or the controller.

## October 29 fresh-image runtime audit

The controller is functioning well in standby, but there are concrete correctness
and reliability fixes left. This pass inspected the actual newly installed image,
not the older overlay used for the October 28 audit. It changed no controller or
router configuration, services, kernel settings, clock, firmware or hardware.
There was no stress test, UART open, watchdog open, reboot or package install.
Workstation-only evidence and documentation were written.

### Identity and observation limits

- Running graph: `7055fbe5738126848417be71d94a1e4fb6b83a7c507287527bce2ae2e3babf48`, OpenSolo 5 development.
- Kernel: `6.6.52-lts-next-ge0f9e2afd4cf`, LATEST boot, `pci=nomsi`, one online CPU.
- Boot ID: `f8c09a1a-018c-4fb6-b193-3bf1494c57dd`; saved live captures include uptimes 356.8–594.1 seconds of that same boot.
- Source heads remain integration `6adb5e6`, product `ad893d0`, BSP `1602ee5`, SoloLink `cd3ff3c`; no application/layer sources or lock inputs changed.
- Evidence: [audit gate and captures](/home/jresp/solo/.integration/controller-linux-audit-20261029T143240Z). Sensitive filesystem contents, Wi-Fi passwords and private keys were not collected.
- Target wall time is about nineteen hours behind the workstation/router. Correlate boot ID and uptime; target log timestamps are not trustworthy UTC.

OpenWrt recorded one association at 14:28:39 UTC and an orderly DHCP release plus
disconnect at 14:39:40 UTC. The user confirmed the controller powered down on its
own. This fits the existing idle shutdown; the last live sample has zero ath9k
reset counters. Do not classify this disconnect as a reproduced driver hang.
The blank-card provisioning path, repeated reboot/long soak, runlevel 4, pairing,
video, audio accessories and flight remain outside this audit.

### Healthy or deliberately retained behavior

| Area | Actual observation | Assessment |
|---|---|---|
| CPU and thermal | Ondemand, 396/792/996 MHz; sampled residency 32,031/779/2,569 ticks, about 90.5% at the lowest frequency; 47.8–50.0°C during SSH reads. | Keep current governor and thermal controls. No measured CPU bottleneck or reason for permanent maximum frequency. |
| Scheduling | PREEMPT, HZ100, seven bridge threads; upstream/downstream are actually SCHED_FIFO priorities60/56. | Existing real-time policy is active. No priority/RT-budget change is justified by standby reads. |
| RAM | 498,108 KiB visible; about179.6 MiB available at the main capture; no swap or observed OOM. | Useful standby headroom; paired/video peaks are still unknown. |
| Wi-Fi | Station-only, about−24 to−29 dBm in network captures, power save off, normal hardware crypto, legacy IRQs increasing, zero resets/timeouts at last read. | Previous MSI regression is absent in this boot. No channel, crypto or PCI change is warranted here. |
| Access | Existing key authenticates; root and sololog have unusable password markers, no empty password; sshd requires publickey, password/keyboard-interactive off. | Fresh provisioning is effective. sololog retains its forced receiver command and DisableForwarding; root forwarding is an administrator capability, not evidence of an authentication bypass. |
| Firewall | Both IPv4 and IPv6 have the wlan0 management chain with established traffic, SSH, DHCP, ICMP and mDNS allowed, then DROP. Forwarding is off. | The older missing-filter-support finding is fixed. Bridge TCP5016/5021 listeners bind all IPv4 addresses, but inspected wlan0 rules block unsolicited access; no negative connection test or flight-path acceptance was run. |
| Storage | Boot and SquashFS are read-only; RWFS about84.4 MiB free; LOG about6.4 GiB free; mq-deadline/128 KiB readahead. | Current capacity/use is healthy. SquashFS showing100% is expected for the immutable image. No swap, scheduler or journal change is indicated. |
| Service policy | Runlevel3; AP-disabled pairing wrapper waits quietly. Video/app/telemetry services assigned to runlevel4 are inactive. | Expected standby behavior, not missing daemons. Hostapd's unprovisioned-AP message is expected for this station-only card. |
| i.MX–MCU | Only the running bridge owns `/dev/ttymxc1`; RX rises457,045→795,456 bytes and TX1,291→2,239 between captures. Regular packet summaries continue. | Actual two-way communication is present. Artoo reports hardware3 / OpenSolo-4dev; no ROM/programming test was performed. |
| Entropy | CRNG initialized at about11.2s and later entropy count256. | The earlier jitterentropy initialization warning does not establish entropy failure. Keep the working randomness sources. |

The [kernel CPUFreq documentation](https://docs.kernel.org/admin-guide/pm/cpufreq.html)
describes the power/performance tradeoff; the local measurements support retaining
ondemand. CMA/GPU and stack figures below are candidates for measured reductions,
not an instruction to alter live reservations.

### R01 — fix the fortified/custom C logger boundary first

This is a newly traced root cause beyond the earlier generic logging follow-up.
`stm32.cpp` calls `openlog(..., LOG_LOCAL0)`, and the binary defines its own
`openlog`, `syslog`, `vsyslog`, `closelog` and `setlogmask` from the legacy utility.
The build uses `_FORTIFY_SOURCE=2`. The exact deployed binary also imports glibc
`__syslog_chk`; Thumb disassembly shows the startup sequence calling custom
`openlog` at `0xa9f0`, then glibc `__syslog_chk` through `0x2778`.

The custom initialization and libc logging therefore use different internal
state. Runtime bridge messages arrive as `user.info`, while the product route
expects `local0` and `/log/3dr-stm32.log` is absent. Temperature and Python updater
facilities do route, so the active syslog configuration is not wholly disconnected.
glibc documents [fortified call substitution, including syslog](https://sourceware.org/glibc/manual/latest/html_node/Source-Fortification.html).
The specific cross-library state mismatch is established by our source/binary/runtime
evidence, not asserted by that general documentation.

This also bypasses the custom logger's intended nonblocking/drop-accounting path.
No blocking-induced control delay was reproduced; that risk needs a bounded
saturated-log-socket test. The fix should use a distinct OpenSolo logging API
consistently, with compiler format checking and the nonblocking contract retained,
rather than disabling compiler hardening or globally interposing libc names.
Review every owned consumer of this utility and the real-time call sites together.

The live `/usr/bin/stm32` SHA256 is
`356830f47fa2af8616e1c8e72bcef5b56ff145b3ede49d8c4c46c1f76fd539c5`,
identical to the inspected image binary. Adjacent `stm32-logging-symbols.txt`,
`stm32-logging-calls.txt` and `fortify-build-command.txt` preserve the proof.
Owners: SoloLink `flightcode/util/syslog.c` and its C/C++ callers; product recipe pin.

### R02 — make the requested log rotation settings effective

The installed `/etc/syslog-startup.conf` still has commented ROTATESIZE and
ROTATEGENS. The owned BusyBox append's substitutions match only uncommented keys,
but the selected core file has `#ROTATESIZE=0` / `#ROTATEGENS=3`. Consequently
the daemon command line is `/sbin/syslogd -n -O /log/messages`, with no requested
`-s 2048 -b 3`.

The selected BusyBox build has rotation enabled and its source defaults to200 KiB
and one generation; do not describe logging as unbounded from absent arguments alone.
The intended retention contract is nevertheless not installed. Replace or
normalize the actual selected config robustly, and test the real install boundary
with commented upstream inputs. Verify all facility files plus direct appenders,
rotation ownership and disk-full handling on the resulting image.
Owner: `meta-3dr/recipes-core/busybox/busybox_1.36.1.bbappend`.

### R03 — provide a trustworthy time source without blocking offline boot

The timestamp-format/persistence repair is deployed: the saved checkpoint has14
digits, and one managed daemon checkpoints it each minute. That preserves a time
floor but cannot learn current time. The live RTC reads January1,1970; system time
advances from the image/build floor. No standalone ntpd/chronyd/sntp command was
found, and no external synchronization was observed.

Add a small explicitly configured network-time client when a trusted network is
available, while retaining monotonic offline startup and the saved floor. Review
clock-step effects on telemetry/logging and TLS before selecting startup-only
steps versus later discipline. Never weaken certificate validation to hide wrong
time. Do not assume the router serves NTP without checking its configuration.
Owners: product package/service policy and SoloLink clock/timestamp consumers.

### R04 — correct console session ownership

The NXP `rc_mxc.S` wrapper is session/group leader PID571, but launches getty as
child577 without `exec`. The getty remains in SID/PGID571. The installed util-linux
agetty attempts TIOCSCTTY and logs the observed EPERM / process-group warnings.
This is a concrete console recovery defect, not an SSH failure or a proven
respawn storm: the observed getty PID stays stable.

Use an owned controller console definition that executes getty as the init child,
preserving the intended console session. Keep console UART0 separate from the
MCU-owned UART1; the inherited wrapper's generic console guessing should not
grant UART1 to getty on this product. Validate terminal ownership and actual
console login separately from SSH. Owner: product sysvinit-inittab override;
vendor sources stay immutable.

### R05 — close the Linux watchdog ownership gap deliberately

The imx2+ watchdog is exposed and reports `state=inactive`, `timeout=60`,
`nowayout=0`, `bootstatus=0`. No process owns a watchdog descriptor. This improves
observability over the October28 configuration, but does not supply automatic
Linux hang recovery. STM32 power/idle behavior is a separate mechanism.

Select a single watchdog owner with health criteria that include required product
services, then qualify boot, maintenance/update duration, shutdown and recovery on
the bench. Do not arm it merely to remove an audit finding. No watchdog device
was opened during this pass.

### R06 — measure GPU/CMA and thread stacks under the supported workload

The GPU reports a128 MiB system pool with only18,448 bytes used in standby. CMA is
320 MiB, with about92.5 MiB free in the main capture. The pools/accounting may
overlap; do not add them and claim448 MiB reclaimable. Attribute allocation peaks
under video/HDMI start, resolution changes, buffering and repeated restart before
selecting smaller device-tree/driver reservations.

The bridge has about51.6 MiB RSS and53.5 MiB VmLck. Its maps contain six large
anonymous areas around8 MiB each, consistent with the six worker stacks; exact
stack use still requires high-water measurement. Keep mlock and FIFO priorities;
measure and bound thread stacks with a conservative worst-case margin before
reducing them. Neither standby mappings nor successful imports prove safe smaller
stacks. Owners: BSP graphics/memory policy and SoloLink thread construction.

### R07 — minor write-policy and update-capacity follow-ups

The vendor `rc_gpu.S` deletes and recreates image-owned OpenVG selection symlinks
at boot. The fresh overlay consequently gains `/usr/lib` entries; Python also
creates bytecode entries. This is not evidence that the previous overlay was
restored. Prefer deterministic board-selected library packaging, or an idempotent
startup selection if it must remain dynamic; verify actual graphics consumers.
The current one-minute timestamp writes are intentional, not the old ten-second
bug. A further durability/write-frequency change needs an explicit policy.

LATEST has about17.9 MiB filesystem free. Its176,160,768-byte partition exceeds
the current157,200,355-byte payload plus the maintenance16 MiB headroom requirement
by only2,183,197 bytes. The current image fits; future package/kernel growth can
trip the deliberate capacity guard. Plan both-slot geometry before a larger
maintenance cohort rather than weakening the guard or silently resizing this card.

Other observed vendor warnings include USB dummy supplies and early HDMI-audio
probe ordering. A later ALSA HDMI card exists, so that early error alone does not
prove final audio failure. USB gadget configuration is empty and its UDC is not
attached; no USB export or audio/video functionality was exercised. These remain
hardware/peripheral qualification items, not grounds for deleting support.

### Recommended next batch and acceptance

1. Fix R01 logger ownership/fortify interaction and R02 actual log-rotation install
   semantics together. Qualify a full logging call path with real compiler flags,
   correct facilities, full-queue nonblocking behavior and bounded retention.
2. Fix R03 time-source integration and R04 console ownership in owned sources,
   preserving offline boot, role selection, UART isolation and fresh access policy.
3. Coordinate R05 watchdog policy with the existing power/maintenance lifecycle.
   Retain the current normal MCU idle behavior until a separately selected bench
   firmware deployment; do not use an update-screen hold as a substitute.
4. Complete the already documented glibc/OpenSSL maintenance and other scoped
   security follow-ups in the ledger. Working SSH/firewall and this audit do not
   clear the existing source/CVE findings.
5. After related source changes settle, pin a new combined candidate and record
   a new source/consumer gate. Build/validate both roles and profiles as warranted;
   include actual installed configuration checks, not source-text checks alone.
   Deploy only an explicitly selected candidate, then test boot/reboot, longer
   networking observation, actual serial console and product state transitions.
6. Measure video/control latency, GPU/CMA peaks and stack high-water before R06
   tuning. Keep governor, VM, I/O and real-time policy unchanged without a measured
   deficit. Yocto6 and license work remain deferred.

The controller powered down before a final additional read; that failed read and
the router's orderly release are preserved. No claim of a completed long soak,
reboot acceptance, full paired operation or an ideally configured production
system follows from this audit. The new section and its local commit supersede
the October28 report-creation note about being uncommitted; historical findings
and receipts remain intact.

Probe limits are retained in stderr: this image lacks PSI files and tune2fs,
and its BusyBox df/netstat omit some host-style options. Replacement procfs and
supported-command observations were used where available. Missing instrumentation
is not a zero-pressure result, and no mounted-filesystem fsck was attempted.
