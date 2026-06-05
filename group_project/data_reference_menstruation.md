# Data Reference — Menstrual Cycle Study (Group B)

> **Auto-generated:** 2026-06-03
> **Purpose:** Read this file to quickly recall everything about the dataset, what was done, and what was learned.
> **Location:** `data/group_b_menstruation/`

---

## 1. Study overview

This is a study on the **menstrual cycle** comparing participants who use birth
control vs those who don't. The goal is to understand how immune markers
(cytokines and immune cell populations) change across the menstrual cycle.

**Key variables of interest:**
- **Arm:** `birth_control` vs `no_birth_control`
- **Time points:** `onset` (start of bleeding), `end_bleeding`,
  `week_prior` (1 week before), `week_post` (1 week after)
- **PCOS status:** `pcos` vs `no disease`
- **Age range:** 26–41

---

## 2. The four CSV files

### 2a. `00_sample_ids_period.csv`
- **Rows:** 108
- **What a row represents:** One biological sample from one participant at one time point
- **Columns:**
  | Column | Type | Values |
  |--------|------|--------|
  | `pid` | chr | `pid_01` … `pid_27` |
  | `time_point` | chr | `onset`, `end_bleeding`, `week_prior`, `week_post` |
  | `arm` | chr | `birth_control`, `no_birth_control` |
  | `sample_id` | chr | `SAMP001` … `SAMP108` |
- **Notes:** This is the **linking table** — joins everything via `sample_id`.
  Each participant appears multiple times (one row per sample collected).
  Every `sample_id` is unique (no duplicates).

### 2b. `01_participant_metadata_period.csv`
- **Rows:** 27 (one per participant)
- **What a row represents:** One study participant with demographics and clinical info
- **Columns:**
  | Column | Type | Values |
  |--------|------|--------|
  | `pid` | chr | `pid_01` … `pid_27` |
  | `arm` | chr | `birth_control`, `no_birth_control` |
  | `age` | num | 26–41 |
  | `pcos_status` | chr | `pcos`, `no disease` |
  | `period_product` | chr | `tampon`, `pad`, `menstrual_cup` |
  | `sex` | chr | All `female` |
- **Notes:** The `arm` column here is **redundant** with the one in
  `00_sample_ids_period.csv` (verified: 0 mismatches across all 108 rows).
  Drop `arm` from metadata when joining to avoid `arm.x` / `arm.y`.

### 2c. `02_luminex_period.csv`
- **Rows:** 864 (= 108 samples × 8 cytokines)
- **What a row represents:** Measured concentration of **one cytokine** in **one sample** (long / tidy format)
- **Columns:**
  | Column | Type | Values |
  |--------|------|--------|
  | `sample_id` | chr | Links to sample_ids |
  | `cytokine` | chr | `IL-1a`, `IL-1b`, `IL-6`, `TNFa`, `MIG`, `IFNg`, `IP-10`, `MIP-3a` |
  | `conc` | num | Concentration in pg/mL |
  | `limits` | chr | `not_censored`, `below_detection_limit`, `above_detect_limit` |

- **Censoring notes:**
  - When `limits == "below_detection_limit"`, the `conc` value is the
    assay's **detection limit**, not a true concentration. Detection limits
    are constant per cytokine:
    | Cytokine | Detection limit | # below limit |
    |----------|----------------|--------------:|
    | TNFa | 0.214 | 57 / 108 |
    | IL-6 | 0.092 | 39 / 108 |
    | IFNg | 0.305 | 38 / 108 |
    | IL-1b | 0.244 | 33 / 108 |
    | MIG | 1.526 | 19 / 108 |
    | MIP-3a | 0.305 | 14 / 108 |
    | IP-10 | 0.305 | 6 / 108 |
    | IL-1a | — | 0 / 108 |
  - When `limits == "above_detect_limit"`: only **SAMP044** IL-1a at
    2500 pg/mL. True value is higher.
  - When `limits == "not_censored"`: the measurement is reliable.
  - **Common approaches** for censored data: treat as missing, keep as-is
    (the detection limit), or impute.

### 2d. `03_flow_cytometry_period.csv`
- **Rows:** 108 (one per sample)
- **What a row represents:** Cell counts from flow cytometry gating for one sample
- **Columns:**
  | Column | Type | Meaning |
  |--------|------|---------|
  | `sample_id` | chr | Links to sample_ids |
  | `live_cd19_negative` | num | Starting population: live cells after B cell exclusion |
  | `cd45_negative` | num | Non-immune / stromal cells |
  | `cd45_positive` | num | Leukocytes (immune cells) |
  | `neutrophils` | num | Neutrophil granulocytes |
  | `non_neutrophils` | num | CD45+ cells that are not neutrophils |
  | `cd3_negative` | num | Non-T immune cells (NK cells, monocytes, etc.) |
  | `cd3_positive` | num | All T cells |
  | `cd4_t_cells` | num | CD4+ helper T cells |
  | `cd8_t_cells` | num | CD8+ cytotoxic T cells |

- **Gating strategy (approximate):**
  ```
  Live + CD19- cells
    ├── CD45- (stromal)
    └── CD45+ (leukocytes)
         ├── Neutrophils
         └── Non-neutrophils
              ├── CD3- (NK, monocytes, etc.)
              └── CD3+ (T cells)
                   ├── CD4+ helper
                   ├── CD8+ cytotoxic
                   └── Other (DN, γδ, DP)
  ```

- **Additivity notes (important!):**
  - **L0** (`live_cd19_negative` vs CD45⁻ + CD45⁺): Some samples off by
    up to 28,124 cells — likely debris or cells falling outside the CD45 gate.
  - **L1** (`cd45_positive` vs neutrophils + non-neutrophils): Very tight,
    most within 10 cells.
  - **L2** (`non_neutrophils` vs CD3⁻ + CD3⁺): Always positive — 246–12,409
    extra cells not accounted for. These are likely NK cells, dendritic cells,
    or monocytes that were not gated further.
  - **L3** (`cd3_positive` vs CD4⁺ + CD8⁺): **Always** positive — every
    single sample has extra CD3⁺ cells that are neither CD4⁺ nor CD8⁺
    (range: 68–16,055 cells). These are **double-negative T cells**,
    **gamma-delta T cells**, and possibly double-positive (immature) T cells.

---

## 3. Table relationships

```
01_participant_metadata ──── pid ──── 00_sample_ids_period
                                         │
                                         │ sample_id
                                         │
                 ┌────────────────────────┼─────────────────┐
                 │ sample_id                                │ sample_id
                 ▼                                          ▼
       02_luminex_period                         03_flow_cytometry_period
       (8 rows per sample)                       (1 row per sample)
```

- `pid` links metadata ↔ sample_ids
- `sample_id` links sample_ids ↔ luminex ↔ flow
- Full join: 864 rows (108 samples × 8 cytokines), no missing values

---

## 4. What was produced

### `group_project/data_dictionary_menstruation.qmd`
A Quarto document (R + tidyverse) that:
- Loads and describes each CSV file
- Shows column meanings & what a row represents
- Includes runnable code chunks for additivity checks, pivoting, censoring
- Shows the table relationship diagram
- Outputs to HTML with `toc: true`, `code-fold: true`, `embed-resources: true`

### Files generated:
| File | Description |
|------|-------------|
| `group_project/data_dictionary_menstruation.qmd` | Source document (editable) |
| `group_project/data_dictionary_menstruation.html` | Rendered HTML (1.2 MB) |

### Preview:
- URL: `https://{CODESPACE}-4322.{DOMAIN}/`
- Port: 4322
- Process runs detached via `setsid`, watches for `.qmd` changes

---

## 5. Analysis tips for next time

1. **Censored cytokines:** TNFa, IL-6, and IFNg have the most
   below-detection-limit values (50%+). Be careful drawing conclusions
   about these — consider using survival-style analysis or treat as
   interval-censored data.

2. **Flow cytometry subpopulations:** Don't expect strict additivity.
   The "other T cells" (CD3⁺ CD4⁻ CD8⁻) population is substantial and
   varies across time points — this could be a biologically interesting
   finding to explore.

3. **Covariates to check:** PCOS status, age, and period product type
   could all be confounders. Check balance across the `arm` groups.

4. **Repeated measures:** Each participant contributes multiple samples
   (up to 4 time points). Use mixed-effects models or participant ID as
   a random effect, not naive independent tests.

5. **Data quality:** The arms are perfectly consistent between the sample
   IDs and metadata tables. The flow cytometry is clean despite the
   non-additivity (which is expected for real gated data).