# Artoo journal firmware flash preparation — October 30

Preparation is complete locally; live readiness is not established. No device files, GPIOs, services, boot markers or firmware were changed. The user asked to prepare, not to start programming. Battery/controller implementation work is [saved and deferred](artoo-battery-controller-followups-20261030.md).

## Selected artifacts

- MCU source: `629c14b71ce6748272b9bd711d12e875b6c3cc9f`.
- DVT/HW3 bench version: `5.0.0-dev+g629c14b71ce6-bench-noidle`.
- Binary:192,636 bytes; SHA256 `f974f6e65b4a662ed5ec766a9f3acf1263e47cf66498d2a202dec0ccb97d8518`.
- Factory commands off; experimental IWDG off; idle-only timeout off, retaining the prior bench preference for the disconnected paddle.
- Journal-v1 marker verified at offset304; image ends below page126. Pages126/127 must both be preserved.
- Controller development Linux graph: `5b00a1619b3bc97ffbd98b978583bbf17d9024cf53c2919fe7e62ed6f68bad10`.
- Preserved SD image:201,327,616 bytes; SHA256 `67744c3d8def65b7819f4c20dc53057ed49ff8a3cf28529f06f34d52bb626b16`.
- All eight preserved Linux deployment files, selected MCU BIN/ELF hashes, profile flags, vector bounds and marker agree with existing qualification. No rebuild or new emulator run was needed.

[Candidate/check receipt](/home/jresp/solo/.integration/artoo-flash-preparation-20261030T173912Z/candidate-check.json) records exact paths, hashes, source gate and future operation order. The firmware copy in that local receipt is inert; nothing was copied to the controller's `/firmware` directory.

## Required order

1. Install the matching combined Linux image, including raw U-Boot, kernel, DT and userspace. The recorded physical be98 plus Chrony overlay lacks permanent power holds and the journal-aware tools. A Python-only copy is insufficient. Use the existing offline card procedure after detecting the actual device and receiving exact-target storage authorization; historical Disk4 or `/dev/sdf` names are not current authority. No SD backup is planned, following the user's existing preference.
2. Boot with automatic Artoo update disabled. Verify VERSION graph, Wi-Fi/SSH, effective loader/updater hashes and `/sys/bus/platform/devices/controller-power/driver` resolving to `opensolo-controller-power`. Existing overlays can mask new image files: inspect effective bytes, do not indiscriminately delete overlays. The full new image already contains the Chrony fix.
3. Verify running MCU UID/HW3/version, calibration and full172-byte parameters through the sole normal bridge. Capture parameter evidence privately. Under separate reset/service-operation authorization, complete a bounded ROM ID/geometry/option-mode/read/release check with erase/write blocked, confirming the i.MX boot ID survives.
4. Perform one explicitly authorized, observed manual update using the new guarded loader/updater and exclusive UART ownership. The current and selected versions both parse as5.0.0, so ordinary semver upgrade selection skips the new development build. Any needed force selection must be scoped to that authorized invocation; do not create persistent FORCE_UPDATE/ENABLE_UPDATE markers or schedule replay. Preserve the full4,096 reserved bytes durably before erasing pages0–125. Never use the retired temporary/GDB actors or old127-page loader command.
5. Require full application readback, identical reserved pages before application release, exact running version and all172 parameter bytes after startup. Restore the normal bridge, confirm stable Linux boot identity and diagnostics. No immediate retry/unprotect or automatic calibration repair on failure. Journal migration occurs on first parameter save, not simply on boot; avoid gratuitous calibration writes as a smoke test.

The previous firmware is not a drop-in rollback once journals are written. No journal-to-legacy downgrade converter is supplied. Keep prior recovery evidence and the new reserved-page backups; never erase journal pages to bypass a mismatch. Firmware parameter backups are distinct from an SD-card backup.

The new battery/controller findings remain open. This selects the existing qualified bench candidate and does not establish charging/flight acceptance or enable the experimental watchdog. Yocto6/license work and publication remain deferred.

## Connection observation

The normal strict-key SSH alias did not resolve, OpenWrt showed no active controller DHCP lease (only a stale192.168.2.207 neighbor), and direct access to the prior address timed out. The user's current power/card status is pending; this is not evidence of a firmware failure. No physical SD device appeared in WSL's block-device listing. No reset/ROM check or current battery/power-source determination was possible.
