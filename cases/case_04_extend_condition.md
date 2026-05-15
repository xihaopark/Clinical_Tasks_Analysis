# Case 04 — `admiral/extend_condition`

**Task ID:** `pharmaverse/admiral/extend_condition`
**Package:** admiral | **Track:** clinical_pilot | **Level:** L2
**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **Pass rate:** 0/5

---

## Prompt (verbatim)

```
Write R code to implement the **Extend condition** workflow using the `admiral` package.
At the beginning, load required packages: library(admiral).

**Inputs:**
- `inputs/cond.tsv`  — a condition string, e.g. "PARAMCD == 'SYSBP'"
- `inputs/var.tsv`   — a variable name string, e.g. "AVISIT"
- `inputs/is.tsv`    — a value string, e.g. "BASELINE"

**Required outputs for grading (exact paths):**
- `outputs/result.csv`

Use `admiral::extend_condition` when it is the correct public API for this task.
```

---

## Inputs

**`inputs/cond.tsv`**
```
cond
yes
```

**`inputs/var.tsv`**
```
var
yes
```

**`inputs/is.tsv`**
```
is
yes
```

---

## Reference Solution

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

cond    <- as.character(read.delim("inputs/cond.tsv",  stringsAsFactors=FALSE)[[1]][1])
var     <- as.character(read.delim("inputs/var.tsv",   stringsAsFactors=FALSE)[[1]][1])
is_val  <- as.character(read.delim("inputs/is.tsv",    stringsAsFactors=FALSE)[[1]][1])

result <- admiral::extend_condition(cond, var, is_val)

dir.create("outputs", showWarnings = FALSE)
write.csv(
  data.frame(cond = cond, var = var, is_val = is_val, result = result),
  "outputs/result.csv",
  row.names = FALSE
)
```

---

## Expected Output (`outputs/result.csv`)

```
"cond","var","is_val","result"
"yes","yes","yes","yes & yes == 'yes'"
```

Output shape: **(1, 4)**

---

## Agent Failure

**Pass rate: 0/5**

**Representative run (sample_01) — function not found:**

```
Error in extend_condition(cond = c, var = v, is = i) :
  could not find function "extend_condition"
Calls: mapply -> <Anonymous>
Execution halted
```

The model called `extend_condition()` via `mapply` iterating over multiple conditions — it did not know the function takes three **scalar strings** and returns one string. When the function wasn't available as `extend_condition()` (it requires `admiral::extend_condition()`), the run crashed.

**sample_00 (ran but wrong output):**
```
Processing complete.
Number of conditions processed: 5
Output files written to outputs/ directory:
  - extended_conditions.csv    ← wrong filename (should be result.csv)
```
Output had 5 rows (iterated row-by-row) and was missing the `result` column.

**Root cause:** `extend_condition(cond, var, is)` is a scalar string utility: call it once with three strings, get back one combined condition string. The model treats it as a row-wise dataset operation.

| Failure mode | Runs | What happened |
|---|---|---|
| `could not find function "extend_condition"` | 2/5 | Used bare name; should be `admiral::extend_condition()` |
| Wrong shape (5 rows instead of 1) | 3/5 | Applied row-by-row; missing `result` column |
