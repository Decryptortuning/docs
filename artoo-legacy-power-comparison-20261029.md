# Artoo reset and legacy power-control comparison — October 29

The comparison found a concrete BSP migration omission: **legacy controller U-Boot drives GPIO1_IO19 and GPIO1_IO21 high; the modern board code does not.** Legacy source explicitly describes these as the STM32 and i.MX6 POWER-ON pins. Both modern device trees retain their GPIO mux entries, but those entries do not configure output direction or assert a level. This is the leading source-backed explanation for losing the i.MX when the STM32 resets. It is a confirmed source difference, not yet an electrically proven root cause.

This was a read-only comparison. No GPIO export/write, reset, service change, reboot, ROM operation or flashing occurred. No product source was changed or rebuilt. SSH could not resolve opensolo-controller.lan, so this pass added no fresh live register sample. The previous one-shot updater hook remains restored and no retry was armed. **Flashing remains blocked.**

## Historical authority

The recovered full vendor trees allow comparison against the actual tags selected by the old recipes, rather than relying only on archived device-tree copies:

- Legacy U-Boot recipe selects `solo_v1.0.0`, commit `b0bd4a36d3ed71e2ba89720b0c46b2342a4af11b` in `/home/jresp/solo/OPENSOLO_REFERENCE/modules/imx6-uboot`.
- Legacy kernel recipe selects `sololink_v1.3.0-5`, commit `4176d5778007e557256a06c79e270e65cc753ffd` in `/home/jresp/solo/OPENSOLO_REFERENCE/modules/imx6-linux`. The recovered kernel working HEAD is a different commit; the receipt preserves the recipe-selected tag's files explicitly.
- Canonical old loader, recipes and MCU source are in `/home/jresp/Active_GH/OpenSolo`, commit `5b745243a2d88691ccd045e7238742431c668a5a`. This source comparison does not identify the exact source hash of the installed OpenSolo-4dev MCU binary.
- Current owned BSP is `1602ee542d832ba51b7badad15faad6755acb2a7`. Actual Linux source inspected is `e0f9e2afd4cff3f02d71891244b4aa5899dfc786`; current Artoo candidate source is `48034572145e9e62dff881141277564e1ed1d3a6`.

The old machine selects `3dr_hdtc_artoo_config`. Its boards.cfg entry defines CONFIG_IMX6_3DR_TYPE_ARTOO, so the power-on code is active for the historical controller target; this is not merely unused code behind an unrelated option.

## Signal comparison

| Function | Legacy | Current | Finding |
|---|---|---|---|
| BOOT0 selection | Global45, GPIO2_IO13, SD4_DAT5 | Same | No numbering/mux migration error found |
| Reset request | Global46, GPIO2_IO14, SD4_DAT6; high100ms then low, wait500ms | Same loader sequence | No logical polarity/delay regression found; external inversion is not established by these software values |
| MCU serial | UART2 SD4_DAT4 RX / SD4_DAT7 TX, ttymxc1,1152008E1 during loader | Same | No RTS/CTS pad takeover in the UART pin group |
| Controller power-on outputs | GPIO1_IO19/SD1_DAT2 and GPIO1_IO21/SD1_DAT3 explicitly output-high in U-Boot | Mux entries remain; no matching output-high initialization in current board code/device trees | Confirmed migration omission |
| Reset/BOOT defaults | U-Boot explicitly makes GPIO2_IO14/13 inputs | No equivalent explicit board initialization | Secondary missing initialization; prior live registers already showed these as inputs before probing |
| MCU i.MX enable / keep-on | DVT PC14 / PC15 | Same | No pin reassignment in new firmware candidate |

Both kernels use the GPIO alias index times32 for the global base. The earlier live receipt independently confirms GPIO2 at base32, giving45=32+13 and46=32+14. Old and current Solo/DualLite pin-function headers have identical reset/BOOT mux tuples: SD4_DAT5 `0x354 0x73c 0x000 0x5 0x0`, SD4_DAT6 `0x358 0x740 0x000 0x5 0x0`. UART2 RX/TX tuples also match. Both board descriptions include imx6dl.dtsi.

Both pinctrl drivers interpret `0x80000000` as NO_PAD_CTL. It preserves pad-control configuration; it does not request an output-high GPIO. The group named hoggrp-1 is a **pinctrl** hog, not a GPIO hog with output-high. Neither current controller tree has a replacement GPIO consumer that asserts19/21. The current shared board_early_init_f returns immediately. The controller's rc_mxc.S only starts UART0 getty; it does not restore the missing power outputs.

The old GPIO generic output path writes the value before enabling output; the current path has the same value-first default. Modern gpio-mxc additionally uses BGPIOF_READ_OUTPUT_REG_SET, so software output reads are not a substitute for measuring a pin or checking PSR. This is no evidence of a45/46 bank swap or reversed direction semantics.

## How the omission fits the restart

Legacy board_early_init_f explicitly sets both19/21 output-high before Linux. Its comment says “Turn on the STM32 and i.MX6 POWER-ON pins.” GPIO data/direction can then survive handoff to Linux, whose pinctrl entries select the GPIO function. The modern port retained the mux entries while omitting the early direction/level setup.

The MCU owns another part of the power path. Legacy and current main.cpp, board_dvt.h and powermanager.h match byte-for-byte. Both PowerManager::init implementations assert PC15 keep-on, configure PC14 and explicitly disable i.MX; later entering Running enables it again. The differences elsewhere in powermanager.cpp concern button/task handling, not this early power initialization. The new MCU candidate has not been flashed, so its modernization cannot have caused the currently observed resets.

If the old i.MX-side outputs provide the intended power hold across MCU reset, omitting them explains why both an application reset and a ROM-entry reset can take down the uploading host. This fits all prior attempts, including actual rcS/S60 before the normal bridge, without needing a UART-open fault or programming activity. It also gives a concrete reason to investigate board power control before attributing the behavior to battery capacity or assuming a charger will cure it.

However, the available HDTC schematic/pinout does not show the complete STM32 mainboard power circuit. The legacy comment identifies the pair's purpose but does not prove each pin's individual net, electrical polarity, gating or shutdown behavior on this physical revision. The earlier live debugfs dump has no consumer on19/21; that alone does not establish their direction or level, because bootloader-inherited outputs need not have a Linux consumer. GPIO1 DR/GDIR/PSR have not been sampled in this comparison. The power-hold explanation remains a strong, testable hypothesis rather than a measured rail-collapse diagnosis.

The old loader's setPin also has a wrong-close bug: after writing value it closes dir_fd again instead of explicitly closing val_fd. The Python3 port closes the correct descriptor. CPython's reference cleanup generally closed the old local value file on return, and the newer preopened-descriptor reset diagnostics reproduced the failure without this helper. There is no basis to restore that bug or reverse the reset polarity. No reset or programming code was changed in this pass.

## Recommended next work

1. On the next reachable controller, read GPIO1 DR/GDIR/PSR at209c000/004/008 and the SD1_DAT2/3 mux/pad registers without exporting or driving these pins. Compare bits19 and21 against legacy output-high behavior. Preserve boot ID and source identity with those reads. Existing debugfs claims alone are insufficient.
2. Prepare a local, controller-specific restoration of the legacy power-hold initialization, including GPIO mux selection and safe output-high ordering before Linux handoff. Preserve shared drone behavior. Decide explicitly how19/21 are retained through Linux and released during normal shutdown; do not introduce a generic permanent GPIO hog without reviewing that behavior. Do not repurpose the MCU reset pin as a power workaround.
3. Review and freeze that combined candidate before build/configure checks. Qualify its actual compiled U-Boot/DTB and controller-only selection locally. No such implementation/build is claimed here.
4. A later authorized hardware trial should first verify the power-hold levels and normal shutdown, then repeat one guarded non-programming ROM identity/readback/release check. J2 console and a scope/logic analyzer remain useful if it still fails. Do not enable erase/write or install an automatic firmware candidate while the uploader host still disappears.

GPIO19/21 also appear in the historical drone baudcheck USB-routing utility. That use is not a controller startup replacement and reinforces why the restoration must be machine-specific rather than a shared unconditional change.

[Comparison receipt](/home/jresp/solo/.integration/artoo-legacy-power-audit-20261029T224119Z/result.json) and [source inventory and producer/consumer review](/home/jresp/solo/.integration/artoo-legacy-power-audit-20261029T224119Z/source-review.json) retain exact source hashes and recipe-tag snapshots. Prior [reset tests](artoo-reset-diagnosis-20261029.md), [automatic-path preflight](artoo-boot-preflight-20261029.md) and [readiness evidence](artoo-flash-readiness-20261029.md) remain immutable. Changes are local documentation only; Yocto6 and license work remain deferred.
