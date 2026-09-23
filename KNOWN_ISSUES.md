# Known issues

This file records where the code and the article agree, where they diverge, and
what was corrected before submission. It is written to be read by a reviewer.

---

## 1. Label leakage in preprocessing — found, fixed, results regenerated

**The defect.** Both dataset loaders split features from labels positionally:

```python
X = data.iloc[:, :-1]     # every column except the last
y = data.iloc[:, -1]
```

Both datasets carry **two** label columns, not one.

| Dataset | Last columns | Consequence |
|---|---|---|
| WUSTL-IIoT-2021 | `… , Traffic, Target` | `Traffic` (attack type as text) was used as a feature |
| ToN-IoT network | `… , label, type` | `type` (attack type as text) was used as a feature |

In both cases the retained column determines the label exactly — `Traffic ==
'normal'` if and only if `Target == 0`, on every row — so the model was given
the answer. The symptom was visible in the feature count: the code reported 42
features on WUSTL where the article states 41.

**The fix.** Both loaders now drop the offending column and **assert** its
absence, so a run stops rather than silently leaking. `src_ip` and `dst_ip` are
also dropped from ToN-IoT: with only 51 distinct source addresses, 44.5% of rows
map to a single label through the source address alone, which is a shortcut
specific to the testbed rather than a property of attacks.

**Effect on the results.** Roughly one point of accuracy on WUSTL. The clearest
indicator is the non-private reference, which fell from 99.998% (5 errors in
209,715 test rows) to 99.92% (165 errors).

## 2. Duplicate rows

The ToN-IoT network file contains 27,462 rows that are exact duplicates once the
label and identifier columns are removed. Identical rows falling on both sides of
a random split let a model partly memorise the test set, so duplicates are now
removed before splitting. 211,043 rows become 183,581.

## 3. What was regenerated, and what was not

**Regenerated after the fix** (results in `results/`, notebooks in `notebooks/`):

| Article artifact | Notebook |
|---|---|
| Table 4, Table 5, Figure 2, §6.1 per-class figures | `TABLE3_HEADLINE_3H_NOLEAK.ipynb` |
| Table 9, WUSTL-IIoT-2021 column | `TABLE8_VERIFY_3H_NOLEAK.ipynb` |
| Table 6, Figures 3 and 4 | `TABLE5_TONIOT_FEDERATED_T30.ipynb` |

**Not regenerated.** Table 7 (ToN-IoT Linux process), Table 8 (RC devices),
Table 10 (poisoning) and the SCVIC-APT-2021 figures stand on the earlier runs.
They were not thought to be affected: the loaders for those subsets do not
exhibit the positional split above, and their reported performance — 72%
accuracy on the Linux subset, minority-class F1 between 0.32 and 0.57 on
SCVIC — is inconsistent with a label being available as a feature, which would
drive results close to perfect. This is reasoning from evidence, not a
re-verification, and is stated here so the distinction is visible.

## 4. Protocol differs by dataset

Round counts were fixed per dataset at the point where validation accuracy
stopped improving. WUSTL-IIoT-2021 was flat from round six, so 15 rounds; the
ToN-IoT network subset was still improving at round 15, so 30 rounds. The WUSTL
runs use a 30% stratified sample of the training split, with the full test split
retained; ToN-IoT uses the full training split. All reported metrics come from
the **final-round** model of a **single seed (42)**.

Two consequences a reader should know:

- Validation accuracy fluctuates by roughly ±0.9 points between late rounds on
  ToN-IoT. The ε = 1.0 row of Table 6 landed on a downswing, which is why it sits
  close to the ε = 0.5 row. No row was selected by its validation score.
- The Table 9 columns are not comparable across datasets: WUSTL uses the protocol
  above, the ToN-IoT Linux column is carried forward from an earlier evaluation at
  T = 50 over three seeds.

## 5. The FedTrust trust scalar is outside the accounted budget

The performance component of the trust score (Eq. 6) is computed by each device
from its own held-out slice of raw data, and the resulting scalar is transmitted
to the server. Because it is a function of raw records rather than of a
differentially private release, it is **not** covered by the per-round ε
accounted in §3.2.2. The guarantee reported in the article applies to the
transmitted model updates.

The channel is narrow — one bounded scalar per device per round, aggregated
through a softmax that discards its scale — but it is a channel. §3.2.3 of the
article states this, and privatising or quantifying it is identified as future
work. The behavioural component is unaffected: it is computed from the noised
parameters the device releases in any case, and is therefore post-processing.

## 6. Clipping granularity

Gradients are clipped per batch rather than per example. The article's Eq. (2)
describes the per-batch form, so code and text agree, but this is weaker than the
per-example clipping of DP-SGD and is noted for readers who assume the latter.

## 7. Superseded material removed

The following were removed at v2.0 because they produced results no longer in the
article, or contained defects. They remain in the git history and in the v1.x
Zenodo releases:

- `VERIFY_WUSTL_RR_eps0.5_alpha5.0.ipynb` and its results — leaked, and used a
  learning rate and client count that do not match the article
- `HIDS_IIoT_RR_ToNIoT_Network_Enhanced.ipynb` — produced the previous Table 6
  with the `type` leak; it also trained a single model centrally rather than a
  federation, and the five other ε values of the previous table came from a
  privacy-budget plot rather than from training runs
- `HIDS_FL_LRDP_RR_IIoT_WUSTL_SCADA_NETWORK2021_with_poisoning.ipynb` —
  superseded by the two WUSTL notebooks above
- intermediate ablation and architecture-check notebooks whose driver called
  `run_one` with the wrong signature and therefore never executed
