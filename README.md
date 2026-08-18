# BrainIAC and MRI-CORE Latent Feature Extraction

This repository contains latent features extracted from the **FeTS2024 training dataset** using two foundation models:

* **BrainIAC**
* **MRI-CORE**

The latent features are extracted from the four MRI modalities:

* T1
* T2
* T1ce (TCE)
* FLAIR

The features are stored as `.csv` files.

## Repository Structure

```text
.
├── BrainIAC/
│   ├── Output_240x240x155/
│   │   └── input/
│   │       └── *.csv
│   │
│   ├── Output_cropped_equally/
│   │   └── input/
│   │       └── *.csv
│   │
│   └── Output_cropped_seperate/
│       └── input/
│           └── *.csv
│
└── MRI-CORE/
    ├── Output_240x240x155/
    │   └── input/
    │       └── *.csv
    │
    ├── Output_cropped_equally/
    │   └── input/
    │       └── *.csv
    │
    └── Output_cropped_seperate/
        └── input/
            └── *.csv
```

---

## 1. BrainIAC

The `BrainIAC` folder contains latent embeddings extracted using the **BrainIAC** model.

Each preprocessing strategy is stored in a separate output folder:

### `Output_240x240x155`

The original FeTS2024 MRI volumes have dimensions:

```text
240 × 240 × 155
```

For this experiment, the original images were **resized directly from 240 × 240 × 155 to the required BrainIAC model input size**.

The resized images were then passed through BrainIAC to extract their latent representations.

This approach preserves the complete original field of view while adapting the image dimensions to the model requirements.

### `Output_cropped_equally`

The FeTS2024 images contain zero-padding surrounding the brain region. A common bounding box was identified algorithmically across the FeTS2024 training dataset to remove this surrounding zero-padding.

The common bounding box is:

```text
X: 37  → 215
Y: 10  → 228
Z: 0   → 154
```

Resulting output shape:

```text
(179, 219, 155)
```

The bounding box was selected so that it is applicable to **all FeTS2024 training images**.

After cropping, the resulting volumes were reshaped/resized as necessary to meet the BrainIAC model input requirements, and latent embeddings were extracted.

### `Output_cropped_seperate`

In this preprocessing strategy, the zero-padding was removed **individually for each image** rather than using a single common bounding box.

Therefore, the cropped volumes can have different spatial dimensions depending on the amount of zero-padding present in each image.

The workflow was:

```text
Original FeTS2024 image
        ↓
Individual zero-padding removal
        ↓
Image-specific cropped volume
        ↓
Reshape/resize to BrainIAC input size
        ↓
BrainIAC
        ↓
Latent embedding
```

This allows the cropping to adapt to each individual image instead of forcing every image to use the same bounding box.

---

## 2. MRI-CORE

The `MRI-CORE` folder follows the same three preprocessing strategies described for BrainIAC, but the latent features are extracted using **MRI-CORE**.

### `Output_240x240x155`

The original FeTS2024 images with dimensions:

```text
240 × 240 × 155
```

were resized to the required MRI-CORE model input dimensions before feature extraction.

### `Output_cropped_equally`

A common bounding box was applied to all FeTS2024 training images:

```text
X: 37  → 215
Y: 10  → 228
Z: 0   → 154
```

Resulting shape:

```text
(179, 219, 155)
```

The cropped volumes were subsequently reshaped/resized to the MRI-CORE model input size before extracting latent features.

### `Output_cropped_seperate`

Each FeTS2024 image was individually cropped to remove its surrounding zero-padding.

Because the amount of zero-padding differs between images, the resulting cropped volumes have different spatial dimensions. Each cropped volume was then reshaped/resized to the required MRI-CORE input dimensions before feature extraction.

---

## 3. Latent Embedding Dimensions

Each CSV file contains the latent embeddings extracted from the FeTS2024 training images.

### BrainIAC

BrainIAC produces a **768-dimensional latent representation**.

For the FeTS2024 training dataset:

```text
Number of images: 1251
Embedding dimension: 768

CSV shape:
1251 × 768
```

Therefore, each row represents one FeTS2024 training image and contains its corresponding 768-dimensional BrainIAC embedding.

### MRI-CORE

MRI-CORE produces a **255-dimensional latent representation**.

For the FeTS2024 training dataset:

```text
Number of images: 1251
Embedding dimension: 255

CSV shape:
1251 × 255
```

Each row represents one FeTS2024 training image and contains its corresponding 255-dimensional MRI-CORE embedding.

---

## 4. Preprocessing Comparison

| Folder                    | Preprocessing                   | Bounding Box                   | Resulting Input Before Model           | Feature Extraction  |
| ------------------------- | ------------------------------- | ------------------------------ | -------------------------------------- | ------------------- |
| `Output_240x240x155`      | Resize original volume          | None                           | Original volume resized to model input | BrainIAC / MRI-CORE |
| `Output_cropped_equally`  | Common cropping                 | X: 37–215, Y: 10–228, Z: 0–154 | `(179, 219, 155)` → model input size   | BrainIAC / MRI-CORE |
| `Output_cropped_seperate` | Individual zero-padding removal | Image-specific                 | Variable → model input size            | BrainIAC / MRI-CORE |

---

## 5. Overall Feature Extraction Pipeline

The three preprocessing approaches can be summarized as follows:

### Approach 1 — Original Image Resizing

```text
FeTS2024 MRI
    ↓
Original volume: 240 × 240 × 155
    ↓
Resize to model input size
    ↓
BrainIAC / MRI-CORE
    ↓
Latent feature extraction
    ↓
CSV embeddings
```

### Approach 2 — Common Bounding Box

```text
FeTS2024 MRI
    ↓
Common zero-padding removal
    ↓
Bounding box:
X: 37–215
Y: 10–228
Z: 0–154
    ↓
Shape: 179 × 219 × 155
    ↓
Resize/reshape to model input size
    ↓
BrainIAC / MRI-CORE
    ↓
Latent feature extraction
    ↓
CSV embeddings
```

### Approach 3 — Individual Cropping

```text
FeTS2024 MRI
    ↓
Individual zero-padding removal
    ↓
Image-specific cropped volume
    ↓
Variable spatial dimensions
    ↓
Resize/reshape to model input size
    ↓
BrainIAC / MRI-CORE
    ↓
Latent feature extraction
    ↓
CSV embeddings
```

---

## 6. Purpose of the Three Preprocessing Strategies

The three approaches are intended to evaluate how different spatial preprocessing strategies affect the latent representations extracted by BrainIAC and MRI-CORE.

### 1. Original resizing

Provides a baseline in which the complete original FeTS2024 volume is retained and directly adapted to the model input dimensions.

### 2. Common cropping

Removes a shared region of zero-padding using one bounding box that is valid for the entire FeTS2024 training dataset.

This provides a standardized cropped representation across all subjects.

### 3. Individual cropping

Removes zero-padding separately for each image.

This produces a tighter image-specific representation but results in variable original spatial dimensions before the final resizing/reshaping step.

---

## 7. CSV Data Format

The extracted latent features are stored in CSV format.

For BrainIAC:

```text
Rows    = FeTS2024 training images
Columns = 768 latent features
Shape   = 1251 × 768
```

For MRI-CORE:

```text
Rows    = FeTS2024 training images
Columns = 255 latent features
Shape   = 1251 × 255
```

## 8. Dataset

The feature extraction is performed on the **FeTS2024 training dataset**.

The four MRI modalities considered are:

```text
T1
T2
T1ce / TCE
FLAIR
```



