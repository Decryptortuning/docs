# OpenSolo build and validation

Run the pinned recovery build through the integration scripts from a normal Ubuntu WSL user. The scripts verify source inputs and managed configuration before invoking BitBake. These commands build and inspect files; a physical hardware session is a separate milestone.

## Resume the controller build

Run inside WSL:

```bash
cd /home/jresp/Active_GH/OPenSolo_2
python3 scripts/configure.py controller --check
bash scripts/build.sh controller -p
bash scripts/build.sh controller -n
bash scripts/build.sh controller
```

`--check` verifies the existing configuration without replacing it. If this checkout was deliberately updated, run `python3 scripts/configure.py controller` to regenerate configuration first. Generation saves the previous configuration under `/home/jresp/solo/recovery/config-history`.

The target is `MACHINE = "imx_controller"`, `DISTRO = "3dr-imx-fb"`, image `3dr-controller`. Outputs are under `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller`.

## Validate the controller output

From the integration repository:

```bash
python3 scripts/validate-image.py controller \
  --deploy /home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller \
  --report /home/jresp/solo/.integration/controller-maintenance-validation.json
python3 scripts/validate-initramfs.py controller \
  --report /home/jresp/solo/.integration/controller-initramfs-source-validation.json
python3 -m unittest discover -s tests -v
python3 scripts/smoke-controller.py \
  --report /home/jresp/solo/.integration/controller-userspace-smoke.json
python3 scripts/smoke-maintenance.py \
  --report /home/jresp/solo/.integration/controller-maintenance-arm-smoke.json
```

| Check | What it establishes |
|---|---|
| `validate-image.py` | MBR/GOLDEN bounds, exact embedded producer payloads, raw U-Boot placement, update archive/MD5, versioned SD link and controller DHCP configuration. |
| `validate-initramfs.py` | Embedded scripts equal committed source; correct machine identity; controller initramfs BusyBox equals the ARM probe binary. |
| Unit tests | Provisioning/update failure and resume behavior with simulated physical commands and temporary regular-file raw writes. |
| `smoke-controller.py` | Selected ARM Python/SLIP/configuration checks and DNSMasq syntax without starting its daemon. |
| `smoke-maintenance.py` | Production archive preflight and raw-file readback using actual ARM BusyBox utilities; no GNU utility fallback. |

The image validator's `--baseline` option relaxes deployment-link and DHCP checks for old saved images. Use the full checks for new output. Validation success describes the checks above; physical boot, radio, pairing and peripheral acceptance remain in the [roadmap](roadmap.md).

Working report files may be overwritten. Preserve an accepted artifact set with its source lock, build source record and validation reports before further source changes. The [current verified snapshot](status-and-evidence.md#verified-controller-baseline) already includes these records.

## Configure the drone

```bash
python3 scripts/configure.py drone --check
bash scripts/build.sh drone -p
bash scripts/build.sh drone -n
```

The drone target is `imx_drone`, image `3dr-solo`, build directory `build-solo`. Build and validate it from the integration repository:

```bash
bash scripts/build.sh drone
python3 scripts/validate-image.py drone \
  --deploy /home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-solo/tmp/deploy/images/imx_drone
python3 scripts/validate-initramfs.py drone
```

Controller metadata and common maintenance code do not establish drone capture support. Complete the [drone board work](board-support.md#drone-media-pipeline) before treating a generated drone image as a hardware candidate.

## Regenerate an image to check packaging

A normal build can succeed with every task already current. To exercise controller rootfs creation, squashfs, SD packaging and update archive production again while reusing compiled packages:

```bash
bash scripts/build.sh controller -C rootfs
```

This deliberately invalidates the rootfs task and runs its dependent image tasks. BitBake reports a forced-task/tainted-hash warning for this request. Validate and preserve the resulting artifact set; this is image regeneration with cached dependencies, not a full clean rebuild.

## Changing pinned inputs

1. Make the change in its owning source repository and review/commit it there. The build guard requires clean vendor and custom-layer repositories.
2. Update that repository's `revision` and branch information in the integration `workspace-lock.json`. For an application change, also deliberately update the relevant `recipe_sources` entry or use a future explicit development profile.
3. Run `python3 scripts/configure.py controller` and `python3 scripts/configure.py drone` to update both targets and the native lock copy.
4. Verify both with `--check`, then run relevant metadata/build/validation checks. Commit the integration lock change with its reason and evidence.

The baseline builds fetched pinned application sources. A local app commit must be available to its configured fetcher before a changed pin can build. No automatic publish or development-source override is part of this procedure.

The configured vendor baseline is manifest branch `imx-linux-scarthgap`, file `imx-6.6.52-2.2.0.xml`. Keep its resolved repository commits from the lock when restoring; synchronizing the named branch to a newer state would create different inputs.

## WSL host and CPU settings

The recovered host is Ubuntu 24.04 under WSL2. It exposes 16 Ryzen cores and 32 threads with about 30 GiB RAM and 8 GiB swap. `BB_NUMBER_THREADS` and `PARALLEL_MAKE` use CPU affinity automatically; this host resolves to 32 tasks and `-j 32`. CPU, memory and I/O pressure limits defer additional work during contention. No extra CPU tuning is needed for the current configuration.

Builds and caches live on native WSL storage. `SOLO_WORKSPACE` changes the build wrapper's workspace; pass the same location to `configure.py --workspace`. The image/ARM validation tools also need their corresponding workspace or deployment arguments when using another location.

On a new Ubuntu WSL host, install dependencies with `sudo bash scripts/install-host-deps.sh`, then run builds as the normal user. This installer has already been applied here. The recovered source checkout and caches must also be restored; the installer does not recreate the project.

## Known build behavior

| Condition | Current handling |
|---|---|
| Ubuntu tar/pseudo incompatibility | Patched host tar uses direct `openat2` calls that pinned pseudo cannot track. The wrapper preserves Yocto's native tar under `.integration/tools/tar-1.35`, verifies selection and records its hash. |
| Vendor setup overwrites configuration | Use the integration generator/wrapper. It manages `local.conf`, `bblayers.conf`, `auto.conf` and their `.org` copies. |
| Complete-image shared state | This pinned release disables `do_image_complete` cache creation. Cleanup/regeneration reuses package state and rebuilds final images; a complete-image cache restore has not been demonstrated. |
| Host distribution warning | Ubuntu 24.04 is outside this pinned vendor release's validated host list. Successful builds retain that warning. |
| I/O build statistics warning | The WSL kernel lacks `CONFIG_TASK_IO_ACCOUNTING`; that portion of build statistics is unavailable. |
| Different hashes after regeneration | `/VERSION` includes the wall-clock build date. `/etc/timestamp` is normalized separately. Input pinning alone does not establish bit-for-bit reproducibility. |

Build logs are saved under `/home/jresp/solo/.integration/logs`, alongside `.sources.json` and native-tar hash records. A failed input guard should be resolved deliberately; silently editing generated configuration defeats that verification.

## Local controller LAN and SSH profile

The current development card has station mode enabled and its SoloLink Wi-Fi AP
disabled, at the owner’s request, to isolate LAN bring-up on the shared radio.
Controller application services remain enabled. LAN DHCP uses hostname
`opensolo-controller`; the system hostname remains `3dr_controller` because
existing role-selection scripts depend on it.

OpenWrt supplied the WLAN configuration through the existing read-only router SSH
alias. Credentials live only in private workstation staging files and the card's
RWFS overlay. They are not embedded in the Yocto recipe, source lock or Git docs.
The controller config checksum sidecars were updated so configinit preserves the
changes on reboot. This profile is specific to this development card.

The workstation SSH alias is:

```bash
ssh opensolo-controller
```

It uses a dedicated Ed25519 key, public-key root login and a separately pinned
controller host key. Password/keyboard-interactive SSH authentication is disabled
by the card's local drop-in. Existing authorized public keys are preserved.
Private staging/evidence is under `/home/jresp/solo/.integration/controller-lan`.

Writing the settings mounted RWFS normally, replayed its pending journal, then
cleanly unmounted it. A subsequent read-only e2fsck passed without the previous
allocation-count differences. LAN association and actual SSH access still require
a boot/connection verification; installing the profile alone does not prove them.

The initial combined AP/station boot briefly associated with OpenWrt but did not
complete WPA authentication or obtain a DHCP lease. The card logs show AP startup
falling back to automatic channel selection while station association was still
coming up. This suggests a shared-radio startup conflict, pending verification.
The next profile disables `ApEnable`, retains `StationEnable`, and enables
wpa_supplicant syslog output through a card-local network hook override. SSH
started and listened on port 22 in the failed LAN boot; network reachability and
key login remain unverified. RWFS again passed its check after clean unmount.
