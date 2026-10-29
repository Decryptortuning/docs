# OpenSolo board support and porting work

**Latest local BSP candidate:** [permanent power-hold and updater startup fix](controller-power-fix-20261029.md), graph7bac1220…, passes four builds and16 baseline checks. Controller U-Boot asserts holds; Linux owns them until final power-off. Startup verification resynchronizes within a deadline and programming requires the bound driver. This candidate is not installed; current physical Linux/MCU remain unchanged.

The current source candidate stays on NXP Scarthgap and Linux 6.6.52. Both Solo
machines now use the owned `solo-imx6-common.inc`, requiring NXP's `imx-base.inc`
and Cortex-A9 tune directly. They no longer inherit the complete SABRE board.
The source refactor preserves SoC overrides/providers and enumerates Solo USB,
PCIe Wi-Fi, audio and display features without SABRE SDIO/Bluetooth/touchscreen
or OP-TEE identity. Build and hardware qualification are separate gates; see the
[refactor report](scarthgap-refactor-20261028.md) for the combined
candidate and results as they become available.

## Active definitions

Paths below are relative to `/home/jresp/solo/workspace/layers/meta-solo-bsp`.

| Area | Controller | Drone |
|---|---|---|
| Machine | `conf/machine/imx_controller.conf` | `conf/machine/imx_drone.conf` |
| Kernel device tree | `recipes-kernel/linux/linux-imx/files/arch/arm/boot/dts/nxp/imx/imx6solo-controller.dts` | Same directory, `imx6solo-drone.dts` |
| U-Boot defconfig | `recipes-bsp/u-boot/files/configs/imx6_controller_defconfig` | Same directory, `imx6_drone_defconfig` |
| U-Boot device tree | `recipes-bsp/u-boot/files/arch/arm/dts/imx6solo-controller.dts` | Same directory, `imx6solo-drone.dts` |

Shared U-Boot board C/DCD files remain under
`recipes-bsp/u-boot/files/board/3dr/3dr_hdtc`. The authoritative slot commands
are in `files/include/configs/3dr_hdtc.h`; board defconfigs supply bootcmd and
identity. Poky's generated `u-boot-initial-env` supplies the installed
`/etc/u-boot/boot.env`. Handwritten duplicate environment files were removed.
The normal boot path uses compiled defaults or a saved raw environment; it does
not need to import that installed file. Existing saved environments can override
new defaults and require an explicit migration decision.

Both slots now require their own successful kernel and DTB loads before bootm.
LATEST failure permits a fresh GOLDEN attempt; neither path boots stale RAM or
omits the DTB. Image source requires the bundled-initramfs kernel, exact board
DTB and machine U-Boot for FAT, update archive and raw bootloader placement.
MBR, uImage, GOLDEN/LATEST, squashfs, ext3 RWFS/LOG, raw environment offset
0xC0000/size 0x2000 and the 64 MiB bootm allowance are preserved.

The kernel append is now `linux-imx_6.6.bbappend`. Fragments still use NXP's
`DELTA_KERNEL_DEFCONFIG` merge after defconfig selection. Both boards use
`KERNEL_LOCALVERSION=-lts-next` for external-module alignment; actual release,
vermagic and symbol CRC checks remain acceptance requirements. Historical DTS
are retained under `reference/legacy-dts`, outside active recipe inputs.
Product distro, images and maintenance scripts belong to `meta-3dr`.

## Controller and shared acceptance limits

The controller's scoped `pci=nomsi` and `CONFIG_CMDLINE_EXTEND` policy is
preserved. The physical card currently combines the older rootfs with the
qualified nomsi kernel; it is not the new refactor image. The MCU reset-readiness
work remains on hold. The refactor report records successful offline builds and checks; physical
acceptance of the new image remains separate.

| Item | Current assessment and remaining gate |
|---|---|
| DDR and boot mapping | Preserve DCD programming, U-Boot mmc1 and Linux mmcblk0 aliases. Confirm the actual boot-device identity and detected banks for each board. |
| Early GPIO sequencing | Modern early board code omits controller legacy GPIO1_IO19/21 high outputs and GPIO2_IO14/13 inputs. Establish schematic/revision, STM32 BOOT/RESET, power and pull states before choosing a new owner. A DTS mux entry alone does not establish direction, output level or timing. |
| Physical factory reset | Legacy three-second active-low polling is absent: controller GPIO3_IO24 and drone GPIO4_IO05. Button behavior and the factory-reset request path require deliberate implementation and hardware verification. |
| Console and MCU | UART1 console observations do not prove safe MCU reset sequencing or ROM-loader readiness. Preserve the reset-readiness hold and existing hardware evidence. |
| Wi-Fi and DHCP | Retain controller PCIe ath9k and packaged DHCP policy. Validate current candidate leases, addresses, pairing and sustained operation independently of older logs. |
| RAM/CMA | Header default is 512 MiB; drone DTS retains a historical 1 GiB placeholder. Neither establishes fitted capacity. Preserve `imx_ddr_size()` and explicit OF_LIBFDT memory-bank fixup. Assert DCD geometry, detected banks and Linux handed-off memory agree; check boot payload ranges and media/CMA needs against those banks. |

The controller retains the NXP framebuffer/HDMI stack. Numerical GPIO helpers
remain migration work, but their existence does not prove every legacy GPIO
number is wrong.

## Drone media pipeline

The source candidate selects one media-controller path:
ADV7610/adv7604 -> video-mux/MMIO mux -> IPUv3 CSI -> capture. Its drone-only
fragment enables staging, VIDEO_DEV/subdev, IPUv3 and mux prerequisites and
removes nonexistent symbol requests. In this NXP tree VIDEO_IMX_MEDIA builds
the i.MX6 capture components; VIDEO_IMX_CAPTURE gates i.MX8 components. The
competing legacy MXC IPU/capture stack and its legacy capture DT node are disabled
for drone. Effective configuration and module/DTS validation must still prove
that the requested graph survives dependency resolution.

The reset GPIO now uses GPIO_ACTIVE_HIGH because the pinned adv7604 driver
writes descriptor logical 0 then 1 for a physical low reset pulse. The previous
ACTIVE_LOW mapping held physical reset asserted at the end. This source-derived
correction is not a measured HDMI probe/capture result.

Remaining functional gates:

1. Resolve the CSI bridge's `get_mbus_config` expectations against the receiver
   implementation and establish HDMI input, EDID, timing, link and pad-format
   ownership. The EDID helper alone does not configure the complete graph.
2. Port the actual userspace consumers together. `vidlaunch.cpp` still invokes
   `mxc_v4l2_capture`, opens `/dev/video0`, and uses
   `mfw_v4lsrc`/`mfw_ipucsc`/`vpuenc`; `hdmiout.c` uses `vpudec`/`mfw_isink`.
   Those legacy assumptions are not made compatible by enabling the modern
   kernel graph. This remains a distinct userspace functional-port gap.
3. Qualify eight-bit CSI/BT.656 wiring and UART1 on CSI0_DAT10/11. The earlier
   1080p path used sixteen-bit BT.1120 and disabled UART1; current source does
   not certify that mode.
4. Verify capture, encoding, restart, flight-controller link and pairing on
   the actual drone after offline acceptance.

UART5's legacy KEY_COL1/KEY_ROW1 pin group is also absent. Establish accessory
requirements before treating that as a current application failure; the earlier
source audit found no application reference to `ttymxc4`.

## Evidence boundaries

The [original board support audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md)
retains earlier source and effective-config observations. The
[BSP source contract](/home/jresp/solo/workspace/layers/meta-solo-bsp/README.md)
and [refactor report](scarthgap-refactor-20261028.md) describe the current delta.
Use the latter report for pinned build/test results when recorded. Archived
successful boots and earlier image checks do not qualify this candidate.
