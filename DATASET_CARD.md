# Dataset Card: CBIS-DDSM cropped ROI images (benign vs. malignant)

## Overview
- **Task:** binary classification (0 = benign incl. benign-without-callback, 1 = malignant) of cropped mammography ROI images.
- **Source:** CBIS-DDSM (Curated Breast Imaging Subset of DDSM), JPEG version from the Kaggle mirror `awsaf49/cbis-ddsm-breast-cancer-image-dataset`.
- **Licence:** listed as CC-BY-SA-3.0 by the Kaggle CLI for this mirror. Verify the terms of the original TCIA collection before any redistribution. Images are **not** redistributed or committed to the repository.
- **Content:** mass and calcification cases; official train / test CSVs; pathology label from biopsy.

## Composition after cleaning
|                            |   BENIGN |   BENIGN_WITHOUT_CALLBACK |   MALIGNANT |
|:---------------------------|---------:|--------------------------:|------------:|
| ('test', 'calcification')  |      130 |                        67 |         129 |
| ('test', 'mass')           |      194 |                        37 |         147 |
| ('train', 'calcification') |      522 |                       449 |         518 |
| ('train', 'mass')          |      566 |                        98 |         627 |

## Splits (patient-level, leakage-safe)
- Test = official CBIS-DDSM test set. Validation = one patient-grouped, label-stratified fold of the official train set (seed 42).
- Verified in code: no patient appears in more than one of train / val / test (mass and calcification combined).

| split   |   images |   patients |   malignant |   malignant % |   mass |   calcification |
|:--------|---------:|-----------:|------------:|--------------:|-------:|----------------:|
| train   |     2188 |        975 |         895 |          40.9 |   1029 |            1159 |
| val     |      592 |        242 |         250 |          42.2 |    262 |             330 |
| test    |      704 |        349 |         276 |          39.2 |    378 |             326 |

## Cleaning
- Rows in the original CSVs: 3568. Excluded rows: 84. Reasons: {'patient_overlap_train_test': 83, 'no_match': 1}.
- Excluded: unmatched, ambiguous, unreadable and mask-like files; duplicate images (same file bytes; train copies removed when also in test; conflicting-label duplicates removed); train rows of patients that also appear in test.
- Full log: `outputs/excluded_files.csv` and `outputs/match_log.csv`.

## Preprocessing and augmentation
- ROI crop padded to a square (black) and resized to 64x64, grayscale, normalised with train mean/std (0.4049 / 0.2588).
- Training augmentation only: horizontal/vertical flip, rotation up to 15 degrees, translation 5 %, scale 0.9-1.1, brightness/contrast jitter 0.2.

## Label audit
- Automatic lesion-view consistency check: 3 of 1486 multi-view lesion groups have conflicting labels (`outputs/label_consistency_flags.csv`).
- Manual review of a random sample: 0/100 problems found; 95% upper bound on the problem rate = 3.0%.
- The audit checks image/label correspondence, not the biopsy result itself.

## Known limitations and biases
- Digitised film mammograms from a historical US collection; scanner, patient population and imaging protocol may not match modern digital mammography.
- ROI crops vary in size and context; some lesions are very small after resizing.
- Mass and calcification cases differ visually and are pooled in one classifier.
- One patient can contribute several images (views, lesions): metrics are correlated, so confidence intervals are computed over patients.
- Pathology labels are taken from the dataset and were not independently re-verified.

## Intended and out-of-scope use
- Intended: coursework on dataset preparation and image classification.
- **Out of scope:** any clinical, diagnostic or screening use.
