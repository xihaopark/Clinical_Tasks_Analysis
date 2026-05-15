# Case 01 — `admiral/derive_var_pchg`

**Task ID:** `pharmaverse/admiral/derive_var_pchg`
**Package:** admiral | **Track:** clinical_pilot | **Level:** L1
**Agent:** biomni | **Model:** deepseek/deepseek-v3.2-exp | **Pass rate:** 0/5

---

## Prompt (verbatim)

```
Derive **percent change from baseline** (`PCHG`). Load `library(admiral)`.

**Inputs:** `inputs/dataset.tsv` — ensure **`AVAL`** and **`BASE`** exist
(reference fills from a value column if needed).

**Computation:** **`admiral::derive_var_pchg(dataset)`**.

**Required outputs for grading (exact paths):**
- `outputs/result.csv`

Create `outputs/` with `dir.create('outputs', showWarnings=FALSE)` if needed.
For CSV use `write.csv(..., row.names=FALSE)`.
```

---

## Input

**`inputs/dataset.tsv`**

```
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

The dataset does not have `AVAL` or `BASE` columns. The reference solution synthesises them: `AVAL` from the `value` column, `BASE` as the first `AVAL` value (i.e. `AVAL[1]`). Any model that constructs `BASE` differently will get wrong `PCHG` values.

---

## Reference Solution

```r
suppressPackageStartupMessages(library(admiral))

dataset <- read.delim("inputs/dataset.tsv", check.names = FALSE, stringsAsFactors = FALSE)

if (!"AVAL" %in% names(dataset)) {
  val_col <- if ("value" %in% names(dataset)) "value" else names(dataset)[2]
  dataset$AVAL <- as.numeric(dataset[[val_col]])
}
if (!"BASE" %in% names(dataset)) dataset$BASE <- dataset$AVAL[1]

result <- admiral::derive_var_pchg(dataset)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

---

## Expected Output (`outputs/result.csv`)

```
"id","value","group","category","AVAL","BASE","PCHG"
1,10.5,"A","Type1",10.5,10.5,0
2,20.3,"B","Type2",20.3,10.5,93.3333333333333
3,30.7,"A","Type1",30.7,10.5,192.380952380952
4,40.2,"B","Type2",40.2,10.5,282.857142857143
5,50.9,"A","Type1",50.9,10.5,384.761904761905
```

Output shape: **(5, 7)**. Numeric columns are compared with tolerance; the values above are full-precision as written by `write.csv()`.

---

## Agent Failure

**Pass rate: 0/5**

**Representative run (sample_00) — wrong `BASE` per group:**

The agent assigned `BASE` as the first `AVAL` within each group (`A` → 10.5, `B` → 20.3), not the global first row. It also saved to `outputs/result_with_pchg.csv` instead of `outputs/result.csv`.

**Actual stdout:**

```
Data prepared for PCHG calculation:
# A tibble: 5 × 6
     id value group category  BASE  AVAL
  <dbl> <dbl> <chr> <chr>    <dbl> <dbl>
1     1  10.5 A     Type1     10.5  10.5
2     2  20.3 B     Type2     20.3  20.3   ← BASE = 20.3 (wrong; should be 10.5)
3     3  30.7 A     Type1     10.5  30.7
4     4  40.2 B     Type2     20.3  40.2   ← PCHG wrong because BASE wrong
5     5  50.9 A     Type1     10.5  50.9

Results saved to: outputs/result_with_pchg.csv   ← wrong filename
```

**Root cause:** The model constructs `BASE` per group rather than using the global first row `AVAL[1]`. Additionally, it outputs to a non-standard filename, so grading cannot find `outputs/result.csv`.

| Failure mode | Runs | What happened |
|---|---|---|
| Wrong `BASE` (per-group or self-value) | 3/5 | `PCHG` values numerically incorrect |
| Wrong output filename | 5/5 | All runs saved to `result_with_pchg.csv` or `dataset_with_pchg.csv` |
