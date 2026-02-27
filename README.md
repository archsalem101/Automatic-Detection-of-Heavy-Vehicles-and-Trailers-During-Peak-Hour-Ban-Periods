# Automatic Detection of Heavy Vehicles & Trailers During Peak-Hour Ban Periods

> **Module 4 · Unit 3 · FMP Group Assignment**
> MAICEN 1125 — AI & Computer Vision in the Built Environment

---

## Problem Statement

### Urban Planning Context

Many cities apply **time-based restrictions** on heavy vehicles — morning and evening peak-hour bans — to reduce congestion on key corridors, improve travel times for commuters and public transport, and reduce conflicts between freight and passenger traffic.

These policies are **hard to monitor in real time**. Manual checks by enforcement teams and occasional camera reviews are not sufficient to answer critical urban management questions:

- How often are heavy-vehicle bans actually violated?
- On which **roads and time windows** are violations concentrated?
- Are the restrictions genuinely **easing congestion**, or being routinely ignored?

Without reliable data, transport planners cannot evaluate policy effectiveness, allocate enforcement resources efficiently, or justify investment in infrastructure changes.

### Our Approach

This project demonstrates how an **object detection model** can serve as a building block in a **smart urban traffic management system**. We train a YOLOv8 model to automatically detect heavy vehicles and trailers in peak-hour traffic images captured by roadside cameras. The model outputs become structured data — class, confidence, timestamp — that feeds directly into an enforcement and analytics pipeline.

The intended deployment: a camera feed passes frames through the model during ban windows; any frame containing a `bus` or `truck` triggers a timestamped alert for human review. Aggregated detections over time reveal violation hotspots and inform evidence-based policy revision.

**Success criteria:**
- mAP@0.5 ≥ 0.70 on the validation split
- Precision ≥ 0.70 and Recall ≥ 0.65 for `bus` and `truck` classes (the enforcement-critical classes)
- Notebook runs end-to-end in Google Colab (GPU T4) without local installs

---

## Class Definitions & Label Rules

The dataset `data.yaml` contains **5 class entries** due to capitalisation inconsistencies introduced during annotation. These map to 3 semantic categories:

| Class ID | Label in data.yaml | Semantic class | Description | Label rule |
|----------|--------------------|---------------|-------------|------------|
| 0 | `Car` | car | Passenger car (capitalised variant) | Tight bounding box; exclude shadows |
| 1 | `Trucks` | truck | HGV / lorry (capitalised variant) | Include cab + trailer if attached |
| 2 | `bus` | bus | Single- or double-deck passenger bus, minibus ≥ 5m | Tight bounding box; include mirrors |
| 3 | `car` | car | Passenger car, SUV, van < 3.5 t GVW | Tight bounding box; exclude shadows |
| 4 | `truck` | truck | HGV, lorry, rigid or articulated; any vehicle > 3.5 t GVW | Include cab + trailer; one box per unit |

> ⚠️ **Known labelling issue:** Classes `Car`/`car` and `Trucks`/`truck` are semantic duplicates with different capitalisation. This reduces effective training signal for heavy-vehicle classes and is a priority fix for the next dataset version (see [`/docs/error_analysis.md`](./docs/error_analysis.md)).

**Exclusions:** motorcycles, cyclists, pedestrians, static objects. Partially visible vehicles (< 30 % visible) are skipped.

---

## Dataset

| Property | Value |
|----------|-------|
| Host | GitHub (this repo) |
| Folder | [`/images dataset/`](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) |
| Original source | Roboflow — Automatic Detection of Heavy Vehicles v1 |
| Total images | 360 (train: 273 · valid: 87) |
| Split | ~76 % train / ~24 % validation |
| Classes | 5 labels in `data.yaml` (3 semantic classes — see note above) |
| Format | YOLOv8 (normalised XYWH) |
| License | CC BY 4.0 |
| **Dataset link** | [View dataset on GitHub](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) |
| Roboflow source | [app.roboflow.com — project v1](https://app.roboflow.com/salem-w0yva/automatic-detection-of-heavy-vehicles-and-trailers-during-peak-hour-ban-periods/1) |

> The dataset is stored directly in this repository under `images dataset/`. The notebooks clone the repo automatically — **no Roboflow account or API key required**.

---

## Quick Start — How to Reproduce (Google Colab)

The repo uses **two notebooks** — run them in order:

### Step 1 — Training
[![Open 01_Training in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/blob/main/notebooks/01_Training.ipynb)

1. Open `notebooks/01_Training.ipynb` in Colab.
2. **Runtime → Change runtime type → T4 GPU**
3. Set `VERIFY_ONLY = False` for full 50-epoch training (or `True` for a 5-epoch quick check).
4. Run all cells. The notebook will clone this repo automatically and download the dataset.
5. Outputs: metrics table, training curves, `best.pt` weights, `training_outputs.zip`.

> **No API key needed** — dataset is pulled directly from this GitHub repo (Cell 3).

### Step 2 — Inference
[![Open 02_Inference in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/blob/main/notebooks/02_Inference.ipynb)

1. Open `notebooks/02_Inference.ipynb` in Colab (same session or new session).
2. Pre-trained weights are downloaded automatically from [`/02_results/best.pt`](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/raw/main/02_results/best.pt) in Cell 2 (Option C). No manual upload needed.
3. Run all cells. The notebook clones the repo for validation images automatically.
4. Outputs: 10 validation prediction images, 5 new-image predictions, SAM notes, `evidence_package.zip`.

> **If GPU is unavailable:** set `VERIFY_ONLY = True` in `01_Training.ipynb` (5 epochs, ~3–5 min on CPU), then load pre-trained weights in `02_Inference.ipynb` for full inference. Document this clearly in the reproducibility log.

---

## Results Summary

> Training run completed: **27/02/2026** · Tesla T4 · 50 epochs · ~28 min

### Overall Metrics

| Metric | Value |
|--------|-------|
| Precision | **0.633** |
| Recall | **0.599** |
| mAP@0.5 | **0.601** |
| mAP@0.5:0.95 | **0.449** |

### Per-Class Metrics

| Class | Precision | Recall | mAP@0.5 | mAP@0.5:0.95 |
|-------|-----------|--------|---------|--------------|
| Car *(capitalised)* | 0.450 | 0.491 | 0.420 | 0.246 |
| Trucks *(capitalised)* | 0.340 | 0.500 | 0.384 | 0.289 |
| bus | 0.734 | 0.487 | 0.514 | 0.433 |
| car | 0.829 | 0.779 | **0.875** | 0.641 |
| truck | 0.811 | 0.736 | **0.812** | 0.634 |

### Key Takeaways

1. **`car` and `truck` (lowercase) perform well** — mAP@0.5 of 0.875 and 0.812 respectively, both exceeding the 0.70 success criterion.
2. **Capitalised duplicate labels hurt heavy-vehicle performance** — `Car` (mAP 0.42) and `Trucks` (mAP 0.38) are the same semantic classes as `car` and `truck` but split training signal across two label IDs, dragging down overall mAP. Merging these is the single highest-priority fix.
3. **`bus` recall is low (0.487)** — the model finds buses when they are visible, but misses nearly half of them. More diverse bus images (side views, partial occlusion, night) are needed.

Training curves and prediction samples are in [`/results/`](./results/).

---

## Reproducibility Checklist

- [x] Dataset: GitHub repo `/images dataset/` — [link](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) (Roboflow CC BY 4.0, v1)
- [x] Model variant: `yolov8n.pt` (nano)
- [x] Epochs: 50 (verification mode: 5)
- [x] Batch size: 16
- [x] Image size: 640 × 640
- [x] Ultralytics version: `ultralytics==8.4.18`
- [x] PyTorch version: `2.10.0+cu128`
- [x] Platform: Google Colab · Python 3.12.12 · CUDA 13.0
- [x] Trained weights: [`/02_results/best.pt`](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/raw/main/02_results/best.pt)
- [x] Notebooks: `01_Training.ipynb` → `02_Inference.ipynb` (run in order)

---

## Reproducibility Proof

| Field | Value |
|-------|-------|
| Last successful run | 27/02/2026 · ~02:14–04:51 UTC |
| GPU | Tesla T4 (15,360 MiB) |
| CUDA version | 13.0 |
| Ultralytics | 8.4.18 |
| PyTorch | 2.10.0+cu128 |
| Python | 3.12.12 |
| Runtime (50 epochs) | ~28 min on T4 GPU |
| Runtime (5-epoch verify) | ~3–5 min on T4 GPU |
| mAP@0.5 achieved | 0.601 |

---

## Repository Structure

```
├── README.md
├── notebooks/
│   ├── 01_Training.ipynb          ← install, download data, train, metrics, curves
│   └── 02_Inference.ipynb         ← load weights, val predictions, new images, SAM notes
├── results/
│   ├── curves/                    ← loss/mAP/confusion/PR curves (PNG)
│   └── evidence/                  ← annotation + prediction screenshots
└── docs/
    ├── class_definitions.md       ← detailed label rules
    ├── error_analysis.md          ← FP/FN analysis + data improvement plan
    └── governance_checklist.md    ← data provenance, PII, risk, human-in-loop, licences
```

---

## Governance & Licensing

- Code: **MIT License**
- Dataset: **CC BY 4.0** (hosted in this repo; originally from Roboflow Smart City Cars Detection v1)
- See [`/docs/governance_checklist.md`](./docs/governance_checklist.md) for full compliance notes.

---

## References

- Jocher, G. et al. (2023). *Ultralytics YOLOv8*. [https://github.com/ultralytics/ultralytics](https://github.com/ultralytics/ultralytics)
- Roboflow (2026). *Smart City Cars Detection Dataset v1*. CC BY 4.0. Hosted at: [/images dataset/](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset)
- MAICEN 1125 Lecture Notes — Module 4, Unit 3 (LS2, LS3, LS4).
- Ultralytics Colab Reference Notebook: [https://colab.research.google.com/drive/1CFqZ75Hxgjvt6p5Pv713H_3-V73cQIbD](https://colab.research.google.com/drive/1CFqZ75Hxgjvt6p5Pv713H_3-V73cQIbD)
