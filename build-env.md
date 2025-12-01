OpenSolo Yocto workspace bootstrap (Scarthgap 6.6.x)
===================================================

User/paths
- User: `jresp`
- Base: `/home/jresp/solo`
- Yocto tree: `/home/jresp/solo/workspace/yocto/imx-yocto-bsp`
- Custom layers: `/home/jresp/solo/workspace/layers/{meta-solo,meta-3dr}`
- Caches: `/home/jresp/solo/yocto-cache/{downloads,sstate-cache}`
- Artifacts: `/home/jresp/solo/artifacts`

Host dependencies (Ubuntu 24.04)
```bash
sudo apt update
sudo apt install \
  gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio \
  python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping \
  python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev python3-subunit \
  mesa-common-dev zstd liblz4-tool file locales libacl1
sudo locale-gen en_US.UTF-8
```

Install `repo` locally (no sudo)
```bash
mkdir -p /home/jresp/bin
curl -o /home/jresp/bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod +x /home/jresp/bin/repo
export PATH="/home/jresp/bin:${PATH}"
```
Add the PATH line to your shell rc if you want it permanent.

Fetch NXP BSP (Scarthgap 6.6.52_2.2.0)
```bash
cd /home/jresp/solo/workspace/yocto/imx-yocto-bsp
PATH="/home/jresp/bin:${PATH}" repo init -u https://github.com/nxp-imx/imx-manifest.git \
  -b imx-linux-scarthgap -m imx-6.6.52-2.2.0.xml
PATH="/home/jresp/bin:${PATH}" repo sync
```

Yocto build directory setup (once)
```bash
cd /home/jresp/solo/workspace/yocto/imx-yocto-bsp
MACHINE=imx6qsabresd DISTRO=fsl-imx-fb \
  source imx-setup-release.sh -b build-solo
```
Then edit `build-solo/conf/local.conf` to set:
```
DL_DIR ?= "/home/jresp/solo/yocto-cache/downloads"
SSTATE_DIR ?= "/home/jresp/solo/yocto-cache/sstate-cache"
```
and add the custom layers to `bblayers.conf`:
```
/home/jresp/solo/workspace/layers/meta-3dr
/home/jresp/solo/workspace/layers/meta-solo
```
Meta-solo will hold MACHINE/kernel/U-Boot tweaks for Solo; meta-3dr is the upstream product layer.

First reference build (after configs)
```bash
cd /home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-solo
bitbake imx-image-core
```
Later replace `imx-image-core` with Solo-specific images (3dr-solo, 3dr-controller) once recipes are adapted.

Artifacts
- Yocto deploy dir: `/home/jresp/solo/workspace/yocto/imx-yocto-bsp/build-solo/tmp/deploy/images/<machine>/`
- Copy finished outputs to `/home/jresp/solo/artifacts/` for flashing.
