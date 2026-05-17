# OCR Training with Augmented Handwritten Line Images

This project presents an OCR pipeline for handwritten line recognition. The model was trained on an augmented handwritten text-line dataset using a CRNN architecture with CTC loss. The main objective was to improve recognition robustness through image augmentation while preserving a leakage-safe train, validation, and test split.

## Project Overview

The task is to recognize handwritten text from line images. The dataset contains original handwritten line images and augmented versions of selected images. Augmented images were used only for training, while validation and test sets contained only original images.

The project includes image preprocessing, dataset splitting, CRNN model training, OCR evaluation, and visualization of model performance.

## Dataset

The metadata file `sentences_augmented.csv` is included in this repository.

The file contains four columns:

- `id`: image filename
- `sentence`: ground-truth transcription
- `original_id`: identifier of the original image
- `augmentation`: augmentation type

The image folder is not stored directly in this repository because of its size. It should be downloaded separately from Google Drive:

[Dataset download link] https://drive.google.com/file/d/1Xn_mz3IdTN2P537cPfADBy6aRBeXpsHH/view?usp=sharing

After downloading and unzipping the image archive, the project folder should contain the notebook, the CSV file, and the folder `lines flattened augmented`.


## Data Augmentation

The augmented dataset contains original handwritten line images and transformed versions of selected images. The applied augmentations include Gaussian noise, rotation, compression, blur, affine transformation, erosion, dilation, scale-resizing, and sharpening.

The purpose of augmentation was to expose the OCR model to common image distortions and improve its ability to generalize to unseen handwritten text.


## Leakage-Safe Splitting Strategy

A key methodological requirement was to avoid data leakage between original images and their augmented versions.

For this reason, the dataset was split by `original_id`, not by individual image rows. This ensured that an original image and all of its augmented versions always belonged to the same subset.

The training set contains both original and augmented images. The validation and test sets contain only original images. Original images with augmented versions were forced into the training set, ensuring that no augmented version of a validation or test image appeared during training.

The final split contained:

- 10,517 training observations
- 1,868 validation observations
- 1,868 test observations


## Model

The OCR model is based on a CRNN architecture. First, convolutional layers extract visual features from the handwritten line images. Then, these features are treated as a sequence and passed through bidirectional LSTM layers. Finally, the model is trained with CTC loss, which allows sequence prediction without requiring character-level alignment.

This architecture is appropriate for handwritten line recognition because the exact position of each character in the image is not known in advance.

## Handling Variable-Width Images

Handwritten line images have different widths. Instead of padding all images to the same width, the model was trained with batch size equal to 1. This allowed every image to keep its natural width and avoided adding artificial empty regions.

To avoid unstable updates after every single image, gradient accumulation was used. The model processed images one by one, but the optimizer was updated only after several samples. This preserved the practical effect of a larger batch size without requiring padding.


## Evaluation Metrics

The model was evaluated using CTC loss, Character Error Rate, Word Error Rate, and Exact Match Rate.

Character Error Rate is especially important for OCR because it measures recognition quality at the character level. Word Error Rate and Exact Match Rate are stricter because even a small character-level mistake can make a whole word or full line incorrect.

## Results

The final evaluation produced the following results:

| Dataset | Mean Loss | CER | WER | Exact Match Rate |
|---|---:|---:|---:|---:|
| Train | 0.476 | 0.132 | 0.442 | 0.107 |
| Validation | 0.614 | 0.159 | 0.507 | 0.062 |
| Test | 0.636 | 0.162 | 0.505 | 0.055 |

The validation and test results are very similar, which suggests that the evaluation procedure was stable and that the leakage-safe split worked as intended. The final test Character Error Rate of approximately 16.2% indicates that the model learned meaningful character-level recognition patterns.

## Repository Contents

The repository contains:

- `modelling_v1.ipynb`
- `sentences_augmented.csv`
- `results/`
- `README.md`
- `.gitignore`

Large image files and model checkpoints are excluded from the repository. The image dataset should be downloaded separately from the provided Google Drive link.


## Reproducibility Notes

To reproduce the project, download the image archive from the Google Drive link, unzip it into the project folder, and run the notebook from top to bottom.

The notebook expects the image folder to be named:

`lines flattened augmented`