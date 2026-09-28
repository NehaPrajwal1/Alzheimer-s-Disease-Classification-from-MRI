# Experiment notebooks

These are historical research notebooks, not a tested run-all pipeline. Read the
[reproduction notes](../docs/REPRODUCIBILITY.md) and
[code review](../docs/IMPROVEMENTS.md) before training or interpreting results.

| File | Original file | Contents |
| --- | --- | --- |
| [01_baseline_resnet50.ipynb](01_baseline_resnet50.ipynb) | `ADNI_project.ipynb` | NIfTI-to-PNG preprocessing, simple CNN, TensorFlow ResNet50, split audits/rebuilding, and a final PyTorch AD/CN ResNet50 experiment |
| [02_efficientnetb3.ipynb](02_efficientnetb3.ipynb) | `cnn_trian (1).ipynb` | EfficientNetB3 with GeM and pooled feature branches, focal loss, staged training, and attempted subject aggregation |
| [cnn_train_final_v7.ipynb](../cnn_train_final_v7.ipynb) | Existing GitHub notebook | EfficientNetB5 with resumed training, repeated inference, test-time augmentation, aggregation, and saved plots |

The previously committed B5 notebook stays at its original path and is unchanged.
The two added notebooks preserve the original cell order and experimental code;
saved outputs, execution counts, and cell metadata have been removed. Literal
participant identifiers in example paths/comments are replaced by the synthetic
`000_S_0000`. Replace that example path with a locally authorized scan before use.
The original workspace files are unchanged.

Cell references in the documentation are **zero-based JSON cell indices**. They
do not mean execution counts, which may be missing or out of order.

Suggested reading order: baseline preprocessing (cells 0–11), baseline models
(12–71), split investigations (72–78), binary PyTorch run (79), B3, then B5.
Repeated configuration and copy cells in the baseline represent separate sessions;
running every cell sequentially can fail or use stale paths/state.
