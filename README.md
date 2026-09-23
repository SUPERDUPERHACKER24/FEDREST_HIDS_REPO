# FedREST-HIDS

Notebooks and results accompanying **"FedREST-HIDS: Split-Federated Intrusion
Detection for Resource-Heterogeneous Industrial IoT"** (Vyas, Hwang & Lin),
submitted to *Computer Communications*.

FedREST-HIDS partitions intrusion detection across three tiers. Resource-rich
(RR) devices train complete local models and take part in federated learning.
Resource-constrained (RC) devices execute only the initial feature-extraction
layer and transmit clipped, noised activations to an edge server that runs the
remaining layers. Federated transfer learning aligns the edge models globally,
and a trust-aware aggregation rule (FedTrust) bounds the influence of any single
client. Privacy is enforced locally under Rényi differential privacy, with the
per-round budget converted once to an (ε, δ) guarantee.

---

## Start here

**Read [`docs/KNOWN_ISSUES.md`](docs/KNOWN_ISSUES.md) first.** It records a
label-leakage defect found in the preprocessing before submission, what was
regenerated after fixing it, and — equally important — what was not.

[`notebooks/MANIFEST.md`](notebooks/MANIFEST.md) maps every table and figure in
the article to the notebook that produced it.

---

## Layout

```
notebooks/   one notebook per experiment, plus MANIFEST.md
results/     raw JSON output of the runs behind the regenerated results
figures/     Figures 2, 3 and 4 as they appear in the article
docs/        KNOWN_ISSUES.md, DATA_PATHS.md
scripts/     verify_results.py -- checks results/ against the article's tables
```

## Reproducing the regenerated results

Three notebooks produce every WUSTL-IIoT-2021 and ToN-IoT network result in the
article. Each runs on a single Colab T4.

| Notebook | Produces | Time |
|---|---|---|
| `TABLE3_HEADLINE_3H_NOLEAK.ipynb` | Tables 4, 5; Figure 2; §6.1 | ~1.5 h |
| `TABLE8_VERIFY_3H_NOLEAK.ipynb` | Table 9, WUSTL column | ~1.8 h |
| `TABLE5_TONIOT_FEDERATED_T30.ipynb` | Table 6; Figures 3, 4 | ~2.9 h |

Each begins with a settings cell and prints a **label-leakage guard** line on
startup. On WUSTL that line must read

```
label-leakage guard: dropped ['Traffic'] -> 41 features
```

and on ToN-IoT

```
leakage guard: removed ['label', 'type', 'src_ip', 'dst_ip'] -> 40 features
```

If either prints anything else, stop: the run is not the one reported.

Results are written to JSON as each configuration finishes, so an interrupted
Colab session resumes where it stopped.

## Verifying without running anything

```bash
python scripts/verify_results.py
```

Reads `results/*.json` and checks every value against the article's tables.
No dataset, no GPU, about a second.

## Protocol actually used

| | WUSTL-IIoT-2021 (RR) | ToN-IoT network (RR) |
|---|---|---|
| Rounds | 15 | 30 |
| Training data | 30% stratified sample | full training split |
| Test data | full split | full split |
| Clients | 5, FedTrust, Dirichlet 0.5 | 5, FedTrust, Dirichlet 0.5 |
| Detector | MLP-HIDS `[256,128,64]` | MLP-HIDS `[256,128,64]` |
| Seed | 42 | 42 |

Round counts were fixed per dataset at the point where validation accuracy
stopped improving. Reported metrics are those of the **final-round** model of a
single seed.

## Datasets

Not redistributed. Obtain them from their original sources and see
[`docs/DATA_PATHS.md`](docs/DATA_PATHS.md) for the expected filenames and the
columns that must be excluded.

- WUSTL-IIoT-2021 — Washington University in St. Louis
- ToN-IoT — UNSW Canberra
- SCVIC-APT-2021 — University of Waterloo

## Requirements

`pip install -r requirements.txt`. The notebooks are written for Google Colab
and run unmodified there.

## Citation

See `CITATION.cff`. Concept DOI **10.5281/zenodo.22804616** always resolves to
the newest release.

## License

MIT. See `LICENSE`.
