# Post-Shift Sweep Stability Summary

This summary uses the rerun after applying the short-day adjustment: day 327 daily volatility was multiplied by `np.sqrt(391 / 211)` for every stock column before feature/target construction.

## Run Metadata

| field | value |
| --- | --- |
| artifact directory | artifacts/eval_rv_post_shift_sweep/run_20260522_232728 |
| valid start days | 92, 151, 156 |
| skipped start days | 215 (insufficient pre-test rows, rows=59); 277 (insufficient pre-test rows, rows=17) |
| step size | 42 trading rows (~2 months) |
| test start day | 300 |
| stocks | ra, rb, rc, rd, re, rf |
| score / param / coefficient rows | 378 / 360 / 576 |
| figures | 360 |
| errors | 0 |

## Definitions

- `CV pick` means the candidate with the lowest validation QLIKE within one `(start day, stock)` panel.
- `test-best` means the candidate with the lowest held-out test QLIKE in that panel. This is diagnostic only; it should not be used for model selection.
- Model/feature families are normalized across stocks by replacing the target stock token with `r`, e.g. `ra_avg42` and `rb_avg42` become `r_avg42`.
- Rank metrics compare candidates inside each stock/run panel; rank 1 is best. Ranks are more comparable across stocks than raw QLIKE.

## Main Takeaways

- Every stock has a `2/3` exact CV-selection stability: the day-151 and day-156 runs choose the same family, while day 92 often chooses a different one.
- The validation winners are not universal across stocks: `r_avg42` wins for `ra`/`rf`, `r_ewma10` wins for `rb`/`rc`, log-EWMA ridge wins for `rd`, and log-EWMA/wlog ridge wins for `re`.
- The held-out test-best families are more concentrated in long EWMA baselines: `r_ewma63` is test-best in 9 of 18 stock-run panels and `r_ewma21` in 5 of 18.
- This validation/test mismatch is worth flagging in the report: the CV split often favors shorter or average-based features, while the fixed test window favors longer EWMAs for several stocks.
- Among CV-selected regularized models, alpha is not fully stable: `rd` log-EWMA ridge moves from `alpha=1` to `alpha=0.01`; `re` wlog-EWMA ridge moves from `alpha=100` to `alpha=50`.

## 1. Stock Stability Across Runs

| stock | CV picks by start | modal CV pick | selected test QLIKE | test-best by start | best mean CV-rank family |
| --- | --- | --- | --- | --- | --- |
| ra | 92: r_avg21; 151: r_avg42; 156: r_avg42 | r_avg42 (2/3) | 92=0.115, 151=0.249, 156=0.249 | 92: r_ewma63; 151: r_ewma63; 156: r_ewma63 | r_avg42 (rank 1.3, sd 0.6) |
| rb | 92: r_avg10; 151: r_ewma10; 156: r_ewma10 | r_ewma10 (2/3) | 92=0.073, 151=0.018, 156=0.018 | 92: r_ewma21; 151: r_ewma21; 156: r_ewma21 | r_ewma10 (rank 2.7, sd 2.9) |
| rc | 92: r_avg10; 151: r_ewma10; 156: r_ewma10 | r_ewma10 (2/3) | 92=0.077, 151=0.061, 156=0.061 | 92: r_ewma63; 151: r_ewma63; 156: r_ewma63 | r_ewma5 (rank 2.7, sd 1.2) |
| rd | 92: r_avg21; 151: ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21; 156: ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 (2/3) | 92=0.250, 151=0.442, 156=0.526 | 92: log HAR (Ridge scaled): log_r, log_r_avg5, log_r_avg21; 151: r_avg63; 156: r_avg63 | r_avg21 (rank 5.0, sd 3.6) |
| re | 92: ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21; 151: ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63; 156: ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 (2/3) | 92=0.0020, 151=0.0010, 156=0.0010 | 92: r_ewma63; 151: r_ewma63; 156: r_ewma63 | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 (rank 1.0, sd 0.0) |
| rf | 92: GARCH (pass): r_garch_fwd; 151: r_avg42; 156: r_avg42 | r_avg42 (2/3) | 92=0.043, 151=0.074, 156=0.074 | 92: wlog HAR (Ridge scaled): log_r_avg5, log_r_avg10, log_r_avg21, log_r_avg42, log_r_avg63; 151: r_ewma21; 156: r_ewma21 | r_avg42 (rank 1.3, sd 0.6) |

Interpretation: the later two starts are stable for all stocks, but the earlier start day 92 changes the selected family for each stock. `re` is the strongest validation case by rank stability; `rd` is the least comfortable because the selected log-EWMA model has volatile validation rank across starts even though it wins the later two starts.

## 2. Model/Feature Stability Across Stocks

### CV-Selected Family Matrix

Counts are how often each family is selected as CV-best over the three runs for each stock.

| family | ra | rb | rc | rd | re | rf | total |
| --- | --- | --- | --- | --- | --- | --- | --- |
| r_ewma10 | 0 | 2 | 2 | 0 | 0 | 0 | 4 |
| r_avg42 | 2 | 0 | 0 | 0 | 0 | 2 | 4 |
| ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 0 | 0 | 0 | 2 | 1 | 0 | 3 |
| r_avg21 | 1 | 0 | 0 | 1 | 0 | 0 | 2 |
| ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | 0 | 0 | 0 | 0 | 2 | 0 | 2 |
| r_avg10 | 0 | 1 | 1 | 0 | 0 | 0 | 2 |
| GARCH (pass): r_garch_fwd | 0 | 0 | 0 | 0 | 0 | 1 | 1 |

### Most Frequently CV-Selected Families

| family | CV selections | stocks selected | mean CV rank | sd CV rank | mean test rank |
| --- | --- | --- | --- | --- | --- |
| r_avg42 | 4/18 | ra, rf | 4.9 | 3.9 | 9.8 |
| r_ewma10 | 4/18 | rb, rc | 7.2 | 4.8 | 6.3 |
| ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 3/18 | rd, re | 10.6 | 7.2 | 9.3 |
| r_avg21 | 2/18 | ra, rd | 6.8 | 4.3 | 9.5 |
| r_avg10 | 2/18 | rb, rc | 8.4 | 5.1 | 11.4 |
| ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | 2/18 | re | 14.4 | 8.4 | 14.0 |
| GARCH (pass): r_garch_fwd | 1/18 | rf | 11.9 | 5.9 | 11.1 |
| r_avg63 | 0/18 | - | 4.9 | 2.5 | 12.2 |
| r_ewma5 | 0/18 | - | 8.2 | 5.0 | 10.3 |
| r_ewma21 | 0/18 | - | 8.7 | 3.5 | 4.3 |

### Best Mean Validation-Rank Families

| family | mean CV rank | sd CV rank | median CV rank | mean test rank | CV selections |
| --- | --- | --- | --- | --- | --- |
| r_avg63 | 4.9 | 2.5 | 4.5 | 12.2 | 0/18 |
| r_avg42 | 4.9 | 3.9 | 4.0 | 9.8 | 4/18 |
| r_avg21 | 6.8 | 4.3 | 6.0 | 9.5 | 2/18 |
| r_ewma10 | 7.2 | 4.8 | 6.0 | 6.3 | 4/18 |
| r_ewma5 | 8.2 | 5.0 | 9.0 | 10.3 | 0/18 |
| r_avg10 | 8.4 | 5.1 | 8.5 | 11.4 | 2/18 |
| r_ewma21 | 8.7 | 3.5 | 8.0 | 4.3 | 0/18 |
| log HAR (Ridge scaled): log_r, log_r_avg5, log_r_avg21 | 8.8 | 5.1 | 10.0 | 5.6 | 0/18 |
| HAR (Ridge scaled): r, r_avg5, r_avg21 | 9.7 | 5.5 | 10.0 | 6.4 | 0/18 |
| ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 10.6 | 7.2 | 14.5 | 9.3 | 3/18 |

Interpretation: `r_avg42` and `r_avg63` have the best average validation ranks across stock/run panels, but only `r_avg42` is repeatedly selected. `r_ewma10` is selected just as often as `r_avg42`, concentrated in `rb` and `rc`. No single family is stable across all stocks.

### Test-Best Family Matrix

Counts are how often each family is test-best over the three runs for each stock.

| family | ra | rb | rc | rd | re | rf | total |
| --- | --- | --- | --- | --- | --- | --- | --- |
| r_ewma63 | 3 | 0 | 3 | 0 | 3 | 0 | 9 |
| r_ewma21 | 0 | 3 | 0 | 0 | 0 | 2 | 5 |
| r_avg63 | 0 | 0 | 0 | 2 | 0 | 0 | 2 |
| log HAR (Ridge scaled): log_r, log_r_avg5, log_r_avg21 | 0 | 0 | 0 | 1 | 0 | 0 | 1 |
| wlog HAR (Ridge scaled): log_r_avg5, log_r_avg10, log_r_avg21, log_r_avg42, log_r_avg63 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |

## Regularization And Coefficient Notes

### Alpha For CV-Selected Ridge Models

| start | stock | CV-selected family | alpha | log target |
| --- | --- | --- | --- | --- |
| 151 | rd | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 1.000 | True |
| 156 | rd | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 0.010 | True |
| 92 | re | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | 50.000 | True |
| 151 | re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | 100.000 | True |
| 156 | re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | 50.000 | True |

### Coefficients For CV-Selected Ridge Models

| stock | family | feature | n | mean coef | range | same sign |
| --- | --- | --- | --- | --- | --- | --- |
| rd | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r | 2 | 0.0107 | 0.0061 to 0.0152 | yes |
| rd | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r_ewma21 | 2 | 0.3073 | 0.2902 to 0.3243 | yes |
| rd | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r_ewma5 | 2 | -0.2095 | -0.2289 to -0.1901 | yes |
| re | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r | 1 | -0.0000 | -0.0000 to -0.0000 | yes |
| re | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r_ewma21 | 1 | -0.0154 | -0.0154 to -0.0154 | yes |
| re | ewma log HAR (Ridge scaled): log_r, log_r_ewma5, log_r_ewma21 | log_r_ewma5 | 1 | 0.0087 | 0.0087 to 0.0087 | yes |
| re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | log_r_ewma10 | 2 | -0.0007 | -0.0008 to -0.0006 | yes |
| re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | log_r_ewma21 | 2 | -0.0068 | -0.0083 to -0.0052 | yes |
| re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | log_r_ewma42 | 2 | -0.0028 | -0.0034 to -0.0022 | yes |
| re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | log_r_ewma5 | 2 | 0.0033 | 0.0022 to 0.0044 | yes |
| re | ewma wlog HAR (Ridge scaled): log_r_ewma5, log_r_ewma10, log_r_ewma21, log_r_ewma42, log_r_ewma63 | log_r_ewma63 | 2 | 0.0013 | 0.0007 to 0.0018 | yes |

## Saved Summary Tables

- `cv_selected_summary.csv`
- `test_best_summary.csv`
- `stock_stability_summary.csv`
- `stock_top_ranked_families.csv`
- `candidate_family_stability_summary.csv`
- `cv_selection_matrix_by_stock.csv`
- `test_best_matrix_by_stock.csv`
- `cv_selected_params.csv`
- `cv_selected_coefficient_stability.csv`
