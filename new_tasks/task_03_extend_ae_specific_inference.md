# Task 03 — `metalite.ae/extend_ae_specific_inference`

**Package:** metalite.ae | **Function:** `extend_ae_specific_inference()`
**Domain:** Safety reporting — AE risk difference with CI

---

## Why This Task Matters

`extend_ae_specific_inference()` adds **risk difference, confidence intervals, and p-values** to SOC/PT-level AE counts. This is the statistical backbone of the AE frequency table with inference — required in nearly every regulatory safety submission that compares AE rates between treatment arms.

---

## Prompt

```
Write R code to add risk difference inference statistics to an AE-specific analysis.
At the beginning, load: library(metalite.ae)

Input files (TSV, in inputs/ directory):
  - adsl.tsv : ADSL subject-level dataset
  - adae.tsv : ADAE adverse event dataset

Step 1 — Build meta_adam (same pattern as prepare_ae_summary):
  adsl <- read.delim("inputs/adsl.tsv")
  adae <- read.delim("inputs/adae.tsv")
  meta <- meta_ae_example()
  meta$data_population <- adsl
  meta$data_observation <- adae

Step 2 — Prepare AE-specific analysis:
  outdata <- prepare_ae_specific(
    meta        = meta,
    population  = "apat",
    observation = "wk12",
    parameter   = "any"
  )
  # outdata$order encodes hierarchy:
  #   multiples of 1000 (1000, 2000, ...) = SOC (System Organ Class) level
  #   non-multiples              (1001, 1002, ...) = PT (Preferred Term) level

Step 3 — Add inference:
  out_inf <- extend_ae_specific_inference(outdata, ci = 0.95)
  # Adds to the list:
  #   out_inf$diff     : data frame with diff_2, diff_3 (risk difference vs arm 1)
  #   out_inf$ci_lower : data frame with lower_2, lower_3
  #   out_inf$ci_upper : data frame with upper_2, upper_3
  #   out_inf$p        : data frame with p_2, p_3 (two-sided p-values)

Step 4 — Extract first 5 SOC rows:
  soc_idx <- which(out_inf$order %% 1000 == 0 & out_inf$order > 200)
  idx     <- soc_idx[1:5]

Step 5 — Build and save result:
  result <- data.frame(
    name    = out_inf$name[idx],
    n_1     = out_inf$n$n_1[idx],
    n_2     = out_inf$n$n_2[idx],
    n_3     = out_inf$n$n_3[idx],
    diff_2  = round(out_inf$diff$diff_2[idx], 4),
    diff_3  = round(out_inf$diff$diff_3[idx], 4),
    lower_2 = round(out_inf$ci_lower$lower_2[idx], 4),
    upper_2 = round(out_inf$ci_upper$upper_2[idx], 4),
    p_2     = round(out_inf$p$p_2[idx], 4)
  )
  write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Inputs

Same `adsl.tsv` and `adae.tsv` as Task 02 (CDISCPILOT01 dataset).

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

outdata <- prepare_ae_specific(
  meta        = meta,
  population  = "apat",
  observation = "wk12",
  parameter   = "any"
)

out_inf <- extend_ae_specific_inference(outdata, ci = 0.95)

soc_idx <- which(out_inf$order %% 1000 == 0 & out_inf$order > 200)
idx     <- soc_idx[1:5]

result <- data.frame(
  name    = out_inf$name[idx],
  n_1     = out_inf$n$n_1[idx],
  n_2     = out_inf$n$n_2[idx],
  n_3     = out_inf$n$n_3[idx],
  diff_2  = round(out_inf$diff$diff_2[idx], 4),
  diff_3  = round(out_inf$diff$diff_3[idx], 4),
  lower_2 = round(out_inf$ci_lower$lower_2[idx], 4),
  upper_2 = round(out_inf$ci_upper$upper_2[idx], 4),
  p_2     = round(out_inf$p$p_2[idx], 4)
)

write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"name","n_1","n_2","n_3","diff_2","diff_3","lower_2","upper_2","p_2"
"Cardiac disorders",18,13,13,-5.9524,-6.3123,-17.8906,5.9518,0.8393
"Congenital, familial and genetic disorders",2,1,0,-1.1905,-2.381,-7.2459,4.3176,0.7193
"Ear and labyrinth disorders",1,2,1,1.1905,-0.0277,-4.3176,7.2459,0.2807
"Eye disorders",1,2,4,1.1905,3.4607,-4.3176,7.2459,0.2807
"Gastrointestinal disorders",21,15,17,-7.1429,-5.2326,-19.6267,5.3925,0.8697
```

Output shape: **(5, 9)**

Columns: `diff_2`/`diff_3` = risk difference of Low Dose / High Dose vs Placebo (arm 1); `lower_2`/`upper_2` = 95% CI; `p_2` = two-sided p-value.

---

## Agent Failure (biomni / deepseek-v3.2-exp)

**Pass rate: 0/3**

**Representative failure (sample_02) — skipped the pipeline, rejected `.tsv` input:**

```r
# Agent tried to read ADaM files with a custom reader that only accepts CSV/RDS/RData
# Then called extend_ae_specific_inference() directly on raw data
result <- metalite.ae::extend_ae_specific_inference(
  data        = adae,       # ← wrong: first arg must be an outdata list
  population  = adsl,
  observation = "any"
)
```

**Actual stdout:**
```
Step 1: Reading input data from 'inputs/' directory...
Found 2 input file(s):
  - adae.tsv
  - adsl.tsv

Fatal error in script execution:
No data could be read from input files. Please check file formats and content.

Script terminated with errors.
```

**Actual stderr:**
```
Warning: Unsupported file format: .tsv (file: adae.tsv)
Warning: Unsupported file format: .tsv (file: adsl.tsv)
```

**Root cause:** The agent used a custom file reader that rejected `.tsv` format. It never reached `extend_ae_specific_inference()`. The correct two-step pipeline is: `prepare_ae_specific()` → `extend_ae_specific_inference(outdata, ci=0.95)`. The agent had no knowledge of the intermediate `outdata` list and tried to call the inference function directly on raw ADaM datasets.
