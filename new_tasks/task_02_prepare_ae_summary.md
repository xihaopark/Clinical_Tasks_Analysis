# Task 02 — `metalite.ae/prepare_ae_summary`

**Package:** metalite.ae | **Function:** `prepare_ae_summary()`
**Domain:** Safety reporting — AE summary table

---

## Why This Task Matters

`prepare_ae_summary()` is the **entry point for AE safety tables** in metalite.ae. It computes incidence counts and proportions across treatment arms for AE parameters (any AE, drug-related, serious). The function requires a `meta_adam` object — a pre-configured metadata container — not a raw data frame. Understanding this metalite pattern is essential for generating regulatory-grade safety outputs.

---

## Prompt

```
Write R code to prepare an adverse event summary analysis using metalite.ae.
At the beginning, load: library(metalite.ae)

Input files (TSV, in inputs/ directory):
  - adsl.tsv : ADSL subject-level dataset (CDISC ADaM format)
  - adae.tsv : ADAE adverse event dataset (CDISC ADaM format)

metalite.ae uses a pre-configured meta_adam object to define population/observation/parameter
mappings. Start from the built-in example and replace only the data:

  adsl <- read.delim("inputs/adsl.tsv")
  adae <- read.delim("inputs/adae.tsv")

  meta <- meta_ae_example()       # built-in structure with 'apat'/'wk12' predefined
  meta$data_population <- adsl    # replace ADSL
  meta$data_observation <- adae   # replace ADAE

Then call:
  out <- prepare_ae_summary(
    meta        = meta,
    population  = "apat",
    observation = "wk12",
    parameter   = "any;rel;ser"
  )

The result is a list. Build a flat data frame combining:
  out$name    - AE parameter label (5 rows: population total + any/none/drug-related/serious)
  out$order   - ordering index
  out$n       - data frame with n_1, n_2, n_3, n_4 (counts per arm + total)
  out$prop    - data frame with prop_1, prop_2, prop_3, prop_4 (% per arm + total)

  result <- cbind(data.frame(name=out$name, order=out$order), out$n, out$prop)

Save to outputs/result.csv (row.names = FALSE).
```

---

## Inputs

`inputs/adsl.tsv` — 254-subject ADSL (CDISCPILOT01 study), key columns:
`USUBJID, TRT01A, TRT01AN, SAFFL, TRTA, TRTAN`

`inputs/adae.tsv` — 1191-record ADAE, key columns:
`USUBJID, TRTA, TRTAN, SAFFL, TRTEMFL, AEREL, AESER, AEBODSYS, AEDECOD`

*(Full files available in inputs/ — 49-column ADSL and 60-column ADAE from the pharmaverse CDISC Pilot dataset.)*

---

## Reference Solution

```r
library(metalite.ae)

dir.create("outputs", showWarnings = FALSE)

adsl <- read.delim("inputs/adsl.tsv")
adae <- read.delim("inputs/adae.tsv")

meta <- meta_ae_example()
meta$data_population <- adsl
meta$data_observation <- adae

out <- prepare_ae_summary(
  meta        = meta,
  population  = "apat",
  observation = "wk12",
  parameter   = "any;rel;ser"
)

result <- cbind(
  data.frame(name = out$name, order = out$order),
  out$n,
  out$prop
)

write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"name","order","n_1","n_2","n_3","n_4","prop_1","prop_2","prop_3","prop_4"
"Participants in population",1,84,84,86,254,NA,NA,NA,NA
"with one or more adverse events",100,79,77,69,225,94.048,91.667,80.233,88.583
"with no adverse events",200,5,7,17,29,5.952,8.333,19.767,11.417
"with drug-related{^a} adverse events",300,70,73,44,187,83.333,86.905,51.163,73.622
"with serious adverse events",400,2,1,0,3,2.381,1.190,0.000,1.181
```

Output shape: **(5, 10)**

Treatment arms: n_1 = High Dose (84), n_2 = Low Dose (84), n_3 = Placebo (86), n_4 = Total (254)

---

## Agent Failure (biomni / deepseek-v3.2-exp)

**Pass rate: 0/3**

**Representative failure (sample_00) — passed raw data frames instead of meta_adam:**

```r
result <- prepare_ae_summary(
  adsl        = adsl,          # ← wrong: first arg must be a meta_adam object
  adae        = adae,
  population  = "SAFFL == 'Y'",
  parameter   = "any;rel;ser",
)
```

**Actual stderr:**
```
any
Error in prepare_ae_specific(meta, population = population, ...):
  unused arguments (adsl = list(c("CDISCPILOT01", "CDISCPILOT01", ...)))
Calls: prepare_ae_summary -> lapply -> FUN
Execution halted
```

**Root cause:** The agent treats `metalite.ae` functions like standard tidyverse verbs that accept data frames. It has no knowledge of the `meta_adam` object pattern — the correct approach is `meta <- meta_ae_example(); meta$data_population <- adsl; meta$data_observation <- adae`.

Note: sample_01 called `prepare_ae_summary()` correctly using the `meta_adam` pattern and the function ran successfully (printed "AE summary prepared successfully! Result contains 13 elements"), but it saved to multiple separate CSVs instead of the single flat `cbind()` result required by the task.
