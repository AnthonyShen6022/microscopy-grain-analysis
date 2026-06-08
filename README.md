# microscopy-grain-analysis

# Grain Size Analysis via SAM 2

Automated grain segmentation and size measurement from Polarized Optical Microscopy (POM) images using Meta's [Segment Anything Model 2 (SAM 2)](https://github.com/facebookresearch/sam2).

## Overview

This pipeline takes a raw POM image and a known scale factor (mm/pixel), segments individual grains using SAM 2, and computes per-grain size

## Example Output

| Raw POM Image | Segmented Overlay | Size Distribution |
|---|---|---|
| ![raw](docs/example_raw.png) | ![segmented](docs/example_segmented.png) | ![histogram](docs/example_histogram.png) |

## Requirements

- Python 3.9+
- CPU or CUDA-enabled GPU 

Install dependencies:

```bash
pip install torch torchvision
pip install git+https://github.com/facebookresearch/sam2.git
pip install opencv-python numpy pandas matplotlib scipy ipywidgets
pip install jupyter
```

## SAM 2 Model Checkpoint

Download a checkpoint from the [SAM 2 model zoo](https://github.com/facebookresearch/sam2#model-checkpoints) and place it in a `checkpoints/` folder.

| Checkpoint | Speed (CPU) | Accuracy |
|---|---|---|
| `sam2.1_hiera_tiny.pt` | Fastest | Good |
| `sam2.1_hiera_small.pt` | Fast | Better |
| `sam2.1_hiera_large.pt` | Slow | Best |

> **Note:** Checkpoints are not included in this repository due to file size. Do not commit `.pt` files to Git.

## Usage

### Jupyter Notebook (recommended)

```bash
jupyter notebook grain_segmentation.ipynb
```

Edit the inputs cell at the top:

```python
IMAGE_PATH      = "path/to/your_image.tif"
MM_PER_PIXEL    = 0.000423          # your scale factor
SAM2_CHECKPOINT = "checkpoints/sam2.1_hiera_tiny.pt"
```

Then run all cells top to bottom.

### Python Script

```bash
python grain_segmentation.py your_image.tif 0.000423 --checkpoint checkpoints/sam2.1_hiera_tiny.pt
```

## Output Files

For an input image `my_image.tif`, the pipeline produces:

| File | Description |
|---|---|
| `my_image_grain_stats.csv` | Per-grain: area (px, mm²), equivalent diameter (µm), centroid |
| `my_image_segmented.png` | Coloured mask overlay on original image |
| `my_image_results_summary.png` | 3-panel figure: raw / segmented / size histogram |

### CSV columns

| Column | Description |
|---|---|
| `grain_id` | Index of the grain |
| `area_px` | Area in pixels² |
| `area_mm2` | Area in mm² |
| `equiv_diameter_px` | Equivalent circular diameter in pixels |
| `equiv_diameter_um` | Equivalent circular diameter in µm |
| `centroid_x/y` | Centroid position in pixels |

Equivalent diameter is computed as:

$$d = 2\sqrt{\frac{A}{\pi}}$$

## Tuning Parameters

| Parameter | Default | Effect |
|---|---|---|
| `points_per_side` | 32 | Denser grid → finds smaller grains. Increase for fine-grained samples |
| `pred_iou_thresh` | 0.80 | Higher → stricter mask quality filter |
| `stability_score_thresh` | 0.90 | Higher → removes unstable/fragmented masks |
| `MIN_GRAIN_AREA_PX` | 200 | Minimum grain area in px² — raise to filter noise |
| `GAUSSIAN_KSIZE` | (5,5) | Increase for high-resolution images |
| `MEDIAN_KSIZE` | 5 | Must be odd. Increase to suppress salt-and-pepper noise |

## Performance Notes

SAM 2 on CPU is slow for large images. If runtime is too long:
- Use `sam2.1_hiera_tiny.pt` (fastest checkpoint)
- Downscale the image before segmentation and adjust `MM_PER_PIXEL` proportionally

## References

- Kirillov, A. et al. (2023). [Segment Anything](https://arxiv.org/abs/2304.02643). arXiv.
- Ravi, N. et al. (2024). [SAM 2: Segment Anything in Images and Videos](https://arxiv.org/abs/2408.00714). arXiv.

## License

MIT
