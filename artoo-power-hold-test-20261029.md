# Artoo power-hold reset and ROM tests — October 29

**Temporarily restoring legacy GPIO19/21 output-high behavior prevented the i.MX restart.** The same guarded application reset passed, followed by ROM sync, identity, geometry, repeated bounded reads and return to the old application. Linux retained boot ID `ac525ce9-07bc-4cd4-80f8-fa48ac7ddc26` throughout both tests. No MCU programming occurred.

This establishes a practical cause of the observed reset blocker: the missing legacy controller power-hold assertions matter on this board. Both were asserted together; their individual electrical roles and the rail waveform were not isolated. No negative reset was repeated without the holds, since three earlier unheld application/ROM attempts already restarted Linux. This passes the temporary diagnostic arrangement, not a corrected production updater or permanent BSP implementation.

## Baseline and guarded test

The user authorized investigation/testing and powered on the controller. Its be98 image plus existing Chrony correction was unchanged. Read-only O_RDONLY/ACCESS_READ register access showed GPIO1 GDIR `0x20000004`: GPIO19 and21 were inputs with zero data/sample values, mux5 and pad control `0x1b0b0`. GPIO45/46 were also unclaimed inputs. Twenty172-byte parameter replies matched factory SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454`.

The temporary wrapper claims only unexported19/21 and uses sysfs direction=high to set data before output direction. It verifies high data latches and output directions before calling the existing guarded actor. Hold GDIR was `0x20280004`, asserting bits19 and21. It restores both hold pins to their original input direction and unexports them after the inner actor cleans up. There are no startup hooks or automatic retries.

The first attempt rejected PSR readback before any reset. That guard was too strict: mux5 has SION off, so PSR is not valid proof of output voltage. [NXP AN5078, section8.2.1](https://www.nxp.com/docs/en/application-note/AN5078.pdf) explains that enabling SION is required to read the real pad level while driving an output. The source was corrected and refrozen, preserving the rejected attempt. No SION/pad-register write was introduced; the successful test verifies commanded output state rather than claiming measured voltage.

| Test | Result |
|---|---|
| Application reset with both holds | Requested100ms; measured software interval100.310ms; Linux boot unchanged |
| Application recovery | One initial UDP timeout, then20 matching factory-record replies |
| ROM request with both holds | Requested1ms reset; software interval1.308ms; sync response79 |
| Identity | ROM version22hex; chip ID414hex;256KiB flash; expected UID matched |
| Bounded readback | Initial vectors and factory parameters repeatedly matched across five iterations |
| Return to application | BOOT0low/reset, original UART settings restored, existing bridge resumed; Linux boot unchanged |
| Final recovery check | Twenty additional factory-record replies; GPIO19/21/45/46 all inputs and unexported |

These software intervals do not measure pin pulse width. No new UART device open occurred: the ROM actor paused every thread of bridge PID523, duplicated its existing UART descriptor and temporarily selected1152008E1. Its command whitelist allows only00/01/02/11 and four fixed read ranges. No erase, write, protection change or Go command was used. The main application reset leaves the bridge/8N1 settings untouched.

UART frame/parity counts increased during reset/parity transitions to fe133/pe23. They stayed at those values in the settled capture while RX advanced from216103 to241640. The bridge logged resynchronization and resumed normal reports. This is not a zero-error transition claim or a long soak test.

## Current state and next implementation

All temporary control pins are released. The original updater wrapper remains byte-identical, SHA256 `7fcf04e592e3fba1db01fb9c83e6176e1a428b2bf344cdac77f2b7a710187ad3`; installed loader hash is unchanged. No automatic firmware binary or update-enable marker was installed. Old MCU firmware and calibration remain in place. Inert diagnostic files/logs are retained; no retry/rearm is scheduled.

The controller again has the uncorrected startup power-pin state after cleanup. **Do not run the normal uploader expecting this temporary test to have permanently fixed it.** The next local implementation should restore controller-specific early power-hold initialization in U-Boot, review Linux handoff and normal shutdown, and ensure the updater safely retains power through ROM entry and release. Preserve drone behavior. Freeze and qualify the combined source before any build or later hardware installation; flashing remains disabled for this investigation.

Integration source commit `f72152419593d9681ba34b6bcf67ce626685848d` adds the guarded wrapper and five cleanup tests. Four existing pulse tests and three ROM descriptor/command-guard tests passed at the recorded source freezes. Host tests do not establish hardware acceptance; the physical results above have their own evidence.

[Result](/home/jresp/solo/.integration/artoo-power-hold-test-20261029T224520Z/result.json), [baseline registers](/home/jresp/solo/.integration/artoo-power-hold-test-20261029T224520Z/baseline-registers.json), [application reset](/home/jresp/solo/.integration/artoo-power-hold-test-20261029T224520Z/application-reset-result.json), [ROM result](/home/jresp/solo/.integration/artoo-power-hold-test-20261029T224520Z/rom-result.json), [final registers](/home/jresp/solo/.integration/artoo-power-hold-test-20261029T224520Z/post-ROM-registers.json) and adjacent execution gates/source snapshots preserve the session. The [legacy comparison](artoo-legacy-power-comparison-20261029.md) supplies the historical source lineage. All source/docs changes remain local; Yocto6 and license work remain deferred.
