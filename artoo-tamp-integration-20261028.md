# Artoo Tamp asset integration — October 28, 2026

The local production DVT image is **190,892 bytes**, saving **59,472 bytes**
(58.1 KiB) against the accepted correctness baseline `66336f0`. Application
flash headroom grows from 9,732 to **69,204 bytes**. Static RAM increases by
**312 bytes**, with no target heap allocation or framebuffer in the decoder.
Physical display timing remains unqualified. Nothing was flashed, pushed,
promoted into Yocto, or changed on a controller.

Source: `6eddd3338da30a53f510fe07b0d40ceed7ac2264`, version
`5.0.0-dev+g6eddd3338da3`. The source preserves the preceding correctness fixes,
172-byte parameter layout/reserved page, board profiles and wire protocol.
OpenSolo 5 is still local development branding, not an upstream release claim.

## Implementation and provenance

Portable Tamp C is vendored verbatim from
[`5279e7cf2322eb0a5f85acb9ca6de6d1b77561ea`](https://github.com/BrianPugh/tamp/tree/5279e7cf2322eb0a5f85acb9ca6de6d1b77561ea)
(the C implementation matches v2.5.0). `vendor/tamp/UPSTREAM.json` records
per-file SHA-256 values; MIT and Apache-2.0 license texts are preserved. Normal
asset generation verifies those hashes and compiles the host-only compressor
and desktop match helper into a temporary shared library. No Tamp pip package,
external checkout or network download is needed to reproduce a build. The
existing GCC 13.2 ARM/newlib/Tup/Python/Pillow toolchain remains unchanged.
Only `common.c` and `decompressor.c` link into the MCU image; post-link checks
reject compressor symbols.

The generator selects the smallest actual representation independently for
every image and glyph. Ties prefer ordinary RLE, then Tamp indices, then Tamp
RLE. Every representation includes its envelope cost:

| Tag | Body | Envelope before body |
| --- | --- | --- |
| 0 | Existing palette RLE, including its EOF | One-byte tag |
| 1 | Tamp-compressed palette indices | Tag + three-byte little-endian decoded byte count |
| 2 | Tamp-compressed palette RLE | Tag + three-byte little-endian decoded byte count |

Compressed bodies include Tamp's own standard stream header. Streams use an
independent 256-byte window, extended tokens, the smallest supported literal
width that fits the symbols (5–8 bits), default dictionary initialization and
host-only lazy matching. Custom dictionaries, dictionary resets/append,
extra headers and larger windows are rejected by the target adapter.

Existing descriptor sizes remain unchanged: 20-byte images, 8-byte glyphs on
ARM. The existing encoded-length field includes the envelope. Fonts share a
packed format identifier while each glyph chooses its own envelope tag.
Palette ordering, font bearings/geometry, RGB565 interpretation and the
transparency key remain unchanged. The existing font color conversion still
runs in the Gfx output callback. No PNG/JPEG decoder or display-driver rewrite
was introduced.

`AssetDecoder::decode` owns one static workspace: 256-byte history, 32-byte
output chunk, Tamp state and a busy flag, totaling 312 bytes on this target.
It rejects interrupt-context and recursive calls, resets state for every
compressed asset, and streams directly to the existing foreground pixel sink.
Its boolean result is true only when source, decoded-byte and pixel budgets,
palette bounds, RLE EOF and canonical zero tail padding all pass. Indexed
output must equal the image's pixel count; decoded RLE is capped at
`2 × pixels + 1`. Input and pixel quotas plus a progress bound prevent an
arbitrary three-byte declaration from creating unbounded work.

Bounds are not an integrity signature. Some bit changes can still describe a
different valid image. A malformed stream may draw a validated prefix before
failure; there is no atomic replacement or full framebuffer. Full pixel
clipping and physical display/control latency remain separate acceptance gates.

## Measured linked result

| Profile | Baseline binary | Tamp binary | Flash saved | Flash headroom |
| --- | ---: | ---: | ---: | ---: |
| Production DVT | 250,364 | 190,892 | 59,472 | 69,204 |
| Production BB03 | 250,300 | 190,828 | 59,472 | 69,268 |
| Factory DVT | 251,764 | 192,300 | 59,464 | 67,796 |
| Factory BB03 | 251,700 | 192,236 | 59,464 | 67,860 |

All four profiles use 6,840 static RAM bytes, up from 6,528. Bounded heap
capacity is 38,216 bytes below the unchanged minimum 4,096-byte stack reserve.
The largest individual compiled frame remains 288 bytes. Relevant individual
frames are 120 bytes for AssetDecoder, 112 for Tamp decompression, 72 for the
Gfx bitmap wrapper and 24 for its pixel callback. These are not a complete
call-chain/interrupt nesting proof or physical high-water measurement.

The 758 linked image/glyph payloads fall from 155,231 to 92,943 bytes,
including all envelope bytes. The same 1,542 palette bytes remain unchanged.
The linked selection is 694 Tamp-index, 28 Tamp-RLE and 36 ordinary-RLE
assets. Actual total-image savings include all new decoder/glue code and
alignment, unlike an archive ratio or isolated decoder projection. The
[earlier benchmark](artoo-compression-20261028.md) remains historical evidence;
this report supplies the actual firmware link delta.

[Production DVT receipt](/home/jresp/solo/.integration/artoo-builds/tamp-integration-20261028/production-dvt/verification.json):
BIN SHA-256 `c423a8c0793e2acec18b94e5eb7fd1f414b258a977008433110dd265b2f526c9`;
ELF SHA-256 `dee36037bbbdc380e53d0754f0fd944ebbe6291a54a4ea1cf684d16377ea8743`;
initial SP `0x2000c000`, reset vector `0x08006ec5`, 11 constructors.

## Software qualification

Source was frozen and reviewed before execution. The initial `4f60c85`
vendoring omitted the compressor's included desktop match helper; the first
host build stopped before runtime tests. The exact pinned file and hash entry
were added in `6eddd33`. The failure log remains in `native-attempt1`; no vendor
algorithm or firmware behavior was changed by this packaging correction.

All eight native suites passed on the final source. The graphics suite uses
the immutable pre-compression generator from `66336f0` and independently
decodes its RLE to golden RGB565 pixels. It checks **all 800 generated assets
and 405,717 pixels** through the actual mixed decoder and actual Gfx sink,
including tinted glyphs and transparency. It also verifies descriptor/palette
equality, repeated-generation determinism, 1/7/32-byte chunks, stream truncation,
invalid headers/lengths, extra tail bytes, output quotas, ISR/reentry rejection
and 100,000 deterministic malformed streams under ASan/UBSan. Valid-but-changed
streams are checked for boundedness, not falsely required to fail a nonexistent
checksum. Existing runtime, concurrency, flash/parameter/command reliability,
actual Linux SLIP transport, CRC, board-backlight and filter suites also passed.

All four cross-builds passed with explicit compiler board/profile arguments.
Checks verify decoder-only target C, constructor/vector/data/BSS/heap/parameter
bounds, retained ABI attributes, factory handler presence/absence, stack files,
all 765 compiled generated arrays matching generator output, unchanged palettes
and unchanged linked descriptor sizes. Existing libnosys warnings remain.

Receipts, maps, ELF/BIN/HEX files, generated sources, build commands, stack
reports and logs are preserved under
`/home/jresp/solo/.integration/artoo-builds/tamp-integration-20261028`.
The accepted `66336f0` receipts and earlier compression experiment are untouched.
Canonical Renode qualification completed against this exact saved image as recorded below.

The user-owned `build-env.md` remains unchanged at SHA-256
`b32bd9c481fab728573c3c65cb0928780d6c88a8b68474e702ae3fe158c577db`.

## Canonical Renode qualification

[Final Renode result](/home/jresp/solo/.integration/artoo-builds/renode-tamp-6eddd33-qualification-r2-20261028/result.json)
passed all 32 checks against the saved `6eddd33` production DVT BIN/ELF:
three positive boot scenarios, two expected-failure controls, 26 UART cases and
one compressed-render scenario. The recorded inputs did not change during the
run. Renode is `1.17.0+20260907gitf1dd1b4af`.

The render observer follows the real splash path: bitmap wrapper → packed
asset tag 1 → actual portable C Tamp stream function → decoder boolean success
1 → splash continuation and main-loop execution. The observed splash stream
is 2,038 bytes including its four-byte envelope, decoding 10,176 palette indices
for the unchanged 212×48 logo. The 1.25-second software scenario observes 62
SysTick rollovers, 1.24 seconds accumulated at the intended 50 Hz cadence and
9 MHz reference. The ordinary half-second task scenarios observe 24 rollovers.

Startup still verifies poisoned-RAM data/BSS handling and all 11 constructors.
Boot/Running task dispatch and the 26 existing UART cases remain passing.
The harness retains 99 power-wait, 101 Running, 1,407 UART and 119 render
warnings documenting peripheral-model gaps. Its RCC shim and incomplete
ADC/flash/DMA/LCD/power models do not establish physical timing or display
appearance. Exact pixel/color/transparency evidence comes from the native
actual-Gfx golden tests; Renode adds execution of the real ARM compressed-render
path without modifying firmware code or building a full controller emulator.

Local software qualification is complete. Physical display responsiveness and
the existing hardware/design gates in the [correctness ledger](artoo-modernization-resolution-20261028.md)
remain explicit; firmware promotion and hardware programming are separate work.
