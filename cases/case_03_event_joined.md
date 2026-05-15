# Case 03 — `admiral/event_joined`

**Task ID:** `pharmaverse/admiral/event_joined`
**Package:** admiral | **Track:** clinical_pilot | **Level:** L3
**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **Pass rate:** 0/5

---

## Prompt (verbatim)

```
Write R code to construct an event_joined specification object using admiral.
At the beginning, load required packages: library(admiral).

**Inputs:**
- `inputs/dataset_name.tsv` — name of the source dataset (column: dataset_name)

Call admiral::event_joined() with the dataset_name from the input file.
For the remaining arguments use these defaults:
  condition = TRUE
  join_vars = exprs()
  join_type = "all"

The function returns an S3 specification object of class "event_joined".
It does NOT perform a join on data — it creates a declaration that
derive_param_tte() consumes later.

Save:
  - outputs/result.rds  — the full object (saveRDS)
  - outputs/result.csv  — one-row data frame: data.frame(class = class(result)[1])
```

---

## Input

**`inputs/dataset_name.tsv`**

```
dataset_name
adsl
```

---

## Reference Solution

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

dn <- read.delim("inputs/dataset_name.tsv", check.names = FALSE, stringsAsFactors = FALSE)
dataset_name <- as.character(dn[[1]][1])

result <- admiral::event_joined(
  dataset_name = dataset_name,
  condition    = TRUE,
  join_vars    = exprs(),
  join_type    = "all"
)

dir.create("outputs", showWarnings = FALSE)
saveRDS(result, "outputs/result.rds")
write.csv(data.frame(class = class(result)[1]), "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"class"
"event_joined"
```

Output shape: **(1, 1)**

The graded CSV records the S3 class name of the returned specification object. The full object is in `result.rds`.

---

## Agent Failure

**Pass rate: 0/5**

**Representative run (sample_00) — treated arguments as data columns, crashed on type check:**

```
Creating event_joined object...

ERROR: Failed to create event_joined object:
  Argument `first_cond_upper` must be a filter condition, but is a symbol
```

The agent read the input files (dataset_name, join_vars, etc.) and tried to pass them directly as arguments, including `first_cond_upper` which it inferred from the input filenames. admiral's argument validation rejected the symbol.

**Another failure (sample_01):**
```
Error during execution: invalid regular expression '[+\-*/()]', reason 'Invalid character range'
```
The agent tried to parse the `order` input field as an arithmetic expression using regex, which produced a malformed pattern.

**Root cause:** The model treats `event_joined()` as a data-joining function, not a constructor. It has no knowledge that the function builds an S3 specification object — it tries to perform an actual join or process data rows, leading to type errors or wrong output shape.

| Failure mode | Runs | What happened |
|---|---|---|
| `first_cond_upper` type error | 2/5 | Passed symbol instead of filter condition |
| Regex parse error in `order` | 1/5 | Tried to parse expression strings with malformed regex |
| Wrong output shape (data frame rows) | 2/5 | Returned dataset rows instead of S3 constructor object |
