# i.MX ↔ Artoo serial preflight — October 28, 2026

Status: loader source fixes and ten offline regressions passed. Live application
traffic and exclusive bridge ownership were observed, but the controller shut
down before the new ROM preflight could run. ROM identity/geometry, repeated
readback and return-to-application checks below are **pending**. No STM32 flash
operation has been performed or authorized by this preflight tool.

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
These source fixes are not yet installed on the controller or rebuilt into an
image. They must be installed and hash-checked before the live preflight.

## Prepared non-flashing live test

[Controller-side helper](/home/jresp/Active_GH/OPenSolo_2/scripts/artoo-serial-preflight.py)
requires an explicit `--probe-rom` flag and a new evidence directory. Run it
detached so losing the SSH connection does not terminate cleanup. It:

1. Requires standby runlevel 3 and the normal bridge as the sole UART owner.
2. Reads the 172-byte parameter response twenty times and checks equality.
3. Switches to runlevel 2 and checks that the UART has no remaining owner.
4. Repeats three ROM-entry/read/release cycles. ROM command dispatch is restricted
   to Get, GetVersion, GetID and ReadMemory; programming, erasing, Go and
   protection changes cannot be dispatched by the helper.
5. Checks family ID `0x414`, 256 KiB geometry, stable MCU UID, plausible existing
   application vectors, repeated application-byte reads and page-127 parameters.
   Unexpected geometry aborts for review rather than guessing an erase boundary.
6. After every release, confirms application SysInfo at 8N1, records its board
   ID/version, and compares the application UID with the ROM UID.
7. Restores runlevel 3 and checks the bridge and unchanged runtime parameters.

The helper stores a durable result receipt after each cycle and attempts service
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
