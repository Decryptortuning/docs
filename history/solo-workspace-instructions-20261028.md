# /home/jresp/solo — OpenSolo (3DR Solo) Scarthgap Port Workspace

This directory is a multi-repo workspace for porting the OpenSolo / 3DR Solo software stack to a modern Yocto **Scarthgap** build for the **i.MX6 Solo** “sololink” boards (controller + drone). It mixes:

- Upstream app repos (under `apps/`) on a `PORT` branch where Python 3.x and modernization work happens.
- Yocto layers (under `workspace/layers/`) that define images, machines, kernels, and U‑Boot integration.
- A full Yocto checkout + build trees (under `workspace/yocto/`) used to build and generate SD card images.
- A reference checkout of the upstream project (under `OPENSOLO_REFERENCE/`) for comparison only.

## Repo Layout (what lives where)

### App git repos (`/home/jresp/solo/apps/`)
- `apps/sololink` (branch: `PORT`) — core SoloLink software, includes `STM32Loader/checkArtooAndUpdate.py` used on boot.
- `apps/sololink-python` (branch: `PORT`) — python package(s) used by the system.
- `apps/shotmanager` (branch: `PORT`) — ShotManager app; has Python 3.9+ compatibility fixes.
- `apps/stm32loader` (branch: `PORT`) — `stm32loader.py`; ported to Python 3 (bytes-safe serial I/O).
- `apps/artoo-firmware` (branch: `PORT`) — firmware + python tooling under `tools/` (ported to Python 3).

### Yocto layers (`/home/jresp/solo/workspace/layers/`)
- `workspace/layers/meta-3dr` (branch: `PORT`) — image recipes (e.g. `3dr-controller.bb`), and app recipes (sololink/shotmanager/stm32loader).
- `workspace/layers/meta-solo` (branch: `PORT`) — additional Solo layer content.
- `workspace/layers/meta-solo-bsp` (branch: `main`) — BSP bits (U‑Boot/kernel DTS/defconfigs, machine/distro definitions):
  - Recent fixes include `CONFIG_SYS_BOOTM_LEN` increase, MMC env device mapping, and console move to UART1.

### Yocto checkout + build trees (`/home/jresp/solo/workspace/yocto/`)
- `workspace/yocto/imx-yocto-bsp/` — repo-tool managed Yocto checkout (do **not** commit to `.repo/`).
  - `sources/` — upstream layers (often detached HEAD; treat as vendor code).
  - `build-controller/` — controller build directory.
  - `build-solo/` — drone build directory.

### Cache + reference
- `yocto-cache/` — shared `downloads/` and `sstate-cache/` (safe to keep, don’t commit).
- `OPENSOLO_REFERENCE/` — cloned upstream OpenSolo reference (not part of the active build; use for diffing).

## Branching / “Are we building the right code?”

Most app repos are built from `branch=PORT` in their recipes. Some recipes pin a specific commit via `SRCREV`.

When you push new commits to the app repos, ensure Yocto recipes reference them:
- `workspace/layers/meta-3dr/recipes-sololink/shotmanager/shotmanager.bb` → update `SRCREV` when needed.
- `workspace/layers/meta-3dr/recipes-sololink/sololink/sololink.bb` → update `SRCREV` when needed.
- `workspace/layers/meta-3dr/recipes-sololink/stm32loader/stm32loader.bb` uses `SRCREV = "${AUTOREV}"` (builds latest `PORT` unless changed).

Quick checks:
- `git -C apps/<repo> status -sb`
- `git -C workspace/layers/<layer> status -sb`
- In a build env: `bitbake-layers show-recipes <name>` and `bitbake -e <recipe> | rg ^SRCREV=`

## Yocto build: controller (typical workflow)

From `workspace/yocto/imx-yocto-bsp/`:

1. Enter the build environment:
   - `source setup-environment build-controller`
2. Ensure build config matches controller:
   - `conf/local.conf` should set `MACHINE = "imx_controller"` and `DISTRO = "3dr-imx-fb"`.
   - `conf/bblayers.conf` must include `workspace/layers/meta-solo-bsp` (BSP layer).
   - Note: `imx-setup-release.sh` overwrites `conf/local.conf` and `conf/bblayers.conf` from their `.org` backups on each run; if you use it, update both the live file and the `.org` copy.
3. Build:
   - `bitbake 3dr-controller`
4. Outputs land in:
   - `build-controller/tmp/deploy/images/imx_controller/`
   - Key artifact: `3dr-controller-imx_controller.rootfs.sdcard`

## SD card image + partitioning (controller card)

The Yocto `*.rootfs.sdcard` image contains U‑Boot plus the **GOLDEN** boot partition image.

The controller’s expected layout is:
- `p1` `GOLDEN` (vfat) — kernel + dtb + squashfs + u-boot.imx
- `p2` `LATEST` (vfat) — typically a copy of `GOLDEN`
- `p3` `RWFS` (ext3)
- `p4` `LOG` (ext3)

When flashing with `dd`, **always** confirm the target device with `lsblk` first; writing to the wrong disk is destructive.

## Current bring-up status (controller)

### Sanity check: are we booting/flashing the right SD?

Before chasing boot/runtime issues, confirm the physical SD card actually contains the latest Yocto output. A frequent failure mode is “I rebuilt” but the controller is still booting an older SD (or the flasher wrote the wrong device).

Quick check (host, after inserting SD):
- `lsblk -o NAME,PATH,SIZE,FSTYPE,LABEL,MOUNTPOINTS /dev/<sd>`
- `ls -al /media/$USER/GOLDEN /media/$USER/LATEST`
- `sha256sum /media/$USER/GOLDEN/*.squashfs`
- Compare against `workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller/3dr-controller-imx_controller.rootfs.squashfs`

Red flags:
- GOLDEN files have “ancient” timestamps (e.g. 2011) or hashes don’t match the build artifacts.
- `LATEST` mounts but is empty even after the first boot (seed logic not running, or you’re not on the updated initramfs).

Recent root cause: the generated `*.rootfs.sdcard` image was copying an **old squashfs** into the GOLDEN partition, so the controller booted stale rootfs content (module ABI mismatch, missing `/usr/bin/python`, configinit failures). Fix applied in `workspace/layers/meta-3dr/classes/image_types_3dr.bbclass` to **prefer** `${IMAGE_NAME}.squashfs` when building the sdcard image (ensure the updated file is committed).

Symptoms of the old-rootfs problem:
- `cfg80211` / `imx_vdoa` "disagrees about version of symbol" and `modprobe ath9k` failures.
- `env: can't execute 'python'` and missing `/etc/sololink.conf`.
- GOLDEN partition squashfs hash did not match the latest build output.

Recent code changes to keep in mind:
- `apps/sololink/config/configinit` updated to use `/mnt/rootfs.ro` when present and fall back to `/etc/*.orig` for config templates. Commit: `bf11e54` on `Decryptortuning/sololink` `PORT`.
- `workspace/layers/meta-3dr/classes/image_types_3dr.bbclass` updated to prefer `${IMAGE_NAME}.squashfs` when packaging sdcard images. Commit: `3529343` on `meta-3dr` `PORT`.
- Initramfs first-boot partitioning now seeds `LATEST` from `GOLDEN` so U-Boot can boot partition 2 on the next reboot (`workspace/layers/meta-3dr/recipes-core/initrdscripts/files/init`).
- Controller image now installs ALSA UCM profiles (`alsa-ucm-conf`) to avoid `alsa-lib` UCM errors (`workspace/layers/meta-3dr/recipes-core/images/3dr-controller.bb`).

Previously-seen issues (now resolved in healthier logs):
- `/etc/sololink.conf` missing → `checkArtooAndUpdate.py` / `pair_server.py` crashes in `logging.config.fileConfig`.
- `/etc/hostapd.conf` missing → hostapd init fails.
- `env: can't execute 'python'` → missing `/usr/bin/python` shim or legacy `#!/usr/bin/env python` shebangs.
- Kernel/module ABI mismatches (`cfg80211` / `imx_vdoa` "disagrees about version of symbol") → usually an old squashfs got packaged into GOLDEN; verify hashes if this returns.

Latest boot log summary (post-fix):
- Improvements: kernel/modules ABI mismatch resolved; `cfg80211` and `ath9k` load normally and Wi-Fi brings up; `configinit` now generates `/etc/hostapd.conf`, `/etc/wpa_supplicant.conf`, and `/etc/sololink.conf`, and network config completes.

Latest boot log problems (after `configinit` started creating configs):
- U-Boot: `*** Warning - bad CRC, using default environment` → expected on a fresh card until `saveenv` is run (or until env area is pre-seeded); verify `CONFIG_ENV_IS_IN_MMC`, `CONFIG_SYS_MMC_ENV_DEV`, and `CONFIG_ENV_OFFSET` if `saveenv` doesn’t stick.
- U-Boot: `** Invalid partition 2 **` on the *first boot after flashing* is expected → the `.rootfs.sdcard` image initially ships with only `p1` (GOLDEN); initramfs creates `p2–p4` on first boot.
  - If `Invalid partition 2` still appears on the *second* boot, investigate MMC device numbering (U-Boot vs Linux) and verify the card actually has `p2` after first boot.
- U-Boot: `Failed to load 'uImage'` (after partitions exist) → `p2` exists but is empty/missing boot payload; fix is to seed `LATEST` from `GOLDEN` after creating partitions (initramfs `seed_latest_from_golden` in `workspace/layers/meta-3dr/recipes-core/initrdscripts/files/init`).
- Kernel: `Unknown kernel command line parameters "factoryreset=0"` → either drop or handle in userspace.
- Kernel: `mxs_phy ... Data pin can't make good contact.` → USB PHY warning, could be hardware or DT/regulator issue.
- RTC/time: `snvs_rtc ... setting system clock to 1970-01-01` and `hwclock: settimeofday() failed: Invalid argument`.
- ALSA UCM: `snd_use_case_mgr_open` errors for `hw:0` (missing UCM config) → install `alsa-ucm-conf` (added to `workspace/layers/meta-3dr/recipes-core/images/3dr-controller.bb`).
- `jitterentropy: Initialization failed with host not compliant with requirements: 9`.
- After `rotating 3dr logs`, the console shows garbage then U-Boot starts again → unexpected reboot/reset (watchdog or crash) needs root-cause.
  - Likely causes: a script writing binary data to stdout, a process dumping raw bytes to serial, or serial decoding noise.

Latest sdcard image path (controller):
- `workspace/yocto/imx-yocto-bsp/build-controller/tmp/deploy/images/imx_controller/3dr-controller-imx_controller.rootfs.sdcard`

## Agent options (best next steps)

1) **Validate the sdcard image (no hardware needed)**
   - Mount the image GOLDEN partition (4 MiB offset):
     - `sudo mount -o ro,loop,offset=4194304 <rootfs.sdcard> /tmp/solo_sdcard_img_golden`
   - Compare squashfs hash to latest build:
     - `sha256sum /tmp/solo_sdcard_img_golden/3dr-controller-imx_controller.squashfs`
     - Compare with `.../3dr-controller-imx_controller.rootfs-*.squashfs` in `tmp/deploy/images/imx_controller/`
   - Unmount:
     - `sudo umount /tmp/solo_sdcard_img_golden`

2) **Validate the flashed SD card (requires user confirmation)**
   - `lsblk -o NAME,PATH,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINTS /dev/<device>`
   - Check GOLDEN contents and compare hashes against the latest squashfs.
   - Inspect RWFS overlay for stale files:
     - `/media/<user>/RWFS/upper`
   - If RWFS must be wiped, **ask for explicit confirmation** before deleting.

3) **Check configinit inputs**
   - Confirm template files exist inside the squashfs:
     - `/etc/sololink.conf.orig`
     - `/etc/hostapd.conf.orig` (or whatever `configinit` expects)
   - If templates are missing, fix in `apps/sololink` or its recipe packaging.

4) **Rebuild images**
   - For sdcard packaging only:
     - `bitbake -c image_sdcard -f 3dr-controller`
   - For app changes (sololink, etc.):
     - `bitbake -c cleansstate sololink`
     - `bitbake 3dr-controller`

5) **Commit discipline**
   - Commit only in `apps/` or `workspace/layers/` repos.
   - Do not edit Yocto workdirs under `build-*/tmp/work/.../git/`.

## Working practices (important in this tree)

- Routine hardware steps do not require a separate confirmation. Use device detection and the user’s latest status to continue authorized work. If a physical action is still needed, tell the user what to do and resume when the device is available.
- Don’t take destructive/interruptive actions without an explicit confirmation:
  - No `dd`/`mkfs`/`parted`/`wipefs` unless I confirm the exact target device path (e.g. `/dev/mmcblk0`).
  - Don’t kill processes (e.g. something already using `/dev/ttyUSB0`) unless I ask; instead, tell me what has the device open and ask how to proceed.

- Don’t “fix” code inside Yocto build outputs like `workspace/yocto/imx-yocto-bsp/build-*/tmp/work/.../git/`.
  - Those are ephemeral task workdirs and will be regenerated.
  - Make changes in the real git repos under `apps/` or in `workspace/layers/` (recipes/patches).
- Avoid committing generated artifacts:
  - `__pycache__/`, `*.pyc`, build logs, `tmp/`, `sstate-cache/`, `downloads/`, etc.
- Prefer small, descriptive commits per repo/layer.
- If Python changes are involved, sanity check with:
  - `python3 -m compileall -q <dir>`

## Known bring-up notes (from recent debugging)

- U‑Boot `uncompress error -28` during `bootm` is commonly a too-small bootm workspace; fixes landed in `meta-solo-bsp` by increasing `CONFIG_SYS_BOOTM_LEN`.
- Boot media/device numbering mismatches (Boot ROM USDHC instance vs U‑Boot MMC index) can break env/fastboot targeting; a mapping fix exists in `meta-solo-bsp`.
