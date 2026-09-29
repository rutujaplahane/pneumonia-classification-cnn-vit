# CNN and Vision Transformer-Based Pneumonia Classification in Chest X-Rays

## Overview

This project develops and compares deep learning models for **pneumonia classification from chest X-ray images** using the **RSNA Pneumonia Detection Challenge** dataset.

Two transfer-learning approaches are evaluated:

- **DenseNet121** as the CNN-based classifier
- **Vision Transformer (ViT-B/16)** as the transformer-based classifier

The complete pipeline includes dataset inspection, data cleaning, exploratory data analysis, patient-level data splitting, model training, threshold selection, held-out test evaluation, subgroup error analysis, and model explainability using **Grad-CAM for DenseNet121** and **self-attention visualization for ViT**.

---

## Dataset

The project uses chest X-ray images and annotations from the **RSNA Pneumonia Detection Challenge**.

### Dataset Summary

| Item             |           Value |
| ---------------- | --------------: |
| Unique patients  |          26,684 |
| No Pneumonia     | 20,672 (77.47%) |
| Pneumonia        |  6,012 (22.53%) |
| Image format     |           DICOM |
| Image resolution |     1024 × 1024 |

The original RSNA annotations contain three diagnostic categories:

| Diagnostic Class             | Patients |
| ---------------------------- | -------: |
| No Lung Opacity / Not Normal |   11,821 |
| Normal                       |    8,851 |
| Lung Opacity                 |    6,012 |

For binary classification, **Lung Opacity** cases are treated as pneumonia-positive (`Target = 1`), while the other two categories correspond to `Target = 0`.

The raw DICOM dataset is not included in this repository.

---

## Data Split

A patient-level stratified split was used to prevent patient overlap between training, validation, and test sets.

| Split      | Patients | Pneumonia | No Pneumonia |
| ---------- | -------: | --------: | -----------: |
| Train      |   18,678 |     4,208 |       14,470 |
| Validation |    4,003 |       902 |        3,101 |
| Test       |    4,003 |       902 |        3,101 |

Pneumonia prevalence is approximately **22.53%** in each split.

The validation set is used for model selection and decision-threshold selection. The test set remains held out until final evaluation.

---

## Project Pipeline

```text
Raw RSNA Data
      │
      ▼
Data Inspection
      │
      ▼
Data Cleaning
      │
      ▼
Exploratory Data Analysis
      │
      ▼
Patient-Level Train / Validation / Test Split
      │
      ├─────────────────────┐
      ▼                     ▼
DenseNet121              ViT-B/16
      │                     │
      ▼                     ▼
Validation-Based Model & Threshold Selection
      │                     │
      └──────────┬──────────┘
                 ▼
             Test Set
                 │
                 ▼
       Model Comparison
                 │
        ┌────────┴────────┐
        ▼                 ▼
     Grad-CAM        Attention Maps
```

---

## Preprocessing

The preprocessing pipeline includes:

- DICOM image loading using `pydicom`
- Pixel normalization based on DICOM bit depth
- Handling of `MONOCHROME1` images
- Conversion to RGB
- Resizing to `224 × 224`
- ImageNet normalization
- Training-time data augmentation
- Patient-level stratified splitting

To improve training efficiency, preprocessed `224 × 224` images were cached during model development.

The training augmentation included small rotations and translations while avoiding transformations that could substantially alter medically relevant image structure.

---

## Models

### 1. DenseNet121

A pretrained **DenseNet121** model was used as the CNN classifier.

The original classification layer was replaced with a single-output binary classification head.

Key training settings included:

- ImageNet pretrained weights
- Weighted binary cross-entropy loss
- AdamW optimizer
- Learning-rate scheduling
- Automatic mixed precision
- Validation ROC-AUC based checkpoint selection
- Early stopping
- Validation-based decision-threshold optimization

The positive-class weight was used to account for class imbalance.

### 2. Vision Transformer

A pretrained **ViT-B/16** model was used as the transformer-based classifier.

Images are divided into fixed-size patches, which are represented as tokens and processed through transformer encoder blocks using multi-head self-attention.

Training used:

- ImageNet pretrained weights
- Weighted binary cross-entropy loss
- AdamW optimizer
- Learning-rate scheduling
- Automatic mixed precision
- Validation ROC-AUC based checkpoint selection
- Early stopping
- Validation-based decision-threshold optimization

The best ViT checkpoint was selected at **epoch 3**.

---

## Evaluation Metrics

The models were evaluated using:

- ROC-AUC
- Average Precision / PR-AUC
- Accuracy
- Balanced Accuracy
- Precision
- Recall / Sensitivity
- Specificity
- Negative Predictive Value
- F1-score
- Matthews Correlation Coefficient
- Brier Score
- Confusion Matrix

Because the dataset is imbalanced, evaluation is not based on accuracy alone.

---

## Final Test Results

| Metric            | DenseNet121 |   ViT-B/16 |
| ----------------- | ----------: | ---------: |
| ROC-AUC           |      0.8818 | **0.8850** |
| PR-AUC            |  **0.7004** |     0.6984 |
| Accuracy          |  **0.8271** |     0.8119 |
| Balanced Accuracy |      0.7902 | **0.8000** |
| Precision         |  **0.5960** |     0.5594 |
| Recall            |      0.7228 | **0.7783** |
| Specificity       |  **0.8575** |     0.8217 |
| F1-score          |  **0.6533** |     0.6509 |
| MCC               |  **0.5440** |     0.5403 |
| Brier Score ↓     |      0.1466 | **0.1460** |

Validation-selected decision thresholds:

- **DenseNet121:** 0.60
- **ViT-B/16:** 0.57

Both models achieved similar overall discrimination, with different operating characteristics. ViT-B/16 produced higher ROC-AUC and recall, while DenseNet121 produced higher precision, specificity, accuracy, and F1-score.

---

## Confusion Matrices

### DenseNet121

|                 | Predicted Negative | Predicted Positive |
| --------------- | -----------------: | -----------------: |
| Actual Negative |              2,659 |                442 |
| Actual Positive |                250 |                652 |

### ViT-B/16

|                 | Predicted Negative | Predicted Positive |
| --------------- | -----------------: | -----------------: |
| Actual Negative |              2,548 |                553 |
| Actual Positive |                200 |                702 |

The ViT model identified more pneumonia-positive cases but also produced more false-positive predictions.

---

## Diagnostic Subgroup Analysis

Performance was further examined using the original RSNA diagnostic categories.

For ViT-B/16:

| Diagnostic Class             | Samples | Errors | Error Rate |
| ---------------------------- | ------: | -----: | ---------: |
| Lung Opacity                 |     902 |    200 |     22.17% |
| No Lung Opacity / Not Normal |   1,780 |    544 |     30.56% |
| Normal                       |   1,321 |      9 |      0.68% |

A similar pattern was observed for DenseNet121.

The models distinguish normal radiographs effectively, while the **No Lung Opacity / Not Normal** subgroup is substantially more challenging. These images are abnormal but do not contain the lung opacity associated with the positive pneumonia label, making them important difficult-negative cases.

---

## Explainability

### Grad-CAM

Grad-CAM was applied to the DenseNet121 model to visualize image regions contributing to classification decisions.

Representative examples were generated for:

- True Positive
- True Negative
- False Positive
- False Negative

Where available, pneumonia bounding-box annotations were included for comparison with model activation regions.

### Vision Transformer Attention

Self-attention from the final transformer block was extracted from ViT-B/16.

Attention from the classification token to image patch tokens was transformed into spatial attention maps and overlaid on the original chest X-rays.

Representative TP, TN, FP, and FN cases are included in the project results.

---

## Repository Structure

```text
pneumonia-classification-cnn-vit/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── README.md
│   ├── processed/
│   └── splits/
│
├── notebooks/
│   ├── 01_data_inspection.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_eda.ipynb
│   ├── 04_cnn_pipeline.ipynb
│   ├── 05_vit_pipeline.ipynb
│   └── 06_model_comparison.ipynb
│
└── results/
    ├── cnn/
    ├── vit/
    └── comparison/
```

### Notebooks

| Notebook                    | Description                                                              |
| --------------------------- | ------------------------------------------------------------------------ |
| `01_data_inspection.ipynb`  | Dataset structure, metadata and DICOM inspection                         |
| `02_data_cleaning.ipynb`    | Metadata validation and preparation                                      |
| `03_eda.ipynb`              | Class distributions, bounding-box statistics and image-level EDA         |
| `04_cnn_pipeline.ipynb`     | DenseNet121 preprocessing, training, evaluation and Grad-CAM             |
| `05_vit_pipeline.ipynb`     | ViT-B/16 preprocessing, training, evaluation and attention visualization |
| `06_model_comparison.ipynb` | Final CNN–ViT comparison and analysis                                    |

---

## Installation

Clone the repository and install the required packages:

```bash
git clone <repository-url>
cd pneumonia-classification-cnn-vit
pip install -r requirements.txt
```

Main dependencies include:

```text
torch
torchvision
timm
pydicom
numpy
pandas
scikit-learn
matplotlib
Pillow
tqdm
```

---

## Running the Project

The notebooks are intended to be executed sequentially:

```text
01_data_inspection.ipynb
        ↓
02_data_cleaning.ipynb
        ↓
03_eda.ipynb
        ↓
04_cnn_pipeline.ipynb
        ↓
05_vit_pipeline.ipynb
        ↓
06_model_comparison.ipynb
```

The raw RSNA dataset must be downloaded separately and the dataset path configured before running the data-processing and model-training notebooks.

GPU acceleration is recommended for the CNN and ViT training notebooks.

---

## Saved Results

The `results/` directory contains the main reproducibility and evaluation artifacts, including:

- Training histories
- Validation-selected thresholds
- Test metrics
- Test predictions
- ROC and precision-recall curves
- Confusion matrices
- Calibration analysis
- Diagnostic subgroup analysis
- Grad-CAM visualizations
- ViT attention visualizations
- CNN–ViT comparison outputs

Large model checkpoint files (`.pth`) and raw DICOM images are excluded from GitHub.

---

## Key Findings

DenseNet121 and ViT-B/16 achieved comparable performance on the held-out test set.

ViT-B/16 achieved a slightly higher ROC-AUC (**0.8850**) and higher recall (**0.7783**), while DenseNet121 achieved slightly higher F1-score (**0.6533**), precision (**0.5960**), and specificity (**0.8575**).

Subgroup analysis showed that both models perform particularly well on normal radiographs but have greater difficulty distinguishing pneumonia from abnormal non-pneumonia radiographs.

The results demonstrate the importance of evaluating medical image classifiers beyond overall accuracy and examining threshold-dependent metrics, clinically relevant error types, diagnostic subgroups, and model explanations.

---

## Limitations

- The project performs **binary image-level classification**, not pneumonia localization or segmentation.
- The models are evaluated on a held-out subset of the same RSNA dataset and have not been externally validated on an independent clinical dataset.
- Attention maps and Grad-CAM provide model-behavior visualizations but should not be interpreted as definitive clinical explanations.
- Model predictions are intended for research and educational purposes and are not suitable for clinical diagnosis.

---

## Technologies

**Python · PyTorch · Torchvision · timm · Scikit-learn · Pandas · NumPy · Matplotlib · pydicom · CNN · DenseNet121 · Vision Transformer · Grad-CAM**
