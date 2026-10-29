# Automatic-update boot-time Artoo preflight — October 29

**Later live test:** [temporary legacy power-hold test](artoo-power-hold-test-20261029.md) passes application reset and ROM identity/geometry/repeated readback/release with both GPIO19/21 driven high. Linux stays up; calibration matches and pins are released afterward. Permanent BSP/updater correction remains pending; no flashing occurred.

The explicitly authorized one-shot probe ran at the real S60updateArtoo stage. Version lookup and calibration reads passed; the installed loader opened the UART. Linux then restarted during initChip before a recorded ROM sync ACK. The recovery boot skipped the consumed probe. **No MCU erase, write, protection change or flashing occurred.** The original updater hook has been restored and no retry is scheduled.

This answers the boot-context question: the early-boot test did not avoid the reset seen in the running-system tests. It does not establish the electrical cause, prove actual ROM entry, or qualify automatic programming.

## Actual path exercised

The existing `/etc/rcS.d/S60updateArtoo.sh` symlink stayed unchanged. Its `/etc/init.d/updateArtoo` wrapper was temporarily replaced with a dedicated preflight dispatcher. The ordinary updater's run_update/updateStm32 functions were never invoked. Instead the probe used the installed updater's version query, the same temporary child-bridge calibration pattern and actual installed loader open/initChip implementation with its original nominal100ms reset sequence. No firmware file was an input.

The recorded parent is `/bin/sh /etc/init.d/rc S`. Actual RUNLEVEL and runlevel are bothS, and PREVLEVEL/previous areN. There was no normal runlevel3 bridge at entry. The probe created and terminated only its own calibration bridge, checked the full172-byte factory record and validated its six-axis calibration, then required UART ownership to be released before opening the loader. It added no storage sync or logging inside the asserted-reset interval.

Programming was blocked at both the exposed method and ROM-command layers: erase, memory write, protect/unprotect and Go raise before transport; only Get/GetVersion/GetID/ReadMemory are allowed. Memory reads are restricted to device geometry, UID, existing vectors and the parameter record. No firmware discovery, semver selection, update-enable flag or ordinary updater retry loop was exercised. This is a boot-stage hardware preflight, not a successful automatic firmware-update transaction.

## One-shot guard

An atomic persistent attempt directory and a claim file were committed before any UART or GPIO action. An existing or partial claim always skips the action, including after exceptions or a failed claim sync. While the temporary hook was installed it never fell back to the ordinary updater on later boots.

Five host tests passed: claim-before-action/second-run skip, no retry after action failure, no action/retry after claim-sync failure, programming/protection/range rejection, and refusal to acquire/clean up a foreign GPIO. Source was frozen at integrationc2f146b before these tests or target deployment. These host tests are separate from the actual recovery-boot evidence below.

## Observed result

| Stage | Identity/result |
|---|---|
| Before requested Linux reboot | `97c6c675-e557-4c47-a457-7f55ae3e6bc9` |
| Boot that executed the probe | `d042f472-9768-4a30-8278-d86ac1e24a9f` |
| Last durable phase | `automatic-loader-init-pending` |
| Version identified before reset | `OpenSolo-4dev` |
| Calibration read before reset | SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454` |
| Recovery boot | `2df1cdae-f842-47f4-a9da-30b6b71fc775` |
| Recovery boot log | `Artoo boot preflight: consumed, skipping; normal updater NOT invoked` |

There was one MCU preflight attempt. The two Linux boot transitions consist of the requested reboot into the test and the unexpected restart during that attempt; the recovery boot did not repeat it. No ROM ACK, device ID or flash-geometry result was recorded.

## Restoration and recovery checks

The wrapper was atomically restored to its exact original root-owned0755 bytes, SHA256 `7fcf04e592e3fba1db01fb9c83e6176e1a428b2bf344cdac77f2b7a710187ad3`. Its existing S60 symlink is unchanged. The consumed claim, diagnostic source and evidence remain preserved; they have no active boot hook. Restoration wrote the original wrapper bytes through the overlay and did not delete overlay data.

The recovered controller has one normal stm32 bridge (PID523), both control GPIOs unexported, no automatic firmware candidates and no ENABLE_UPDATE/FORCE_UPDATE/UPDATE markers. Twenty fresh172-byte reads match the preflight factory record exactly, and startup identifies the unchanged OpenSolo-4dev firmware. The final short capture is uptime122.09seconds in the recovery boot, with Chrony synchronized and the prior startup correction intact. No filesystem/MMC error was reported by the scoped kernel check. No additional reboot was used merely to test restoration.

The running Linux identity remains be98 plus the documented Chrony init correction and a byte-identical restored updater wrapper in the overlay. The complete faf image and newer Artoo firmware were not installed. Do not enable unattended firmware programming while reset initialization still takes down its Linux host. The next useful evidence remains J2 console/reset-cause output and actual BOOT0/NRST/i.MX reset-power behavior; changing the supply is not an established remedy.

[Result](/home/jresp/solo/.integration/artoo-boot-preflight-20261029T220600Z/result.json), [source/producer-consumer gate](/home/jresp/solo/.integration/artoo-boot-preflight-20261029T220600Z/execution-gate.json), [recovery and skip evidence](/home/jresp/solo/.integration/artoo-boot-preflight-20261029T220600Z/recovery-result.json), [restoration](/home/jresp/solo/.integration/artoo-boot-preflight-20261029T220600Z/restored.json) and [self-contained diagnostic source](/home/jresp/solo/.integration/artoo-boot-preflight-20261029T220600Z/diagnostic-source) preserve exact identities. Work and commits remain local; Yocto6 and license work remain deferred.
