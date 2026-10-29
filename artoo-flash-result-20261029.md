# Artoo OpenSolo 5 bench firmware installed — October 29

The user explicitly authorized flashing with the tested temporary power holds. **One upload installed `5.0.0-dev+g48034572145e-bench-noidle` on the physical controller.** Complete ROM readback matches the190976-byte candidate; the entire2048-byte reserved page127 is unchanged. Settled application identity/calibration and normal bridge communication pass. The user sees the OpenSolo5 startup branding and Waiting for Solo screen.

## Installed identity and programming boundary

| Item | Verified value |
|---|---|
| Source | `48034572145e9e62dff881141277564e1ed1d3a6` |
| Profile | DVT/HW3 bench, factory commands disabled, idle-only shutdown disabled |
| Version | `5.0.0-dev+g48034572145e-bench-noidle` |
| Binary |190976bytes; SHA256 `b4909e99ef10f8ef723b8e3b3fb39edb5eb0be58234704c88f84a8d5b3b1a2da` |
| Actual chip | ID414hex, ROM22hex,256KiB flash; expected UID and HW3 match |
| Erase | One explicit list, pages0–126; no mass erase/unprotect/protection/Go |
| Reserved page | Page127, address0803f800hex; full2048bytes unchanged before/after |
| Factory record |172bytes; SHA256 `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454` |
| Linux boot | `ac525ce9-07bc-4cd4-80f8-fa48ac7ddc26` unchanged through upload/recovery |

The existing exact candidate and prior Renode qualification were reused; no firmware rebuild was performed. Bench was selected for this testing session with the disconnected paddle, preserving the earlier request to avoid idle shutdown. This is not the production-idle profile.

The actor claimed the update lock, required the known controller image and sole bridge, asserted temporary GPIO19/21 holds, and paused every thread of bridge PID523. It duplicated the existing UART descriptor and used the previously successful BOOT0/reset sequence. It preserved the full old256KiB STM32 flash privately before erase, including page127; no SD-card backup was made.

Sequential256-byte writes were restricted to exact candidate bytes and addresses. After programming, the complete256KiB flash was read back: the binary matches, the application tail up to page127 is erased FF, and the complete reserved page matches its saved bytes. Independent workstation byte comparisons confirm those results. No automatic retry occurred.

## Startup anomaly and settled verification

The immediate post-reset SysInfo query rejected an oversized serial packet. The actor recorded the failure, retained the power holds and kept the bridge paused instead of declaring success or retrying programming. The full flash readback had already passed. The specific initial bytes causing the oversized packet were not saved, so their exact mechanism is unresolved.

A later passive capture showed ordinary framed packets, and the user reported the OpenSolo5 splash/Waiting for Solo screen. A separate frozen verification helper drained the stale receive backlog and queried the settled application through the same bridge descriptor. It returned the exact bench version, expected UID/HW3 and unchanged172-byte calibration. **No additional reset, erase or write was used for this follow-up.**

The helper resumed the original bridge and received20 identical factory-record replies through its normal UDP5011 path, then restored GPIO46/45/21/19 to inputs and unexported them. Linux retained the same boot ID. The original updater wrapper and installed loader hashes remain unchanged, and automatic firmware discovery/enable markers are absent. Diagnostic source/log files are inert; there is no armed boot-time retry.

## Current scope and next work

The physical controller now runs the OpenSolo5 bench MCU image; Linux remains be98 plus the existing Chrony overlay correction. This is a successful flash/readback and short settled startup/UART/calibration check. It does not establish pairing, Solo flight, joystick/paddle behavior, every LCD asset interaction, power-cycle/normal-shutdown acceptance or a long soak. Bench idle behavior is selected by the verified compiled profile; no ten-minute physical acceptance test was added here.

The permanent BSP/updater power-hold correction is still pending. The temporary workaround was released after verification, so a future onboard upload must retain the same explicit hold precautions until that correction is implemented. Ordinary automatic updates were not enabled. The initial startup-query anomaly should inform a future bounded resynchronization/settling improvement rather than a blind programming retry.

Integration commit `82cda9e` adds the one-shot guarded actor and seven native guard tests. They cover forbidden/unarmed commands, mass/reserved-page erase rejection, single erase, exact sequential writes, bounded reads, invalid candidates and packet framing. Its source/hash gate precedes host checks and hardware programming. The settled verification has its own frozen source/gate and preserves the initial failed check.

[Final qualification](/home/jresp/solo/.integration/artoo-temp-flash-20261029T225106Z/qualification-result.json), [initial actor result](/home/jresp/solo/.integration/artoo-temp-flash-20261029T225106Z/result.json), [settled runtime verification](/home/jresp/solo/.integration/artoo-temp-flash-20261029T225106Z/post-startup-result.json), [execution gate](/home/jresp/solo/.integration/artoo-temp-flash-20261029T225106Z/execution-gate.json) and adjacent private before/after flash captures preserve the session. Work and documentation are locally committed; no pushes occurred. The owner's build-env.md, historical receipts and Yocto6/license deferrals remain unchanged.
