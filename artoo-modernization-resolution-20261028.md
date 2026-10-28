# Artoo reliability implementation ledger — 2026-10-28

This tracks local implementation of the findings in [the audit](/home/jresp/solo/workspace/docs/artoo-modernization-audit-20261028.md). The baseline is Artoo `60a2707305e671137ed11f2e406964a8e36cd2b0`, with its existing `opensolo5-branding` receipt preserved. The controller Linux build is set aside; these changes do not update its source pins or packaged firmware.

Accepted local software baseline: **`66336f01c57de971a85c001aa627308871a3747f`**. Four production/factory DVT/BB03 profiles passed their software checks; the final production DVT image also passed canonical Renode boot and UART regression. Physical acceptance and design gates are listed at the end.

All work remains local. There is no flashing, serial/GPIO/probe access, target connection, or push. Native tests and cross-builds begin only after each frozen source wave has received the parent's producer/consumer review. Tests do not establish physical qualification.

## Wave 1 — transport and bounded text

Candidate: Artoo `2187860`, followed by producer-contract review correction `f346bed` and host-fixture ABI correction `3e0c6ff`. Status: source reviewed; native tests and cross-build passed. Accepted by parent as the software baseline for wave 2.

- A01: checked packet appends with sticky failure; separate 264-byte raw and 530-byte encoded storage; no DMA send of a failed aggregate; compile-time aggregate-input and 172-byte parameter-layout checks.
- A02: button events use the production SLIP encoder, including the binary all-buttons mask.
- A03: production bounded SLIP decoder rejects complete malformed/oversized frames; UART byte loss invalidates queued data and requires a new delimiter; message length validation and bounded label decoding; pairing identity bounds match the Linux sender.
- Button label storage is explicitly truncated to 19 characters plus NUL within the unchanged 20-byte field. Full 24-byte wire configurations and header-only empty labels are accepted. Shot text accepts up to the Linux producer's 64 bytes and is terminated in a temporary local buffer.
- Parent review identified that Linux `SysInfo::ping()` sends one space, while the boot updater sends ID3 alone. Both actual request forms are now validated, with the one-byte form restricted to `0x20`.
- Hardware-free tests passed using the actual production codecs and actual Linux SLIP implementation under ASan/UBSan, all 41,472 button mask/ID/event combinations, maximum/overflow boundaries, malformed streams/recovery, one million deterministic stream bytes, and text/length contracts. The independent MAVLink CRC/unaligned UBSan regression also passed.
- The first native test attempt failed because actual SoloLink SLIP assumes unsigned ARM `char` but x86 defaults to signed `char`. That failed log is retained. After review, the test was changed to model the target ABI with `-funsigned-char` and a compile-time assertion. Linux source was unchanged; this does not claim its decoder is portable to signed-char targets.
- [Wave 1 receipt](/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave1/verification.json): binary 245,976 bytes (+1,232); flash gap 14,120; static RAM 5,948 bytes (+288); initial SP `0x2000c000`; reset `0x08005f0d`; 11 valid constructors. All 765 linked asset byte arrays match the original branding baseline, including the previously validated logo and palette. The compile-time input aggregate bound passed the cross-build. Libnosys warnings remain visible.
- RX tests establish bounded codec framing and recovery. At this wave A07 remained open; wave 3 subsequently repaired the reviewed ownership paths. Physical UART stress and full scheduling acceptance remain.

## Wave 2 — persistence, semantic validity, IRQ shutdown and response identity

Candidate: `ff928f0`, `a8ced69`, `2b8d9e2`, followed by review correction `7663415` restoring the shared button-backlight shutdown call. Status: reviewed, qualified locally and accepted by the parent before wave 3.

[Wave 2 receipt](/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave2/verification.json): binary 247,368 bytes (+1,392 from wave 1), flash gap 12,728 bytes, static RAM 5,980 bytes (+32), initial SP `0x2000c000`, reset `0x08006475`, 11 valid constructors. All 765 linked generated asset arrays remain byte-identical to the original branding baseline. Reliability ASan/UBSan/float-sanitizer tests, actual Linux SLIP/transport tests and MAVLink CRC UBSan regression all passed on the frozen source. Known libnosys warnings remain. No hardware access or firmware packaging occurred.

- A04: production `ParamStore::write` relocks on erase/program/final-status failure, checks the final halfword completion, and verifies all 172 bytes by readback. `Params::save` snapshots under a PRIMASK-preserving guard and clears dirty state only when the saved generation still matches. Failed transactions retain pending state. The single reserved page remains non-atomic across power loss; flash controller busy waits remain a hardware/fault-policy gate.
- A05: new six-axis transactions validate before mutation. Effective mappings must be a permutation of the four RC inputs, with both direct gimbal inputs preserved. Mode 1 `[3,0,1,2]`, mode 2 `[3,2,1,0]`, independent `0xff` default fields, arbitrary nonzero forward direction, and reserved sentinel bytes remain supported. Invalid stored mappings retain their bytes and suppress control validity/reports until explicitly repaired; raw diagnostics remain available. Calibration denominators include the actual axis dead zone; nonfinite presets and overflowing/reversed/zero sweep ranges are rejected. Uncalibrated arithmetic returns neutral diagnostic values while control validity is false. Axis raw identity now follows the mapping for fault attribution. Floating-point preparation occurs outside the short publication critical section.
- A08: `Button::disableIRQ` disables its EXTI mask. Shutdown visits every button except the power/wake button exactly once.
- A10: track the serialized MAV_CMD rather than the internal request enum. Responses require the vehicle sysid/component and the expected response kind. SET_MODE completes from a matching vehicle heartbeat, mission requests from the matching sequence, and COMMAND_LONG from a matching COMMAND_ACK. A five-second tracking deadline clears stale tracking/takeoff continuation but sends no retry. Background home queries defer behind tracked commands; pending disarm has priority over pending land/RTL/pause, which have priority over ordinary intent. New deliberate intent may replace an already-sent command. **MAVLink 1 cannot distinguish late responses to two requests sharing the same command ID (including arm/disarm 400); this remains explicit protocol ambiguity.** Existing authorized takeoff continuation requires confirmed LOITER telemetry; native tests send no vehicle commands.
- A11: reject impossible top/bottom ADC ordering/range before unsigned subtraction; disable charge and balancing outputs for that invalid sample, retaining the last valid level until later samples recover. Publish the new battery level before notifications read it. No empirical voltage, thermal, charge, filter or alert thresholds changed. Physical sensor-fault/charging acceptance remains required.
- Passed native coverage: actual production flash transaction with failures at all 86 halfwords, final failure/readback mismatch/erase failure; dirty-generation completion policy; semantic domains and shipped mappings; actual `StickAxis` arithmetic over 3,801 samples per dead-zone profile under ASan/UBSan/float sanitizers and `-ffast-math`; command identity/source/replacement/expiry; nested interrupt-mask restoration; actual button/GPIO EXTI-mask helper. Existing transport/actual Linux SLIP and CRC regressions also passed. Full UI/battery GPIO behavior, end-to-end live mapping publication, interrupt timing and real flash power failure are not simulated by these fixtures.
- Wave 3 still owns general ISR/application state ownership, UART publication/reset ordering, coherent SysTime and DMA errors. Wave 2 does not claim those concurrency defects fixed.

## Later asset tuning experiment

The immutable wave 1 image contains 765 linked generated asset arrays totaling 156,773 bytes. The parent completed a separate benchmark of that frozen corpus against pinned heatshrink and Tamp codecs; see [the compression report](artoo-compression-20261028.md). Firmware keeps its current palette/RLE implementation through these correctness waves. Any later integration must measure net flash after decoder cost, fixed RAM, display latency and bounded work, and verify exact RGB565 output. A full 320×240 RGB565 framebuffer exceeds the MCU's entire 48 KiB RAM and is not an option.

## Audit resolution status

| Finding | Current status |
| --- | --- |
| A01 packet capacity/bounds | Wave 1 source and native/cross-build validation complete; hardware acceptance pending |
| A02 button SLIP escaping | Wave 1 source and native/cross-build validation complete; hardware acceptance pending |
| A03 payload/text/framing | Wave 1 source and native/cross-build validation complete; physical UART/ISR acceptance pending |
| A04 persistence failure handling | Wave 2 source/native/cross-build validation complete; hardware acceptance pending; single-page power-loss atomicity remains a separate format/migration design |
| A05 parameter semantics | Wave 2 source/native/cross-build validation complete; hardware acceptance pending; preserve supported stick modes and each `0xff` field-default sentinel |
| A06 time snapshots | Wave 3 software validation passed; collapsed ticks cannot reconstruct unobserved elapsed time |
| A07 ISR/shared state | Wave 3 software validation passed for reviewed ownership/publication changes; full scheduler and physical timing acceptance remain |
| A08 button IRQ shutdown | Wave 2 source/native/cross-build validation complete; hardware acceptance pending |
| A09 DMA error completion | Wave 3 software validation passed; real peripheral-error acceptance remains |
| A10 command identity/lifecycle | Wave 2 source/native/cross-build validation complete; hardware acceptance pending; no automatic flight-command retries; preserve urgent command replacement |
| A11 battery ordering/sample arithmetic | Wave 2 source/native/cross-build validation complete; hardware acceptance pending; empirical thresholds and charging policy require hardware evidence |
| A12 factory isolation | Production gates and separate factory profile implemented and verified; raw calibration diagnostics retained; physical factory acceptance remains |
| A13 startup/linker/runtime/stack | Retained vectors/init arrays, heap/stack bounds and machine-checked image contracts implemented; final Renode startup/task/UART checks passed below; watchdog/reset/fault policy and physical stack/timing remain separate |
| A14 scheduling/control timing | Bounded RX, foreground ownership and atomic task handoff validated in wave 3; hardware control/display latency and coalesced-tick limits remain; retain existing control math/filter behavior |
| A15 graphics/assets | Bounded production RLE, metadata limits, newline/missing-glyph/empty-rectangle/placement fixes and all-asset pixel checks passed; general pixel clipping and physical LCD latency remain |
| A16 latent helpers/board support | Confirmed timer/NVIC/DMA/bit/filter edges repaired; BB03 digital backlight restored and four profiles compiled; BB02 and unused peripheral variants remain unqualified |

The original `build-env.md` edit is preserved. No library, compiler, MAVLink dialect, RTOS, HAL, parameter format, parameter page address, battery threshold, or flight-control retry policy has changed.

## Wave 3 review contract — time, ownership, bounded work and DMA errors

Reviewed source `3933ec7`/`e3e2ebe`, with fixture-only corrections `78bcb0b` (event name shadowing) and `df93d87` (board LED sentinel identity). Both initial fixture compile/link failure logs are retained. Status: all native tests and cross-build passed on clean `df93d87`; accepted by the parent before wave 4.

[Wave 3 receipt](/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave3/verification.json): binary 251,568 bytes (+4,200 from wave 2), flash gap 8,528 bytes; static RAM 6,532 bytes (+552); SP `0x2000c000`, reset `0x08007295`, 11 constructors. All 765 linked asset arrays match the original branding baseline. Native concurrency, reliability, transport/actual Linux SLIP and CRC regressions passed; production SysTime disassembly confirms PRIMASK save/mask/restore around the full capture and publication. Libnosys warnings remain.

- Clock reads and SysTick's 64-bit base update preserve PRIMASK, including nested ISR callers. The production capture helper samples VAL/pending/VAL, accounts for a pending rollover, and clamps the last published value. It never claims to recover multiple periods collapsed into one pending bit. LCD TE's 64-bit timestamp also receives a guarded snapshot.
- `RingBuffer` uses lock-free acquire/release index publication and rejects full enqueue. UI producers and button-edge producers serialize with short PRIMASK guards. Task-mask capture is a single atomic exchange so same-bit work posted during dispatch remains pending. These operations do not change NVIC priorities.
- Only foreground HostProtocol task constructs packets and owns its DMA buffer. RX consumes at most 256 queued bytes per invocation; incomplete frames remain in the decoder, and remaining work retriggers the task. TX has an explicit enabled/busy policy; requests cannot re-enable disabled TX. Cancellation disables DMA and clears its peripheral flags. Completion/error only releases ownership and schedules foreground work. Failed transmissions are dropped, never replayed.
- USART/ADC/SPI completion callbacks now carry success, ignore nonterminal flags, and treat error as dominant over simultaneous complete. ADC errors never consume partial samples and withhold control validity until successful capture. DMA2's shared channel 4/5 vector dispatches both channels; invalid registration indices are rejected. SPI is currently an unused driver, so source coverage is not board qualification.
- Completed ADC samples are copied before rearming DMA. Foreground packet production snapshots all six axes, validity and completed raw samples under one short guard; DSM and input/raw reports use that snapshot. Raw diagnostics no longer read a buffer while DMA is changing it. Invalid-stick records publish only after fields are filled and consume through a guarded copy/reset.
- EXTI captures physical button level and timestamp into a 31-entry queue. Foreground processes FIFO edges, updates historical gesture/mask state, then dispatches application callbacks. Hold polling uses the existing 50 Hz heartbeat: thresholds are unchanged, but event time is quantized by up to 20 ms plus foreground scheduling latency. This requires hardware acceptance. Overflow drops incomplete queued history, resynchronizes physical levels, and suppresses already-held gestures until release/new press; it emits no synthetic click/hold from lost history. An already-dispatched event cannot be recalled if overflow happens later.
- Boot-held power is sampled during button initialization. `PowerManager::waitForCompleteStartup` now services the foreground tasks while preserving its physical-power continuation condition. Its only call in main occurs after HostProtocol, ButtonManager, UI, CameraControl, FlightManager, Inputs and Haptic initialization. Initial ADC capture starts once, after parameters and validity state are ready. Idle power-button wake uses foreground dispatch.
- Battery charging/sample calculations retain their ISR cadence. Battery-level alert/dismiss processing moves to the foreground heartbeat after publication, avoiding ISR mutation of the alert manager. Charge/voltage/filter thresholds remain unchanged.
- Passed native validation exercises the real time-capture policy with scripted rollover/pending/coalesced cases, production ring publication with one producer and guarded competing producers, actual Button gesture methods with captured timestamps/boot-held inputs/quarantine, actual DMA shared vector/register dispatcher, and the production TX ownership state. Register adapters do not emulate analog values or physical timing. Existing reliability/transport/CRC fixtures and full cross-build also passed.

### Lightweight software boot target, separately sequenced

The initial QEMU startup-seam proposal was superseded by the canonical Renode 1.17 runner in `/home/jresp/Active_GH/OPenSolo_2/tests/renode/run_artoo.py` (integration commit `bf106b628f2360aea59a05846c2561cf6dbd2004`); see [the Renode guide](artoo-renode.md). It loads the exact saved binary and ELF symbols into bounded 256 KiB flash/48 KiB RAM with a test-only RCC readiness/status shim. No production startup seams are needed.

The parent reports the saved wave 3 image passed actual vector/startup execution, poisoned-RAM data/BSS checks, all 11 constructors, main entry, foreground task dispatch in the power-wait Boot state and the PINRSTF path into Running. Both task scenarios observed 24 SysTick rollovers over 0.5 simulated seconds with the corrected 9 MHz reference. Two negative controls also passed. This is bounded software execution, not physical controller qualification; recorded unmodeled peripheral warnings and clock-model seams remain material limits. Wave 4 must rerun the canonical runner against its own saved final image.

## Wave 4 review contract — production profile, runtime bounds and renderer/helpers

Source frozen, reviewed and accepted through final `66336f0`; native tests, four-profile cross-build qualification and canonical Renode results are recorded below. Production and factory builds now have explicit
tracked Tup configurations for DVT and BB03. Production preserves SetRawIo and
calibration diagnostics but rejects OutputTest/GpioTest/SelfTest/TestEvent; the
factory-only handlers are compiled out of the production dispatch. Factory
self-test saves/restores each tested pin's control/output state on success or
failure. Factory hardware acceptance remains separate.

Linker changes retain vectors, preinit and priority-sorted init arrays, preserve
ARM attributes, assert the 304-byte F103 vector layout and reserved parameter
boundary, and reserve a **minimum** 4 KiB stack region. `_sbrk` now rejects heap
growth/shrink outside linker-owned bounds. GCC stack-usage reports and a new
read-only image check quantify static frames and remaining memory; they are not
a complete nested stack proof. No watchdog, reset response or oscillator-timeout
policy has been invented. Startup clock comments now reflect the actual 72 MHz
HCLK/APB2, 36 MHz APB1 and 12 MHz ADC configuration.

The palette/RLE stream bytes remain unchanged. Generated metadata adds encoded
lengths (glyph length uses existing ARM padding; each image descriptor grows by
four bytes). The production decoder checks input length, EOF, palette indices
and pixel quota, preserving transparent RGB565 values and the run output loop.
A production line visitor fixes trailing/adjacent-newline traversal. Native tests
regenerate assets into a temporary directory, compare every image/glyph's
RGB565 checksum with independent decoding, and exercise truncation/malformed
streams; timing and LCD rendering still need physical acceptance.

Reachable helper hardening includes bounded mode-bit tests, defined zero/full
rotations and zero bit counts, saturating INT_MIN absolute value, complementary
timer bit positions/channel bounds, TIM8 clock support, and masked NVIC priority
replacement. These fixes do not assign new NVIC priorities. UART5 DMA is explicitly
unsupported on this F103 rather than borrowing unrelated DMA channels; DMA
registration failure is checked by driver callers before dereferencing channels.
Unused peripheral variants remain unqualified.

Completed native/cross-build scope: new runtime/register/heap/factory policy and all-asset tests,
existing concurrency/reliability/transport/CRC regressions, production DVT and
BB03 builds in both production and factory profiles, linker/vector/constructors/stack/handler
presence checks, and original-byte asset comparisons. The separate integration
agent owns the canonical Renode harness, so no duplicate firmware runner or
production startup seam is being introduced. The canonical runner uses only a saved frozen ELF/BIN pair with its exact scope and hardware-model limitations recorded.

Pre-freeze review caught and repaired an overly broad linker text replacement before any wave 4 test/build. Every section-end symbol again uses its own location counter; there is one top-level heap/stack assertion block.

### Final wave 4 qualification and review corrections

The final source is `66336f0` (full revision in each receipt), following `071ba39` (runtime/factory/graphics/helpers), `cc00c95` (reviewed graphics edges/asset limits), `400014a` (generated Tup dictionary ignore), `dfc249e` (BB03 backlight), and fixture alignment correction `a43257f`.

- Review repaired the pre-freeze linker replacement error and added empty rectangle no-op behavior, missing monospace-box no-op behavior, bounded right/center placement and generator byte/uint16 storage validation. Valid text placement retains its prior rounding. Overwide labels begin at the supplied origin; general LCD pixel clipping remains a separate design item.
- Explicit Tup configurations generate `tup-generate.vardict`; the first factory DVT build/image check passed but clean-tree receipt generation stopped on that unignored generated file. Its outputs and logs remain in `wave4/factory-dvt-attempt1`. The ignore-only correction is retained separately.
- Actual BB03 compilation found its existing unconditional DVT PWM-channel reference. The repaired setter preserves DVT TIM3/channel-4 PWM and uses BB03's existing PC9 high/on, low/off semantics, with percent clamped to 100. No PWM mapping was invented. The failed build log remains in `wave4/factory-bb03-400014a-failed`.
- The actual LCD-driver register fixture first failed because its host GPIO object lacked the 256-byte alignment required by production GPIOPin's encoded address. The fixture was corrected; the first ASan failure remains in `backlight-attempt1.log`, and both board fixtures pass in `backlight-attempt2.log`. This is an adapter correction, not a hardware GPIO fault.
- The final A16 ledger pass confirmed that the filter's stored-average product still multiplied in int32. Large accepted custom sweep values made that overflow reachable. Widening that operand to int64 preserves weights, the `+32768` rounding, division and alpha-65535 fast path. The real filter now passes nominal battery/camera golden sequences, signed boundaries, valid large sweep values and 100,000 deterministic steps against a wider oracle under ASan/UBSan/float sanitizers and fast-math. No control filter tuning changed.

The six original native suites passed on `cc00c95`: runtime/register/heap/factory policy; actual Gfx wrappers with a narrow LCD sink; all 800 generated images/glyphs totaling 405,717 RGB565 pixels compared with independent decoding; 100,000 malformed RLE trials; concurrency; reliability/flash fault injection; transport/actual Linux SLIP; and MAVLink CRC. Graphics is one suite with both wrapper and corpus checks. The affected actual LCD-driver fixture passed on `a43257f`; the actual moving-average fixture passed on `66336f0`. Exact source-history diffs explain reuse of unchanged suites, rather than falsely claiming they reran at every version-only rebuild.

Every profile's generated compiler command is checked for the correct `-DBOARD=2/3` and `-DARTOO_FACTORY_TESTS=0/1`; each receipt records its identity. Factory handlers must be linked in factory images and absent from production. Post-link checks retain ARM ABI attributes, validate all 76 vectors, initial SP/reset, 11 constructor pointers and the empty preinit array, data/BSS/flash bounds, the reserved parameter page, `_sbrk`, lock-free atomic imports and compiler stack frames. Every profile retains all 765 linked baseline asset byte arrays exactly.

Final artifacts live under `/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave4/final-66336f0/`. Earlier `a43257f` four-profile receipts remain separately preserved at the wave4 root and are interim evidence. No candidate has been packaged into Yocto or accepted on hardware.

| Final profile | Binary bytes | Gap before parameter page | Static RAM bytes |
| --- | ---: | ---: | ---: |
| Production DVT | 250,364 | 9,732 | 6,528 |
| Production BB03 | 250,300 | 9,796 | 6,528 |
| Factory DVT | 251,764 | 8,332 | 6,528 |
| Factory BB03 | 251,700 | 8,396 | 6,528 |

[Final production DVT receipt](/home/jresp/solo/.integration/artoo-builds/modernization-20261028/wave4/final-66336f0/production-dvt/verification.json). Its version string is `5.0.0-dev+g66336f01c57d`, initial SP `0x2000c000`, reset `0x08006c25`, binary SHA-256 `d028b55d4822af7738196a14a44489c593794abcf7d0563ecdcc4b8ae0f215fb`. Compared with the preserved branding baseline, production DVT adds 5,620 flash bytes and 868 static RAM bytes; compared with wave 3 it saves 1,204 flash bytes. The bounded heap capacity is 38,528 bytes below the minimum 4,096-byte stack reserve. The `.su` report covers 513 compiled functions (including linker-discarded functions), with a largest individual static frame of 288 bytes. This is not a complete call-chain or ISR nesting bound, nor an allocation or physical high-water measurement. Existing libnosys warnings remain visible; `_printf_float`, toolchain/library versions and the existing asset codec were retained.

### Final canonical Renode evidence

[Final Renode result](/home/jresp/solo/.integration/artoo-builds/renode-final-66336f0-qualification-20261028/result.json) passed against the exact final production DVT BIN/ELF/receipt, with no input changes during execution. Renode version is `1.17.0+20260907gitf1dd1b4af`. The parent independently verified hashes, harness provenance and acceptance.

Three positive scenarios executed real vectors/startup, poisoned-RAM data/BSS initialization, all 11 constructors, main, foreground dispatch during the power-wait Boot state, and the PINRSTF route through production Running state. Both 0.5-second task scenarios observed 24 SysTick rollovers (480 ms accumulated), reload 179,999 and a 9 MHz reference clock. The clock-stall and memory-corruption negative controls were detected as intended. The additional 26-case UART regression passed production framing/length/state handling, SysInfo producer variants, malformed and oversized recovery, raw diagnostic toggles, production factory-request rejection, text/pair bounds and sweep semantics.

The harness is deliberately a bounded software model: RCC readiness/status is supplied by a test shim, and ADC/LCD/flash/power/DMA details are not physically represented. It retains 99 warnings in power-wait, 102 in Running and 1,407 in the UART scenario rather than hiding those model gaps. These checks do not establish real UART baud/framing timing, DMA delivery, charging safety, analog calibration, LCD appearance or physical boot. The separate cross-build receipt's `renode_status` records that it was pending when that immutable receipt was written; the linked later Renode result supplies completion.

### Remaining acceptance and design gates

- **Before hardware acceptance (P1):** validate real control/diagnostic inputs, both supported mappings, invalid-calibration lockout and repair, UART loss/error recovery, button boot/hold/shutdown behavior, real flash status/readback, charging and power behavior on the intended board. No physical qualification has been inferred from host registers or emulation.
- **Persistence and failure policy:** the existing single parameter page is not power-loss atomic. A journal/alternate page requires a reviewed format/migration design. Flash/oscillator/peripheral busy waits, unexpected interrupt/fault records and watchdog/reset behavior need a coordinated safe-output/recovery policy before changing them.
- **Time and scheduling:** masked/coalesced SysTick periods cannot be reconstructed. Button holds have up to 20 ms polling quantization plus foreground delay. Full UI/ADC/UART scheduling latency, interrupt nesting, camera/control timing, flash stalls and the work-check/WFI boundary require focused timing acceptance. Native concurrency tests do not prove the entire scheduler.
- **Wire identity:** MAVLink 1 has no nonce for two commands sharing an ID; a late arm/disarm ACK can remain ambiguous. Wire/source checks do not remove that protocol limitation. No automatic command retries were added.
- **Display and board scope:** general pixel clipping/offscreen caller handling remains outside these bounded-decoder/placement repairs. Physical LCD timing, factory pin tests and unused peripheral variants remain unqualified. BB02 is not in the supported matrix; UART5 DMA is explicitly unsupported on this part.
- **Later modernization:** codec integration, LTO, CMake/Ninja migration, newer GCC/newlib/CMSIS/ST references and any MAVLink regeneration remain separate measured changes after this correctness baseline. The compression benchmark is evidence for an experiment, not production integration or physical display timing proof.

The user-owned `build-env.md` bytes remain unchanged at SHA-256 `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`. OpenSolo 5 remains local development branding, not a new upstream release or a claim of complete hardware modernization.
