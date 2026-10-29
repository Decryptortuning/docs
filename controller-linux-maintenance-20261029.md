# Controller Linux maintenance — October 29

**Physical follow-up:** be98 boot confirms scoped R01/R02/R04 behavior but exposes a missing install utility in R03 startup. The [Chrony startup follow-up](chrony-boot-fix-20261029.md) records corrected local graphfaf, stronger tests and pending live deployment. The original software qualification below retains its historical scope.

The October29 live audit's R01–R04 fixes are implemented locally on top of the completed MAVLink/Python modernization. Candidate graph is `be98b385025c2eff8569d94ab732079ac332ddacfd139c960140f660c5dc7888`. Local software qualification is complete: four builds, sixteen baseline checks, four expanded ARM runtime suites, four compiled C wire suites and four new Linux feature suites pass. The physical controller remains on graph7055 with Artoo OpenSolo-4dev. No installation, flashing, router change or publication is part of this work.

## Source ownership and behavior

R01 replaces libc-symbol interposition with an explicit `opensolo_*` logging API in SoloLink. Every custom-linked C/C++ consumer uses that API; libc-only consumers retain a consistent libc interface. Compiler fortification remains enabled, with printf-format checking. The custom logger uses nonblocking, close-on-exec Unix datagrams, bounded reconnect/formatting and atomic drop accounting. It preserves errno and avoids blocking stderr/console fallbacks. Initialization precedes worker startup and shutdown follows worker joins. This repairs the demonstrated fortified-call state split and restores the intended local0 facility route; it does not establish physical control-loop latency.

R02 installs a complete owned BusyBox startup configuration instead of substitutions that miss commented upstream rotation keys. Syslogd owns its facility-file rotation at2MiB with three retained generations. The obsolete boot-time external renamer is no longer installed. Periodic direct-stream maintenance remains at1MiB with three copytruncate generations for its existing files. Inactive historical source configs are retained, without a second installed rotation owner.

R03 adds a minimal Chrony4.9 client, with its official archive/version/SHA256 included in the source lock. Network NTP-serving and UDP command ports are disabled; the local Unix monitor socket remains. The client drops privileges to chronyd. NTS, authenticated NTP, RTC discipline and refclocks are not enabled. The default configurable source is the standard OpenEmbedded pool. A read-only OpenWrt check established that the router's NTP client is enabled but its server is disabled; no router settings were changed. Plain public NTP is not a cryptographically trusted time source.

The saved image/persistent time floor remains. At initial boot only, a route-present query can step time before normal services, with a10-second bound; an offline boot skips that query. Later standby/maintenance operation slews only, without makestep/initstepslew. Runlevel4 stops the network owner before flight services. An owned volatile marker fences the retained SoloLink GPS step path; failed daemon termination retains the fence. Start/restart is restricted to S/3/5, and manual flight-level restart is refused. Existing GPS-message trust assumptions are unchanged. This establishes a single selected clock owner in the software lifecycle tests; physical runlevel transitions, synchronization on the real network and loaded telemetry behavior remain acceptance work.

R04 supplies machine-specific owned console inputs consumed by the existing NXP recipe. Both roles execute getty directly on UART0 (`ttymxc0`), preserving init's child/session leadership. UART1 remains for the MCU bridge. An attempted late install override failed because the vendor append installs later; the corrected file-selection approach leaves vendor checkouts immutable.

## Frozen inputs and test infrastructure

| Authority | Local revision |
|---|---|
| Integration and checkers | `f9b09225a2d399bb58751f61ec56a2442f21e9d6` |
| Product layer | `fbf7744acc532d2be973afc64c1cb4e6aa9e6b85` |
| SoloLink | `ed88f8c97ba84aefca178e0b3aeccca9d45950d2` |

All changes are locally committed on `codex/controller-linux-maintenance`. BSP, application dependency pins and18 vendor pins retain the preceding candidate's selections. The [source/checker gate](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/source-freeze-5.json) records the complete tracked hashes and actual producer/consumer review before qualification. Earlier freezes and failed results are preserved. Controller development's successful build predates checker-only commits; its image-producing graph is unchanged and the gate records reuse.

The installed QEMU8.2.2 lacks ARM time64 syscalls required for Chrony's nonmutating query. A private static ARM Linux-user tool uses upstream QEMU11.1.3 revision005c61c0ef6f6ae615283440b4bb7be51b2c6ddd plus an owned narrow NULL-timeout recvmmsg_time64 compatibility patch. Non-NULL timeout remains ENOSYS. The actual Chrony call uses NULL on a nonblocking socket; the shim reuses existing message-vector ABI conversion. A tracked builder exports and patches a private source copy; the upstream checkout remains clean. This tool is neither shipped nor selected as a Yocto provider. Its [build receipt](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/qemu-arm-tool/result.json) records binary SHA256 `a75288a2747ae361afe21d709958e1d334dff676682f5878692e793bb877e67b`. It is bounded userspace qualification, not board emulation.

The new checker compiles the actual fetched logger for native fortified/UBSan and ARM execution, rejects a deliberately wrong printf argument, and verifies the linked MCU bridge symbols. It tests saturation/drop recovery, masks, errno, long-message truncation, threaded sends, receiver replacement and the GPS time fence. Actual installed ARM syslogd runs in a private rootless mount/user namespace, with its installed role facility config; a scaled rotation threshold checks retention and a private /dev/full mount proves ENOSPC isolation. Actual ARM logrotate exercises the direct-stream threshold/retention contract. Actual ARM getty produces a login prompt on a private UART0-named PTY and retains session leadership; authentication and electrical UART behavior are outside that test. Actual ARM chronyd parses the installed configuration and queries a controlled loopback NTP server with clock control disabled. Public NTP and the host clock are not touched. Service lifecycle tests substitute only process/OS boundaries and cover offline/timeout, once-only early step, standby restart, flight refusal and stop-failure fence retention.

The first console fixture incorrectly supplied an absolute PTY name to getty, which adds /dev itself; the corrected test mounts its owned PTY at private /dev/ttymxc0. The initial Chrony fixture failure identified the old QEMU syscall gap. Neither failure was treated as target acceptance or hidden by changing production behavior.

## Remaining acceptance

R05 watchdog ownership and R06 GPU/CMA/thread-stack tuning remain deferred until a powered bench session and loaded video/control measurements can establish safe limits. No watchdog was armed and no memory reservation or real-time priority was changed. Artoo firmware selection/programming is separate; a successful Linux build does not authorize its update.

The retained CVE/applicability findings, OpenSSL3.5.9 follow-up and kernel/media cohort remain open. Incremental/cache-based builds and userspace tests are not a clean full rebuild, security clearance, physical reboot/long-soak, pairing/video or flight acceptance. Yocto6 and license work remain deferred. Preserve the older7055 physical and28fd software artifacts independently.

## Qualification and preserved artifacts

[Profile results](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/profile-results.json) identify every report and actual rootfs VERSION. Controller development's build is recorded in wave2-controller-build.log; final-controller-release-build.log, final-drone-development-build.log and final-drone-release-build.log record the remaining successful builds. Earlier checker failures remain beside successful versioned outputs. One drone-release TLS fixture failed during a measured backward host wall-clock jump (the recorded command elapsed time is negative); its unchanged isolated rerun passed with certificate/hostname rejection checks retained. The clock observation is evidence of a test-environment disturbance, not a reproduced target TLS defect.

All four new Linux suites pass actual ARM logger/syslogd/logrotate/getty/Chrony tests and the bounded lifecycle fixtures. The earlier MAVLink contract remains:166 messages and2768 C layout assertions per profile, byte-exact frame parsing/reserialization and bad-CRC recovery. Both ARM drone profiles pass the11 actual legacy DroneKit/Solo consumer tests alongside retained cryptography, TLS rejection, IPC, numerical and safe-import coverage. No target process or device was accessed.

Six dereferenced artifacts per profile are preserved independently of mutable deploy links: SD image, squashfs, package manifest, update archive, MD5 sidecar and boot receipt. [Artifact hashes](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/artifact-sha256.json) retain the original resolved producer paths and checked copies.

| Profile | SD SHA256 | SD bytes | GOLDEN bytes |
|---|---|---|---|
| controller-development | `e1bb825c5ee130351d5d7e86d40dd977c681e2fad7bb96232e9cc41ec1fdf31a` | 201327616 | 197132288 |
| controller-release | `cbfe7203f6c10cea22d3fe93705a4af5e64938279e040b848b95e78ec324d684` | 180356096 | 176160768 |
| drone-development | `53f41e158c5db5cdddf4c69c25744146cbbcdb902278e9d875253b9f57e1215d` | 192939008 | 188743680 |
| drone-release | `6da8afe3e38b6b8ca54521710b69606f8b76ab949bfbf2293cf10f26c0beb3bd` | 171967488 | 167772160 |

Controller development's SD image is192MiB plus its1024-byte trailer, up4MiB from the preceding28fd candidate. GOLDEN is188MiB, versus168MiB on the physical7055 card. Use the new layout or independently verify capacity before an authorized update; these tests did not modify the card.

The frozen tracked inputs and18 immutable vendor pins were rechecked. Source transport and four selected checksum-verified release archives are preserved separately under this receipt directory. They cover the pinned Git/firmware transport and explicitly locked archives, not a complete mirror of every third-party/Rust download. No new independent offline bootstrap, cached-rootfs reproducibility repeat or clean full rebuild is claimed. The earlier7055 recovery and28fd transport remain unchanged. Existing NXP QEMU preference, prior forced-packaging taint, WSL accounting and deferred license-identifier warnings remain in the logs.

[Final qualification receipt](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/qualification-result.json) binds the candidate, checks, preserved artifacts and limits. All implementation commits and documentation are local; publication and hardware acceptance remain separate.

## October29 be98 controller card prepared — boot pending

After exact-target approval, the7,948,206,080-byte controller card (Windows Disk4, WSL /dev/sdf, reader000000002961/LUN1) received the qualified be98 development image. All201,327,616 image bytes passed cache-invalidated readback. The production provisioning functions created188MiB GOLDEN and LATEST,100MiB RWFS and the remaining LOG, and seeded matching boot payloads. Fresh station-only Wi-Fi/DHCP and key-only administrator access are provisioned; the workstation pins the new on-card public SSH host identity with strict checking retained. Both slot payloads and raw U-Boot match the qualified manifest, and all four unmounted read-only filesystem checks pass.

[Card result](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/card-complete.json), [settings](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/settings-complete.json), [image readback](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/image-readback.json) and [SSH public-key pin](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/ssh-host-pin.json) preserve exact evidence. The card was flushed, all filesystems unmounted and the reader detached from WSL. No card backup, old-overlay restore, private host-key export or STM32 operation occurred. Offline layout/settings preparation changes bytes after successful image readback; no final whole-card/source-image equality is claimed. Actual boot, LAN/SSH, clock discipline, dedicated logging, console warnings and reboot/soak remain to be checked. The last observed running baseline was7055 with OpenSolo-4dev; card preparation alone does not establish a new running baseline.
