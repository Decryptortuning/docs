# Controller Wi-Fi PCIe fix — October 28, 2026

The permanent controller kernel fix is installed and has passed two warm
reboots with automatic LAN association, DHCP and authenticated SSH. No runtime
PCI-register adjustment is required. Cold power-cycle and long-duration soak
qualification remain separate hardware acceptance steps.

## Cause and scope

The modern NXP kernel enabled MSI on the i.MX6 PCIe root port while ath9k's
normal path used legacy interrupts. With that combination the radio received
zero legacy interrupts. Forcing endpoint MSI restored partial connectivity but
produced multi-second packet delays and TX watchdog resets. Disabling MSI on
both host and endpoint restored working legacy interrupt delivery. The old
3DR Artoo Linux configuration disabled PCI_MSI, which the modern port missed.
This establishes the configuration regression; it does not fully trace the
internal reason MSI-mode completions stalled. “TX Path Hang” was a software
watchdog observation, not independent proof of failed radio hardware.

BSP `843c4b23ba9b30b5e0acb5d52dc025dd4ab582cb` includes the controller-only
`pci=nomsi` built-in command line with `CONFIG_CMDLINE_EXTEND=y`. Bootloader
console, root partition, nosmp and other arguments are preserved. It also aligns
`KERNEL_LOCALVERSION=-lts-next` with NXP's actual kernel build so staged headers,
external modules and kernel metadata agree. Integration pin: `7f96005`.
Drone kernel policy is unchanged. Controller ath9k uses no separately loaded
radio firmware blob; this change does not replace EEPROM calibration.

## Build and compatibility validation

All 6,788 Yocto tasks completed successfully. SD-card packaging, update payloads,
initramfs/source identity, ARM Python 3.12/networking smoke checks and maintenance
archive checks passed. The host emits Ubuntu 24.04 validation and WSL process-I/O
accounting warnings; these did not fail the build.

Kernel-only deployment was gated by all 14,136 exported symbols remaining
byte-identical, matching full vermagic for 333 in-tree loadable modules and all
334 rootfs modules including external cryptodev, matching main/shared release
headers, verified uImage CRCs and unchanged device-tree bytes. The semantic
kernel configuration delta is exactly the three CMDLINE settings. Kernel release
remains `6.6.52-lts-next-ge0f9e2afd4cf`; uname alone does not identify this fix.

Selected payload is the **bundled initramfs kernel**, not the bare uImage:

```
uImage-initramfs--6.6.52+git0+e0f9e2afd4-r0-imx_controller-20261028231641.bin
26,582,592 bytes
SHA256 57e6f86dcfb1aa9b1c2d8c34dc9409211f5aa688b0dd4e1a37efef5238b39fad
```

[Validation receipts](/home/jresp/solo/.integration/controller-pci-nomsi-20261028)
contain build logs, image/initramfs/userspace/maintenance results and the
independent kernel-ABI review.

## Actual installation and reboot results

Only `/mnt/boot/uImage` on LATEST (`/dev/mmcblk0p2`) was replaced. The installer
checked the old and staged hashes, exact mount and free space, copied a `.new`
file, verified it, synced, renamed it, synced and verified the installed bytes.
The FAT boot filesystem is read-only again. The mounted squashfs, U-Boot and
DTB were not replaced. The old qualified kernel remains preserved on the host
in the evidence directory's `before/uImage`; no full-card backup was taken.
The complete rebuilt rootfs remains local and was not installed over a mounted
squashfs. Existing rootfs/overlay Python and networking fixes remain in use.

First warm boot passed with software crypto retained as a controlled variable.
The temporary ath9k diagnostic modprobe override was then removed. Second warm
boot passed with normal hardware crypto (`nohwcrypt=0`, `use_msi=0`). On both:

- Actual command line contains `pci=nomsi` alongside the original root/console.
- Root-port and endpoint MSI Enable bits are clear; endpoint INTx-disable is clear.
- GPC 123 legacy ath9k interrupt counts increase (Linux IRQ 294 on these boots).
- Eight byte-exact SSH transfer checks passed: 32 KiB, 256 KiB, 1 MiB and 10 MiB
  uploaded and downloaded on each boot, 16 checks total.
- All ath9k reset counters are zero, including TX Path Hang and transmit timeout.
  No module-version mismatch or unknown-symbol errors were observed.

With default hardware crypto, 10 MiB took 2.487 s up and 1.673 s down including
SSH startup. These are bounded acceptance measurements, not a throughput benchmark.
Raw before/after PCI state, IRQs, driver counters and kernel logs are retained in
`reboot-software-crypto.json` and `reboot-hardware-crypto.json` in the receipts.

The controller is reachable using the pinned `opensolo-controller` SSH identity
at `192.168.2.207`. Keys and private Wi-Fi settings were preserved. The controller
still reports Artoo `OpenSolo-4dev`; no STM32 programming marker exists. The
preserved 172-byte calibration backup hash is unchanged. The unsafe updater-UI
maintenance hold is removed; normal idle timeout remains until a separately
selected bench Artoo firmware is actually flashed.

## Artoo local qualification

The separate frozen Artoo source `4803457` built all six board/profile
combinations. Native regressions and both normal/bench DVT Renode suites passed
(32 checks each). The idle-only bench image and production image remain local;
neither was flashed or added to automatic update discovery. See
[the Artoo bench qualification](artoo-bench-idle-20261028.md).
