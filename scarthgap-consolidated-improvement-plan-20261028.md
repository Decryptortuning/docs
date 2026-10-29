# Consolidated Scarthgap improvement implementation plan — 28 October 2026

This is the central implementation plan for the frozen OpenSolo Yocto 5 / Scarthgap stack. It combines the independent Grok review, the **corrected final** Muse cross-check, and every disposition in the package-maintenance audit, including its source9 addendum. It proposes work; it does not implement, build, test, publish, contact hardware or authorize flashing. Yocto 6/Wrynose is deferred. **All license work is excluded: there are no license tasks or gates in this plan.** Artoo modernization is separate; this plan covers its source/artifact/updater interfaces only.

The recommended next implementation is one combined correctness-and-core-maintenance candidate, followed by a separate vendor/BSP candidate only when its compatibility review is ready. Preserve source9 as immutable historical qualification. Do the documentation corrections first, prepare related source changes together, freeze and review the actual producers and consumers, then run one four-profile qualification campaign. Do not perform a four-image build after each small cleanup. Hardware holds do not block source planning or offline maintenance qualification.

## Evidence and baseline authority

The following are the primary inputs, read in full. Their conclusions are adjudicated below rather than accepted by majority vote:

- [Grok F1–F17](/home/jresp/Active_GH/OPenSolo_2-reviews/scarthgap-20261028/comprehensive-report.md), SHA-256 `18843f93694b5143682898b250067225b53ffecde8bcc1bce394d02592ea578a`.
- [Corrected Muse synthesis](/home/jresp/Active_GH/OPenSolo_2-reviews/scarthgap-20261028/muse/lead/comprehensive-report.md), SHA-256 `744ffc95bba0b6e7ab05772b090154b259270bce8d07829adc9bc32627ba14ab`. Its first-pass sibling is superseded. Specialist new findings are mapped separately below.
- [Package maintenance audit, including final source9 inventory](scarthgap-package-update-audit-20261028.md), [refactor result](scarthgap-refactor-20261028.md), [documentation index](README.md), [roadmap](roadmap.md), [build guide](build-and-test.md), [board support](board-support.md), and [status/evidence](status-and-evidence.md).
- Frozen receipts: [summary](/home/jresp/solo/.integration/scarthgap-refactor-20261028/summary.json), [source recovery](/home/jresp/solo/.integration/scarthgap-refactor-20261028/source-recovery-final.json), [packaging comparison](/home/jresp/solo/.integration/scarthgap-refactor-20261028/repro-comparison.json), [qualified copies](/home/jresp/solo/.integration/scarthgap-refactor-20261028/qualified-artifacts.json), and [source9 package identity](/home/jresp/solo/.integration/scarthgap-package-update-audit-20261028/source9-final-identity-comparison.json).

Applicable instructions are [the workspace instructions](/home/jresp/solo/AGENTS.md) and the applicable workstation/build-order portions of [the parent instructions](/home/jresp/Active_GH/AGENTS.md). The parent dealer-product architecture is unrelated to OpenSolo requirements. The owner requires related edits to settle, a combined candidate to be pinned, and a review of actual producers/consumers **before any future tests, builds or publication**. This is sequencing, not repeated permission. Routine physical steps in an authorized session do not need repeated confirmation; destructive writes require the explicitly authorized exact device. This planning-only task creates no permanent SSH or hardware prohibition.

| Authority | Frozen revision / identity |
|---|---|
| Integration, `/home/jresp/Active_GH/OPenSolo_2`, `codex/wsl-recovery` | `f485da5fa5f73d6c19afdf0ec35bd7b39ba82a38` |
| Product, `/home/jresp/solo/workspace/layers/meta-3dr`, `codex/wsl-recovery` | `a83fd2cfaa75eb4587f6081c0c6b6a89ab941712` |
| BSP, `/home/jresp/solo/workspace/layers/meta-solo-bsp` | `3c179413923ba9f51c99878b1127db846609ce9e` |
| SoloLink, `/home/jresp/solo/apps/sololink`, `PORT` | `74e468149e4fde37171d076773d6fa88647a87bd` |
| Central docs, `/home/jresp/solo/workspace/docs`, `codex/project-docs` | `f163a2329dd3f82386b760d54b2469058c7923c7`; inventory-only, not an image input |
| Source graph | `36f841398e89201e2e462391cfa2779188349999090295c8b91c967f29380138` |
| Vendor/core | Poky 5.0.4 `200d12b6a58ad961d60a7774ca0f7a9d29498724`; NXP 6.6.52-2.2.0 |
| Compiled kernel / U-Boot | Linux 6.6.52 `e0f9e2afd4cff3f02d71891244b4aa5899dfc786`; U-Boot 2024.04 `6c4545203d123c246c5d7995f2893959506d28e0` |

| Source9 role/profile | Main packages / source recipes | All module packages | SD SHA-256 |
|---|---:|---:|---|
| Controller development | 971 / 166 | 333 | `a9c23f81afa8a1d412b08d791d055601d74b831a925f69e2ea2249f197cb0660` |
| Controller release | 956 / 158 | 333 | `0d57c450cac9165ba34ad913b14870176194ebc8776ad4ce6b12e0fffb5861f3` |
| Drone development | 957 / 168 | 323 | `f95be255ba4bb63da5908d6bea2ba9e385b98b9becdc1be5bd01a422c6926e1e` |
| Drone release | 943 / 161 | 323 | `b106bb4e2cc2d29a3bdf851c1a8d932d1a4bbd1ce7855dad4497cb0a951491f7` |

Each of four initramfs manifests has 190 packages / 58 recipes. Across the eight scopes there are 4,587 package rows and **180 unique source recipes**, with zero manifest/pkgdata version mismatches. The historical inventory was 185 recipes; five local source recipes were added and ten historical units are now absent. Source9 has no package/version delta from wave6, but has the final graph identity in all eight testdata files and four main receipts. The earlier pending wave5 account failure and wave6 initramfs metadata identity are historical, resolved statuses. Counts of 332/322 refer to linux-imx modules alone; cryptodev adds one, giving 333/323 total. Development adds 15 controller / 14 drone packages, not merely three binaries: pip/setuptools, editor splits and their dependencies account for the closure. No shared package has a dev/release version difference.

Source9 records four successful builds, sixteen image/initramfs/ARM checks, and 82 relevant passing tests assembled from unchanged and updated suites (52 + 13 + 11 + 6). Repeat packaging reused unchanged rootfs and cached dependencies. Source recovery restored **34** Git stores without alternates, including the integration clone. None of that establishes a clean full-world rebuild, live authentication, hardware acceptance, MCU flash clearance or comprehensive CVE clearance.

Preserve [qualified-source9](/home/jresp/solo/recovery/scarthgap-refactor-20261028/qualified-source9) and the original historical/audit evidence unchanged. Preserve user-dirty [build-env.md](build-env.md) exactly, SHA-256 `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`; never edit, stage or commit it as part of this work.

## Identity and scope rules for every work package

[workspace.py:39](/home/jresp/Active_GH/OPenSolo_2/scripts/workspace.py:39) hashes every build-input repository's path/revision, recipe URL/revision/branch, firmware records, epoch and integration-policy hashes. Consequently **any committed build-input layer edit followed by repinning changes the graph identity and VERSION/receipts/FAT/MBR/whole-image hashes for all four profiles**. A removed unused recipe, layer README change, source comment or checksum pin for unchanged firmware bytes is still a new image cohort. Such changes need not cause full compilation, but they do require new identity/rootfs/packaging outputs before a qualification claim. Never label repinned outputs byte-identical to source9.

Central docs have `build_input:false`; integration prose outside hashed policy can change without changing the graph. However, **`scripts/workspace.py` is already in `integration_policy`**. Its bootstrap or generated RECOVERY.md changes are not graph-neutral, contrary to Muse R8's blanket “no rebuild” label. Immediate prose guidance can live in central docs; code changes join the next cohort. Keep integration HEAD as separate receipt metadata to avoid a circular self-pin. Preserve the existing graph schema for now; document vendor inventory versus compiled recipe revisions rather than silently changing what it hashes.

Use these owner abbreviations below: **I** = integration repository above; **P** = meta-3dr; **B** = meta-solo-bsp; **S** = SoloLink; **D** = central docs; **V** = selected vendor/core commits under `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/sources`. Vendor updates mean reviewed pins or patches in owned layers, never edits inside existing vendor checkouts or `tmp/work`. Future isolated Linux checkouts belong under `/home/jresp/Active_GH/<repository-name>` unless an explicit workspace destination is selected. C/D means controller/drone; both profiles are affected unless stated otherwise. Effort is hands-on engineering time, not elapsed builds or hardware availability.

## Ordered work packages

### W0 — Correct the front door and preserve the evidence (0.5–1 day, low risk)

**Owner/files:** D `README.md`, `status-and-evidence.md`, `build-and-test.md`, `roadmap.md`, later history copy of the old `/home/jresp/solo/AGENTS.md`; I `README.md` and a scoped new `AGENTS.md`. This document is the only change made now; W0 edits are future work.

Make the first screen identify source9, its four hashes, “not flashed,” and the older live controller state. Point build-guide “current snapshot” links at the refactor result. Retitle relocation sections historical, mark source9 offline acceptance complete for that graph, and keep future-candidate qualification requirements. Clarify OpenSolo/project, OPenSolo_2/integration, inactive meta-solo and active meta-solo-bsp without renaming interfaces. Replace the bring-up diary procedure with a short current-guide pointer while retaining its dated history and applicable physical-work instructions. Explain SSH identity versus Wi-Fi provisioning separately. Record vermagic validation as the release token plus imported CRC checks, not full vermagic/aliases/exported-CRC proof. Describe udev ownership as the platform package's mount ignore list, not a nonexistent udev append. Document hostnames versus DHCP client name.

Add recovery guidance: detached restore is intentional; development requires a deliberate local branch at the pinned commit, followed by explicit mirror seeding. If the named branch already exists, inspect it rather than force-moving it. Network-origin setup is an explicit development transition, not a prerequisite for offline restore. Mark `build_input:false` entries as unchecked inventory, including separate Artoo source/artifact authority; retain their provenance instead of pretending they were verified image inputs.

**Acceptance/dependencies:** manually follow integration → central index → source9 → hardware limits and recovery instructions. No build needed for prose-only changes. Do not move layer README/comments into this independent docs commit; W4 batches those. Evidence and `build-env.md` remain unchanged. Rollback is a scoped docs revert; historical reports are retained.

### W1 — Close source-selection holes and define recovery closure (1–2 days, low/medium risk)

**Owner/files:** P eight app recipes listed in the Grok F7 matrix below; `recipes-firmware/pixhawk/pixhawk-{green,stock}-fw_*.bb`; I `workspace-lock.json`, `scripts/workspace.py`, `scripts/configure.py`, `tests/test_workspace.py`, recovery prose. C/D source guards; Pixhawk payloads are D-only.

Replace the eight AUTOREV defaults with the exact reviewed lock commits, retaining `SRCREV:pn-*` overrides and URL/branch verification. Ensure raw supported recipe use cannot silently float; missing selected objects should fail clearly. For new app branches, change recipe branch text, recipe_sources and checkout inventory together. No implicit seed/fetch/reset in configure/build.

Before any new Pixhawk network fetch, read and hash the known qualified drone payload and available cached/workdir copy; custom wget wrote into WORKDIR, so do not assume DL_DIR contains the authority. Reconcile both stock and green bytes with the recorded image. If copies disagree, preserve each and resolve provenance; do not select a fresh download by convenience. Replace overridden do_fetch with normal `SRC_URI`, distinct download names and SHA-256; use an immutable source URL when available. Keep installed firmware destinations/names unchanged. Capture exact bytes in a separately checksummed archive mirror or extend firmware transport deliberately; current `verify_artifact` is Artoo-specific and cannot simply accept a Pixhawk record unchanged. The preferred small implementation is standard fetch/mirror coverage plus an archive-mirror receipt linked to the transport. Prove missing/corrupt blobs fail and the mirrored bytes equal the qualified payload. This does not promote firmware or authorize programming.

Keep bootstrap detached by default. Improve generated recovery instructions and split dirty-checkout from wrong-HEAD errors. Provide a documented explicit branch/remote preparation step for development; no automatic branch overwrite or mandatory network-origin rewrite. Confirm `bootstrap → deliberate branch creation → seed` works entirely offline and preserves divergent/dirty work. Apply the same remote-provenance guidance to mirrors. `BB_NO_NETWORK` is an acceptance mode once mirrors are complete, not a default that falsely promises an offline archive closure. Network-disabled fetching does not replace immutable pins.

**Acceptance/dependencies:** after W8 freeze gate, use disposable self-contained transport and missing/corrupt source cases; inspect actual recipe expansion and selected revisions for both machines; include all ten recipe sources and staged artifacts. Record archive closure separately from Git closure. W1 code changes alter graph identity and join cohort A. Rollback restores the previous owned commits and lock; never modifies source9 or refreshes its firmware in place.

### W2 — Stop log transfer from operating on an unmounted or redirected source (0.5–1 day, medium data-loss risk)

**Owner/files:** S `flightcode/dflog/dataFlashMAVLink-to-artoo.py` and `tests/test_secure_transfer.py`; P `recipes-sololink/sololink/sololink.bb`, `recipes-core/sysvinit/sysvinit-inittab/imx_drone/inittab`, `recipes-core/opensolo-access/files/{opensolo-log-receiver,README.md}`; I lock and relevant ARM smoke coverage. Runtime D sender → C receiver; both profiles.

Before mkdir, stale snapshot cleanup, pruning, RECENT-link writes or rsync, verify that `/log` is the intended mounted LOG filesystem and neither `/log` nor `/log/dataflash` is a symlink. Fail closed and retry with bounded logging/sleep; put initialization under the error handling so missing mounts do not cause an inittab respawn storm. Recheck on each cycle and before transfer; a startup-only check misses later mount loss. Bind any accepted source to that mount, and do not send an empty fallback overlay tree after an unmount. Preserve regular numeric files, bounded hardlink snapshots, strict host identity, per-device keys and intentional `--delete` inside the receiver jail.

The actual lifecycle is drone `DSYN:4:respawn:dataFlashMAVLink-to-artoo.py` plus independent `DFL:4:respawn:dataflash_logger`. Controller `LOGR:345:respawn:/usr/bin/log_maintenance` is **not** the receiver; receiver directory setup is SysV `opensolo-log-receiver` at start priority 18 in runlevels 2–5, and sshd launches rrsync through ForceCommand. Trace mountall/rcS, receiver setup and sender entry without adding another supervisor. In fstab `/log auto defaults`, `auto` is the filesystem type, not a delayed-mount option.

**Acceptance/dependencies:** real sender entry points in temporary namespaces/harnesses: absent mount, symlinked parent/source, later mount loss, valid empty mounted source, stale snapshot, growing log, interruption/retry. Invalid cases must perform no prune or rsync and leave a populated receiver unchanged; valid cases must still prune stale mirrored files with the selected rsync/rrsync. W5 rsync work is a dependency of final transfer qualification. Record supervisor count and actual target commands. Rollback is a coordinated S/P/I pin revert; no live log deletion is part of validation.

### W3 — Provision Wi-Fi without stranding controller, drone or phone (2–4 days plus hardware evidence, high compatibility risk)

**Owner/files:** P `recipes-connectivity/hostapd/hostapd_2.10.bbappend`, its live `hostap-daemon-2.4/{hostapd.conf,init}` files, `recipes-connectivity/wpa_supplicant/*`; S `config/{configinit,sololink_config,hostapdconfig.py}`, pairing scripts including `pair_solo.py`, existing `opensolo-log-provision` helper and recipe installation; D provisioning/runbook sections. Both roles package hostapd; actual AP/station activation follows role/configuration. Phone/bench joiners are external compatibility consumers.

First write the pair credential lifecycle: a controller AP has a unique provisioned PSK, its intended drone receives matching station credentials, and the operator has a recoverable phone/bench join path. Choose an offline workstation/spare-overlay provisioner as the initial workflow, reusing symlink-safe path handling and atomic replacement patterns, but using restrictive permissions for Wi-Fi secrets. Do not copy public-key file modes blindly. Keep secrets out of source, image, logs, command receipts and the lock. No password or WPS PIN value belongs in this document.

Replace the 1,802-line example only after inventorying its effective product assignments, including interface/SSID, country/channel, AP+station interaction, driver and pairing/WPS settings. The new golden template must not contain a shared usable credential. Gate AP startup on valid provisioned configuration; do not fall back to an open AP. Review WPS enrollment and the hardcoded factory-PIN dependency as part of the same pairing contract; its future policy cannot be inferred merely from SSH changes. Preserve a wired/console/offline recovery route before requiring association.

**Mandatory consumer fix:** [sololink_config](/home/jresp/solo/apps/sololink/config/sololink_config) currently assumes an existing passphrase line and replaces it with `sed`; deleting the template line alone makes the setter ineffective. Update set/reset/get password and SSID paths together, including factory fallback creation, and ensure no old default is recreated. Review `configinit` valid/upgraded/merge/restore and failed-patch fallbacks against new templates. Provision the correct `.conf/.base/.orig` checksum state so first boot and updates preserve intended credentials. A default copied on recovery must remain unprovisioned and fail closed, not silently resurrect the factory secret. Keep personalized SSH admin/host/transfer credentials separate.

Default recommendation: stage unique pair provisioning in cohort A. Retire TKIP only when the required controller/drone radio, phone/app and pairing paths show CCMP compatibility; if evidence is not available, retain its current negotiation temporarily with a specific open acceptance item. Do not declare CCMP-only universally compatible from source inspection. Configinit/WPS concurrency and fallback behavior are test gaps, not demonstrated races.

**Acceptance/dependencies:** host integration tests execute real provisioning/configinit/setter paths against a disposable overlay: missing/invalid secret, escaping/symlinks, interruption, first/second boot, valid old settings, template upgrade, failed merge, rotation and factory reset. Image inspection proves golden templates contain no live shared credential and no fallback creates one. Later W9 proves real pair/phone association, DHCP, re-pair and recovery, including old-overlay migration. Do not apply only the controller half to a mixed fleet. If the workflow decision cannot be settled before cohort A freeze, explicitly defer W3 implementation and all fresh-media wireless acceptance rather than merging secret deletion alone. Rollback uses the saved matched controller/drone provisioning set on spare media; never publishes secrets or wipes the existing controller overlay.

### W4 — Retire selectable legacy paths and batch identity/metadata hygiene (1–2 days, low/medium risk)

**Owner/files:** P `conf/{layer.conf,distro/3dr.conf,machine/imx6solo-3dr-*.conf}`, `recipes-core/images/3dr-image.bb`, `recipes-firmware/artoo/artoo-firmware_3.0.0.bb`, museum recipes, `classes/image_types_3dr.bbclass`, `recipes-core/packagegroups/*`, distro policy, `recipes-extended/{persist-logs,wtmp-rotate}/*`, `recipes-sololink/sololink/sololink.bb`; B kernel append, U-Boot append/patches, machine include, DTS/board comments/README; I profile usage and validators.

Use narrowly scoped BBMASK for retired `.bb/.bbappend` recipes, or move them outside BBFILES; **BBMASK does not retire machine/distro `.conf` discovery**. Move the two unsupported machine configs and obsolete distro outside `conf/machine` and `conf/distro` to a documented reference area, preserving history. Retire generic `3dr-image` with debug-tweaks and Artoo 3.0.0 after inspecting saved rootfs/package selection for accidental auto-discoverable firmware. Keep staged 5.0.0 verification and updater search isolation.

Move hostapd's live files to a maintained recipe-support directory and update FILESEXTRAPATHS before masking the old hostap-daemon recipe. Classify iw 4.1, pyserial 2.7, compat-wireless, disabled 3.10 kernel and libusb-compat separately by provider/dependency evidence. **Do not treat active libusb1 1.0.20 as unused museum content**; W6 owns its migration. Remove U-Boot patch 0002 if the compiled environment is unchanged after its deletion; review associated conditional references, but preserve the required `TARGET_3DR_HDTC` Kconfig/board target introduced by patch 0001. The mere presence of that target is not evidence of dead Kconfig to delete.

Derive shared kernel fragment URI and DELTA lists from one ordered BitBake variable, retaining NXP's `DELTA_KERNEL_DEFCONFIG` application path. Keep effective configs equivalent for this refactor. Add pointers for DRAM placeholders/runtime fixup, early GPIO/reset gaps, U-Boot DT-alias-dependent MMC numbering, intentional empty SERIAL_CONSOLES, and unported drone media. Do not change DRAM sizes, GPIO values or MMC numbers by comment cleanup. Keep netfilter on both boards until consumers are traced; review ath6k inclusion separately from the documented ath9k requirement.

Include profile in the FAT/MBR serial preimage (`graph:machine:profile`), retaining deterministic timestamp/packing behavior. Define one development-package policy variable used by packagegroups and emitted for validator inspection, while retaining independent expected release assertions and actual manifest closure comparison. Correct build.sh help to show `--profile`. Narrow wtmp/config package FILES to installed ownership; remove the dead `DEPENDS_${PN}` line or replace with a real dependency only if a consumer requires one. Broader bindir glob narrowing is optional after an installed-file inventory, not an arbitrary rename campaign.

**Acceptance/dependencies:** after freeze, parse both machines, inspect providers/appends, assert old selections fail clearly, verify hostapd files and Artoo staging, compare ordered effective configs and generated boot env, and inspect package ownership/no missing files. Serial identity must be stable within a profile and distinct across four machine/profile pairs. W4 includes W0's layer README/comments; all enter cohort A identity. No full recompilation solely for metadata deletion; new packaging qualification still applies. Rollback is the scoped owned commits plus lock.

### W5 — Prepare the coherent Scarthgap core/security candidate (3–5 days review/integration, medium/high risk)

**Owner/files:** I `workspace-lock.json`, `config/{common.conf.in,bblayers.conf.in}`, `scripts/{configure.py,build.sh,preserve-native-tar.py}`; V selected Poky/OE commits; P required provider appends/backports. Use table P01–P15 for package-level ownership and tests.

Use Poky 5.0.21 as the concrete reviewed candidate, with a compatible pinned Scarthgap meta-openembedded revision. Review every selected custom append/provider preference against it. Inspect NXP 6.6.52-2.2.2 metadata deltas, but do not automatically replace the entire vendor stack. Its manifest still names the old Poky/OE base; combining newer core with it is **OpenSolo integration work, not NXP-qualified support**. Retain NXP kernel/U-Boot/media pins initially unless a compatibility or urgent security fix requires a reviewed delta. Vendor-only 2.2.2 fixes proven independent may join A; interdependent kernel/media changes move together to W7.

Prioritize actual source fixes for OpenSSH, rsync/rrsync and exposed GStreamer RTP before broad version chasing. Preferred rsync candidate is a reviewed Scarthgap 3.5.1 recipe/backport with matched rrsync; patched 3.2.7 is an explicitly bounded temporary minimum, not blanket security clearance. Retain Python 3.12 and consider 3.12.16 beyond the 5.0.21 3.12.14 recipe only after recipe/patch review. OpenSSL 3.5 LTS provider/engine behavior needs ARM verification despite the same major ABI. Include stable glibc/core/fs/network/parser/trust/native-tool maintenance, including patch-only changes.

Produce a dated scoped CVE ledger for the selected package/source closure: database version/time, PN/PV/SRCREV, patch/fix-commit evidence, configuration/feature reachability, advisory applicability, disposition (fixed, not affected with reason, deferred with owner, unresolved) and tests. Cover the 37 originally prioritized families first, then triage scanner output for the remaining closure; the original 148 individually unreviewed recipes stay explicitly unreviewed until evidence exists. Validate mapping for renamed splits and vendor/fork versions; absence of a CVE filename does not prove absence of the fix. Separate kernel and vendor-binary limitations. Do not manufacture a zero-findings or comprehensive-security claim.

Keep current host affinity/pressure/RAM settings and shared caches. Review new pseudo against Ubuntu tar/openat2 behavior in an isolated candidate TMPDIR; preserve the native-tar safeguard until the real fakeroot ownership/extraction path passes. Avoid reusing the old preserved tar binary merely because its PV is still 1.35; stage/hash the candidate native tool separately and bind it in receipts. No generic host tuning or clean-all campaign.

**Acceptance/dependencies:** W1–W6 source contracts settled; W8 freezes the candidate before metadata checks/tests. Inspect effective providers and patches, all eight package manifests, native tools and runtime consumers. Security report generation uses the existing paired `configure.py --cve-check` and `OPENSOLO_CVE_CHECK=1` build setting consistently; a changed setting must not fail configure --check or be silently omitted. Rollback selects old vendor commits and owned overrides in a preserved candidate workspace, with separate caches/output identity; source9 remains untouched.

### W6 — Modernize owned dependency consumers and define release contents (1–3 days, medium risk)

**Owner/files:** P `recipes-support/libusb/*`, Python recipes, packagegroups, images/policy appends; owned app repositories under `/home/jresp/solo/apps`; fork sources identified by I `recipe_sources`; D release/dependency decision ledger. Both roles/profiles, with development-only tooling as noted in P11.

Remove the local libusb1 1.0.20 shadow in favor of core 1.0.27 plus maintained fixes, once its library location, udev/hotplug and actual linked consumers are reviewed. Priority 6 beats core priority 5 across differing PV/filenames for the same PN; do not try to solve this by a global layer-priority increase. Use pkgdata/ELF dependencies and source call sites to identify USB clients, including recovery/debug/peripheral tooling, before claiming USB behavior is covered. Preserve any required libusb0 compatibility deliberately.

Retain NumPy 1.26.4 and rebuild native extensions with maintained Python 3.12. Review posix-ipc 1.1.1 → 1.2.0 for naming, ownership, timeout and cleanup behavior. Search actual fork/application consumers for monotonic, convert remaining users to `time.monotonic()` and remove the explicit packagegroup dependency only after closure inspection; ShotManager already uses stdlib in several real paths. Do not assume the recipe's presence proves remaining imports. Keep PyMAVlink/DroneKit/Solo dialect forks pinned; review APIs, generated dialect/message IDs, byte order/CRC and listener/thread behavior for selective maintenance rather than replacing with current PyPI majors. Coordinate SoloLink, sololink-python, ShotManager and loader interfaces. MAVProxy is locked but uninstalled; pin it safely, without adding it to images.

Recommended release default for cohort A: retain current package-management/recovery capability while documenting it honestly; do not redefine “release” as a closed rootfs by implication. Trace drone rpm use and remove it only if no actual update/recovery consumer needs it. Review broad Python modules/ensurepip, locales, GStreamer groups and development closures with observed consumers. Absence of pip alone does not prevent runtime installation. Preserve recovery utilities and distinguish BusyBox tar/gzip applets from native GNU tools. Keep iw 6.7/libnl 3.9.0 unless maintenance requires a concrete fix; keep wireless-tools 30.pre9 until legacy iwconfig/ioctl consumers are traced and migrated. The ten removed Bluetooth/NFC/cellular/NXP WLAN/RPC units are completed removals, not new tasks.

**Acceptance/dependencies:** W5 Python/core and W2/W3 actual app paths reviewed together. Test ARM imports/extensions, IPC contention/timeouts/cleanup, monotonic clock behavior, serial/SLIP, MAVLink frame/dialect interoperability and app startup; inspect dev/release manifests independently. Hardware USB reconnect remains W9, not a host-test claim. Retained exceptions get owner, reason and re-review trigger. Rollback reverts consumer and package pin as one unit.

### W7 — Review vendor, kernel, bootloader and media as a separate compatibility cohort (4–8 days source work; hardware duration separate, high risk)

**Owner/files:** B `recipes-{kernel/linux,bsp/u-boot}/**`, machine includes/DTS/defconfigs; P NXP media appends, `adv7610-edid`, UCM/platform packaging; S `vidlaunch.cpp`, `hdmiout.c` and their launch/configuration consumers; I vendor/recipe locks. C boot/display/audio and D capture/encode/pairing are different acceptance surfaces.

Select an exact NXP lf-6.6.y and lf_v2024.04 delta only after comparing source fixes and the required upstream 6.6 LTS security set. The audited branch-tip/kernel.org numbers are investigation references, not a drop-in kernel target or evidence NXP backports are missing. Preserve controller pci=nomsi/CMDLINE_EXTEND, module localversion/cryptodev ABI, DT/reset semantics, UART1, 64 MiB bootm allowance, DCD geometry and slot fallback. Rebase local patches in owned layers. Inspect u-boot-imx-tools dependency/PN separately; do not assume the bootloader SRCREV override pins the tools recipe.

Keep GStreamer 1.24 vendor APIs, FFmpeg/libav library majors, imx-codec/parser/plugin, VPU, GPU/G2D, libdrm, imx-lib/vpuwrap, firmware-imx and ALSA integration coherent. Apply urgent source-level media backports in A where they preserve those interfaces; broader vendor upgrades wait for B. No core GStreamer downgrade or newest 1.28/FFmpeg6/i.MX8/9 binary substitution by version alone. Record vendor binary advisory gaps explicitly.

Drone functional completion is a distinct subpackage: establish ADV7610 input/EDID/timing, media links/pads/formats, `get_mbus_config` expectations and V4L2 node ownership; port the actual legacy `mxc_v4l2_capture`, `/dev/video0`, `mfw_v4lsrc`/`mfw_ipucsc`/`vpuenc` and controller-side decoder/sink consumers together. A dependency/config refactor did not implement this. A base/radio/video/dev packagegroup split may accompany that real consumer work; keep MACHINE_ARCH before inherit. Do not block useful cohort A offline acceptance on completing drone video, but do not claim full drone functional acceptance until it passes.

**Acceptance/dependencies:** a separately pinned B candidate goes through W8 again, using A's evidence as comparison. Validate kernel/module/DT/initramfs and compiled boot environment as a set; then W9 physical boot, radio/audio/video/USB/serial, memory/CMA and recovery. Keep netfilter/ath6k/UUU manufacturing environment until actual consumer requirements justify removals. Rollback preserves whole boot/runtime payload sets and saved environment policy; never mixes a new kernel with old modules. Existing single raw U-Boot/FAT power-loss limits remain visible.

### W8 — Freeze, review, qualify and preserve one candidate (1–2 days review/checks plus build time, medium risk)

**Owner/files:** I lock/wrappers/validators/tests and new dated `.integration`/recovery evidence outside Git; P/S/B source review records; D candidate report. This gate is required before every testing/build wave, including an emergency security-only wave.

1. Finish related source edits; commit only owned reviewed inputs, resolve branch/URL/source availability, update repository and recipe pins, firmware records and policy hashes. Record full SHAs and a new graph ID. No moving source set while tests/builds run.
2. Cross-review actual producer/consumer chains: source lock → generated pins → fetch mirror → recipe; packagegroup → package splits → image; config templates/provisioner → configinit → AP/station/pairing; sender/supervisor → sshd Match/rrsync jail; compiled U-Boot → bundling/packaging → validator → updater/initramfs; Artoo manifest → staged path → updater exclusion; native tar → pseudo → filesystem ownership. Close contradictions before tests. Review reports alone and self-consistent fixtures are insufficient.
3. Generate/check all four configurations only after that source gate. Run parse/provider/append inspection for imx_controller and imx_drone, both profile closures, exact ten SRCREVs, native provider selection, expected package architecture and config fragments. Stop on an unexpected override. These are future checks, not checks run by this plan.
4. Execute targeted host tests for W1–W6 and existing maintenance regressions. Add actual `check_rootfs` negative coverage for private-key headers and nonempty unexpected authorization/trust; test intended valid empty/unprovisioned states separately. `sololog` home resolves correctly to `/var/lib/sololog`; assert existence, root:root and mode 0755, not a fabricated mismatch. Existing validator additionally forbids even empty files at designated pristine-image provisioning paths: preserve or explicitly document that artifact contract; it is not proof that an empty authorized_keys grants access. Verify account creation/order and final root marker with real rootfs output.
5. Build each target/profile once using its own TMPDIR/output, sharing DL_DIR/sstate and existing CPU/pressure policy. Check actual selected package sources/patches and retain logs/source receipts. A new TMPDIR with shared sstate is a fresh work directory, not a clean compiler rebuild. No routine cleansstate/cleanall. Changed toolchains/libraries naturally rebuild dependent tasks.
6. Run image, initramfs, target ARM userspace/SSH-effective-policy and target ARM maintenance checks for **all four** outputs (16 checks), with explicit profile/deploy arguments. Check actual VERSION/receipt/profile/epoch, kernel release token/imported CRCs, DTB, module counts, cryptodev, boot bounds, slot fallback, bundled scripts/BusyBox, Artoo stage isolation, package contents, native tools, file ownership and new volume IDs. Do not reuse the other profile's kernel/rootfs. QEMU userspace proves neither board kernel behavior nor radio/electrical readiness.
7. Preserve outputs, then repeat `-C image` packaging for each unchanged rootfs and compare squashfs/card/archive/sidecar/receipt bytes. `-C rootfs` is a broader rootfs regeneration experiment and must be labeled separately. Record within-candidate comparisons only; source9 hashes are comparison history, not new expected identity.
8. Export a self-contained transport and restore to a new native Linux workspace with original objects unavailable. Verify no alternates, exact reachable tags/branches, integration provenance, ten recipe mirrors, firmware and explicit archive closure. Demonstrate offline fetch from a separately complete archive mirror with network disabled before claiming it. The source9 34-store result is retained; count the new inventory rather than hardcoding 34 as a universal constant.
9. For a release reproducibility claim, perform one planned independent rebuild from empty TMPDIR and empty separate sstate with all sources supplied, after the first candidate passes; build both machine closures and both profile images, compare outputs and explain differences. This remains required evidence for that claim, not for every cleanup. Do not erase shared caches or say that source recovery/packaging repeats already proved it.

On failure, preserve the failed logs/artifacts and exact graph, fix related sources together, repin and repeat the affected behavior checks. Regenerate all four identity-bearing artifacts when the graph changes; reuse compilation/cache results where signatures permit. W8 does not lift hardware/MCU holds. Record validator/test script hashes in the qualification receipt alongside integration HEAD; **do not add them to image build_input solely for checker changes**. This deliberately declines the optional checker-hashing suggestion while making qualification reproducible.

### W9 — Physical acceptance and later publication (separate scoped sessions, high consequence)

**Owner:** board/integration owner with D runbooks and exact candidate receipts. No hardware actions or publication are requested now.

Start later with a selected controller profile on fresh spare media, fresh RWFS/LOG and offline SSH/Wi-Fi provisioning. Detect the actual card and retain its identity/authorization before destructive writes. Preserve the working controller and existing serial/log evidence. Validate first/second boot, GOLDEN/LATEST payload hashes, partitions, boot args/RAM, U-Boot MMC and Linux mapping, environment persistence/default migration, host-key generation, real pinned-key authentication/rrsync restrictions, DHCP, pair/unpair, phone join, sustained radio/audio and authorized peripheral reconnect. Decide old-overlay and saved-environment migration explicitly; a new squashfs does not revoke old keys or reset saved U-Boot environment. Do not blind-wipe an old overlay. Separate GOLDEN migration, Linux-started recovery, update interruption and raw single-copy bootloader handling; use disposable media for fault cases.

Drone acceptance follows actual media-port readiness and independent board evidence: capture/encode/decode/restart, flight-controller communication, pairing, USB/serial, eight-bit CSI/BT.656 wiring and UART1 pin use. Do not infer 16-bit 1080p support or UART5 accessory needs from historical names. Physical DRAM/DCD/runtime bank fixup/CMA must agree for each board. Early GPIO1_IO19/21, GPIO2_IO14/13 and three-second reset inputs GPIO3_IO24/controller and GPIO4_IO05/drone need schematic/revision/sequence ownership before implementation.

The known battery-only STM32 reset caused an i.MX restart; ROM-loader/flash readiness remains blocked. Charger A/B is separately deferred. Maintain the selected Artoo source `60a2707…` manifest/hash/staged_only contract; newer native development firmware is not promoted by this plan. A separate Artoo branch is permitted, including future use of `codex/artoo-wsl`; no PORT history rewrite or new MCU reimplementation is needed. MCU investigation/programming must meet its own scope and readiness, independent of image packaging success.

When publication is separately authorized, use the sequence below. A source/offline-qualified artifact may be labeled as such without pretending it has hardware acceptance; public claims must state the exact achieved scope.

## Recommended execution and branch sequence

| Phase | Concrete sequence and stop condition | Image work |
|---|---|---|
| 0: immediate | W0 prose authority and inventory distinctions; preserve source9 and user-dirty docs. | None. |
| A preparation | W1 source holes → W2 sender → W3 provisioning contract → W4 hygiene; prepare W5 core/security and W6 consumer changes alongside those source tasks, without tests while related edits move. | No intermediate four-profile rebuild. |
| A freeze/qualification | Select exact compatible core/OE plus vendor exceptions; finish W1–W6, lock and cross-review W8, then one four-profile build/check/packaging/recovery campaign. | New A identity for all four, cache reuse. |
| Urgent fallback, only if justified | If compatibility/Wi-Fi work materially delays an applicable urgent fix, freeze a deliberately smaller A0 containing W1/W2 and reviewed source-security backports; explicitly retain unresolved W3/W5 items and hardware restrictions. Do not deploy a half-provisioned Wi-Fi change. | One extra cohort only for a documented security/acceptance need, not automatic. |
| B vendor/BSP | W7 coherent vendor/kernel/U-Boot/media candidate after its source review; use A as baseline, W8 again. An independently ready source fix may be batched, not cherry-picked by newest PV. | Separate high-risk cohort; no repeated full clean rebuild per package. |
| Hardware | W9 on authorized exact spare media, first selected controller profile then relevant drone/profile evidence. Qualification is per actual tested profile, not inferred across dev/release. | No MCU clearance implied. |
| Publication | Reviewed multi-repo refs/source availability → immutable source/archive transport → labeled artifacts and central evidence. | Only after separate authorization; no push now. |

For future implementation, create scoped `codex/scarthgap-maintenance-<cohort>` branches in affected owned repos without resetting existing work; a different requested branch takes precedence. App changes commit first, then P/B recipes/configuration referring to those exact commits, then I combined lock/policy commit. A recipe's branch and lock branch must match; an app branch change is not just a revision update. Use explicit local seed/transport to make unpublished commits fetchable for qualification; do not push merely to satisfy BitBake. Forks without a local source checkout need a deliberate isolated checkout before changing them. Keep vendor pins detached/immutable and carry downstream changes in owned layers.

D may commit its own reviewed plan/status later, explicitly excluding `build-env.md`; no commits are made now. After all related commits settle, record the combined freeze/review before tests. If later authorized to publish, publish required app/fork refs first, then P/B refs and any selected immutable vendor references, then integration lock, then docs/artifact receipts. Verify remote object availability without substituting branch tips. Preserve exact integration commit in transport manifest, source graph, recipe URLs/branches/SHAs, firmware digest/sidecar, archive mirror checksums, tool hashes, CVE evidence date, all image/profile hashes and qualification scope. Publish new immutable directories/manifests, never overwrite source9. A publication failure does not justify forced ref updates or rewriting PORT.

## Prioritized package maintenance table

P01–P15 correspond one-for-one to the original audit's fifteen priority rows; P16–P23 capture its P2/ownership/image-content items. Baselines below are final source9 installed values unless explicitly H/native or boot artifact. Targets are proposals based on the audit's source evidence, not verified runtime support. All conditional vendor combinations require W8 provider/source review. Security backports can enter A without forcing the W7 broad upgrade. **C/D** = main roles, **I** = initramfs where selected, **H** = host/native.

| ID / priority / group | Baseline → proposed approach | Ownership, compatibility and reason | Required build/runtime evidence |
|---|---|---|---|
| P01 P0, A/W5 | OpenSSH 9.6p1 → maintained 9.6p1 Scarthgap backports | V Poky recipes-connectivity/openssh; P openssh append/access. Existing CVE-2024-6387 patch stays; inspect equivalent fixes for 2025-26465/26466. VerifyHostKeyDNS client exposure is configuration-dependent; server resource exhaustion separate. C/D. | Real root/sololog account policy, Match tail ordering, disposable-key sshd -T, later first-boot keys/pinned login, SCP/SFTP/recovery and restricted transfer. |
| P02 P0, A/W5 | OpenSSL 3.2.3 → proposed 5.0.21 3.5.8 | V recipes-connectivity/openssl. Maintained LTS; review providers/engines/cryptodev/CAAM despite shared major ABI. C/D/I/H. | Rebuild consumers/extensions, ARM ssl/hashlib/TLS/certificates/entropy/throughput and used providers. |
| P03 P0, A/W2/W5 | rsync 3.2.7 → reviewed 3.5.1 preferred; patched 3.2.7 only bounded fallback | V recipes-devtools/rsync; P rsync_%.bbappend and rsync-rrsync split. Tag backports include 2024-12084–12088/12747, but 3.5.1 is beyond that tag. C/D. | Same-source rrsync/shebang/deps; actual forced-command transfer, protocol negotiation, jail escape rejection, metadata, partial/retry/delete and W2 mount cases. |
| P04 P0/P1, A backport / B upgrade | GStreamer core/base/good/bad 1.24.7.imx → vendor 1.24 security backports | V meta-imx multimedia; P owned appends. Audit CVE-2026-18649 RTP H.264/H.265 reassembly source/usage; no core 1.22 downgrade or automatic 1.28 replacement. C/D. | Inspect actual depayloader/pipeline; bounded fragmented RTP memory, valid stream/reconnect, then W7/W9 DMA/VPU/latency. |
| P05 P1, B/W7 | Linux 6.6.52 e0f9e2afd4cf → selected NXP lf-6.6.y + required 6.6 LTS fixes | B linux-imx append/DTS/fragments; V source pin. Audit observation 5a0a5e71d2bd still labels 6.6.52; upstream 6.6.160 was context, neither is a qualified target. C/D/I. | Source CVE/backport delta, effective config, DT, module/cryptodev ABI, bundled initramfs, ath9k/MMC/USB/time64/DMA and physical boot. |
| P06 P1, B/W7 | U-Boot 2024.04 6c4545203d12 → assess lf_v2024.04 e3219a5a7344 | B U-Boot append/board/env; V recipe source. Retain current pin in A except reviewed necessary fix. Vendor checkout de16f4f… is manifest inventory, not compiled authority. | UART1, MMC aliases/env, bootm bounds, generated env/duplicate allowlist, both slot load failures, initial/second boot, saved-env migration; tools PN inspected separately. |
| P07 P1, A/W5 | glibc 2.39 84f6bfce2c37 → audited 2.39 be1e627cd72d + tag patches | V recipes-core/glibc and glibc-locale. Stable fixes incl. audited 2026-5435/6238; retain correct prior 2024-2961 disposition. C/D/I. | Rebuild dependent userspace; resolver, pthreads, locale/iconv, ARM32 time/file interfaces and Python extensions. |
| P08 P1, A/W5 | BusyBox 1.36.1, util-linux 2.39.3, e2fsprogs 1.47.0, dosfstools 4.2 → maintained same-family recipes | V core/fs; P BusyBox/init/recovery appends. Patch-only maintenance matters, including BusyBox 2024-58251. C/D/I/H selection. | Real ARM applets, mount/blkid/fsck, filesystem features compatible with recovery, partition seeding and update fault paths on disposable images. |
| P09 P1, A/W3/W5 | hostapd/wpa-supplicant 2.10, dnsmasq 2.90 → maintained Scarthgap backports | V/P network recipes/appends. Keep 2024-3596 fixes; check enabled-feature SAE/DHCP advisories. iw 6.7/libnl 3.9.0 stay unless needed fix. C/D. | AP/station, pairing/WPS policy, DHCP/reconnect/regulatory/channel concurrency and latency; real hardware gate separate. |
| P10 P1, A/W5/W6 | Python 3.12.6 → tag 3.12.14; assess 3.12.16 security increment | V recipes-devtools/python. Stay 3.12; 3.12.16 is extra beyond tag, not silently included. C/D/H. | Native extension rebuild, serial/bytes/pickle/subprocess/imports, archive/XML/cookie/tempfile consumers and app startup. |
| P11 P1, A/W5/W6 | pip 24.0 / setuptools 69.1.1 → maintained Scarthgap backports | V Python recipes. Development-only main-image closures now; H setup/build dependencies remain. Release ensurepip/modules separately reviewed. | Build-time setup consumers, ARM development tools, explicit manifests; no removal of build dependencies because runtime pip is absent. |
| P12 P1, B or bounded A fix/W7 | FFmpeg 4.4.1 + gst-libav 1.24.0.imx → assess 4.4.8/backports | V meta-freescale/meta-imx multimedia; retain libavcodec58/libavformat58 and NXP patches. Controller installation; core 6.1.4 is not an automatic provider replacement. | Used codec/demuxer and malformed-input tests, actual gst-libav linkage, video decode/display and restart. |
| P13 P1, A/W5 | Expat 2.6.3 → 2.8.3; CA 20211016 → 20260816; GLib 2.78.6, libxml2 2.12.8, libpng 1.6.42, GnuTLS 3.8.4 → maintained selected Scarthgap recipes | V parser/trust libraries. Audit targets, no invented latest PV for remaining members; assess actual patch applicability. C/D and native closure. | XML/image/metadata consumers, TLS trust/clock dependence, graphics/media and linked-library regressions. |
| P14 P1/P2, A/W6 | libusb1 1.0.20 → core 1.0.27 + fixes | P legacy override actively shadows core across PV; remove consciously, preserve needed library/udev behavior. C/D/H. | Dynamic provider proof, ELF/pkgdata consumer closure, ARM USB calls and later actual peripheral reconnect/hotplug. |
| P15 P1, A/W5 | pseudo-native 1.9.0+git 374089f2 → audited 1.9.11 ba8887e5; git-native 2.44.1 → 2.44.4; tar-native 1.35 → patched 1.35; squashfs-tools-native 4.6.1 retain version | V native recipes; I preserved tar selection. H only; do not infer patch state from same PV. | Candidate clean TMPDIR, fakeroot ownership/openat2, archive extraction, Git transport, tool hashes, deterministic packaging; remove workaround only after proof. |
| P16 P2, A/W6 | NumPy 1.26.4 retain; posix-ipc 1.1.1 → assess 1.2.0; monotonic 0.6 → stdlib consumers then remove | P Python recipes/packagegroups, owned app consumers. NumPy2 deferred; on-disk posix-ipc0.9.9 is not installed baseline. | ABI/imports, IPC naming/ownership/timeouts/cleanup and monotonic timeout behavior. |
| P17 P2, A selective/W6 | PyMAVlink 1.1.56+git3887c3f7b4, DroneKit2.4.0/9b1fa763…, DroneKit-Solo1.2.0/5d53ef72… retain owned forks | P recipes-mavlink and exact locked source repos. Owned support obligation; no blind PyPI replacement. | Real dialect generation/IDs/CRC/bytes and application listener/thread lifecycle, ARM imports. |
| P18 P2, A selective/W1/W2/W3/W6 | SoloLink74e468149e, ShotManagerfc9fd5bdfb, sololink-python975bda06…, stm32loadera8db3edf… | Owned apps + P recipes/I pins. Current source9 alignment supersedes old image versions; change only reviewed interfaces. | Config/pairing/logging/serial/updater callers; no MCU reset/flash induced by tests. MAVProxy pinned but remains uninstalled. |
| P19 retain, W1/W4/W9 | Artoo selected5.0.0-dev+g60a2707…, Pixhawk green3.5.4/stock1.5.4, gimbal1.3.6 retain payloads | P firmware recipes/I artifact authority. Checksummed Pixhawk and retired Artoo3.0 improve selection without firmware promotion. | Byte equality to known payload, staged_only/path isolation/sidecar; later separate protocol/hardware authority. |
| P20 P2, B/W7 | imx-codec4.9.0, parser/plugin4.9.2, VPU5.4.39.3, GPU/G2D6.4.11.p2.10, firmware-imx8.26 plus imx-lib/vpuwrap/ALSA/libdrm retain coherent vendor set initially | V/P vendor stack. Upgrade only verified i.MX6 ARMv7-compatible bundle or required firmware subset; vendor advisory coverage incomplete. | DMA/buffer/codec/display/audio/CMA and actual drone graph; no i.MX8/9 compatibility inference. |
| P21 complete, W0/W6 | Ten historical Bluetooth/cellular/NFC/NXP WLAN/RPC recipes already absent | Preserve absence in new manifest; firmware-imx remains. No duplicate removal implementation. Readline remains in initramfs. | Eight-scope manifest diff; investigate any reintroduction. |
| P22 P2, A policy/W6 | broad GStreamer/Python/ensurepip/locales, package managers incl. drone rpm and dev tools | Consumer-led minimization; default retain recovery and current package-management while use is traced. Split groups later with video work. | Actual installed closure/consumers; no “release means immutable” assumption. Distinguish BusyBox tar/gzip from native/standalone tools. |
| P23 P2, retain/review W6 | wireless-tools30.pre9, iw6.7, libnl3.9.0 | Legacy wireless-tools consumers need migration before removal; iw/libnl only change for required maintenance. | Source/ioctl/command inventory and AP/station network behavior. |

Targeted primary-source confirmation was performed on 2026-10-28 America/Phoenix (2026-10-29 UTC where applicable). The [official 5.0.21 release notes](https://downloads.yoctoproject.org/releases/yocto/yocto-5.0.21/RELEASENOTES) were read directly after the web reader failed; they confirm Poky `4fb7016a07507683359ebd620f1951fd5ea44bfa`, OE-Core `b4f06303e55c7270a8482a610927a1c41d5dbf9c`, BitBake `d3b32587c2314febf96897f97e0172790b4fc1cf`, and Expat 2.8.3. These are candidate pins, not changes to the current workspace. Other package targets retain the audit's linked recipe/advisory evidence and need exact-commit validation during W5.

The [NXP 2.2.2 manifest](https://raw.githubusercontent.com/nxp-imx/imx-manifest/imx-linux-scarthgap/imx-6.6.52-2.2.2.xml) still pins Poky `200d12b6a58ad961d60a7774ca0f7a9d29498724` and OE `72018ca1b1a471226917e8246e8bbf9a374ccf97`; it does not establish support for the proposed core refresh. The audit's OE `2db6aece32c7f8ebf6015b468e1b4e21de072e3b` is a review starting point, not a compatible lock selected by this plan. [GStreamer's advisory](https://gstreamer.freedesktop.org/security/sa-2026-0076.html) recommends a patch for older branches' RTP reassembly growth. [Upstream rsync](https://rsync.samba.org/) confirms 3.5.1 and protocol 33. Neither page proves OpenSolo runtime compatibility; W5/W8 own that determination.

## Traceability and adjudication

“Implement” means a concrete future task, not work performed now. “Retain” and “defer” are explicit dispositions with reasons. The reviews' advice to leave package upgrades outside their original review scope does not exclude them from this user-requested combined plan.

| Grok finding | Disposition / task | Adjudication and acceptance |
|---|---|---|
| F1 authority | Implement W0; layer pointers W4 | Stale entry points are operational risk. Preserve routine-step authorization; review-only no-hardware scope is not permanent SSH ban. Prose-only no graph churn; layer docs do churn. |
| F2 factory AP PSK | Implement coordinated W3 | Inherited Wi-Fi boundary, separate from SSH. Provision controller/drone/phone and fallback/setter consumers together; no lone secret deletion. |
| F3 custom wget | Implement W1/P19 | Drone-only standard fetch/hash/mirror using known bytes. Firmware may stay identical; repinned image identity cannot. |
| F4 sender mount | Implement W2/P03 | Verify mounted source before all destructive operations/transfer; preserve intentional --delete and existing single supervisor. No asserted live failure. |
| F5 detached recovery | Reclassify ergonomics; W0/W1 | Detached restore/configure works. Explicit development branch/remote transition; no auto-reset. workspace.py changes are hashed. |
| F6 legacy selections | Implement W4 | Supported images do not contain debug-tweaks. BBMASK recipe paths; relocate obsolete machine/distro conf. Parse/selection checks after freeze. |
| F7 eight AUTOREV | Implement W1 | Wrapper already pins safely; not a standards violation. Pin sololink, sololink-python, shotmanager, stm32loader, dronekit, dronekit-solo, pymavlink, mavproxy defaults and preserve overrides. |
| F8 distro override | Retain/defer W4 trigger | No current failing consumer demonstrated. Extend fsl with product override only when needed; prefer feature variables for actual features. |
| F9 stale inventory | Implement W0/W1 documentation | build_input:false not verified/restored image authority; docs/artoo/scripts provenance explicit. Do not add docs to image graph or restore stale docs blindly. |
| F10 profile serial | Implement W4 | graph:machine:profile; operational uniqueness, not authentication. Four-way determinism gate. |
| F11 release definition | Implement policy W6 + W4 helper | One declared dev-package policy; inspect 15/14-package closure. Keep package-management by default until explicit consumer-based change; rpm/ensurepip open. |
| F12 dead/museum | Implement W4; active USB W6 | Preserve live hostap files before retirement, isolate old Artoo, remove dead env patch only with generated-env proof. Required board target Kconfig stays. |
| F13 fragments/comments | Implement W4; behavior W7/W9 | One fragment list, same effective config; DRAM/GPIO/reset comments point to gates. Netfilter/ath6k removal requires consumer evidence. |
| F14 U-Boot identities | Implement labels W0/W4; tools watch W7 | Compiled recipe SHA and manifest checkout SHA differ intentionally. Retain graph schema; no checkout overwrite to conceal distinction. |
| F15 evidence/tests | Implement W0/W8 | 82 stitch and 34 stores correct; packaging vs clean rebuild bounded; actual check_rootfs coverage and release-token precision. Empty keys grant nobody access. |
| F16 metadata | Implement W4 | Narrow owned FILES, remove dead dependency syntax; provider-owned udev claim corrected. Legal broad FILES is maintainability, not theft of other recipes' files. |
| F17 priority/breadth | Correct and implement W6; retain breadth | Same PN priority applies across differing PV/filenames: local libusb1 actively shadows core. No wholesale priority increase; declared BSP→product dependency valid. Wide NXP layer removal deferred until closure benefit/parse proof. |

| Corrected Muse input | Disposition / task |
|---|---|
| Lead N1 sololog home | Reject mismatch; assert actual root-owned0755 path in W8. |
| Lead N2 TKIP/template breadth | W3 effective assignment inventory and compatibility-gated TKIP retirement; packaged old secret remains readable despite overlay rotation. |
| Lead N3 missing offline Wi-Fi provisioning | W3 pair lifecycle, offline helper and configinit/setter changes. |
| Lead N4 MMC aliases | W4 comment guard; W7/W9 verify nodes/aliases/env mapping, keep numbering. |
| Lead N5 empty SERIAL_CONSOLES | Retain; W4 comment. Linux getty ownership is not U-Boot CONSOLE_DEV. |
| Lead N6 configinit/WPS/fallback | W3/W8 integration coverage and W9 first-boot evidence; reject proven-race characterization. |
| Lead R1 | W0 immediate prose; W4 layer comments/README. |
| Lead R2 | W4 retirement; correct BBMASK-versus-conf distinction. |
| Lead R3 | W4 hostap files/dead patch/museum; W6 active USB. |
| Lead R4 | W1 fixed defaults; cohort identity changes even matching pin. |
| Lead R5 | W1 drone fetch integrity, known bytes/mirror closure. |
| Lead R6 | W3 coordinated credential migration; CCMP gated. “Fifth image” becomes coherent four-profile new graph cohort. |
| Lead R7 | W2 gate before initialization and every cycle, not merely rsync mock. |
| Lead R8 | W0 prose immediate; W1 optional helper/code batched because workspace.py hashed. Preserve intentional detach and explicit branch development. |
| Lead R9 | W0 inventory, W4 identity/metadata, W6 release decision. Decline hashing validators into image graph; W8 receipts bind their exact versions. |
| Lead R10 | W9 spare-media per-profile acceptance; hardware/MCU/video limits distinct. |
| Yocto-policy N1/N2/N3/N4/N5 | W4 dead DEPENDS/config ownership/retirement; retain PACKAGE_ARCH-before-inherit; W1 optional offline mode only after archive closure. Broad bindir narrowing optional. A selected legacy image does not itself select a different DISTRO; reject that causal overstatement. |
| Workflow N1/N2 | W0/W1 branch prerequisite and optional remote transition; all restored repos/mirrors covered. |
| Workflow N3/N4/N5/N6 | W1 offline closure; retain common source pins with separate profiles; retain noncircular graph; W8 checker receipts instead of image graph expansion. |
| BSP N1/N2/N3/N4/N5 | W4/W7/W9 alias/getty/DRAM guards; retain runtime bank fixup/64MiB bootm; explicit INITRAMFS_IMAGE_NAME already exists despite N4 heading; W8 validates bundle but cannot prove physical boot; retain controller-only nomsi. |
| Access NF-A1/NF-A3 | W3 offline helper, original golden-secret exposure and migration; no unrelated mDNS change. |
| Access NF-A2 | Reject “auto delays mount”: fstype, not option; W2 actual mount/supervision trace. |
| Access NF-A4 | Reject home mismatch at current localstatedir; W8 actual ownership assertion. |
| Muse negative-test correction | W8 distinguish no-access empty keys from nonempty authorization and stricter pristine-image file-absence contract. |
| Muse scope/identity corrections | All four hashes change after layer repin; W0/W4 split; no permanent approval ban; no standards violation from allowed AUTOREV overrides/LAYERDEPENDS/distro choices. |

| Package-audit work item/addendum | Concrete disposition |
|---|---|
| Original priority rows 1–15 | P01–P15/W5–W7 cover every baseline/target, including native tools and vendor exceptions. |
| P2 NumPy/IPC/monotonic | P16/W6 retain/rebuild, assess update, migrate then remove respectively. |
| P2 forks/app source epochs | P17/P18/W6; current source9 pins supersede old image values. |
| P2 firmware/vendor binaries | P19/P20/W1/W7/W9; retain artifacts and coherent vendor set, separate flash acceptance. |
| P2 removal/minimization/utilities | P21–P23/W6; completed removals retained, remaining consumer policy explicit, BusyBox/native distinction. |
| Audit WP1 freeze inventory | Completed for source9; preserve receipts W0 and repeat for new candidate W8. |
| Audit WP2 coherent candidate | W5 complete pins/patch deltas; rsync/Python beyond-tag exceptions explicit. |
| Audit WP3 vendor/fork exceptions | W6/W7, named retained-exception ledger; no latest-major replacement. |
| Audit WP4 frozen source gate | W8 steps1–2, before tests/builds/publication. |
| Audit WP5 qualify candidate | W8 four-way/ARM/CVE/source recovery, W9 physical acceptance; no inferred MCU fix. |
| Source addendum: two role packagegroups | W4 policy variable/architecture, W6 contents, W8 closures; five new source units accounted in appendix. |
| Source addendum: opensolo-access + receiver split | W2/W5/W8 root/useradd order, OpenSSH/Match placement, mounted jail. Old wave5 failure resolved; retain regression coverage. Receiver is binary split, not extra source recipe. |
| Source addendum: opensolo-platform-config | W5/W8 sysctl initialization, udev mount-ignore ownership, Python shim, init/recovery ordering. |
| Source addendum: controller UCM | W5/W7/W9 ALSA data versus kernel audio card naming; C-only. |
| Source addendum: rsync-rrsync | P03/W2/W5 selected same source, options/shebang/deps; C receiver binary split. |
| Source addendum: image/policy classes and provider appends | W4/W5/W8 identity/profile validation, BusyBox/init/network/multimedia/BSP append closure; not extra installed recipes. |
| Wave6 completed images/additions/removals | W0 current history labels; W6 preserve ten removals; sololink-config/sololink-led are two splits of sololink, no source-count inflation. |
| Wave6 dev/release closure | W4/W6/W8 actual15/14-package delta and shared-version equality, not a three-binary assumption. |
| Wave6 reused initramfs metadata | Historical limitation superseded by source9 identities; retain evidence, no rebuild just to refresh unused ambient metadata. |
| Final source9 | W0/W8 preserve eight identities, 4587 rows/180 recipes, no wave6 package delta, 333/323 module totals, packaging-only repeat limits. |
| 148 originally not individually CVE-reviewed | W5 dated triage explicitly scoped; appendix routes every remaining unit. No implied clearance by table presence. |

## Retained design and deferred preferences

Keep the custom locker, self-contained bundles/tag preservation/no alternates, explicit seed, dirty-work refusal, native WSL paths, source epoch policy, current public boot/update/machine names, and exact four-member update archive format/sidecar contract. Retain machine SoC/tune factoring, standard extrausers/useradd ordering, non-authenticating root marker, key-only SSH, Match-at-tail, a real receiver shell for ForceCommand, strict sender host identity, generated boot environment and both slot load gates. Keep the narrow duplicate-key allowlist (`initrd_high` identical and empty `mtdparts`) that rejects unknown/conflicting/boot-critical repeats. UUU/manufacturing environment removal is a separate real-consumer decision.

Do not adopt kas as a second authority, rename interfaces, force a UBOOT_SUFFIX assignment already supplied correctly by NXP, or split all packagegroups for taste alone. Layer breadth may be reduced only with measured dependency closure and both-machine parse proof, after higher-value maintenance. DISTROOVERRIDES extension waits for a real product override. Optional externalsrc/devtool would be a distinct future profile with explicit source/task identity; it does not replace pinned development/release now. No Wrynose implementation or automatic flight/MCU payload upgrade is included.

## Decisions remaining at implementation freeze

| Decision | Recommended default / required evidence | Blocking scope |
|---|---|---|
| Exact compatible core/OE/NXP set | Poky5.0.21 candidate; review OE2db6aece… and vendor2.2.2 deltas, retain pinned high-risk vendor binaries/kernel initially. W5 exact provider/append/source review. | Cohort A source freeze; no assumed vendor support. |
| Wi-Fi pair lifecycle and supported phone/bench clients | Offline matched controller/drone provisioning with recoverable operator join; no shared image credential. Specify WPS policy and rotation/reset behavior. | W3 implementation/fresh-media wireless acceptance. |
| TKIP removal | CCMP-only goal after required client compatibility evidence; retain temporarily if absent. | TKIP retirement, not all source maintenance. |
| rsync3.5.1 / Python3.12.16 | Review exact Scarthgap recipes and source deltas; 3.5.1 preferred, Python tag baseline acceptable only with explicit advisory disposition. | Relevant package selection/security acceptance. |
| Release package-manager/rpm/minimal modules | Retain recovery/package-management for A; remove rpm or modules only with no-consumer proof and agreed release behavior. | Removal only; no need to stall unrelated fixes. |
| Vendor/kernel/U-Boot/media pin and drone userspace port | W7 coherent compatibility set and actual producer/consumer implementation. | Cohort B and full drone functional acceptance. |
| Saved environment/old-overlay/GOLDEN migration | Fresh spare-media acceptance first; preserve prior state, document migration and rollback before later updates. | Physical cutover, not offline qualification. |
| Physical device/profile and MCU readiness | Identify authorized exact spare device/profile at session time; battery-reset hold/charger A/B separate. | Hardware/destructive work/MCU only. |

No decision is pending about license scope, Yocto generation, rewriting Artoo PORT, keeping source9, or replacing the locker: these are already resolved. Execution of this plan, hardware sessions, commits and publication have not been performed by this document-writing task.

## Complete source-recipe disposition appendix

This appendix joins the historical `recipe-groups.json` to final source9 `package_rows`, rather than reproducing 4,587 binaries. It names every current source unit, including all five new local units, and separately the ten historical removals: **190 distinct recipe names = 180 current + 10 removed**. The 37 originally prioritized families are all routed; the other 148 historical units retain explicit maintenance/retention dispositions. Table PV is current source9 recipe PV (not the full binary package version/epoch); exact package strings, scopes and mapping evidence remain in [final source9 JSON](/home/jresp/solo/.integration/scarthgap-package-update-audit-20261028/source9-final-identity-comparison.json), [historical recipe groups](/home/jresp/solo/.integration/scarthgap-package-update-audit-20261028/recipe-groups.json) and [wave6 groups](/home/jresp/solo/.integration/scarthgap-package-update-audit-20261028/wave6-current-recipe-groups.json).

Routes below are substantive dispositions, not claims of individual CVE review. `Core` means retain selection and consume coherent Scarthgap recipe/backport maintenance in W5/W8, with installed consumers/closure checked; no independent latest-version bump. `Local` means retain owned source, apply the stated behavior task, and verify packaging/runtime contract. `Vendor` means retain A pin and review only as W7 coherent B set or a justified A security backport. `Removed` means already absent; check no unintended reintroduction.

| Current recipe | Source9 PV | Explicit route / disposition |
|---|---|---|
| `3dr-initscript` | `1.0` | Local W4/W8 retain maintenance/splash/log ownership; narrow metadata |
| `3dr-splash` | `1.0.0` | Local W4/W8 retain maintenance/splash/log ownership; narrow metadata |
| `adv7610-edid` | `1.0` | Local W7 actual drone EDID/media graph consumers; no complete-capture claim |
| `alsa-lib` | `1.2.11` | Core W5; W7/W9 controller audio/UCM/vendor integration |
| `alsa-state` | `0.2.0` | Core W5; W7/W9 controller audio/UCM/vendor integration |
| `alsa-ucm-conf` | `1.2.11` | Core W5; W7/W9 controller audio/UCM/vendor integration |
| `alsa-utils` | `1.2.11` | Core W5; W7/W9 controller audio/UCM/vendor integration |
| `apt` | `2.6.1` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `artoo-firmware` | `5.0.0` | P19; W1/W4 retain payloads, integrity/staging; W9 separate hardware |
| `base-files` | `3.0.14` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `base-passwd` | `3.6.3` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `busybox` | `1.36.1` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `ca-certificates` | `20211016` | P13; W5 parser/trust maintenance and consumers |
| `cryptodev-module` | `1.14` | P05; W7 kernel/external-module ABI together; retain A pin |
| `desktop-file-utils` | `0.27` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `dnsmasq` | `2.90` | P09; W3/W5 provisioning/network backports |
| `dosfstools` | `4.2` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `dpkg` | `1.22.0` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `dronekit` | `2.4.0` | P17; retain owned forks; W6 selective API/protocol maintenance |
| `dronekit-solo` | `1.2.0` | P17; retain owned forks; W6 selective API/protocol maintenance |
| `e2fsprogs` | `1.47.0` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `eudev` | `3.2.14` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `expat` | `2.6.3` | P13; W5 parser/trust maintenance and consumers |
| `ffmpeg` | `4.4.1` | P12; retain ABI, W7 vendor review / justified A fix |
| `file` | `5.45` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `firmware-imx` | `8.26` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `fsl-rc-local` | `1.0` | Vendor retain A; W5/W7/W8 init/udev startup compatibility |
| `gimbal-firmware` | `1.3.6` | P19; W1/W4 retain payloads, integrity/staging; W9 separate hardware |
| `glib-2.0` | `2.78.6` | P13; W5 parser/trust maintenance and consumers |
| `glibc` | `2.39+git` | P07; W5 stable libc/locale and rebuilt consumers |
| `glibc-locale` | `2.39+git` | P07; W5 stable libc/locale and rebuilt consumers |
| `gnutls` | `3.8.4` | P13; W5 parser/trust maintenance and consumers |
| `gstreamer1.0` | `1.24.7.imx` | P04; A source security backport / W7 vendor compatibility |
| `gstreamer1.0-libav` | `1.24.0.imx` | P12; retain ABI, W7 vendor review / justified A fix |
| `gstreamer1.0-plugins-bad` | `1.24.7.imx` | P04; A source security backport / W7 vendor compatibility |
| `gstreamer1.0-plugins-base` | `1.24.7.imx` | P04; A source security backport / W7 vendor compatibility |
| `gstreamer1.0-plugins-good` | `1.24.7.imx` | P04; A source security backport / W7 vendor compatibility |
| `hostapd` | `2.10` | P09; W3/W5 provisioning/network backports |
| `imx-alsa-plugins` | `1.0.26+` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-codec` | `4.9.0` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-gpu-g2d` | `6.4.11.p2.10` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-gpu-viv` | `6.4.11.p2.10-aarch32` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-gst1.0-plugin` | `4.9.2+git` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-lib` | `5.9+` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-parser` | `4.9.2` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-vpu` | `5.4.39.3` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `imx-vpuwrap` | `git` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `init-ifupdown` | `1.0` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `init-system-helpers` | `1.66` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `initscripts` | `1.0` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `iptables` | `1.8.10` | Core W5; retain both roles until W4/W6 firewall/NAT consumer review |
| `iw` | `6.7` | P23; W6 retain until consumer/required-fix evidence |
| `libdrm` | `2.4.116.imx` | P20; Vendor W7 coherent i.MX6 set; retain A pins |
| `libnl` | `3.9.0` | P23; W6 retain until consumer/required-fix evidence |
| `libpng` | `1.6.42` | P13; W5 parser/trust maintenance and consumers |
| `libusb1` | `1.0.20` | P14; W6 core-provider/hotplug/USB consumer qualification |
| `libxml2` | `2.12.8` | P13; W5 parser/trust maintenance and consumers |
| `linux-imx` | `6.6.52+git` | P05; W7 kernel/external-module ABI together; retain A pin |
| `lzlib` | `1.14` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `lzo` | `2.10` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `modutils-initscripts` | `1.0` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `mtd-utils` | `2.1.6` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `nano` | `7.2` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `opensolo-access` | `1.0` | NEW Local W2/W5/W8 account/SSH/receiver package contracts |
| `opensolo-controller-ucm` | `1.0` | NEW Local W5/W7/W9 controller UCM/kernel-card contract |
| `opensolo-platform-config` | `1.0` | NEW Local W5/W8 sysctl/udev ignore/Python/init contract |
| `openssh` | `9.6p1` | P01–P03; W2/W5/W8 security/access/transfer |
| `openssl` | `3.2.3` | P01–P03; W2/W5/W8 security/access/transfer |
| `opkg-utils` | `0.6.3` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `packagegroup-opensolo-controller` | `1.0` | NEW Local W4/W6/W8 MACHINE_ARCH/profile/package closure |
| `packagegroup-opensolo-solo` | `1.0` | NEW Local W4/W6/W8 MACHINE_ARCH/profile/package closure |
| `parted` | `3.6` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `persist-logs` | `1.0.0` | Local W4/W8 retain maintenance/splash/log ownership; narrow metadata |
| `pixhawk-green-fw` | `3.5.4` | P19; W1/W4 retain payloads, integrity/staging; W9 separate hardware |
| `pixhawk-stock-fw` | `1.5.4` | P19; W1/W4 retain payloads, integrity/staging; W9 separate hardware |
| `pulseaudio` | `17.0` | Core W5; W7/W9 controller audio/UCM/vendor integration |
| `pymavlink` | `1.1.56` | P17; retain owned forks; W6 selective API/protocol maintenance |
| `python3` | `3.12.6` | P10/P11; W5/W6 same-major runtime/build/development policy |
| `python3-monotonic` | `0.6` | P16; W6 retain NumPy / assess IPC / migrate monotonic then remove |
| `python3-numpy` | `1.26.4` | P16; W6 retain NumPy / assess IPC / migrate monotonic then remove |
| `python3-pip` | `24.0` | P10/P11; W5/W6 same-major runtime/build/development policy |
| `python3-posix-ipc` | `1.1.1` | P16; W6 retain NumPy / assess IPC / migrate monotonic then remove |
| `python3-setuptools` | `69.1.1` | P10/P11; W5/W6 same-major runtime/build/development policy |
| `rpm` | `4.19.1.1` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `rsync` | `3.2.7` | P01–P03; W2/W5/W8 security/access/transfer |
| `shadow` | `4.14.2` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `shadow-securetty` | `4.6` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `shotmanager` | `3.0.0` | P18; Local W1/W2/W3/W6 coordinated interfaces and pins |
| `sololink` | `0.0.0` | P18; Local W1/W2/W3/W6 coordinated interfaces and pins |
| `sololink-python` | `0.0.0` | P18; Local W1/W2/W3/W6 coordinated interfaces and pins |
| `stm32loader` | `1.0` | P18; Local W1/W2/W3/W6 coordinated interfaces and pins |
| `sysvinit` | `3.04` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `sysvinit-inittab` | `2.88dsf` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `ttyrun` | `2.31.0` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `udev-rules-imx` | `1.0` | Vendor retain A; W5/W7/W8 init/udev startup compatibility |
| `update-rc.d` | `0.8+git` | Core W5/W8 plus P appends; account/init/mount/getty/supervision contracts |
| `usbutils` | `017` | P14; W6 core-provider/hotplug/USB consumer qualification |
| `util-linux` | `2.39.3` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `util-linux-libuuid` | `2.39.3` | P08; W5/W8 recovery/core tools; retain necessary initramfs closure |
| `v4l-utils` | `1.26.1+git` | Core W5 package maintenance; W7/W9 actual V4L2 graph consumers |
| `vim` | `9.1.0698` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |
| `wireless-tools` | `30.pre9` | P23; W6 retain until consumer/required-fix evidence |
| `wpa-supplicant` | `2.10` | P09; W3/W5 provisioning/network backports |
| `wtmp-rotate` | `1.0.0` | Local W4/W8 retain maintenance/splash/log ownership; narrow metadata |
| `zstd` | `1.5.5` | P22; Core W5 plus W6 release/development closure; remove only with consumer evidence |

The remaining **75 current routine units** all have the `Core` disposition (W5 coherent maintenance and scoped CVE triage; W8 dependency/manifest/runtime closure). They are grouped for readability, not omitted or declared CVE-clear:

| Routine current source units, with source9 PV | Disposition |
|---|---|
| `acl` 2.3.2; `attr` 2.5.1; `avahi` 0.8; `bash` 5.2.21; `bzip2` 1.0.8; `cairo` 1.18.0; `db` 5.3.28 | Core W5/W8; retain functionality and test real affected consumers. |
| `dbus` 1.14.10; `flac` 1.4.3; `fontconfig` 2.15.0; `freetype` 2.13.2; `fribidi` 1.0.14; `gcc-runtime` 13.3.0; `gdbm` 1.23 | Core W5/W8; retain functionality and test real affected consumers. |
| `gdk-pixbuf` 2.42.12; `glib-networking` 2.78.1; `gmp` 6.3.0; `harfbuzz` 8.3.0; `icu` 74-2; `kmod` 31; `lame` 3.100 | Core W5/W8; retain functionality and test real affected consumers. |
| `libcap` 2.69; `libcap-ng` 0.8.5; `libdaemon` 0.14; `libedit` 20230828-3.1; `libffi` 3.4.6; `libgcc` 13.3.0; `libgcrypt` 1.10.3 | Core W5/W8; retain functionality and test real affected consumers. |
| `libgpg-error` 1.48; `libgudev` 238; `libidn2` 2.3.7; `libjpeg-turbo` 3.0.1; `libmd` 1.1.0; `libnsl2` 2.0.1; `libnss-mdns` 0.15.1 | Core W5/W8; retain functionality and test real affected consumers. |
| `libogg` 1.3.5; `libpcre2` 10.43; `libpsl` 0.21.5; `libsndfile1` 1.2.2; `libsoup` 3.4.4; `libtasn1` 4.19.0; `libtheora` 1.1.1 | Core W5/W8; retain functionality and test real affected consumers. |
| `libtirpc` 1.3.4; `libunistring` 1.2; `libvorbis` 1.3.7; `libxcrypt` 4.4.36; `logrotate` 3.21.0; `lz4` 1.9.4; `mpg123` 1.32.6 | Core W5/W8; retain functionality and test real affected consumers. |
| `ncurses` 6.4; `netbase` 6.4; `nettle` 3.9.1; `nghttp2` 1.61.0; `orc` 0.4.39; `p11-kit` 0.25.3; `packagegroup-base` 1.0 | Core W5/W8; retain functionality and test real affected consumers. |
| `packagegroup-core-boot` 1.0; `packagegroup-core-ssh-openssh` 1.0; `pango` 1.52.1; `pciutils` 3.11.1; `perl` 5.38.2; `pixman` 0.42.2; `popt` 1.19 | Core W5/W8; retain functionality and test real affected consumers. |
| `psplash` 0.1+git; `python3-pyparsing` 3.1.1; `python3-pyserial` 3.5; `readline` 8.2; `shared-mime-info` 2.4; `speex` 1.2.1; `sqlite3` 3.45.1 | Core W5/W8; retain functionality and test real affected consumers. |
| `taglib` 2.0.1; `wireless-regdb` 2024.07.04; `xxhash` 0.8.2; `xz` 5.4.6; `zlib` 1.3.1 | Core W5/W8; retain functionality and test real affected consumers. |

| Historical recipe now absent | Historical PV | Disposition |
|---|---|---|
| `bluez5` | `5.72` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `ell` | `0.64` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `firmware-nxp-wifi` | `1.0` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `kernel-module-nxp-wlan` | `git` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `linux-firmware` | `20240909` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `mobile-broadband-provider-info` | `20240407` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `neard` | `0.19` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `nxp-wlan-sdk` | `git` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `ofono` | `2.4` | Removed already; P21/W6/W8 verify absence, no new removal task. |
| `rpcbind` | `1.2.6` | Removed already; P21/W6/W8 verify absence, no new removal task. |

U-Boot is a boot artifact and the four P15 native tools are additional maintenance units outside the 180 installed-recipe count. MAVProxy is one of ten locked recipe sources but is not installed. Both are explicitly covered above, not silently counted as packages. Binary splits `opensolo-log-receiver`, `rsync-rrsync`, `sololink-config` and `sololink-led` remain owned by their parent recipes. The new source units are exactly `opensolo-access`, `opensolo-controller-ucm`, `opensolo-platform-config` and the two role packagegroups.

Coverage reconciliation: Grok **17/17**; corrected Muse lead **6/6 new findings and 10/10 recommendation rows**, plus **20/20 specialist new-finding identifiers** (Yocto5, workflow6, BSP5, access4); package audit **15/15 priority rows**, all P2 narrative families, **5/5 work packages**, source-only/wave6/source9 addenda, and **190/190 historical/current recipe-union names**. Every retained/deferred/rejected item has a task or rationale above. No code, package, configuration, existing document, qualified artifact, Git ref, build cache or device was changed to produce this plan.
