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

## Agent Run Results

**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **k = 3**
**Pass rate: 0/3**

| Sample | Status | Root cause |
|--------|--------|------------|
| sample_00 | NO_OUTPUT | Agent output was bash exploration commands, not R code; `unexpected symbol` parse error |
| sample_01 | FAIL | Built custom `outdata` list; `extend_ae_specific_inference` failed silently; returned original data |
| sample_02 | EXEC_FAIL | Only recognised `.csv`/`.rds` file extensions; skipped `.tsv` inputs → empty data; function crashed |

### Agent code — sample 0

```
# The agent output was not valid R code — it emitted shell exploration commands
# formatted as if inside a bash execution block, e.g.:
#
#   Rscript -e "library(metalite.ae); help('extend_ae_specific_inference')" 2>&1
#
# No usable R script was produced. The agent attempted to discover the function's
# signature at runtime rather than synthesising a solution from the prompt.
```

### Agent code — sample 1

```r
library(metalite.ae)
dir.create("outputs", showWarnings = FALSE)

# Looks only for .csv, .rds, .RData files — misses the .tsv inputs entirely
input_files <- list.files("inputs", pattern = "\\.(csv|rds|RData)$", full.names = TRUE)
# → finds nothing, tries to use a hardcoded example outdata

result <- tryCatch({
  metalite.ae::extend_ae_specific_inference(outdata, ci = 0.95)
  # ↑ 'outdata' is a hallucinated local variable (not a real outdata list)
}, error = function(e) {
  return(outdata)
})

saveRDS(result, file = "outputs/extended_ae_inference.rds")
```

### Agent code — sample 2

```r
library(metalite.ae)
dir.create("outputs", showWarnings = FALSE)

# Reads adsl.tsv and adae.tsv — correct inputs — but calls with wrong API
ae_data   <- read.delim("inputs/adae.tsv")
subj_data <- read.delim("inputs/adsl.tsv")

result <- metalite.ae::extend_ae_specific_inference(
  data        = ae_data,     # ← wrong: expects outdata list from prepare_ae_specific()
  population  = subj_data,   # ← wrong parameter name and type
  observation = "any",       # ← wrong
  parameter   = "any",       # ← wrong
  analysis    = "desc"       # ← wrong
)
```

### Failure patterns

| Error | Description |
|-------|-------------|
| **Missing package** | `metalite.ae` not in sandbox — would fail before running |
| **Sample 0: no R code produced** | Agent emitted bash exploration commands; confused tool-use context with code output |
| **Sample 1: ignored TSV inputs** | Searched only for `.csv`/`.rds` files, found nothing; never loaded adsl/adae |
| **Sample 2: wrong function signature** | `extend_ae_specific_inference()` takes an `outdata` list (output of `prepare_ae_specific()`), not raw data frames with `data=` / `population=` / `analysis=` parameters |
| **All: missing two-step pipeline** | The correct approach requires `prepare_ae_specific()` first, then `extend_ae_specific_inference()` on its output — none of the 3 samples followed this two-step pattern |
