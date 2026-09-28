# Data access and preparation

The project uses an ADNI1-derived collection organized into AD, CN, and MCI
folders. Exact cohort selection, acquisition protocol, visit inclusion rules,
and diagnostic label dates are not established by the notebooks. Record these
before treating the cohort as reproducible.

## Access

Apply through [ADNI Data](https://adni.loni.usc.edu/data-samples/adni-data/).
Approved access is managed through the LONI Image and Data Archive (IDA).
Read the [getting-started guide](https://adni.loni.usc.edu/quick-start-guide-asset/getting_started.html)
and the applicable [Data Use Agreement](https://adni.loni.usc.edu/wp-content/themes/adni_2023/documents/ADNI_Data_Use_Agreement.pdf).
Keep restricted data in your authorized environment. Do not commit participant
records, MRI volumes, derived slice datasets, or credentials.

Ignore rules cover common data/model directories and formats; review staged
files as well, because ignore rules cannot recognize all restricted content.

## Expected local layout

The baseline expects `ADNI1_AD`, `ADNI1_CN`, and `ADNI1_MCI` directories of local
NIfTI files under its configured `base_path`. Training notebooks expect:

```text
processed/
├── train/
│   ├── AD/
│   ├── CN/
│   └── MCI/
├── val/        # Same class directories
└── test/       # Same class directories
```

Different sessions use `processed`, `processed_clean`, and `processed_fixed`.
A directory name does not prove that its split is correct.

## Historical preprocessing

Baseline cells 7–9 split volume filenames per class, load NIfTI arrays, select
the middle 60 axis-2 slices, min-max normalize each slice, and resize to 160 × 160.
EfficientNet loaders then resize to 224 × 224. The code does not establish
canonical orientation, common voxel spacing, registration, or skull stripping.
It assumes at least 60 slices along the selected axis.

These example names use a **synthetic participant ID**:

```text
ADNI_000_S_0000.nii_55.png
m6_000_S_0000_55.png
```

Both must resolve to participant `000_S_0000`. Visit, scan, and slice identifiers
should be separate fields. Unrecognized names must raise errors instead of
becoming separate pseudo-participants.

## Required split protocol for the next run

1. Build a private manifest of participant ID, scan/visit ID, diagnosis at the
   selected visit, source path, and image inclusion/QC decisions.
2. Define how changing diagnoses and repeated visits are handled for the task.
3. Assign each participant globally to one split, across all classes and visits.
   Use a fixed seed and save the manifest locally.
4. Derive slice membership from that assignment. Use fresh output directories
   so files from an earlier split cannot remain in the dataset.
5. Assert all three pairwise participant intersections are empty; reject
   duplicate scans and inconsistent labels at the chosen evaluation unit.
6. Record aggregate participant, scan, and slice counts per class and split.
   Fit learned preprocessing on training data only.

The saved scan-level zero-overlap output does not replace these checks. Exact
participant counts cannot be recovered from the classification reports.
