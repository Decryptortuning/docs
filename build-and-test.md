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
| `validate-image.py` | Exact kernel/DTB/rootfs/U-Boot payloads; raw boot/env bounds; deterministic FAT/tar metadata; graph/profile receipt and actual VERSION; kernel configuration/module ABI; fresh-image SSH policy and staged Artoo isolation. |
| `validate-initramfs.py` | Embedded scripts equal committed source; correct machine identity; controller initramfs BusyBox equals the ARM probe binary. |
| Unit tests | Provisioning/update failure and resume behavior with simulated physical commands and temporary regular-file raw writes. |
| `smoke-controller.py` | ARM Python/SLIP/configuration checks, DNSMasq syntax and actual target OpenSSH effective policy, including the restricted controller log receiver. |
| `smoke-maintenance.py` | Production archive preflight and raw-file readback using actual ARM BusyBox utilities; no GNU utility fallback. |

The image validator's `--baseline` option relaxes deployment-link and DHCP checks for old saved images. Use the full checks for new output. Validation success describes the checks above; physical boot, radio, pairing and peripheral acceptance remain in the [roadmap](roadmap.md).

Working report files may be overwritten. Preserve an accepted artifact set with its source lock, build source record and validation reports before further source changes. The [current verified snapshot](scarthgap-refactor-20261028.md) already includes these records.

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

## Development and release profiles

The default `development` profile contains bench conveniences. The `release` profile excludes pip, nano and vim, uses separate build directories, and retains key-only SSH. Neither profile embeds a private device identity or enables automatic promotion of the staged Artoo candidate. A development image is not an externalsrc build: application source selection stays immutable in both profiles.

```bash
python3 scripts/configure.py controller --profile release
bash scripts/build.sh controller --profile release
python3 scripts/validate-image.py controller --profile release \
  --deploy /home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller-release/tmp/deploy/images/imx_controller
python3 scripts/validate-initramfs.py controller --profile release
python3 scripts/smoke-controller.py --target controller --profile release
python3 scripts/smoke-maintenance.py --target controller --profile release
```

Use `drone` and `build-solo-release` for the aircraft. Validators require the selected profile in generated configuration, the boot receipt and actual squashfs VERSION. Copying development outputs into a release directory does not qualify them.

New images start without administrator keys, pairing identities or pinned peer keys. Provision these deliberately offline using `/usr/sbin/opensolo-log-provision`; see [current refactor](scarthgap-refactor-20261028.md). Existing RWFS overlays can shadow new SSH policy and retain old keys. Use fresh/spare media with a fresh writable overlay for first acceptance; an ordinary image update does not revoke retained credentials.

## Recover exact sources on another WSL installation

Export from a clean reviewed integration checkout and verified workspace into a new output directory:

```bash
python3 scripts/workspace.py verify
python3 scripts/workspace.py export /absolute/native/path/to/new-source-transport
```

Follow the generated `RECOVERY.md`. It first restores the exact integration Git bundle/revision, then runs that checkout's `workspace.py bootstrap --transport` to recover the locked repositories, reachable version tags, recipe mirrors and immutable Artoo payload. Bundles are self-contained; restored repositories do not use exporting-machine object alternates. Existing dirty checkouts are rejected, never reset automatically.

This transport covers the locked Git inputs and staged firmware. It is not a complete offline mirror of every checksummed recipe archive. Archive downloads still require upstream access or separately supplied Yocto downloads. Shared state is a build optimization, not source authority.

## Changing pinned inputs

1. Make the change in its owning source repository and review/commit it there. The build guard requires clean vendor and custom-layer repositories.
2. Update that repository's `revision` and branch information in the integration `workspace-lock.json`. For an application change, also deliberately update the relevant `recipe_sources` entry or use a future explicit development profile.
3. Update locked integration-policy hashes when changing build helpers/templates. Commit the source lock, then regenerate controller/drone configuration for every profile being qualified.
4. Commit the integration lock/policy changes, wait for related work to finish, pin the combined candidate and review actual producer/consumer behavior. Only then verify configurations and run relevant tests/builds. Dirty tracked build-policy files are rejected. Build receipts separately record the integration commit and tracked status.

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
| Reproducibility | VERSION and image packaging use the locked source epoch; FAT serial, MBR ID and tar/gzip metadata are normalized. Compare actual equivalent outputs before claiming reproducibility; a cached build is not a clean-room rebuild. |

Build logs are saved under `/home/jresp/solo/.integration/logs`, alongside `.sources.json` and native-tar hash records. A failed input guard should be resolved deliberately; silently editing generated configuration defeats that verification.

## Local controller LAN and SSH profile

The development card prepared on October 29 has fresh station mode settings and
its SoloLink Wi-Fi AP disabled, at the owner’s request, to isolate LAN bring-up on the shared radio.
Controller application services remain enabled. LAN DHCP uses hostname
`opensolo-controller`; the system hostname remains `3dr_controller` because
existing role-selection scripts depend on it.

OpenWrt supplied the WLAN configuration through the existing read-only router SSH
alias. Credentials live only in private workstation staging files and the card's
RWFS overlay. They are not embedded in the Yocto recipe, source lock or Git docs.
The October 29 preparation used the installed Wi-Fi helper and canonical templates,
with matching base/config checksum markers and a successful next-boot merge check.
It created a new on-card SSH host identity and installed the existing workstation
administrator public key; its public host key is pinned locally. First physical boot, association, DHCP and authenticated SSH now pass for this fresh card; reboot/long-soak acceptance remains open. This profile is specific to this development card.

The workstation SSH alias is:

```bash
ssh opensolo-controller
```

It uses a dedicated Ed25519 key, public-key root login and a separately pinned
controller host key. Password/keyboard-interactive SSH authentication is disabled
by the card's local drop-in. Existing authorized public keys are preserved.
Private staging/evidence is under `/home/jresp/solo/.integration/controller-lan`.

The root cause and earlier accepted kernel fix are documented in [Controller Wi-Fi kernel fix](controller-wifi-pci-20261028.md). The newly prepared Maintenance A image includes the `pci=nomsi` policy with normal ath9k hardware crypto and no legacy diagnostic module override. Historical MSI/station-only experiments remain in their evidence directories. The October 29 card replaces the old rootfs/overlay; its first controller boot passes the scoped network/SSH checks. STM32 firmware is unchanged. See the maintenance ledger's card handoff for exact evidence.

## Candidate sequencing and offline development

Finish related edits, commit and pin all inputs, then review actual producers and
consumers before running tests, configure checks, parsing or builds. Source9 offline
acceptance is complete for its graph; each repinned candidate needs new identities.

Restored repositories intentionally have detached HEADs. To develop, inspect local
branches, then deliberately create the recipe branch at its locked commit using
`git switch -c <branch> <locked-commit>`. If it exists, inspect it; never force-move
it. Update recipe, checkout inventory and recipe_sources branches together. Explicit
`workspace.py seed` makes committed local changes available without a push.
Bundle origins remain valid offline. Adding a network origin is an explicit later
development choice, not a prerequisite for restore; the same applies to mirrors.
Git transport closure does not imply archive closure or network-free fetching.

SSH host/admin/transfer identities and Wi-Fi pairing are separate provisioning
contracts. A system hostname is not the DHCP client-name configuration. The
platform package owns the udev mount-ignore list; there is no udev append.
Module checks compare the release token and imported CRCs; they do not establish
full vermagic/aliases/exported-CRC equivalence or physical board behavior.


## Maintenance A image identity and qualification

The October29 maintenance candidate uses graph/profile-based versioned image names, with stable deploy links retained. A build-ID-only change must invalidate rootfs, squashfs and SD signatures; do not repair stale identities by editing tmp/work or generated links. The [maintenance ledger](scarthgap-maintenance-a-20261028.md) records actual signature controls, SD-only cache reuse, four-profile packaging repeats and retained failures. These are cached-rootfs/dependency results, not a second clean toolchain build. Normal managed configurations are restored to CVE mode0 after the separately paired CVE report runs. Current CVE findings remain in the ledger.
