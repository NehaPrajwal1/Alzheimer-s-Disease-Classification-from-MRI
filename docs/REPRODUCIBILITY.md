# Reproduction status and execution guide

These are exploratory notebooks. Model training has not been rerun for this
documentation update; ADNI data and trained checkpoints were not available.

## Observed environments

| Notebook | Original saved output | Packages used |
| --- | --- | --- |
| Baseline / ResNet50 | TensorFlow 2.19.0; Colab paths | TensorFlow, NumPy, NiBabel, OpenCV (`opencv-python`), matplotlib, seaborn, scikit-learn, Pillow, psutil; torch/torchvision for the last binary cell |
| EfficientNetB3 | TensorFlow 2.10.0 | TensorFlow/Keras, NumPy, matplotlib, seaborn, scikit-learn |
| EfficientNetB5 | TensorFlow 2.10.0 | TensorFlow/Keras, NumPy, matplotlib, seaborn, scikit-learn |

Use Jupyter or Colab to inspect notebooks. `google.colab` imports are specific
to Colab; local users must replace Drive mounting and paths. Python, CUDA,
cuDNN, PyTorch, and most package versions were not recorded. This table is an
inventory, not a tested dependency lock. No universal `requirements.txt` is
provided because compatibility across these sessions has not been established.

B3/B5 use older Keras APIs and `.h5` weight filenames. Verify compatibility with
the chosen environment; current Keras may require API/filename changes. The B5
NumPy alias patch is historical compatibility code, not an environment specification.

## Before running

1. Obtain authorized data and apply the participant protocol in [DATA.md](DATA.md).
2. Work in a separate experiment copy; distinguish new runs from archival code.
3. Configure paths, class order, seed, and a unique output directory per run.
4. Address the training/evaluation issues in [IMPROVEMENTS.md](IMPROVEMENTS.md).
5. Check a small batch through loading, forward/backward passes, checkpoint
   save/reload, and evaluation before full training.

## Notebook-specific notes

**Baseline / ResNet50:** cells 0–11 prepare slices in Drive, but cell 16 points
to `/content/processed` before later copying occurs. Align those paths. Repeated
copy/configuration cells represent separate sessions: Run All can fail or use
stale state. Cells 63–71 are another TensorFlow ResNet session; cells 72–78 audit
and rebuild splits. Cell 79 is a separate PyTorch binary experiment reading
`processed_fixed`, and does not use TensorFlow weights.

**EfficientNetB3:** cell 1 reads `./processed` relative to the kernel working
directory and writes `./models`. Head training is present; intended fine-tuning
needs the trainability and loss fixes in the review. Evaluation does not validate
the input split.

**EfficientNetB5:** replace the Windows `os.chdir` and absolute `BASE_PATH` in
cell 2. Cell 8 loads `./models/best_effnet_p1.h5` without training phase 1. That
checkpoint and the B5 phase-1 recipe are absent. The similarly named B3 checkpoint
uses a different backbone and is not a substitute. A new B5 warm-up experiment
cannot be claimed to reproduce the historical run.

An additional ResNet file mentioned by the author is pending retrieval. Its
contents are unknown; it is a separate item from the missing B5 phase-1 artifact.
The available baseline already contains ResNet experiments.

## Save for the next reproducible run

- Package/Python/framework versions, GPU details, seed, commit, and configuration.
- Private cohort/split manifests and checksums, with aggregate counts for sharing.
- Class mapping, checkpoint selection criterion, training history, chosen weights.
- Private per-scan predictions linked to participants and preprocessing parameters.
- Aggregate confusion matrices, balanced accuracy, macro F1, per-class metrics,
  ROC/AUC definitions, and participant-resampled confidence intervals.

Select hyperparameters and checkpoints on training/validation data. Evaluate the
held-out test set after those decisions are fixed.
