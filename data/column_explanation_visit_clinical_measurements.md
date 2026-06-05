# Column Guide: `02_visit_clinical_measurements_UKZN_workshop_2023.csv`

This dataset comes from a **UKZN (University of KwaZulu-Natal) workshop** study.
It tracks 44 participants across three time points, measuring markers related to
vaginal microbiome health.

---

## Column-by-column explanation

### `pid` — Participant ID
A code identifying each person in the study, e.g. `pid_01`, `pid_02` … `pid_44`.
Each participant keeps the same ID across all data files.

### `time_point` — When the measurement was taken
Three time points per participant:
| Time point | Meaning |
|---|---|
| **baseline** | Before any treatment started (the starting point) |
| **week_1** | 1 week into the study |
| **week_7** | 7 weeks into the study |

Each participant appears **3 times** (once per time point).

### `arm` — Which group the participant was in
| Group | Meaning |
|---|---|
| **placebo** | Received a dummy treatment (no active ingredient) |
| **treatment** | Received the actual treatment being tested |

This lets you compare the treatment vs. no treatment.

### `nugent_score` — Bacterial vaginosis (BV) indicator
A score from **0 to 10** based on examining a vaginal swab under a microscope.
It tells you about the balance of bacteria in the vagina.

| Score range | What it means |
|---|---|
| **0–3** | **Normal / healthy** — mostly *Lactobacillus* bacteria |
| **4–6** | **Intermediate** — mixed bacterial types |
| **7–10** | **Bacterial vaginosis (BV)** — too few good bacteria |

**Higher Nugent score = less healthy microbiome.**

### `crp_blood` — C-reactive protein in blood (mg/L)
A protein your body produces when there's **inflammation** somewhere. It's a
general "is your body fighting something?" marker. Normal is typically under
3 mg/L. Higher values mean more inflammation.

### `ph` — Vaginal pH
How acidic or alkaline the vagina is.

| pH range | What it suggests |
|---|---|
| **3.5–4.5** | **Healthy** — lots of *Lactobacillus* bacteria producing lactic acid |
| **> 4.5** | **Less healthy** — fewer good bacteria, possible imbalance |

---

## How these columns relate to each other

### Nugent Score ↔ pH

These two are **strongly related**. Here's the biological reason:

1. Healthy *Lactobacillus* bacteria produce **lactic acid**, keeping the vagina
   acidic (low pH ~3.5–4.5).
2. When the Nugent score is **low** (0–3, healthy), pH is also **low** (acidic
   and healthy).
3. When the Nugent score is **high** (7–10, BV), the good bacteria are gone,
   so less lactic acid is produced, and pH rises (less acidic → higher number).

**They move in the same direction** — high Nugent goes with high pH, low Nugent
goes with low pH.

Look at the treatment group participants in the data: many go from pH ~4.5–5.5
at baseline down to pH ~3.0–3.7 at week 1, and their Nugent scores drop too.
That's this relationship in action.

### Nugent Score ↔ CRP

CRP is a general inflammation marker. BV (high Nugent score) is a type of
infection/inflammation, so you might expect higher CRP when Nugent is high.
However, this relationship is **looser** than with pH, because CRP can go up
for many reasons (a cold, an injury, stress).

### pH ↔ CRP

These two are **indirectly** related through the Nugent score. If someone has
BV (high Nugent → high pH), the inflammation may also raise CRP. But again,
CRP goes up for many reasons, so this connection is weaker.

### Treatment arm's effect

If you compare **placebo** vs **treatment** participants, the treatment seems
to lower Nugent scores and pH (especially at week 1). This is the kind of
question the dataset was designed to explore — does the treatment help restore
a healthy vaginal microbiome?

---

## Quick visual summary

```
Healthy Lactobacillus bacteria
        │
        ├── produce lactic acid  →  low pH (~3.5–4.5)
        │
        └── keep bad bacteria out →  low Nugent score (0–3)

When good bacteria are lost (BV):
        │
        ├── less lactic acid     →  pH goes up (> 4.5)
        │
        └── bad bacteria multiply →  Nugent score goes up (7–10)
```

**Bottom line:** Nugent score and pH are two different ways of measuring the
same underlying thing — the health of the vaginal microbiome.
