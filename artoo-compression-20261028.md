# Artoo asset compression experiment — October 28, 2026

This is the historical codec comparison. The subsequent [Tamp implementation](artoo-tamp-integration-20261028.md) records the integrated firmware and its qualification.

**Tamp extended with a 256-byte window leads the measured comparison.**
Selecting a format independently for each asset projects **59,800 bytes
(58.4 KiB) of net flash savings**, including an allowance for stream metadata
and an isolated Cortex-M3 decoder/wrapper. Compression has not been integrated
into Artoo or the Yocto payload. Physical redraw timing remains unmeasured.

## Frozen inputs

An isolated export of Artoo `3e0c6ffa7162249e311950a568f40b9d531a12a2`
(accepted transport wave1) was regenerated with pinned Pillow 10.2.0. All
765 linked generated arrays match the saved ELF/BIN byte for byte. Ongoing
firmware modernization is outside this experiment.

| Corpus | Images | Glyphs | Existing RLE | Palettes | Raw pixel indices |
| --- | ---: | ---: | ---: | ---: | ---: |
| All generated | 123 | 677 | 173,775 B | 1,672 B | 405,717 B |
| Linked in firmware | 91 | 667 | 155,231 B | 1,542 B | 351,784 B |

Savings below use linked assets only. Each image/glyph starts a fresh
dictionary and remains independently drawable. PNG/source/archive compression
is not substituted for the firmware's actual byte representations.

Upstream checkouts, using native WSL Git:

- [Heatshrink](https://github.com/atomicobject/heatshrink),
  `/home/jresp/Active_GH/heatshrink`, revision
  `7d419e1fa4830d0b919b9b6a91fe2fb786cf3280` (`v0.4.1-1`).
- [Portable Tamp](https://github.com/BrianPugh/tamp),
  `/home/jresp/Active_GH/tamp`, revision
  `5279e7cf2322eb0a5f85acb9ca6de6d1b77561ea` (`v2.5.0-8`).
  Its C sources match release v2.5.0. Both classic and extended formats tested.
- The suggested [esp-tamp](https://github.com/BitsForPeople/esp-tamp) is an
  ESP32-optimized fork. This experiment uses portable upstream C, with ESP
  paths disabled. Neither upstream checkout was modified.

## Method and results

Thirty configurations cover 256-byte through 4 KiB windows: twenty heatshrink
window/lookahead pairs and five windows each for Tamp classic/extended.
Both existing RLE and decoded palette-index streams are compressed. Tamp uses
the smallest supported literal width that fits the bytes (5–8 bits), with
host-only lazy matching. No custom/shared dictionary is used.

The mixed strategy keeps ordinary RLE or chooses the smaller compressed
RLE/indexed stream for each asset. **Four extra bytes per compressed asset**
allow for bounds/length metadata. Palettes remain unchanged. The production
descriptors and renderer have not been modified; this remains a projection.

| Candidate | Assets + palettes + stream allowance | ARM decoder probe flash | Projected net flash saving | Static probe RAM, plus stack |
| --- | ---: | ---: | ---: | ---: |
| Current RLE | 156,773 B | Existing renderer retained | — | Existing renderer |
| Heatshrink W8/L5 | 104,667 B | 1,184 B | 50,922 B / 49.7 KiB | 338 B |
| Tamp classic W8 | 103,488 B | 1,588 B | 51,697 B / 50.5 KiB | 308 B |
| **Tamp extended W8** | **94,449 B** | **2,524 B** | **59,800 B / 58.4 KiB** | **312 B** |

ARM probes use GCC 13.2, Cortex-M3 Thumb, soft float, `-Os`, section garbage
collection, nano libc and no LTO. Their flash includes core, streaming wrapper,
checksum sink, small entry/sample and retained libc helpers. These are standalone
footprints, not a firmware link-map delta. Integration glue, descriptor layout
and alignment may change the actual saving. Existing RLE support is retained;
no saving from removing its code is assumed.

Static probe RAM includes a 32-byte output chunk and a four-byte checksum.
Heatshrink's 302-byte state includes its 256-byte history and 32-byte input
buffer. Tamp classic uses 16 bytes of state plus 256 bytes history; extended
uses 20 bytes plus history. No heap allocation is linked. Largest individual
compiled stack frames are 48 B heatshrink, 104 B classic and 112 B extended;
these are **not** complete peak call-chain/interrupt-stack measurements.

Tamp extended compresses 719 of 758 linked assets: 690 from indices and 29
from RLE; 39 retain ordinary RLE. Both images and fonts shrink. Larger windows
were worse on this corpus of small independent assets.

The 212 × 48 OpenSolo 5 startup logo is 3,021 B in current RLE, 2,034 B as
Tamp extended indexed and 2,160 B as heatshrink W8/L5 indexed, before the
four-byte allowance. Its RGB565 colors and geometry remain identical.

For the complete generated inventory, including linker-discarded assets,
mixed totals with palettes/allowances are 115,725 B heatshrink, 114,553 B
classic and 104,998 B extended. These inventory totals are not used to claim
the linked firmware savings above.

## Checks and limitations

- **48,000** independent compression/decompression cases passed: 800 assets,
  30 configurations, two byte representations.
- Those streams passed **192,000 byte-exact ASan/UBSan checks** with
  1-, 7-, 32- and 128-byte output chunks, including empty glyphs.
- Three Cortex-M3 QEMU binaries each passed **all 800 assets**, checking final
  RGB565 CRCs, pixel counts, palette bounds and RLE termination across chunks.
  They used `lm3s6965evb` and a test linker limited to 48 KiB SRAM. These
  are decoder algorithm tests, not Artoo boot or peripheral qualification.
- Native WSL static-decoder medians were approximately 1.67 ms heatshrink,
  1.17 ms classic and 1.30 ms extended per selected linked compressed-asset
  pass. These include initialization/Python-call overhead, have differing
  input/output work and exclude second-stage RLE and LCD writes. They do
  **not** establish STM32 redraw speed or end-to-end relative performance.

Prefer a separate Tamp extended W8 prototype after the correctness changes
settle: host compression, static C decoder, bounded streaming to the existing
palette/LCD path and ordinary RLE where smaller. No full framebuffer is needed.
Before acceptance, measure actual firmware size, descriptors/alignment, full
stack usage, foreground draw budget and physical responsiveness. Add truncation,
invalid-header, expansion-bound, reentrancy and exact font-color tests.

## Local evidence

All scripts, frozen inputs, compressed streams and probe ELFs are under
`/home/jresp/solo/.integration/artoo-builds/compression-20261028`:

- [Corpus/hashes](/home/jresp/solo/.integration/artoo-builds/compression-20261028/corpus.json)
- [Configuration totals](/home/jresp/solo/.integration/artoo-builds/compression-20261028/totals.csv)
- [Every asset/configuration](/home/jresp/solo/.integration/artoo-builds/compression-20261028/per-asset.csv)
- [ARM footprint/host timing](/home/jresp/solo/.integration/artoo-builds/compression-20261028/footprint.json)
- [Sanitizer result](/home/jresp/solo/.integration/artoo-builds/compression-20261028/sanitizer.log)
- [Cortex-M3 RGB565 results](/home/jresp/solo/.integration/artoo-builds/compression-20261028/qemu.json)

Reproduction scripts there are `compare.py`, `footprint.py`, `qemu_verify.py`,
in that order, using native WSL Python/GCC, ARM GCC/QEMU and pinned codec
checkouts. They regenerate experiment evidence, not firmware source or accepted
MCU artifacts. Corpus regeneration additionally uses the Artoo virtual
environment's pinned Pillow. Reviewed phase manifests retain source hashes.

No controller/SD changes, UART access, flashing, firmware integration, Yocto
payload replacement or remote publication occurred.
