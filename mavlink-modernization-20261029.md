# MAVLink library modernization — October 29

Local software qualification is complete for graph `28fd826a52ca4accf0743f3ef3492ae2fe87e608bcb9adc8f4509abc635db244`. Four controller/drone development/release images build and pass sixteen baseline checks, four expanded ARM runtime suites and four compiled ARM C wire suites. No hardware changes or publication occurred. The physical controller still runs the previously qualified7055 graph and Artoo OpenSolo-4dev.

The owner's prior Python3 port remains the foundation. This pass updates the underlying runtime and generator while preserving the existing Solo protocol in a separately named `solo` dialect.

| Component | Selected version and ownership |
|---|---|
| PyMAVLink | Released2.4.50; canonical PyPI archive and SHA256 pinned in the integration lock. |
| DroneKit |2.9.2; retained fork9b1fa763 runtime/setup/requirements already equal current upstream243ce0ac. Existing stdlib/Python3 repairs remain; explicit Solo EKF imports added. |
| DroneKit-Solo |1.3.0; retained fork5d53ef72 runtime/setup/requirements already equal current upstreamc7df21ec. |
| fastCRC |0.5.0; cross-compiled native ARM extension, declared runtime dependency. |
| Maturin |1.16.0 native build backend only; no target Rust compiler installation. |
| Rust mixin | Official Scarthgap `scarthgap/rust` commit868a66da39f2ddbbffaf3e2540c45659558e8301, Rust1.89.0 and paired librsvg2.61.0 build provider. |

The three release archives, versions and SHA256 values participate in graph identity. Cargo.lock crate checksums are pinned. Runtime DroneKit version floors are now satisfied by the modern PyMAVLink package; bounded API tests are distinct from exhaustive upstream compatibility. This uses the minimum official supported Rust cohort needed by the selected current packages, with Yocto6 and NXP kernel/media version upgrades deferred. The SVG provider builds as part of the dependency closure; it is not listed as an installed runtime package in these four image manifests. Existing license work remains deferred.

## Shared protocol source and retained behavior

Pythonv1/v2 and C v1 headers come from one owned XML closure and current generator. SoloLink's C consumers use recipe-provided headers; old generated vendored headers were removed from the app repository. Standard current Python dialects remain independently available. Solo GPS_ACCURACY225 and standard EFI_STATUS225 coexist through distinct dialects, rather than a merged global schema.

Ten missing XML descriptions were added to satisfy strict modern schema validation. An additive mission-type enum supports current waypoint helpers without changing existing enum values or advertising new vehicle capabilities. Optional modern mode/waypoint-color lookups respect absent legacy enums. Accepted-heartbeat cache behavior, lossless char-field text/raw bytes and payload `name` metadata remain compatible with actual app consumers. Optional DataFlash indexing remains disabled, as in the prior product configuration. Generated Pythonv2 is present; this pass qualifies the existing product MAVLink1 wire contract, not a wire-protocol migration.

The independent fixture records166 message IDs/names/layouts/lengths/CRC extras,567 existing enum values and1135 field offsets/widths from the original C/XML contract. The ARM C probes compile2768 assertions per profile and parse/reserialize every complete golden frame byte for byte, including recovery after a damaged CRC. C tail padding is correctly excluded from wire length; a failed initial checker assumption and its stronger field-based replacement remain in the receipt.

Eleven actual host-library tests pass. Both drone profiles pass the same eleven methods under actual ARM Python, covering all256 CRC-extra bytes, golden messages, fragmentation/bad-CRC recovery, text/NUL/non-ASCII forwarding, DroneKit/SoloVehicle construction/listeners, mission commands and GoPro/gimbal paths. All four ARM suites verify actual native fastCRC provenance and known CRC vectors, plus retained crypto/TLS rejection, IPC, numerical and safe-import checks. Drone suites also retain actual ShotManager Follow ROI coverage. The228 common protocol source/XML/native files match across all four profiles.

## Build and recovery evidence

Integration/checkers: `f1484c0965adbec8f8c96d6aa792e90439277976`; product layer: `411217f62fe824276f3575c5f6467d723607be6b`; SoloLink: `1cd50aa3707d37d7ce8c34e5f85fdfbd08c9770a`. BSP1602ee54 and ShotManager2b2e067c are unchanged. Source repositories are locally committed on `codex/mavlink-modernization`. Controller development completed before final checker-only commits; image-producing inputs and graph remained unchanged, and the final source/checker gate records that reuse. Final tracked source/checker hashes and all18 vendor pins were rechecked.

LLVM compilation uses12 internal jobs, one link job and ARM/X86 targets; ordinary BitBake concurrency and pressure limits remain enabled. Native Rust compilation is cached for subsequent builds. A controller-release Meson configure first detected0.2799seconds of host clock skew; an unchanged standard retry passed. No timestamp edits or weakened checks were used. Existing NXP QEMU preferences, WSL I/O accounting, prior forced-packaging taints and deferred license-identifier warnings remain recorded in logs.

The optional new Maturin Cargo wheel inventory contained absolute build paths. Its supported per-project opt-out removes that duplicate nonportable metadata; ordinary Yocto source metadata and pinned crate checksums remain. Package QA then passed without the new buildpaths warning. Earlier strict patch/schema/runtime failures and their repairs are retained.

[Qualification result](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/qualification-result.json), [source/checker gate](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/source-freeze-9.json), [installed provenance](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/installed-protocol-provenance.json) and [preserved artifacts](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/artifacts) identify this candidate. Six files per profile include SD, squashfs, manifest, update archive, MD5 sidecar and boot receipt.

| Profile | SD SHA256 | SD bytes |
|---|---|---|
| controller-development | `67d403e85337f4472ca49237ed427faaceb0b4e2cd7cb94180d74a19ac604c95` | 197133312 |
| drone-development | `12b444df9d19b9a837d82ecd24936c23dfb3bfe9d9063f9b266018045d691ba7` | 192939008 |
| controller-release | `2707845853e844a8efd335c37e0e3c9127024751c1518c32225ff3fa1cba0226` | 180356096 |
| drone-release | `3e4ee61b1cbd841f4b15a2b9800d73b766bd7b7291107e158e35f05d51e994d9` | 171967488 |

[Source transport](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/source-transport) contains34 self-contained Git bundles with verified hashes and the committed integration authority. The three selected checksum-verified PyPI [release archives](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/release-archives) are saved separately. This is not a complete offline mirror of all Rust/third-party downloads, and a new independent full bootstrap was not repeated. The older7055 independent recovery evidence remains separate.

The controller development SD image grew16MiB, from172MiB to188MiB; its automatically sized GOLDEN partition is184MiB rather than the deployed card's168MiB. A future hardware session must use the new SD layout or independently establish space for an in-place update. No card changes were made here. These are incremental/cache-based builds, not a clean full rebuild, hardware/flight acceptance, throughput measurement or security clearance. Existing CVE findings and live Linux-audit follow-ups remain open. Yocto6 and license work remain deferred.

ROS2/RAS-A [research](ros2-rasa-investigation-20261029.md) is a separate completed investigation. Its recommended first experiment is an offboard, receive-only ROS2 gateway using replay/simulator data. No ROS packages or RAS-A definitions were added to the product.
