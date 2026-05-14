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

## Failure Analysis

`admiral::event_joined()` returns a **declarative S3 specification object**, not a joined dataset. The model does not recognise this pattern.

| Failure mode | Runs | What happened |
|---|---|---|
| `EXEC_FAIL` — attempted data join | 2/5 | Model called `dplyr::left_join()` or similar; crashed |
| `NO_OUTPUT` — returned multi-row data frame | 3/5 | Model returned source dataset rows instead of the constructor object |

The model's mental model of admiral treats all functions as `dataset → dataset` transformations. `event_joined()` is a **constructor** — it builds a specification that `derive_param_tte()` later consumes. The output in normal use is never inspected directly; it is passed as an argument to another function.
