# Artoo serial link and flashing readiness — October 29

The running i.MX–STM32 application link passes the scoped check. **Flashing is not yet cleared:** the reset/ROM-entry path previously restarted Linux and has not completed a successful ROM identification/readback cycle. No MCU reset, GPIO change, bootloader entry, bridge stop, target file installation, erase, write, unprotect or flash was performed in this session.

The controller runs Linux graph be98 plus the documented Chrony init overlay correction. The observed boot is `4ca4253b-8ec9-4c53-8ea7-b26bca12dbb7`; running MCU firmware remains OpenSolo-4dev and reports hardware ID 3. Battery-only operation was confirmed by the user. A different supply has not been shown to fix the historical failure and is not an established prerequisite for this check.

## Live application-path evidence

| Check | Result |
|---|---|
| Internal serial device | `/dev/ttymxc1`, i.MX UART2, connected to STM32 USART1 per DVT source |
| Ownership | Sole owner `stm32`, PID 523; UART0 getty remains separate |
| Actual settings | 115200, 8 data bits, no parity, one stop bit, no hardware/software flow control, raw local flags |
| How settings were read | TCGETS on a pidfd_getfd duplicate of the existing bridge descriptor; no second device open or settings change |
| Readback test | 100 empty UDP5011 stored-parameter requests through the existing bridge; 100 identical 172-byte replies, zero timeouts |
| Round-trip time | Median 19.93 ms, maximum 27.62 ms in this short sample |
| UART1 counters | TX 1615→1923; RX 591657→612843, with no reported framing/parity/overrun entries |
| Record identity | SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454`, identical to the previous factory record |
| Calibration | Actual installed-matching updater validator accepts all six axes, including the linear dial's valid zero trim |

The 172-byte record is preserved privately in the receipt directory. This is a calibration snapshot, not an SD-card backup. The installed uploader and updater exactly match their owned source hashes. Automatic `/firmware/artoo_*.bin` candidates and ENABLE_UPDATE/FORCE_UPDATE/UPDATE markers are absent. The older staged `/firmware/candidates` payload is separate from the latest candidate and cannot be treated as its deployment authority.

## Candidate identity

The primary firmware checkout remains clean at `48034572145e9e62dff881141277564e1ed1d3a6`. Both preserved DVT artifacts pass fresh byte/hash/version/vector/page-boundary checks and match the exact ELF/BIN hashes of their successful prior Renode runs. Those native/emulator results were verified by provenance, not rerun or relabeled as physical tests.

| Profile | Embedded version | BIN bytes | SHA256 |
|---|---|---:|---|
| Production DVT | `5.0.0-dev+g48034572145e` | 190956 | `1d607fc6ca23064bd6823086db1b84fd59c318dd950a481cacd86e32bd78925e` |
| Bench DVT | `5.0.0-dev+g48034572145e-bench-noidle` | 190976 | `b4909e99ef10f8ef723b8e3b3fb39edb5eb0be58234704c88f84a8d5b3b1a2da` |

Both exclude factory commands, contain the OpenSolo 5/Tamp/calibration fixes and leave the reserved parameter page outside the binary and padded programming range. Production retains normal idle shutdown; bench disables only that countdown. Bench DVT is available for development with the disconnected paddle, but no exact image has been selected for flashing by this check. A reported board ID 3 is consistent with DVT, not an independent physical package/PCB identification.

## Reset-path investigation

The live GPIO bank at base 32 is `20a0000.gpio`. The effective pinmux and owned DTS agree on SD4_DAT5/GPIO2_IO13 (global45, loader BOOT0) and SD4_DAT6/GPIO2_IO14 (global46, loader reset). Both are currently unexported and unclaimed as GPIO consumers. These pads also carry UART2 RTS/CTS alternatives; actual flow control is disabled. This verifies the Linux mapping, not external mainboard wiring, inversion or electrical polarity.

The previous run4's durable last phase is `gpio-46-1-pending`, after recording BOOT0 high. Linux restarted without a completed ROM cycle; there is no successful GetID/flash-geometry/calibration-page result. The source drives reset high, then low, and uses UART 115200 8E1 for ROM communication. The canonical flashing guide expects Linux/Wi-Fi to survive ordinary Artoo flashing; the historical restart is an unresolved gate, not acceptance of normal behavior.

DVT firmware controls i.MX power through PC14 and board keep-on through PC15. That provides a plausible reset/power-control interaction to investigate; it does not establish battery brownout or prove a charger remedy. The available HDTC schematic and pinout describe the i.MX board/J3, not a complete STM32-mainboard power circuit.

The existing traced preflight also performs file/directory fsync and global os.sync from inside setPin. When reset is asserted, that can extend the nominal 100 ms pulse with storage latency. A new diagnostic must avoid or characterize that perturbation. It cannot be the sole explanation because an earlier untraced cycle also failed. Do not blindly rerun the historical helper or claim root cause from its final phase alone.

## Remaining gate before any MCU write

1. Plan a controlled **non-flashing reset/ROM diagnostic**, with i.MX console capture if available, preserved before/after boot identity and no added synchronous storage work inside the asserted-reset interval. It may restart Linux and needs a deliberate recovery path; it is outside this session's non-disruptive inspection.
2. Demonstrate ROM ACK, actual device ID/flash geometry/UID and repeated read-only application/parameter-page reads; then restore application mode and verify the original version, bridge and unchanged record. ROM read commands do not authorize erase/write/protection changes.
3. Select an exact manifest version/hash/profile and use a current deployment actor. The historical `hardware-update-20261028/manual-update.py` reconstructs its version from the source revision and does not preserve the latest bench suffix; it is not the current bench deployer.
4. Before a later explicitly authorized flash, deliberately release the normal bridge, scan all UART owners and take the update lock. PySerial exclusive mode alone is advisory and cannot exclude a noncooperating C bridge. Preserve page 127 at `0x0803f800`, limit normal erase to pages 0–126 after geometry verification, read back the complete programmed image and compare the actual running manifest version and parameters. Never escalate to mass erase/readout-unprotect or automatically replay an attempt after a host restart.

[Readiness result](/home/jresp/solo/.integration/artoo-readiness-20261029T212657Z/result.json), [runtime probe](/home/jresp/solo/.integration/artoo-readiness-20261029T212657Z/runtime-probe.json), [candidate verification](/home/jresp/solo/.integration/artoo-readiness-20261029T212657Z/candidate-verification.json) and [future flash plan](/home/jresp/solo/.integration/artoo-readiness-20261029T212657Z/future-flash-plan.json) preserve the exact evidence and planning boundary. All changes remain local. Yocto 6 and license work remain deferred.
