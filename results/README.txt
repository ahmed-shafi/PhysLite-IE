PhysLite-IE Protocol A - run artifacts (organized 2026-09-20)

Canonical run: corrected-2026-09-19c-generalization
  Kaggle run finished 2026-09-20 (v5); seed 42; ~2.8 h on GPU T4 x2.
  Adds the two generalization tasks to the executed same-building protocol:
  unseen-building (70/15/15 at the building level) and unseen-site
  (leave-one-site-out, 14 sites). Executed-run tables replicate the
  2026-09-10/19 runs bit-for-bit (latency drifts with wall-clock only).

Layout
  experiment_manifest.json, preregistration_config.json, site_climate_map.csv
  tables/                 - all result CSVs (ground truth for the paper tables),
                            incl. unseen_building_results.csv,
                            unseen_site_results_per_fold.csv,
                            unseen_site_results_pooled.csv,
                            unseen_generalization_paired_tests.csv,
                            unseen_building_bootstrap_ci.csv, bc_predictions.npz
  figures/                - vector PDFs (paper uses these) + PNG twins
  models/                 - deployed model artifacts

NOTE (recovery): the v5 notebook crashed in the export cell AFTER all CSVs
were written, so final_results/ (summary.json + collected-results JSONs)
was never built for this run; those JSONs exist only in the archived 19b
folder, whose executed-protocol content is identical. All paper numbers
come from tables/*.csv here.

Raw provenance: outputs/archive/results_2026-09-20_kaggle_export_19c.zip.
Superseded runs: outputs/archive/corrected-2026-09-19b-review-baselines/ and
results_2026-09-19_kaggle_export_replication.zip (bit-identical replication).

Key numbers (RMSE_log): same-building deployed LightGBM+Lag 0.14040;
unseen-building 0.16407; unseen-site 0.21547. Unseen-building paired tests
(Holm): vs GRU p=0.012, vs no-lag p<0.001, vs persistence p=0.057 (n.s.).
Unseen-site: vs persistence p=0.91 (n.s.), vs GRU p=0.91 (n.s.), vs no-lag
p=0.025. Paper main.tex matches these tables.
