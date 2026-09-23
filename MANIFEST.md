# Manifest

Every table and figure in the article, and the notebook that produced it.

## Regenerated for the current version

| Notebook | Dataset | Produces | Results file |
|---|---|---|---|
| `TABLE3_HEADLINE_3H_NOLEAK.ipynb` | WUSTL-IIoT-2021 | Table 4; Table 5; Figure 2; §6.1 per-class figures | `results/table3_fast_T15_f30_noleak.json` |
| `TABLE8_VERIFY_3H_NOLEAK.ipynb` | WUSTL-IIoT-2021 | Table 9, WUSTL column | `results/table8_fast_T15_f30_noleak.json` |
| `TABLE5_TONIOT_FEDERATED_T30.ipynb` | ToN-IoT (network) | Table 6; Figures 3, 4 | `results/table5_federated_T30_C.json`, `results/fig_arrays_e1.0.npz` |

All three use the same federated pipeline: 5 clients, FedTrust aggregation,
Dirichlet non-IID partitioning (concentration 0.5), MLP-HIDS `[256,128,64]`,
LR 0.001, seed 42.

## Carried forward from earlier runs

These produced results that were **not** regenerated. See `docs/KNOWN_ISSUES.md`
§3 for why.

| Notebook | Dataset | Produces |
|---|---|---|
| `HIDS_FEDREST_LRDP_RR_TONIOTPROCESS_with_poisoning.ipynb` | ToN-IoT (Linux process) | Table 7; Table 9 ToN-IoT column; Table 10 |
| `HIDS_Fedrest_RR_ToNIoT_Linux_Process.ipynb` | ToN-IoT (Linux process) | Figure 5 |
| `HIDS_FEDREST_LRDP_RC_TONIOTPROCESS.ipynb` | ToN-IoT (Linux process) | Figure 15 |
| `HIDS_FEDSPLITFTIL_RRC_WUSTL.ipynb` | WUSTL-IIoT-2021 | Table 8; Figures 12, 13 |
| `HIDS_IIoT_RC_ToN_IoT_Network.ipynb` | ToN-IoT (network) | Figure 14 |
| `Enhanced_HIDS_SCVIC_RR.ipynb` | SCVIC-APT-2021 | Figures 6–11 |
| `HIDS_FEDSPLITFTIL_RRC_SCVIC2021.ipynb` | SCVIC-APT-2021 | Figures 16–18 |
| `HIDS_FL_LRDP_RC_IIoT_SCVIC_APT_2021.ipynb` | SCVIC-APT-2021 | Figures 16–18 (RC pipeline) |

## Architecture per experiment

The article uses one detector family throughout, a fully-connected feed-forward
network (MLP-HIDS), sized to each dataset. Supplementary Appendix G explains the
choice.

| Experiment | Hidden layers |
|---|---|
| WUSTL-IIoT-2021 RR; ToN-IoT network RR | `[256, 128, 64]` |
| ToN-IoT Linux process RR; SCVIC-APT-2021 RR | `[128, 64]` |
| SCVIC-APT-2021 RC | `[128, 64, 32]` |
| WUSTL-IIoT-2021 RC | `[96, 48]` device, `[48, 24]` edge |

## Notes on names

Notebooks copied out of Colab with names such as `Copy of X.ipynb` or
`X (1).ipynb` were renamed here so that each artifact maps to exactly one file.
The contents are unchanged.
