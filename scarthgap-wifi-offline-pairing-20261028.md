# Offline Wi-Fi pairing contract — maintenance A

This is the new candidate's operator contract, not permission to apply it to devices. Source review, offline qualification and later explicit spare-media acceptance must complete before use. Existing live cards and source9 remain untouched. No controller/router connection or credentials are needed to prepare or qualify the source.

The controller AP and drone station are provisioned together, offline, with a unique `SoloLink_` SSID and an operator-chosen 8–63 printable ASCII passphrase. The SSID fits 32 bytes; leading/trailing whitespace is rejected. Use a strong unique passphrase and retain the mode-0600 recovery JSON privately. It contains the SSID and passphrase in plaintext; the station's hexadecimal companion comment is encoding, not encryption. Golden images contain no passphrase, station network, WPS PIN, private identity or authorization. A fresh or factory-reset controller cannot start its AP until provisioned.

The host helper is the same `sololink/config/opensolo_wifi.py` installed as `/usr/bin/opensolo_wifi.py`. It takes two distinct assembled offline roots, each with its image's `/etc/*.orig` templates available (or the canonical `/mnt/rootfs.ro/etc/*.orig` view), and a new operator recovery-file path outside both device roots. It refuses symlink paths. These roots are deliberately selected staging/merged roots; they are not an instruction to open or write any block device. Use the target owner when eventually preparing a real root: configuration must be root-owned mode0600 on target. In unprivileged disposable tests it must belong to the test user.

After the qualification gate, an operator can supply input without putting secrets in command arguments or shell history. For example, prepare the paths as variables in a local Python session, then:

```python
import getpass, json, subprocess
subprocess.run([
    'python3', helper, 'pair',
    '--controller-root', controller_root,
    '--drone-root', drone_root,
    '--recovery-file', recovery_file,
], input=json.dumps({'ssid': input('Pair SSID: '),
                    'password': getpass.getpass('Pair passphrase: ')}),
   text=True, check=True)
```

The recovery file is fsynced first, then both root configurations are written atomically with checksum markers and secret-free baselines. A power loss can leave only one half updated; keep both roots offline and rerun using the saved values with a **new** recovery-file path. Do not activate one half while the other still uses old credentials. Once reviewed/qualified, run `--root ROOT --kind hostapd ready` on the controller and `--root ROOT --kind wpa_supplicant ready` on the drone. These checks print no credentials. The explicit `get --key password` operator action prints the password by design and must not be captured in public logs or receipts.

The station stores a derived WPA PSK rather than interpolating printable passphrases into wpa_supplicant's quote parser. This supports literal braces, quotes and backslashes consistently with hostapd's raw passphrase format. Its private passphrase companion allows the existing explicit getter and SSID rotation to work. Both files and their backups/markers use mode0600. MD5 markers detect interrupted writes only; they are not authentication.

`configinit` now handles Wi-Fi through this helper before the separate legacy sololink/shotmanager merge. A valid personalized configuration survives template updates; an old password is preserved only if a valid prior baseline proves it differs from the factory value. Missing/corrupt/unknown baselines fail closed. Obsolete backups are cleared before changing baseline, preventing later corruption from resurrecting an old factory credential. Boot does not auto-change the AP SSID from its MAC, so an explicitly provisioned pair remains matched. Channel/country setters update config and MD5 coherently. A newly provisioned AP initializes its country once from the existing boot-partition `.reg` policy (US when absent); subsequent configured country settings survive boot. Confirm regional settings before later wireless activation.

Legacy `sololink_config` set/get SSID/password operations delegate to the same lifecycle and work when the assignment is absent. Arguments are no longer logged; use the helper's stdin interface for private automation. Changing either half alone can break association; normal operator rotation uses the paired operation. Reset and factory/settings reset intentionally remove the active Wi-Fi credential and obsolete backups. Recover by provisioning both halves from the private operator record or selecting a new pair. Restore any old software/card pair as a matched pair only; a one-sided mixed-fleet migration is not qualified.

WPS enrollment, including the former fixed PIN path, is disabled on both controller and drone. The old pair-button WPS action now reports that offline enrollment is required. Once Wi-Fi association exists, the normal Solo connection packet, controller confirmation screen, acceptance/rejection, pairing memory, version and lockout checks remain. Offline tests execute those actual event-dispatch branches; radio association and the physical confirmation screen require W9 hardware evidence.

WPA2 retains `TKIP CCMP` temporarily because required phone/bench client compatibility has not been measured. This is an explicit retirement hold, not a claim that TKIP or arbitrary clients are acceptable. CCMP-only requires paired radio/client evidence. AP/station concurrency, DHCP, country/channel, reconnect, phone clients, latency and recovery after interrupted deployment remain hardware acceptance gates. No full card backup or secret dump belongs in the public source/qualification receipts.
