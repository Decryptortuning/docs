# Permanent BSP STM32 reset and ROM-read result — October 30

**Controlled STM32 reset/ROM preflight passed:** On Linux graph `5b00a161…`, the permanent BSP power holds kept the i.MX on the same boot through ROM entry and application release. Full 256 KiB flash readback, repeated reserved-page/vector reads, exact original firmware identity and 20 post-release parameter replies pass. Erase/write/Go/protection commands were blocked. MCU `4803457` remains installed; selected `629c14b` remains local and unflashed. [Result](artoo-permanent-rom-preflight-20261030.md).

## Authorized scope and source gate

The user explicitly authorized the controlled reset and ROM read while keeping erase/write blocked and flashing disabled. One detached, single-use actor ran on the controller; no retries, Linux reboot, firmware staging, update-enable marker or startup hook was used. Inert tools and private evidence remain under `/log/updates/artoo-permanent-rom-20261030T181322Z` and the corresponding `artoo-permanent-rom-tools-20261030T181322Z` directory. They do not run at boot.

Integration `36a97f7011a7e8e9d18747b7c0a34d7a3139c540` pins the actor and focused tests. [Source gate](/home/jresp/solo/.integration/artoo-permanent-rom-20261030T181322Z/gate.json) records all repository heads, actor/helper/test hashes and the actual producer/consumer review. Thirteen focused host checks passed before execution: real-loader command blocking/read framing, approved ranges, bounded SysInfo parsing, failure-tolerant cleanup, descriptor transport and finally-deassert pulse behavior. Linux/MCU build inputs are unchanged from prior qualification; no rebuild was needed.

The actor uses the existing bridge's UART descriptor through pidfd duplication, pauses/resumes the same bridge, and temporarily sets 115200 8E1 for ROM. It never opens the tty anew. Only BOOT0 GPIO45 and active-high reset GPIO46 are controlled. The kernel retains ownership of GPIO19/21 throughout. Reset pulses contain no logging, file opens or fsync; observed software intervals were 0.100310s entering ROM and 0.100276s returning to the application. These are software timings, not oscilloscope measurements.

## Physical result

- Linux boot ID `aff41622-3d2d-41dc-b90f-69006b642a27` is unchanged before, during and after the ROM cycle. Permanent power driver remains bound with both holds high.
- ROM sync ACK, version `0x22`, chip ID `0x414`, 256 KiB geometry and expected UID all match. Read protection is inactive and option mode selects software IWDG; option bytes were only read.
- All 262,144 flash bytes were read. Full read SHA256: `49eaec5d520b1aeeb73bc346b09d73de525e0493409b3cb1a65c0934b6555f76`.
- Existing 190,976-byte application SHA256: `b4909e99ef10f8ef723b8e3b3fb39edb5eb0be58234704c88f84a8d5b3b1a2da`, matching the prior installed artifact.
- Both reserved pages (4,096 bytes) SHA256: `8266797a96cddee896f830b6aff0a91cf86c16aaa512a276ad6b0875794d82d0`. The compatible loader classifies their legacy record as the same 172-byte application parameters. Three additional reads of both pages and the first 256 bytes matched.
- Parameter SHA256 before/ROM/after: `31fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454`. All 20 post-release runtime replies matched.
- Exact restored identity: HW3, original UID, `5.0.0-dev+g48034572145e-bench-noidle`.
- UART owner remains bridge PID587; original termios restored, controls45/46 returned to input and unexported. No cleanup errors.
- Actual command counters: one GET (`00`), one GET-ID (`02`), 1,078 READ-MEMORY (`11`). All other command bytes are rejected before transport. No erase, programming, Go or protection change occurred.

Private captured bytes were copied back to WSL and independently hashed. [Qualification receipt](/home/jresp/solo/.integration/artoo-permanent-rom-20261030T181322Z/qualification.json), [actor result](/home/jresp/solo/.integration/artoo-permanent-rom-20261030T181322Z/live/result.json), source gate, host test output and settled logs are preserved together. Automatic update markers and active `/firmware/artoo_*.bin` are absent. An older quarantined file in `/firmware/candidates` is not selected or enabled.

## Observations and limits

UART counters rose across the mode/reset transitions to 76 framing and 22 parity errors, then stayed unchanged in subsequent normal-traffic samples. A transition cause is plausible but not electrically proven. The bridge emitted one no-data warning after its deliberate pause, then resumed normal approximately50Hz traffic.

One ath9k Baseband Watchdog reset was counted since the earlier zero-count boot sample. It stayed at one on follow-up; SSH remained available and Linux did not restart. This is a separate Wi-Fi follow-up observation, not a failed ROM read and not evidence of a fully clean network soak.

This completes the controlled permanent-hold reset/ROM/release gate. Next is a separately authorized observed update using the compatible loader, preserving both reserved pages and verifying full application readback plus startup identity. The selected firmware remains the `629c14b` DVT bench image with experimental watchdog disabled. No flashing is authorized by this report. Journal migration/power-loss behavior, watchdog hardware timing, final power-off, pairing/flight and longer soak remain separate acceptance work. Battery/controller changes, Yocto6 and license work stay deferred. No pushes.
