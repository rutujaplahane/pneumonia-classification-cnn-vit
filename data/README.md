# Data

This project uses the **RSNA Pneumonia Detection Challenge** dataset.

The original dataset contains de-identified chest X-ray images in DICOM format together with patient-level labels and pneumonia annotations. The raw DICOM images and original RSNA annotation files are **not included in this repository**.

## Dataset

**Source:** RSNA Pneumonia Detection Challenge, hosted on Kaggle.

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

The split CSV files in this repository contain the patient IDs and classification labels used to reproduce the experimental split. They are derived from the RSNA Pneumonia Detection Challenge annotations and remain subject to the applicable dataset terms and attribution requirements.

## Raw Data

Raw DICOM images and the original RSNA annotation files are intentionally excluded from this repository.

Users should obtain the dataset directly from the **RSNA Pneumonia Detection Challenge** through the official RSNA or Kaggle distribution pages and agree to the applicable terms of use before using the data.

## Dataset Attribution

The RSNA Pneumonia Detection Challenge dataset is derived from the **NIH Chest X-ray Dataset**, with pneumonia annotations developed for the RSNA Pneumonia Detection Challenge.

The **NIH Clinical Center** is acknowledged as the provider of the original chest X-ray dataset.

Please cite the following works when using the dataset:

1. X. Wang, Y. Peng, L. Lu, Z. Lu, M. Bagheri, and R. M. Summers, “ChestX-ray8: Hospital-scale chest X-ray database and benchmarks on weakly-supervised classification and localization of common thorax diseases,” *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*, pp. 3462–3471, 2017.

2. G. Shih et al., “Augmenting the National Institutes of Health Chest Radiograph Dataset with Expert Annotations of Possible Pneumonia,” *Radiology: Artificial Intelligence*, 2019, doi: 10.1148/ryai.2019180041.

Use and redistribution of the dataset are governed by the **RSNA Pneumonia Detection Challenge Terms of Use and Attribution** and the applicable Kaggle competition rules.

This repository contains project code, derived experimental splits, model predictions, metrics, and visualizations. It does not claim ownership of the original medical imaging dataset.