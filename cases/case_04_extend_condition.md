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

## Failure Analysis

`extend_condition(cond, var, is)` is a **string-level meta-programming utility** — it takes three scalar strings and returns one combined condition string. The entire function is a string concatenation.

| Failure mode | Runs | What happened |
|---|---|---|
| Shape mismatch (125 rows) | 1/5 | Model iterated over all combinations in a dataset |
| Shape mismatch (5 rows) | 3/5 | Model applied the condition row-by-row over the input data |
| Missing `result` column | 5/5 | All runs produced 3 columns instead of 4 |

**Root cause:** The model's mental model of admiral treats all functions as `dataset → dataset` transformations. `extend_condition()` is a **meta-programming tool** that manipulates condition strings at the expression level. The model does not recognise functions that operate below the data frame layer.
