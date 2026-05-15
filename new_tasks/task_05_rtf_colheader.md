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

## Agent Failure (biomni / deepseek-v3.2-exp)

**Pass rate: 0/3**

**Representative failure (sample_00) — invented `rtf_doc()`, created synthetic data:**

```r
rtf <- rtf_doc()   # ← does not exist
rtf <- rtf %>%
  rtf_table(data) %>%
  rtf_colheader(
    colheader = c("Patient ID", "Age", ...),
    col_rel_width = c(1.2, 0.8, ...)
  )
rtf_encode(rtf, output_file)
```

**Actual stdout:**
```
No CSV files found in inputs directory. Creating sample data.

First few rows of data:
  PatientID Age Gender Treatment ...

Creating RTF document with column headers...
```

**Actual stderr:**
```
Error in rtf_doc() : could not find function "rtf_doc"
Calls: main
Execution halted
```

The agent ignored `inputs/dataset.tsv`, generated its own synthetic data, then crashed immediately on `rtf_doc()`.

**sample_01** leaked thinking text into the R script:
```
Error: unexpected symbol in:
"**Thinking process:**
I need to write an R script that uses the `r2rtf"
Execution halted
```

**Root cause:** Same misunderstanding as task_04 — the agent invented a document-builder API. The actual usage is `df |> rtf_page() |> rtf_colheader("Label A | Label B | ...")`, which attaches a structured attribute; labels are extracted via `attr(result, "rtf_colheader")[[1]]` — a 1-row data frame with columns `X1, X2, ..., Xn`.
