# Python 3 compatibility review — October 28, 2026

The active owned Python sources were reviewed against the controller/WSL Python
3.12 baseline. The fixes below are local source commits with hardware-free
regression coverage. They have not been pushed, built into a new Yocto image,
deployed to the controller, or used to program an MCU. This review does not
claim compatibility with every historical Python 3 release or complete hardware
qualification.

## Inventory and exclusions

The inventory contains 190 Python/shebang files: 117 target/runtime, 25 host
utilities, 38 tests and 10 explicitly historical files (`old`/`attic`). All 190
pass compilation under Python 3.12 with `SyntaxWarning` treated as an error.
Compilation checks syntax; it does not establish that every imported dependency
or runtime/hardware path works.

Roots reviewed: active SoloLink, SoloLink Python, ShotManager, STM32Loader,
`/home/jresp/Active_GH/artoo-firmware`, integration scripts, workspace scripts,
and owned meta-3dr/meta-solo/meta-solo-bsp layer content. The scan includes
extensionless Python shebang scripts. Layer recipes were checked for packaging
and dependency alignment; a complete BitBake rebuild was not run.

Excluded: archived/reference trees, the recovered duplicate Artoo checkout,
vendor Yocto source, generated build output, caches and virtual environments.
Recipe-fetched DroneKit/DroneKit-Solo/pymavlink have pinned sources but no separate
active owned checkout here; they were reviewed as source/dependency inputs and
were not comprehensively runtime-tested. Existing full legacy suites and GUI
utilities with optional dependencies were not qualified by this pass.

The per-file category/import inventory, compile receipts and local source-mirror
handoff are in `/home/jresp/solo/.integration/python3-audit-20261028`.

## Corrected controller update behavior

- Updater subprocess output is decoded before text processing, including failed
  subprocess output and invalid UTF-8. Imports no longer execute the updater.
- The ROM loader rejects short reads and failed firmware verification. A mismatch
  now produces a failing process rather than printed failure with successful exit.
  Numeric CLI arguments use integer parsing, and required/nonempty input files
  and lengths are checked before opening the UART or resetting the MCU.
- The boot updater makes at most four ordinary page-preserving attempts, then
  fails. It never automatically escalates a generic failure to readout-unprotect
  or mass erase. `FORCE_UPDATE` changes candidate/version selection only; it does
  not authorize mass erase. Destructive recovery needs a separate reviewed
  procedure and is not provided by the normal updater.
- Before programming, the updater validates all six stick calibration triples
  and durably saves a private JSON receipt in `/log/updates`. Missing, erased,
  malformed or unordered calibration prevents programming. The candidate must
  fit below the reserved parameter page. Normal page erasure preserves the
  parameter page, including other MCU settings.
- After successful programming, calibration must read back correctly. A changed
  or unavailable calibration response triggers bounded restoration attempts;
  restoration/readback failure remains an update failure. The running firmware
  version must exactly match the candidate version before archiving the candidate.
- Disabled, skipped or failed candidates stay available. Only a verified update
  moves a candidate to `loaded`. Failures return nonzero, record `UPDATEFAILED`
  and request the failure display even if an older `READY` marker exists.
- Serial handles and the updater-owned temporary STM32 bridge child are cleaned
  up. Packed calibration writes use bytes, and malformed short SysInfo packets
  are ignored instead of indexed past their end.

These changes deliberately tighten the updater's acceptance conditions. Factory
or damaged boards without readable, valid calibration now stop for separate
recovery instead of risking calibration loss. They do not prove the loader GPIO,
ROM bootloader or new firmware on physical hardware. Updates remain disabled on
the running controller, whose existing firmware is `OpenSolo-4dev`.

## Other corrected Python 3 boundaries

| Area | Corrected behavior |
| --- | --- |
| Wi-Fi `iw` helper | Accepts current decimal frequencies (`2462.0`) as well as older integer output, so channel lookup can return channel 11. This does not by itself resolve the live beacon-loss problem. |
| Pairing helpers | Decode subprocess bytes, pass DHCP PIDs as strings, read Linux input events in binary mode, encode/decode wire version strings and use Python 3 byte indexing. |
| WPS utility | Packs/unpacks flags/MAC/name in the documented fixed-width binary field order; this helper is not currently installed by the image recipe. |
| STM32 input report utility | Accumulates binary socket fragments and recognizes binary EOF. |
| Gimbal firmware helper | Uses integer word counts for checksum conversion. |
| ShotManager cable controller | Searches forward/reverse/stationary curvature segments using Python 3 ranges and `zip_longest`. |
| ShotManager geofence manager | Materializes reusable/indexable polygon collections instead of lazy `map` iterators. |
| Developer button client | Supplies the existing sender's enabled-mask argument. |
| ARP utility | Uses raw regex literals without invalid string escapes. |

## Validation and handoff

34 focused tests passed with mocked hardware/network dependencies:

- SoloLink updater: 17 tests, including subprocess success/failure bytes,
  calibration validation/backup/restoration, retry exhaustion, skipped candidates,
  post-flash version mismatch and failure display/marker handling.
- STM32 ROM loader: 4 tests, including short reads, verification mismatch,
  preflight rejection and cleanup after failures.
- SoloLink network/protocol helpers: 10 tests using documented packet offsets,
  fragmented/coalesced streams, binary input events and current `iw` output.
- ShotManager collections: 3 tests covering all curvature search directions and
  reusable/closed polygon geometry.

Commands:

```bash
python3 -m unittest discover -s /home/jresp/solo/apps/sololink/tests -v
python3 -m unittest discover -s /home/jresp/solo/apps/stm32loader/tests -v
python3 -m unittest discover -s /home/jresp/solo/apps/shotmanager/tests -p test_python3_collections.py -v
```

SoloLink and STM32Loader recipes declare their required Python runtime packages,
including JSON/tempfile/PySerial for the updater. The integration lock pins the
new owned commits, and the native Yocto Git download mirrors contain those exact
commits on the required `PORT` refs. Both controller and drone configuration
checks pass. The commits are still local: another machine needs the committed
sources or an explicitly approved push/mirror transfer. The existing deployed
images and their earlier acceptance receipts still identify the old source pins.
