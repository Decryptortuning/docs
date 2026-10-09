# OpenSolo boot and recovery behavior

The SD image retains the legacy raw U-Boot and four-partition ABI while the initramfs uses modern Linux tools and overlayfs. The current recovery implementation resumes recorded provisioning and checks update installation before clearing its request.

## Boot disk layout

| Region | Placement or size | Purpose |
|---|---|---|
| MBR | Beginning of disk | DOS partition table and boot signature. |
| Raw U-Boot | Starts at byte 1,024 | ROM-loaded bootloader, shared by both boot partitions. |
| U-Boot environment | Starts at `0xc0000` | Saved boot environment; raw payload must end before it. |
| p1 GOLDEN | Starts at 4 MiB | Preserved kernel/initramfs, DTB, squashfs and `u-boot.imx`; recovery progress record. |
| p2 LATEST | Same sector count as GOLDEN for new provisioning | Normal boot/update payload. |
| p3 RWFS | 100 MiB | Writable overlay/settings. |
| p4 LOG | Remaining available space | Logs, update requests and staged update payload. |

A newly generated SD image initially contains p1 only. Its size and current payload are validated before it becomes a hardware candidate; first boot creates p2 through p4. The maximum accepted raw bootloader is 785,408 bytes, leaving the environment boundary intact. The current U-Boot is 580,608 bytes.

The compiled U-Boot defaults try LATEST, then GOLDEN if loading or `bootm` returns. Once Linux starts, that fallback does not provide automatic rollback. Both boot slots use the shared RWFS overlay.

## Required payload names

Both machines use `uImage` and `u-boot.imx`. Additional files are machine-specific:

| Machine | DTB | Root filesystem |
|---|---|---|
| `imx_controller` | `imx6solo-controller.dtb` | `3dr-controller-imx_controller.squashfs` |
| `imx_drone` | `imx6solo-drone.dtb` | `3dr-solo-imx_drone.squashfs` |

The initramfs package writes `/etc/opensolo-machine` from `MACHINE` and uses machine package architecture. Recovery checks derive expected names from this embedded identity.

## First boot and interrupted provisioning

The initramfs reads GOLDEN's `.opensolo-provisioning` record before ordinary partition/payload handling. The record contains a version, operation mode, GOLDEN geometry and a completed-filesystem bit mask; it is parsed as data rather than sourced as shell code.

For an empty data layout, it writes/syncs the record, creates partitions using the current GOLDEN size, and verifies each partition's start/end. Each filesystem format is checked, synced and verified for expected type/label before its completion bit is committed.

On resume, completed new-provisioning filesystems are verified and preserved. A missing recorded step can be retried. Unknown partial layouts, invalid state or geometry/filesystem mismatches stop for inspection. An explicit factory reset can deliberately replace an otherwise unrecorded partial data layout.

Seeding copies GOLDEN's boot content to LATEST, excludes progress records, compares all required payload files, and checks sync/unmount results. The transaction record is cleared after provisioning/seed completion. Ordinary boot never silently reformats a damaged RWFS or LOG filesystem.

## Update installation

Update requests are held under `/mnt/log/updates` in the initramfs, exposed as `/log/updates` to the running product system. UPDATE is processed before generic missing-payload reseeding so a failed installation can retry its original archive.

1. Require exactly one archive and a valid MD5 sidecar naming it.
2. Require exactly four regular expected members; reject duplicates, extra paths, wrong-machine names and links. Check staging/LATEST capacity before extraction/formatting.
3. Extract to LOG staging, check expected image headers and bootloader size, and create a SHA-256 payload manifest.
4. Format/mount LATEST, check each copy, verify installed hashes, then check sync and unmount.
5. Compare the existing raw bootloader with the candidate. Skip an identical payload; otherwise bound its write, flush it and compare an exact-length readback.
6. Clear UPDATE only after verification. Installation failures retain the request, write UPDATEFAILED when LOG is writable, and enter the recovery shell instead of an automatic reboot loop.

The legacy MD5 sidecar checks archive integrity; it is not package authentication. Header checks establish expected formats, while successful physical boot remains a separate acceptance step.

Minimal BusyBox lacks the initially assumed advanced `od` and `head` options. The current implementation uses its available `hexdump` and full-block `dd` reads plus an exact final partial block. A scoped BusyBox fragment enables required `dd` conversion/flush support. Actual ARM utility probes verify this behavior.

## Settings and factory reset

Settings reset preserves LATEST and recreates RWFS/LOG. Factory reset also recreates LATEST from GOLDEN and verifies/restores the GOLDEN raw bootloader when it differs. Both operations record the reset intention on GOLDEN before deleting the affected data partitions and can repeat the intended reset after interruption.

For UPDATE plus RESETSETTINGS, the settings-reset transaction is recorded only after payload and raw bootloader verification, before LOG is discarded. RWFS validation is deferred until after explicit reset handling, allowing a reset to repair a damaged settings filesystem.

The initramfs supports explicit reset requests, but the old physical three-second button gesture has not been restored in U-Boot. See [Board support](board-support.md#controller-and-shared-board-work).

## Recovery limits

An ordinary update replaces LATEST while retaining GOLDEN. Existing cards with an older GOLDEN initramfs need a deliberate migration or fresh flash before relying on the new progress record.

GOLDEN uses FAT and has no journal. Sync/rename sequencing improves normal interruption recovery, but power loss can damage a progress record; malformed records stop for inspection. A changed raw U-Boot write still updates a single copy, so interruption can make both boot partitions unreachable. Automatic recovery after Linux startup failure and independently bootable root/settings slots remain open design work.

Physical flashing, formatting, partitioning, serial access, button actions and power cycling require the user's hardware confirmation. The exact physical device must be identified and confirmed before any destructive write. The software tests in [Build and validation](build-and-test.md) do not perform these operations on a real device.

## Owning source and tests

Maintenance is in `meta-3dr/recipes-core/initrdscripts/files/init` and `maintenance.sh`. The package is defined by `3dr-initscript_1.0.bb`; BusyBox support is in `recipes-core/busybox/files/maintenance-dd.cfg`. Disk-image production is in `meta-3dr/classes/image_types_3dr.bbclass`.

The integration `tests/test_initramfs_provisioning.py` exercises the production functions and safe dispatch slices. `scripts/smoke-maintenance.py` probes the actual ARM utilities; `scripts/validate-initramfs.py` ties embedded scripts and BusyBox back to the source/probe identity.
