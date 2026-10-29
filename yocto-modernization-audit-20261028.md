# Yocto, i.MX and project organization audit — October 28, 2026

## Recommendation

Keep the working controller baseline as the reference. Correct the concrete
fetch, security and image/recovery-contract gaps, then bring up a separate
**NXP Wrynose LTS** candidate. The current two-layer split is sound; improve its
contents and orchestration rather than reorganizing everything before proving
the next BSP. Do not treat a newer Yocto release as a cure for the unresolved
STM32 reset/power behavior.

Three Astra agents reviewed active layer/recipe policy, release migration, and
i.MX BSP/board support. Root reviewed the integration/repository workflow and
cross-checked key findings. This was a source/documentation audit: no new image
builds, tests, source migrations, hardware actions or pushes were performed.
Existing build outputs were inspected as evidence, not regenerated. Source-derived
drone defects below are not claims of measured drone behavior.

## Actual baseline and available upgrades

| Component | Current local selection |
|---|---|
| NXP manifest | `imx-linux-scarthgap`, `imx-6.6.52-2.2.0.xml` |
| Yocto/Poky | **5.0.4 Scarthgap**, `200d12b6a58ad961d60a7774ca0f7a9d29498724` |
| NXP metadata | `rel_imx_6.6.52_2.2.0`, meta-imx `a0cf309d…` |
| Kernel | NXP 6.6.52, `e0f9e2afd4cff3f02d71891244b4aa5899dfc786` |
| U-Boot recipe | 2024.04, `6c4545203d123c246c5d7995f2893959506d28e0` |
| Target userspace | GCC 13.3, Python 3.12.6, glibc 2.39 |
| Init/distro | Explicit SysVinit, `3dr-imx-fb` |
| Active custom layers | `meta-3dr`, `meta-solo-bsp`; historical `meta-solo` is inactive |

The separately checked-out `sources/u-boot-imx` is not the recipe's selected
revision. Source inventory and actual recipe fetch pins must be distinguished.
Likewise, immutable Yocto Artoo artifact `60a2707` and locally qualified Artoo
source `4803457` represent different, deliberately unpromoted candidates.

| Route | Verified availability on the audit date | Judgment |
|---|---|---|
| Scarthgap maintenance | Official table lists **5.0.21**; LTS through April 2028 | Lowest-change userspace maintenance route |
| NXP Scarthgap BSP patch | `imx-6.6.52-2.2.2.xml` exists | Still pins the same Poky **5.0.4** commit; not a Yocto maintenance refresh by itself |
| Wrynose | **6.0.4** published; LTS through April 2030 | Preferred longer-term release family |
| NXP Wrynose BSP | **LF6.18.37_2.1.0**, manifest pins **Yocto 6.0.2**, kernel 6.18.37 and U-Boot 2026.04 | Preferred initial vendor-aligned proof of concept |
| Blacksail | **6.1** release artifacts exist; official table calls it stable through April 2027 | Shorter support horizon; no reason to prefer it over Wrynose for this port |
| Community/mainline BSP | Wrynose i.MX6 support exists | Separate, larger display/media/boot port; not the initial migration |

Release-page snapshots disagreed on publication dates during this audit. Use
the verified versions and support horizons above rather than inferring dates
from the calendar. The official table lists 5.0.21; its individual release-note
retrieval was less complete than 6.0.4/6.1 verification. The NXP manifest is
explicitly **6.0.2**, not 6.0.4: qualifying an additional maintenance delta is a
separate task.

Primary references: [Yocto releases](https://www.yoctoproject.org/development/releases/),
[Wrynose 6.0.4 release notes](https://docs.yoctoproject.org/migration-guides/release-notes-6.0.4.html),
[6.1 release artifacts](https://mirrors.kernel.org/yocto/yocto/yocto-6.1/RELEASENOTES),
[NXP exact Wrynose manifest](https://raw.githubusercontent.com/nxp-imx/imx-manifest/imx-linux-wrynose/imx-6.18.37-2.1.0.xml),
[NXP Scarthgap patch manifest](https://raw.githubusercontent.com/nxp-imx/imx-manifest/imx-linux-scarthgap/imx-6.6.52-2.2.2.xml),
[NXP release notes](https://www.nxp.com.cn/docs/en/release-note/RN00210.pdf).

## Findings to address before release or broader qualification

### A1 — P1: published shared identity still permits controller root commands

[Controller image recipe](/home/jresp/solo/workspace/layers/meta-3dr/recipes-core/images/3dr-controller.bb:229)
installs a drone transfer key in root's authorized_keys. The matching private
key is embedded in `3dr-solo.bb:131`. The team derived its public key with
ssh-keygen and verified the match without printing private material; the
temporary mode-0600 file was deleted.

The current `from="10.1.1.0/24",restrict` option limits source addresses and
disables forwarding/PTY/user rc, but it does **not** prohibit commands. This is
AP-subnet root access for anyone possessing the repository-distributed key,
subject to actual SSH/firewall reachability. Controller password authentication
is already disabled; do not confuse this finding with an empty-root-password
claim. Replace the shared credential with per-device provisioning and constrain
the actual transfer operation using an appropriate account/forced command.
Review the live deployment and image policy together before any rollout.
[OpenSSH option semantics](https://man.openbsd.org/sshd.8#restrict).

### A2 — P1 workflow gap: latest loader pin is absent from the build fetch mirror

[workspace-lock.json](/home/jresp/Active_GH/OPenSolo_2/workspace-lock.json:272)
now selects loader `a8db3edfb6f91b366d0d4990bfe921d1595c2718`. Root verified it exists
in the owned checkout but **not** in
`yocto-cache/downloads/git2/github.com.Decryptortuning.stm32loader.git`.
Remote-tracking refs do not contain it either. Its provenance string still says
the mirror was seeded. This is a gap from the latest loader change, caught by
this pass; no new image build has validated that pin. The prior controller image
and installed standalone loader have separate, preserved identities.

SoloLink `46d677a` and Shotmanager `fc9fd5b` do exist in their local fetch mirrors,
but are also not contained in current remote-tracking refs. This does not by
itself prove remote absence; remote publication was not verified in this pass.
The workspace currently relies on locally retained objects and artifact stores.
Fix source availability deliberately before the next build, then add a fetch
availability gate and an exportable source mirror/bundle or reachable published
refs. A SHA alone is not a clean-machine recovery procedure.

### A3 — P2: rootfs postprocessing bypasses input tracking and package ownership

[Controller UCM installation](/home/jresp/solo/workspace/layers/meta-3dr/recipes-core/images/3dr-controller.bb:184)
reads `${THISDIR}/files/...` directly without corresponding SRC_URI/checksum
dependencies. Editing those files alone need not invalidate the relevant task.
Many image configuration changes are also not owned by packages. Move UCM and
controller-specific configuration into recipes using SRC_URI, do_install and
explicit FILES. Make the image primarily compose packages and packagegroups.
This improves rebuild correctness, upgrade behavior and SBOM attribution.

### A4 — P2: recovery and boot-payload selection are still permissive

[U-Boot GOLDEN fallback](/home/jresp/solo/workspace/layers/meta-solo-bsp/recipes-bsp/u-boot/files/include/configs/3dr_hdtc.h:56)
runs boot even when GOLDEN kernel loading fails. Stale RAM from an earlier
LATEST load could be combined with a different root/DT. Duplicate environment
files repeat the contract. Guard kernel and DT loads and keep one authoritative
environment definition; later verify failure cases, not just successful boot.

[Image packaging](/home/jresp/solo/workspace/layers/meta-3dr/classes/image_types_3dr.bbclass:138)
falls back from the required bundled-initramfs kernel to a bare kernel, skips
missing DTBs during one packaging stage, and uses generic `u-boot.imx` for FAT
while other outputs use a machine-specific U-Boot path. Require a consistent
kernel/DTB/bootloader set everywhere and record all selected hashes. The strict
selection of this task's own squashfs is already fixed and should be preserved.

### A5 — P2: byte reproducibility is incomplete

Controller recipe lines 15 and 162 introduce wall-clock build/timestamp contents.
Image class lines 137 and 188 create FAT/update archives without fully normalized
metadata. The locked REPRODUCIBLE_TIMESTAMP_ROOTFS does not make those custom
contents deterministic. Use a selected source epoch throughout, deterministic
volume/mtime/archive metadata and stable ordering, then compare independent
clean builds. Immutable inputs and byte-identical outputs are distinct gates.
[Yocto reproducibility](https://docs.yoctoproject.org/scarthgap/test-manual/reproducible-builds.html).

### A6 — P2: license metadata and vulnerability evidence need an owned policy

SoloLink, SoloLink-Python and Shotmanager recipes declare GPLv3 while checking
Apache license text. Audit bundled notices and set accurate SPDX expressions;
do not infer every included file has the same license. Review the binary Artoo
recipe's CLOSED declaration against its actual source/distribution obligations.

SPDX generation is **already inherited** through Poky; reporting it as absent
would be wrong. The issue is accurate inputs and retention. No explicit active
product cve-check/triage policy was found. Enable and retain CVE reports, map
custom versions and document reviewed exclusions; a reported match is not proof
of exploitability. [License tracking](https://docs.yoctoproject.org/scarthgap/dev-manual/licenses.html),
[CVE workflow](https://docs.yoctoproject.org/scarthgap/dev-manual/vulnerabilities.html).

### A7 — P2 organization: accumulated evaluation-board and recipe policy

Both machines require an entire SABRE board definition. Controller removes
irrelevant wireless features; drone still inherits SABRE SDIO/Bluetooth feature
identities. Factor actual Solo common SoC/tune settings and enumerate board
capabilities explicitly. Do not replace inherited configuration without checking
the effective result.

Remove empty version-specific append placeholders rather than wildcarding
everything. Consolidate the two active dnsmasq append locations. Scope the
product layer's unqualified U-Boot deployment hook to Solo/BSP ownership.
Archive inactive `meta-solo` and duplicate stubs clearly. Dormant .disabled
recipes are not proof those versions are installed: for example the current
controller has posix-ipc 1.1.1, not the old recipe's 0.9.9.

Separate development/release profiles. Global `debug-tweaks` is removed by the
controller image, so that image is not automatically password-insecure. Drone
still carries development credentials/private-key and USB-export policies and
needs separate qualification. Split SoloLink's broad `FILES += "/"` package and
machine configuration into useful ownership units rather than moving everything
to new layers merely for naming consistency.

## Drone-specific BSP blockers

These do not invalidate the working controller and have not been hardware-tested.

| Priority | Evidence and consequence |
|---|---|
| P1 | [Drone DTS reset polarity](/home/jresp/solo/workspace/layers/meta-solo-bsp/recipes-kernel/linux/linux-imx/files/arch/arm/boot/dts/nxp/imx/imx6solo-drone.dts:141) is active-low while the pinned adv7604 driver ends its reset sequence at descriptor logical 1. That maps to asserted physical low. Review DT and driver together; predicted HDMI probe/capture failure. The driver does support adi,adv7610. |
| P1 | [solo-media.cfg](/home/jresp/solo/workspace/layers/meta-solo-bsp/recipes-kernel/linux/linux-imx/files/solo-media.cfg:6) requests nonexistent symbols and misses effective prerequisites. Existing drone config disables IMX_IPUV3_CORE and VIDEO_IMX_CAPTURE. Compiled DTS plus an ADV7604 module is not a complete capture stack. Choose vendor legacy or media-controller capture and configure its entire dependency graph. |
| P2 | Drone kernel/U-Boot DTS describe 1 GiB, shared U-Boot header says 512 MiB, and board code uses imx_ddr_size(). U-Boot may fix up the DT, so this is not proof Linux uses nonexistent RAM. Reconcile physical DRAM, DCD geometry, detected RAM and handed-off DT. |
| P2 | Controller's KERNEL_LOCALVERSION alignment has not been applied/qualified for drone external modules. Verify actual release/CRC/vermagic independently. |

Kernel references: the pinned `adv7604.c:3394` reset sequence and `:3468`
descriptor acquisition; `drivers/staging/media/imx/Kconfig:8`; existing drone
`kernel-build-artifacts/.config:4405` and `:5935`.
[GPIO descriptor semantics](https://docs.kernel.org/driver-api/gpio/consumer.html).

Do not spend two full efforts repairing every legacy media detail on 6.6 and
again on 6.18 unless immediate drone bring-up needs it. Use these findings as
explicit acceptance criteria on the selected destination BSP.

## Organization to converge toward

Retain separate application repositories and **two active owned layers**:

| Owner | Contents |
|---|---|
| Integration repository | Checkout/config orchestration, selected source/artifact locks, dev/release and board profiles, CI/tests, artifact receipts |
| meta-solo-bsp | Machines/common SoC include, kernel/U-Boot patches and board descriptions, fragments, hardware-specific firmware |
| meta-3dr | Distro, image/packagegroup composition, app/config recipes, init/update/recovery policy |
| Application repositories | C/C++/Python source and its own tests; Artoo keeps its separate ARM bare-metal build |
| Central docs | Current ownership, build/release procedure and dated evidence |

New developer checkouts belong under `/home/jresp/Active_GH` per user preference;
preserve the recovered `/home/jresp/solo` layout until an explicit transition.
Generated builds, shared DL_DIR/SSTATE_DIR and immutable release artifacts remain
outside source repositories. Different release candidates need separate source
sets, build directories and generated configuration; current shared
`.integration/common.conf` is not an isolated two-release setup.

There is currently no clean checkout/bootstrap command or CI configuration in
the integration repository. The lock inventories seventeen vendor repositories,
but configuration generation assumes they already exist and assumes a qualified
Artoo binary already exists under a local ignored artifact directory. Preserve
artifact selection intentionally while making that store recoverable and
content-addressed from a manifest, with no private credentials committed.

Recommended orchestration direction: evaluate **kas** profiles/lockfiles as the
maintained replacement for bespoke checkout/config generation. It supports
repository commits, layer selection and combined board/product/dev profiles.
Use NXP's exact manifest as the reviewed vendor input, generate/import one pinned
vendor profile, and keep **one** authoritative product graph. Do not maintain
independent handwritten NXP XML, JSON and kas locks that can contradict each
other. Keeping NXP repo plus a product local manifest is also valid if preferred;
the essential requirement is a one-command pinned bootstrap with clean-machine
fetch verification. No kas installation or migration was performed here.
[kas project/lockfile documentation](https://kas.readthedocs.io/en/latest/userguide/project-configuration.html).

Reduce active BBLAYERS to the proven dependency closure rather than blindly
activating all vendor ML/V2X/Matter/Qt/demo layers. Verify providers/appends before
removing any layer. Preserve small patches with rationale and upstream disposition;
send generic fixes upstream when independently justified. Run native WSL builds
for development and use an appropriate native Linux runner for CI; no Docker
fallback is required for this organization work.

## Why Wrynose is a port, not a version substitution

1. NXP now splits `sources/bitbake`, `sources/openembedded-core` and
   `sources/meta-yocto`; current wrappers and BBLAYERS hard-code `sources/poky`.
2. Newer Git unpack layout and UNPACKDIR rules break recipes with
   `S="${WORKDIR}/git"`, direct WORKDIR file installation and `subdir=git`
   overlays. Review each recipe's unpack/install paths.
3. Removed `debug-tweaks`, exact package pins and provider changes require review.
4. `linux-imx_6.6.%` and `u-boot-imx_2024.04` appends will not cover the new BSP.
   Port board files, DDR setup, patches, bindings and fragment symbols.
5. Wrynose's GCC 15.2, Python 3.14 and glibc 2.43 require new application/native
   extension qualification. Existing Python 3.12 tests do not establish 3.14 support.
6. SysVinit remains an explicit product choice; a systemd migration is not
   inherently required. Review removed compatibility recipes and default features.
7. Preserve the boot contract: MBR, uImage+bundled initramfs, GOLDEN/LATEST,
   squashfs, ext3 RWFS/LOG, first-boot seeding, raw environment offset `0xC0000`,
   environment size `0x2000`, and bootm workspace. Do not simultaneously replace
   this with Wic/GPT/RAUC/FIT merely to modernize names.
8. Community/mainline changes machine identity to imx6qdlsabresd and defaults to
   U-Boot SPL/u-boot.img, while this port uses u-boot.imx and NXP display/media
   interfaces. That is a separate route with larger hardware qualification.

References: [5.1 migration](https://docs.yoctoproject.org/migration-guides/migration-5.1.html),
[5.2 migration](https://docs.yoctoproject.org/migration-guides/migration-5.2.html),
[5.3 unpack/Poky changes](https://docs.yoctoproject.org/migration-guides/migration-5.3.html),
[6.0 migration](https://docs.yoctoproject.org/migration-guides/migration-6.0.html),
[6.0 release notes](https://docs.yoctoproject.org/dev/migration-guides/release-notes-6.0.html),
[NXP Solo machine retained](https://raw.githubusercontent.com/nxp-imx/meta-imx/rel_imx_6.18.37_2.1.0/meta-imx-bsp/conf/machine/imx6solosabresd.conf),
[community i.MX6 machine](https://raw.githubusercontent.com/Freescale/meta-freescale/wrynose/conf/machine/imx6qdlsabresd.conf).

Ubuntu 24.04 is explicitly supported by newer Yocto host requirements, unlike
the pinned 5.0.4 host list. WSL2 itself remains usable without upstream host
validation. Review whether the current tar/pseudo workaround is still needed
after selecting a coherent newer pseudo/native-tools stack; preserve it on the
working branch until evidence justifies removal.
[Host requirements](https://docs.yoctoproject.org/6.0/ref-manual/system-requirements.html).

## Suggested execution order and gates

1. **Baseline hygiene:** make loader/app/artifact inputs recoverable; remove the
   shared root identity through coordinated provisioning; package tracked config;
   fail closed on boot components and recovery loads. Preserve working controller
   artifacts, keys and calibration. Do not flash Artoo as part of this wave.
2. **Reproducible workspace:** choose one orchestrator, explicit dev/release
   profiles, clean vendor pins and role packagegroups. Gate: fresh checkout can
   fetch every selected input and recreate configuration without old build/cache
   contents. Later add layer checks, DT schema checks, CVE/SPDX/license retention
   and independent deterministic-image comparisons to CI.
3. **Isolated Wrynose controller candidate:** start with exact NXP
   6.18.37-2.1.0 / Yocto 6.0.2, review combined metadata/board changes before
   building, then evaluate the additional 6.0.4 maintenance delta explicitly.
   Keep live Scarthgap unchanged. If current maintenance is needed first, use a
   coherent Scarthgap layer/core update; NXP 2.2.2 alone is insufficient.
4. **Offline gate:** provider/append closure; effective kernel fragments; module
   ABI/release; all boot payload hashes/addresses/limits; partition/recovery
   failure cases; userspace tests on the new Python/toolchain.
5. **Hardware gate on spare media:** controller cold/warm boot, PCI IRQs and
   ath9k AP/station modes, UART without assuming ROM flashing, storage, shutdown
   and sustained operation. Separately qualify drone capture, flight-controller
   link, pairing and applications. Promote neither image merely because it built.

The current CPU/pressure configuration already uses the available host cores.
Keep it unless new measured build behavior justifies retuning. No exact porting
time estimate is justified before the first new-BSP parse/build and hardware gate.
