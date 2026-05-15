# Case 05 — `admiral/derive_param_bmi`

**Task ID:** `pharmaverse/admiral/derive_param_bmi`
**Package:** admiral | **Track:** clinical_pilot | **Level:** L2
**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **Pass rate:** 0/5

---

## Prompt (verbatim)

```
Write R code to derive a BMI parameter and append it to a BDS dataset using admiral.
At the beginning, load required packages: library(admiral), library(rlang).

**Inputs:**
- `inputs/dataset.tsv`     — BDS dataset with HEIGHT and WEIGHT records
                             Columns: USUBJID, AVISIT, PARAMCD, AVAL, PARAM
- `inputs/height_code.tsv` — PARAMCD value that identifies HEIGHT records (column: height_code)
- `inputs/weight_code.tsv` — PARAMCD value that identifies WEIGHT records (column: weight_code)

Call:
  result <- admiral::derive_param_bmi(
    dataset       = dataset,
    by_vars       = exprs(USUBJID, AVISIT),   # must be exprs(), NOT c("USUBJID","AVISIT")
    weight_code   = weight_code,
    height_code   = height_code,
    get_unit_expr = extract_unit(PARAM)
  )

The function appends one BMI row per subject/visit to the dataset.
Save to outputs/result.csv (row.names = FALSE).
Create outputs/ with dir.create("outputs", showWarnings=FALSE).
```

---

## Inputs

**`inputs/dataset.tsv`**

```
USUBJID	AVISIT	PARAMCD	AVAL	PARAM
SUBJ-01	BASELINE	WEIGHT	70.0	Weight (kg)
SUBJ-01	BASELINE	HEIGHT	170.0	Height (cm)
SUBJ-02	BASELINE	WEIGHT	85.5	Weight (kg)
SUBJ-02	BASELINE	HEIGHT	175.0	Height (cm)
SUBJ-03	BASELINE	WEIGHT	62.0	Weight (kg)
SUBJ-03	BASELINE	HEIGHT	162.0	Height (cm)
```

**`inputs/height_code.tsv`**

```
height_code
HEIGHT
```

**`inputs/weight_code.tsv`**

```
weight_code
WEIGHT
```

---

## Reference Solution

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

dataset     <- read.delim("inputs/dataset.tsv",     check.names = FALSE, stringsAsFactors = FALSE)
height_code <- read.delim("inputs/height_code.tsv", check.names = FALSE, stringsAsFactors = FALSE)[[1]][1]
weight_code <- read.delim("inputs/weight_code.tsv", check.names = FALSE, stringsAsFactors = FALSE)[[1]][1]

result <- admiral::derive_param_bmi(
  dataset       = dataset,
  by_vars       = exprs(USUBJID, AVISIT),
  weight_code   = as.character(weight_code),
  height_code   = as.character(height_code),
  get_unit_expr = extract_unit(PARAM)
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"USUBJID","AVISIT","PARAMCD","AVAL","PARAM"
"SUBJ-01","BASELINE","WEIGHT",70,"Weight (kg)"
"SUBJ-01","BASELINE","HEIGHT",170,"Height (cm)"
"SUBJ-02","BASELINE","WEIGHT",85.5,"Weight (kg)"
"SUBJ-02","BASELINE","HEIGHT",175,"Height (cm)"
"SUBJ-03","BASELINE","WEIGHT",62,"Weight (kg)"
"SUBJ-03","BASELINE","HEIGHT",162,"Height (cm)"
"SUBJ-01","BASELINE","BMI",24.22145,NA
"SUBJ-02","BASELINE","BMI",27.91837,NA
"SUBJ-03","BASELINE","BMI",23.62445,NA
```

Output shape: **(9, 5)** — 6 original HEIGHT/WEIGHT rows + 3 appended BMI rows.

---

## Agent Failure

**Pass rate: 0/5**

**Representative run (sample_00) — `exprs()` vs character vector:**

```
Error in `derive_param_bmi()`:
! Argument `by_vars` must be a list of <symbol>, e.g., `exprs(USUBJID, VISIT)`.
Backtrace:
    ▆
 1. ├─base::tryCatch(...)
 2. │ └─base (local) tryCatchList(expr, classes, parentenv, handlers)
 3. │   └─base (local) tryCatchOne(expr, names, parentenv, handlers[[1L]])
 4. │     └─value[[3L]](cond)
```

The error message even shows the correct fix (`exprs(USUBJID, VISIT)`), but the model passed `c("USUBJID","AVISIT")`.

**sample_01** — missing required argument:
```
Error in derive_param_bmi: Argument `get_unit_expr` cannot be missing.
```
Model called `derive_param_bmi()` without `get_unit_expr = extract_unit(PARAM)`.

**sample_02** — wrong argument:
```
Error during BMI derivation:
unused argument (optional = TRUE)
```
Model passed a non-existent `optional` argument.

**Root cause:** The `exprs()` pattern is admiral's NSE convention — variable names must be passed as unevaluated symbols, not strings. The model consistently uses `c("VAR1", "VAR2")`, which works in base R/dplyr but crashes on admiral's `admiraldev::assert_vars()`.

| Failure mode | Runs | What happened |
|---|---|---|
| `by_vars must be list of <symbol>` | 3/5 | Passed character vector instead of `exprs()` |
| `get_unit_expr cannot be missing` | 1/5 | Omitted required argument |
| Wrong argument name | 1/5 | Passed `optional = TRUE` which doesn't exist |
