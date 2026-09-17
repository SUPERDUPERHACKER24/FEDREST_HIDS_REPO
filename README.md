# FedREST-HIDS

[![DOI](https://zenodo.org/badge/1373028984.svg)](https://doi.org/10.5281/zenodo.22804616)

Code accompanying **"FedREST-HIDS: Split-Federated Intrusion Detection for
Resource-Heterogeneous Industrial IoT"** (Vyas, Hwang & Lin), submitted to
*Computer Communications*.

FedREST-HIDS partitions intrusion detection across three tiers. Resource-rich
devices train complete local models and take part in federated learning.
Resource-constrained devices execute only the initial feature-extraction layer
and send clipped, noised activations to an edge server that runs the remaining
layers. Federated transfer learning then aligns the edge models globally, and a
trust-aware aggregation rule bounds the influence of any single client. Privacy
is enforced locally under Rényi differential privacy.

---

## Verify the article's numbers in one second

Every privacy and communication figure printed in the article is checked
against this implementation. No dataset, no GPU.

```bash
python scripts/verify_against_paper.py
```

```
Privacy accounting -- Section 3.2.2, Section 5.2, Table 4
  [PASS] sigma, WUSTL headline (C=1, a=5, eps=0.5)      got 2.2361   article 2.2360
  [PASS] eps_DP at delta=1e-5, alpha=5                  got 27.8803  article 27.8800
  ...
24/24 checks passed
```

---

## Layout

```
fedrest-hids/
├── src/fedrest/          reference implementation
│   ├── privacy.py        Rényi DP: noise scale, composition, RDP→DP, allocation
│   ├── fedtrust.py       trust scoring and softmax-weighted aggregation
│   ├── comms.py          communication cost model (Section 6.5)
│   ├── models.py         CNN-HIDS, client-side and edge-side split models
│   ├── training.py       Algorithms 1–4
│   ├── legacy.py         the alternative noise expression used in some notebooks
│   └── poisoning.py      label-flip and sign-flip attacks
├── notebooks/            the 16 notebooks behind the reported results
│   └── MANIFEST.md       which notebook produced which table or figure
├── scripts/
│   ├── verify_against_paper.py   24 checks against the article
│   ├── audit_configs.py          each notebook's config vs the article
│   └── make_figure2.py           regenerates Figure 2 from the training log
├── results/              verification output, regenerated figure
└── docs/KNOWN_ISSUES.md  where code and article agree, and where they don't
```

The notebooks are Colab exports preserved **as run**. Refactoring them would
break their correspondence with the article, so `src/fedrest/` holds a cleaned
reference version of the mechanisms they each reimplement.

> Read [`docs/KNOWN_ISSUES.md`](docs/KNOWN_ISSUES.md) before using this code.

---

## Install

```bash
git clone https://github.com/<user>/fedrest-hids.git
cd fedrest-hids
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Python 3.10+. Both PyTorch and TensorFlow are listed because twelve notebooks
use the former and three the latter.

---

## Datasets

Not redistributed. Obtain them from the original sources:

| Dataset | Source |
|---|---|
| WUSTL-IIoT-2021 | https://www.cse.wustl.edu/~jain/iiot2/index.html |
| ToN-IoT | https://research.unsw.edu.au/projects/toniot-datasets |
| SCVIC-APT-2021 | https://ieee-dataport.org/documents/scvic-apt-2021 |

Place the CSVs under `data/` (git-ignored) and adjust the path at the top of
whichever notebook you run — see [`docs/DATA_PATHS.md`](docs/DATA_PATHS.md).
The notebooks were written for Colab and mount Google Drive; those cells need
editing for local use.

---

## Using the library

```python
from fedrest import LocalRenyiDP, FedTrust, CommsProfile, build_split_pair

dp = LocalRenyiDP(clip_norm=1.0, alpha=5.0, epsilon=0.5)
noised = dp.privatize(gradient)
print(dp.report())
#  rounds=1  alpha=5.0  eps_per_round=0.5  sigma=2.2361 ...

client, edge = build_split_pair(n_features=41, n_classes=2)
print(CommsProfile(n_samples=28_000,
                   activation_dim=client.activation_dim,
                   n_parameters=100_000).summary())
```

---

## Reproducibility

Stated plainly, so nobody is misled:

- **The federation is simulated.** Clients run sequentially in one process. No
  wall-clock latency or energy figure is reported anywhere.
- **Headline results are single runs at seed 42.** Tables 6, 8 and 9 report
  three-seed means with standard deviations; the headline numbers do not.
- **Exact reproduction needs the original Colab environment.** GPU model,
  library versions and CUDA version affect the third decimal place.
- **The noise-scale expression differs between notebooks.** See KNOWN_ISSUES §2.

---

## Citing

Please cite both the software and the article.

```bibtex
@software{fedresthids_code,
  author    = {Vyas, Abhishek and Hwang, Ren-Hung and Lin, Po-Ching},
  title     = {{FedREST-HIDS}: split-federated intrusion detection for
               resource-heterogeneous {IIoT}},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22804616},
  url       = {https://doi.org/10.5281/zenodo.22804616}
}
```

`10.5281/zenodo.22804616` is the concept DOI and always resolves to the newest
release. Cite it rather than a version DOI unless you need to pin a specific
release. See also `CITATION.cff`.

## Licence

MIT — see `LICENSE`. The datasets carry their own licences.
