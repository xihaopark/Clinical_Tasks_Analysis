# RBioBench Error Cases — pharmaverse Clinical Track

Error cases and new tasks from the RBioBench `clinical_pilot` track.

---

## Error Cases — Admiral

Failures from benchmark run: `rbiobench_stable_v1`, agent `biomni`, model `deepseek/deepseek-v3.2-exp`, overall pass@1 = **1.57%** across 254 clinical tasks.

Each case includes: the verbatim prompt, input data, reference solution (R code), and expected output.

| # | Task | Package | Failure pattern |
|---|------|---------|----------------|
| [01](cases/case_01_derive_var_pchg.md) | `derive_var_pchg` | admiral | ADaM derivation order — wrong per-subject baseline |
| [02](cases/case_02_get_unified_time_unit.md) | `get_unified_time_unit` | admiral | Non-exported internal function — `asNamespace()` access pattern |
| [03](cases/case_03_event_joined.md) | `event_joined` | admiral | S3 specification object treated as a data join |
| [04](cases/case_04_extend_condition.md) | `extend_condition` | admiral | Expression-level meta-programming treated as row-level data operation |
| [05](cases/case_05_derive_param_bmi.md) | `derive_param_bmi` | admiral | `exprs()` NSE pattern replaced with character vector |

---

## New Tasks — r2rtf and metalite.ae

Five tasks covering functions absent from the current pharma-skills skills.

Each task includes: the full prompt (with sufficient detail for an LLM to complete it), reference solution, input files, and expected output.

| # | Task | Package | Function |
|---|------|---------|---------|
| [01](new_tasks/task_01_rate_compare_sum.md) | `rate_compare_sum` | metalite.ae | Miettinen-Nurminen stratified rate comparison |
| [02](new_tasks/task_02_prepare_ae_summary.md) | `prepare_ae_summary` | metalite.ae | AE summary table — counts and proportions by arm |
| [03](new_tasks/task_03_extend_ae_specific_inference.md) | `extend_ae_specific_inference` | metalite.ae | Risk difference + CI + p-value at SOC/PT level |
| [04](new_tasks/task_04_rtf_body.md) | `rtf_body` | r2rtf | RTF table body — column widths and cell formatting |
| [05](new_tasks/task_05_rtf_colheader.md) | `rtf_colheader` | r2rtf | RTF column header — pipe-delimited label parsing |
