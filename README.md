# OGH-Net: Orientation-Guided Homography for Fine-Grained Cross-View Localization

<p align="center">
  <a href="https://doi.org/10.1109/TIP.2026.3697650">
    <img src="https://img.shields.io/badge/Paper-IEEE%20TIP-00629B.svg" alt="Paper">
  </a>
  <a href="https://github.com/YC-Zhang2025/OGH-Net">
    <img src="https://img.shields.io/badge/Code-GitHub-181717.svg" alt="Code">
  </a>
  <img src="https://img.shields.io/badge/PyTorch-Implementation-EE4C2C.svg" alt="PyTorch">
</p>

<p align="center">
  <b>Yangchun Zhang</b>, Xudong Kang, Puhong Duan, Shutao Li, Ze Song
</p>

<p align="center">
  IEEE Transactions on Image Processing (TIP), 2026
</p>

Official implementation of **“Orientation-Guided Homography for Fine-Grained Cross-View Localization.”**

OGH-Net estimates the precise **2D position and yaw orientation** of a ground-view camera by aligning a ground panorama with a geo-referenced satellite image. The method is designed for fine-grained cross-view localization under large initial orientation uncertainty.

> **Scope.** OGH-Net addresses fine-grained pose refinement given a corresponding satellite patch. It is not intended as a city-scale image-retrieval system.

<!--
Add the overview image from Fig. 2 to assets/oghnet_overview.png, and then uncomment:

<p align="center">
  <img src="assets/oghnet_overview.png" width="95%" alt="OGH-Net overview">
</p>
-->

## News

- **2026:** The paper was published in *IEEE Transactions on Image Processing*.
- Code, pretrained models, and data-preparation instructions will be released progressively.

## Highlights

- **Hybrid BEV transformation.** Smoothly combines spherical and polar-style mappings to preserve central geometry, reduce peripheral distortion, and expand the usable field of view.
- **Full-range orientation prior.** A lightweight orientation-prior estimation module predicts a coarse yaw hypothesis over the full **±180°** range from a global cross-view correlation volume.
- **Ambiguity-aware orientation decision.** The AAOD strategy mitigates opposing-direction ambiguity in urban-canyon and repetitive-structure scenes.
- **Multiscale homography refinement.** Coarse-to-fine iterative correlation matching progressively refines the homography and recovers camera position and yaw.
- **Real-time inference.** OGH-Net reaches up to **107 FPS** on a single NVIDIA RTX 3090 while maintaining competitive localization accuracy.

## Method

OGH-Net contains three principal components:

### 1. Hybrid BEV Transformation

The ground panorama is transformed into a bird's-eye-view representation. The proposed mapping preserves the geometric consistency of the central region while reducing peripheral distortion and increasing spatial coverage.

The recommended parameters used in the paper are:

| Parameter | Value |
|---|---:|
| Field of view | 85° |
| Transition radius \(r^*\) | 0.65 |

### 2. Orientation-Prior Estimation

The deepest ground and satellite features are used to construct a global correlation volume. An orientation decoder predicts confidence scores for four coarse yaw hypotheses:

\[
\{-90^\circ,\ 0^\circ,\ 90^\circ,\ 180^\circ\}.
\]

The resulting yaw prior initializes the subsequent homography refinement. The ambiguity-aware orientation decision threshold used in the paper is:

| Parameter | Value |
|---|---:|
| AAOD threshold \(\tau\) | 0.04 |

### 3. Multiscale Homography Iteration

The initial homography is refined from coarse to fine. At each feature scale, local correlation volumes are decoded into residual corner displacements, which are iteratively accumulated to update the homography matrix.

The final homography is used to recover the camera position and yaw.

## Quantitative Results

### VIGOR

| Setting | Mean location error (m) | Median location error (m) | Mean yaw error (°) | Median yaw error (°) |
|---|---:|---:|---:|---:|
| Same-Area, ±45° | 2.36 | 1.03 | 1.74 | 0.78 |
| Cross-Area, ±45° | 3.16 | 1.48 | 2.44 | 1.03 |
| Same-Area, ±180° | 3.98 | 1.55 | 12.78 | 2.53 |
| Cross-Area, ±180° | 5.03 | 2.33 | 20.22 | 3.72 |

Under the **Cross-Area, ±180°** setting, OGH-Net reduces the mean localization error by **11%** and the mean orientation error by **27%** relative to the comparison method reported in the paper.

### KITTI

| Setting | Mean location error (m) | Median location error (m) | Mean yaw error (°) | Median yaw error (°) |
|---|---:|---:|---:|---:|
| Same-Area, ±10° | 0.77 | 0.42 | 0.37 | 0.28 |
| Cross-Area, ±10° | 7.50 | 4.12 | 2.69 | 1.25 |
| Same-Area, ±180° | 0.91 | 0.68 | 10.17 | 7.35 |
| Cross-Area, ±180° | 10.19 | 9.24 | 30.61 | 13.10 |

Under the **Cross-Area, ±180°** setting, OGH-Net reduces the mean localization error by **27%** relative to the comparison method reported in the paper.

### Computational Efficiency

Measured using an Intel Core i9-12900K CPU and a single NVIDIA RTX 3090 GPU:

| Metric | OGH-Net |
|---|---:|
| Parameters | 12.58 M |
| Peak inference memory | 1,972 MiB |
| Inference time | 9.3 ms/image |
| Maximum throughput | 107 FPS |

## Reproduction Settings

The principal settings reported in the paper are listed below.

| Item | Setting |
|---|---|
| Backbone | EfficientNet-B0 |
| Feature extraction | Pseudo-Siamese branches with independent weights |
| Input resolution | 512 × 512 |
| Feature scales | 64 × 64 and 32 × 32 |
| Refinement iterations | 4 iterations per scale |
| Optimizer | Adam |
| Maximum learning rate | \(3.8 \times 10^{-4}\) |
| Batch size | 20 |
#| Training epochs | 68 |
#| Position-loss weight \(\alpha_1\) | 0.1 |
#| Orientation-loss weight \(\alpha_2\) | 1 |
#| Classification-loss weight \(\alpha_3\) | 50 |
#| Iteration decay factor \(\lambda\) | 0.85 |
#| Local correlation radius | 4 (9 × 9 window) |

## Installation

<!-- TODO: Update dependency versions after testing the public repository. -->

```bash
git clone https://github.com/YC-Zhang2025/OGH-Net.git
cd OGH-Net

conda create -n oghnet python=3.10 -y
conda activate oghnet

pip install -r requirements.txt
```

## Dataset Preparation

### VIGOR

Download VIGOR from its [official repository](https://github.com/Jeff-Zilence/VIGOR) and follow the original license and usage terms.

### KITTI

Download the required KITTI images and calibration data from the [official KITTI website](https://www.cvlibs.net/datasets/kitti/).

A recommended directory layout is:

```text
data/
├── VIGOR/
│   ├── panorama/
│   ├── satellite/
│   └── labels/
└── KITTI/
    ├── image_02/
    ├── calibration/
    ├── satellite/
    └── labels/
```

<!-- TODO: Replace the directory tree above with the exact paths expected by the released dataloader. -->

## Pretrained Models

Pretrained weights will be provided for the following settings:

| Dataset | Same-Area | Cross-Area | Orientation setting |
|---|:---:|:---:|---|
| VIGOR | ☐ | ☐ | 0°, ±45°, ±180° |
| KITTI | ☐ | ☐ | ±10°, ±180° |

<!-- TODO: Add checkpoint links, file sizes, and SHA-256 hashes. -->

## Training

<!-- TODO: Update script names and configuration paths to match the final repository. -->

Example:

```bash
# VIGOR: cross-area training with unknown orientation
python train.py \
  --config configs/vigor_cross_area_180.yaml \
  --data-root data/VIGOR \
  --output-dir outputs/vigor_cross_area_180
```

To reproduce the paper settings, ensure that:

1. Ground and satellite images are resized to 512 × 512.
2. Two feature-pyramid levels are used.
3. Four homography-refinement iterations are performed at each level.
4. Orientation perturbations follow the setting defined by the selected experiment.

## Evaluation

<!-- TODO: Update the command-line arguments to match the final implementation. -->

```bash
python evaluate.py \
  --config configs/vigor_cross_area_180.yaml \
  --checkpoint checkpoints/oghnet_vigor_cross_area_180.pth \
  --data-root data/VIGOR
```

The main evaluation metrics are:

- Mean and median localization error in meters.
- Localization accuracy within 1 m, 3 m, and 5 m.
- Mean and median orientation error in degrees.
- Orientation accuracy within 1°, 3°, and 5°.

## Inference

<!-- TODO: Update the demo entry point and input specification. -->

```bash
python demo.py \
  --ground-image examples/ground_panorama.jpg \
  --satellite-image examples/satellite_patch.jpg \
  --checkpoint checkpoints/oghnet_vigor.pth \
  --output-dir outputs/demo
```

Expected outputs include:

- Estimated horizontal and vertical position offsets.
- Estimated camera yaw.
- Estimated homography matrix.
- Optional cross-view alignment visualization.

## Repository Status

- [ ] Data-preparation scripts
- [ ] Training code
- [ ] Evaluation code
- [ ] Inference demo
- [ ] Pretrained VIGOR models
- [ ] Pretrained KITTI models
- [ ] Reproduction logs


## Citation

If this work is useful in your research, please cite:

```bibtex
@article{zhang2026orientation,
  title   = {Orientation-Guided Homography for Fine-Grained Cross-View Localization},
  author  = {Zhang, Yangchun and Kang, Xudong and Duan, Puhong and Li, Shutao and Song, Ze},
  journal = {IEEE Transactions on Image Processing},
  volume  = {35},
  pages   = {6139--6152},
  year    = {2026},
  doi     = {10.1109/TIP.2026.3697650}
}
```

## Acknowledgements

This work builds on research in fine-grained cross-view localization, learned homography estimation, and bird's-eye-view transformation. We thank the authors and maintainers of VIGOR, KITTI, EfficientNet, and the related open-source projects used in our experiments.

## License

The source-code license will be specified in the `LICENSE` file. The VIGOR and KITTI datasets are distributed under their respective licenses and terms of use.


