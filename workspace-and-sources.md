# OpenSolo workspace and source ownership

The workspace keeps application repositories, product metadata, board support, vendor sources and generated builds separate. The integration repository ties them together through a source lock, configuration templates, scripts and tests.

## Locations on this machine

| Location | Role |
|---|---|
| `/home/jresp/Active_GH/OPenSolo_2` | Integration Git repository ([Decryptortuning/OPenSolo_2](https://github.com/Decryptortuning/OPenSolo_2)), build wrappers, `workspace-lock.json`, tests and audit. |
| `/home/jresp/solo/apps` | Recovered application checkouts; the original Artoo copy is preserved here. |
| `/home/jresp/Active_GH/artoo-firmware` | Primary rebased Artoo MCU firmware checkout and native WSL build. |
| `/home/jresp/Active_GH/OpenSolo` | Canonical upstream clone for component-history extraction; original reference remains preserved. |
| `/home/jresp/solo/workspace/layers/meta-3dr` | Product layer: distro policy, applications, networking, image generation and boot/update scripts. |
| `/home/jresp/solo/workspace/layers/meta-solo-bsp` | BSP layer: machines, kernel device trees/configuration and U-Boot board support. |
| `/home/jresp/solo/workspace/layers/meta-solo` | Preserved historical layer; omitted from active `BBLAYERS`. |
| `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/sources` | Pinned vendor layer/source checkout managed by the recovered NXP manifest. |
| `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-controller` | Generated controller build tree. |
| `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-solo` | Generated drone build tree. |
| `/home/jresp/solo/yocto-cache` | Shared `downloads` and `sstate-cache`. |
| `/home/jresp/solo/.integration` | Generated native configuration, preserved native tar, logs and validation reports. |
| `/home/jresp/solo/workspace/docs` | These central project guides. |
| `/home/jresp/solo/workspace/scripts` | Recovered helper scripts with preserved local changes. |
| `/home/jresp/solo/OPENSOLO_REFERENCE` | Original OpenSolo reference repositories and initialized submodules used for comparison. |
| `/home/jresp/solo/hardware-docs` | Controller/drone schematics, pinout PDFs and BSP research notes. |
| `/home/jresp/solo/logs` and `/home/jresp/solo/artifacts` | Recovered boot logs and older image material. |
| `/home/jresp/solo/recovery` | Original state, configuration history and verified image snapshots. |
| `C:\Users\JResp\Desktop\opensolotemp` | Original Windows archive location. |

The integration repository, source checkouts, builds and caches all use native WSL storage. The integration checkout is `/home/jresp/Active_GH/OPenSolo_2`; the multi-repository build workspace remains `/home/jresp/solo`. Generated configuration is copied to `.integration` before BitBake reads it. The former Windows integration location is no longer used.

## Layer responsibilities

`meta-3dr` owns the `3dr-imx-fb` distro, application recipes, controller DHCP configuration, custom SD/update image generation, initramfs maintenance and product startup integration. `meta-solo-bsp` owns the two machines and their kernel/U-Boot hardware definitions. The shared machine override is `imx-solo`; controller-only behavior uses the controller override.

The product distro has moved out of the BSP, and both layers declare dependencies. The shared machine include now describes actual Solo hardware rather than inheriting the entire SABRE SD machine. Canonical U-Boot defaults generate the installed boot.env; hardware deployment policy belongs to the BSP. Product images compose common policy and role packagegroups; machine configuration is installed by its owning provider recipe. Product initramfs dependencies remain explicit. See [Scarthgap refactor](scarthgap-refactor-20261028.md).

## Application source selection

The main recovered app repositories are `sololink`, `sololink-python`, `shotmanager`, `stm32loader` and `artoo-firmware`. Their configured origins are under `Decryptortuning`. DroneKit, DroneKit Solo, PyMAVLink and MAVProxy also have recipe source entries in the integration lock. MAVProxy is pinned but unused by the controller baseline.

The lock records ten project repositories, seventeen vendor repositories and ten recipe source pins, including explicit kernel and U-Boot commits. `scripts/configure.py` turns the recipe pins into native `SRCREV:pn-*` overrides and verifies vendor/custom-layer revisions and cleanliness before generating configuration.

The baseline builds pinned fetched sources. Editing an app checkout alone does not change the image. The development profile selects bench packages while retaining pinned sources; a separate devtool/externalsrc workflow remains future work. SoloLink installation now copies role-selected files directly into `D` instead of rewriting its source checkout.

The controller packages an immutable hash-locked Artoo candidate selected by `workspace-lock.json`, under `/firmware/candidates`. It does not compile the Artoo checkout or expose the candidate to the automatic updater. The selected Yocto artifact and newer primary Artoo development checkout are distinct until an explicit promotion updates the lock.

## Making changes

| Change | Owning location |
|---|---|
| Controller/drone pins, regulators, peripheral nodes, kernel options or U-Boot board behavior | `workspace/layers/meta-solo-bsp` |
| App recipe/package policy, DHCP, image layout or initramfs maintenance | `workspace/layers/meta-3dr` |
| App logic | Owning app Git repository, then deliberate recipe pin/development-profile selection |
| Build workflow, templates, source lock or maintenance tests | Integration repository |
| Current status, procedures and roadmap | This docs repository |

Edit real source repositories. Files under `build-*/tmp/work` are generated task outputs and will be replaced. Preserve original archives, reference checkouts and recovered local patches.

After an intentional layer commit, update its lock entry and regenerate both targets as described in [Build and validation](build-and-test.md). Keep the preserved dirty state of historical docs/scripts separate from new commits; do not silently include those edits.

New Linux, web, Android and kernel repositories belong under `/home/jresp/Active_GH` in WSL `Ubuntu-24.04`, using WSL Git and native Linux tooling. Repositories requiring native Windows tools use `C:\Users\JResp\Documents\Active_GH`. An explicitly requested destination takes precedence. The restored `/home/jresp/solo` workspace remains in its existing native location.

Build-input repository paths are workspace-relative and portable. Inventory-only entries do not select build inputs. The primary Artoo development checkout is outside the restored workspace; the lock selects the immutable binary and manifest used by Yocto independently. Export transports include exact integration code, pinned repositories/recipe mirrors and firmware, with recovery instructions; see [Build and validation](build-and-test.md#recover-exact-sources-on-another-wsl-installation).
