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

| Class ID | Name  | Description | Label rule |
|----------|-------|-------------|------------|
| 0        | bus   | Single- or double-deck passenger bus, minibus ≥ 5m | Tight bounding box around full visible body; include mirrors |
| 1        | car   | Passenger car, SUV, van < 3.5 t GVW | Tight bounding box; exclude shadows |
| 2        | truck | HGV, lorry, rigid or articulated; any vehicle > 3.5 t GVW | Include cab + trailer if attached; one box per unit |

**Exclusions:** motorcycles, cyclists, pedestrians, static objects. Partially visible vehicles (< 30 % visible) are skipped.

---

## Dataset

| Property | Value |
|----------|-------|
| Host | GitHub (this repo) |
| Folder | [`/images dataset/`](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) |
| Original source | Roboflow — Smart City Cars Detection v1 |
| Total images | 360 |
| Split | 80 % train / 20 % validation |
| Format | YOLOv8 (normalised XYWH) |
| License | CC BY 4.0 |
| **Dataset link** | [View dataset on GitHub](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) |

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
2. Set `WEIGHTS_PATH` in **Cell 2** to your `best.pt` — OR download pre-trained weights from the [GitHub Release](../../releases) and upload to Colab.
3. Run all cells. The notebook clones the repo for validation images automatically.
4. Outputs: 10 validation prediction images, 5 new-image predictions, SAM notes, `evidence_package.zip`.

> **If GPU is unavailable:** set `VERIFY_ONLY = True` in `01_Training.ipynb` (5 epochs, ~3–5 min on CPU), then load pre-trained weights in `02_Inference.ipynb` for full inference. Document this clearly in the reproducibility log.

---

## Results Summary

> ⚠️ **Placeholder — fill in after your training run completes.**

| Metric | Value |
|--------|-------|
| Precision | 0.6329 |
| Recall | 0.5987 |
| mAP@0.5 | 0.6011 |
| mAP@0.5:0.95 |  0.4486 |

**Key takeaways (draft — update after training):**
1. `truck` class is expected to be hardest due to heavy occlusion and scale variation.
2. `car` likely dominates training distribution — watch for class imbalance.
3. Small model (YOLOv8n) is chosen to stay within Colab free-tier runtime limits.

Training curves and prediction samples are in [`/results/`](./results/).

---

## Reproducibility Checklist

- [ ] Dataset: GitHub repo `/images dataset/` — [link](https://github.com/archsalem101/Automatic-Detection-of-Heavy-Vehicles-and-Trailers-During-Peak-Hour-Ban-Periods/tree/main/images%20dataset) (originally from Roboflow Smart City Cars Detection v1, CC BY 4.0)
- [ ] Model variant: `yolov8n.pt` (nano)
- [ ] Epochs: 50 (verification mode: 5)
- [ ] Batch size: 16
- [ ] Image size: 640 × 640
- [ ] Ultralytics version: `ultralytics==8.3.x` (see pip freeze snippet — `01_Training.ipynb` Cell 1)
- [ ] Framework: PyTorch (installed via ultralytics)
- [ ] Platform: Google Colab (Python 3.10, CUDA 12.x)
- [ ] Trained weights: [GitHub Release](../../releases) → `best.pt`
- [ ] Notebooks: `01_Training.ipynb` → `02_Inference.ipynb` (run in order)

---

## Reproducibility Proof

> ⚠️ **Fill in after your successful run.**

| Field | Value |
|-------|-------|
| Last successful run | _DD/MM/YYYY HH:MM UTC_ |
| Runtime used | _T4 GPU / CPU_ |
| Expected runtime (50 epochs) | _~25–35 min on T4 GPU_ |
| Expected runtime (5-epoch verify) | _~3–5 min on T4 GPU_ |

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
