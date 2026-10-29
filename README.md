# OpenSolo modern port project documentation

The OpenSolo port builds controller and drone images on Yocto 5 Scarthgap, Linux 6.6.52 and U-Boot 2024.04 in native Ubuntu WSL2. Latest locally qualified full-image software is [the Chrony startup follow-up](chrony-boot-fix-20261029.md), graph `faf3df65…`: four builds, sixteen baseline checks and four each of ARM runtime, C wire and Linux suites pass. The controller runs be98 plus the explicitly authorized init-script overlay correction. One warm reboot passes Wi-Fi/strict SSH, early network-time synchronization, dedicated logging, console ownership and unchanged MCU communication. The complete faf image is not installed. The [maintenance ledger](scarthgap-maintenance-a-20261028.md) records scope and limits.

The preceding physical controller baseline was `7055fbe5738126848417be71d94a1e4fb6b83a7c507287527bce2ae2e3babf48`. The CVE task closures completed against the recorded current NVD snapshot, but the security review retains findings and coverage gaps. OpenSSL 3.5.9 follow-up, retained kernel/media updates and remaining applicability work are documented in the ledger. Passing software checks does not establish security clearance, a clean full rebuild or hardware acceptance. On October 29, the explicitly authorized controller SD card received the development image, verified readback and fresh offline Wi-Fi/SSH provisioning. That earlier card’s first controller boot passed Wi-Fi association, DHCP and authenticated SSH; STM32 firmware is unchanged. The short check includes matching 1 MiB memory-only SSH transfers in both directions and zero ath9k reset counters; reboot/long-soak acceptance remains open. New resume-fix commits are local and unpublished; the earlier pause checkpoint was pushed separately. Yocto 6 and license work remain deferred.

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
| [Chrony startup follow-up](chrony-boot-fix-20261029.md) | Physical be98 startup defect, corrected local candidate, actual ARM utility checks and pending deployment. |
| [Controller Linux maintenance](controller-linux-maintenance-20261029.md) | Latest local R01–R04 fixes, four-profile software qualification and remaining physical gates. |
| [Live controller Linux audit](controller-optimization-audit-20261028.md#october-29-fresh-image-runtime-audit) | October29 fresh-image measurements, logger/rotation/time/console/watchdog findings and the next bounded improvement batch. |
| [Python 3 compatibility](python3-compatibility-20261028.md) | Reviewed source scope, updater safety fixes, Python 3.12 regression coverage and deployment limits. |
| [Controller Wi-Fi kernel fix](controller-wifi-pci-20261028.md) | PCIe interrupt root cause, permanent kernel policy, module compatibility and reboot acceptance. |
| [Artoo bench idle option](artoo-bench-idle-20261028.md) | Local idle-only development profile, all-six builds and normal/bench Renode qualification. |
| [Artoo reset diagnosis](artoo-reset-diagnosis-20261029.md) | Controlled application/ROM-request reset comparisons, recovered controller and unresolved physical gate. |
| [Artoo flashing readiness](artoo-flash-readiness-20261029.md) | Current application UART, candidate/calibration checks and unresolved reset/ROM gate; no flashing. |
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

## October29 be98 first boot and corrected Chrony candidate

The prepared be98 controller boots and passes scoped Wi-Fi/DHCP, strict-key SSH and memory-only transfer checks. R01 dedicated local0 logging, R02 actual `-s 2048 -b 3` and R04 getty session ownership are confirmed; continuing UART1 STM32 hardware3/OpenSolo-4dev traffic and zero sampled ath9k resets are recorded. R03 fails because its boot script invokes an install utility absent from the minimal image. This defect was masked by the earlier host fixture and is not an established network/NTP fault.

The [Chrony startup follow-up](chrony-boot-fix-20261029.md) records the physical evidence, local correction and stronger actual-ARM utility coverage. New local graph `faf3df65da77abf796fe1ff23ea5e657d64a3b4245d051ff4407bc94300a40a3` passes four builds, sixteen baseline checks and four each of expanded ARM, C wire and Linux suites. Productab95c94 and integrationd886047 are locally committed. Actual installed script hashes, all frozen source inputs and18 vendor pins match;24 artifacts and verified34-bundle transport are preserved. The live board remains be98; no clock/service/file change, reboot or STM32 programming occurred during this inspection. A prepared init-script-only patch and Linux reboot await live-change authorization. Watchdog/memory work, broader hardware/security acceptance, Yocto6 and license work remain deferred.

## Authorized live correction and warm reboot — pass

The user explicitly approved the init-only SSH correction and Linux reboot. The actor verified the running be98 identity, standby level and exact old script SHA256, then atomically installed the tested root-owned0755 script and verified its syntax/hash. The i.MX reboot changed boot ID from7ed88434-bf80-4b49-bc3a-70d8e4e54067 to4ca4253b-8ec9-4c53-8ea7-b26bca12dbb7. The same strict host-key SSH alias reconnects after Wi-Fi authentication/DHCP. STM32 remains hardware3/OpenSolo-4dev with continuing UART1 reports; no MCU programming occurred.

The early Chrony query corrected the saved-floor offset by94138.686032seconds before runlevel3 and bridge startup. The background process is `/usr/sbin/chronyd -R -f /etc/chrony.conf`, running UID/GID998 with only CAP_SYS_TIME effective; sampled RSS is1664KiB. Chrony reports stratum2, normal leap status and three online sources. Its owned0750 runtime directory and root-owned GPS-fence marker exist. Actual UDP tables show no NTP-server123 or command323 listener. Plain NTP is still unauthenticated; no authenticated-time claim is made.

The last capture is uptime117.91seconds in that new boot. The saved14-digit checkpoint advances from20261029212014 to20261029212114. Dedicated local0 STM32 logging, actual2MiB/three-generation syslog arguments and UART0 getty session leadership remain correct; all sampled ath9k reset counters are zero. This passes one scoped warm reboot and standby clock/network/bridge check. Physical flight-level/GPS handoff, electrical console authentication, offline-network boot, loaded soak, pairing/video and flight remain outside this session.

[Live result](/home/jresp/solo/.integration/chrony-boot-fix-20261029T211214Z/live-install-20261029T211931Z/result.json), [execution gate](/home/jresp/solo/.integration/chrony-boot-fix-20261029T211214Z/live-install-20261029T211931Z/execution-gate.json) and adjacent captures preserve exact evidence. The running device is **be98 plus one documented init-script overlay correction** (SHA256f0b357900dcb71630cf9c43f7a93772a569a0284973f25e00c09ffa9bebfdf2f). Its VERSION is unchanged; the complete faf image was not installed. Future full-image maintenance must account for this overlay override. Source changes and docs remain local, with Yocto6/license work deferred.

## October29 Artoo application link passes; reset/ROM gate remains open

The [Artoo readiness review](artoo-flash-readiness-20261029.md) verifies sole UART1 bridge ownership, actual115200/8N1/no-flow-control settings without a new UART open, and100 identical172-byte parameter readbacks with no timeout (median19.93ms). The current record matches the prior factory SHA and passes the installed-matching calibration validator. Loader/updater hashes match owned sources; automatic firmware candidates and update-enable markers are absent.

Both current4803457 DVT production/bench artifacts match their manifests and exact prior Renode evidence. No new build/emulator run was relabeled. **Flashing is not cleared yet:** prior GPIO46 reset attempts restarted Linux and no complete ROM ID/geometry/readback/release cycle is qualified. Battery-only power is observed, not established as the cause, and no charger remedy is claimed. The reset/power-control path and intrusive old tracing timing need a controlled non-flashing diagnostic. No bridge stop, GPIO write, reset, ROM entry, target file write or MCU programming occurred in this readiness session; calibration is preserved privately. The old manual helper is not the current bench deployer because it reconstructs a version without the bench suffix.

## October29 authorized reset diagnosis — flashing remains blocked

[Reset diagnosis](artoo-reset-diagnosis-20261029.md) records three controlled comparisons. A normal BOOT0low reset restarted Linux with both nominal100ms and1ms pulses; the normal bridge/8N1/UARTdescriptor remained untouched. A separate BOOT0high/8E1 test using the paused bridge's duplicated descriptor also restarted Linux before a recorded ROM ACK. Inactive-low preparation verified live replies, and the new pulse interval contains no storage sync/logging. These results locate the failure in the active reset/boot path, without proving electrical pulse duration, actual ROM entry, a power-brownout cause or a charger remedy.

The controller recovered to boot39e83092 with OpenSolo-4dev, one UARTbridge, unexported control GPIOs and20 matching factory-record readbacks. No erase/write/protection/MCU flash occurred, no updater marker was enabled and no automatic retry is scheduled. Early J2 console/reset-cause capture and actual BOOT0/NRST/power observation are the next useful hardware evidence; no console adapter is connected. Diagnostic source and central docs are locally committed; existing image/source9 receipts and owner build-env.md remain preserved.
