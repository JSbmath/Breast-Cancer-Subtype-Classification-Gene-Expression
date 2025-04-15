# Breast Cancer Subtype Classification using Gene Expression Data (TCGA)

## Overview

This project aims to classify subtypes of Breast Invasive Carcinoma (BRCA) using mRNA gene expression data from The Cancer Genome Atlas (TCGA).
This repository contains a Jupyter Notebook demonstrating a machine learning workflow to predict cancer subtypes based on gene expression profiles. The process involves data acquisition, extensive preprocessing, feature selection, handling significant class imbalance using SMOTE, and model training/evaluation using RandomForestClassifier.

## Data

* **Source:** cBioPortal for Cancer Genomics ([cbioportal.org](https://www.cbioportal.org/))
* **Study:** Breast Invasive Carcinoma (TCGA, PanCancer Atlas)
* **Files Used:**
    * Clinical Data: `data_clinical_sample.txt` (containing sample annotations, including subtypes)
    * Expression Data: `data_mrna_seq_v2_rsem_zscores_ref_diploid_samples.txt` (mRNA expression z-scores relative to diploid samples)
* **Target Variable:** The `CANCER_TYPE_DETAILED` column from the clinical data was used as the target for classification after preprocessing (removing single-sample classes and grouping rare subtypes).

## Methodology / Workflow

The analysis followed these steps:

1.  **Data Loading & Initial Cleaning:** Loaded clinical and expression data using Pandas. Set appropriate indices (`SAMPLE_ID` for clinical, `Hugo_Symbol` for expression).
2.  **Data Alignment:** Ensured both clinical and expression datasets contained the same samples in the same order.
3.  **Handling Gene Duplicates:** Removed duplicate gene symbols from the expression data, keeping the instance with the highest mean expression across samples.
4.  **Missing Value Analysis:** Identified and removed genes with a high percentage (>50%) of missing values (NaNs). Confirmed no further imputation was needed after this step.
5.  **Exploratory Data Analysis (EDA):** Applied StandardScaler and performed dimensionality reduction using PCA and UMAP to visualize the high-dimensional gene expression data and observe potential clustering based on cancer subtypes.
6.  **Handling Class Imbalance:**
    * Removed classes with only one sample to enable stratified splitting.
    * Grouped remaining rare subtypes (those with < 25 samples) into an 'Other_Subtype' category.
    * Applied **SMOTE (Synthetic Minority Over-sampling Technique)** to the *training data* to balance the class distribution before model training.
7.  **Feature Selection:** Used `SelectKBest` with the ANOVA F-statistic (`f_classif`) to select the top 1000 most relevant genes associated with the cancer subtypes in the training data.
8.  **Feature Scaling:** Applied `StandardScaler` to the selected gene features, fitting *only* on the training data and transforming both training and test sets.
9.  **Train/Test Split:** Split the data (selected features and target variable) into training (70%) and testing (30%) sets using stratification to maintain class proportions.
10. **Model Training:** Trained a `RandomForestClassifier` model on the preprocessed, feature-selected, scaled, and SMOTE-balanced training data.
11. **Model Evaluation:** Evaluated the trained model on the unseen test set using accuracy, a detailed classification report (precision, recall, F1-score per class), and a confusion matrix.

## Results

* Initial models struggled significantly with the highly imbalanced nature of the subtypes, primarily predicting the majority class ('Breast Invasive Ductal Carcinoma') and failing to identify minority classes.
* Feature selection (reducing from ~20k to 1000 genes) notably improved the recall for the second largest class ('Breast Invasive Lobular Carcinoma') from ~47% to ~57%.
* Applying SMOTE to the training data (after feature selection) dramatically improved the model's ability to recognize minority classes ('Breast Invasive Carcinoma (NOS)', 'Other_Subtype'), achieving non-zero recall for the first time. It also further improved recall for 'Lobular' carcinoma to ~73%.
* This improvement in minority class detection came with a slight trade-off in reduced recall for the majority class, resulting in a more balanced overall model (Macro Avg F1-score improved significantly).
* The final accuracy remained around 79%, highlighting why accuracy alone is insufficient for evaluating performance on imbalanced datasets.

**Final Classification Report (RandomForest + SelectKBest + SMOTE):**

```text

                                  precision    recall  f1-score   support

  Breast Invasive Carcinoma (NOS)       0.25      0.04      0.07        23
 Breast Invasive Ductal Carcinoma       0.84      0.89      0.87       234
Breast Invasive Lobular Carcinoma       0.64      0.73      0.68        60
                    Other_Subtype       0.75      0.38      0.50         8

                         accuracy                           0.79       325
                        macro avg       0.62      0.51      0.53       325
                     weighted avg       0.76      0.79      0.77       325
