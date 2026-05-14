# Task 04 — `r2rtf/rtf_body`

**Package:** r2rtf | **Function:** `rtf_body()`
**Domain:** Clinical report generation — RTF table body formatting

---

## Why This Task Matters

`rtf_body()` is the **core function** of the r2rtf pipeline. Every RTF table in a clinical study report passes through it. It attaches formatting attributes (column widths, text justification, font size, borders) to a data frame that `write_rtf()` later serialises to RTF. Understanding the attribute-accumulation pattern — where r2rtf functions add metadata to the data frame rather than transforming it — is essential for building regulatory-grade clinical reports.

---

## Prompt

```
Write R code to apply body formatting attributes to a data frame using r2rtf.
At the beginning, load:
  library(rlang)   # required for r2rtf internal operators
  library(r2rtf)

Input file: inputs/dataset.tsv
Columns: USUBJID, TRT01P, AGE, SEX, BMIBL

Pipeline:
  df <- read.delim("inputs/dataset.tsv")

  result <- df |>
    rtf_page() |>
    rtf_body(
      col_rel_width      = c(3, 3, 1, 1, 1),
      text_justification = c("l", "l", "c", "c", "c"),
      text_font_size     = 9
    )

rtf_body() attaches formatting as attributes on the data frame (not nested in "rtf_body").
Key attribute access:
  attr(result, "col_rel_width")       — numeric vector, length = ncol(df)
  attr(result, "text_justification")  — character vector, length = nrow(df) * ncol(df),
                                        stored COLUMN-MAJOR (col 1 all rows, then col 2, ...)
  attr(result, "text_font_size")      — numeric vector (column-major, same length)
  attr(result, "border_left")         — character vector (column-major)
  attr(result, "border_right")        — character vector (column-major)

To extract one value per column from text_justification:
  tj <- attr(result, "text_justification")
  per_col_tj <- tj[seq(1, length(tj), by = nrow(df))]   # stride = nrow(df)

Build output data frame (one row per column):
  output <- data.frame(
    col_name           = colnames(df),
    col_rel_width      = attr(result, "col_rel_width"),
    text_justification = per_col_tj,
    text_font_size     = attr(result, "text_font_size")[1],
    border_left        = attr(result, "border_left")[1],
    border_right       = attr(result, "border_right")[1]
  )

Save to outputs/result.csv (row.names = FALSE).
```

---

## Input

**`inputs/dataset.tsv`** (first 5 columns of r2rtf_adsl, 10 rows)

```
USUBJID	TRT01P	AGE	SEX	BMIBL
01-701-1015	Placebo	63	F	25.1
01-701-1023	Placebo	64	M	30.4
...
```

---

## Reference Solution

```r
library(rlang)
library(r2rtf)

dir.create("outputs", showWarnings = FALSE)

df <- read.delim("inputs/dataset.tsv")

result <- df |>
  rtf_page() |>
  rtf_body(
    col_rel_width      = c(3, 3, 1, 1, 1),
    text_justification = c("l", "l", "c", "c", "c"),
    text_font_size     = 9
  )

tj <- attr(result, "text_justification")
per_col_tj <- tj[seq(1, length(tj), by = nrow(df))]

output <- data.frame(
  col_name           = colnames(df),
  col_rel_width      = attr(result, "col_rel_width"),
  text_justification = per_col_tj,
  text_font_size     = attr(result, "text_font_size")[1],
  border_left        = attr(result, "border_left")[1],
  border_right       = attr(result, "border_right")[1]
)

write.csv(output, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"col_name","col_rel_width","text_justification","text_font_size","border_left","border_right"
"USUBJID",3,"l",9,"single","single"
"TRT01P",3,"l",9,"single","single"
"AGE",1,"c",9,"single","single"
"SEX",1,"c",9,"single","single"
"BMIBL",1,"c",9,"single","single"
```

Output shape: **(5, 6)**

---

## Agent Run Results

**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **k = 3**
**Pass rate: 0/3**

| Sample | Status | Root cause |
|--------|--------|------------|
| sample_00 | NO_OUTPUT | `rtf_new()` does not exist → script crashed immediately |
| sample_01 | NO_OUTPUT | Read `inputs/strata.tsv` (rate_compare_sum inputs) — wrong task context; file not found → crash |
| sample_02 | NO_OUTPUT | Read `inputs/n1.tsv`, `inputs/x0.tsv`, ... (rate_compare_sum inputs) — wrong task context; files not found |

### Agent code — sample 0

```r
library(r2rtf)
dir.create('outputs', showWarnings = FALSE)

# Reads the input file correctly, but then calls non-existent functions
data <- read_input_data()   # custom helper that reads inputs/

# Hallucinated r2rtf API:
rtf <- rtf_new()            # ← does not exist; r2rtf uses pipe-based attribute accumulation
rtf <- rtf %>% rtf_title(title = "Data Analysis Report", subtitle = ...)
rtf <- rtf %>% rtf_body(tbl = data, col_rel_width = rep(1, ncol(data)))
rtf <- rtf %>% rtf_page_header("Summary Statistics") %>% ...
rtf <- rtf %>% rtf_page_footer(paste("Page", rtf_page_number()))

rtf_write(rtf_doc, file = output_file)    # ← rtf_write() doesn't work this way in r2rtf
```

### Agent code — sample 1

```r
# Defines %||% operator (correct insight that r2rtf needs it)
`%||%` <- function(x, y) if (is.null(x)) y else x

library(r2rtf)
library(dplyr)
dir.create("outputs", showWarnings = FALSE)

# Reads rate_compare_sum task inputs instead of the correct dataset.tsv:
strata <- read.delim("inputs/strata.tsv", header = FALSE, skip = 1)[[1]]
n0     <- read.delim("inputs/n0.tsv",     header = FALSE, skip = 1)[[1]]
# ... (builds a data frame from rate_compare_sum inputs, not from dataset.tsv)

df_rtf <- df %>%
  rtf_body(col_rel_width = c(2,1,1,1,1,1,1), text_justification = c("l",rep("c",6)), ...)

# Uses rtf_encode() / write_rtf() — these may work but reads wrong inputs
rtf <- rtf_encode(df_rtf)
write_rtf(rtf, file = "outputs/stratified_table.rtf")
```

### Agent code — sample 2

```r
library(r2rtf)
dir.create("outputs", showWarnings = FALSE)

# Reads TSV files from inputs/, but then uses incorrect rtf_body() call
tbl_with_attrs <- rtf_body(
  tbl                = data,
  col_rel_width      = rep(1, ncol(data)),
  as_colheader       = TRUE,    # ← not a valid rtf_body() parameter
  border_left        = "single",
  border_right       = "single",
  border_top         = "single",
  border_bottom      = "single",
  text_justification = rep("c", ncol(data)),
  text_font_size     = 10,
  last_row           = TRUE     # ← not a valid parameter
)
rtf_doc <- rtf_encode(tbl_with_attrs)
write_rtf(rtf_doc, file = output_path)
# No result.csv created — never reads attributes or builds the output data frame
```

### Failure patterns

| Error | Description |
|-------|-------------|
| **Sample 0: hallucinated API** | `rtf_new()`, `rtf_title()`, `rtf_page_header()`, `rtf_page_footer()` do not exist |
| **Sample 1: wrong input files** | Read `strata.tsv`, `n0.tsv`, ... from the rate_compare_sum task instead of `dataset.tsv`; mixed up task contexts |
| **Sample 2: wrong output format** | Wrote an RTF file instead of the required `result.csv`; never extracted attributes into a data frame |
| **All: missed the attribute-extraction step** | The task requires reading `attr(result, "text_justification")` with column-major stride — no sample attempted this |
