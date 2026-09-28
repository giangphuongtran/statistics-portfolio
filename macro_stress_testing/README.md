# Macro Stress Testing Engine for Credit Portfolio Valuation Impact

**Repo:** [statistics-portfolio/macro_stress_testing](https://github.com/giangphuongtran/statistics-portfolio/tree/main/macro_stress_testing)

Python notebook linking **US macro risk factors (FRED)** to **credit-card charge-off rates**, then translating stressed rates into an illustrative **expected-loss (EL)** impact on a $1bn book.

## Business question

If macro conditions deteriorate (GFC-like, COVID-like, or a designed adverse multi-factor shock), how much could credit-card charge-off rates — and therefore expected loss on an illustrative book — move versus today’s level?

Charge-off % is analyst language; **$ millions of ΔEL** is senior language. This project bridges both.

## What the notebook does

[`test.ipynb`](test.ipynb) is the full analysis (no package imports from this folder):

1. **Data lineage** — pull FRED series (or use cached CSVs in `data/`), align to quarter-end (`resample("QE").last()`), build GDP YoY.
2. **EDA** — trends and crisis windows (GFC, COVID).
3. **Time-series diagnostics** — ADF / KPSS before modeling.
4. **Lag design** — charge-offs react with delay; ARDL uses lagged CO + lagged macros.
5. **Primary model** — explainable ARDL OLS (train ≤ 2018; OOT after).
6. **Diagnostics** — VIF, residuals, Ljung-Box; sklearn OLS replication (effective challenge).
7. **Challenger / benchmarks** — Gradient Boosting vs naive-last / naive-mean on OOT MAE.
8. **Stress** — historical one-step and multi-quarter **path peak**; single-factor and combined hypotheticals.
9. **Valuation bridge** — EL and ΔEL on assumed $1bn EAD; tornado of which shock hurts most.

## Measured results (do not invent beyond these)

| Metric | Value |
|--------|--------|
| OOT MAE (post-2018) | ARDL **0.323** vs naive-last **0.730** (GBM challenger 0.455) |
| Train R² | ≈ **0.915** |
| Latest charge-off | ≈ **3.82%** |
| GFC path peak | ≈ **7.83%** → ΔEL ≈ **+$40.1m** on $1bn EAD |
| COVID path peak | ≈ **5.30%** → ΔEL ≈ **+$14.8m** |
| Combined adverse (one-step) | ≈ **4.64%** → ΔEL ≈ **+$8.2m** |

Mark all dollar figures as **illustrative** — not a real bank P&L.

## One-step vs path peak (how to read stress)

| Mode | Mechanics | Use when |
|------|-----------|----------|
| **One-step** | Shocked / crisis-mean macros + **latest actual** CO as own lag → one prediction | Sensitivity near today’s book state |
| **Path peak** | Walk crisis macros quarter by quarter; feed each prediction into the next own lag; take the **max** CO | “What if a crisis unfolds over time?” |

Own-lag coefficient ≈ **0.91**, so path peaks (e.g. GFC 7.83%) are much larger than one-step (≈4.89%). Crises compound; a single-shot stress understates multi-quarter loss.

**Tornado (single-factor):** near today’s baseline, **GDP YoY** and **credit spread** move EL most; VIX is weak; unemployment is poorly identified in this OLS — do not oversell it.

## How to run

```bash
cd macro_stress_testing
# use the statistics-portfolio venv / jupyter kernel
jupyter notebook test.ipynb
```

- Live FRED pulls need network access.
- Cached panel and series live in [`data/`](data/) (`quarterly_panel.csv` plus per-series CSVs).

## Layout

```
macro_stress_testing/
  README.md      # this file
  test.ipynb     # full narrative + code
  data/          # cached FRED panel + series
```

## Limitations (honest)

- US aggregate FRED series ≠ your institution’s portfolio mix or underwriting.
- Fixed illustrative EAD (no attrition / origination freeze).
- Linear ARDL may miss nonlinear crisis cliffs.
- Not a CCAR / IFRS 9 staging / IPV / AVA production model.
- OLS primary is for **explainability**; GBM is challenger only.

## Interview one-liner

> I built a governance-ready macro stress engine: ARDL OLS maps lagged unemployment, GDP, rates, spreads, and VIX into charge-off rates; I beat naive benchmarks out-of-time, challenged with GBM, and quantified GFC/COVID/adverse ΔEL on a $1bn book — with diagnostics, replication, and clear limitations in the notebook.
