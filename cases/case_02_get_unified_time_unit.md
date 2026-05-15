# Case 02 — `admiral/get_unified_time_unit`

**Task ID:** `pharmaverse/admiral/get_unified_time_unit`
**Package:** admiral | **Track:** clinical_pilot | **Level:** L2
**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **Pass rate:** 0/5

---

## Prompt (verbatim)

```
Normalize mixed **time unit** labels. Load `library(admiral)`, `library(dplyr)`.

**Inputs:** `inputs/time_unit.tsv`.

Some admiral helpers are **not exported**; the reference uses
`get("<fn>", envir = asNamespace("admiral"))(...)`. Using the same call is
fair game and avoids false failures from `admiral::<fn>` not existing.

**Computation:** **`get_unified_time_unit(time_unit)`**.

**Required outputs:** `outputs/result.csv`
```

---

## Input

**`inputs/time_unit.tsv`**

```
time_unit
days
```

---

## Reference Solution

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))

time_unit_df <- read.delim("inputs/time_unit.tsv", check.names = FALSE, stringsAsFactors = FALSE)
time_unit <- as.character(time_unit_df[[ncol(time_unit_df)]])
if (length(time_unit) == 0 || time_unit[1] == "test_value") time_unit <- "days"

get_unified_time_unit <- get("get_unified_time_unit", envir = asNamespace("admiral"))
result <- get_unified_time_unit(time_unit)

dir.create("outputs", showWarnings = FALSE)
write.csv(
  data.frame(time_unit = time_unit, result = result),
  "outputs/result.csv",
  row.names = FALSE
)
```

---

## Expected Output (`outputs/result.csv`)

```
"time_unit","result"
"days","days"
```

Output shape: **(1, 2)**

---

## Agent Failure

**Pass rate: 0/5**

**Representative run (sample_02) — function not found, returned original values unchanged:**

```
Warning message:
! Function 'get_unified_time_unit' not found in admiral package. Returning original values.

Output files successfully created:
  - outputs/time_unit_mapped.csv

First few rows of processed data:
# A tibble: 1 × 4
  time_unit  standardized_term processing_date package_version
  <chr>      <chr>             <date>          <pckg_vrs>
1 test_value test_value        2026-05-14      1.4.1
```

The agent wrapped the call in a `tryCatch`, swallowed the namespace error, and silently returned the input unchanged. The output CSV has four columns instead of two, and the `result` column contains the raw input rather than the standardised term.

**Root cause:** `get_unified_time_unit` is not exported; `admiral::get_unified_time_unit()` throws `"object not found"`. The prompt explicitly provides the workaround (`get("...", envir = asNamespace("admiral"))`), but the model ignores it and substitutes its own normalisation logic or degrades gracefully with wrong output.

| Failure mode | Runs | What happened |
|---|---|---|
| Returned original value unchanged | 3/5 | Model caught the namespace error and fell back to identity |
| Wrong canonical form (`"day"`, `"DAYS"`) | 2/5 | Model used `tolower`/`gsub` instead of the function |
