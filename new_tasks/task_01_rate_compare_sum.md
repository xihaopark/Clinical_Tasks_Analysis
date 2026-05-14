# Task 01 — `metalite.ae/rate_compare_sum`

**Package:** metalite.ae | **Function:** `rate_compare_sum()`
**Statistical method:** Miettinen-Nurminen stratified rate comparison

---

## Why This Task Matters

`rate_compare_sum()` implements the **Miettinen-Nurminen (MN) test** for the difference of two proportions, with optional CMH stratification. This is the **FDA-preferred method** for adverse event rate comparison in regulatory submissions — not `prop.test()`, not a chi-square test. It takes aggregate-level data (counts per stratum) rather than subject-level records.

---

## Prompt

```
Write R code to compute a stratified Miettinen-Nurminen rate comparison.
At the beginning, load: library(metalite.ae)

Input files (TSV, in inputs/ directory):
  - n0.tsv   : sample sizes in control group, one row per stratum (column: n0)
  - n1.tsv   : sample sizes in treatment group (column: n1)
  - x0.tsv   : event counts in control group (column: x0)
  - x1.tsv   : event counts in treatment group (column: x1)
  - strata.tsv: stratum labels (column: strata)

Load each file and extract the column as a vector:
  n0 <- read.delim("inputs/n0.tsv")$n0
  (repeat for n1, x0, x1, strata)

Call:
  result <- rate_compare_sum(
    n0     = n0,
    n1     = n1,
    x0     = x0,
    x1     = x1,
    strata = strata,
    weight = "cmh",
    test   = "two.sided",
    alpha  = 0.05
  )

The function returns a 1-row data frame with columns:
  est      - risk difference (proportion treatment minus proportion control)
  z_score  - test statistic
  p        - two-sided p-value
  lower    - lower bound of 95% CI
  upper    - upper bound of 95% CI

Save to outputs/result.csv (row.names = FALSE).
```

---

## Inputs

**`inputs/n0.tsv`**
```
n0
86
86
86
```

**`inputs/n1.tsv`**
```
n1
84
84
84
```

**`inputs/x0.tsv`**
```
x0
70
9
44
```

**`inputs/x1.tsv`**
```
x1
77
5
73
```

**`inputs/strata.tsv`**
```
strata
701
702
703
```

---

## Reference Solution

```r
library(metalite.ae)

dir.create("outputs", showWarnings = FALSE)

n0     <- read.delim("inputs/n0.tsv")$n0
n1     <- read.delim("inputs/n1.tsv")$n1
x0     <- read.delim("inputs/x0.tsv")$x0
x1     <- read.delim("inputs/x1.tsv")$x1
strata <- read.delim("inputs/strata.tsv")$strata

result <- rate_compare_sum(
  n0     = n0,
  n1     = n1,
  x0     = x0,
  x1     = x1,
  strata = strata,
  weight = "cmh",
  test   = "two.sided",
  alpha  = 0.05
)

write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"est","z_score","p","lower","upper"
0.138335179,4.227383911,2.36424e-05,0.073810088,0.205266635
```

Output shape: **(1, 5)**

---

## Agent Run Results

**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **k = 3**
**Pass rate: 1/3**

| Sample | Status | Root cause |
|--------|--------|------------|
| sample_00 | FAIL | Wrote `outputs/result.tsv` (tab-delimited) instead of `outputs/result.csv` |
| sample_01 | **PASS** | Correct output file; used `weight="ss"` but strata are balanced (n0=n1 per stratum) so ss==cmh numerically |
| sample_02 | FAIL | Wrote to `outputs/rate_compare_result.csv` (wrong filename) |

### Agent code — sample 0

```r
dir.create('outputs', showWarnings = FALSE)
library(metalite.ae)

n0     <- scan('inputs/n0.tsv',     skip = 1, quiet = TRUE)
n1     <- scan('inputs/n1.tsv',     skip = 1, quiet = TRUE)
x0     <- scan('inputs/x0.tsv',     skip = 1, quiet = TRUE)
x1     <- scan('inputs/x1.tsv',     skip = 1, quiet = TRUE)
strata <- scan('inputs/strata.tsv', what = character(), skip = 1, quiet = TRUE)

result <- rate_compare_sum(
  n0 = n0, n1 = n1, x0 = x0, x1 = x1, strata = strata,
  delta = 0,
  weight = "ss",        # ← should be "cmh"
  test   = "one.sided", # ← should be "two.sided"
  alpha  = 0.05
)

write.table(result, file = 'outputs/result.tsv', sep = '\t', row.names = FALSE)
# ↑ wrote TSV, not CSV; task expects outputs/result.csv
```

### Agent code — sample 1

```r
library(metalite.ae)
dir.create("outputs", showWarnings = FALSE)

n0     <- scan("inputs/n0.tsv",     what = numeric(),   skip = 1, quiet = TRUE)
n1     <- scan("inputs/n1.tsv",     what = numeric(),   skip = 1, quiet = TRUE)
strata <- scan("inputs/strata.tsv", what = character(), skip = 1, quiet = TRUE)
x0     <- scan("inputs/x0.tsv",     what = numeric(),   skip = 1, quiet = TRUE)
x1     <- scan("inputs/x1.tsv",     what = numeric(),   skip = 1, quiet = TRUE)

result <- rate_compare_sum(
  n0 = n0, n1 = n1, x0 = x0, x1 = x1, strata = strata,
  delta = 0,
  weight = "ss",       # ← should be "cmh"
  test   = "two.sided",
  alpha  = 0.05
)
write.csv(result, file = "outputs/result.csv", row.names = FALSE)
# Also tried to write result$est, result$z_score etc. as separate .tsv — those don't exist
```

### Agent code — sample 2

```r
library(metalite.ae)
dir.create('outputs', showWarnings = FALSE)

n0     <- scan("inputs/n0.tsv",     skip = 1, quiet = TRUE)
n1     <- scan("inputs/n1.tsv",     skip = 1, quiet = TRUE)
x0     <- scan("inputs/x0.tsv",     skip = 1, quiet = TRUE)
x1     <- scan("inputs/x1.tsv",     skip = 1, quiet = TRUE)
strata <- scan("inputs/strata.tsv", what = character(), skip = 1, quiet = TRUE)

result <- rate_compare_sum(
  n0 = n0, n1 = n1, x0 = x0, x1 = x1, strata = strata,
  delta = 0,
  weight = "ss",       # ← should be "cmh"
  test   = "two.sided",
  alpha  = 0.05
)
write.csv(result, file = "outputs/rate_compare_result.csv", row.names = FALSE)
# ↑ wrong filename; task expects outputs/result.csv
```

### Failure patterns

| Error | Description |
|-------|-------------|
| **Missing package** | `metalite.ae` not installed in sandbox — hard fail before any logic ran |
| **`scan()` instead of `read.delim()`** | All 3 samples used `scan()` with `skip=1`; prompt explicitly showed `read.delim("inputs/n0.tsv")$n0` |
| **`weight = "ss"`** | All 3 samples used sample-size weighting; prompt and reference use `"cmh"` (Cochran-Mantel-Haenszel) |
| **Output file name / format** | Sample 0 wrote `.tsv`; sample 2 used a different filename |
