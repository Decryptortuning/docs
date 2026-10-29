# OpenSolo Wrynose migration plan

October 28, 2026 — proposed implementation sequence, not an execution receipt.

## Scope and frozen reference

Port controller first, then drone, to NXP **LF6.18.37_2.1.0** using
`imx-linux-wrynose/imx-6.18.37-2.1.0.xml`: kernel 6.18.37, U-Boot 2026.04,
and the manifest's exact **Yocto 6.0.2** graph. Qualify a coherent 6.0.4
maintenance delta separately after that vendor base passes. Wrynose is LTS
through April 2030; Scarthgap through April 2028. The audit verified published
6.0.4, 5.0.21 and 6.1; publication dates differed between cached pages and are
not a basis for this plan. See the [audit](yocto-modernization-audit-20261028.md),
[official releases](https://www.yoctoproject.org/development/releases/) and
[exact NXP manifest](https://raw.githubusercontent.com/nxp-imx/imx-manifest/imx-linux-wrynose/imx-6.18.37-2.1.0.xml).

This document is the only planning output. No migration, installation, clone,
build, test, source-pin change, publication or hardware action was executed.
Implementation branches should use `codex/wrynose-<purpose>`. Preserve the existing
`/home/jresp/solo` workspace, artifacts and working media throughout.

The reference is Scarthgap 5.0.4/NXP 6.6.52-2.2.0, GCC 13.3/Python 3.12.6,
explicit SysVinit and the custom MBR boot layout. Distinguish three identities:

| Identity | Meaning |
|---|---|
| Live controller | Older rootfs plus overlays and the qualified `pci=nomsi` kernel; two warm boots and subsequent user cold boot established the current working reference. |
| Offline image | The later full SD image passed offline validation; that entire rootfs was not installed on the controller. |
| Integration `e98a68a` and current lock | Selected sources include loader `a8db3ed`; this complete selection has not been rebuilt and deployed. It is not a qualified image receipt. |

Preserve `.integration/controller-pci-nomsi-20261028/installed.json` and its
26,582,592-byte kernel SHA-256
`57e6f86dcfb1aa9b1c2d8c34dc9409211f5aa688b0dd4e1a37efef5238b39fad`.
The older `before/uImage` with hash beginning `dec1529` is the MSI-problem kernel,
not the preferred rollback. Keep the controller-only `pci=nomsi` policy and
6.6 `KERNEL_LOCALVERSION = "-lts-next"` fix in the reference. Review the 6.18
recipe before deciding its corresponding release suffix; do not copy it blindly.

Artoo image artifact `60a2707` and development source `4803457` stay distinct;
this plan promotes neither. STM32 flashing remains **blocked**: GPIO46 reset
restarts i.MX on battery power and the charger comparison is pending. That is a
separate hardware stream unless basic UART operation prevents controller
acceptance. No ROM-loader entry, reset experiment or STM32 flash belongs here.
No simultaneous Wic/GPT, RAUC, FIT, systemd, mainline BSP or repository mass rename.

## Ownership and isolated layout

Keep two active custom layers. `meta-solo-bsp` owns machine/SoC definitions,
U-Boot, kernel, hardware fragments and board firmware. `meta-3dr` owns
`3dr-imx-fb`, images/packagegroups, application/config packages and
init/update/recovery policy. Application repositories own application code.
The integration repository owns orchestration, CI, validation and receipts.
Historical `meta-solo` stays excluded; archive its status in documentation,
without deleting the recovered checkout.

New Linux repositories use WSL Git and native Linux paths, with repository-named
subfolders. Proposed locations are:

| Path | Purpose |
|---|---|
| `/home/jresp/Active_GH/OPenSolo_2` | Existing integration repository; isolated branch/worktree for implementation when needed. |
| `/home/jresp/Active_GH/opensolo-wrynose/sources/<repository-name>` | New pinned vendor/custom/app checkouts, e.g. `meta-3dr`, `meta-solo-bsp`, `openembedded-core`, `bitbake`, `meta-yocto`, `sololink`. |
| `/home/jresp/opensolo-builds/wrynose/<candidate>/<board>` | Generated configuration, TMPDIR, logs and deploy outputs. Controller/drone and release candidates never share TMPDIR. |
| `/home/jresp/opensolo-cache/{downloads,sstate-cache}` | New shared cache location; existing cache can be a reviewed source/mirror, not silently relocated. |
| `/home/jresp/opensolo-artifacts/sha256/<digest>` | Local content-addressed staging backed by a durable, separately recoverable artifact store. |

Keep Scarthgap and Wrynose source graphs/configuration separate; never point the
old `.integration/common.conf` at new sources. Downloads and sstate may be shared
through their normal signatures/locking, but caches are disposable accelerators,
not source authority. Do not run cache cleanup against another active build.
Use separate empty caches for clean-fetch and reproducibility gates.

## One orchestration authority

Use **kas**, pinned with its Python dependencies in an integration-owned tooling
requirements file. Its repository pins, profile composition and lock support
replace the bespoke Wrynose checkout/configuration generator. Preserve existing
Scarthgap wrappers unchanged. [kas configuration documentation](https://kas.readthedocs.io/en/latest/userguide/project-configuration.html)
describes ordered includes and adjacent `.lock.yml` files overriding repository
commits. Use that mechanism with the following proposed files:

| Integration file | Responsibility |
|---|---|
| `vendor/nxp/wrynose/manifest.xml` and provenance record | Immutable imported upstream evidence, including manifest repository commit, included manifests and hashes. Not a second editable checkout configuration. |
| `scripts/import-nxp-manifest.py` | Resolve the whole manifest graph, including remotes/includes, into the vendor profile and lock; fail on unsupported constructs. |
| `kas/vendor/wrynose.yml`, `wrynose.lock.yml` | Generated vendor URLs/paths/layer candidates and exact imported commits. |
| `kas/product.yml`, `product.lock.yml` | Two owned layers plus app source repositories and reviewed commits. App-only entries activate no layers. |
| `kas/boards/controller.yml`, `drone.yml` | Machine and image targets; no source-pin overrides. |
| `kas/profiles/release.yml`, `development.yml` | Explicit package/debug policy; development does not silently substitute local sources. |
| `kas/wrynose-controller.yml`, `wrynose-drone.yml` | Entrypoints including vendor, product, board and release profiles in that order. |
| `artifacts/artoo.json`, `scripts/bootstrap-wrynose.py` | Binary digest/provenance/location and artifact retrieval plus deterministic generation of app SRCREV overrides from the resolved kas graph. |

Run the native kas CLI in its pinned virtual environment, not kas-container.
The proposed bootstrap interface is
`python3 scripts/bootstrap-wrynose.py --board controller --candidate <receipt-id>`;
it must resolve paths, retrieve artifacts and generate pins without starting a
build. After its wave gate, the intended build entrypoint is
`kas build kas/wrynose-controller.yml`, with `KAS_WORK_DIR` and `KAS_BUILD_DIR`
set by the reviewed wrapper to the isolated source and candidate locations above.
Use the drone entrypoint only in its later wave. These interfaces and environment
settings must be checked against the pinned kas version during implementation.

The resolved kas graph is the sole Wrynose commit authority. Generated
`source-pins.conf` and receipts are derived outputs; no independently edited
Wrynose JSON lock. Existing `workspace-lock.json` remains the historical/current
Scarthgap authority. Do not let an app checkout commit and its recipe SRCREV
silently diverge: bootstrap must compare every generated override with the
resolved product commit and reject AUTOREV/unresolved inputs. Record the
integration commit separately, avoiding a self-referential pin.

NXP now splits BitBake, OE-Core and meta-yocto. Remove Wrynose assumptions about
`sources/poky` in environment discovery and layer paths. Import the exact vendor
graph first, then activate only the proven dependency closure in BBLAYERS;
ML/V2X/Matter/Qt/demo layers are not automatically product dependencies. Every
omission needs provider/append evidence. Record resolved profiles, host/tool
versions, source epoch, artifact digests and license/CVE/SPDX reports with each
candidate. WSL Ubuntu 24.04 is usable but WSL itself is not upstream validated;
use a supported native Linux CI runner. Do not start Docker Desktop or a local
daemon. See [host requirements](https://docs.yoctoproject.org/6.0/ref-manual/system-requirements.html).

## Execution waves and review gates

Every wave ends with settled related work, an immutable combined candidate and
review of actual producers, serializers and consumers **before any build or test**.
Record scope, component commits, contract decisions and permitted verification in
a wave receipt under proposed `reviews/wrynose/`. This follows the owner
[integration gate](/home/jresp/Active_GH/COORDINATION/deliverables/master/INTEGRATION_REVIEW_GATE.md);
it is sequencing, not a blanket extra permission request. If a fix changes the
candidate, re-pin and review the affected contract before rerunning its checks.
The commands below are proposed, not run; new scripts/profiles do not yet exist.

### Wave 0 — recoverable inputs and first implementation batch

Prerequisite: snapshot current source identities and qualified receipts without
modifying the existing workspace. Preserve user-dirty `build-env.md` exactly
(SHA-256 `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`).

First implement only the integration provenance/import/bootstrap files above,
source-availability inventory, artifact manifest and frozen baseline receipt.
Make loader `a8db3edfb6f91b366d0d4990bfe921d1595c2718` recoverable: it exists
in the app checkout but is absent from the old fetch mirror despite the lock's
“seeded” claim. SoloLink `46d677a` and Shotmanager `fc9fd5b` exist in local
mirrors but are not contained in current remote-tracking refs. Remote availability
has not been verified. Correct provenance through evidence; do not auto-push.

Choose reachable reviewed publication or durable Git bundles/mirrors retaining
all selected objects, submodules and required refs. For recipe fetches, prove
branch containment too, or explicitly document a justified fetch-mode change.
Store the selected Artoo binary (currently local/ignored) with digest
`7284dd28ce6a046aa15154c505a09037dbff6e553b0dda60da4428cb4e01a8a5`, source
revision, version and licensing provenance. A durable store needs an owner,
retention policy and restore procedure; its endpoint remains a decision, not an
invented URL. No credentials in Git.

Deliverable: reviewed bootstrap/source recovery contract. After freezing it,
exercise a clean-machine checkout/fetch with no access to `/home/jresp/solo`,
old downloads or ignored binaries. Recover the graph and binary from the chosen
sources; compare every identity. Later, once metadata parses, run the complete
recipe fetch closure with `bitbake --runall=fetch 3dr-controller`. Successful
Git checkout alone does not establish all recipe inputs are available.

### Wave 1 — metadata and product contract port

Rebase owning recipes, not generated `tmp/work` files. Inventory every WORKDIR
reference in `recipes-sololink`, initramfs/config recipes and BSP appends. Adapt
`S = "${WORKDIR}/git"`, direct file installs and `subdir=git` overlays to the
selected fetch/unpack layout; verify actual S/UNPACKDIR and where overlays land.
Do not blanket-substitute paths. Follow the [5.1](https://docs.yoctoproject.org/migration-guides/migration-5.1.html),
[5.2](https://docs.yoctoproject.org/migration-guides/migration-5.2.html),
[5.3](https://docs.yoctoproject.org/migration-guides/migration-5.3.html) and
[6.0 migration guides](https://docs.yoctoproject.org/migration-guides/migration-6.0.html).

Keep SysVinit explicit in `conf/distro/3dr-imx-fb.conf`; audit removed packages,
debug-tweaks and distro defaults. Move controller UCM/config files from
`recipes-core/images/3dr-controller.bb` postprocessing into proposed
`recipes-core/opensolo-config/` packages with SRC_URI/do_install/FILES. Split
SoloLink's `FILES += "/"` into owned packages and compose role packagegroups.
Consolidate dnsmasq appends; remove empty version placeholders and scope U-Boot
deployment hooks to the BSP. Development credentials/USB export are explicit
profile choices and cannot leak into the release image.

Replace the shared AP root identity in both image recipes as a coordinated
transfer contract. The real producer is
`sololink/flightcode/dflog/dataFlashMAVLink-to-artoo.py`: rsync uses a private key,
root destination `/log/solo/dataflash`, `--partial`, `--delete` and disabled host
verification. Proposed replacement: per-device provisioning, pinned peer host
identity, dedicated receiving account and a constrained rsync receiver rooted
only at the intended log directory. Specify ownership, quota, partial/resume,
symlink handling and deletion boundaries; validate options rather than passing
arbitrary SSH commands through a shell. Update sender and controller log readers
with the image policy together. Preserve log transfer behavior; removing the key
alone is not completion. Do not assume `restrict` forbids remote commands;
[OpenSSH documents its limits](https://man.openbsd.org/sshd.8#restrict).

Freeze/review producer-consumer changes, then require deterministic parse and
provider evidence: `bitbake -p`, `bitbake-layers show-layers`, `show-appends`,
`show-recipes`, and `bitbake -e` for images/kernel/U-Boot/apps. Reject dangling
appends, unresolved pins, unexpected providers or source paths into Scarthgap.
Deliverable: effective configuration/provider report and complete fetch receipt.

### Wave 2 — controller BSP and fail-closed image construction

Port `linux-imx_6.6.%.bbappend` to the actual 6.18 recipe match and
`u-boot-imx_2024.04.bbappend` to the actual 2026.04 match; use narrow matching,
not universal wildcards. Review patches, DTS bindings, defconfigs, DCD/DDR,
MMC/environment mapping, UART1, decompression workspace and fragment merge task
ordering against new NXP sources. Extract shared SoC/tune settings into proposed
`conf/machine/include/opensolo-imx6.inc`; replace whole-SABRE inheritance only
after comparing effective values and enumerating real board capabilities.

Guard both kernel and DT loads before boot in `include/configs/3dr_hdtc.h` and
consolidate duplicated `controller-boot.env`/`drone-boot.env` definitions.
In `classes/image_types_3dr.bbclass`, require bundled-initramfs kernel, correct
DTB and one machine-specific U-Boot producer everywhere; remove bare-kernel and
missing-DTB fallbacks. Preserve exact-own-squashfs selection. Review these with
`recipes-core/initrdscripts/files/{init,maintenance.sh}`, `3dr-initscript_1.0.bb`
and integration validators, including archive serialization and machine identity.

Preserve raw U-Boot at byte 1,024; environment offset `0xC0000`, size `0x2000`;
GOLDEN at 4 MiB; LATEST, ext3 RWFS/LOG, squashfs, uImage/initramfs and first-boot
seeding. Raw U-Boot must remain below 785,408 bytes. Recheck load addresses,
expanded initramfs/kernel sizes and bootm memory overlap; a larger 2026 binary
must not silently exceed the ABI. After combined review, build controller and
retain DT schema results, effective fragment/config evidence, kernel release,
Module.symvers and external-module vermagic/CRC agreement.

### Wave 3 — offline and userspace qualification

Port existing `validate-image.py`, `validate-initramfs.py`,
`smoke-controller.py`, `smoke-maintenance.py` and
`tests/test_initramfs_provisioning.py` to candidate paths without weakening their
checks. Freeze their changes with producers before execution. Require exact
embedded payload hashes, MBR bounds, update archive members/MD5, embedded scripts
and ARM BusyBox identity. Exercise missing/corrupt LATEST and GOLDEN components,
wrong machine, truncated archives, write/readback failure, interrupted provisioning
and malformed progress records using regular files/simulation first. Stop rather
than boot stale RAM or format an unrecognized layout.

Qualify GCC 15/Python 3.14/glibc changes across SoloLink, SoloLink-Python,
Shotmanager, DroneKit, PyMAVLink and native extensions. Review real
`flightcode/stm32/{SLIP.cpp,PacketHandler.cpp,packetTypes.h}`, Python message
encoders and Artoo counterpart consumers; preserve byte widths, endian order,
framing and calibration semantics. Host compileall is supplementary: run ARM
userspace probes plus application behavior tests. Loader tests remain offline;
boot startup must not unexpectedly enter flashing during migration acceptance.

Correct GPL/Apache recipe metadata after auditing notices, review Artoo binary
obligations, retain existing SPDX generation and add explicit CVE triage with
version mapping and justified exclusions. Normalize `/VERSION`, timestamps,
FAT identifiers and tar ordering/metadata using the chosen source epoch.
Build twice with independent TMPDIR/sstate to compare release outputs; diagnose
remaining differences rather than calling pinned inputs byte-reproducible.
Deliverable: offline acceptance report, licenses/SPDX/CVE inventory and reproducibility
comparison. See [Yocto reproducibility](https://docs.yoctoproject.org/scarthgap/test-manual/reproducible-builds.html).

### Wave 4 — controller hardware, then drone

Only deploy after authorization identifies the target and spare media; never
turn this document into generic copy-paste disk-writing commands. Preserve the
working card; the user does not want a full-card backup. Retain selected payloads,
configuration/calibration and receipts instead. Verify spare-media identity,
capacity and written hashes before boot.

Controller gates: cold/warm/repeated boot, first/second boot partition behavior,
UART and button/joystick input operation, GPIO/power/shutdown, storage, audio/display,
ath9k AP/station/pairing and sustained operation. Capture actual PCI IRQ mode,
interrupt counters and logs per board; do not infer MSI state from a command-line
flag. Keep the known controller legacy-IRQ policy until a separately reviewed
comparison supports changing it. Unknown reset/power behavior stops affected
hardware checks; no ROM flashing experiment is a migration workaround.

After controller acceptance, freeze a drone wave. Reconcile
`imx6solo-drone.dts` ADV7610 reset polarity with the selected adv7604 driver;
prove physical deassertion, probe and capture. Replace ineffective
`solo-media.cfg` symbols with a complete chosen capture dependency graph and
verify effective IPU/capture configuration. Reconcile physical DRAM, DCD,
`imx_ddr_size()`, U-Boot header and handed-off DT before memory stress. Qualify
external-module ABI independently, flight-controller link, pairing, Shotmanager,
video and log transfer. These are acceptance criteria, not reasons to repair
both 6.6 and 6.18 media stacks unless immediate legacy use requires it.

## Decisions, stops and rollback

| Risk/decision | Required disposition |
|---|---|
| Durable source/artifact endpoint | Choose owner/access/retention before clean-machine acceptance; no automatic publication. |
| New U-Boot size or DDR/boot ABI conflict | Stop hardware promotion; explicit design decision if preserved layout cannot fit. |
| IRQ or power regression | Preserve board-specific evidence; return to qualified Scarthgap card/kernel, not old MSI kernel. |
| Capture model / reset polarity | Decide from selected driver and board evidence before drone hardware qualification. |
| Host pseudo/tar workaround | Keep on Scarthgap; remove from Wrynose only after coherent native-tools evidence. |
| 6.0.4 maintenance | Separate locked graph/review and relevant regression gates after vendor 6.0.2 acceptance. |

On failure retain task log, source/profile hashes and payload receipts, classify
fetch, metadata, toolchain, image-contract or hardware failure, and fix its owning
source in a new reviewed candidate. Do not overwrite the previous build/artifact
set or “solve” failures by relaxing checks. GOLDEN fallback cannot recover every
post-Linux failure; both slots share RWFS and raw bootloader writes have a single
copy. Rollback uses preserved working media and qualified payloads, with any
settings restoration reviewed explicitly. No automatic replay of failed updates.

## Work packages and completion checklist

Planning estimates are effort bands, not calendar promises: input/orchestration
work is several focused engineering days (medium confidence); metadata/product
and controller BSP work each several days to multiple weeks (low confidence);
drone capture and hardware qualification are open-ended until first evidence.
Re-estimate after parse, first controller image and first drone capture gates.

- [ ] Wave 0: first batch reviewed; every selected source/binary recoverable without old workspace/cache; provenance corrected.
- [ ] Wave 1: kas is sole Wrynose authority; explicit profiles and complete fetch/provider closure pass.
- [ ] Wave 2: controller BSP/image contract frozen, built and checked without changing Scarthgap.
- [ ] Wave 3: offline recovery, ARM userspace, ABI, security, license/CVE/SPDX and independent reproducibility gates pass.
- [ ] Wave 4: authorized spare-media controller acceptance passes; drone separately meets media/memory/link criteria.
- [ ] Promotion: immutable receipts and rollback procedure retained; central ownership/build docs updated to the accepted graph.
- [ ] Optional maintenance: 6.0.4 is independently qualified, never relabeled as the initial vendor base.

Definition of done: another supported Linux machine can recover and build the
same selected inputs, explain any byte differences, verify the complete boot and
update payload contract, and reproduce recorded controller/drone acceptance.
Each promoted board has explicit evidence and recoverable rollback. A successful
image build alone does not satisfy this definition.
