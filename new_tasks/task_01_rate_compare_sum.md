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

## Agent Failure (biomni / deepseek-v3.2-exp)

**Pass rate: 1/3** — sample_01 passed by coincidence: balanced strata (n=86/86/86 vs 84/84/84) make `weight="ss"` numerically equal to `weight="cmh"`.

**Representative failure (sample_00) — wrong options, wrong output format:**

```r
n0     <- scan('inputs/n0.tsv', skip = 1, quiet = TRUE)   # ← should be read.delim()$n0
strata <- scan('inputs/strata.tsv', what = character(), skip = 1, quiet = TRUE)

result <- rate_compare_sum(
  ...,
  weight = "ss",        # ← wrong: should be "cmh"
  test   = "one.sided", # ← wrong: should be "two.sided"
)

write.table(result, file = 'outputs/result.tsv', ...)  # ← wrong: should be CSV at result.csv
```

**Actual stdout:**
```
Analysis completed successfully.
Results saved to outputs/result.tsv

Summary of results:
        est  z_score           p      lower     upper
1 0.1383352 4.227384 1.18212e-05 0.07381009 0.2052666
```

The numeric values look close but the p-value differs (`1.18212e-05` vs expected `2.36424e-05`) because `test="one.sided"` halves the two-sided p-value. Output saved to `.tsv` not `.csv`, so grading cannot find `outputs/result.csv`.

**Key errors:** used `scan()` instead of `read.delim()$col`; defaulted to `weight="ss"` and `test="one.sided"`; wrote TSV instead of CSV.
