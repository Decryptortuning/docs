# Artoo bench idle option — October 28, 2026

An explicit `bench` build profile disables only the normal idle countdown for
local development. Production and factory profiles retain their existing idle
warning at ten minutes and shutdown request seven seconds later. The bench
profile keeps factory features disabled. It introduces no persistent setting,
wire field, calibration change, fake vehicle connection, or updater-screen hold.

Source is frozen at `48034572145e9e62dff881141277564e1ed1d3a6` on top of the
qualified linear-calibration/Tamp baseline `89fc1d4`. The sole runtime change
initializes `IdleTimeout::userDisabled` from the default-off
`ARTOO_BENCH_NO_IDLE` build macro. The existing countdown, reset and disable
combo methods are unchanged. `PowerManager`, manual button shutdown, battery
logic, task dispatch, UI state, and the 172-byte stored record are unchanged.
This avoids the earlier updater-screen workaround, whose UI state also blocks
normal shutdown through `PowerManager::canShutDown()`.

## Selecting and identifying an image

```
./scripts/build-wsl.sh bench dvt
./scripts/build-wsl.sh bench bb03
```

Omitting arguments still selects normal `production dvt`. Normal production
firmware restores the default idle countdown on subsequent boots. There is no
runtime enable/disable protocol extension in this change.

Bench version: `5.0.0-dev+g48034572145e-bench-noidle`.
Normal version: `5.0.0-dev+g48034572145e`.
Both fit the Linux runtime's 64-byte version field. These remain local OpenSolo
5 development identifiers, not upstream release versions.

Each wrapper build writes `build-manifest.json` with the exact embedded
version, profile, board, factory/bench flags, source/config identity, and
ELF/BIN/HEX/map hashes. It verifies generated compiler flags against the selected
profile and confirms the version string occurs in the binary. Copy this
manifest with the saved artifacts. The versioned packaging helper verifies
artifact hashes and uses the manifest version; it does not derive a deployment
version from whatever Git checkout happens to be current. Direct Tup builds do
not produce this wrapper manifest. No new image was promoted to Yocto or
selected for flashing by this subtask.

## Qualification

The coordinated source review preceded all tests and builds. Native sanitizer
tests of the actual `IdleTimeout.cpp` passed in three modes: macro omitted,
explicit zero, and explicit one. The default modes still warn and request
shutdown at the exact original thresholds, reset correctly, pause for the
existing link/updater conditions, and honor the original three-button disable
combo. Bench mode does not warn or request idle shutdown through several timeout
periods and resets, while UI remains normal and the vehicle remains disconnected.
The fixture uses inert external input/UI/shutdown sinks, so it makes no claim
about physical power shutdown. Disposable-tree tests of the real packaging
helper preserve the bench suffix and reject artifact hash changes. Actual Inputs
calibration and the existing reliability regression also passed.

[Native results](/home/jresp/solo/.integration/artoo-builds/bench-idle-20261028/native/results.json).

All six profile/board combinations passed cross-build, exact compiler flag and
manifest/version checks, vector/constructor checks, and linker stack/heap bounds.
All 765 linked generated asset arrays are byte-identical to `89fc1d4` in each
profile. Previous qualified artifacts remain preserved.

| Profile | Board | BIN bytes | Flash headroom | Static RAM |
| --- | --- | ---: | ---: | ---: |
| Production | DVT | 190,956 | 69,140 | 6,840 |
| Production | BB03 | 190,892 | 69,204 | 6,840 |
| Factory | DVT | 192,356 | 67,740 | 6,840 |
| Factory | BB03 | 192,292 | 67,804 | 6,840 |
| Bench | DVT | 190,976 | 69,120 | 6,848 |
| Bench | BB03 | 190,912 | 69,184 | 6,848 |

The four existing profiles retain baseline sizes and RAM use. The bench profile
costs 20 additional flash bytes and 8 static RAM bytes, including version and
initialized-data/alignment differences. Bench bounded heap capacity is 38,208
bytes below the unchanged minimum 4,096-byte stack reserve. Initial SP remains
`0x2000c000`, reset vector `0x08006edd`, and all 11 constructors are retained.
These static checks do not establish complete stack nesting or physical latency.

[Bench DVT receipt](/home/jresp/solo/.integration/artoo-builds/bench-idle-20261028/bench-dvt/verification.json)
and [build manifest](/home/jresp/solo/.integration/artoo-builds/bench-idle-20261028/bench-dvt/build-manifest.json)
bind the selected artifact:

- BIN SHA-256 `b4909e99ef10f8ef723b8e3b3fb39edb5eb0be58234704c88f84a8d5b3b1a2da`.
- ELF SHA-256 `e4dfa003adf1281a9f21fe7e2f77da39622c9b9cfb865eecaee670218e211286`.

The unchanged canonical Renode harness passed all 32 checks on **both** saved
DVT candidates:

- [Bench DVT result](/home/jresp/solo/.integration/artoo-builds/renode-bench-4803457-qualification-20261028/result.json).
- [Normal production DVT result](/home/jresp/solo/.integration/artoo-builds/renode-production-4803457-qualification-20261028/result.json).

Each run covers three positive startup/task scenarios, two negative controls,
26 UART cases, and actual Tamp splash decoding with success returned and main
loop continuation. Inputs did not change during either run. The emulator's
bounded memory platform, RCC readiness shim, and retained peripheral warnings
remain explicit model limits. The long idle-countdown behavior is established
by the native actual-function fixture; Renode is not presented as a ten-minute
physical power simulation.

This wave is local only: no SSH, serial/GPIO access, hardware resets, flashing,
or pushing were performed by this agent. Physical battery/manual shutdown and
other peripheral behavior retain their separate hardware acceptance limits.
