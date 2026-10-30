# Artoo journal firmware installation — October 30

**Artoo journal firmware installed and verified:** The physical HW3 controller now runs `5.0.0-dev+g629c14b71ce6-bench-noidle` on Linux graph `5b00a161…`. Full flash readback passed; both 2 KiB reserved pages and all 172 parameter bytes are unchanged. The original serial bridge and diagnostic interface work, and Linux stayed on the same boot. Experimental watchdog and idle shutdown remain disabled. [Installation result](artoo-journal-flash-result-20261030.md).

## Scope and selected artifact

The user explicitly authorized flashing the new Artoo firmware, preserving calibration and checking startup. One detached manual update ran using the permanent BSP holds and the installed journal-aware loader's validation, backup and transport primitives. The normal boot updater was not invoked or enabled. No automatic retries, mass erase, protection changes, firmware rollback or parameter writes were performed.

- Firmware source: `629c14b71ce6748272b9bd711d12e875b6c3cc9f`.
- Selected binary: 192,636 bytes, SHA256 `f974f6e65b4a662ed5ec766a9f3acf1263e47cf66498d2a202dec0ccb97d8518`.
- Actual padded write: 192,768 bytes in 753 blocks, below reserved page126.
- Version: `5.0.0-dev+g629c14b71ce6-bench-noidle`; HW3 DVT and expected UID match.
- Bench profile: factory commands off, experimental IWDG off, idle shutdown disabled for the disconnected paddle. Other existing power behavior is not disabled.
- Linux graph: `5b00a1619b3bc97ffbd98b978583bbf17d9024cf53c2919fe7e62ed6f68bad10`.

[Source gate](/home/jresp/solo/.integration/artoo-journal-flash-20261030T181849Z/gate.json) pins the combined candidate, actor/helpers/tests and producer/consumer review. Integration `cf710b8` adds the manual actor and focused guards. Fifteen host checks passed before execution, including installed-loader exact erase lists, sequential payload/padding, forbidden commands, failed-operation replay blocking, recovery retention, descriptor reads and finally-deassert pulses. The already qualified MCU artifact was not rebuilt or changed.

## Flash and preservation evidence

Before mutation, the actor checked ROM identity, geometry, software-watchdog option mode, full old application identity and parameter payload. The installed loader durably backed up both reserved pages; a complete prior flash read and the runtime parameter record are also preserved. This is MCU recovery evidence, not an SD-card backup.

Exactly one erase command selected pages0–125. Both pages126/127 were excluded. The actor programmed only the exact candidate plus final FF padding, then read all256KiB back before releasing the MCU. It checked the complete application, every remaining application byte as FF, and both reserved pages unchanged. Raw before/after files and the durable page backup were retrieved and independently compared in WSL.

| Evidence | SHA256 |
|---|---|
| Full flash before | `49eaec5d520b1aeeb73bc346b09d73de525e0493409b3cb1a65c0934b6555f76` |
| Full flash after | `7646d3d08a26d5516a3a0b0bff686f8e59380e920f4ceaedd3f85fa987ea3b14` |
| Reserved 4,096 bytes, before and after | `8266797a96cddee896f830b6aff0a91cf86c16aaa512a276ad6b0875794d82d0` |
| Full 172-byte parameters, before and after | `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454` |

ROM command totals were GET1, GET-ID1, READ-MEMORY2069, explicit page erase1 and WRITE-MEMORY753. There was no mass erase, Go, protection modification or retry. [Actor result](/home/jresp/solo/.integration/artoo-journal-flash-20261030T181849Z/live/result.json) and [independent comparison](/home/jresp/solo/.integration/artoo-journal-flash-20261030T181849Z/independent-readback.json) record success.

## Startup and settled operation

The verified new image started and returned its exact version, original UID and HW3 identity. Twenty consecutive parameter requests through the restored normal bridge matched all172 original bytes; a later request also matched. Both GPIO45/46 were returned to input and unexported, the original UART mode restored, and the same bridge PID587 resumed. GPIO19/21 stayed owned high by the permanent BSP driver. Linux boot ID `aff41622-3d2d-41dc-b90f-69006b642a27` remained unchanged throughout.

The new UDP5027 diagnostic interface responds correctly. Samples show progressing heartbeats and normal approximately50Hz application traffic, no retained fault, zero UART line errors, overruns, dropped bytes, TX errors or out-of-memory events. Experimental watchdog, started state, quarantine and reset streak are all off/zero. One rejected frame was present in the first diagnostic sample and stayed at one; it was not diagnosed as a continuing fault. Startup maximum loop gap was64.547ms and remained unchanged in the sampled interval; this is not loaded timing acceptance.

Linux UART framing/parity counters increased across mode/reset transitions to158/45, then remained flat while normal traffic continued. The previous ath9k Baseband Watchdog count stayed at one, with SSH available and no Linux restart during programming. The prior Wi-Fi observation remains a separate follow-up.

The manual actor does not call boot-updater bookkeeping. A separately pinned [finalization step](/home/jresp/solo/.integration/artoo-journal-flash-20261030T181849Z/finalize-gate.json) rechecked runtime parameters, diagnostics, bridge, boot and disabled update markers, then atomically refreshed Linux `/STM_VERSION` from480 to the verified629 version. No additional MCU write or reset occurred. The firmware copy remains inside the inert private operation bundle, outside active `/firmware/artoo_*.bin` selection; no persistent enable/force marker or boot hook was created.

## Current state and limits

[Final qualification](/home/jresp/solo/.integration/artoo-journal-flash-20261030T181849Z/qualification.json) records successful programming, preservation and startup. User observation of the LCD is pending; software identity/traffic alone does not prove the visible display. The raw legacy parameters remain byte-for-byte preserved: reading/importing them does not exercise journal migration, which occurs on a later legitimate save. No gratuitous calibration save, forced watchdog fault, interrupted flash operation or destructive recovery test was performed.

New firmware is installed, but journal power-loss behavior, active watchdog timing, final power-off/reboot, loaded pairing/flight and longer soak remain separate acceptance work. Do not use the previous single-page flash actor or assume old firmware can interpret a journal after migration. Battery/controller follow-ups, Yocto6 and license work remain deferred. Source and documentation commits are local only; no pushes. Owner build-env.md remains unchanged.
