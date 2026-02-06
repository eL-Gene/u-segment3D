# Phase 0 Execution Checklist (Baseline + Dataset Definition)

## Scope
This checklist operationalizes Phase 0 from `docs/learned_fusion_research_plan.md` into concrete tasks, outputs, and gates.

## Phase 0 Exit Gate
Phase 0 is complete only when all of the following are true:
- Baseline runs are frozen and reproducible.
- Train/val/test splits are fixed at stack level.
- Hard regions are tagged and tracked separately.
- Metrics and robustness protocol are defined and versioned.
- Experiment config template exists and is in use.

## Required Artifacts
Create these under `artifacts/phase0/`:
- `baseline_report.md`
- `baseline_metrics.csv`
- `data_manifest.csv`
- `splits_v1.json`
- `hard_region_catalog.csv`
- `experiment_template.yaml`
- `run_registry.csv`

## Preflight Gate (Before Any Expensive GPU Run)
- [ ] Working tree is clean or intentional (`git status --short` reviewed).
- [ ] Commit SHA logged for the run (`git rev-parse HEAD`).
- [ ] Environment is installable from source (`pip install -e .`).
- [ ] Optional full stack install validated when needed (`pip install -e ".[all]"`).
- [ ] Basic import smoke test passes:
  - `python -c "import segment3D; import segment3D.usegment3d as s; print('ok')"`
- [ ] Run config and output directory are created before launch.

## Task 1: Freeze Baseline Runs
- [ ] Select fixed baseline scripts/configs (direct and indirect paths where applicable).
- [ ] Run baseline on chosen reference stacks.
- [ ] Save raw outputs and postprocessed outputs with stable naming.
- [ ] Log runtime, hardware, and commit SHA for each run in `run_registry.csv`.

Minimum metadata per run:
- `run_id`
- `date_utc`
- `commit_sha`
- `dataset_version`
- `split_version`
- `script_or_entrypoint`
- `key_params`
- `runtime_min`
- `device`

## Task 2: Define Dataset Manifest and Splits
- [ ] Build `data_manifest.csv` at stack level (one row per volume).
- [ ] Include fields: `stack_id`, `path`, `source`, `voxel_size`, `notes`.
- [ ] Create fixed split file `splits_v1.json`:
  - `train`: list of stack IDs
  - `val`: list of stack IDs
  - `test`: list of stack IDs
- [ ] Verify no stack leakage across splits.
- [ ] Record split rationale in `baseline_report.md`.

## Task 3: Hard-Region Tagging
- [ ] Define hard-region criteria:
  - low SNR
  - crowded boundaries
  - curvature/anisotropy artifacts
- [ ] Tag hard regions in `hard_region_catalog.csv`.
- [ ] Ensure each tagged region maps to `stack_id` and coordinates/slices.
- [ ] Add a summary count by difficulty type in `baseline_report.md`.

Suggested columns:
- `region_id`
- `stack_id`
- `z_start`
- `z_end`
- `y_start`
- `y_end`
- `x_start`
- `x_end`
- `difficulty_type`
- `annotator`
- `notes`

## Task 4: Metrics + Robustness Protocol
- [ ] Lock primary metrics (overall + hard-region subset).
- [ ] Lock robustness corruption suite (noise/blur/dropout/intensity drift levels).
- [ ] Store exact metric definitions and corruption settings in `baseline_report.md`.
- [ ] Confirm metric code path references `segment3D/metrics.py` where applicable.

Recommended metric set for Phase 0:
- Instance IoU (overall, hard-region subset)
- Dice (overall, hard-region subset)
- Flow consistency error (if flow outputs are available)
- Runtime and peak memory

## Task 5: Experiment Template and Governance
- [ ] Create `experiment_template.yaml` with:
  - dataset/split version
  - model/fusion mode
  - loss block
  - augmentation block
  - optimizer/scheduler
  - checkpoint cadence
  - output/log paths
- [ ] Create `run_registry.csv` and use it for every run.
- [ ] Define weekly run budget caps (GPU hours and max run count).

## Task 6: Signoff Checklist
- [ ] All required artifacts exist in `artifacts/phase0/`.
- [ ] One reviewer validates reproducibility from documented steps.
- [ ] Phase 0 summary and recommendation added to `baseline_report.md`.
- [ ] Explicit decision recorded: `Phase 0 -> Phase 1` or `hold`.

## Pull-to-Compute Workflow (Colab/HPC)
Use GitHub commit pinning for every compute run:
- `git fetch --all --prune`
- `git checkout research/learned-fusion-3d-plan`
- `git pull`
- `git rev-parse HEAD` (copy into run metadata)

For long jobs:
- checkpoint every 15-30 minutes
- persist logs/artifacts off ephemeral disks
- resume from latest checkpoint on preemption
