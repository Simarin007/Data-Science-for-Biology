# Session 1 — Answers: Menstrual Cycle Dataset (Group B)

---

## Q1. How many tables are there, and what does each one contain?

There are **4 CSV files** in `data/group_b_menstruation/`:

1. **00_sample_ids_period.csv** — Links sample IDs to participants, time points
   relative to the menstrual period, and study arm (birth control vs. no birth
   control). This is the "hub" table that connects everything.

2. **01_participant_metadata_period.csv** — Demographic and clinical information
   about each participant: age, PCOS status (yes/no), and which menstrual
   product they use (tampon, pad, or menstrual cup).

3. **02_luminex_period.csv** — Cytokine and chemokine concentrations measured
   using a Luminex assay. The table is in long (tidy) format: each sample
   appears in 8 rows, one for each of the 8 immune markers measured.

4. **03_flow_cytometry_period.csv** — Immune cell counts from flow cytometry,
   showing how many cells of each type were found in each sample after
   gating. Populations include neutrophils, CD4+ and CD8+ T cells, and
   other leukocyte subsets.

The dimensions are: sample_ids (108 rows × 4 columns), metadata (27 × 6),
luminex (864 × 4), and flow (108 × 10).

---

## Q2. For each table, what does one row represent (the unit of observation)?

- **00_sample_ids_period.csv**: One biological sample collected from a
  participant at a specific time point (e.g. at the onset of bleeding, one
  week after the period ends, etc.). Each row gives that sample a unique
  ID (`sample_id`) and tags it with the participant ID and time point.

- **01_participant_metadata_period.csv**: One study participant. Each of
  the 27 participants appears once, with their age, PCOS status, and other
  background information.

- **02_luminex_period.csv**: The measured concentration of one cytokine in
  one sample. Every sample has 8 such rows (one per cytokine), so each
  sample_id appears 8 times in this table.

- **03_flow_cytometry_period.csv**: The complete set of cell counts from
  flow cytometry for one sample. One row per sample (108 total), with
  counts for each gated immune cell population in separate columns.

---

## Q3. What does each column mean? Which are categorical, numeric, or identifiers?

### 00_sample_ids_period.csv

| Column | Role | Values |
|--------|------|--------|
| `pid` | Identifier | `pid_01` through `pid_27` |
| `time_point` | Categorical | `onset`, `end_bleeding`, `week_prior`, `week_post` |
| `arm` | Categorical | `birth_control`, `no_birth_control` |
| `sample_id` | Identifier (unique) | `SAMP001` through `SAMP108` |

### 01_participant_metadata_period.csv

| Column | Role | Values |
|--------|------|--------|
| `pid` | Identifier | `pid_01` through `pid_27` |
| `arm` | Categorical | `birth_control`, `no_birth_control` |
| `age` | Numeric | Range 26–41 years |
| `pcos_status` | Categorical | `no disease`, `pcos` |
| `period_product` | Categorical | `tampon`, `pad`, `menstrual_cup` |
| `sex` | Categorical | All `female` |

### 02_luminex_period.csv

| Column | Role | Values |
|--------|------|--------|
| `sample_id` | Identifier | Links to sample_ids table |
| `cytokine` | Categorical | `IL-1a`, `IL-1b`, `IL-6`, `TNFa`, `MIG`, `IFNg`, `IP-10`, `MIP-3a` |
| `conc` | Numeric | Concentration in pg/mL |
| `limits` | Categorical (flag) | `not_censored`, `below_detection_limit`, `above_detect_limit` |

### 03_flow_cytometry_period.csv

| Column | Role | Meaning |
|--------|------|---------|
| `sample_id` | Identifier | Links to sample_ids table |
| `live_cd19_negative` | Numeric | Starting population (live cells after B cell exclusion) |
| `cd45_negative` | Numeric | Non-immune / stromal cells |
| `cd45_positive` | Numeric | Leukocytes (immune cells) |
| `neutrophils` | Numeric | Neutrophil count |
| `non_neutrophils` | Numeric | CD45+ cells that are not neutrophils |
| `cd3_negative` | Numeric | Non-T immune cells (NK cells, monocytes, etc.) |
| `cd3_positive` | Numeric | All T cells |
| `cd4_t_cells` | Numeric | CD4+ helper T cells |
| `cd8_t_cells` | Numeric | CD8+ cytotoxic T cells |

---

## Q4. How are the tables linked? What columns to join on?

There are two linking columns that connect all four tables:

- **`pid`** connects metadata to sample_ids. This is a one-to-many
  relationship: each participant in the metadata table can appear multiple
  times in the sample_ids table (once per sample collected at each time
  point).

- **`sample_id`** connects sample_ids to the luminex and flow tables.
  This is also one-to-many for luminex (one sample_id has 8 cytokine
  measurements) and one-to-one for flow (one sample_id has one set of cell
  counts).

Every sample_id in the luminex and flow tables exists in the sample_ids
table, and every pid in the sample_ids table exists in the metadata table.
So all tables join cleanly with no orphans.

The arm column appears in both sample_ids and metadata, but it is
**identical** across both tables for every participant (verified: 0
mismatches in 108 rows). When joining, you should drop the `arm` column
from one of the tables to avoid getting duplicate columns named `arm.x`
and `arm.y`.

A full join of all four tables produces 864 rows (108 samples × 8
cytokines) with no missing values.

---

## Q5. How many participants? How many observations per participant?

There are **27 participants** in the study. They contributed **108 samples**
in total, or about 4 per participant on average. Most participants provided
samples at all four time points (onset, end_bleeding, week_prior,
week_post), but a few contributed only 2 or 3 time points.

The 108 samples generated:
- **864 luminex measurements** (8 cytokines × 108 samples)
- **108 flow cytometry cell counts** (one complete gating profile per sample)

---

## Q6. How are the numeric variables distributed? Are any skewed? Suspicious values?

### Cytokine concentrations

The cytokine data is **heavily right-skewed** for every marker. Most samples
have low concentrations, but there is a long tail of very high values,
especially for MIG (CXCL9) and IP-10 (CXCL10). The coefficient of variation
(CV) is above 1.0 for most cytokines, meaning the standard deviation is
larger than the mean — a classic sign of strong skew.

**Suspicious values in the Luminex data:**

1. **SAMP044, IL-1a: 2500 pg/mL (above_detect_limit).** This is the only
   measurement that exceeded the assay's upper detection limit. The true
   concentration is higher than 2500, but we don't know by how much.

2. **Below-detection-limit values.** 206 out of 864 measurements (23.8%)
   are flagged as below the detection limit. The worst-affected cytokine is
   TNFa, where 57 out of 108 samples (52.8%) are below the limit. IFNg
   (35.2%) and IL-6 (36.1%) are also heavily affected. When a value is
   flagged this way, the reported concentration is not a true measurement
   — it is the assay's fixed detection limit for that cytokine (e.g. 0.09
   for IL-6, 0.24 for IL-1b, 0.21 for TNFa, 1.53 for MIG). These are
   "informative missing" values: we know the true value is somewhere below
   the limit, but not exactly where.

### Flow cytometry cell counts

The cell counts are also right-skewed, but less extremely than the
cytokines. The starting population (`live_cd19_negative`) ranges from
about 105,000 to 1.26 million cells. The CD4+ and CD8+ T cell counts
show the most variation.

**Suspicious values in the flow cytometry data:**

Two samples stand out as having unusually low cell counts:
- **SAMP005**: Only 168,895 starting cells and just 4,739 CD45+ immune
  cells. This is far below the typical range.
- **SAMP019**: Extremely low T cell counts — only 37 CD4+ and 96 CD8+
  cells, compared to a typical range of hundreds to thousands.

These could be low-quality samples (e.g. poor cell recovery during
processing) or could have biological meaning. They are worth flagging for
the downstream analysis.

Also notable: the flow cytometry columns do not add up perfectly. The
CD3+ T cell count is always larger than the sum of CD4+ and CD8+ cells
combined (by 68 to 16,055 cells, depending on the sample). This is
expected — the extra cells are likely double-negative T cells (CD4−CD8−),
gamma-delta T cells, and possibly double-positive (immature) T cells that
express CD3 but not CD4 or CD8.

---

## Q7. Where are the missing values, and what do they mean?

There are **no explicit NA values** in any of the four tables. However,
the dataset contains **informative missingness** through the `limits`
column in the Luminex table:

- **below_detection_limit** (206/864, 23.8%): The true concentration is
  lower than the assay can measure. The reported number is the assay's
  detection limit, not a real measurement. This is especially common for
  TNFa (52.8% of samples), IL-6 (36.1%), and IFNg (35.2%).

- **above_detect_limit** (1/864): Only SAMP044's IL-1a measurement. The
  true concentration is higher than 2500 pg/mL.

- **not_censored** (657/864, 76.0%): The measurement is reliable and
  within the assay's detection range.

IL-1a is the only cytokine with no censored values at all — all 108
samples have reliable IL-1a measurements.

The sample_ids, metadata, and flow tables have no missing values
whatsoever.

---

## Q8. What do you not understand about the dataset yet?

1. **What is the exact gating strategy used for flow cytometry?** The cell
   counts don't add up perfectly (CD3+ > CD4+ + CD8+ for every sample).
   Without the lab's gating protocol, we can only guess that the extra
   cells are double-negative T cells, gamma-delta T cells, or other
   unidentified subsets.

2. **What do the cytokine time trends look like?** We have samples from
   four time points across the menstrual cycle, but we haven't plotted
   them yet. It would be interesting to see whether certain cytokines peak
   at specific phases, and whether this differs between the birth control
   and no-birth-control groups.

3. **Why is the birth control arm larger than the no-birth-control arm?**
   The study is imbalanced: 61 samples from the birth control group vs. 47
   from the no-birth-control group. We don't know if this was intentional
   or happened by chance.

4. **What is the right way to handle the censored cytokine data?**
   Should we treat below-detection-limit values as missing, keep the
   detection limit as a placeholder, or use a statistical method to impute
   them? The best choice depends on the planned analysis.

5. **Does PCOS status confound the relationship between birth control use
   and immune markers?** PCOS causes hormonal differences that could
   independently affect cytokine levels. Checking whether PCOS is
   balanced across the two study arms is important.

6. **What does the period product information tell us?** Participants who
   use tampons, pads, or menstrual cups might have different vaginal
   microenvironments, which could influence local immune markers. It's not
   clear whether this variable is a confounder, a variable of interest, or
   just background information.

7. **What is the broader clinical context?** We don't know the study's
   original hypothesis, the inclusion and exclusion criteria for
   participants, or why these particular 8 cytokines and immune cell
   populations were chosen for measurement.