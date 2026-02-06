# Phase 0 Artifacts

This folder contains required outputs for Phase 0 execution.

## Files
- `baseline_report.md`: narrative summary, decisions, and exit-gate signoff.
- `baseline_metrics.csv`: quantitative metrics for baseline runs.
- `data_manifest.csv`: stack-level dataset manifest.
- `splits_v1.json`: fixed train/val/test split by stack ID.
- `hard_region_catalog.csv`: hard-region tags and coordinates.
- `experiment_template.yaml`: reusable run configuration template.
- `run_registry.csv`: metadata registry for every run.

## Usage Notes
- Do not overwrite split versions. Create `splits_v2.json` if changed.
- Keep `run_id` unique across all runs.
- Record `commit_sha` and `dataset_version` for reproducibility.
