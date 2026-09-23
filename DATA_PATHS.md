# Datasets

None of the three datasets is redistributed here. Obtain each from its original
source, place the file where the notebook's settings cell expects it, and check
the leakage-guard line printed on startup.

## WUSTL-IIoT-2021

Source: Washington University in St. Louis, IIoT testbed dataset.

Expected file: `wustl-iiot-2021.csv`, 43 columns, ending
`… , SrcJitAct, DstJitAct, Traffic, Target`.

| | |
|---|---|
| Label | `Target` (0 benign, 1 attack) |
| **Must be excluded** | **`Traffic`** — the attack type as text; `Traffic == 'normal'` if and only if `Target == 0` |
| Features after exclusion | **41** |

Guard line: `label-leakage guard: dropped ['Traffic'] -> 41 features`

## ToN-IoT (network subset)

Source: UNSW Canberra, ToN-IoT datasets.

Expected file: `ton_iot__network.csv`, 44 columns, ending `… , label, type`.

| | |
|---|---|
| Label | `label` (0 normal, 1 attack) |
| **Must be excluded** | **`type`** — the attack type as text, which determines `label` exactly |
| Also excluded | `src_ip`, `dst_ip` — only 51 distinct source addresses; 44.5% of rows map to a single label through the source address alone, a testbed artifact |
| Deduplication | 27,462 exact duplicate rows removed before splitting; 211,043 → 183,581 |
| Features after exclusion | **40** |

Guard line: `leakage guard: removed ['label', 'type', 'src_ip', 'dst_ip'] -> 40 features`

## ToN-IoT (Linux process subset)

Source: UNSW Canberra. 90,112 records, 16 process-level attributes. Used by the
notebooks carried forward from earlier runs; see `docs/KNOWN_ISSUES.md` §3.

## SCVIC-APT-2021

Source: University of Waterloo. Multi-class APT stages. Used by the SCVIC
notebooks carried forward from earlier runs.

## A general caution

Both network datasets ship **two** label columns — one binary, one naming the
attack type. Taking "every column except the last" as features retains the other
one and hands the model the answer. That is the defect described in
`docs/KNOWN_ISSUES.md` §1. Any new loader should name the label column
explicitly and assert that the companion column is absent.
