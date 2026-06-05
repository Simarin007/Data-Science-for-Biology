# Group Project — Session 1: Get to know your dataset

This session is about exploring the dataset your group has been assigned.
You're not analyzing it yet — you're building a mental map of what's in it.
Open the CSVs in `data/group_a_yogurt/` or `data/group_b_menstruation/`,
read them into R, and look around. Use Pi to help you when something is
unclear.

## Questions to answer as a group

- How many tables are there in your dataset, and what does each one contain?
4 Tables

- For each table, what does **one row** represent? (the unit of observation)
Each table's row indicate sample, participant, one cytokine per sample, one sample's cell counts

- For each table, what does **each column** mean? Which columns are
  categorical, which are numeric, which are identifiers?
  5 identifiers, 8 categorical, 10 numeric 

- How are the tables linked to each other? What column(s) would you use to
  join them?
  id (1→many) and sample_id (1→8 for luminex, 1→1 for flow)    

- How many participants are in the study? How many observations per
  participant?
  27 participants, ~4 samples each, 864 luminex measurements, 108 flow runs 

- How are the numeric variables distributed? Are any heavily skewed? Do any
  have suspicious values?
  Heavily right-skewed cytokines (CV >1). Flagged SAMP044 IL-1a (above limit), SAMP005 & SAMP019 (very low cell counts), and the CD3+ > CD4+ + CD8+ gap

- Where are the missing values, and what do you think they mean?
No NAs. But 206/864 (23.8%) are below detection limit (TNFa worst at 52.8%), 1 above limit

- What do you **not** understand about the dataset yet?
gating strategy, time trends, arm imbalance, censoring approach, PCOS confounding, period product meaning, clinical context 