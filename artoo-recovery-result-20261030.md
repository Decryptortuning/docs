# Artoo watchdog, parameter journal and flash containment — October 30

This local candidate implements the next reliability layer on top of the completed Artoo modernization. The normal firmware keeps the watchdog disabled. A separate bench image enables the experimental runtime watchdog. Parameter journaling and SRAM flash containment are included in all seven built variants. No SSH/hardware access, card writes, MCU programming, staged-firmware promotion or pushes occurred. Physical Linux remains be98 plus its documented Chrony overlay; the MCU remains the previously installed `4803457` bench firmware.

Evidence lives in [/home/jresp/solo/.integration/artoo-recovery-20261030](/home/jresp/solo/.integration/artoo-recovery-20261030). Freeze 4 is the final source/checker gate; runtime and Linux build inputs have not changed since freeze 3. The preceding [modernization report](artoo-modernization-result-20261029.md) remains a separate immutable result.

## Sources and integration contract

| Owner | Final local revision | Responsibility |
|---|---|---|
| Artoo | `629c14b71ce6748272b9bd711d12e875b6c3cc9f` | Journal, SRAM flash path, experimental IWDG, diagnostics and build metadata |
| SoloLink | `3d9357434754452cc87f22152d23221be4f4c049` | Full parameter preservation/verification and diagnostic decoding |
| stm32loader | `327dac386743533e11eee5509e7b54042a8c39bd` | Reserved-page backup, guarded programming and installed-layout checks |
| meta-3dr | `7cbe9d6fa423ffdd7443db23e6c168e652f9b7f5` | Combined app source pins |
| OPenSolo_2 | `a7e8c509029e4a87c0309f47f4da31e6d3a21b48` | Source lock, retired temporary actor, Renode and installed-ARM checks |
| meta-solo-bsp | `281976648cf778b27151f4a3d330c52beea652cb` | Existing permanent controller power-hold fix; unchanged |

The reviewed chain is the actual 172-byte `StoredValues` producer, journal serialization and MCU flash consumer, image marker and linker bounds, Linux loader/updater and post-startup parameter reader. Source pins were committed before qualification. The new MCU is still a separate artifact: Yocto's staged MCU inventory was not replaced. Its older unmarked image is intentionally rejected by the new updater if selected. Automatic-update enable policy remains unchanged.

## Two-page parameter journal

Flash pages 126 and 127, starting at `0x0803f000` and `0x0803f800`, are reserved. Application capacity is 258,048 bytes. Each 2 KiB page holds one 192-byte snapshot; saves alternate pages. This is redundant snapshot storage, not an append journal or a wear-leveling system.

The little-endian record is `OSJ1`, schema 1, payload length 172, a 32-bit generation, CRC32, the unchanged 172-byte payload, commit halfword `0xa55a`, and erased padding. CRC covers the header before its CRC field plus the payload. Selection requires schema, size, CRC and commit validity. Generations compare with wrap handling; exactly half-range ambiguity and conflicting equal-generation records fail closed.

A save erases only the inactive page, programs and verifies the body, then writes and verifies the commit marker. The old valid page survives until the new snapshot is complete. Dirty-generation tracking remains intact; failed saves remain pending. Native interruption testing covers every modeled erase/program halfword boundary for both first migration and rollover: 2,242 cases in total.

Legacy raw data on page 127 can be imported only when no valid journal exists, page 127 does not start with journal magic, and its bytes after the 172-byte legacy record are erased. The first journal commit uses page 126, preserving raw page 127. Loading alone does not migrate storage. Legacy raw data has no CRC; its transitional fallback cannot provide the same integrity guarantee as committed journal records. Existing calibration/mapping validation still applies. Mature corrupt/unknown journal storage is not silently initialized or overwritten; invalid controls remain locked out. There is no general downgrade conversion tool.

A fixed 16-byte layout marker at image offset 304 contains `OS5JNL1!`, the reserved start, payload size and schema. Link checks and the manifest declare this layout. The loader rejects unmarked or oversized new images.

## Guarded Linux update path

The supported programming contract requires `--artoo --artoo-backup-dir DIR -wv -s 126`. This is documentation of a future authorized operation, not a command run on hardware here.

Before erasing, the loader checks chip ID `0x414`, 256 KiB flash geometry and software-watchdog option mode. It reads/classifies all 4,096 reserved bytes, creates a unique mode-0600 backup, and fsyncs the file and directory. It erases pages 0–125 only, verifies the new application, and requires both reserved pages to remain byte-identical before releasing the application. Mutation failure leaves the MCU in ROM; no retry, unprotect escalation or reset/release is attempted. No option bytes are modified.

Generic loader mutations also inspect the installed layout marker and reserved-page magic. This blocks an old-image downgrade, erase-only request or unprotect request from bypassing the Artoo guard on an already migrated device, including its first-save transition. Unreadable geometry/storage checks fail closed. This is a guard in the owned loader, not protection against arbitrary external ROM/debugger tools.

The updater requires the permanent bound power-hold driver, backs up the full 172-byte application parameter response, and compares all 172 bytes after startup. A mismatch does not trigger an automatic calibration rewrite. The old direct-GDB script, alternate shell updater and temporary-flash actor are retired/fail-closed. Historical raw-page read-only preflights are not qualified journal readers.

A future installation must establish the compatible Linux loader/updater and permanent power-hold behavior before installing this MCU layout. The currently installed older tools must not be used to program or downgrade a journal-enabled controller.

## SRAM flash containment

Parameter erase/program operations execute synchronously from SRAM. The complete busy interval includes its polling loop, helpers, literal pools and exception paths. PRIMASK blocks ordinary interrupts; VTOR temporarily selects a private aligned SRAM table whose exception entries go to the SRAM containment handler. Normal completion restores the original VTOR and PRIMASK only after BSY clears. Polling is bounded independently of SysTick.

A stuck BSY records diagnostic stage 11; an exception during the interval records stage 12. The SRAM terminal disables charging, both battery shunts, vibration and buzzer output while leaving controller/i.MX power holds untouched. It records the retained fault checksum and remains in SRAM. It does not return into busy flash, retry or automatically reset. This is fault containment; automatic recovery from an uncertain flash operation remains intentionally unimplemented.

If IWDG is already active, one consumed healthy foreground credit admits a finite journal transaction. The foreground admission refreshes IWDG while flash is idle, and each SRAM primitive refreshes it before its busy interval. A normal transaction has one erase, 94 body halfwords and one commit halfword. Repeated parameter writes cannot independently renew that credit without fresh health progress. The terminal deliberately continues feeding IWDG so an uncertain flash operation is not reset; this is the explicit exception to progress-only feeding.

All seven linked images pass the SRAM branch/address checker. Final disassembly and literal-pool review found no flash-resident helper or data reference in the busy/terminal path. The `.ramfunc` placement in copied `.data` intentionally produces an executable data segment and a linker RWX warning; no MPU isolation is claimed. A peripheral/bus stall that prevents SRAM execution itself is outside the software poll guarantee.

## Experimental runtime watchdog

`ARTOO_EXPERIMENTAL_IWDG=ON` is accepted only for the bench profile and adds the `-iwdg` version suffix and manifest flag. It defaults off; production and factory attempts to enable it are rejected. Existing startup bounding remains separate: this watchdog starts after startup, not during the early boot waits.

Activation requires 150 accumulated successful foreground health epochs. Each requires a completed foreground batch, dispatched heartbeat, new completed ADC sequence and no pending UART task. These are accumulated observations, not a consecutive/window guarantee. Neither an interrupt alone nor the presence of UART/LCD traffic feeds it. After activation, fresh health progress feeds and grants one flash credit.

The divider is 256 with reload 4095. ST's 30–60 kHz LSI range implies approximately 17.5–35 seconds, nominally 26.2 seconds; this is not a measured controller timeout. Retained state records reset streak and running state. Power-on reset begins a new session; invalid retained state accompanying a watchdog reset quarantines activation. Two successive watchdog recoveries stop new watchdog activation. Quarantine means the application can run with the watchdog disabled, not a terminal safe mode. After 3,000 healthy epochs the streak clears. A valid watchdog recovery of the Running state preserves the software i.MX enable path; its physical behavior remains unqualified.

Diagnostic metrics word 31 exposes experimental/started/quarantined bits and reset streak. The Linux diagnostic CLI decodes these plus new failure stages 11–13. The actual watchdog driver is tested with instrumented registers, including activation keys, stale ADC, pending UART, reset precedence and flash-credit consumption.

## Qualification and remaining limits

[Final qualification receipt](/home/jresp/solo/.integration/artoo-recovery-20261030/qualification-result.json) binds sources, checkers and all final results. [Linux artifact manifests](/home/jresp/solo/.integration/artoo-recovery-20261030/linux-artifacts/result.json) identify 32 independent deployment copies; [firmware matrix](/home/jresp/solo/.integration/artoo-recovery-20261030/build-matrix3.json) identifies seven exact binary/ELF pairs. Owned-source bundles are recorded in `source-bundles/result.json` under the same receipt; these are not a new complete vendor/offline closure. Scope includes native tests, real ARM userspace with memory-only ROM fixtures, linked-image inspection and Renode. No test here establishes electrical power-loss tolerance, watchdog reset timing, battery/charger behavior or operational flight acceptance.

The three final DVT Renode runs (production, bench and experimental bench) each pass 14 scenarios. The added test injects synthetic FLASH.BSY persistence and invokes the actual linked flash wrapper; it observes execution/VTOR in SRAM and a valid stage-11 retained record. It is not an analog flash model. The experimental image boots in Renode, but modeled ADC progress does not activate IWDG; Renode does not prove watchdog expiry or the physical reset/quarantine sequence. Existing startup, UART, Tamp, MAVLink and retained-fault scenarios still pass.

Further hardware work needs separate authorization: permanent power-hold cold/warm/shutdown checks; staged old-to-journal migration with independent backups; reset/power interruption at representative save boundaries; UART/ADC latency through real flash writes; IWDG expiry and two-reset quarantine; actual LSI/polling margins; and return to a normal watchdog-disabled image. Do not erase reserved pages as a recovery shortcut. The current work does not enable an automatic updater or schedule a device action.

## Exact candidate and test results

Linux graph: `5b00a1619b3bc97ffbd98b978583bbf17d9024cf53c2919fe7e62ed6f68bad10`. All four incremental/cached builds and all 16 image/initramfs/ARM-userspace/maintenance checks pass. Both controller profiles execute the installed loader under ARM userspace with a memory-only ROM, forbidden hardware serial opens and temporary backup files. Installed loader/updater/diagnostic hashes match their frozen sources. Drone images intentionally contain no Artoo loader.

Fourteen native firmware suites and the build-tool suite pass. Integration tests:108; SoloLink tests:44 (unchanged 3d93574 source from freeze1); loader tests:15. Seven firmware builds pass linked-image and SRAM-call checks. Bench-DVT uses9,080 static RAM bytes plus584 retained bytes, reserves4,096 stack bytes and leaves35,976 heap-capacity bytes; static compiler frames are not a full stack high-water/call-chain proof. The normal bench binary is192,636 bytes, leaving65,412 bytes before reserved flash.

| Firmware variant | Binary bytes | SHA-256 |
|---|---:|---|
| production-dvt | 192248 | `6d36c4261ba2ac91788fe688b19ae73947d51f1b2fc1fe832dcdbb7473a0bc9a` |
| bench-dvt | 192636 | `f974f6e65b4a662ed5ec766a9f3acf1263e47cf66498d2a202dec0ccb97d8518` |
| factory-dvt | 193656 | `3dc115877c1eb72d794e2e88313209d349aa86ce1b3df9bb3e06e9423c01dd43` |
| production-bb03 | 192192 | `4aa81b66d6f636adcc05a3374f74747513e3eb1ca264d1740153c1bafa9971e9` |
| bench-bb03 | 192580 | `daa5a02cf43dcb8cdb126fa06cfc5b28e09bec152c46375433276e876e20d698` |
| factory-bb03 | 193608 | `b8678dbfe0f70cb902375e13dd89df73dbfe620bf6019a96a14788b7011b9967` |
| bench-dvt-iwdg | 193340 | `3889a112028627391b9a9e1cd264df104affe4db1d55df0343d1ec0f91a20ed9` |

| Linux SD image | SHA-256 |
|---|---|
| controller-release | `e3a41b195301cf5f669b6d4434e77e2f3f1f33d2f8003855da1a8a41a9db7227` |
| drone-release | `547450e268d989594cf10237e7eaedffef5eaef114899345d00a980d742ff8df` |
| drone-development | `80d664689913996176edeec265348f413ccdf4e6fe835c1cfcc0f52954194bb3` |
| controller-development | `67744c3d8def65b7819f4c20dc53057ed49ff8a3cf28529f06f34d52bb626b16` |

The version base is `5.0.0-dev+g629c14b71ce6`; bench adds `-bench-noidle`, and the experimental image also adds `-iwdg`. Exact variant metadata is in each preserved build manifest.

Qualification history is preserved: the first Renode containment test reached the terminal but failed to capture after halt; moving capture to the containment hook fixed the test. Review then found a generic-loader bypass and near-expiry watchdog flash entry; both were fixed and refrozen. An initial transport invocation omitted its required SoloLink root and was corrected unchanged. The first drone storage-probe invocation was too broad: its expected missing controller-only loader produced FileNotFoundError; controller-only scope was corrected without a source change. These attempts are not represented as final passes.

## Reproduction and primary references

Use `build-environment.json` in the receipt to load the preserved isolated Arm 15.3/Pillow 13 environment and MAVLink export 7, override `ARTOO_BUILD_ROOT` with a new output directory, then run the firmware wrapper with the desired profile/board. Do not source the previous candidate's revision-locked convenience script for this revision. The original system compiler and Python environment were not changed.

The flash design follows [ST PM0075](https://www.st.com.cn/resource/en/programming_manual/pm0075-stm32f10xxx-flash-memory-microcontrollers-stmicroelectronics.pdf) and [RM0008](https://www.st.com/resource/en/reference_manual/rm0008-stm32f103xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf); watchdog frequency bounds come from the [STM32F103 high-density datasheet](https://www.st.com/resource/en/datasheet/stm32f103vc.pdf). These references do not replace controller measurements.

Yocto 6 and license work remain deferred. Existing QEMU preferred-version/host-accounting warnings and the loader's obsolete license-label warning remain recorded; no unrelated warning cleanup was included. The owner's `build-env.md` edit and historical source9/modernization receipts remain preserved.
