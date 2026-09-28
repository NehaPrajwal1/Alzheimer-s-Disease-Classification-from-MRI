# Alzheimer's Disease Classification from MRI (ADNI)

A research project exploring 2D brain MRI classification using simple CNN,
ResNet50, EfficientNetB3, and EfficientNetB5 models on an ADNI1-derived dataset.
The labels are **AD** (Alzheimer's disease), **CN** (cognitively normal), and
**MCI** (mild cognitive impairment). A separate ResNet50 experiment uses AD/CN only.

This repository records model experiments and lessons from investigating data
leakage. It is an experimental classification study, not a validated diagnostic
system or a demonstrated predictor of future disease progression. The available
code uses MRI slices; cognitive and speech features are not implemented.

## Project status

Three experiment notebooks are available. Historical outputs are documented,
but **patient-level split independence and patient-level metrics are not yet
verified**. The final notebook's subject parser does not handle all filename
formats seen in the earlier experiments. Its phase-1 checkpoint and training
code are missing, so exact end-to-end reproduction is currently incomplete.

## Experiments

| Experiment | Implementation | What it explores |
| --- | --- | --- |
| [Baseline and ResNet50](notebooks/01_baseline_resnet50.ipynb) | TensorFlow/Keras; later PyTorch | Slice preprocessing, simple CNN, ResNet50, split audits, binary AD/CN classification |
| [EfficientNetB3](notebooks/02_efficientnetb3.ipynb) | TensorFlow/Keras | GeM and pooled feature branches, focal loss, class weighting, staged training |
| [EfficientNetB5](cnn_train_final_v7.ipynb) | TensorFlow/Keras | Resumed training, test-time augmentation, attempted subject-level soft voting |

See the [notebook guide](notebooks/README.md) for provenance and reading order.

## Pipeline

1. Load locally obtained ADNI NIfTI volumes using NiBabel.
2. Extract the middle 60 slices along array axis 2, normalize each slice to
   8-bit intensity values, and resize to 160 × 160 PNGs. Anatomical orientation
   is not verified by the extraction code.
3. Prepare train/validation/test folders with approximately 70/15/15 proportions.
   Several splitting attempts are present: the initial version splits volume
   filenames before extraction; later versions regroup existing PNGs.
4. Train classifiers. EfficientNet and binary PyTorch inputs are resized to
   224 × 224. Loss, augmentation, and fine-tuning settings vary by experiment.
5. Evaluate slice predictions and, in the EfficientNet notebooks, aggregate
   probabilities using identifiers parsed from filenames.

## What the leakage investigation found

The baseline notebook's saved audit reported 38 overlapping scan keys between
train/test, 40 between train/validation, and 8 between validation/test. A later
rebuild reported zero overlaps using its scan-key parser.

That is useful debugging evidence, but it does **not** establish participant
independence: baseline and month-6 scans can have different scan keys for the
same participant. Grouping separately within each diagnosis can also split a
participant whose diagnosis changes across visits. A global participant manifest
and strict overlap assertions are the next priority.

## Historical results

These are saved outputs, not results from a new training run. The B5 numbers
describe the existing filename-based aggregation and must not be cited as
verified patient-level performance.

| Experiment | Evaluation unit | Saved result |
| --- | --- | --- |
| EfficientNetB5, AD/CN/MCI | Filename-derived groups; patient grouping unverified | Accuracy **49.33%**, balanced accuracy **41.54%**, macro OvR AUC **0.6019** |
| EfficientNetB5, AD/CN subset | Same grouping; CN score from the three-class model | AUC **0.6156** |
| PyTorch ResNet50, AD/CN | **Slices**, after a scan-key split rebuild | Accuracy **67.38%** (3,032 / 4,500 slices; printed report rounds to 0.67) |

The B5 report counts 6,890 groups; unmatched filenames become entire groups, so
this count cannot be described as 6,890 patients. The binary and three-class
experiments use different tasks and units and are not a direct model comparison.
See [results and provenance](results/README.md) for class metrics and limitations.

![Historical B5 training curves](results/training_curves.png)

*Saved B5 training curves, extracted from the existing notebook. They do not
establish that the intended backbone unfreezing took effect.*

## Data and setup

ADNI data and trained model weights are not included. Obtain data through
[ADNI's official access process](https://adni.loni.usc.edu/data-samples/adni-data/)
and follow the applicable agreement. See [data preparation](docs/DATA.md).

```bash
git clone https://github.com/NehaPrajwal1/Alzheimer-s-Disease-Classification-from-MRI.git
cd Alzheimer-s-Disease-Classification-from-MRI
```

Read the [reproduction guide](docs/REPRODUCIBILITY.md) before installing packages
or executing notebooks. It lists observed framework versions, dependencies, path
changes, and missing artifacts. A single environment and one-command training
workflow have not yet been verified.

## Repository structure

```text
.
├── README.md
├── cnn_train_final_v7.ipynb       # Existing B5 notebook and saved outputs
├── notebooks/
│   ├── README.md
│   ├── 01_baseline_resnet50.ipynb
│   └── 02_efficientnetb3.ipynb
├── docs/
│   ├── DATA.md
│   ├── REPRODUCIBILITY.md
│   └── IMPROVEMENTS.md
└── results/
    ├── README.md
    ├── confusion_matrices.png
    ├── training_curves.png
    └── roc_curves.png
```

## Improvements to prioritize

1. Correct participant parsing and split membership across all visits/classes;
   rerun evaluation on a held-out participant split.
2. Verify backbone trainability, return per-example focal losses, and correct
   the inference procedure labeled MC Dropout.
3. Recover B5 phase-1 artifacts and record a reproducible environment and configuration.
4. Then compare simpler baselines, multi-slice/3D approaches, and additional
   modalities on the same splits. Their benefit is a hypothesis to test.

The [prioritized code review](docs/IMPROVEMENTS.md) includes cell references and
checks for each change. These model fixes have not been applied or evaluated in
this documentation update; the historical experiments are preserved.

## Acknowledgment

This project uses data from the Alzheimer's Disease Neuroimaging Initiative
(ADNI). ADNI investigators contributed to study design, implementation, and/or
data collection; they did not participate in this project's analysis or writing.
Consult the [official ADNI publication guidance](https://adni.loni.usc.edu/data-samples/adni-data/)
for the required acknowledgment language and investigator list for publications.
