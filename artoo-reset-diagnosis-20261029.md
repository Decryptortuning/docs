# Artoo reset diagnosis — October 29

**Later live test:** [temporary legacy power-hold test](artoo-power-hold-test-20261029.md) passes application reset and ROM identity/geometry/repeated readback/release with both GPIO19/21 driven high. Linux stays up; calibration matches and pins are released afterward. Permanent BSP/updater correction remains pending; no flashing occurred.

**New source finding:** [legacy power-control comparison](artoo-legacy-power-comparison-20261029.md) identifies omitted controller U-Boot GPIO19/21 output-high initialization. Reset/BOOT mapping matches the old BSP. This is the leading source-backed explanation, pending live register/electrical confirmation; no restoration or flashing has been performed.

**Later automatic-path test:** [one-shot boot preflight](artoo-boot-preflight-20261029.md) also restarted Linux during loader initialization. Its recovery skip guard worked, the original hook was restored and firmware/calibration remain unchanged; no flashing occurred.

The authorized diagnostic round reproduced the i.MX restart with a normal application reset, then with a shorter requested pulse and with ROM mode requested. **No MCU erase, write, protection change or flashing occurred.** The controller recovered after each attempt. The final observed MCU remains OpenSolo-4dev, with the same factory parameter record and a normal single bridge owner.

The failure is now isolated more narrowly than the earlier ROM preflight: inactive-low BOOT0/reset preparation succeeds, and Linux restarts after the active reset transition is attempted. It does not require ROM selection, a UART parity change, a new serial open, stopping the bridge or programming. The exact electrical/software reset mechanism is not yet established. In particular, a different supply has not been shown to solve it.

## Controlled comparisons

| Test | Conditions | Last durable phase | Result |
|---|---|---|---|
| Application reset, requested100ms | BOOT0low, existing1152008N1 bridge remains running, no new UARTopen | application-reset-pending | i.MX restarted, recovered4dev |
| Application reset, requested1ms | Same conditions, shorter requested reset hold | application-reset-pending | i.MX restarted, recovered4dev |
| ROM-read diagnostic, requested1ms | BOOT0high; existing bridge tasks paused; its descriptor duplicated; actual1152008E1 verified; no new UARTopen | ROM-reset-pending | i.MX restarted before a recorded ROM sync ACK; recovered4dev |

These are **requested**, not electrically measured, pulse widths. The processes did not persist a completed deassertion interval. The actual delivered waveform and the precise moment of host power/reset failure remain unknown. Shortening the requested hold did not prevent the restart; it does not establish a measured duration threshold.

All new pulse paths use a preopened GPIO value descriptor, a high write, a bounded sleep and a finally-low write. No file open, network call, logging, fsync or global sync occurs inside that interval. Durable phase records are written before assertion and after deassertion. Host tests cover the pulse ordering and attempted deassertion on interrupted sleep/write error, plus descriptor I/O fragmentation/short reads and rejection of non-read ROM commands/ranges. They do not measure a physical edge or rail.

The first two tests retained the normal bridge and all its UART settings. Both drove GPIO45 and46 inactive-low and verified the unchanged application reply after each preparation. The restart only followed the later reset attempt. The ROM diagnostic paused every bridge thread, duplicated its existing UART description with pidfd_getfd, set parity even without a PySerial open/RTS-DTR initialization and recorded software GPIO45=1/GPIO46=0 before reset. No runlevel transition was used. There was no erase/write path in the diagnostic actor.

## Identity and recovery

| Stage | Linux boot ID |
|---|---|
| Before application100ms test | `4ca4253b-8ec9-4c53-8ea7-b26bca12dbb7` |
| After100ms / before1ms | `8cfa5ceb-af79-43f4-a263-8a79b6996869` |
| After1ms / beforeROM request | `b88831f7-cbdd-4f7f-80c5-58119223becd` |
| AfterROM request | `39e83092-d263-4b3c-9a18-ae5a9bdff393` |

The final recovery capture is uptime165.94seconds. Twenty fresh172-byte application reads match the prior factory SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454`. The sole UART1 owner is the new `stm32` PID522. Both control GPIOs are unexported again after reboot, update-enable/force/request markers and automatic firmware candidates are absent, and pstore contains no crash records. Read-only kernel checks show no reported filesystem/MMC error. The Chrony correction remains intact and synchronized. Sparse/NUL portions in interrupted ordinary log files are retained as evidence of the abrupt interruptions; no filesystem repair was run.

The running Linux identity is still be98 plus the qualified Chrony init overlay correction. The complete faf image was not installed by this work. New diagnostic source commits are8dc79aa,9be9398 and01638d2 on the existing local branch. A self-contained integration source bundle is retained beside the final receipt; prior full-image transports remain separate.

## What remains unresolved

DVT source drives i.MX enable on PC14 and board keep-on on PC15. Its application startup calls PowerManager::init first, explicitly disables i.MX, initializes subsystems, and later enables i.MX on entering Running. That is a plausible contributor, not a proven causal explanation. The ROM-request test did not obtain an ACK, so it cannot show whether the MCU actually reached ROM or whether the physical BOOT0 level differed from the software output value. Reset routing/polarity, power-enable coupling and rail transients remain possibilities.

The physical mainboard circuit is not fully described by the available i.MX HDTC/J3 documents. There is no J2 Linux console adapter connected. The user was asked whether the LCD visibly restarted during the attempts; no observation had been received when this report was completed. Neither a battery-brownout diagnosis nor a charger remedy is justified by the current evidence.

A1ms comparison was chosen above the NRST pin filtering timescale documented in [ST's high-density STM32F103 datasheet, Table49](https://www.st.com/resource/en/datasheet/stm32f103zc.pdf). That pin-level reference does not establish the external board waveform or validate this board's reset routing.

## Next gate

Capture J2/UART1 Linux-console output during one planned reset to obtain early boot/reset-cause evidence, and check actual STM32 BOOT0/NRST against i.MX reset/enable/power behavior. A logic analyzer or scope can distinguish a pulse/routing issue from a rail collapse. Use only identified3.3V console/debug wiring; do not open unrelated attached serial devices.

Do not program through the onboard uploader while its Linux host disappears before ROM ACK/identity. If an independent STM32 programmer is available, it is a separate route to evaluate because it would not depend on i.MX staying alive. Exact MCU geometry/identity, selected DVT manifest/hash, reserved-page preservation and full readback remain required before any programming. No new firmware was selected or transferred to an automatic update location, and no blind reset or programming retry is scheduled.

[Three-case summary](/home/jresp/solo/.integration/artoo-rom-diagnostic-20261029T214734Z/diagnosis-summary.json), [recovery result](/home/jresp/solo/.integration/artoo-rom-diagnostic-20261029T214734Z/recovery-result.json), [100ms attempt](/home/jresp/solo/.integration/artoo-reset-diagnostic-20261029T213926Z), [1ms attempt](/home/jresp/solo/.integration/artoo-reset-diagnostic-20261029T214228Z) and [ROM-request attempt](/home/jresp/solo/.integration/artoo-rom-diagnostic-20261029T214734Z) retain the frozen sources, host checks, launch intent, interrupted records and recovered identities. Work remains local; Yocto6 and license work remain deferred.
