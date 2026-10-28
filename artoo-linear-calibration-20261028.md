# Artoo linear rate-dial calibration correction — October 28, 2026

Hardware preflight, before an MCU flash, exposed a validation regression in the
local correctness work: the sixth calibration record belongs to the linear
rate dial, whose unused trim can legitimately be zero. Requiring an angular
center for all six axes rejected the controller's existing factory calibration.
The preceding native/emulator qualification did not cover that real record.

Firmware commit `89fc1d4` corrects this without changing calibration bytes,
ADC mapping, the 172-byte parameter record, or its reserved flash page.
The coordinated Linux updater correction is SoloLink `46d677a`.

The observed raw-axis triples, in physical ADC order, were:

```
733,2299,3701
806,2261,3699
778,2304,3719
754,2127,3511
896,2024,3257
1191,0,2751
```

Raw inputs 0–3 remain angular and require `min + 1 < trim < max - 1`.
Raw input 4, gimbal pitch, requires `min + 250 < trim < max - 250`.
Raw input 5, gimbal rate, requires only `0 <= min < max < 4096`; its unused
16-bit trim is preserved verbatim, including `0` or `65535`. Every axis still
requires valid ADC endpoints. Erased all-`65535` records remain invalid.

`Params::StickCalibration::isValid(rawID, deadZone)` now supplies the shared
rule for `StickAxis::setCalibration`, boot validity initialization, and the
all-or-none `Inputs::applyStickParams` transaction. A rejected new calibration
changes neither live axes nor stored bytes. Invalid persisted endpoints remain
invalid, including during a configuration-only transaction. An accidental
angular read of the linear dial returns neutral instead of interpreting its
unused trim; actual camera angular control uses raw input 4. Linear math,
direction, filtering and mapping are unchanged.

## Local qualification

Source was frozen and reviewed together with the updater before testing.
`tests/check_calibration.py` links the actual `Inputs.cpp` and `StickAxis.cpp`
with narrow ADC/UI substitutes. ASan/UBSan checks passed for the supplied
six-axis snapshot and configuration, production boot mask, transaction
publication/rejection, exact trim preservation, every sample across the observed
linear span in both directions, smallest/widest legal endpoint spans, invalid
stored endpoints, and strict angular dead-zone failures. The existing
reliability suite also passed. The fixture performs no ADC acquisition, serial
operation, UI rendering, or vehicle command; the boot-mask call is a native
software test rather than a physical initialization test.

Native evidence:
[/home/jresp/solo/.integration/artoo-builds/linear-calibration-20261028/native/results.json](/home/jresp/solo/.integration/artoo-builds/linear-calibration-20261028/native/results.json).

All four profiles passed the cross-build, compiler-profile checks, vector and
constructor inspection, heap/stack bounds, and byte-for-byte comparison of all
765 linked asset arrays against `6eddd333`. The existing graphics/pixel suites
remain bound to that unchanged graphics implementation and asset corpus; they
were not relabeled as newly executed tests.

| Profile | Binary bytes | Change from Tamp baseline | Flash headroom |
| --- | ---: | ---: | ---: |
| Production DVT | 190,956 | +64 | 69,140 |
| Production BB03 | 190,892 | +64 | 69,204 |
| Factory DVT | 192,356 | +56 | 67,740 |
| Factory BB03 | 192,292 | +56 | 67,804 |

All profiles retain 6,840 bytes static RAM, 38,216 bytes bounded heap capacity,
and the minimum 4,096-byte stack reserve. Production DVT has initial SP
`0x2000c000`, reset vector `0x08006edd`, and 11 constructors.

[Production DVT receipt](/home/jresp/solo/.integration/artoo-builds/linear-calibration-20261028/production-dvt/verification.json)
binds exact source `89fc1d44680cac36c506bba4032078d2fb500160` to:

- BIN SHA-256 `124cf1c3cde4715f2f458f9c621c3ae157dc5a761a6f1d0efa2b4fd09296bbf3`.
- ELF SHA-256 `0236f2b694c23be1544f7961c831de762b5f54c96dc2635a0b1bc1d28a611ef7`.

[Canonical Renode result](/home/jresp/solo/.integration/artoo-builds/renode-linear-89fc1d4-qualification-20261028/result.json)
passed all 32 checks on this exact saved candidate: three startup/task scenarios,
two negative controls, 26 UART cases, and actual Tamp splash rendering through
the production C decoder. Inputs remained unchanged during the run. The
bounded platform and RCC readiness shim retain their documented peripheral
model gaps; this is software qualification, not a physical ADC, display,
charging, or timing claim. The parent accepted the local software candidate.
Earlier immutable receipts remain preserved under `tamp-integration-20261028`.

This firmware subtask performed no hardware access or MCU flash. Physical
qualification and deployment remain separately owned by the parent task.
