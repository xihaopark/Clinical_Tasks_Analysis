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

## Failure Analysis

The model fails to satisfy the **preprocessing contract**: it either does not construct `AVAL`/`BASE`, or constructs `BASE` differently from the reference.

| Failure mode | Runs | What happened |
|---|---|---|
| `BASE` computed as mean or self-value | 3/5 | `PCHG` values numerically wrong |
| `Missing required columns: AVAL, BASE` | 2/5 | Passed raw dataset without synthesising columns |

The task tests whether the model will follow the explicit preprocessing instruction in the prompt. When `AVAL` and `BASE` are absent, `admiral::derive_var_pchg()` errors. The model either ignores this requirement or constructs `BASE` incorrectly (e.g. taking each row's own AVAL as its baseline).
