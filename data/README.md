# Data

This project uses the **RSNA Pneumonia Detection Challenge** dataset.

The original dataset contains chest X-ray images in DICOM format and associated
patient-level labels. The raw DICOM images are not included in this repository
because of their size.

## Dataset

Source: RSNA Pneumonia Detection Challenge (Kaggle)

The classification dataset used in this project contains:

- 26,684 unique patients
- 20,672 non-pneumonia cases
- 6,012 pneumonia cases
- Pneumonia prevalence: 22.53%

The original DICOM images are 1024 × 1024 grayscale chest X-rays.

## Data Splits

A stratified patient-level split was used:

| Split | Patients | Pneumonia | No Pneumonia |
|---|---:|---:|---:|
| Train | 18,678 | 4,208 | 14,470 |
| Validation | 4,003 | 902 | 3,101 |
| Test | 4,003 | 902 | 3,101 |
| Total | 26,684 | 6,012 | 20,672 |

There is no patient overlap between the training, validation, and test sets.

The CSV files in this directory contain the patient IDs and labels used for
each split.

## Raw Data

Raw DICOM images are intentionally excluded from this repository.

Download the RSNA Pneumonia Detection Challenge dataset from Kaggle and place
the images in the appropriate local data directory before running the training
pipeline.