# Controller permanent power holds and updater startup verification — October 29

The permanent fix is implemented locally in the owned BSP and SoloLink repositories. Controller U-Boot asserts the two legacy power holds before Linux; a built-in Linux driver retains them through MCU reset and warm reboot, then releases them at final power-off. Updater startup verification now tolerates noisy/partial/oversized frames within a bounded receive window, and programming is refused unless the power driver is bound. No flashing, device deployment, SSH modification or push occurred in this implementation session.

The physical controller remains on its previously verified be98 Linux plus Chrony overlay and OpenSolo5 bench MCU image. These local Linux/BSP changes are not installed. Their cold boot, reset behavior, warm reboot and final electrical shutdown still require later hardware acceptance.

## Power ownership and shutdown

The controller defconfig explicitly enables HDTC_CONTROLLER_POWER_HOLD; the drone does not. The existing early-board callback runs before normal driver-model GPIO ownership, so it uses the SoC register definitions directly. It latches GPIO1_IO19/21 high before enabling their output directions and selecting the GPIO mux. It preserves unrelated bank bits and restores GPIO2_IO13/14 BOOT0/reset to inputs. Actual ARM disassembly confirms the280000hex hold mask is written to data before direction,6000hex is cleared from GPIO2 direction, and mux selection follows.

Linux's controller-only opensolo,controller-power-hold node describes two active-high hold GPIOs. The new built-in opensolo-controller-power driver requests both high and owns them for the boot lifetime. It registers only SYS_OFF_MODE_POWER_OFF, using non-sleeping SoC GPIOs. There is no device-shutdown or restart callback that would inadvertently release power during a warm reboot. Userspace unbind is suppressed. At final power-off, both pins become inputs, matching the state already tested on this board and allowing the STM32 to withdraw its own PC14/PC15 enables.

The actual image halt script invokes halt -d -f -p after normal rc0 shutdown; -p selects the final kernel power-off path. The MCU shutdown-request consumer invokes shutdown -h now, which reaches that script. A separate reboot script selects reboot. This puts power release after the ordinary userspace/filesystem shutdown sequence rather than in an early init-script stop action. The new driver does not independently command the STM32 to shut down or establish the physical rail timing.

The kernel addition is an owned source file, local binding, config fragment and Kconfig/Makefile patch in meta-solo-bsp. The fragment/patch/driver/node are controller-only. Vendor checkouts and generated work directories were not edited manually. Kernel ownership gives the updater a concrete readiness check and avoids a permanently asserted GPIO hog with no shutdown lifecycle.

## Startup verification and programming boundary

The boot updater clears stale receive data once and queries SysInfo periodically while consuming other packets for up to eight seconds. Its short serial reads have an explicit write timeout and cooperative exclusivity. SLIP sync/frame parsing accepts an optional monotonic deadline, so continuous traffic without a delimiter cannot evade the receive budget. Oversized frames drop synchronization and permit the next complete frame to be read. A reply needs the expected packet type and a nonempty printable version; trailing NUL and the conventional leading v are normalized while build suffixes remain intact.

The standalone temporary-flash actor also resynchronizes after oversized startup fragments within its existing deadline. It is not executed on hardware here. The exact bytes behind the earlier immediate startup-query failure were not captured; these changes provide bounded recovery without claiming a new electrical/baud diagnosis.

Before programming, the production updater requires the bound opensolo-controller-power platform driver. An old or partially updated kernel therefore cannot silently program without the permanent holds. A loader failure now produces one failed attempt, with no repeat programming in that invocation or unprotect escalation. Existing calibration backup/validation, page127 preservation, exact post-update version comparison, failure marking and candidate retention remain in force. Default automatic-update enable policy is unchanged.

Deploy the combined U-Boot/kernel/device-tree/userspace candidate when later authorized; copying only the Python updater does not provide the BSP fix. No automatic firmware candidate was promoted. The existing staged-only MCU artifact inventory is unchanged; the separately installed4803457 bench image has its own prior installation receipt.

## Qualification and source lineage

Final candidate graph is7bac1220def2ca664d584961eaf08ff6cac409e1c2601f4688e7a586db468a8e. BSP2819766, SoloLink045b213, product2d18eeb and integration72731fa are local commits. The recipe pin and integration lock select the combined source; the reviewed SoloLink commit was explicitly seeded into the local BitBake mirror without publication.

Host checks pass108 integration and41 SoloLink tests, including native execution of the owned early-boot/power-off C against mocked GPIO/registers,24 updater contracts and8 standalone-flash guards. The C tests cover controller/drone separation, register ordering, unrelated bits, high handoff, final-power-off registration, release of both pins and rejection of sleeping/incorrect GPIO arrays. Parser tests include startup fragments, oversized frames, continuous unframed traffic, many unrelated packets, a real deadline and no automatic programming retry. These mocks do not establish electrical hardware acceptance.

The first native Linux stub lacked stdbool.h; the stub was fixed and refrozen. The first controller DTB build rejected the new child node preceding root properties; it was moved below them, repinned and rebuilt. The old in-flight drone build was stopped at a task boundary and excluded from final qualification. All failed/superseded evidence remains alongside the successful receipts. The final source change only reordered that DT node; unchanged Python/parser tests retain their recorded pass, with focused C checks rerun after the correction.

All four builds and16 baseline checks pass. Each profile passes image/payload/identity validation, embedded-initramfs/source checks, actual ARM Python/SSH/DNSMasq checks and ARM maintenance regular-file checks. The installed ARM updater recovers from a5000-byte startup fragment and40 unrelated packets before the version reply; deadline handling is exercised without device access.

| Profile | Build | Four baseline checks | SD SHA256 |
|---|---|---|---|
| controller-development | Pass | Pass | `8441eac23632ea8f1084e2384d3af20a963445d2f74d59038b256bf6348fa82d` |
| controller-release | Pass | Pass | `5fe46f5383462c09048c219786f576500b8d259bb942608e4ad8eb7815fb26e4` |
| drone-development | Pass | Pass | `09fa70a1a3578e189b5312c0dabb71eeaaee1925f8f7d07438e527d38cd07e70` |
| drone-release | Pass | Pass | `962702e5996fe56e04a914aafa67042daa299cfd3dc9dba583a3b474c8a02f95` |

All four installed updater/SLIP files match the pinned source bytes. Both controller kernels link the built-in power driver, and their deployed DTBs identify exactly GPIO1 pins19/21 as active-high holds. Both drone kernels exclude the driver and power node. Development/release U-Boot and DTB bytes agree within each machine. The fresh images retain staged-only MCU artifacts and existing automatic-update disable policy.


[Final receipt](/home/jresp/solo/.integration/controller-power-fix-20261029T225936Z/qualification-result.json) and [32 preserved artifacts](/home/jresp/solo/.integration/controller-power-fix-20261029T225936Z/artifacts) identify this candidate. It retains source freezes, lock copies, build/check logs, compiled-controller/drone checks, U-Boot disassembly, shutdown-path review and four owned-source Git bundles. Four owned-source bundles are verified; this is not a new complete vendor/download transport or offline bootstrap. This is an incremental cached qualification, not a clean/offline recovery, CVE clearance or hardware acceptance. Existing security/watchdog/media follow-ups, Yocto6 and license work remain deferred; the owner's build-env.md and earlier receipts are preserved.
