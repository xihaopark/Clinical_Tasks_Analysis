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

## Agent Run Results

**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **k = 3**
**Pass rate: 0/3**

| Sample | Status | Root cause |
|--------|--------|------------|
| sample_00 | NO_OUTPUT | Called `prepare_ae_summary(adsl=adsl, adae=adae, ...)` — raw dataframes passed as kwargs; wrong API signature |
| sample_01 | FAIL | Built `meta_adam()` + `define_population()` chain from scratch; function ran but output structure differed |
| sample_02 | NO_OUTPUT | Expected `inputs/meta.rds` which doesn't exist; tried to create it but inputs dir is read-only |

### Agent code — sample 0

```r
library(metalite.ae)
library(dplyr)
library(readr)
dir.create('outputs', showWarnings = FALSE)

adsl <- read.delim('inputs/adsl.tsv', sep = '\t', stringsAsFactors = FALSE)
adae <- read.delim('inputs/adae.tsv', sep = '\t', stringsAsFactors = FALSE)

# Calls prepare_ae_summary with raw data frames — wrong API
result <- prepare_ae_summary(
  adsl        = adsl,
  adae        = adae,
  population  = "SAFFL == 'Y'",
  observation = "TRTEMFL == 'Y'",
  parameter   = "any;rel;ser",
  analysis    = "overview",
  ...
)
write.csv(result$table, file = "outputs/ae_summary_table.csv", row.names = FALSE)
```

### Agent code — sample 1

```r
library(metalite)
library(metalite.ae)
dir.create('outputs', showWarnings = FALSE)

adsl <- read.delim("inputs/adsl.tsv", stringsAsFactors = FALSE)
adae <- read.delim("inputs/adae.tsv", stringsAsFactors = FALSE)

# Attempts to build meta_adam from scratch — correct direction but missing structure
meta <- meta_adam(population = adsl, observation = adae)

plan <- plan(analysis = "ae_summary", population = "apat",
             observation = "wk12", parameter = "any;rel;ser")
meta <- meta |>
  define_plan(plan = plan) |>
  define_population(name = "apat", id = "USUBJID", group = "TRTA",
                    subset = SAFFL == "Y", ...) |>
  define_observation(name = "wk12", ...) |>
  define_parameter(name = "any", ...) |>
  define_parameter(name = "rel", subset = AEREL %in% c("PROBABLE","POSSIBLE"), ...) |>
  define_parameter(name = "ser", subset = AESER == "Y", ...)

result <- prepare_ae_summary(meta, population = "apat",
                              observation = "wk12", parameter = "any;rel;ser")
# Tries to access result$n, result$prop, result$diff, result$n_pop
```

### Agent code — sample 2

```r
library(metalite.ae)
dir.create('outputs', showWarnings = FALSE)

# Expects inputs/meta.rds, inputs/population.csv, inputs/observation.csv, inputs/parameter.txt
# — none of these files exist; task only provides adsl.tsv and adae.tsv
if (!file.exists('inputs/meta.rds')) {
  meta <- metalite::meta_adam(
    population  = data.frame(USUBJID = paste0("S", 1:100), ...),
    observation = data.frame(USUBJID = ..., AEDECOD = ...)
  )
}
result <- metalite.ae::prepare_ae_summary(
  meta       = meta,
  population = population,   # ← passes entire data frame, not string "apat"
  ...
)
```

### Failure patterns

| Error | Description |
|-------|-------------|
| **Missing package** | `metalite.ae` not in sandbox — would fail before running |
| **Sample 0: wrong API** | Passed raw `adsl`/`adae` directly; `prepare_ae_summary()` requires a `meta_adam` object |
| **Sample 1: correct meta direction, wrong builder** | Used `meta_adam()` constructor + manual `define_*()` chain; should use `meta_ae_example()` as base and replace `$data_population` / `$data_observation` |
| **Sample 2: invented file structure** | Expected RDS + CSV files that don't exist; prompt explicitly provided only `.tsv` inputs |
