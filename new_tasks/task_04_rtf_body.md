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

## Agent Failure (biomni / deepseek-v3.2-exp)

**Pass rate: 0/3**

**Representative failure (sample_00):**

```r
library(r2rtf)

# Agent invented a document-object API that doesn't exist in r2rtf
rtf <- rtf_new()                           # ← function does not exist
rtf <- rtf %>% rtf_title("Report")        # ← does not exist
rtf <- rtf %>% rtf_body(tbl = data, ...)  # ← wrong usage; rtf_body() is called on the df, not an rtf object
rtf <- rtf %>% rtf_page_footer(...)       # ← does not exist

rtf_write(rtf, file = "output.rtf")       # ← does not exist
```

**Error:** `could not find function "rtf_new"` — crashed on the first line.

**Root cause:** The agent assumed r2rtf follows a document-builder pattern (create object → chain methods → write). The actual API is attribute-accumulation: `df |> rtf_page() |> rtf_body(...)` attaches formatting metadata as attributes on the data frame. The task additionally requires extracting those attributes (`attr(result, "text_justification")`) and writing them as a CSV — a step no agent attempted.
