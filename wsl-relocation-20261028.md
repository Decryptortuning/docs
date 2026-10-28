# WSL relocation and build verification — October 28, 2026

Both image targets build from the integration repository at `/home/jresp/Active_GH/OPenSolo_2`. The multi-repository source, build and cache workspace remains `/home/jresp/solo`. This verification exercised fresh image production and target userspace without accessing a physical controller or drone.

This record covers build verification before the card was written. The later [controller card preparation](status-and-evidence.md#controller-card-preparation-on-october-28) records the completed flash/readback and the user-requested omission of a backup.

## Located material

All ten project Git repositories and seventeen pinned vendor repositories were located. Active layer/vendor revisions are clean and match the working lock. The preserved local edits remain in `workspace/docs/build-env.md` and `workspace/scripts`.

The [workspace map](workspace-and-sources.md) lists the application checkouts, custom/vendor layers, build trees, caches, docs, hardware schematics, earlier boot logs, original OpenSolo reference/submodules and recovery snapshots. Original split archives remain under `C:\Users\JResp\Desktop\opensolotemp`. The [machine-readable inventory](/home/jresp/solo/.integration/relocation-20261028/inventory.json) records the checked paths and reference submodules.

## Build and validation results

| Check | Result |
|---|---|
| Both target configuration guards | Passed after regenerating managed configuration. |
| Controller rootfs and image regeneration | 7,039 tasks succeeded; 7,030 already current. |
| Controller build at final product revision | 7,039 tasks succeeded; all current; all eight artifact hashes unchanged. |
| Drone image build | 7,026 tasks succeeded; 6,985 already current. Kernel and U-Boot were compiled during this run. |
| Both SD images and update archives | GOLDEN/update payloads match exact deployed producers; raw U-Boot and versioned deployment links checked. Controller DHCP configuration checked. |
| Both initramfs archives | Maintenance source and embedded machine identity match; controller BusyBox matches its ARM probe binary. |
| Controller fault tests | 26 passed with simulated physical commands and regular-file raw writes. |
| Controller ARM checks | DNSMasq syntax, Python/SLIP/pairing configuration, archive preflight and BusyBox raw-file boundary/readback probes passed. |
| Drone KMS package | Effective architecture `imx_drone`, installed Debian architecture `imx-drone`, ARM `libgstkms.so` present. |

The active product revision is `2f5bee6a16480dcea72e00d733d423f13d19ac82`; BSP revision is `cfae891da533653a1a82d69499370d968934a035`. Controller regeneration preceded the drone-only metadata fix; the final-source recheck proves its accepted artifacts stayed unchanged. Build wrappers, source pins and templates retain their existing behavior. CPU affinity exposes 32 threads; the observed pressure controls defer new tasks during contention.

The [summary](/home/jresp/solo/.integration/relocation-20261028/summary.json) links exact build logs, source inputs, hashes and validation reports. Full evidence is under `/home/jresp/solo/.integration/relocation-20261028`.

## Drone package correction

The first drone build compiled its kernel and U-Boot, then failed rootfs creation because `3dr-solo` requested the KMS plugin while NXP's i.MX6 recipe configuration omitted it. The new `gstreamer1.0-plugins-bad_1.24.7.imx.bbappend` in `meta-3dr` enables `kms` for `imx_drone`.

An initial architecture assignment was overridden by `fsl-dynamic-packagearch`. The final append uses the class's recipe-local `MACHINE_ARCH_FILTER` for the drone. The actual package manifest and ELF check verify the result; controller configuration and accepted artifacts are unchanged. The initial build failure and architecture-scope finding are retained in the evidence directory.

This supplies an existing image dependency. It does not complete the ADV7610/IPU capture pipeline or establish that a physical KMS output works.

## Preserved artifacts

| Target | Snapshot | SD image SHA-256 |
|---|---|---|
| Controller | [Verification](/home/jresp/solo/recovery/wsl-relocation-20261028/controller/verification.json) | `bc56f806b3d9f0241ce613d979a044385c24f3836946adf08816dccf40958632` |
| Drone | [Verification](/home/jresp/solo/recovery/wsl-relocation-20261028/drone/verification.json) | `d0fd41ae64198bccdfe6ce4ea882c18471032e3a6c78461c3b1db71b7841e5e7` |

Each target snapshot contains eight copied, checksum-verified deployment artifacts, its source lock/build inputs and validation reports. Earlier October 8 recovery snapshots remain preserved.

## Remaining limits and next step

These are builds with reused compiled/shared-state inputs and fresh image production. A full clean build and bit-for-bit reproducible release remain open. Ubuntu 24.04 host-validation and missing I/O-accounting warnings persist; controller regeneration deliberately creates a forced-task warning. Existing license-metadata warnings appeared when old drone recipes were rebuilt.

The current hardware observation is limited to Windows storage metadata: Disk 4 is a 7,948,206,080-byte USB card with GOLDEN on G: and LATEST on H:. Its existing boot partitions are smaller than the current payload requires. WSL does not directly expose the reader as a block device, and no card writes were performed. Re-identify the card and confirm the exact target before a backup/reflash session; disk numbering can change.

Physical controller boot/network/pairing acceptance and the drone capture/reset/routing work remain in the [roadmap](roadmap.md).
