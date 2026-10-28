# i.MX ↔ Artoo serial preflight — October 28, 2026

Status: **not ready to approve flashing**. The corrected loader is installed on
Linux and all ten offline regressions passed. A live 100-request parameter test
passed with no timeouts, identical 172-byte responses and a maximum 25.063 ms
round trip. Its SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454`
matches the saved factory-calibration record. Artoo remains `OpenSolo-4dev`.

The first ROM preflight launch was followed by a Linux restart. Boot ID changed
from `fd5b3819-0085-40fa-97c7-f85bf3531703` to
`3c4f00b3-d4b8-474f-828c-4ae08cfca8d8`; neither the job result nor its new log
survived. Therefore no ROM identity/readback result can be claimed. The bridge
and Wi-Fi/SSH subsequently returned, and calibration still matches. No flash
commands are available through the helper's command whitelist.

Artoo's existing power initialization configures PC14 and briefly disables
IMX6 power before starting normally. This is a candidate explanation, not a
proven reset cause. The original helper's assumption that Linux survives all
MCU resets is now unqualified. Do not rerun the old helper or proceed with the
historical manual flash job. Clarification about physical power-button activity
is pending; the exact reset phase must be isolated first.

Integration `05c5636` prepares a revised **single-cycle** helper with durable
boot-ID/phase receipts before runlevel changes, ROM entry, identity, readbacks,
release and bridge restoration. It fsyncs the result and directory metadata
before reset-sensitive steps. It passed syntax validation, but has not been
installed or run. Evidence is in
`/home/jresp/solo/.integration/artoo-serial-preflight-20261028/`.

## Observed connection

- i.MX UART2, `/dev/ttymxc1`, MMIO `0x021e8000`, Linux IRQ 276.
- Device tree uses SD4_DAT4 RX and SD4_DAT7 TX with UART2 enabled.
- Application transport is SLIP at 115200, 8N1, without RTS/CTS or software flow
  control. ROM transport changes parity to 8E1 at the same baud.
- The `stm32` process owns the port in standby runlevel 3. SysV inittab respawns
  it in runlevels 3/4/5; killing it directly would not provide exclusive access.
- Runlevel 2 removes that bridge while retaining SSH. Restore the original
  standby level after every attempt. Never run a serial reader beside the bridge.
- Loader BOOT control is GPIO 45 (GPIO2_IO13); its reset-control pulse uses GPIO
  46 (GPIO2_IO14). Confirm gpiochip32 belongs to `20a0000.gpio` before using the
  factory numbering. The control signal's pulse polarity is implemented by the
  existing board-specific loader; do not infer external NRST polarity from it.
- Linux previously identified the running application as `OpenSolo-4dev`.
  The exact MCU/board identity must come from the live preflight, not a nickname
  such as STM32R2 or the selected build profile.

## Loader fixes

Source `/home/jresp/solo/apps/stm32loader` revision
`a8db3edfb6f91b366d0d4990bfe921d1595c2718` is pinned in integration `4c26379`.
The previous extended erase path silently replaced a supplied page list with
global mass erase. It now encodes the actual count and 16-bit page numbers with
the XOR checksum specified in [ST AN3155](https://www.st.com/resource/en/application_note/cd00264342-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf).
Both normal and extended paths reject invalid page lists. Hardware-independent
tests explicitly check pages 0–126 and exclusion of page 127.

Serial opens now use bounded write timeouts and pyserial exclusive mode. This
does not replace stopping the normal bridge: its C++ serial open does not use
the same advisory lock. CLI cleanup closes the descriptor even if releasing
reset raises an error. Get/GetID replies reject short reads. Existing full
firmware verify mismatch/short-read failures remain fatal. No automatic unprotect
or new mass-erase fallback was introduced.

Ten offline tests passed, including erase frames, invalid-page rejection,
timeout restoration after NACK, short identity reads, failed-reset cleanup and
firmware verification failures. Source review and logs:
`/home/jresp/solo/.integration/artoo-serial-preflight-20261028/`.
The loader fixes are installed and hash-checked on the controller as an overlay
(SHA256 `d3a094529e9d250d919e2988e360c12febdb06a6311cb2d3bfcf99504337ddf4`).
They are pinned for future image builds; no new full image build was needed for
this standalone Python tool. The revised diagnostic helper itself is still local.

## Prepared non-flashing live test

[Controller-side helper](/home/jresp/Active_GH/OPenSolo_2/scripts/artoo-serial-preflight.py)
requires an explicit `--probe-rom` flag and a new evidence directory. Run it
detached so losing the SSH connection does not terminate cleanup. It:

1. Requires standby runlevel 3 and the normal bridge as the sole UART owner.
2. Reads the 172-byte parameter response twenty times and checks equality.
3. Switches to runlevel 2 and checks that the UART has no remaining owner.
4. Qualifies one ROM-entry/read/release cycle before any repetition. ROM command dispatch is restricted
   to Get, GetVersion, GetID and ReadMemory; programming, erasing, Go and
   protection changes cannot be dispatched by the helper.
5. Checks family ID `0x414`, 256 KiB geometry, stable MCU UID, plausible existing
   application vectors, repeated application-byte reads and page-127 parameters.
   Unexpected geometry aborts for review rather than guessing an erase boundary.
6. After every release, confirms application SysInfo at 8N1, records its board
   ID/version, and compares the application UID with the ROM UID.
7. Restores runlevel 3 and checks the bridge and unchanged runtime parameters.

The revised helper stores durable phase receipts before reset-sensitive steps and attempts service
restoration on errors. ROM entry resets the MCU/LCD briefly even though no flash
bytes are changed. It does not use the updater UI to suppress idle shutdown.

## Flashing gate after preflight

Passing read-only tests establishes UART/GPIO/readback readiness, not a guarantee
of a successful erase/program operation or power stability. Actual programming
must use the selected saved manifest's exact embedded version, board/profile and
BIN hash, preserve the calibration page, stream a detached job log, verify all
programmed bytes, and check the running version and unchanged calibration.
The older `/firmware/manual/manual-update.py` package is bound to the historical
`89fc1d4` artifact; do not substitute a bench manifest into it. The newer bench
version has an explicit suffix that must be selected from its saved manifest.

Cold-start Wi-Fi and longer-duration stability remain separate acceptance items.
