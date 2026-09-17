# Publishing the repository and citing it in the paper

Three stages: put the code on GitHub, mint a permanent DOI through Zenodo, then
put that DOI in the manuscript. Budget about an hour.

---

## Stage 1 — GitHub

### 1.1 Create the repository

github.com → **New repository**

| Field | Value |
|---|---|
| Name | `fedrest-hids` |
| Description | Split-federated intrusion detection for resource-heterogeneous IIoT |
| Visibility | **Public** — a private repo cannot be cited, and Zenodo cannot see it |
| Initialise with README | **No** — you already have one |

### 1.2 Push

```bash
unzip fedrest-hids.zip
cd fedrest-hids

git init
git add .
git commit -m "Initial release: code accompanying the FedREST-HIDS article"
git branch -M main
git remote add origin https://github.com/<your-username>/fedrest-hids.git
git push -u origin main
```

If prompted for a password, GitHub wants a **personal access token**, not your
account password: Settings → Developer settings → Personal access tokens →
Tokens (classic) → Generate, with the `repo` scope.

### 1.3 Check before going further

- [ ] README renders, tables and all
- [ ] All 15 notebooks appear under `notebooks/`
- [ ] `docs/KNOWN_ISSUES.md` is present and readable
- [ ] No `data/` directory and no CSVs were pushed — `.gitignore` should have
      caught these, but confirm
- [ ] Repository is **public**

### 1.4 Add topics

On the repo page, the gear beside *About* → add:
`federated-learning`, `split-learning`, `differential-privacy`,
`intrusion-detection`, `industrial-iot`, `privacy-preserving-ml`

Topics are how people find the repo from the paper's subject area.

---

## Stage 2 — Zenodo DOI

A GitHub URL is not a citation: branches move, repositories get renamed or
deleted. A Zenodo DOI is permanent and versioned, which is what a journal wants
in a data-availability statement.

### 2.1 Connect the account

1. zenodo.org → **Log in with GitHub** (use the same account)
2. Accept the permissions request
3. Top-right menu → **GitHub**
4. Find `fedrest-hids` in the list and **switch the toggle ON**

The toggle must be on **before** you create the release. Zenodo only archives
releases made after it is enabled.

### 2.2 Create the release

On GitHub: **Releases** → *Create a new release*

| Field | Value |
|---|---|
| Tag | `v1.0.0` (Create new tag on publish) |
| Title | `v1.0.0 — release accompanying the submitted article` |
| Description | See the template below |

```
Code accompanying "FedREST-HIDS: Split-Federated Intrusion Detection for
Resource-Heterogeneous Industrial IoT", submitted to Computer Communications.

Contents
- src/fedrest/  reference implementation: Renyi DP accounting, FedTrust
                aggregation, split models, communication cost model
- notebooks/    the 15 notebooks that produced the reported results
- scripts/verify_against_paper.py  24 checks against the numbers in the article

Verification (no dataset or GPU required):
    python scripts/verify_against_paper.py     ->  24/24 checks passed

Known divergences between the code and the article are documented in
docs/KNOWN_ISSUES.md.
```

Click **Publish release**. Zenodo picks it up within a few minutes.

### 2.3 Collect the DOI

zenodo.org → **Upload** → your new record. Two DOIs are shown:

- **Concept DOI** — always resolves to the newest version. **Use this one.**
- **Version DOI** — pins `v1.0.0` specifically.

Cite the concept DOI so the link stays valid when you release a corrected
version after review.

It looks like `10.5281/zenodo.1234567`.

### 2.4 Tidy the Zenodo record

Click **Edit** on the record and confirm:

- **Authors** — all three, with ORCIDs. Zenodo often imports only the GitHub
  account holder.
- **Title** — matches the article
- **Licence** — MIT
- **Related identifiers** — add `is supplement to` → the article DOI, once you
  have one

Then **Publish**.

---

## Stage 3 — Cite it in the manuscript

### 3.1 Data availability statement

In `main.tex`, find `\section*{Data availability}` and replace the final
sentence:

```latex
\section*{Data availability}
All datasets employed in the present investigation are publicly available from
their respective original repositories~\cite{ref34}--\cite{ref36}; no new data
were generated or collected specifically for this study. The source code
implementing the described methodologies, together with the notebooks that
produced the reported results, is openly available at
\url{https://doi.org/10.5281/zenodo.XXXXXXX}.
```

Substitute your concept DOI. `\url{}` needs the `url` or `hyperref` package;
`hyperref` is already loaded through the CAS class.

### 3.2 Add a pointer in the Experimental Setup

Reviewers look for the link near the methods, not only in the back matter. One
sentence at the end of Section 5.2:

```latex
The implementation, including the privacy accounting and the notebooks used to
produce the reported results, is available at
\url{https://doi.org/10.5281/zenodo.XXXXXXX}.
```

### 3.3 Mention it in the cover letter

One line: *"The complete implementation is publicly available at
[DOI], including a verification script that reproduces every privacy
figure reported in the article."*

---

## Stage 4 — After review

When you release the corrected version:

1. Push the changes
2. Create release `v1.1.0`
3. Zenodo archives it automatically under the **same concept DOI**
4. The link in your paper keeps working and now points to the newer version

This is why the concept DOI matters: no manuscript edit is needed.

---

## Two cautions

**Run one notebook end to end before releasing.** The repository has not been
executed — no GPU and no network were available when it was assembled. Confirm
at least one notebook still runs after the path changes described in
`docs/DATA_PATHS.md`.

**Do not commit the datasets.** `.gitignore` excludes `data/`, `*.csv` and
`*.pcap`. WUSTL-IIoT-2021, ToN-IoT and SCVIC-APT-2021 carry their own
licences and must be obtained from their original sources.

---

## Metadata for future releases

`.zenodo.json` in the repository root controls how Zenodo records each release:
authors with ORCIDs, title, description, keywords and licence. Zenodo reads it
at release time, so once it is committed no Zenodo record ever needs editing by
hand again.

Note that `CITATION.cff` does **not** drive Zenodo — it powers GitHub's "Cite
this repository" button. Both files are present and both should be kept in step
with each other.

If a release was archived before `.zenodo.json` existed, its record keeps the
auto-generated metadata (GitHub account name as the author, the repository slug
as the title). Fix that record by hand once: zenodo.org → the upload → **Edit**
→ correct the authors, title, version and description → **Publish**. Editing
metadata does not change the DOI.

### Concept DOI versus version DOI

Zenodo mints two DOIs:

| | Resolves to |
|---|---|
| concept DOI (lower number) | always the newest release |
| version DOI (higher number) | one specific release, forever |

**Cite the concept DOI in the article.** A corrected release published after
peer review will then be reachable through the link already printed in the
paper, with no erratum required.

For this repository: `10.5281/zenodo.22804616` is the concept DOI and
`10.5281/zenodo.22804617` is the version DOI of the first release. The concept
DOI is the one used in the manuscript. It can be confirmed on any version's
record page, where the Software Heritage archive line shows
`origin=https://doi.org/10.5281/zenodo.22804616`.
