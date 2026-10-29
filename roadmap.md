# OpenSolo project roadmap

Status date: October 29, 2026. The current implementation wave modernizes the
owned Scarthgap metadata and workflow while retaining NXP Linux 6.6.52 and the
existing boot/storage contract. Yocto 6 is deferred. License work is deferred
by the user and is not part of this wave's acceptance scope.

Source implementation, offline validation and physical qualification are
separate milestones. The [refactor report](scarthgap-refactor-20261028.md)
records the combined pins, successful four-profile builds and offline/ARM
qualification. Physical acceptance remains separate.

## Implemented source foundation

| Area | Current source ownership and behavior |
|---|---|
| Workspace inputs | Integration owns pinned source selection, bootstrap/export recovery, generated configuration and input checks. Immutable artifacts have explicit identities rather than reliance on an old working directory alone. |
| Layer responsibilities | `meta-solo-bsp` owns common SoC/tune, machines, kernel/U-Boot and board files. `meta-3dr` owns distro, images, packagegroups, configuration and maintenance. Historical `meta-solo` is inactive. |
| Package composition | Board packagegroups compose the images; tracked configuration/UCM/init inputs have recipes. Image-level version identity remains explicit. Required package and QA results belong in the refactor report. |
| Development/release profiles | Both use immutable selected sources and key-only access. Release has separate output directories and omits explicit pip/nano/vim development packages. Receipts and actual rootfs VERSION bind profile, graph identity and epoch. This development profile is not an externalsrc workflow. |
| Boot payloads and determinism | Require bundled kernel, matching DTB and machine U-Boot; guard both slot loads; generate the environment from compiled sources. Selected epoch, stable FAT/MBR metadata, deterministic update tar/gzip and payload receipts replace wall-clock packaging inputs. Independent output comparison remains a qualification gate. |
| Access and transfer | Both roles package key-only SSH without embedded device keys. Controller log transfer uses the constrained sololog/rrsync receiver; drone's existing inittab owns one sender lifecycle. Pairing/admin authorization is explicit per-device provisioning. A clean-overlay cutover is required; old overlay keys/policy are not revoked merely by building a new squashfs. |
| Artoo handoff | Yocto stages the selected immutable candidate and manifest under `/firmware/candidates`, outside the automatic updater's `/firmware/artoo_*.bin` search. Native firmware and software-model evidence remain distinct from device programming authority. |
| Recovery checks | Existing partition/update failure handling is retained, with both-role fault scenarios and ARM userspace/utility checks available. Do not reinterpret earlier successful tests as results for newly changed sources. |

## Future candidate offline acceptance (latest be98 software checks complete)

After related components are frozen, pin and review the complete producer and
consumer contract before tests/builds. Record provider/append closure, effective
kernel fragments, module release-token/imported-CRC alignment and actual board DTBs.
Run image/initramfs checks against the selected development or release build,
never the other profile's kernel or rootfs. Check compiled environment failure
branches, exact packaged hashes, deterministic metadata, staged Artoo isolation,
access package composition and effective target sshd policy.

The source mirror/export gate must demonstrate recovery of every selected input
and the selected firmware artifacts. Compare independently regenerated images
before claiming byte reproducibility. Rootfs configuration tests should cover
missing templates, valid existing settings, interrupted generation and invalid
input without using a live controller filesystem.

An optional future externalsrc/devtool workflow must show effective source/task
selection, avoid mutating application checkouts during installation and restore
the pinned fetched source cleanly. It must not silently replace either current
profile's immutable source contract.

## Controller hardware acceptance

The last observed running controller uses graphbe98 with the qualified nomsi kernel,
Wi-Fi/DHCP/key-only SSH and unchanged OpenSolo-4dev MCU firmware. Latest local
software graphbe98 includes the MAVLink/Python modernization and R01–R04 Linux
fixes, with all four profiles qualified in software; see the
[maintenance result](controller-linux-maintenance-20261029.md). Controller
development now needs188MiB GOLDEN versus168MiB on that card. MCU reset readiness remains on hold;
source packaging or an offline test does not lift that hold. Preserve the
[recorded baseline and evidence](status-and-evidence.md) and use the authorized
hardware scope when a session is resumed. Identify the actual storage/serial
device before destructive writes; no blanket per-step confirmation workflow is
introduced here.

1. Capture versioned boot identity, command line, detected RAM and device mapping.
2. Verify first provisioning, the next boot's GOLDEN/LATEST/RWFS/LOG layout and
   exact slot payloads on authorized spare media.
3. Exercise configuration initialization, PCIe Wi-Fi, DHCP, pairing and sustained
   controller operation with the selected candidate.
4. Address the early GPIO and physical factory-reset gaps in
   [board support](board-support.md), preserving known electrical sequencing.
5. Resume MCU reset/communication investigation only within its separately
   established scope. Do not program Artoo merely because a candidate is staged.
6. Exercise update/reset interruptions on disposable media with recovery evidence
   retained. A login prompt alone does not complete board acceptance.

## Drone functional completion

The source refactor selects a coherent media-controller dependency graph and
corrects reset polarity for the pinned adv7604 implementation. It does not port
the legacy GStreamer/capture consumers or establish electrical capture behavior.
Complete graph/input/timing/format ownership and the userspace port described in
[board support](board-support.md#drone-media-pipeline), then qualify capture,
encoding and restart behavior together. Inspect effective kernel configuration
and DT bindings before hardware tests.

Physical DRAM remains unqualified: compare fitted parts and DCD geometry with
U-Boot detection and Linux's handed-off memory range. Retained static sizes are
not evidence of capacity. Qualify flight-controller communication, pairing and
accessory requirements separately from controller results.

## Release and recovery decisions

Define GOLDEN migration, recovery after Linux starts, writable-overlay rollback
and the policy for replacing the sole raw U-Boot copy. FAT durability and that
single bootloader copy remain power-loss limitations. New default environments
also need an explicit policy for devices retaining saved raw environments.

Retain vulnerability/build evidence and resolve findings according to the owned
release policy. A locally built bring-up image is not a production qualification.
License review remains deferred for the user's later pre-publication work.

## October29 next hardware session

The approved controller SD now contains be98 with fresh188MiB GOLDEN/LATEST and offline LAN/SSH provisioning; image readback, payload and filesystem checks pass. First boot is pending. Verify actual VERSION, stable Wi-Fi/SSH and expected MCU identity, then R01 dedicated log routing, R02 effective rotation arguments, R03 clock-source/owner/runlevel behavior and R04 absence of console ownership warnings. Keep STM32 flashing, watchdog arming and memory/stack tuning separate from this Linux boot check. See the [card preparation ledger](scarthgap-maintenance-a-20261028.md#october29-be98-controller-card-prepared--boot-pending).

## Chrony physical follow-up

Be98 now boots and has scoped network/bridge/logging/console evidence, but its clock client fails on an absent install utility. Corrected local graphfaf and stronger actual ARM utility/lifecycle checks pass in all four profiles. The [startup report](chrony-boot-fix-20261029.md) records the prepared SSH-only patch and pending Linux reboot authorization. Do not describe R03 as physically accepted until that deployment and real network time synchronization are observed.
