# Scarthgap best-practice refactor — October 28, 2026

This is a local refactor of the existing Yocto 5 build, implemented by Astra agents. The frozen candidate uses integration `f485da5fa5f73d6c19afdf0ec35bd7b39ba82a38`, product layer `a83fd2cfaa75eb4587f6081c0c6b6a89ab941712`, BSP `3c179413923ba9f51c99878b1127db846609ce9e` and SoloLink `74e468149e4fde37171d076773d6fa88647a87bd`. Validation results are being collected under `/home/jresp/solo/.integration/scarthgap-refactor-20261028/`.

Poky remains 5.0.4 (`200d12b6a58ad961d60a7774ca0f7a9d29498724`), with NXP 6.6.52-2.2.0, Linux 6.6.52 and U-Boot 2024.04. Yocto 6 is deferred. The [package maintenance audit](scarthgap-package-update-audit-20261028.md) proposes later Scarthgap updates; those updates are not included here. License review/remediation is deferred at the owner's request; changes to existing license declarations made during this pass were reverted.

## Source and package ownership

Controller and drone image recipes now compose shared image policy plus role packagegroups. Provider recipe appends own their machine-specific configuration: base files, network setup, BusyBox, SysV init/inittab, hostapd, wpa_supplicant, DNSMasq and udev. UCM profiles have an explicit package. SoloLink declares concrete binaries/configuration/LED packages rather than claiming the whole root filesystem, restores debug splitting, and declares the runtime dependencies of its configuration scripts.

The drone retains exactly one DataFlash producer and one DSYN log-transfer supervisor in runlevel 4. The additional proposed sender service was removed to prevent duplicate lifecycle ownership. Clock synchronization likewise has one init-service owner.

The role packagegroups declare their machine architecture before inheriting `packagegroup`, matching Scarthgap's immediate architecture snapshot and allowing normal backend package-name resolution. Root account policy uses standard image-level `extrausers` after package postinsts complete shadow conversion; the receiver account retains standard package `useradd`.

The `development` and `release` profiles have separate output directories. Release excludes pip, nano and vim; both profiles require public-key SSH and keep MCU updates staged. Development conveniences do not enable externalsrc or change pinned application selection.

## Board support

A shared Solo i.MX6 machine include replaces inheritance from the entire SABRE SD machine. Actual SoC/tune/provider dependencies and intended features are explicit. Kernel appends target the selected 6.6 recipe and retain the NXP `DELTA_KERNEL_DEFCONFIG` path. Controller `pci=nomsi` and aligned kernel localversion are preserved. Obsolete Atheros option names are removed in favor of the supported mac80211/Minstrel owners. Hidden USB/mux helpers are requested as modules in line with their actual Kconfig selectors, with effective configuration equality checked during qualification.

U-Boot board headers and defconfigs are authoritative. The installed `/etc/u-boot/boot.env` is generated from the same compiled defaults, replacing duplicate hand-maintained environment files. Boot requires successful kernel and DTB loads before `bootm`; a failed LATEST load can proceed to GOLDEN without accidentally booting stale RAM contents. Raw U-Boot writes remain bounded below the environment area.

Drone media configuration now selects the compatible media-controller/IPUv3 bridge dependency chain and correct ADV7610 reset polarity for the pinned driver's logical reset sequence. This does not port the old application pipeline: existing `mfw_v4lsrc`/`mfw_isink` consumers still require a separate media-controller userspace change and physical video qualification. RAM descriptions, early GPIO sequencing, physical reset gestures and board electrical behavior retain the limits in [Board support](board-support.md).

## Source recovery and provenance

`workspace-lock.json` is the canonical authority for vendor/custom repository commits, ten recipe source revisions including kernel/U-Boot, immutable staged firmware and integration-policy hashes. Verification rejects dirty or mismatched build inputs, unavailable recipe commits/branches, changed source URLs and firmware digest mismatches. Local commits enter recipe mirrors only through an explicit seed operation; configure/build never silently fetch an alternate source or publish anything.

A source transport contains self-contained Git bundles, exact reachable version-tag objects, immutable firmware, the source lock and `RECOVERY.md`. It also contains the exact integration checkout needed to restore the workspace. Integration HEAD is recorded separately from the source graph to avoid a self-referencing commit pin. Export requires clean tracked integration content and rechecks it before publishing the manifest. Recovered repositories must work without original object stores.

This is not a complete offline mirror of every third-party recipe archive. Those archives use checksummed upstream downloads or a separately supplied Yocto download directory. Recovery preserves existing dirty work rather than resetting it.

## Image identity and packaging

The image class requires the selected machine's bundled kernel, one DTB, matching U-Boot and this image's squashfs. It rejects ambiguous/generic fallback payloads and oversized bootloaders. The card retains a 4 MiB GOLDEN start, raw U-Boot at 1 KiB and a zeroed saved-environment area at `0xc0000`.

VERSION uses the locked graph identity, selected profile and source epoch. FAT serial/MBR disk identity and FAT timestamps are deterministic; update tar ownership/order/modes/mtime and gzip headers are normalized. A JSON boot receipt binds graph, machine, profile, epoch and exact card/archive/payload hashes. The existing four-member update archive wire format and MD5 sidecar are retained. Offline validation checks the receipt, actual rootfs VERSION, effective kernel fragment/module ABI and generated boot environment. It follows the actual U-Boot import semantics for the two identical repeated NXP manufacturing defaults while rejecting conflicting or boot-critical duplicates. Release validation rejects the explicit development package set.

## SSH and log-transfer cutover

Images no longer embed a shared private key, administrator authorization or peer trust. Standard image-level `extrausers` applies the final root password marker after package postinsts finish shadow conversion; package-level `useradd` owns the receiver account. The controller's dedicated `sololog` receiver uses upstream `rrsync` forced to the intended log directory, with no shell/TTY/forwarding. The sender uses a per-device identity and pinned controller host key, bounds its source snapshot to regular numeric `.BIN` files and reports unavailable transfer while unpaired. `/usr/sbin/opensolo-log-provision` prepares identity, pairing and administrator material explicitly offline.

Use fresh/spare media and a fresh RWFS overlay for first acceptance. A retained overlay can shadow the new SSH configuration or preserve old authorized keys; installing a new squashfs alone does not revoke them. This pass does not wipe, migrate or change the working controller card.

## Qualification and deployment limits

All four frozen images build successfully and pass full image, initramfs, ARM userspace/SSH-policy and ARM maintenance checks. The verification records and exact artifact copies are under [qualified-source9](/home/jresp/solo/recovery/scarthgap-refactor-20261028/qualified-source9). Build graph identity is `36f841398e89201e2e462391cfa2779188349999090295c8b91c967f29380138`.

| Role/profile | Successful build tasks | Main packages | Checked modules | SD image SHA-256 |
| --- | ---: | ---: | ---: | --- |
| Controller development | 6,815 | 971 | 333 | `a9c23f81afa8a1d412b08d791d055601d74b831a925f69e2ea2249f197cb0660` |
| Controller release | 6,780 | 956 | 333 | `0d57c450cac9165ba34ad913b14870176194ebc8776ad4ce6b12e0fffb5861f3` |
| Drone development | 6,840 | 957 | 323 | `f95be255ba4bb63da5908d6bea2ba9e385b98b9becdc1be5bd01a422c6926e1e` |
| Drone release | 6,805 | 943 | 323 | `b106bb4e2cc2d29a3bdf851c1a8d932d1a4bbd1ce7855dad4497cb0a951491f7` |

Relevant unchanged-source and updated-unit results total **82 passing tests**: 52 maintenance fault cases across both roles, 13 source/workspace tests, 11 image-validation tests and 6 secure-transfer tests using actual rsync plus the selected upstream rrsync. All four artifacts pass the full four-check qualification matrix. Target OpenSSH policy is evaluated with disposable host keys, without starting a daemon, writing image keys or touching controller hardware. This does not qualify first-boot host-key generation or real authentication on a device.

All four profiles were regenerated with `-C image`: squashfs, SD image, update archive, MD5 sidecar and boot receipt are byte-identical to their saved pre-regeneration copies. [Reproduction comparison](/home/jresp/solo/.integration/scarthgap-refactor-20261028/repro-comparison.json) records each hash. This demonstrates packaging repeatability from unchanged rootfs and cached dependencies; it is not a second clean toolchain/rootfs build.

All four effective kernel configuration hashes equal their pre-cleanup values. The selected release is `6.6.52-lts-next-ge0f9e2afd4cf`. Module checks cover release/vermagic and imported symbol CRCs, including cryptodev. The [package audit](scarthgap-package-update-audit-20261028.md) independently records 4,587 current main/initramfs package rows across 180 recipes, with no package-version delta from the preceding successfully built refactor wave. Its next-round recommendations remain separate from this acceptance.

Final source recovery passed from [the self-contained transport](/home/jresp/solo/recovery/scarthgap-refactor-20261028/source-transport-scarthgap-refactor/RECOVERY.md) into `/home/jresp/Active_GH/opensolo-scarthgap-qualification-source9/workspace`. All 34 restored Git stores have no object alternates, source/firmware verification and configuration generation pass, and no original build/shared-state cache is borrowed. This verifies exact source recovery, not a complete offline archive mirror or a clean rebuild. [Recovery receipt](/home/jresp/solo/.integration/scarthgap-refactor-20261028/source-recovery-final.json) and [complete qualification summary](/home/jresp/solo/.integration/scarthgap-refactor-20261028/summary.json) record the scope. Earlier failed checks/builds remain in the evidence directory alongside their corrections and source-review receipts. The two meaningful package failures were corrected with direct Bash ownership and standard account/packagegroup ordering; validators and fixtures were aligned with actual pinned vendor/Kconfig behavior rather than weakening boot/profile/ABI checks.

Warnings retained from the frozen vendor base concern the unvalidated Ubuntu 24.04 host and unavailable WSL process-I/O accounting. Forced image regeneration additionally reports the expected tainted-task warning. No build error remains.

The working controller's validated no-MSI kernel and older rootfs/overlay remain unchanged. The selected Yocto Artoo binary is a hash-locked staged candidate from source `60a2707`; the newer primary development firmware is separate and has not been promoted or flashed. The unresolved battery-only STM32 reset/i.MX power event remains a hardware preflight blocker. This refactor performs no SSH hardware checks, SD writes or MCU flashing.
