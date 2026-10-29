# Maintenance A pause and resume checkpoint — October 28, 2026

**Historical checkpoint:** the user resumed after the WSL restart on October 29. Current results and exact final pins are in the [maintenance ledger](scarthgap-maintenance-a-20261028.md). This document retains the earlier paused graph, publication and resume instructions; it does not describe the current candidate.

**Work is paused at the user's request. This is an unqualified source checkpoint.**
The user authorized documentation updates, commits and pushes after the running
build stopped. No new image is ready for flashing. Source implementation builds
on the owner's recovered Decryptortuning/PORT Python upgrades and source9; the
[180-unit package investigation](scarthgap-package-update-investigation-20261028.md)
and [owned-fork investigation](scarthgap-owned-fork-update-investigation-20261028.md)
remain part of the checkpoint. No wholesale PyPI replacement was made.

## Frozen product identity

Current graph: `7f9d289975c05df45843d81e8b5656c33589977005c7a1db50094d8b8c99ac36`.

| Repository | Frozen qualification source |
|---|---|
| Integration OPenSolo_2 | `e8263eef6bf7f02e503aecafa61fc2675d58d342` before this graph-neutral documentation checkpoint |
| meta-3dr | `d0bdc66e388768712cec31f54d683d5af1d5a63d` |
| meta-solo-bsp | `1602ee542d832ba51b7badad15faad6755acb2a7` |
| SoloLink | `cd3ff3c7e311a5498a746b0187c38e21aa660438` |
| Documentation | `389c20207422823e2f7b32c95f614a9543df318a` before this documentation checkpoint; inventory-only, not an image input |
| Poky 5.0.21 | `4fb7016a07507683359ebd620f1951fd5ea44bfa` |
| meta-openembedded Scarthgap | `2db6aece32c7f8ebf6015b468e1b4e21de072e3b` |

The complete lock and wave3 freeze retain the other vendor/application pins.
Integration HEAD is receipt metadata, not a circular self-pin. Documentation and
unhashed README/AGENTS changes do not change the product graph. A future
build-input edit or layer repin changes all four whole-image identities and
requires another source review before execution.

## Completed evidence and deliberate stop

Evidence root: `/home/jresp/solo/.integration/scarthgap-maintenance-a-20261028/`.
Original attempts and repair receipts remain intact.

| Evidence | Result and limit |
|---|---|
| Host tests | 139 aggregate passes: integration 79, SoloLink 35, Wi-Fi 11 and matched secure-transfer 14. These are host results, not ARM acceptance. |
| Input preparation | Local SoloLink seed, input verification and all four profile configurations passed; verify/configure repeated for wave3. |
| Metadata | Controller and drone parse passed with zero errors; controller critical 14 providers, append selection and recipe inventory matched intended versions. |
| Native rsync | rsync3.5.1 native build passed 145 tasks; local transfer tests used its matched upstream rrsync. |
| Syntax | Shell and Python syntax checks passed. |
| Full patch sets | Controller 1793 tasks and drone 1797 tasks passed, including actual DroneKit zero-fuzz QA. |
| Controller-development full build | User-requested graceful stop completed at `2026-10-29T04:57:35.896005Z` (October 28 Phoenix time). 5,502 tasks attempted, 2,640 reused; all attempted tasks succeeded. Exit1/“Stopped build” records the intentional pause. No completed candidate image is claimed. |

Wave1's integration test failure was a fixture that changed its lock without
committing the fixture authority. The fixture was corrected, then all 15 affected
workspace tests passed; the other 64 integration results were retained.
Wave2's full build stopped on DroneKit patch-fuzz QA. Wave3 added missing trailing
context only, without suppressing QA or editing a build workdir. All three patched
Python files were byte-identical to the prior output; actual clean patch QA and
output hashes subsequently passed. Both failed-attempt logs remain evidence.

Key receipts: `wave1/closure.json`, `wave2/root-review.json`,
`wave3/source-freeze.json`, `wave3/root-review.json`,
`wave3/patch-qa-result.json`, `wave3/all-patches-result.json`,
`wave3/controller-development-build-result.json` and its complete build log.
`root-progress.json` records the live coordination checkpoint.

## Remaining qualification when the user resumes

No completed new image, installed-candidate DroneKit/Solo runtime result, ARM
TLS/IPC/NumPy result, executed RTP regression, scoped CVE database report,
repeat-packaging result or complete fresh offline recovery is claimed.

1. Reconfirm the exact frozen lock and the selected active source state and review the pause and
   wave3 handoff. Keep shared downloads/sstate and the existing 32-CPU pressure
   gates. Resume controller-development, then drone-development,
   controller-release and drone-release sequentially; do not run four builds
   concurrently. Completed tasks can be reused.
2. For each finished profile, run the existing artifact, image/initramfs,
   userspace and maintenance validations; verify boot/update identities,
   release/development package differences, permissions and unchanged staged
   firmware payloads. All image identities use the current graph.
3. Run `tests/runtime/test_dronekit_legacy.py` against the actual installed
   candidate libraries, then ARM QEMU. Its unconditional SoloVehicle import
   requires a drone closure. Historical source9 installed DroneKit/PyMAVLink on
   both roles and DroneKitSolo only on drone; inspect the new manifests first.
   Controller-only checks must use its actual installed closure, with source
   identity proof before reusing any corresponding drone result.
4. Author and independently review the small additional ARM userspace checker
   described below before compiling/executing it. This checker is still only a
   read-only design; no source was authored before the pause.
5. Compile/run the reviewed `wave2/rtp-fragment-contract.c` against candidate ARM
   libraries and the candidate RTP plugin. Check H264/H265 cap/reset/recovery;
   separately run preserved source9 plugins as negative controls that must pass
   normal NAL output and fail specifically at the cap. The checker has not run.
6. Capture scoped CVE reports with the paired configure/build flags, triage
   applicability and retained exceptions, repeat packaging from frozen rootfs
   inputs and compare all four profiles. Export complete Git/tag/source transport
   plus firmware/archive blobs, restore independently offline and verify closure.

If a failure requires any product/checker edit, first let active tests/builds
finish, make the scoped repair, repin if needed and record a new source-review
gate. Do not expand into Yocto6, license work, broad W7 vendor/BSP/media migration,
W9 hardware or MCU/flight/gimbal promotion as part of resuming this checkpoint.

The planned ARM checker will use only the new rootfs's Python/modules/native
libraries, with rooted PYTHONHOME/sys.path, 32-bit/little-endian assertions and
module/library provenance. It will generate ephemeral P-256 certificates outside
the image, test TLS 1.2/1.3 with MemoryBIO and explicit trusted/untrusted/wrong-host
cases, and check hash/HMAC/PBKDF2 vectors. It will exercise actual posix_ipc 1.2 and
`sololink.rc_ipc` using unique O_CREX shared-memory/semaphore names and a semaphore
initialized to one as pixrc does; independently validate the packed 26-byte
`<QH8H` RC contract, both mapping directions and locking, then remove only its own
objects. It must never attach `/rc_shm` or `/rc_sem`. NumPy native math/linear
algebra/FFT and safe app imports supplement the existing actual-library suite;
daemon/main imports, sockets, device writes and flight workers are excluded.

OpenSSL CLI/provider behavior must be checked from the new installed closure.
Historical source9 had openssl-bin/conf/legacy provider but no openssl-engines.
Preserved cryptodev/KTLS build settings do not prove an installed engine,
activation or hardware acceleration. The probe must not initialize hardware
engines; physical offload/throughput remains deferred.

## Publication checkpoint and preserved state

The user authorized commits and pushes of this paused checkpoint. All eight
checkpoint repositories were pushed and their exact remote revisions verified.
The feature ref for OPenSolo_2, meta-3dr, meta-solo-bsp, SoloLink and docs is
`codex/scarthgap-maintenance-a`. Required ShotManager and STM32Loader source pins
use existing `PORT` refs with verified fast-forward updates. Artoo's prior primary
`4803457` checkpoint uses `codex/artoo-wsl` separately; it does not replace the
unchanged Yocto-staged `60a2707` firmware authority. This is not a staged firmware
upgrade. No merges, force pushes or default-branch changes were made. The
coordinator retained the complete push/verification receipt as
`publication-after.json` in the evidence root. Publication does not qualify an image.

Published source checkpoints (October 28 Phoenix time):

| Repository | Ref | Verified commit |
|---|---|---|
| [sololink](https://github.com/Decryptortuning/sololink/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `cd3ff3c7e311a5498a746b0187c38e21aa660438` |
| [shotmanager](https://github.com/Decryptortuning/shotmanager/tree/PORT) | `PORT` | `fc9fd5bdfb73ccf5968817f50fbbde054ba0da39` |
| [stm32loader](https://github.com/Decryptortuning/stm32loader/tree/PORT) | `PORT` | `a8db3edfb6f91b366d0d4990bfe921d1595c2718` |
| [artoo-firmware](https://github.com/Decryptortuning/artoo-firmware/tree/codex/artoo-wsl) | `codex/artoo-wsl` | `48034572145e9e62dff881141277564e1ed1d3a6` |
| [meta-3dr](https://github.com/Decryptortuning/meta-3dr/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `d0bdc66e388768712cec31f54d683d5af1d5a63d` |
| [meta-solo-bsp](https://github.com/Decryptortuning/meta-solo-bsp/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `1602ee542d832ba51b7badad15faad6755acb2a7` |
| [docs](https://github.com/Decryptortuning/docs/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `41d98f2e1c7b921af91c7db27dd61b5381ac00a7` |
| [OPenSolo_2](https://github.com/Decryptortuning/OPenSolo_2/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `dda3531d1510ddf4c24255c775b1a9e4f854f907` |

The docs row identifies the initial pause checkpoint; the subsequent publication
receipt update changes documentation only. SoloLink Python was already published
at its locked `975bda067750ca47a43d69d8022382a9c4b384b4` PORT revision.

The immutable source9 graph
`36f841398e89201e2e462391cfa2779188349999090295c8b91c967f29380138`
remains the last offline-qualified baseline: four builds, sixteen image/initramfs/
ARM checks, 82 relevant tests, identical cached-rootfs repeat packaging and 34
independently restored Git stores. It was not a full clean rebuild, complete CVE
assessment, hardware/MCU qualification or a flash of the current controller.
Preserve `/home/jresp/solo/recovery/scarthgap-refactor-20261028/qualified-source9`.
The physical controller still has its older rootfs/overlay and no-MSI kernel.

The user-requested RAM cleanup reclaimed 17.8697 GiB of clean caches without process
termination, swapoff or WSL restart. Existing `autoMemoryReclaim=gradual` moved to
`.wslconfig`'s `[experimental]` section and takes effect at the next normal WSL VM
restart. About 27 GiB was free/available after the build stopped; no further cleanup
was needed. Receipt: `wsl-memory/cleanup-result.json`.

Dirty `build-env.md` remains excluded from staging/commits and byte-identical at
SHA256 `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`.

Inactive `workspace/scripts` remains excluded inventory (`build_input:false`,
`dirty:true`): pre-existing edits to README.md/yocto-build.sh and untracked
requirements-sd-card-flasher-gui.txt, run-sd-card-flasher-gui.sh and
sd_card_flasher_gui.py are preserved. Do not sweep them into this publication or
claim that the entire workspace is clean.
