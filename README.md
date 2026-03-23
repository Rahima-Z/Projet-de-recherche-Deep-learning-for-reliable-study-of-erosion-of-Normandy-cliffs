# Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs

# Deep learning for reliable study of erosion of Normandy cliffs

Automated extraction of cliff-top edges from very-high-resolution Pléiades satellite imagery
using semantic segmentation models (U-Net, DeepLabV3+), applied to the Dieppe coastline
(Seine-Maritime, Normandy, France).

> M2 SIC research project — CY University / ENSEA — March 2026
> Author: Rahima ZOUHHAD

---

## Repository structure
```
├── Decoupe_tif.ipynb              # Notebook to slice files contained in \dataset\TIF_files.zip in 256x256 tiles saved in dataset\Images and dataset\Masks
├── Images/                        # 256×256 px image tiles (Pléiades, 8-bit RGB)
├── Masks/                         # Corresponding binary mask tiles (cliff edge = 1)
├── Code_UNet_V12_commenté.ipynb   # U-Net pipeline — fully commented
├── DeepLabV3Plus_V4.ipynb         # DeepLabV3+ V4 (known issues, see below)
└── DeepLabV3Plus_V5.ipynb         # DeepLabV3+ V5 — corrected (training in progress)

```

---

## Dataset

- **Source image**: Pléiades-1A, acquired 5 May 2025 at 07:41 UTC over the Seine-Maritime coastline near Dieppe. Delivered in DIMAP 16-bit format, clipped to 8-bit RGB in QGIS (4695 × 2344 px).
- **Ground truth**: cliff-top edge manually digitised in QGIS and rasterised to a binary mask (cliff = 1, background = 0).
- **Tiling**: 256 × 256 px tiles, sliding window stride = 64, filtered by `MIN_COAST_PIXELS = 50` → **405 tiles** retained from 2310 candidates (17.5%).
- **Split**: 80% train (323 tiles) / 10% val (42) / 10% test (40) via **Iterative Pixel Stratification (IPS)** to ensure balanced class distributions across splits.
- **Class imbalance**: cliff-edge pixels represent ~0.66% of all pixels in the filtered dataset.

---

## Models

### U-Net V12
- Trained **from scratch** (no pretrained backbone), 31 M parameters
- **Loss**: weighted BCE (`pos_weight=10`) + Dice Loss, equal weighting
- **Optimiser**: Adam, lr=1e-4, CosineAnnealingLR (1e-4 → 1e-6 over 50 epochs)
- **Augmentation**: HorizontalFlip, ColorJitter (via albumentations)

### DeepLabV3+ V4
- ResNet-50 backbone pretrained on ImageNet
- Loss: Focal + Tversky — ⚠ **known issues** (automatic `pos_weight ≈ 147` caused near-zero inference probabilities → black output mask). See V5.

### DeepLabV3+ V5 *(training in progress)*
Corrections over V4:
1. `pos_weight` fixed to 10.0 (removing double-penalisation from Focal Loss)
2. `_apply_dilation()` replaced by `_freeze_stride_to_one()` — annuls all residual strides in layer3/layer4
3. `ReduceLROnPlateau` → `CosineAnnealingLR` (avoids premature lr reduction)
4. Inference cell includes automatic probability map diagnostics

---

## Results — U-Net V12

| Metric | Train | Validation | Test |
|---|---|---|---|
| IoU (cliff) | 0.869 | 0.867 | **0.875** |
| Mean IoU | 0.934 | 0.933 | 0.937 |
| Precision | 0.915 | 0.910 | **0.918** |
| Recall | 0.946 | 0.948 | **0.950** |
| F1 | 0.930 | 0.929 | **0.933** |

**Inference settings**: sliding window stride = 128 (50% overlap), binarisation threshold = 0.5, connected-component filter `min_size = 5000`. Output: georeferenced GeoTIFF (Lambert 93, EPSG:2154).

![Probability map](decoupe80_probmap_deeplabv3plus_v4_stride128_png.png)
![Binary mask](decoupe80_deeplabv3plus_v4_stride128_png.png)

> DeepLabV3+ results will be added once V5 training completes.

---

## Installation
```bash
pip install torch torchvision albumentations rasterio scikit-learn scipy tqdm matplotlib
```

The IPS stratification requires the [`SemanticStratification`](https://github.com/SEA-AI/SemanticStratification) library:
```bash
pip install git+https://github.com/SEA-AI/SemanticStratification
```

**Hardware**: all experiments were run on CPU (~8.5 h per 50-epoch U-Net run). GPU strongly recommended for DeepLabV3+.

---

## How to run

1. **Adapt paths** in cell 1 of each notebook (`image_dir`, `mask_dir`, output directories).
2. **Run cells sequentially**. The best model checkpoint is saved automatically based on validation F1.
3. **Run inference** in the last cell with your own Pléiades `.tif` file — the pipeline handles tiling, sliding-window reconstruction, and GeoTIFF export automatically.

---

## Known limitations

- **RGB only** (no NIR band due to QGIS 8-bit conversion): causes false positives on cliff shadows and agricultural field boundaries.
- **Single acquisition**: no temporal analysis yet — cliff retreat rates cannot be computed from a single image.
- **Small dataset**: 323 training tiles from one 4695 × 2344 px scene; aggressive augmentation degrades performance at this scale.

---

## Future work

- Integrate the near-infrared band (full 16-bit Pléiades pipeline)
- Shadow-based semi-automatic labelling (geometrically grounded cliff-edge proxy)
- Multi-temporal series for retreat-rate computation
- Swin Transformer architecture (long-range spatial dependencies)
- Extension to the full 120 km Normandy coastline

-----

## Appendix: Intermediate Results

Prior to the final results presented and analysed in the report, several experimental
iterations were conducted to progressively refine the pipeline.

### Non-overlapping tiling (no stride) for 30 epochs

The first dataset was constructed without any stride, resulting in a limited number of
tiles containing cliff-edge pixels. Both qualitative and quantitative performance were poor.

**Inference output**
![Inference — no stride](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/decoupe_mosaic_sans_stride.png)

**Loss curves**
![Loss — no stride](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/loss_unet_sans_stride.png)

**Evaluation metrics**
![Metrics — no stride](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/metriques_sans_stride.png)

**ROC curve**
![ROC — no stride](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/roc_curve_unet_sans_stride.png)

The low performance is primarily attributable to the scarcity of tiles containing
cliff-edge pixels, which severely limited the model's exposure to the positive class
during training. It should also be noted that the ROC curve is an unreliable performance
indicator in this setting, as the extreme class imbalance inflates the AUC regardless
of the model's actual cliff-detection ability.

---

### Overlapping tiling (stride = 64) with pixel count filtering for 30 epochs

To address the data scarcity issue, the source image was re-tiled using a sliding window
with a stride of 64 pixels, substantially increasing the number of tiles containing
cliff-edge pixels. A minimum pixel-count filter (`MIN_COAST_PIXELS`) was additionally
applied to discard tiles whose masks contained too few positive pixels.

**Inference output**
![Inference — stride + filtering](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/decoupe30_mosaic_stride.png)

**Loss curves**
![Loss — stride + filtering](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/loss_unet_v7_filtr%C3%A9_30_stride.png.png)

**Evaluation metrics**
![Metrics — stride + filtering](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/metriques_stride.png)

In both configurations, the inference outputs show visible tile-boundary artefacts and
an inability to produce a continuous, well-localised cliff-edge prediction. The evaluation
metrics confirm this qualitative observation: precision and recall remain severely
imbalanced, indicating that the model either over-predicts or misses the cliff edge
systematically.

The final results reported in the main section were obtained through two additional
improvements: the adoption of Iterative Pixel Stratification (IPS) for dataset splitting,
which ensures a balanced class distribution across training, validation, and test sets,
and a careful calibration of the hyperparameters described in the accompanying report.


