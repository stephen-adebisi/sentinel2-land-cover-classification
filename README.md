# 🛰️ Mapping Land Cover with Sentinel-2
### Random Forest vs. an Encoder–Decoder CNN in Brookings County, South Dakota

🌍 Earth Observation · 🧠 Machine Learning · 🎨 Semantic Segmentation · 🐍 Python

## 📌 Overview

This project explores land-cover classification using five-band Sentinel-2 imagery and ESA WorldCover reference labels.

Two approaches are implemented:

- **Random Forest:** classifies pixels using their spectral values.
- **Encoder–Decoder CNN:** predicts pixel-level land-cover categories using spatial and spectral information.

The workflow covers image preparation, class-distribution analysis, model training, evaluation, and visualization of predicted maps.

## 🎯 Objectives

- Build a workflow for classifying multispectral satellite imagery.
- Compare a traditional machine learning approach with a convolutional segmentation model.
- Examine how class imbalance affects interpretation of model performance.
- Visualize differences between reference labels and predicted land-cover maps.

## 🌍 Study Area and Data

The study focuses on **Brookings County, South Dakota, USA**.

| Component | Description |
|---|---|
| Satellite imagery | Sentinel-2 |
| Input channels | Five bands, treated as Blue, Green, Red, NIR, and SWIR |
| Reference labels | ESA WorldCover |
| Image patches | 1,322 |
| CNN patch dimensions | 256 × 256 pixels |
| Training partition | 925 patches |
| Evaluation partition | 397 patches |
| Patch split | Random 70/30 split, seed 42 |

Exact imagery dates, WorldCover version, band identifiers, and patch-generation settings should be confirmed from the source-data preparation records.

ESA WorldCover is used as a reference map. Agreement with its labels does not constitute independent field validation.

## 🎨 Land-Cover Categories

| WorldCover ID | Category |
|---:|---|
| 10 | Tree cover |
| 30 | Grassland |
| 40 | Cropland |
| 50 | Built-up |
| 60 | Bare / sparse vegetation |
| 80 | Permanent water bodies |
| 90 | Herbaceous wetland |

The CNN remaps these IDs to consecutive values from 0 to 6.

## ⚙️ Workflow

1. Load matching image patches and label masks.
2. Create a reproducible patch-level split.
3. Extract spectral features for Random Forest.
4. Inspect the sampled training-class distribution.
5. Prepare image tensors and remap labels for the CNN.
6. Train both models.
7. Calculate classification metrics.
8. Compare imagery, reference labels, and predicted maps visually.

## 🌲 Random Forest

The Random Forest model uses:

- 100 decision trees
- Five spectral features per pixel
- Random seed of 42
- Parallel CPU processing
- Every 100th valid pixel from each patch for training and evaluation

Pixels with reference label 0 are excluded during Random Forest feature extraction.

## 🧠 Encoder–Decoder CNN

The segmentation network contains:

- Two encoder stages with 32 and 64 convolutional filters
- Max pooling for downsampling
- A bottleneck with 128 filters
- Two decoder stages with upsampling
- A seven-class softmax output

Training settings:

| Setting | Value |
|---|---|
| Framework | TensorFlow / Keras |
| Optimizer | Adam |
| Initial learning rate | 0.0001 |
| Loss | Sparse categorical cross-entropy |
| Batch size | 8 |
| Maximum epochs | 50 |
| Early stopping | Validation-loss patience of 5 |
| Weight restoration | Best validation-loss weights |

**Architecture note:** this network has no encoder-to-decoder skip connections. It is an encoder–decoder CNN rather than a standard U-Net.

## 📊 Recorded Results

The uploaded notebook contains the following saved outputs:

| Model | Recorded pixel accuracy |
|---|---:|
| Random Forest | 83.76% |
| Encoder–Decoder CNN | 86.81% |

Additional recorded CNN metrics:

| Metric | Value |
|---|---:|
| Weighted precision | 0.8663 |
| Weighted recall | 0.8681 |
| Weighted F1 | 0.8667 |
| Weighted IoU | 0.7720 |

**These are preliminary notebook results, not a controlled independent-test comparison.**

The CNN evaluation partition was also used to monitor validation loss and select model weights. Random Forest was evaluated on subsampled valid pixels, while CNN metrics were calculated over full label arrays.

The Random Forest training-normalization code also needs reconciliation with the test preprocessing before its score can be reproduced reliably.

The reported IoU is **class-frequency-weighted IoU**, not mean IoU.

## ⚖️ Class Imbalance

The sampled Random Forest training data were strongly imbalanced:

| Category | Share of sampled training pixels |
|---|---:|
| Cropland | 62.36% |
| Grassland | 29.60% |
| Tree cover | 3.52% |
| Permanent water bodies | 2.85% |
| Built-up | 1.44% |
| Bare / sparse vegetation | 0.14% |
| Herbaceous wetland | 0.10% |

This distribution makes class-specific evaluation essential.

A high overall accuracy can coexist with poor predictions for uncommon categories. In the recorded Random Forest report, herbaceous wetland had an F1 score of approximately 0.01.

## 🖼️ Visual Outputs

The notebook includes:

- Satellite image and reference-label examples
- A normalized Random Forest confusion matrix
- CNN training and validation curves
- Side-by-side reference and prediction maps
- Predictions for multiple evaluation patches

Visual comparisons help identify missed features and differences that aggregate metrics can hide.

## 🚀 Getting Started

### Dependencies

Install the main packages:

    pip install numpy pandas matplotlib seaborn scikit-learn tensorflow tifffile imagecodecs pillow jupyter

### Prepare the data

The notebook expects this folder structure:

    Patches/
        images/
            example.tif
        labels/
            example.tif

Each image must have a matching label file with the same filename.

### Run the notebook

1. Download the project notebook.
2. Obtain or recreate the imagery patches and corresponding labels.
3. Update `base_path` to your local `Patches` directory.
4. Confirm the image band order and array dimensions.
5. Run the cells in sequence.

The imagery patches and trained model weights are not embedded in the notebook. A GPU is helpful for CNN training.

## 🔍 Limitations and Next Steps

- **Preprocessing consistency:** apply identical reflectance scaling to Random Forest training and evaluation inputs.
- **Independent evaluation:** create separate training, validation, and test partitions.
- **Spatial generalization:** split by spatial blocks or source scenes to reduce dependence between nearby patches.
- **No-data handling:** explicitly mask no-data and unmapped labels in CNN training and metrics; the current mapping defaults them to the tree-cover index.
- **Comparable metrics:** evaluate both models on the same valid pixels.
- **Balanced reporting:** add per-class IoU, mean IoU, and macro F1.
- **Architecture extension:** implement U-Net skip connections and retrain for a separate experiment.
- **Reproducibility:** document data dates, source versions, package versions, and patch-generation settings.

## 🛠️ Tools

Python · NumPy · pandas · scikit-learn · TensorFlow/Keras · Matplotlib · Seaborn · tifffile

## 📚 Data Sources

- [Copernicus Sentinel-2](https://www.copernicus.eu/en/about-copernicus/infrastructure/discover-our-satellites)
- [ESA WorldCover](https://esa-worldcover.org/en)

Source imagery and reference products remain subject to their respective attribution and usage terms.

## 💡 Project Takeaway

This project demonstrates an end-to-end land-cover classification workflow and highlights why preprocessing, reference-label quality, class imbalance, and evaluation design matter alongside model choice.
