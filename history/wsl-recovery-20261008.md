# Archived WSL recovery notes — October 8, 2026

This is the integration README before the central project guides were created. It preserves the recovery chronology; use the [central documentation](../README.md) for current status and procedures. Commands below assume the integration repository at `/home/jresp/Active_GH/OPenSolo_2`. The board-audit link has been adjusted for this archive location.

---

# OpenSolo WSL recovery

This repository records the integration of the recovered December 2025 Scarthgap port. Source, downloads, shared state, and builds live in the native WSL filesystem at `/home/jresp/solo`. The original Windows archives remain untouched.

`workspace-lock.json` records all ten project repositories, seventeen vendor repositories, recipe source pins, and preserved image hashes. Application pins come from the archived download mirrors; the unused MAVProxy pin was checked on GitHub. The controller image installs the layer's existing Artoo firmware binary; it does not compile the `apps/artoo-firmware` checkout.

Preserved histories, uncommitted changes, configurations, and old build outputs are under `/home/jresp/solo/recovery/baseline-20261008`. Shared caches remain at `/home/jresp/solo/yocto-cache`. No old image should be mistaken for a new build output.

The host dependency installer is `scripts/install-host-deps.sh`; run it with `sudo` inside Ubuntu WSL on a new host. It was already applied on this machine. Run builds as the normal WSL user.

Run from this repository in WSL:

```bash
python3 scripts/configure.py controller
bash scripts/build.sh controller -p      # Parse metadata
bash scripts/build.sh controller -n      # Resolve the task graph
bash scripts/build.sh controller         # Build the image
python3 scripts/validate-image.py controller \
  --deploy /home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller
```

Replace `controller` with `drone` to configure or build the drone target. `SOLO_WORKSPACE` selects another native WSL workspace for the build wrapper; pass the same directory with `configure.py --workspace`. Configuration generation verifies vendor and custom layer commits and cleanliness before writing files, and saves the previous build configuration. The build wrapper checks the lock and every managed config file again, and records its verified inputs alongside the log. Commit intentional layer changes and update the integration lock before rebuilding. Build logs are saved in `/home/jresp/solo/.integration/logs`.

Parallelism follows CPU affinity automatically. This machine exposes the Ryzen's 16 cores and 32 threads to WSL; both BitBake task concurrency and Make use 32 threads. CPU, memory, and I/O pressure limits defer additional tasks during contention. WSL currently has 30 GiB RAM and 8 GiB swap, out of the Windows host's 64 GB. Resource settings follow [Yocto's concurrency guidance](https://docs.yoctoproject.org/5.0.12/ref-manual/variables.html#term-BB_NUMBER_THREADS). The successful comparison build used this full CPU policy.

The recovered `auto.conf` masks matched no recipes and its local Sololink override was inactive. The baseline profile removes those misleading settings and builds pinned fetched sources. Local application edits require an explicit development profile later; editing `apps/sololink` alone does not change this build. Generated configuration is copied onto the native filesystem so BitBake need not read configuration from the Windows filesystem.

Ubuntu's `tar 1.35+dfsg-3ubuntu0.4` uses direct `openat2` syscalls that the pinned pseudo packaging tool cannot track. This caused `Cannot mkdir: Bad address` during package creation and was reproduced with a small nested archive outside BitBake. The build wrapper bootstraps the pinned Yocto `tar-replacement-native` recipe and keeps a persistent copy under `.integration/tools/tar-1.35`. It exports that directory into PATH and preserves/recreates generated tool links when they point elsewhere. It verifies BitBake selected the compatible tar before starting the image build. Yocto's default `HOSTTOOLS_DIR` is retained to preserve cache signatures. A wrapper uses Ubuntu's x86_64 loader so cleaning either target's `tmp` cannot remove its runtime interpreter. The same archive extracts correctly with native tar under pseudo. The native tar hash is saved with each build log.

The image validator reads image files, copies their GOLDEN payload to a temporary directory, and compares hashes with this build's kernel, DTB, rootfs, U-Boot, and update archive. It also checks the partition boundary, the versioned SD image link, and the controller DHCP snippet and init-script include path. It never mounts or writes a physical device. `--baseline` allows the old direct image output and skips the known missing controller DHCP configuration when checking recovered outputs.

The active custom layers are `meta-3dr` for distro policy, applications, networking, boot/update integration and images, and `meta-solo-bsp` for board support. The unchanged `3dr-imx-fb` distro moved into `meta-3dr`; both layers now declare their dependencies. `meta-solo` remains in the recovered repository inventory but is omitted from active `BBLAYERS`. The BSP still depends on the product layer's initramfs and init policy; making it independently reusable is future work. Obsolete generic recipe copies remain for a later, deliberate comparison.

The new controller comparison build completed all 7,039 tasks successfully, with 6,935 already current. Its SD image and update archive passed payload validation. A checksum-verified deployment copy is preserved under `/home/jresp/solo/recovery/new-baseline-20261008`. Unlike that new controller baseline, the archived drone SD image's embedded rootfs does not match its archived deployed rootfs; regenerate it before using it for bring-up.

The DHCP/staging build also completed all 7,039 tasks successfully, with 7,012 already current. Its SD image passed all validator checks, including DHCP and the versioned deployment link. Its kernel, DTB and U-Boot hashes are unchanged from the comparison build. A verified copy is preserved under `/home/jresp/solo/recovery/controller-fixed-20261008`. The image deployment manifest owns the SD image, stable link, update archive and checksum. This pinned Yocto release disables complete-image cache creation by default (`SSTATE_SKIP_CREATION:task-image-complete = '1'`); ordinary recipe cleanup removes those owned outputs and regeneration reuses package shared state. The recovery profile retains that default.

The cleanup/regeneration check completed successfully with 7,028 of 7,039 tasks already current, and the regenerated image passed all payload, DHCP and deployment checks. Comparison validation reports are `/home/jresp/solo/.integration/controller-regenerated-validation.json` and `/home/jresp/solo/.integration/effective-metadata-validation.json`. Both targets resolve the same source pins and 32-thread settings; the DHCP snippet is present only for the controller. Drone metadata parses successfully, but a new drone image and hardware tests remain pending. Inputs are pinned, but a bit-for-bit reproducible release remains future work: rootfs postprocessing writes the wall-clock build date into `/VERSION`, and regenerated rootfs and SD hashes differ. `/etc/timestamp` is subsequently normalized by Yocto and is unchanged in these two images.

Recovery proceeds in this order:

1. Preserve the recovered state and establish a pinned controller baseline.
2. Restore controller DHCP and stage image outputs through Yocto's image deployment directory; verify the complete image payload.
3. Test controller boot and networking with confirmed hardware steps.
4. Complete drone capture support, ADV7610 reset/routing, and effective kernel configuration.
5. Tighten recipe/package ownership, replace obsolete generic recipe copies with scoped appends, and improve update/provisioning failure handling.

Steps 1 and 2 are complete for the controller. Restartable provisioning and checked update handling are now implemented and tested without hardware. Further controller software work includes configuration initialization/upgrade tests and tighter recipe ownership. Controller boot, DHCP leases and pairing remain the hardware milestone, followed by completing and rebuilding the drone media pipeline.

Provisioning now writes a versioned progress record on GOLDEN before creating or formatting data partitions. After interruption, it verifies the recorded geometry and each completed filesystem before resuming; completed new-provisioning filesystems are preserved. Unknown partial layouts and corrupt records stop for inspection. Seeding checks every machine-specific payload file and excludes the transaction record. Factory/settings reset intentions are recorded before discarding data, and explicit resets can repair a partial layout or damaged RWFS. Normal boot stops on unrecoverable RWFS/LOG errors instead of silently formatting them.

Updates require one archive, the legacy MD5 sidecar, exactly four regular machine-specific payload files, expected image headers, sufficient staging/LATEST capacity, and a bootloader size within the reserved raw area. Extraction and SHA-256 manifest generation occur on LOG before LATEST formatting. Installation checks every copy, the installed hashes, sync/unmount results, and an exact-length raw bootloader readback. Unchanged raw bootloaders skip the write. Installation failures retain UPDATE, record UPDATEFAILED where LOG is writable, and enter the recovery shell. An accompanying settings reset is recorded on GOLDEN only after payload and raw bootloader verification.

These recovery paths apply to cards provisioned from this new baseline. An ordinary update leaves GOLDEN's kernel/initramfs in place, so a legacy card needs a deliberate migration or fresh flash before relying on the new recovery behavior. FAT progress records have no journal, and a power interruption can damage their contents; malformed records stop for inspection. A changed raw U-Boot write still uses a single copy and can be interrupted before Linux recovery is reachable. Automatic rollback after Linux starts remains future work.

Run the hardware-free checks after a completed controller build:

```bash
python3 -m unittest discover -s tests -v
python3 scripts/smoke-controller.py \
  --report /home/jresp/solo/.integration/controller-userspace-smoke.json
python3 scripts/validate-initramfs.py controller \
  --report /home/jresp/solo/.integration/controller-initramfs-source-validation.json
python3 scripts/smoke-maintenance.py \
  --report /home/jresp/solo/.integration/controller-maintenance-arm-smoke.json
```

The provisioning/update tests load production functions and isolated dispatch slices with partition, format, mount and sync commands simulated. Raw write tests are confined to temporary regular files and verify MBR, environment and file size preservation. They cover interruption/resume, corrupt state, wrong geometry, invalid archives/checksums/capacity, reset eligibility, failed copies/unmounts/writes, installed hash and raw readback mismatches, and retained update requests. The initramfs validator compares deployed scripts with committed source and checks the embedded machine identity and BusyBox binary. The maintenance smoke script uses actual ARM BusyBox through QEMU for the production update archive preflight, checksum manifest, archive link rejection, and aligned/unaligned raw-file writes. It uses a host shell harness and performs no filesystem mounts or formats.

The userspace smoke script uses the build's native QEMU to run controller ARM userspace: DNSMasq configuration validation and Python dependency/configuration/SLIP packet checks using an in-memory serial object. It does not start the DHCP daemon or test the board's peripherals. The initial static startup review found all 126 startup links and inittab commands present, with no syntax errors in 82 installed Python scripts and 64 shell scripts. Optional developer tools have unresolved imports, and the imported nonexecutable `usb.py` has an incorrect shell shebang; these are recorded separately in `/home/jresp/solo/.integration/controller-readiness-static.json`.

The earlier provisioning guard build completed all 7,039 tasks successfully, with 7,004 already current. Its image passed payload/DHCP/deployment validation; its image report remains `/home/jresp/solo/.integration/controller-provisioning-guards-validation.json`.

The current maintenance build also completed all 7,039 tasks successfully, with 7,004 already current. All 26 fault tests and the ARM userspace/maintenance probes passed. The actual minimal BusyBox lacked the advanced `od` and `head` options initially used; header checks now use its available `hexdump`, and raw readback uses full sectors plus an exact partial sector. The BusyBox fragment enables only the required checked `dd` conversion/flush features. ARM probes cover 511-, 512- and 513-byte candidates and the production 580,608-byte bootloader; unit tests also cover short reads and the maximum boundary before the environment. The deployed init and maintenance library match committed source, and initramfs BusyBox matches the tested ARM binary.

The final SD image and update archive passed all payload, DHCP and deployment checks. DTB and U-Boot hashes remain unchanged; the bundled kernel carries the new initramfs. A checksum-verified copy of all eight deployment artifacts, the source lock and validation reports is preserved under `/home/jresp/solo/recovery/controller-maintenance-20261008`. Current reports under `/home/jresp/solo/.integration` are `controller-maintenance-validation.json`, `controller-initramfs-source-validation.json`, `controller-maintenance-arm-smoke.json`, `controller-userspace-smoke.json`, and `controller-maintenance-summary.json`. Drone metadata parses with the shared maintenance changes; a new drone image remains pending.

The [board support audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md) records the outstanding hardware and updater findings. Layer cleanup follows the [Scarthgap layer guidance](https://docs.yoctoproject.org/5.0.12/dev-manual/layers.html#following-best-practices-when-creating-layers). The recovered vendor layer set is retained. Ubuntu 24.04 triggers a validation warning in this pinned vendor release; that warning is retained in the build logs. The host kernel also lacks process I/O accounting, limiting that part of build statistics. The preserved baseline reports existing obsolete-license QA warnings for SoloLink and STM32Loader; their license metadata remains a cleanup item.

Flashing, SD card provisioning on a physical device, serial access, and power cycling require the user's hardware confirmation. Building image files does not establish board runtime correctness.
