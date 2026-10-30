# OpenSolo publication checkpoint — October 30

The user authorized committing and pushing the coordinated maintenance and Artoo work after successful firmware installation and LCD confirmation. The branches below identify this checkpoint; default branches are not being merged. The private hardware backups and generated build artifacts stay outside Git.

| Repository | Branch | Selected source revision |
|---|---|---|
| [artoo-firmware](https://github.com/Decryptortuning/artoo-firmware/tree/codex/artoo-wsl) | `codex/artoo-wsl` | `629c14b71ce6748272b9bd711d12e875b6c3cc9f` |
| [stm32loader](https://github.com/Decryptortuning/stm32loader/tree/PORT) | `PORT` | `327dac386743533e11eee5509e7b54042a8c39bd` |
| [shotmanager](https://github.com/Decryptortuning/shotmanager/tree/PORT) | `PORT` | `2b2e067c875f45a7216f77c51797d470b4566e75` |
| [sololink](https://github.com/Decryptortuning/sololink/tree/codex/controller-linux-maintenance) | `codex/controller-linux-maintenance` | `3d9357434754452cc87f22152d23221be4f4c049` |
| [meta-solo-bsp](https://github.com/Decryptortuning/meta-solo-bsp/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | `281976648cf778b27151f4a3d330c52beea652cb` |
| [meta-3dr](https://github.com/Decryptortuning/meta-3dr/tree/codex/controller-linux-maintenance) | `codex/controller-linux-maintenance` | `7cbe9d6fa423ffdd7443db23e6c168e652f9b7f5` |
| [OPenSolo_2](https://github.com/Decryptortuning/OPenSolo_2/tree/codex/controller-linux-maintenance) | `codex/controller-linux-maintenance` | `cf710b8de4708b8babecadad3a018adf0c689afe` |
| [docs](https://github.com/Decryptortuning/docs/tree/codex/scarthgap-maintenance-a) | `codex/scarthgap-maintenance-a` | This checkpoint document and preceding installation records |

This set includes the retained maintenance/MAVLink work, permanent controller power lifecycle, Python-compatible journal loader/updater, Artoo reliability changes and manual reset/update helpers. The ShotManager Python3 division fix is included because the integration lock and image consume it. SoloLink-Python is already published at the pinned revision. Vendor repositories and inactive inventory repositories are unchanged.

The active Linux graph remains `5b00a1619b3bc97ffbd98b978583bbf17d9024cf53c2919fe7e62ed6f68bad10`. Physical Artoo runs `5.0.0-dev+g629c14b71ce6-bench-noidle`, with verified full readback, unchanged reserved pages/parameters and normal LCD “Waiting for Solo.” Publication does not rebuild, restage or flash anything. The older firmware artifact in the Yocto lock remains deliberately quarantined; the manually installed629 artifact has its own receipt and is not promoted into automatic selection.

Existing [software qualification](artoo-recovery-result-20261030.md), [permanent-hold ROM check](artoo-permanent-rom-preflight-20261030.md) and [installation result](artoo-journal-flash-result-20261030.md) retain their actual scope. No source changes invalidate those results. Journal power-loss behavior, experimental watchdog hardware timing, loaded pairing/flight and longer soak remain open. Battery/controller follow-ups, Yocto6 and license work remain deferred.

Publication inventory, source gate, push output and remote-head confirmation are recorded under `/home/jresp/solo/.integration/publication-20261030T183043Z`. The source gate pins the final docs commit separately, avoiding a self-referential hash. The owner’s unrelated `build-env.md` edits are excluded and preserved. Earlier statements that work was local describe their dated execution checkpoints.
