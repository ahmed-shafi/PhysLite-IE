# PhysLite-IE

A leakage-aware benchmark for building-electricity load prediction on the
[ASHRAE Great Energy Predictor III](https://www.kaggle.com/competitions/ashrae-energy-prediction)
dataset, with pre-registered hypotheses, a validation-only fusion gate,
generalization experiments (unseen buildings, unseen sites), and single-CPU
deployment measurements.

This repository is the reproducibility package for the manuscript
*"PhysLite-IE: A Leakage-Aware Benchmark Protocol for Building Electricity Load Prediction"*
(under review at *Energy and Buildings*). Everything reported in the paper is
reproducible from the notebook in this repository with a fixed seed.

## What the evaluation does

- **Electricity-only benchmark** over a pre-registered sample of 150 GEPIII buildings
  (1,254,311 hourly meter rows, 14 of 16 sites represented; the dataset contains no
  tropical-climate sites, which bounds the scope of any climate-generalization claim).
- **Three evaluation tasks**: same-building chronological forecasting (executed, primary),
  unseen-building generalization (70/15/15 split at the building level), and
  unseen-site generalization (leave-one-site-out) — both generalization tasks executed.
- **Causal lag features only** (lags 1 h / 24 h / 168 h computed from the past), plus an
  optional psychrometric-enthalpy feature set.
- **Pre-registered confirmatory hypotheses** (H1 enthalpy utility, H2 temporal and
  ensemble utility) with validation-only decision rules, recorded before model comparison.
- **A validation-only fusion gate**: a learned combination of the tree and sequence
  models is deployed only if it clears pre-registered improvement thresholds.
- **Paired significance tests** (building- or site-level units, Holm-corrected) and
  bootstrap confidence intervals (building-level for unseen-building transfer).
- **SHAP interpretability** of the deployed model and a **single-CPU latency benchmark**.

## Headline results (seed 42, build `corrected-2026-09-19c-generalization`)

Full metrics: [`results/corrected-2026-09-19c-generalization/tables/global_results.csv`](results/corrected-2026-09-19c-generalization/tables/global_results.csv)

| Model | RMSE (log) | R² (log) | MAE (log) |
|---|---|---|---|
| **LightGBM + Lag Features (deployed)** | **0.14040** | **0.99141** | **0.06254** |
| LightGBM + Enthalpy + Lag | 0.14064 | 0.99138 | 0.06417 |
| XGBoost + Lag Features | 0.14411 | 0.99095 | 0.06648 |
| CatBoost + Lag Features | 0.14504 | 0.99084 | 0.06922 |
| Persistence (lag-1) | 0.15749 | 0.98920 | 0.06299 |
| GRU, 24 h sequence (best of 5 seeds) | 0.18419 | 0.98540 | 0.10917 |
| Persistence (lag-24) | 0.35715 | 0.94444 | 0.12964 |
| Persistence (lag-168) | 0.43816 | 0.91638 | 0.17010 |
| XGBoost (no lags) | 0.50949 | 0.88694 | 0.33627 |
| LightGBM (no enthalpy, no lags) | 0.52275 | 0.88097 | 0.32280 |
| Ridge Regression | 1.32967 | 0.22989 | 0.99342 |

*(table abridged; all 16 rows are in the CSV above)*

**Generalization** (test RMSE, log space; full table in
[`unseen_building_results.csv`](results/corrected-2026-09-19c-generalization/tables/unseen_building_results.csv)
and [`unseen_site_results_pooled.csv`](results/corrected-2026-09-19c-generalization/tables/unseen_site_results_pooled.csv)):

| Model | Unseen building | Unseen site |
|---|---|---|
| LightGBM + Lag Features (deployed) | 0.164 | 0.215 |
| LightGBM + Enthalpy + Lag | 0.162 | 0.218 |
| XGBoost + Lag Features | 0.173 | 0.263 |
| CatBoost + Lag Features | 0.169 | 0.220 |
| GRU, 24 h (ID-free) | 0.189 | 0.209 |
| Persistence (lag-1) | 0.191 | 0.194 |
| LightGBM (no lags) | 1.180 | 1.578 |

**Pre-registered outcomes, all reported as decided by the gate — including the negatives:**

- **H1 (enthalpy features): rejected** — no significant gain once causal lags are
  present (paired p = 0.844).
- **H2 (sequence model / fusion): rejected** — the GRU is significantly worse than the
  deployed model on matched rows under the same-building protocol (paired p = 6.5 × 10⁻¹⁰).
- **Fusion gate: rejected** — the validation optimum (α = 0.06) gives +0.09 % with 60 %
  seed consistency, below the pre-registered thresholds (+1 % / 80 %). The deployed
  model is the plain tree pathway.
- **Dataset climate scope** — the GEPIII sample contains zero tropical-climate rows;
  reported as a dataset-scope finding (not a hypothesis outcome).

**Transfer findings.** Under unseen-building transfer the deployed model degrades
gracefully (0.164 vs. 0.140) and remains significantly ahead of the GRU
(Holm-adjusted p = 0.012). Under the harder unseen-site transfer its edge over
seasonal persistence and the identifier-free GRU disappears (0.215 vs. 0.194 / 0.209;
both n.s.) — exactly what the SHAP-identified reliance on building and site
identifiers predicts. Causal lags remain the dominant signal under both transfers:
the no-lag ablation collapses to 1.180 / 1.578 (Holm-adjusted p ≤ 0.025 vs. deployed).

**Deployment:** the deployed model runs at **0.654 ms mean / 0.720 ms P95 latency**
(≈ 1,529 predictions/s) on a single training-host CPU core
([`local_latency.csv`](results/corrected-2026-09-19c-generalization/tables/local_latency.csv)).

**Takeaway:** under a leakage-aware protocol, the causal feature information — not the
learner — carries the accuracy, and it survives building-level transfer; site-level
transfer is where identifier-reliant models lose their edge.

## Repository structure

```
PhysLite-IE/
├── notebooks/
│   └── physlite_ie_protocol_a.ipynb     # full pipeline: data prep → models → tasks → export
├── results/
│   └── corrected-2026-09-19c-generalization/
│       ├── experiment_manifest.json     # config, environment, timing
│       ├── preregistration_config.json  # pre-registered hypotheses + gate thresholds
│       ├── tables/                      # all result CSVs (metrics, tests, CIs, SHAP,
│       │                                #   unseen-building/site tasks, predictions .npz)
│       ├── figures/                     # vector PDFs (paper figures) + PNG twins
│       └── models/                      # deployed model artifacts (joblib / text / scalers)
├── docs/
│   ├── protocol_A_experiment_design.pdf # experiment design document
│   └── protocol_A_experiment_design.md
├── requirements.txt
└── LICENSE
```

Note: the v5 run's export cell crashed after all CSVs were written (a JSON
serialization bug, fixed in the notebook), so `summary.json` and the
collected-results JSONs are absent from this run's folder; every paper number
comes from `tables/*.csv`. The notebook in `notebooks/` is the fixed build.

## Reproducing the run

The reference environment is a **Kaggle notebook (GPU T4 x2, CPU for the deployed
pathway)**:

1. Create or open a Kaggle notebook and attach the
   [ASHRAE GEPIII dataset](https://www.kaggle.com/competitions/ashrae-energy-prediction).
2. Upload `notebooks/physlite_ie_protocol_a.ipynb`, point the data directory in the
   configuration cell at the attached dataset (`/kaggle/input/...`), and keep the
   GPU accelerator (the GRU branch trains on GPU; the deployed model evaluates on CPU).
3. **Run all cells** (≈ 2.8 h including the generalization tasks). The seed is fixed
   (42); the run banner records the build id (`corrected-2026-09-19c-generalization`).
4. Compare the exported `tables/*.csv` with `results/...` — every reported metric
   reproduces (LightGBM transfer metrics may vary in the 5th decimal under
   multithreaded fitting); only wall-clock timing and the latency benchmark differ
   between machines.

For a local run, install `requirements.txt`, place the GEPIII CSVs somewhere local, and
adjust the data path in the notebook's configuration cell. Python ≥ 3.10 recommended.

## Data availability

The ASHRAE Great Energy Predictor III dataset is publicly available on
[Kaggle](https://www.kaggle.com/competitions/ashrae-energy-prediction) (see also the
[GEPIII paper](https://arxiv.org/abs/2007.06933) describing the competition data and its
leakage history). No new data were collected for this study.

## Authors

Asfar Hossain Sitab · Parmita Hossain Simia · Md Moon Rahman Nayem · Ahmed Abdal Shafi Rasel

Department of Computer Science and Engineering, East West University, Dhaka, Bangladesh

## License

Released under the [MIT License](LICENSE).
