# Task 05 — `r2rtf/rtf_colheader`

**Package:** r2rtf | **Function:** `rtf_colheader()`
**Domain:** Clinical report generation — RTF column header formatting

---

## Why This Task Matters

`rtf_colheader()` adds **column header rows** to an RTF table — a standard requirement in clinical study report tables. The function parses a pipe-delimited string `"Label A | Label B | Label C"` into individual column labels and stores them as a structured attribute. In practice, clinical tables often use spanning headers (one header spanning multiple columns), requiring careful `col_rel_width` alignment between the colheader and body.

---

## Prompt

```
Write R code to add a column header row to a data frame using r2rtf.
At the beginning, load:
  library(rlang)   # required for r2rtf internal operators
  library(r2rtf)

Input file: inputs/dataset.tsv
Columns: USUBJID, TRT01P, AGE, SEX, BMIBL

Pipeline:
  df <- read.delim("inputs/dataset.tsv")

  result <- df |>
    rtf_page() |>
    rtf_colheader(
      colheader     = "Subject ID | Treatment | Age | Sex | BMI",
      col_rel_width = c(3, 3, 1, 1, 1),
      border_top    = "single",
      border_bottom = "single"
    )

The colheader string is split on " | " and stored as:
  attr(result, "rtf_colheader")[[1]]   — a 1-row data frame with columns X1, X2, ..., X5
  Each Xi contains the parsed label for that column position.

Extract and reshape:
  header_df <- attr(result, "rtf_colheader")[[1]]

  output <- data.frame(
    position = seq_len(ncol(header_df)),
    col_name = colnames(df),
    label    = as.character(t(header_df))
  )

Save to outputs/result.csv (row.names = FALSE).
```

---

## Input

Same `inputs/dataset.tsv` as Task 04.

---

## Reference Solution

```r
library(rlang)
library(r2rtf)

dir.create("outputs", showWarnings = FALSE)

df <- read.delim("inputs/dataset.tsv")

result <- df |>
  rtf_page() |>
  rtf_colheader(
    colheader     = "Subject ID | Treatment | Age | Sex | BMI",
    col_rel_width = c(3, 3, 1, 1, 1),
    border_top    = "single",
    border_bottom = "single"
  )

header_df <- attr(result, "rtf_colheader")[[1]]

output <- data.frame(
  position = seq_len(ncol(header_df)),
  col_name = colnames(df),
  label    = as.character(t(header_df))
)

write.csv(output, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"position","col_name","label"
1,"USUBJID","Subject ID"
2,"TRT01P","Treatment"
3,"AGE","Age"
4,"SEX","Sex"
5,"BMIBL","BMI"
```

Output shape: **(5, 3)**

---

## Agent Run Results

**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **k = 3**
**Pass rate: 0/3**

| Sample | Status | Root cause |
|--------|--------|------------|
| sample_00 | NO_OUTPUT | `rtf_doc()` does not exist → `could not find function "rtf_doc"` crash |
| sample_01 | NO_OUTPUT | Agent emitted bash exploration text; `unexpected symbol` parse error in R |
| sample_02 | NO_OUTPUT | Used `rtf::` package (different CRAN package); tried to write to read-only `inputs/` dir → crash |

### Agent code — sample 0

```r
library(r2rtf)
dir.create('outputs', showWarnings = FALSE)

# Reads actual input (good), but calls hallucinated functions:
rtf <- rtf_doc()            # ← does not exist
rtf <- rtf %>% rtf_title("Clinical Trial Results Summary") %>%
               rtf_subtitle(paste("Generated on:", Sys.Date()))
rtf <- rtf %>%
  rtf_table(data) %>%       # ← does not exist
  rtf_colheader(
    colheader = c("Patient ID", "Age", ...),
    col_rel_width = c(1.2, 0.8, ...)
  )
rtf_encode(rtf, output_file)  # ← wrong signature; rtf_encode() returns RTF text, not write-to-file
```

### Agent code — sample 1

```
# The agent output was not valid R code — it emitted shell exploration commands:
#
#   cat("Current directory contents:\n")
#   system("ls -la")
#   system("ls -la inputs/ 2>/dev/null || echo 'No inputs directory found'")
#   tryCatch({ library(r2rtf); cat("r2rtf package is available\n") }, ...)
#
# No usable R script was produced.
```

### Agent code — sample 2

```r
# Uses 'rtf' package (a different, unrelated package), not 'r2rtf'
library(r2rtf)

# Creates sample data and writes it as inputs/clinical_data.csv (ignores actual inputs/dataset.tsv)
rtf_doc <- rtf::rtf_document(          # ← rtf:: not r2rtf::
  title = "Clinical Trial Results",
  output = "outputs/clinical_report.rtf"
)
col_headers <- r2rtf::rtf_colheader(   # ← rtf_colheader used as standalone; wrong approach
  table_data,
  colheader = c("Patient ID", "Age", "Gender", ...),
  col_rel_width = c(1, 1, 1, ...)
)
rtf_doc <- rtf::rtf_add_table(rtf_doc, table_data, header = TRUE, colheader = col_headers, ...)
rtf::rtf_write(rtf_doc)                # ← wrong package namespace

# No result.csv created; never extracted attr(result, "rtf_colheader")
```

### Failure patterns

| Error | Description |
|-------|-------------|
| **Sample 0: hallucinated API** | `rtf_doc()`, `rtf_table()` do not exist; misused `rtf_encode()` |
| **Sample 1: no R code produced** | Agent emitted bash exploration commands (same pattern as task_03 sample_0) |
| **Sample 2: wrong package** | Used `rtf::` namespace (a different CRAN package) instead of `r2rtf::` |
| **All: missed attribute extraction** | The task requires `attr(result, "rtf_colheader")[[1]]` to get the 1-row header data frame with X1–X5 columns — no sample attempted this |
| **All: wrong output format** | All samples attempted to write an `.rtf` file; task requires `outputs/result.csv` with position/col_name/label columns |
