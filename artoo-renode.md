# Artoo Renode software harness

The local harness in
[`OPenSolo_2/tests/renode`](/home/jresp/Active_GH/OPenSolo_2/tests/renode/README.md)
uses Renode's existing STM32F103 platform to run an immutable Artoo firmware BIN.
It adds one small RCC response seam and observes real production instructions.
It neither builds nor edits Artoo source, and does not contact or write a
physical controller, SoloLink board, SD card, serial port, router or remote host.

## Reproduce a candidate check

Settle and review the harness and the saved candidate first. Run in WSL using a
new output directory, outside the source tree for generated evidence:

```bash
python3 /home/jresp/Active_GH/OPenSolo_2/tests/renode/run_artoo.py \
  --elf /home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave3/artoo.elf \
  --bin /home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave3/artoo.bin \
  --receipt /home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave3/verification.json \
  --output /home/jresp/solo/.integration/artoo-builds/renode-wave3-check \
  --negative-controls
```

The paths above are an example saved candidate, not a built-in selection or
allowlist. Select a future reviewed ELF/BIN/receipt through the same CLI. Renode
1.17.0 portable is installed at
`/home/jresp/.local/opt/renode_1.17.0-portable`; `renode` resolves through
`/home/jresp/.local/bin/renode`. The installed release archive was independently
verified during local setup; per-run records bind the actual executable and
platform files used. GNU Arm binutils and Python 3 are the remaining dependencies.

## What the checks prove

| Scenario | Stimulus and required observations |
| --- | --- |
| `startup` | Real reset vector to `_start`; all 48 KiB SRAM initially poisoned with `0xa5`; `.bss` must become zero and `.data` must match its flash initializer before the first constructor; every selected `.init_array` entry must be reached in order; then actual `main`. CPU halts before application initialization. |
| `power-wait` | The same startup, then real peripheral initialization function entries, SysTick ISR, heartbeat callback and Button/Battery task handlers. `Tasks::work` must be called from `PowerManager::waitForCompleteStartup` with state Boot. This is useful foreground software coverage while waiting for unmodeled battery inputs. |
| `pin-reset-running` | Initial RCC.CSR has PINRSTF set, with PORRSTF and SFTRSTF clear. Real `Rcc::latchAndClearResetFlags` and `PowerManager::waitForCompleteStartup` select the existing pin-reset path. Require real `enterRunningState`, then `Tasks::work` called from `main` with state Running, plus SysTick/heartbeat/task handlers. |
| `clock-stall` negative control | Suppress only SW→SWS mirroring. `_start` must remain within its clock setup and never reach memory initialization or `main`. |
| `memory-corruption` negative control | Change one emulated `.bss` byte immediately before the first constructor check. The memory assertion must reject it and `main` must remain unreachable. No input file is changed. |

Constructor count and addresses come from the selected binary's actual init
array and ELF symbols; optional preinit arrays are observed first and memory
is checked before the first preinit/constructor. No fixed eleven-constructor
assumption exists. Every
positive check verifies reset/memory/constructor/main order. Hooks record first
hits and register state; they do not skip functions, fake return values, patch
firmware instructions, alter pending-task masks, or supply successful ADC results.
The memory-corruption negative control is explicitly the sole RAM injection.

`Tasks::work` caller ranges come from selected ELF symbol sizes and LR, avoiding
the misleading claim that task calls during power startup prove entry into the
main loop. Both task scenarios run for a finite simulated interval (0.5 s by
default); the harness does not claim sustained scheduling, timing or load coverage.

## Deterministic hardware responses

MCU flash is bounded to 256 KiB, aliased at `0x00000000` and `0x08000000`; SRAM
is bounded to 48 KiB at `0x20000000`. The final 2 KiB flash page is reserved for
parameters, and a selected BIN extending into it is rejected. All flash begins
erased (`0xff`), then only the exact BIN bytes are loaded. The ELF is loaded with
`LoadSymbolsFrom`, never `LoadELF`: the latter may populate an ELF BSS load
segment at a flash physical address beyond the programmed BIN. CPU reset state
comes from the real vector table at `0x08000000`, not a synthetic jump to `main`.
There is no saved parameter fixture; parameter persistence is not tested.

The RCC Python peripheral stores register writes, immediately mirrors HSION,
HSEON and PLLON into their ready bits, and mirrors CFGR SW into SWS. For the
pin-reset scenario it supplies PINRSTF. CSR reset flags remain read-only and
writing RMVF clears them, including self-clearing RMVF. This is a software
handshake seam, not a clock-tree, oscillator timing or clock/reset-gating model.

CPU, NVIC, SysTick, UART, GPIO, timers and DMA come from the installed vendor
STM32F103 platform. Its generic FSMC mapped-memory window is a passive write
sink for the LCD bus; no display pixels, bus timing or LCD protocol are validated.
The generated NVIC configuration sets its effective SysTick input to 9 MHz,
matching Artoo's selected 72 MHz AHB/8 reference. This adjustment is needed
because [Renode NVIC source](https://github.com/renode/renode-infrastructure/blob/master/src/Emulator/Cores/Arm-M/NVIC.cs)
uses the configured frequency directly and exposes CLKSOURCE as fixed.
Application scenarios independently read the actual firmware's 64-bit
`tickBase` after the run and require 50 Hz coarse-time advances within two
rollovers below or one above the requested interval, with reload 179999.
This checks timer/ISR flow under the declared model; it does not establish
cycle accuracy or physical clock tolerances.
The platform's online SVD load and fixed RCC tag are removed. No other peripheral
is made to return success. Missing FLASH, PWR, ADC and FSMC-register accesses,
and unimplemented DMA/UART/GPIO bits, remain visible in full logs and the result's
warning list. Initialization milestones prove function entry, not faithful
electrical or peripheral behavior. Simulated PINRSTF selects a supported
software branch; it does not validate a physical reset or bootloader exchange.

## Failures and useful next checks

Unexpected default interrupt/fault handler or `abort` entry logs the handler,
LR, SP, CFSR and HFSR and halts the CPU. Missing milestones, a wrong task caller
or power state, a memory mismatch, Renode execution errors and host timeouts
fail the run. The result includes final PC/LR/SP with ELF symbol attribution;
the full monitor log retains warnings and Renode diagnostics. Unmodeled
peripheral warnings are evidence of coverage limits, not hardware success.

This complements the production-code native protocol, command, stick, button,
clock and IRQ fixtures. Small follow-ups with good software value are a valid
valid/invalid stored-parameter fixtures, finite captured UART input
that exercises actual receive dispatch, and a focused pending-task/heartbeat
assertion at the Running main-loop boundary. Each needs its own explicit input
contract and reviewed assertions. Avoid expanding into analog, charger, radio,
LCD-graphics or full-controller simulation solely to make boot look complete.

Physical verification remains necessary for flash erase/programming and saved
parameters, charging/power rails, ADC/sticks/battery accuracy, UART electrical
transport, LCD appearance, DMA error/timing behavior, IRQ races under real load,
stack margin, and controller/SoloLink integration.

## Local qualification, October 28, 2026

The reviewed saved wave3 candidate, source
`df93d875f6057a604b30869c109046ffdf0b3a49`, passed all three scenarios and both
negative controls under Renode 1.17.0. Evidence lives in
[`renode-wave3-qualification-r2-20261028`](/home/jresp/solo/.integration/artoo-builds/renode-wave3-qualification-r2-20261028/result.json).
The selected BIN is 251568 bytes, SHA-256
`1c4070662b50ecf06e7e8beadf34dd4ff8fa692356ef2a1974090ad54a9920a1`.
Its eleven init-array entries were reached in order, with SRAM initialization
checked before the first. The optional preinit array is absent in this candidate.

The power-wait scenario ended at `0x08006e30` in
`PowerManager::waitForCompleteStartup`; the pin-reset scenario ended at
`0x0800651c` in `main`, with actual Running-state task dispatch observed.
Both application scenarios recorded 24 real SysTick coarse-time increments
(480000000 ns) over the requested 0.5 simulated seconds, with reload 179999.
All 99/101 warning lines remain in the application results and full logs.
The clock-stall control stayed at `0x080072f4` in `_start`; the corrupted-BSS
control stopped before the first constructor at the memory assertion.

Python compilation passed. Candidate originals and copied inputs retained their
recorded hashes; deliberately pairing the saved wave2 ELF with wave3 BIN was
rejected before Renode execution (exit 2). The initial harness API failure,
using `RemoveHook` instead of the address-only `RemoveHooksAt`, is preserved in
[`renode-wave3-qualification-20261028`](/home/jresp/solo/.integration/artoo-builds/renode-wave3-qualification-20261028/result.json).
Qualification was rerun after that harness-only correction; firmware inputs
were unchanged. Later firmware candidates require their own run and receipt.

## Final candidate and bounded UART expansion, October 28, 2026

The final saved production DVT candidate, source
`66336f01c57de971a85c001aa627308871a3747f`, passed the same three boot scenarios
and two negative controls, plus 26 additional UART receive cases. The exact
BIN SHA-256 is
`d028b55d4822af7738196a14a44489c593794abcf7d0563ecdcc4b8ae0f215fb`.
Use the [final result](/home/jresp/solo/.integration/artoo-builds/renode-final-66336f0-qualification-20261028/result.json)
and [manifest](/home/jresp/solo/.integration/artoo-builds/renode-final-66336f0-qualification-20261028/manifest.json)
for current candidate evidence; the wave3 section above is historical.

```bash
candidate_dir=/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave4/final-66336f0/production-dvt
python3 /home/jresp/Active_GH/OPenSolo_2/tests/renode/run_artoo.py \
  --elf "$candidate_dir/artoo.elf" --bin "$candidate_dir/artoo.bin" \
  --receipt "$candidate_dir/verification.json" \
  --output /home/jresp/solo/.integration/artoo-builds/renode-final-new-run \
  --negative-controls --uart-checks
```

The optional UART scenario boots a fresh machine through the same actual
pin-reset/Running path, then feeds wire bytes using the existing STM32 UART
model's `WriteChar` input. Its effective USART1 reference is configured to
72 MHz, matching the firmware's APB2 setup and 115200 baud divider. Each case
runs only long enough for its paced bytes plus a small foreground-dispatch
margin. Nothing directly calls a firmware function, writes pending-task masks,
or changes firmware variables. The UART expansion added approximately 0.159
simulated seconds after the 0.5-second boot interval.

`gdb-multiarch` resolves actual member addresses, array capacities and message
IDs from the selected ELF's DWARF; the test contains no historical member
offsets. These addresses and the exact input vectors are retained in
[`uart-layout.json`](/home/jresp/solo/.integration/artoo-builds/renode-final-66336f0-qualification-20261028/uart-layout.json)
and [`uart-vectors.json`](/home/jresp/solo/.integration/artoo-builds/renode-final-66336f0-qualification-20261028/uart-vectors.json).
The receipt must explicitly contain `factory_tests: false` before these cases
can run. A separate negative preflight check confirmed that a factory-enabled
receipt is rejected with exit 2 before Renode starts.

| Cases | Actual software observations |
| --- | --- |
| Malformed escape, oversized frame, missing/extra SetRawIo bytes, recovery | Exact real RX byte-count deltas, expected rejection-count deltas, unchanged raw-I/O flag on rejected frames, then valid raw-I/O off/on updates. |
| Direct SysInfo and Linux single-space query, invalid SysInfo query, outgoing-only and unknown IDs | Both supported query envelopes accepted, invalid forms rejected by actual host message validation. |
| Production OutputTest, GpioTest, SelfTest magic and TestEvent requests | Each structurally valid factory request increments the rejection counter; production SetRawIo remains usable. |
| Full 64-byte and oversized shot label | The valid label reaches the actual 20-character UI field with its trailing NUL; the oversized label is rejected without replacing it. |
| Maximum pairing identity, missing terminator, hidden suffix, oversized identity and matching result | Actual stored identity accepts 31 characters plus NUL, invalid text preserves the previous identity, the oversized envelope is rejected, and a valid result clears the identity. Consumer-level text rejection correctly does not invent an envelope rejection count. |
| Zero, valid 2/4-second and reversed sweep configuration | Invalid inputs increment rejection counts and preserve stored values; valid input updates the real parameter fields. |

All 26 cases passed: 636 UART bytes were consumed through the real ISR/ring/task
path, with 15 expected envelope rejections, zero UART overruns, zero ring drops
and zero line errors. The boot timer check again observed 24 coarse-time
increments (480000000 ns) with reload 179999. Final UART PC was `0x08006134`
in `main`. All input copies and originals preserved their hashes, frozen
harness hashes matched, and Python compilation passed.

The full UART log retains 1407 warnings, dominated by 636 requests each to
disabled DMA channels 4 and 5 from the vendor UART model. These warnings do not
invalidate the independently counted IRQ receive path. They also make the
limits explicit: this pass does not qualify UART TX/DMA response bytes. SysInfo
coverage here establishes request admission, not decoded response contents.

The expansion intentionally stopped after these easy software checks. GPIO
button-edge stimulus, valid/invalid stored-parameter fixtures, heap execution,
and analog-driven camera/filter work were not added. The widened moving-average
arithmetic remains covered by the actual production native UBSan fixture;
Renode supplies no ADC completion stream to drive that camera task. No new ADC,
charger, LCD, radio or DMA model was introduced, and no physical targets were
contacted or changed.
