# Code review and prioritized improvement plan

This review covers the three available notebooks. It is based on source and
saved outputs, not retraining. References use zero-based JSON cell indices.
The historical notebooks are preserved; the fixes below are proposed work.

## 1. Correct the evaluation unit and split protocol first

**Confirmed parser failure:** B3 cell 2 and B5 cell 12 match only filenames
starting with `ADNI_`. A name such as `m6_000_S_0000_55.png` falls back to the
whole filename, treating each unmatched slice as a group. Reported "subjects"
are therefore unverified. Baseline cell 73 also retains `.nii` in one parsed
participant component, which can separate baseline and month-6 records.

**Incomplete leakage fix:** baseline cells 77–78 strip slice indices but retain
visit prefixes. Their zero-overlap check is at scan-key level. Cell 73 groups
within each class, allowing a participant with changing diagnoses to cross
splits. The earliest split in cell 7 is by volume filename, not a random PNG
split; the precise source of historical overlap cannot be established from
the current files alone.

**Change:** create one strict participant parser/manifest for all visits and
classes; fail on unknown formats. Split globally by participant. Preserve scan
and visit IDs separately and define handling of inconsistent longitudinal labels.
Use that same manifest to align predictions, labels, and aggregation.

**Verify:** synthetic baseline/month-6/multiple-slice names resolve to one
participant; unknown names fail; label conflicts are flagged; all three split
intersections are empty; shuffling input rows does not change aggregation.
Recompute results only after this passes. Patient voting alone cannot fix leakage.

## 2. Verify that intended fine-tuning actually trains the backbone

B3 cell 6 and B5 cell 6 set `base.trainable = False`. Later phases set individual
child layers to trainable without resetting the parent model flag (B3 cell 8;
B5 cells 9–10). This can leave backbone weights excluded from training.

**Change:** set the parent `base_model.trainable = True` before applying the
desired layer-freezing mask, keep BatchNormalization frozen as intended, and
recompile. Keeping the backbone call at `training=False` is compatible with
training convolution weights while preserving BatchNormalization inference behavior.

**Verify:** inspect trainable variable names/counts and confirm that an intended
backbone weight changes after one optimizer step while frozen weights do not.
See the [Keras transfer-learning guide](https://keras.io/guides/transfer_learning/).

## 3. Correct the focal-loss reduction and inference description

**Focal loss:** B3/B5 cell 5 reduces to a scalar batch mean inside `call()` while
training passes per-class weights. That prevents the normal per-example weighting
semantics. Return one loss per example and let Keras apply sample weights and
reduction. Compare weighted output with a hand-calculated two-example batch.

**MC Dropout:** B3 cell 8 and B5 cell 11 repeatedly call `model.predict()`.
Ordinary prediction runs in inference mode, so those calls do not explicitly
activate dropout. B5's augmented passes provide TTA, but its first 20 passes are
ordinary inference, not a demonstrated MC-Dropout uncertainty estimate.

Either remove redundant inference passes or implement dropout-specific stochastic
inference while keeping BatchNormalization in inference mode. Do not blindly
call the entire model with `training=True`, because that also changes other
layers' behavior. Verify varying predictions with dropout enabled and unchanged
BatchNormalization statistics. See the [Keras FAQ](https://keras.io/getting_started/faq/).

## 4. Make experiments reproducible

- Recover the B5 phase-1 recipe/checkpoint loaded in cell 8. A pending ResNet
  file is separate and may not resolve that dependency.
- Replace hardcoded Drive/Windows paths with explicit configuration; consolidate
  repeated baseline sessions and use fresh output directories.
- Save sorted manifests, seeds, environment versions, class mappings, histories,
  and the source commit. Sort input lists before seeded shuffling.
- Use a single checkpoint-selection criterion: current callbacks monitor
  `val_loss` for early stopping and `val_accuracy` for saved weights. Evaluate
  the explicitly chosen checkpoint, including comparison across training phases.
- B3 combines cosine scheduling with ReduceLROnPlateau; the next cosine update
  overwrites the plateau adjustment. Choose a single learning-rate policy.
- Verify checkpoint round trips in the chosen Keras version, including custom
  layer/loss serialization and weight filename requirements.

## 5. Improve MRI preprocessing and model comparisons

The baseline selects array-axis slices without checking NIfTI orientation or
voxel spacing. Establish orientation/spacing, handle short volumes and invalid
data, and document image QC. Audit whether skull stripping or registration was
already performed on the downloaded cohort before adding either.

Review vertical flips (B3 cell 4; B5 cell 4) against the anatomical orientation.
B5 brightness jitter uses a delta of 0.2 with approximately 0–255 EfficientNet
inputs, so it is very small relative to that scale. Inspect transformed samples
locally and validate augmentations before using them.

The binary PyTorch ResNet50 uses normalization with mean/std 0.5 rather than the
pretrained weights' documented preprocessing. Compare preprocessing from
`ResNet50_Weights.DEFAULT.transforms()` with the current setup on the same
validation split; record the chosen pipeline rather than assuming improvement.

After fixing evaluation, compare a simple baseline and smaller backbone first.
Then test multi-slice aggregation, a 3D model, or clinical-feature fusion as
controlled experiments. Larger networks and additional modalities do not
guarantee better performance. The pooled feature branches called SPP reduce each
branch globally; they do not retain the usual spatial-bin layout of a full
spatial pyramid. Describe them precisely or ablate a true spatial-bin version.

## 6. Report evidence appropriate to the claim

Use the same held-out participant split for comparable models. Report class
counts, balanced accuracy, macro F1, per-class precision/recall, confusion
matrices, clearly defined AUCs, and participant-resampled confidence intervals.
Compare against a majority-class baseline. If participants have multiple visits,
define whether metrics are per scan, per visit, or per participant; avoid
silently combining conflicting visit labels into one outcome.

Do not compare binary slice accuracy directly with three-class participant
accuracy, call retrospective classification "early detection" without an
appropriate prediction task, or claim external generalization without an
independent cohort. Add the exact bibliographic reference before retaining the
notebook's claim to replicate a paper baseline; its hardcoded paper numbers are
not a controlled comparison.

## Suggested order of future implementation commits

1. `fix: unify participant parsing and enforce disjoint dataset splits`
2. `fix: validate backbone fine-tuning and weighted focal loss`
3. `fix: correct inference and manifest-based evaluation`
4. `refactor: configure reproducible training and checkpoint selection`
5. `docs: report rerun results with participant-level uncertainty`

Each code fix should include a small targeted regression check. New metric
claims require a fresh, auditable run; they cannot be inferred from this review.
