# Scarthgap maintenance A — implementation and decision ledger

**Local software qualification completed after the user resumed on October 29, 2026.** The qualified graph is `7055fbe5738126848417be71d94a1e4fb6b83a7c507287527bce2ae2e3babf48`: four builds, sixteen baseline checks, four expanded ARM suites, 145 unique host cases in retained/relevant aggregate coverage, four byte-identical cached-rootfs packaging repeats, both-role identity-only signature controls and a real SD-only cache regression. Fresh source restoration and normal offline fetch closures pass. These are scoped software results, with unresolved CVE findings and coverage gaps described below; they do not establish a clean full rebuild, security clearance or hardware acceptance.

The [pause checkpoint](scarthgap-maintenance-a-pause-20261028.md) preserves the earlier `7f9d2899…` stop and eight-repository publication. New resume fixes are committed locally and remain unpublished. Historical source9 and earlier receipts/artifacts remain unchanged. The software qualification itself made no hardware or firmware promotion. The later authorized October 29 controller-card preparation is recorded below; first physical boot now passes the scoped Wi-Fi/SSH checks recorded below. Yocto 6, license work and broad W7 vendor/BSP/media migration remain deferred.

Authority: the [consolidated plan](scarthgap-consolidated-improvement-plan-20261028.md), integration lock/build policy, and owned product/BSP layers. The prior owner Python upgrades are the preserved foundation: recovered Decryptortuning/PORT sources → source9 pins → these reviewed candidate commits/owned-layer patches. No fresh generic upstream/PyPI replacement discards that work. Apparent DroneKit version increases correct stale recipe labels on code already present.

The [180-unit update investigation](scarthgap-package-update-investigation-20261028.md) records current recipes and candidates, including patch-only maintenance. Raw source hashes, upstream source evidence, exact frozen SHAs and later qualification receipts live in `/home/jresp/solo/.integration/scarthgap-maintenance-a-20261028/`. Dirty `build-env.md` remains excluded, unchanged at SHA256 `b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`.

## Work-package disposition

| Work | Current disposition | Remaining limits |
|---|---|---|
| W0 | Central guides and integration front door distinguish the current qualification, historical source9 and pause checkpoint. | Preserve dated history and unrelated owner edits. |
| W1 | Fixed owned recipe revisions/branch checks, explicit seeding, canonical source transport, fresh 34-store restoration, normal offline C/D fetch closure and standard Pixhawk good/missing/corrupt controls pass. | Source/cache recovery is not all-recipes unpack or a clean toolchain/image rebuild. Git mirror archive completion companions are required; ordinary archive checksums remain enforced. |
| W2 | Dirfd-bound mounted-LOG guards and intentional valid-source deletion are retained; fourteen real local rsync/rrsync transfer cases and target version/import plus image/maintenance checks pass. | Actual remote board-to-board SSH transfer and storage behavior remain hardware acceptance. |
| W3 | Atomic offline Wi-Fi pair lifecycle, secret-free golden templates, no WPS enrollment and fail-closed startup/setter/reset paths pass host and installed-image checks. | No application to current devices. Matched physical migration/association/confirmation and TKIP decisions remain deferred. |
| W4 | Legacy provider masks, layer/machine ownership, kernel fragments, retained U-Boot TARGET and graph/profile boot identities pass four-profile builds and validators. Exact unused Matter mask enables fresh offline parsing. | No broad vendor-layer pruning or platform migration. |
| W5 | Selected Scarthgap maintenance sources, rsync3.5.1, OpenSSL 3.5.8 and the bounded RTP patch pass actual build/runtime checks. Standard C/D CVE task closures and dated NVD input provenance completed. | Security findings and CPE coverage gaps remain; prioritize OpenSSL 3.5.9 and the follow-up table below. No blanket security clearance. |
| W6 | Maintained libusb1.0.27/posix-ipc1.2 and stdlib monotonic replacement are installed. Actual ARM TLS/IPC/NumPy, retained-fork imports, FollowShot ROI and nine DroneKit/Solo/legacy-wire methods pass. | Full modern PyMAVLink version-floor compliance and hardware/throughput compatibility are not established. |
| W7 | Separate vendor/BSP/media cohort remains deferred. | Retained kernel/NXP/media pins are not security-clear; drone capture/reset/routing and VPU/DMA/GPU/audio migration remain open. |
| W8 | Four builds, sixteen baseline checks, four expanded ARM suites, 145-case host aggregate, packaging/signature/cache regressions and independent offline recovery completed. | Cached dependencies/rootfs were reused. One certificate-fixture failure after a WSL clock step passed an unchanged strict retry; both attempts remain evidence. CVE applicability has explicit residuals. |
| W9 | User subsequently authorized controller bench preparation; the identified SD card is written, readback-verified and freshly provisioned. | First boot/Wi-Fi/SSH and serial identity pass; reboot/long-soak and broader serial acceptance remain pending. No STM32 flashing; no router settings changed. See the later card handoff below. |

## Exact maintenance sources and security exceptions

Poky `yocto-5.0.21` is `4fb7016a07507683359ebd620f1951fd5ea44bfa`; OE Scarthgap is `2db6aece32c7f8ebf6015b468e1b4e21de072e3b`. Vendor checkouts remain clean and detached. [Official 5.0.21 release notes](https://downloads.yoctoproject.org/releases/yocto/yocto-5.0.21/RELEASENOTES) and local recipe sources support these choices. Effective critical-provider selection passed metadata checks; runtime support still requires installed-image validation.

NXP2.2.2 manifest and meta-imx candidate `e1ec1a20d573e3913f4ad90fa36546ed2bc87715` were investigated read-only. Its 85-file delta includes coupled GPU/media/firmware/kernel and mostly other SoC changes; no independently proven product benefit justified moving the whole vendor set. A retains meta-imx `a0cf309d2e2d544111e520f572db8dff335eca01`, kernel `e0f9e2afd4cf` and U-Boot `6c4545203d12` full pins in the lock. Candidate objects were fetched for review only.

| Family | Selected source evidence / decision | Applicability and unresolved risk |
|---|---|---|
| OpenSSH | Maintained9.6p1 recipe carries2024-6387,2025-26465/26466 and later2025/2026 backports listed in source. Existing key-only/root/Match/receiver policy retained. | VerifyHostKeyDNS exposure and optional Kerberos CVE-2026-59998 depend on actual config. Do not label all SSH findings fixed from patch filenames; sshd -T, real restricted transfer and package/source evidence are W8. |
| OpenSSL | Core3.5.8 replaces3.2.3. Exact NXP3.2 append is masked; owned3.5 append enables generic cryptodev-linux and KTLS for imx-solo. | Three old NXP patches are not wildcard-applied: openssl.cnf KTLS activation and two TLS1.2 e_devcrypto algorithm extensions remain deferred. Building the generic engine does not prove provider/engine activation or acceleration. Actual ARM SSL/hashlib, certificate trust/hostname negatives, TLS1.2/1.3 and provider discovery pass. This does not prove engine activation; hardware offload performance remains W9. |
| rsync/rrsync | Official3.5.1 archive SHA256 `c55f9c9dc10fb8bec397b399a0fdded53cc9a2d8e30891bb0d63724d25c37bef`; same archive's support/rrsync. Reuse maintained core packaging, replace old source patch stack; adapt Makefile regeneration suppression to3.5 source. New optional IDN disabled for numeric receiver address. | Both-role patch QA, native3.5.1 build and 14 local sender/receiver tests passed, including deletion and escape rejection. ARM rsync version and packaged rrsync import/help pass; actual remote transfer remains deferred. The selected upstream script is `upstream/rrsync-3.5.1`. |
| GStreamer | [Official CVE-2026-18649 advisory](https://gstreamer.freedesktop.org/security/sa-2026-0076.html), upstream MR12234 source. Owned exact1.24.7.imx good append adapts H264/H265 adapter caps (32MiB) and reset paths without1.28 property/API changes. | Actual candidate ARM H264/H265 fragment limits and recovery pass; source9 negative controls fail the exact cap. Physical deployment/throughput and broad media ABI/latency/DMA/VPU port remain W7/W9. This one fix does not clear all multimedia CVEs. |
| Python | Selected tag3.12.14 includes CVE-2026-6019 patches. [Official3.12.16 release](https://www.python.org/downloads/release/python-31216/) investigated as an additional same-family security increment. | Additional tarfile CVE-2026-114583, certificate distrust CVE-2026-115557 and mail/tempfile/cookie changes are not claimed present. Bundled expat issues require separate external-expat analysis in Yocto; Windows-only mimetypes change does not establish Linux exposure. Retain tag baseline for A pending focused extra-backport/API review; do not call Python current/security-clear. |
| glibc/base network/fs | Selected maintained Scarthgap recipes/backports for glibc2.39, BusyBox1.36, util-linux2.39, e2fsprogs1.47, hostapd/wpa2.10, dnsmasq and related tools. Every installed recipe has a static candidate/delta row. | Preserve ARM32 ABI, recovery applet/filesystem compatibility and owned append semantics. The resumed reports retain unresolved config-dependent advisories and consumer applicability below. |
| Native build tools | Selected pseudo/tar/rpm-native/patch-native source maintenance; tar wrapper cache is keyed by complete graph and provenance includes source receipt, binary SHA and preserver SHA. | Keep Ubuntu openat2/pseudo workaround until measured fakeroot evidence; new pseudo version alone is insufficient. Preserve CPU pressure gates/all32 affinity/shared caches. No global daemon/cache flush. |

Static filename review found the one active missing-provider append, NXP `openssl_3.2.%.bbappend`, now exactly masked and replaced with owned3.5 policy. Unmatched Qt4, Chromium and Xilinx append filenames belong to inactive dynamic-layer collections; Actual selected-provider and four-profile parse/build checks confirm the active configuration. No `BB_DANGLINGAPPENDS_WARNONLY` or broad version wildcard was added.

The earlier dated source triage was not a CVE database run. The resumed qualification now includes standard controller/drone CVE reports and mapping to 179 retained installed source recipes; the earlier180-unit set differs only by removal of python3-monotonic. Database input, automated report scope and unresolved applicability are recorded below. This is not comprehensive individual security clearance. Evidence/reference material remains separate from product claims.

## Traceability and retained decisions

The plan retains the complete F1–F17, Muse lead N1–N6/R1–R10, twenty specialist identifiers, P01–P23 and190 historical/current unit mapping. Implementation closes F/lead source tasks through the W rows above; W7/W9 and conditional hardware findings stay explicit. P01–P04 are the SSH/OpenSSL/rsync/RTP source changes; P05/P06 stay vendor/BSP holds; P07–P15 consume maintained core/tool sources with the Python/OpenSSL exceptions; P16/P17 cover IPC/NumPy/clock and examined forks; P18 owned app behavior; P19/P20 unchanged payload/vendor integrity; P21 ten removals preserved; P22 recovery/dev policy retained; P23 legacy wireless consumers retained. The all180 ledger supplies a per-unit decision, rather than treating package table presence as clearance.

Rejected prior-audit overstatements remain rejected: fstab `auto` is filesystem type, not delayed-mount permission; current receiver home is `/var/lib/sololog`; validator versions belong in qualification receipts rather than circular image hashes; allowed AUTOREV overrides were not a standards violation but fixed defaults remove an avoidable source hole. No kas authority, broad Python major jump, arbitrary latest package replacement, forced Git refs, Artoo PORT rewrite, UUU removal, or new approval ritual is introduced.

## Source freeze and qualification handoff

The final `source-freeze.json` binds full integration/app/layer/doc SHAs, selected vendor pins, graph, changed paths, source artifact/checker hashes and exact scope. Integration HEAD is receipt metadata, never its own graph pin. Documentation is inventory-only and graph-neutral. Frozen graph source commits must precede all verification execution.

Root seeded the local sololink branch, verified the lock and configured all four profiles; wave3 repeated verify/configure for the current graph. The two standard-fetch Pixhawk files are already copied from proven source9 bytes into `/home/jresp/solo/yocto-cache/downloads/`; a second private copy and `pixhawk-archive-receipt.json` live under the dated evidence directory's `archive-mirror/`. These are exact unchanged packaged payloads, not a new firmware download or promotion. They must accompany exported Git/Artoo source transport; the existing Git transport alone does not prove all source archives available offline.

The October28 handoff requested the four image builds, installed ARM checks, scoped CVE review, packaging repeats and independent recovery. Those local checks are complete in the resumed result below. Future source/checker fixes require a new pin/review gate before execution; hardware work remains separately authorized. Full clean-build reproducibility, broad physical compatibility and security clearance must not be inferred from these software results.


## Final source-review additions and concrete runtime boundaries

The [focused owned-fork report](scarthgap-owned-fork-update-investigation-20261028.md) records exact upstream comparisons. DroneKit and DroneKitSolo selected runtime trees match their latest upstream branches, but that alone did **not** establish compatibility with the separately pinned older PyMAVLink. Source review found missing `source_component` constructor/attribute propagation and `probably_vehicle_heartbeat`, plus Python3 parameter char-array text boundaries. The owned PyMAVLink patches now:

- Append optional source-component arguments without changing existing positional/default-zero callers, propagate them through transport factories/base classes/parser reconstruction, and leave protocol XML unchanged.
- Apply one vehicle-heartbeat predicate to DroneKit callbacks, transport target/mode updates **and the cached heartbeat**, rejecting GCS/gimbal/onboard/ADSB/invalid-autopilot sources coherently. ADSB value27 is confirmed by modern minimal.xml; no enum/dialect replacement is needed.
- Generate char-array text properties backed by exact raw bytes. UTF-8/surrogateescape is reversible; text getters stop at NUL, while original embedded-NUL tails remain in forwarding bytes. Explicit assignment updates backing bytes. Numeric arrays/layouts/IDs/CRCs remain unchanged. Python3 bad-CRC diagnostics use `str(error)` rather than removed `.message`.

DroneKit also uses stdlib time.monotonic and `(str, bytes)` instead of undeclared Python2 compatibility modules. Static source review finds its direct master method names and referenced MAV constants present, but full `pymavlink>=2.2.20` metadata compliance remains **unmet and unclaimed**. Nine actual-library methods now pass against the installed ARM drone packages in both profiles; the common DroneKit/PyMAVLink source/XML trees are byte-identical across all four profiles. These tests target a bounded consumer contract; modern2.4.50 with retained legacy XML is a separate concrete migration experiment. The five changed existing message signatures, ID225 collision, direct dialect imports, C headers and dependency/ARM throughput requirements are recorded in the fork report.

`rsync-patch-dispositions.json` independently adjudicates every old core3.2.7 patch. CVE-2024-12084–12088/12747 are upstream security-history fixes carried forward; CVE-2025-10158 is present and strengthened by negative/upper parent-index bounds in sender.c, and CVE-2026-41035 uses actual `temp_xattr.count` for sorting in xattrs.c. `determism.patch` meets its own drop criterion (`util.c` renamed `util1.c`), missing-prototype fixes are in the newer source/external-zlib layout, and Makefile regeneration suppression is adapted explicitly. These are source dispositions, not proof that all vulnerabilities or runtime contracts are resolved.

Native units beyond the180 installed-recipe count retain explicit maintenance intent: pseudo-native1.9.11 at `ba8887e5f1e922f866681ec7dec1a00b602a9328`, tar-replacement-native1.35, rpm-native4.19.1.1 (`13b4521341781293c41ac898aa9c2d2f6bc1f21d`), and patch-native2.7.6 from selected core. The graph-specific tar wrapper was used successfully by all four rootfs/image builds. This is not a clean-build proof. MAVProxy1.4.20-solo is a locked, uninstalled optional tool with investigated1.8.75 migration; U-Boot is a separately pinned boot artifact. None inflates the180 installed-recipe count.

The earlier paused product graph was `7f9d289975c05df45843d81e8b5656c33589977005c7a1db50094d8b8c99ac36`. Wave3 refreshed only DroneKit patch context and repinned the product layer; all three intended patched Python files are byte-identical to wave2 output, and actual zero-fuzz patch QA passed. Earlier `aba672…` source freezes remain historical receipts. The [pause checkpoint](scarthgap-maintenance-a-pause-20261028.md) preserves the earlier stop; the current qualification is recorded below; the original and wave-specific handoffs remain preserved evidence.


## October 29 resumed qualification — current result

Frozen integration `b7d2bec37a3944f1b3ac16c7f3d20f5e7bcd2689`; product layer `ad893d0ed7fa9ea4004bb647c72279f4b9199c40`; BSP `1602ee542d832ba51b7badad15faad6755acb2a7`; SoloLink `cd3ff3c7e311a5498a746b0187c38e21aa660438`; ShotManager PORT `2b2e067c875f45a7216f77c51797d470b4566e75`. Exact remaining pins are in the integration lock. Integration HEAD is receipt metadata, not a circular graph input. Later documentation/inventory-only checkpoints do not change the qualified graph.

The owner's prior Python upgrades remain the foundation. The following execution waves repaired actual source/consumer boundaries; their numbering is separate from the plan's work packages and historical source9.

| Wave | Observed boundary and resulting repair |
|---|---|
| 4 | The selected core wpa-supplicant recipe installed its generic `conf-sane` runtime file; the old append then renamed that file into the golden template. The owned append now explicitly installs the secret-free owned station template as `/etc/wpa_supplicant.orig`, mode 0600, and removes the generic runtime config. A real install fixture distinguishes competing core/owned inputs and rejects a missing owned source. The image validator caught the original defect and was not weakened. |
| 5 | Actual `FollowShot.filterROI` divides Vector3 by elapsed time, but the retained class only implemented Python 2 `__div__`. The owned app now uses `__truediv__`, preserving component/scalar semantics; real Vector3 tests and a controlled actual FollowShot ROI probe exercise the consumer without sockets, workers or devices. Separately, Avahi's GI scanner reran during install without the library paths exported only for compile. The owned append supplies the same four build-library paths to `do_install`; introspection remains enabled. |
| 6 | High-bit MAVLink CRC-extra values were encoded through UTF-8 text, so otherwise valid messages failed decode. The owned generator patch accumulates exactly one binary byte with `struct.pack('B', crc_extra)`, preserving XML, layouts and decoder behavior. Independent bitwise CRC tests cover all 256 extra values and golden PARAM_VALUE 22 / extra 220 and GOPRO_GET_RESPONSE 217 / extra 202 frames. |
| 7 | The Python 3 text property for a payload field named `name` hid legacy class metadata for DEBUG_VECT 250 and NAMED_VALUE_FLOAT 251 / INT 252. A narrow descriptor preserves the original type string on class access while delegating instance text/raw-byte behavior. Tests cover independent complete golden frames, Unicode/NUL forwarding, assignment, `get_type`, `to_dict`, actual DroneKit dispatch and all 166 legacy message class names. |
| 8 | A fresh offline parse encountered an unconditional AUTOREV in an unsupported, unselected NXP Matter demo. An exact owned BBMASK excludes only that recipe. The active-layer review found no other unconditional fresh-parse exposure; installed product/packagegroup closures do not select Matter. No vendor pin, parser cache bypass or vendor file edit was used. |
| 9 | A legacy semicolon in `ROOTFS_POSTPROCESS_COMMAND` became part of a dependency token, hiding the identity function from BitBake signatures. Removing it restores function/input dependency tracking. Image filenames now use graph/profile identity rather than invocation DATETIME, so an independently rerun SD task addresses its cached squashfs producer correctly. No manual stamps, workdir edits, symlink repair or forced cache clean was used. |

The original Python 3 source-component/heartbeat/char-array backports remain in
place. The bounded tested consumer surface does not claim full compliance with
DroneKit's `pymavlink>=2.2.20` dependency metadata. A modern PyMAVLink migration
with retained Solo XML, direct-import/C-header review and wire/CPU evidence
remains a separate investigation track.


Evidence root: `/home/jresp/solo/.integration/scarthgap-maintenance-a-20261028/resume-20261029T052617Z` (`T`). The source/checker gate is `T/source-wave9/source-freeze.json` plus `root-review.json`. Results are:

- `wave9-profile-matrix-final-result.json`: four successful builds, sixteen baseline checks and four expanded ARM suites. Its lineage preserves the first final drone certificate fixture's error9 rejection when WSL time stepped backward; an unchanged strict retry passed. NTP reported roughly -2 seconds of offset. No verification was weakened or certificate backdated.
- `wave9-repeat-packaging-final-result.json`: all five outputs in each profile match on a forced image packaging repeat from cached rootfs/dependencies. `wave9-independent-sd-repackage-result.json` additionally proves SD-only regeneration reuses rootfs/squashfs and reproduces all five bytes.
- `wave9-signature-identity-result.json`: actual BitBake baseline versus build-ID-only signature controls change rootfs/squashfs/SD hashes for both roles. The noncandidate identity was never used to build an image.
- `wave9-installed-package-census.json`: main controller970 binary packages/165 recipes; drone956/167; initramfs190/58 each. Combined retained source count179 of180; only python3-monotonic was removed. Cached initramfs testdata still names older graphs and is not claimed as a new graph receipt; actual bundled init/source/BusyBox bytes are checked independently.
- `wave9-installed-common-provenance.json`: common DroneKit28 and PyMAVLink78 source/XML files match across all profiles; deployed RTP plugin bytes match the actual ARM positive tests and immutable source9 negative controls. This does not measure physical video throughput.
- `wave9-recovery-summary.json`: fresh34 independent full locked Git stores; normal C814/D816 source-fetch task closures with BB_NO_NETWORK=1; valid stock/green Pixhawk archives accepted and isolated missing/corrupt counterparts rejected by the standard fetcher. Copying64 original Git archive completion companions after archive SHA equality corrected an import omission. Ordinary HTTP/Pixhawk markers remained excluded. Generic binutils input remains an exactly pinned shallow archive, not a fabricated full bare clone. No clean build or unpack-all claim.
- `nvd-standard-incremental-refresh-result.json` and `wave9-cve-run-result.json`: accepted NVD input and paired standard C/D CVE tasks; all four normal configurations restored to CVE mode0 afterward. See the scoped security review below.

The145-case host aggregate is retained139 unchanged/relevant passing cases, one actual Wi-Fi packaging fixture and five ShotManager cases. It is not145 cases freshly repeated at each graph. The nine actual-library ARM methods and their256 CRC subcases/166 metadata comparisons are separate runtime evidence, not extra host-test counts.

All copies and reports are in [software-qualification-7055fbe57381](/home/jresp/solo/recovery/scarthgap-maintenance-a-20261028/software-qualification-7055fbe57381).

| Profile | Preserved SD SHA-256 |
|---|---|
| Controller Development | `d5f8c4c6e4fde132852081d06cb0b4212cea94622e4515d31771337ea20ec2ec` |
| Controller Release | `994d67c5cfec9a1d1c93e9fbcfd3b82e80b9ec46660ff036971b798cc06b6489` |
| Drone Development | `524cd3d8ed1c68dd8bf80c3489027b8271060ae776079c0345e9b2667ce81291` |
| Drone Release | `d1bf4b579b889c94de5eaa849184343ba096a6fc76414e5e249701afd8d7b686` |

## Security review and next maintenance priorities

The current NVD database was populated from checksum-verified official annual/modified feeds using the selected core recipe's exact database functions, followed by a forced standard API incremental refresh from the actual modified-feed watermark. The API processed 50 records through 2026-10-29T07:34:03.481605Z; SQLite integrity passed with 397,484 accepted records and 2,650,831 product rows. Consumed database SHA256: `77beef434ccb70a6ecfbd212a057d72f74af4932e9a23facd18d4048d2145346` in both role checks. Task exit0 alone was not accepted as freshness or clearance.

The installed-target union contains 179 source recipes. Controller main+initramfs has 170 recipes/165 reports; drone has 172/167. Each lacks reports for glibc-locale and four packagegroup metadata recipes. After deduplicating recipe+CVE pairs across roles, the standard reports retain 13,847 Patched, 36 Ignored and 4,613 Unpatched pairs. There are no conflicting role statuses or source-version mismatches after epoch normalization. Controller77 and drone79 reports have no issues and no product records; those are coverage gaps, not security clearance.

The Unpatched severity split is248 Critical,2,076 High,2,245 Medium and44 Low, using nonzero CVSSv4, thenv3, thenv2. Kernel accounts for247 Critical; glibc accounts for the other one. These are automated version matches, not247 demonstrated board attack paths.

| Priority | Concrete source/report evidence | Next bounded work |
|---|---|---|
| glibc | CVE-2026-5450, score9.8. The selected2.39+git source retains the vulnerable scanf buffer-growth expression, with no matching selected patch. The [official advisory](https://sourceware.org/pipermail/libc-alpha/2026-April/176841.html) describes an allocation overflow with specially sized `%mc` input; a reachable Solo caller was not established. | Review/backport the source fix and the separate ungetwc finding, preserve ARM ABI, run a bounded reproducer and repin/requalify. This is unresolved source evidence, not a proven remote controller exploit. |
| OpenSSL | All13 Unpatched findings match3.5.8; four are High. Official3.5.9 fixes the listed issues. ARM64/RISC-V SM2 optimization does not describe this ARM32 target; QUIC/DTLS/CMP/certificate use needs caller-specific review. | Prioritize a same3.5-family update or verified backports, including existing NXP configuration and actual ARM TLS/crypto regression. |
| OpenSSH | Fifteen Unpatched, ten Low/five Medium. Built GSSAPI/KRB5 exclusions and server Compression no narrow specific findings, without changing raw statuses or clearing all clients. | Review installed sftp traversal, username handling, agent binding and forwarding fixes; distinguish actual client/server configuration and source lineage. |
| Python | One standard Unpatched entry, CVE-2025-12781, concerns alternate base64 alphabets. No relevant owned caller was found in the bounded search; third-party/operator callers remain unreviewed. | Review maintained3.12 backports and explicit alphabet validation; continue the separate3.12.16 investigation rather than treating one scanner entry as complete coverage. |
| Kernel/media | Kernel6.6.52 has4,388 raw Unpatched; controller FFmpeg4.4.1 has56; GStreamer core has56. Other retained families include db31/libsoup24/libxml2 8. | Keep the separate W7 vendor/BSP/media cohort explicit; assess enabled ARM/i.MX paths and vendor ABI before a source migration. |
| GStreamer mapping | Base/bad/good/libav product names have no accepted product matches. The good plugin's explicit CVE-2026-18649 Patched entry has source and actual ARM cap/recovery evidence, but does not clear the plugin family. | Correct reviewed component/CPE mappings in a new source gate and rescan; never infer clearance from a missing product match. |

The exact read-only [triage summary](/home/jresp/solo/.integration/scarthgap-maintenance-a-20261028/resume-20261029T052617Z/wave9-cve-triage.md) and9.5MiB JSON retain every raw status, input hashes and bounded source/config annotations. No CVE ignore rule, product mapping or source version was changed during triage. Rsync3.5.1 has zero reported Unpatched and50 Patched per role, which still does not constitute comprehensive security clearance.


The scanner's Patched status includes versions outside affected ranges; it is not a count of independently verified backports. Missing reports, no CPE matches, unanalysed NVD records and simplified AND/OR/negate handling remain coverage limitations. Functional tests do not resolve these findings.

[Official OpenSSL 3.5 release notes](https://openssl-library.org/news/openssl-3.5-notes/) confirm 3.5.9 as a September 29 security patch release, with its most severe fixed issue rated High. A reviewed same-family 3.5.9 recipe/backport and repeat qualification are the immediate crypto follow-up; no change to the frozen 3.5.8 candidate is implied. Python 3.12.16/additional backports, actual SSH client/server applicability, corrected GStreamer product mapping and the separate retained kernel/media cohort remain explicit follow-ups. Yocto 6 and license work remain deferred.

No hardware, controller/router SSH configuration, SD writes or STM32 flashing occurred during the software qualification. The older published checkpoint is distinct from these new local commits and the artifacts that were then unflashed. The subsequent authorized card preparation is a separate operation below.

Recovery inputs are separate: [Git/firmware transport](/home/jresp/solo/recovery/scarthgap-maintenance-a-20261028/source-transport-7055fbe57381) and the independently copied [standard downloads](/home/jresp/Active_GH/opensolo-maintenance-a-qualification-7055fbe57381/workspace/yocto-cache/downloads). Retain both when preserving this recovery set. Git bundles alone do not contain ordinary source archives; fetch-only success does not establish a clean compilation on another host.

## October 29 controller SD preparation — first physical boot pending

After software qualification, the user explicitly approved overwriting the attached
7,948,206,080-byte controller card, Windows Disk 4 / WSL `/dev/sdf`, in USB reader
`05e3:0764`, serial `000000002961`, LUN 1. No card backup was requested or taken.
The write actor rechecked the exact device identity, unchanged original partition
layout and source bytes before opening the card exclusively. Source/actor hashes
and the actual producer/consumer review were frozen before execution in
[execution-gate.json](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/execution-gate.json).

The preserved Controller Development image for graph `7055fbe57381…` was written
and all 180,356,096 bytes passed stream-only readback after block-cache
invalidation, SHA256 `d5f8c4c6e4fde132852081d06cb0b4212cea94622e4515d31771337ea20ec2ec`.
The pinned production initramfs functions then created the normal fresh four-partition
layout offline and seeded LATEST from GOLDEN. This used native WSL utilities and
an adapter that invokes only provisioning/seed functions, never boot dispatch.
Both slots' four payload hashes match the qualified boot manifest, raw U-Boot
matches, and all four read-only filesystem checks returned zero.

Fresh RWFS settings use the installed Wi-Fi helper and canonical secret-free
templates. The read-only OpenWrt query supplied the selected 2.4 GHz WPA2 network;
credentials remain private, with no router settings changed. Station mode is
enabled, SoloLink AP mode is disabled, DHCP hostname is `opensolo-controller`, and
the actual next-boot Wi-Fi merge preserved the provisioned credentials. The
installed offline provisioner generated a fresh host identity on the card and
installed the existing workstation administrator public key. Only its public host
key was read into workstation staging and pinned for the existing SSH alias;
strict host-key checking remains enabled. No private host key was exported.
No old overlay was restored and no programs are shadowed by the fresh overlay.

All filesystems were unmounted, card buffers flushed and the reader detached from
WSL. Evidence: [card-complete.json](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/card-complete.json),
[settings-complete.json](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/settings-complete.json)
and [image-readback.json](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/image-readback.json).
Partition/settings provisioning changes the original SD-image bytes after the
successful readback; no final whole-card/source-image hash equality is claimed.
Offline provisioning does not test the physical blank-card first-boot path.

Next acceptance is actual controller boot, fresh-image Wi-Fi/SSH, reboot stability,
service logs and normal i.MX–STM32 communication. STM32 firmware is unchanged;
there are no automatically discoverable MCU firmware candidates or enable/force
markers. No MCU ROM/programming operation occurred. The development firmware's
normal idle shutdown remains until a separate Artoo bench deployment is selected.
The software candidate's recorded security findings remain unresolved.

### First fresh-image controller boot — Wi-Fi and SSH pass

The user returned the prepared card to the controller and powered it on. The
read-only check verified graph `7055fbe57381…`, OpenSolo 5 development, LATEST boot,
`pci=nomsi`, normal ath9k hardware crypto (`use_msi=0`, `nohwcrypt=0`) and increasing
legacy ath9k interrupts. OpenWrt confirms authenticated/authorized association and
DHCP `192.168.2.207` for MAC `88:dc:96:3b:aa:30`. The ordinary
`ssh opensolo-controller` alias resolves and authenticates with the pinned new host
key and existing administrator key. Initial hostname/SSH attempts preceded network
startup; subsequent connections succeed without disabling strict checking.

Byte-exact 1 MiB memory-only SSH transfers pass in both directions, without target
file writes. The recorded observation retains one boot ID, a stable association
and zero ath9k reset/watchdog/timeout counters. The final capture is at uptime
138.82 seconds. No reboot or prolonged soak was requested or performed.

Startup logs identify STM32 hardware3 and firmware `OpenSolo-4dev` over the opened
`/dev/ttymxc1` bridge. The updater reports no new firmware. The user's LCD reads
“Controller updated”; the user was instructed to press A to dismiss it. No STM32
programming, controller configuration write, clock change or router change was
performed during these checks.

Nonblocking startup follow-ups remain: the controller clock is still at the build
time baseline, the serial-console getty logs controlling-TTY warnings, and the
expected dedicated `/log/3dr-stm32.log` is absent while STM32 messages are present
in `/log/messages`. These were recorded, not repaired in this read-only session.
This qualifies the scoped first boot/network/SSH and MCU identification, not
blank-card provisioning, drone pairing/video, flashing or flight behavior.

[Read-only boot result](/home/jresp/solo/.integration/controller-card-preflight-20261029T141456Z/boot-check-20261029T142858Z/result.json) and its adjacent controller/router captures retain exact evidence.

### Live Linux audit follow-up

The user then requested a broader running-Linux review. The [dated runtime audit](controller-optimization-audit-20261028.md#october-29-fresh-image-runtime-audit) and [raw read-only evidence](/home/jresp/solo/.integration/controller-linux-audit-20261029T143240Z) distinguish the actual Maintenance A image from the historical October28 baseline. No new source candidate, build, live settings change, device/serial/watchdog write or firmware operation occurred.

Highest-value next work is the demonstrated fortified/custom logger split, actual BusyBox rotation installation, a real network-time source and getty session ownership; hardware-watchdog ownership and measured GPU/stack reductions follow with appropriate bench gates. Current dual-stack management firewall, key-only access, AP service gating, normal ath9k interrupts/crypto and real-time bridge threads are working. Six approximately8 MiB anonymous worker mappings and the128 MiB GPU pool are measured opportunities, not proven reclaimable margins under video. Existing CVE findings remain open.

The last good live capture is uptime594.1 seconds of the same boot, with zero ath9k resets and increasing serial counters. OpenWrt logs DHCP release/disconnect at14:39:40 UTC, about11 minutes after association; the user confirms the controller powered down itself. No repeated reconnect or driver hang was observed in that interval. The controller is now off; repeat boot and longer loaded acceptance remain pending.


## October29 modern MAVLink candidate — local software qualification complete

The requested latest-library work is complete locally: [MAVLink modernization](mavlink-modernization-20261029.md). Graph `28fd826a52ca4accf0743f3ef3492ae2fe87e608bcb9adc8f4509abc635db244` passes four builds, sixteen baseline image/initramfs/userspace/maintenance checks, four actual ARM runtime suites and four compiled C wire suites. Eleven current consumer methods pass on host and in both ARM drone profiles; all166 messages,567 old enum values and1135 original C field layouts are retained. Each C suite passes2768 assertions and byte-exact frame roundtrips. The228 common protocol source/XML/native files match across profiles. New self-contained source transport has34 verified Git bundles; three selected release archives are retained separately. A fresh complete offline recovery or clean build is not claimed.

[Final receipt](/home/jresp/solo/.integration/mavlink-modernization-20261029T151457Z/qualification-result.json) records source/checker freeze9, complete per-profile results, preserved artifact hashes and failed-wave/unchanged clock-retry lineage. The prior7055 artifacts and deployed controller remain unchanged. New controller SD/GOLDEN allocations grow16MiB; verify layout/space before any future hardware installation. No pushes or flashing occurred. ROS2/RAS-A [research](ros2-rasa-investigation-20261029.md) completed separately without product integration. Existing Linux-audit/CVE follow-ups, Yocto6 and license work remain deferred as previously recorded.

## October29 controller Linux follow-through — local software complete

[Controller Linux maintenance](controller-linux-maintenance-20261029.md) records the implementation of fresh-image audit R01–R04 on top of the modern MAVLink candidate. Current graph `be98b385025c2eff8569d94ab732079ac332ddacfd139c960140f660c5dc7888`, integrationf9b09225, productfbf7744a and SoloLinked88f8c9 are frozen and locally committed on `codex/controller-linux-maintenance`. The product owns Chrony4.9 configuration/service, effective BusyBox rotation and machine-specific UART0 exec wrappers; SoloLink owns its explicit fortified/nonblocking API and GPS-clock fence. All18 vendor pins remain unchanged.

Four builds and sixteen baseline checks pass, plus four expanded ARM suites, four MAVLink C wire suites and four actual installed Linux feature suites. The latter cover saturation/drop/reconnect/errno/formatting, actual syslogd facility/rotation/ENOSPC, direct-stream rotation, real getty session/prompt and nonmutating loopback NTP. Clock/process boundaries in lifecycle tests are simulated; the host clock and physical devices are untouched. A private QEMU11.1.3 tool with a narrow NULL-timeout ARM time64 syscall shim enables the Chrony test; it is not shipped or installed globally. Prior console-install/checker-fixture failures and the unchanged drone-release TLS retry after a backward WSL wall-clock observation remain in the receipts.

[Qualification receipt](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/qualification-result.json) and [preserved artifacts](/home/jresp/solo/.integration/controller-linux-maintenance-20261029T195113Z/artifacts) bind these results. Source transport and four explicit release archives are retained separately. Controller development grows another4MiB (192MiB SD plus1024bytes;188MiB GOLDEN), compared with168MiB GOLDEN on the deployed card. No new offline bootstrap or clean rebuild is claimed. No hardware changes, flashing or pushes occurred. Physical7055, historical28fd, unresolved security findings and the owner's build-env.md changes are preserved. R05 watchdog and R06 measured memory/stack work remain later bench tasks; Yocto6 and license work remain deferred.

## October29 be98 controller card prepared — boot pending

After exact-target approval, the7,948,206,080-byte controller card (Windows Disk4, WSL /dev/sdf, reader000000002961/LUN1) received the qualified be98 development image. All201,327,616 image bytes passed cache-invalidated readback. The production provisioning functions created188MiB GOLDEN and LATEST,100MiB RWFS and the remaining LOG, and seeded matching boot payloads. Fresh station-only Wi-Fi/DHCP and key-only administrator access are provisioned; the workstation pins the new on-card public SSH host identity with strict checking retained. Both slot payloads and raw U-Boot match the qualified manifest, and all four unmounted read-only filesystem checks pass.

[Card result](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/card-complete.json), [settings](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/settings-complete.json), [image readback](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/image-readback.json) and [SSH public-key pin](/home/jresp/solo/.integration/controller-card-preflight-20261029T210144Z/ssh-host-pin.json) preserve exact evidence. The card was flushed, all filesystems unmounted and the reader detached from WSL. No card backup, old-overlay restore, private host-key export or STM32 operation occurred. Offline layout/settings preparation changes bytes after successful image readback; no final whole-card/source-image equality is claimed. Actual boot, LAN/SSH, clock discipline, dedicated logging, console warnings and reboot/soak remain to be checked. The last observed running baseline was7055 with OpenSolo-4dev; card preparation alone does not establish a new running baseline.

## October29 be98 first boot and corrected Chrony candidate

The prepared be98 controller boots and passes scoped Wi-Fi/DHCP, strict-key SSH and memory-only transfer checks. R01 dedicated local0 logging, R02 actual `-s 2048 -b 3` and R04 getty session ownership are confirmed; continuing UART1 STM32 hardware3/OpenSolo-4dev traffic and zero sampled ath9k resets are recorded. R03 fails because its boot script invokes an install utility absent from the minimal image. This defect was masked by the earlier host fixture and is not an established network/NTP fault.

The [Chrony startup follow-up](chrony-boot-fix-20261029.md) records the physical evidence, local correction and stronger actual-ARM utility coverage. New local graph `faf3df65da77abf796fe1ff23ea5e657d64a3b4245d051ff4407bc94300a40a3` passes four builds, sixteen baseline checks and four each of expanded ARM, C wire and Linux suites. Productab95c94 and integrationd886047 are locally committed. Actual installed script hashes, all frozen source inputs and18 vendor pins match;24 artifacts and verified34-bundle transport are preserved. The live board remains be98; no clock/service/file change, reboot or STM32 programming occurred during this inspection. A prepared init-script-only patch and Linux reboot await live-change authorization. Watchdog/memory work, broader hardware/security acceptance, Yocto6 and license work remain deferred.

## Authorized live correction and warm reboot — pass

The user explicitly approved the init-only SSH correction and Linux reboot. The actor verified the running be98 identity, standby level and exact old script SHA256, then atomically installed the tested root-owned0755 script and verified its syntax/hash. The i.MX reboot changed boot ID from7ed88434-bf80-4b49-bc3a-70d8e4e54067 to4ca4253b-8ec9-4c53-8ea7-b26bca12dbb7. The same strict host-key SSH alias reconnects after Wi-Fi authentication/DHCP. STM32 remains hardware3/OpenSolo-4dev with continuing UART1 reports; no MCU programming occurred.

The early Chrony query corrected the saved-floor offset by94138.686032seconds before runlevel3 and bridge startup. The background process is `/usr/sbin/chronyd -R -f /etc/chrony.conf`, running UID/GID998 with only CAP_SYS_TIME effective; sampled RSS is1664KiB. Chrony reports stratum2, normal leap status and three online sources. Its owned0750 runtime directory and root-owned GPS-fence marker exist. Actual UDP tables show no NTP-server123 or command323 listener. Plain NTP is still unauthenticated; no authenticated-time claim is made.

The last capture is uptime117.91seconds in that new boot. The saved14-digit checkpoint advances from20261029212014 to20261029212114. Dedicated local0 STM32 logging, actual2MiB/three-generation syslog arguments and UART0 getty session leadership remain correct; all sampled ath9k reset counters are zero. This passes one scoped warm reboot and standby clock/network/bridge check. Physical flight-level/GPS handoff, electrical console authentication, offline-network boot, loaded soak, pairing/video and flight remain outside this session.

[Live result](/home/jresp/solo/.integration/chrony-boot-fix-20261029T211214Z/live-install-20261029T211931Z/result.json), [execution gate](/home/jresp/solo/.integration/chrony-boot-fix-20261029T211214Z/live-install-20261029T211931Z/execution-gate.json) and adjacent captures preserve exact evidence. The running device is **be98 plus one documented init-script overlay correction** (SHA256f0b357900dcb71630cf9c43f7a93772a569a0284973f25e00c09ffa9bebfdf2f). Its VERSION is unchanged; the complete faf image was not installed. Future full-image maintenance must account for this overlay override. Source changes and docs remain local, with Yocto6/license work deferred.

## October29 Artoo application link passes; reset/ROM gate remains open

The [Artoo readiness review](artoo-flash-readiness-20261029.md) verifies sole UART1 bridge ownership, actual115200/8N1/no-flow-control settings without a new UART open, and100 identical172-byte parameter readbacks with no timeout (median19.93ms). The current record matches the prior factory SHA and passes the installed-matching calibration validator. Loader/updater hashes match owned sources; automatic firmware candidates and update-enable markers are absent.

Both current4803457 DVT production/bench artifacts match their manifests and exact prior Renode evidence. No new build/emulator run was relabeled. **Flashing is not cleared yet:** prior GPIO46 reset attempts restarted Linux and no complete ROM ID/geometry/readback/release cycle is qualified. Battery-only power is observed, not established as the cause, and no charger remedy is claimed. The reset/power-control path and intrusive old tracing timing need a controlled non-flashing diagnostic. No bridge stop, GPIO write, reset, ROM entry, target file write or MCU programming occurred in this readiness session; calibration is preserved privately. The old manual helper is not the current bench deployer because it reconstructs a version without the bench suffix.

## October29 authorized reset diagnosis — flashing remains blocked

[Reset diagnosis](artoo-reset-diagnosis-20261029.md) records three controlled comparisons. A normal BOOT0low reset restarted Linux with both nominal100ms and1ms pulses; the normal bridge/8N1/UARTdescriptor remained untouched. A separate BOOT0high/8E1 test using the paused bridge's duplicated descriptor also restarted Linux before a recorded ROM ACK. Inactive-low preparation verified live replies, and the new pulse interval contains no storage sync/logging. These results locate the failure in the active reset/boot path, without proving electrical pulse duration, actual ROM entry, a power-brownout cause or a charger remedy.

The controller recovered to boot39e83092 with OpenSolo-4dev, one UARTbridge, unexported control GPIOs and20 matching factory-record readbacks. No erase/write/protection/MCU flash occurred, no updater marker was enabled and no automatic retry is scheduled. Early J2 console/reset-cause capture and actual BOOT0/NRST/power observation are the next useful hardware evidence; no console adapter is connected. Diagnostic source and central docs are locally committed; existing image/source9 receipts and owner build-env.md remain preserved.

## October29 one-shot automatic-update boot preflight — reset reproduced

The [boot-time Artoo preflight](artoo-boot-preflight-20261029.md) ran in actual rcS/S60updateArtoo context, before the normal bridge. Installed version lookup and full calibration reads passed, then Linux restarted during the actual loader initChip call before any recorded ROM ACK. Erase/write/protection/Go were blocked and the ordinary updater run_update/programming path was never called.

The persistent one-shot claim worked on the recovery boot: its boot log explicitly records consumed/skip without invoking the updater. The exact original wrapper and permissions were restored after evidence collection. OpenSolo-4dev, the full factory-record SHA, normal UART ownership and synchronized Linux time remain intact; no firmware candidates or update-enable markers were installed. Early boot did not bypass the reset blocker, so physical reset/console evidence is still needed before programming. Sourcec2f146b and five native guard tests are recorded; no probe retry or rearm is scheduled. Owner build-env.md, historical receipts, Yocto6 and license deferrals are preserved.

## October29 legacy controller reset/power comparison — BSP omission found

The [legacy power comparison](artoo-legacy-power-comparison-20261029.md) traces the old recipe-selected U-Boot tag solo_v1.0.0 and kernel tag sololink_v1.3.0-5. The old controller config actively drives GPIO1_IO19 and GPIO1_IO21 high, described as the STM32/i.MX6 POWER-ON pins. Modern board code omits those writes, and retained pinctrl mux entries do not set GPIO direction/level. Reset GPIO45/46, mux tuples, UART pins and loader pulse sequence match. MCU early PC14/PC15 power handling is unchanged.

This confirmed source omission is the leading explanation for host loss during MCU reset, but the actual GPIO1 direction/levels and mainboard power waveform remain unmeasured. Controller DNS was unavailable; no new hardware operation was attempted. No product source change, build, reset, erase/write or flashing occurred. The next work is a read-only GPIO1 snapshot and a controller-specific local restoration candidate with handoff/shutdown review before any hardware trial. Documentation and tagged source snapshots are preserved locally; owner build-env.md, prior receipts and Yocto6/license deferrals remain unchanged.

## October29 legacy power holds resolve the tested reset blocker

The authorized [live investigation](artoo-power-hold-test-20261029.md) confirms GPIO19/21 are inputs at cold boot. Temporarily driving both high reproduces the legacy controller behavior: the existing100ms application reset now passes with unchanged Linux boot ID and20 identical calibration replies. The following guarded ROM request returns ACK79, ID414hex, ROM22hex,256KiB geometry and matching UID; repeated vector/parameter reads pass before returning to the old application. No erase/write/protection/Go or flashing occurred.

The first hold attempt stopped before reset because PSR checking was too strict with SION disabled; corrected sources were refrozen before retry. All temporary GPIOs are again inputs/unexported, original UART settings/bridge operation recovered, and20 final parameter replies match factory SHA. Transient UART error counters stabilize after recovery. Original updater wrapper and loader hashes remain unchanged, with no firmware discovery/enable markers. Sourcef721524 and docs are local only. This establishes the temporary hold remedy; permanent controller BSP/updater integration and normal shutdown qualification remain pending before ordinary flashing.

## October29 authorized OpenSolo5 bench MCU installation — pass

The user authorized flashing through the tested temporary hold setup. One guarded upload installed DVT `5.0.0-dev+g48034572145e-bench-noidle`, SHA256b4909e99…,190976bytes. Actual chip/UID/256KiB geometry and factory calibration were checked, old full MCU flash preserved privately, and only pages0–126 erased. Complete post-write flash readback matches the candidate and FF application tail; the full2048-byte page127 is unchanged. Linux retains bootac525ce9 throughout.

The immediate application query rejected an oversized packet, so the actor retained holds/paused bridge and did not repeat programming. Later settled verification after receive-buffer drain returned the exact version/HW3/UID and unchanged calibration without another reset/write. The user confirms OpenSolo5 branding and Waiting for Solo. The same bridge resumed with20matching replies; all temporary GPIOs are inputs/unexported. Original updater/loader and automatic-update markers remain unchanged.

[Installation result](artoo-flash-result-20261029.md) binds source82cda9e, seven guard tests, preserved before/after flash, initial failure and separate successful settled verification. Permanent controller power-hold integration, startup resynchronization improvements and broader hardware/shutdown/flight acceptance remain open. New work is local and unpublished; owner build-env.md and Yocto6/license deferrals are preserved.

## October29 permanent controller power lifecycle and startup verification — locally qualified

[Implementation and qualification](controller-power-fix-20261029.md) records graph7bac1220def2ca664d584961eaf08ff6cac409e1c2601f4688e7a586db468a8e, BSP2819766, SoloLink045b213, product2d18eeb and integration72731fa. Controller-only U-Boot asserts GPIO19/21 high with safe latch/direction/mux ordering and leaves BOOT0/reset inputs. The built-in kernel power driver retains holds across MCU reset/reboot and releases both to inputs at final power-off. Actual rc0 halt uses -p; driver binding is required before updater programming. Drone power behavior is excluded by config and DT.

Updater identification now flushes stale RX once, requests periodically and consumes startup/unrelated/oversize frames within an8second monotonic receive budget. Versions remain exact including suffixes. Loader failure is a single programming attempt per invocation, with no immediate retry/unprotect escalation. The standalone temporary-flash reader also resynchronizes within its deadline. Existing update-enable policy, staged firmware inventory and reserved-page/calibration logic remain unchanged.

All four development/release controller/drone builds and16 baseline checks pass. Installed ARM Python exercises startup-noise recovery in all profiles;108 integration and41 SoloLink unit tests pass, with focused C tests repeated after the DT ordering correction. Actual compiled U-Boot instructions, controller-only linked kernel handlers, GPIO phandles and installed app-source hashes are checked.32 deployment artifacts and four verified owned-source bundles are preserved alongside source gates and logs. The first native stub missing boolean definitions and first DT root-property ordering failure were corrected/refrozen; superseded builds remain recorded and an old in-flight drone build was stopped at a task boundary.

No SSH modification, card write, deployment, MCU programming or push occurred. Physical Linux remains be98 plus the existing Chrony overlay and MCU remains the previously installed4803457 bench image. Cold boot/driver binding, warm reboot, guarded reset and normal final electrical shutdown remain later authorized hardware gates. Qualification is cached/incremental and does not establish clean/offline recovery, security clearance or unrelated flight/media/watchdog acceptance. Owner build-env.md, prior receipts, Yocto6 and license deferrals are preserved.

## October29 Artoo modernization research and plan

Three fresh agents reviewed current runtime reliability, build/vendor upgrades and protocol integration. The [consolidated plan](artoo-modernization-plan-20261029.md) records concrete source gaps, ordered implementation waves and acceptance criteria. The Linux Telem.cpp filter incorrectly compares message IDs to LOG_DATA length/CRC constants; this remains an unfixed finding. Tamp relevant C sources already match v2.5.0; Pillow13 and selective CMSIS6.4/CubeF1v1.8.7 are candidates. Arm15.3.Rel2 archive identity remains unresolved. Diagnostics must account for startup clearing the backup domain, pre-runtime clock waits, and the existing sole UART owner. A two-page journal requires loader preservation changes; active watchdog requires reset/power acceptance.

Only central documentation changed, committed locally. No builds/tests, source implementation, package installation, SSH/hardware access, firmware promotion, flashing or pushes occurred. Physical Linux/MCU and graph7bac local qualification remain unchanged. Yocto6/license work stay deferred and owner build-env.md is preserved.


## October29 Artoo modernization — local implementation and qualification complete

The [Artoo modernization implementation](artoo-modernization-result-20261029.md) is the latest local software candidate. Artoo `d3cc937` adds retained fault/runtime diagnostics, bounded startup, a verified modern MAVLink1 export and CMake provenance. The selected isolated Arm15.3/Pillow13 candidate passes all six profiles and production/bench DVT each pass13 Renode scenarios. Linux graph `e0dafae6…` includes the diagnostic bridge, forwarding fix and prior permanent power-hold BSP; all four builds and16 baseline checks pass. Exact artifacts and source/checker identities are preserved. No hardware changes, flashing or pushes occurred; deployed Linux/MCU identities remain unchanged. Flash-stall recovery, persistence/watchdog design and physical acceptance remain open.

The final freeze8 combines MCUd3cc937, SoloLink6d16834, product17da8f8 and integration0ac58d5; BSP2819766 is unchanged. Six baseline, two Pillow-only, two Arm-only and six combined builds pass all16 linked-image checks. Pillow13 preserves800 assets/405717 pixels exactly; Arm15.3 saves1680 production-DVT bytes with8 additional static RAM bytes. Production/bench combined candidates each pass13 Renode scenarios. Twelve native MCU suites,108 integration tests,44 SoloLink tests,166-frame native/ARM wire checks and an actual ARM bridge fixture pass. Native numeric suites are host tests; target coverage does not establish exhaustive arithmetic equivalence or hardware acceptance.

Linux inputs remain frozen sincefreeze5. All four images and16 baseline checks pass;32 independent deployment copies and manifests are preserved in the receipt. Both controller roots install the source-matching diagnostic CLI; neither drone does. The new MCU remains a separate artifact and is not promoted into Yocto's staged firmware. The original system tools/venv and owner build-env.md remain unchanged. New source and docs are local only. The [implementation report](artoo-modernization-result-20261029.md) records exact SHA256 values, producer/consumer contracts, corrected failed attempts, reproduction commands and later physical/design gates.


## October30 watchdog, parameter journaling and SRAM flash containment — local qualification complete

The [watchdog, parameter journal and flash-containment candidate](artoo-recovery-result-20261030.md) is now locally qualified. Artoo `629c14b` adds redundant CRC/commit-last parameter snapshots and SRAM flash containment; a separate bench image enables the experimental watchdog, which remains off by default. Coordinated loader/updater changes preserve both reserved pages and the full 172-byte parameter payload. All seven firmware builds, three 14-scenario Renode runs, four Linux builds and 16 baseline checks pass; both controller roots pass the installed ARM loader fixture. Linux graph `5b00a161…` and exact artifacts are preserved. No hardware access, flashing, staged MCU promotion or pushes occurred. Physical Linux remains be98 plus the Chrony overlay, and MCU remains4803457 bench. Watchdog timing, power-loss behavior, permanent power holds and broader hardware acceptance remain unqualified.

Final freeze4 combines MCU629c14b, SoloLink3d93574, loader327dac3, product7cbe9d6 and integrationa7e8c50; BSP2819766 is unchanged. It adds only an installed ARM checker over freeze3 runtime/build inputs. Seven builds and seven linked-image/SRAM checks pass. Fourteen native firmware suites plus build-tool tests, 2,242 modeled journal interruption positions,108 integration tests,44 unchanged-source SoloLink tests and15 loader tests pass. Production, bench and experimental bench DVT each pass14 Renode scenarios; ADC modeling does not activate the experimental IWDG, so expiry/reset behavior is not established.

All four cached/incremental Linux builds and16 baseline validators pass, along with both actual installed ARM loader fixtures and source-hash comparisons. The loader is intentionally absent from drone roots; an initial overbroad probe invocation and its FileNotFoundError are retained. Thirty-two independent Linux deployment copies, seven MCU artifact sets and owned-source bundles are preserved in the new receipt. Earlier failed containment capture and review-found generic-loader/watchdog-entry gaps were corrected and refrozen before final tests; old attempts remain recorded.

The journal requires a coordinated compatible loader/update path, and old direct flashing actors are retired. No journal downgrade conversion is supplied. Flash-stall handling is terminal SRAM containment, with diagnostic recording and selected load outputs off, not automatic reset/recovery. Watchdog activation remains explicitly bench-only and default-off; quarantine disables new activation rather than stopping the application. The report records migration/raw-legacy limitations, physical power/LSI/flash/UART acceptance gates and reproduction inputs. Owner build-env.md and historical receipts remain intact; changes are locally committed, with Yocto6 and license work deferred.


## October30 battery/controller review deferred; flash preparation resumed

The user deferred implementation of the [battery/controller findings](artoo-battery-controller-followups-20261030.md). Sensor freshness, general-fault output handling, charger/thermistor validity, sample settling, shutdown/persistence and percentage consistency remain open; no runtime source or candidate changed. Preparation selects the existing629c14b DVT bench artifact with experimental IWDG disabled. The journal requires compatible loader/updater and permanent BSP power holds before installation. Preparation permits local checks and read-only connection inspection, not an erase/write, service stop, reset or automatic-update enable. See the separate preparation receipt/result for current connection status.

[Flash preparation](artoo-flash-preparation-20261030.md) selects the exact bench binary and controller-development graph5b00a161. Existing artifact hashes/profile/layout match qualification; live preflight remains pending because the SSH name is unresolved and the prior IP times out. No active DHCP lease or WSL-visible physical SD device was found. Compatible Linux/BSP installation must precede journal firmware; the user’s current power/card status is pending. No deployment, serial open, service stop, GPIO write, reset, erase or programming occurred.


## October30 authorized controller card update — graph5b00a161 prepared, boot pending

**Controller card prepared, boot pending:** The authorized7.95GB card now contains controller-development graph `5b00a161…`, including permanent power holds and journal-aware tools. Full image readback, both boot slots, raw U-Boot and four filesystem checks pass. Fresh station-only Wi-Fi/SSH is provisioned and the new public host key is pinned. No SD backup was made; automatic Artoo updates remain disabled and STM32 is unchanged. The reader is detached from WSL. This is a prepared card, not yet a verified running Linux baseline. [Installation receipt](/home/jresp/solo/.integration/controller-card-preflight-20261030T174903Z/card-complete.json).

Fresh detection identified Windows Disk4 / WSL/dev/sdf,7,948,206,080 bytes, USB05e3:0764 reader000000002961 LUN1. Both previous slots held be98; the controller was not reachable because the SD was in the PC. The user explicitly approved overwriting this exact card, recreating Wi-Fi/SSH and taking no backup. The frozen single-use actor rechecked identity/layout/holders and preserved source hashes, wrote201,327,616 bytes once and verified a cache-invalidated full-length SHA256 readback of67744c3d8def65b7819f4c20dc53057ed49ff8a3cf28529f06f34d52bb626b16.

Pinned production provisioning created188MiB GOLDEN/LATEST,100MiB RWFS and remaining LOG. Both slots and raw U-Boot match the qualified manifest. Router-derived credentials remained private; installed provisioning helpers create station-only Wi-Fi/DHCP, public-key admin access and a fresh on-card host identity. Only its public key was exported/pinned with strict SSH checking retained. Old overlays were not restored; the Chrony correction is already in the new image. No firmware candidates or update-enable markers were installed.

All four unmounted read-only filesystem checks pass. Inspection earlier encountered a short loop-autoclear unmount delay; bounded retries were reviewed/frozen in the new actor and no process was killed. Final card filesystems are unmounted and reader detached from WSL. Settings/partition provisioning intentionally changes bytes after initial image readback; no final whole-card/source-image equality claim is made. Physical boot, driver binding and serial/reset readiness remain pending. MCU remains4803457 bench;629c14b remains workstation-only. No pushes; battery follow-ups, Yocto6 and license work remain deferred, and owner build-env.md is preserved.


## October30 graph5b00a161 first controller boot — scoped pass

**New controller Linux first boot verified:** The physical controller now runs graph `5b00a161…` with Wi-Fi/DHCP and strict-key SSH at192.168.2.207. The permanent power driver is bound, GPIO19/21 are owned high, and effective loader/updater hashes match qualification. The sole UART1 bridge reports unchanged MCU4803457 bench;20 identical172-byte parameter replies match the saved factory record. Automatic MCU updates remain disabled. This establishes scoped first-boot/application-link acceptance, not STM32 reset/ROM or flashing readiness. [Boot receipt](/home/jresp/solo/.integration/controller-card-preflight-20261030T174903Z/boot-check-20261030T180422Z/qualification.json).

Boot IDaff41622-3d2d-41dc-b90f-69006b642a27 remains stable through the inspection. Linux6.6.52-lts-next-g1148f1b0d658 is in standby runlevel3; normal bridge PID587 is sole owner of/dev/ttymxc1. Initial identification reports HW3, expected UID and5.0.0-dev+g48034572145e-bench-noidle. Twenty read-only UDP5011 requests through that bridge return172 identical bytes, SHA25631fe40f57af93231272613617a8e6070c1083a5f80d424f05866f56b75ce0454; calibration endpoints/angular constraints pass and response times are19.51–20.46ms.

The built-in driver binds controller-power; debug GPIO reports19/21 as hold outputs high. All four installed loader/updater/diagnostic hashes match the frozen candidate. No discovered Artoo candidate, ENABLE_UPDATE or FORCE_UPDATE marker exists. The router confirms association/DHCP; on-card generated host identity passes its strict pinned key. Chrony runs with normal leap status; sampled ath9k reset counters are zero. These are short observations, not a network/time soak or electrical timing measurement.

No service stop, direct UART open, GPIO write, reset, reboot, ROM operation, parameter write or MCU programming occurred. The remaining pre-flash gate is a separately controlled reset/ROM read/release check using permanent holds; warm-reboot/final-power-off and journal/hardware acceptance remain open. Battery follow-ups stay deferred. Documentation is local; no pushes.


## October30 permanent BSP reset and ROM read — pass, no flashing

**Controlled STM32 reset/ROM preflight passed:** On Linux graph `5b00a161…`, the permanent BSP power holds kept the i.MX on the same boot through ROM entry and application release. Full 256 KiB flash readback, repeated reserved-page/vector reads, exact original firmware identity and 20 post-release parameter replies pass. Erase/write/Go/protection commands were blocked. MCU `4803457` remains installed; selected `629c14b` remains local and unflashed. [Result](artoo-permanent-rom-preflight-20261030.md).

One 100ms ROM-entry reset and one 100ms application-release reset completed with Linux boot ID unchanged and kernel-owned GPIO19/21 high. GET/GET-ID/1,078 READ-MEMORY commands read all256KiB and repeat both reserved pages/vectors; old application and full172-byte parameters match. Same bridge PID587 resumes, original UART mode and GPIO45/46 ownership are restored. Thirteen focused host checks passed after the source gate; no programming command, update marker, retry or push occurred. Local actor commit36a97f7 and private receipt `/home/jresp/solo/.integration/artoo-permanent-rom-20261030T181322Z` preserve evidence.

Settled UART framing/parity counts76/22 remain flat after transitions; normal traffic resumes. One ath9k Baseband Watchdog reset is newly observed since boot, stable on follow-up with SSH available; Wi-Fi soak remains open. These observations and remaining physical acceptance limits are recorded in the result. MCU629c14b is still unflashed. Owner build-env.md is preserved; battery work, Yocto6 and licenses remain deferred.
