# OpenSolo board support and porting work

Both target machines inherit the NXP i.MX6Solo SABRE SD baseline and supply Solo-specific kernel trees and U-Boot defconfigs. This is a substantial bring-up port; successful compilation still needs to be connected to the board's electrical behavior and userspace configuration.

## Active definitions

Paths below are relative to `/home/jresp/solo/workspace/layers/meta-solo-bsp`.

| Area | Controller | Drone |
|---|---|---|
| Machine | `conf/machine/imx_controller.conf` | `conf/machine/imx_drone.conf` |
| Kernel device tree | `recipes-kernel/linux/linux-imx/files/arch/arm/boot/dts/nxp/imx/imx6solo-controller.dts` | Same directory, `imx6solo-drone.dts` |
| U-Boot defconfig | `recipes-bsp/u-boot/files/configs/imx6_controller_defconfig` | Same directory, `imx6_drone_defconfig` |
| U-Boot device tree | `recipes-bsp/u-boot/files/arch/arm/dts/imx6solo-controller.dts` | Same directory, `imx6solo-drone.dts` |

Shared U-Boot board C/DCD files are under `recipes-bsp/u-boot/files/board/3dr/3dr_hdtc`; compiled environment defaults are in `recipes-bsp/u-boot/files/include/configs/3dr_hdtc.h`. Kernel feature fragments are under `recipes-kernel/linux/linux-imx/files`, including media, Wi-Fi, serial, RFKILL, netfilter, squashfs and USB gadget settings.

Both machines request `uImage` with bundled `3dr-initramfs`, use the `imx-solo` override, and remove OP-TEE for the current SD boot flow. The product distro selects SysV init. Product images and maintenance scripts live in `meta-3dr`.

## Formats crossed by the port

| Area | Earlier representation | Current integration requirement |
|---|---|---|
| Linux board hardware | Linux 3.10 Freescale board trees/platform drivers | Linux 6.6 bindings, pin functions, GPIO descriptors, regulators and explicit peripheral relationships. |
| Video input | Legacy Freescale capture plus ADV7610 integration | A compatible receiver, IPU/CSI bridge, media endpoint graph and userspace routing/format owner. |
| U-Boot | Board C callbacks and CONFIG macros | Driver-model devices, Kconfig/defconfigs, board DTS and CFG macros; preserve direction/level/timing behavior deliberately. |
| Yocto metadata | Underscore overrides and copied recipes | Colon overrides, scoped appends, effective task/configuration verification and staged image ownership. |

A pad listed in DTS establishes muxing only to the extent its consumer uses it. It does not automatically reproduce a legacy C callback that configured direction, output level, timing or reset-button polling. Likewise, a kernel fragment's requested symbol is not proof that it exists or survives dependencies in the effective configuration.

## Controller and shared board work

| Item | Current assessment | Required follow-up |
|---|---|---|
| DDR and boot mapping | Audit found preserved DCD programming and consistent ROM/USDHC3 to U-Boot mmc1 mapping; Linux aliases separately expose the boot disk as mmcblk0. | Confirm detected RAM and boot-device identity on the verified image. |
| Kernel loading | The recovered controller entered Linux with the 64 MiB bootm allowance. | Check actual current boot log; preserve the raw bootloader/environment boundary. |
| Console | J2 was identified as UART1 RX/TX/GND; the final recovered session reached login but could not send commands from the host. | Isolate adapter/wiring/transmit behavior with the confirmed hardware procedure. |
| Wi-Fi and DHCP | Earlier logs showed ath9k/AP setup. Current image installs controller-only DHCP policy and passes DNSMasq syntax. | Verify real leases, expected addresses and pairing on the current image. |
| Early GPIO sequencing | Modern `board_early_init_f` omits the controller's old GPIO1_IO19/21 high outputs and GPIO2_IO14/13 inputs. | Check schematic/board revision, STM32 BOOT/RESET, power and pull states before choosing a modern owner. |
| Physical factory reset | Legacy three-second active-low polling is absent; original inputs were controller GPIO3_IO24 and drone GPIO4_IO05. | Restore button behavior deliberately and verify the kernel factory-reset request end to end. |
| Environment definitions | Installed `boot.env` files are not imported by the recorded boot path; active defaults come from C strings. | Choose one authoritative representation and verify generated behavior. |
| RAM and CMA | Saved logs report 512 MiB RAM; source DTS descriptions used 1 GiB and the saved kernel reserved 320 MiB CMA. | Correct descriptions and size CMA for the selected media workload; this is not proof of a current memory overrun. |

The pinned NXP kernel retains the controller's Freescale framebuffer/HDMI path. Numerical GPIO helpers carry migration debt, but the audited driver/alias setup does not establish that every legacy GPIO number is wrong.

## Drone media pipeline

The recovered effective kernel combines `MXC_IPU`/legacy `VIDEO_MXC_CAPTURE` with the upstream ADV7604 driver used for ADV7610. The requested upstream media bridge depends on `IMX_IPUV3_CORE`, which was disabled. These pieces do not form a complete compatible capture pipeline merely because the receiver node and endpoint graph exist.

Resolve the pipeline as one change with clear driver and userspace ownership:

1. Select a compatible IPU/CSI bridge and effective kernel configuration. Avoid blindly enabling two competing IPU implementations.
2. Resolve the pinned CSI bridge's `get_mbus_config` requirement against the receiver driver's missing operation.
3. Correct reset assertion/deassertion semantics. The DTS uses `GPIO_ACTIVE_LOW`, while the pinned driver ends its logical reset sequence at 1, which drives that physical signal low under descriptor polarity.
4. Select the HDMI input and establish ownership for links, timing, pad formats and ADV7610 output programming. The recorded node omits `default-input`; the EDID helper supplies EDID/queries timings but does not complete input/routing setup.
5. Decide the pixel bus and console assignment together. Current bring-up uses eight CSI data lines and UART1 on CSI0_DAT10/11; the old 1080p path used a sixteen-bit BT.1120 group and disabled UART1.
6. Verify capture, encoding and restart behavior on the drone with the chosen configuration.

UART5's legacy KEY_COL1/KEY_ROW1 pin group is also absent from the modern drone tree. Establish its intended accessory use before treating it as a required runtime failure; the audit found no current application reference to `ttymxc4`.

## Audit context and acceptance

Detailed source/effective-config findings are in the [original board support audit](/home/jresp/Active_GH/OPenSolo_2/OpenSolo-board-support-audit.md). Its evidence directory is local to this restored machine. The audit's DHCP, image staging, release source selection and provisioning/update findings have since received software fixes, recorded in [Project status](status-and-evidence.md) and [Decisions and build history](decisions-and-history.md).

Further BSP work needs effective configuration inspection, device-tree schema checks against the pinned bindings, rebuilt image validation and confirmed board observations. Archived logs remain useful diagnostics; their successful boot or earlier reset symptom is not a result for the current baseline.
