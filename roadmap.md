# OpenSolo project roadmap

The controller's pinned build, DHCP packaging and maintenance failure handling are complete at the current software-test level. Continue with configuration recovery and source/package ownership while arranging a separate controller hardware acceptance session. Drone media completion remains its own larger BSP milestone.

Status date: October 28, 2026. Acceptance items below are pending unless explicitly marked complete.

## Completed foundation

| Milestone | Evidence |
|---|---|
| Preserve the recovered project | Original archives and `/home/jresp/solo/recovery/baseline-20261008` retain histories, patches, configs and old artifacts. |
| Pin and guard build inputs | Ten project repositories, seventeen vendor repositories, eight recipe pins and generated-config checks. |
| Restore layer responsibilities | Product distro in `meta-3dr`, BSP in `meta-solo-bsp`, dependencies declared and `meta-solo` inactive. |
| Restore controller DHCP and image ownership | Verified machine-scoped snippet, `IMGDEPLOYDIR` outputs and exact rootfs producer selection. |
| Rebuild and inspect both image targets | Controller rootfs/image regeneration and new drone image pass producer/update/initramfs validation; October 28 snapshots preserved. |
| Improve maintenance recovery | Recorded provisioning/resets, checked update copies/hashes/unmounts and bounded verified raw writes; 26 fault tests and ARM utility probes passed. |
| Consolidate project documentation | Central status, workspace, build, BSP, recovery and roadmap guides in this repository. |

## Controller software before hardware

### Configuration initialization and upgrade

Create tests against temporary root/overlay fixtures for missing configuration, existing valid settings, interrupted generation, migration from older values and invalid input. Verify `configinit` selects the intended templates and preserves user settings according to an explicit policy. Cover hostapd, wpa_supplicant, SoloLink and related startup ownership.

Acceptance: repeatable tests show first-run initialization and retries produce usable configuration, valid existing settings survive upgrades, and invalid configuration produces a diagnosable failure. No real serial device or controller filesystem is used by these tests.

### Explicit development source profile

Fix SoloLink's source-mutating installation behavior, then add a deliberate externalsrc/devtool workflow alongside the pinned baseline. Show effective `S`, `SRCREV` and task/source selection for a local edit, and show that returning to the baseline selects the recorded fetched revision.

Acceptance: a local change demonstrably enters a development image; the checkout remains unchanged by recipe installation; switching profiles restores the pinned source selection.

### Package and recipe ownership

Give configuration, init integration, UCM profiles and version metadata explicit owning recipes. Review broad `FILES` rules and package-owned rootfs postprocessing. Replace obsolete generic copies such as libusb/monotonic only after comparing and retaining required custom behavior. Correct license metadata and dormant fetcher/bootstrap issues as scoped changes.

Acceptance: packaging ownership and input signatures are clear, relevant QA passes, and controller image/DHCP/maintenance validation remains green after each change.

### Board support preparation

Reconcile controller early GPIO sequencing and physical reset behavior with schematic/board-revision information; consolidate U-Boot environment ownership. Preserve observed direction/level/timing requirements when selecting DTS consumers, regulators or retained early board code.

Acceptance before hardware: reviewed source/board mappings, effective configurations and new image validation. Electrical sequencing and button behavior require the hardware session below.

## Controller hardware acceptance

Start from the [verified baseline](status-and-evidence.md#verified-controller-baseline). Each physical action needs the user's confirmation; identify and confirm the exact storage/serial device before accessing it or writing an image.

1. Resolve host-to-controller UART transmission and capture an uninterrupted current boot log.
2. Verify ROM/U-Boot device mapping, detected RAM and kernel command line against the image being tested.
3. Observe first provisioning, then a second boot with LATEST/RWFS/LOG present and expected filesystem labels/payloads.
4. Verify configuration initialization, controller AP operation, DHCP leases, expected Solo/controller addresses and pairing.
5. Exercise STM32/Artoo communication, relevant HDMI/audio/USB interfaces and planned GPIO/reset behavior.
6. Plan update/reset interruption tests on disposable media, retaining a known recovery image and the selected recovery procedure.

Acceptance: versioned image identity, serial logs and observations demonstrate these behaviors. A login prompt or emulated userspace success alone does not close this milestone.

## Drone completion

The October 28 drone image now builds and passes payload/initramfs validation after the KMS packaging fix. Complete the driver/configuration/graph/input/reset/bus-width work in [Board support](board-support.md#drone-media-pipeline), then rebuild and validate the changed image. Inspect effective kernel options and schema diagnostics before hardware capture testing.

Acceptance: a coherent ADV7610-to-capture pipeline produces the intended formats and encoded video, with verified reset/routing behavior and restart handling. Regenerated SD/update payloads match their exact producers; archived mismatched drone artifacts are not reused as the acceptance image.

## Release and recovery design

Define GOLDEN migration, post-Linux boot failure recovery, writable-overlay rollback and changed raw bootloader update policy. FAT record durability and the single raw U-Boot copy remain explicit power-loss limitations.

Remove wall-clock rootfs metadata where reproducibility is intended, then compare equivalent clean-build outputs. Run release source-resolution, packaging/license, effective kernel, device-tree schema, image/update and board checks. Decide a separate release security policy for the current `debug-tweaks` and update-authentication behavior before claiming a production release.

Acceptance: documented support/migration boundaries, repeatable build evidence and hardware/recovery observations meet the selected release policy. Current software results are a bring-up baseline.
