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

## Failure Analysis

`get_unified_time_unit` is a **non-exported internal function** of admiral. Calling `admiral::get_unified_time_unit()` directly throws an error. The correct access pattern is:

```r
get("get_unified_time_unit", envir = asNamespace("admiral"))
```

The prompt explicitly states this. The model ignores it.

| Failure mode | Runs | What happened |
|---|---|---|
| Wrong canonical form | 3/5 | Model invented its own normalisation (`tolower`, `gsub`) — returned `"day"` or `"DAYS"` instead of `"days"` |
| `NO_OUTPUT` (namespace error) | 2/5 | Model called `admiral::get_unified_time_unit()` directly → error |

In both cases the model substituted its own logic for the function it was explicitly told to call.
