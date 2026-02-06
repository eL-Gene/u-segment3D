# Learned 3D Fusion Research Plan

## Objective
Build and validate a data-driven 3D fusion module that improves robustness of 2D-to-3D reconstruction when orthoview probabilities/flows are noisy or derived from weaker 2D models.

## Hypotheses
1. A learned 3D fusion model can outperform handcrafted `var_combine` under noisy orthoview signals.
2. SSL + weak supervision can reduce dependence on dense 3D labels.
3. Hybrid deployment (learned fusion + existing watershed/postprocess backend) provides the fastest path to practical gains.

## Success Criteria
- Quantitative improvements over baseline fusion on curated and challenging regions.
- Improved robustness under synthetic degradation of orthoview probs/flows.
- Stable inference on large z-stacks using patch-based 3D processing.

---

## Research Phases

### Phase 0 — Baseline and Dataset Definition
**Goal:** Lock baselines, datasets, and metrics.

Checklist:
- [ ] Freeze baseline runs with current direct/indirect fusion settings.
- [ ] Define train/val/test splits at stack level.
- [ ] Tag "hard" regions (low SNR, crowded boundaries, curvature artifacts).
- [ ] Define metrics (instance quality, flow consistency, robustness metrics).
- [ ] Build reproducible experiment config template.

Deliverables:
- Baseline report and experiment registry.
- Data manifest and split files.

### Phase 1 — Data Infrastructure
**Goal:** Build scalable I/O and patching for large TIFF stacks and multiview signals.

Checklist:
- [ ] Implement TIFF loader + memory-mapped/chunked reading.
- [ ] Implement aligned channel assembly for raw image + xy/xz/yz probs + flows.
- [ ] Implement patch sampler with hard-region oversampling.
- [ ] Add cache/index generation scripts.
- [ ] Validate deterministic sampling and reproducibility.

Deliverables:
- Data pipeline module.
- Sanity-check visualization notebook/script.

### Phase 2 — SSL Pretraining
**Goal:** Learn strong 3D priors without labels.

Checklist:
- [ ] Implement masked reconstruction objective (3D MAE-style or denoising autoencoder).
- [ ] Add microscopy-aware augmentations (noise, blur, intensity drift, anisotropy artifacts).
- [ ] Train baseline SSL encoder and track reconstruction/generalization metrics.
- [ ] Export pretrained weights and frozen encoder checkpoints.

Deliverables:
- SSL pretrained backbone.
- Pretraining logs and ablation summary.

### Phase 3 — Weakly Supervised Learned Fusion
**Goal:** Learn fusion from noisy orthoview outputs.

Checklist:
- [ ] Implement 3D fusion network (UNet-style) with optional uncertainty head.
- [ ] Inputs: raw volume + aligned orthoview probs/flows + optional confidence channels.
- [ ] Losses: reprojection consistency, flow consistency, smoothness, uncertainty-weighted residuals.
- [ ] Confidence masking to downweight low-trust pseudo-signals.
- [ ] Compare against handcrafted fusion baseline on val set.

Deliverables:
- Trained learned-fusion model.
- Baseline vs learned-fusion comparison table.

### Phase 4 — Hybrid Integration
**Goal:** Replace only fusion module while retaining current watershed/postprocess.

Checklist:
- [ ] Integrate learned fusion output as input to existing 3D watershed pipeline.
- [ ] Validate compatibility with existing downstream postprocessing.
- [ ] Add inference script for large-volume tiled fusion.
- [ ] Benchmark runtime and memory footprint.

Deliverables:
- End-to-end hybrid pipeline.
- Integration documentation.

### Phase 5 — Targeted Label Fine-tuning (Optional)
**Goal:** Improve challenging tissue behavior with minimal annotation burden.

Checklist:
- [ ] Curate small high-value labeled subset focused on hard regions.
- [ ] Fine-tune with mixed objective (supervised + weak/SSL regularizers).
- [ ] Evaluate generalization across easy vs hard regions separately.

Deliverables:
- Fine-tuned model variant.
- Hard-region performance report.

### Phase 6 — Robustness and Release Decision
**Goal:** Decide go/no-go for production/research adoption.

Checklist:
- [ ] Stress-test against degraded upstream 2D model outputs.
- [ ] Cross-dataset validation (different acquisitions, tissue states).
- [ ] Failure taxonomy and mitigation plan.
- [ ] Prepare reproducible runbook and checkpoint packaging.

Deliverables:
- Final evaluation summary.
- Adoption recommendation and roadmap.

---

## Experiment Governance

### Weekly Cadence
- **Mon:** Plan experiments and define acceptance thresholds.
- **Midweek:** Run experiments and monitor training health.
- **Fri:** Summarize results, decide next ablations.

### Tracking Template (per experiment)
- Experiment ID
- Data split/version
- Model + checkpoint
- Loss configuration
- Runtime and hardware
- Metrics (overall + hard-region subset)
- Qualitative notes and failure cases
- Decision (promote / iterate / discard)

---

## Initial Milestone Checklist (Next 2 Weeks)
- [ ] Branch and repo structure established.
- [ ] Baseline metrics reproduced on current pipeline.
- [ ] Data loader and patch sampler running on full-size stacks.
- [ ] First SSL pretraining run launched.
- [ ] First weak-supervision training config drafted.

Detailed Phase 0 execution checklist: `docs/phase0_execution_checklist.md`

## Risks and Mitigations
- **Risk:** Pseudo-label noise causes confirmation bias.
  - **Mitigation:** Confidence-weighted losses, teacher-student EMA, strong augmentations.
- **Risk:** Memory limits on full-resolution volumes.
  - **Mitigation:** Overlap-tiled training/inference, mixed precision, checkpointing.
- **Risk:** Good metrics on easy regions only.
  - **Mitigation:** Hard-region stratified validation and explicit oversampling.

## Exit Criteria for Phase Progression
- Phase 0 -> 1: Baselines and splits finalized.
- Phase 1 -> 2: Data pipeline stable and validated.
- Phase 2 -> 3: SSL features outperform random initialization in downstream validation.
- Phase 3 -> 4: Learned fusion exceeds handcrafted baseline on at least two robustness metrics.
- Phase 4 -> 5/6: Hybrid pipeline stable in large-stack inference.

