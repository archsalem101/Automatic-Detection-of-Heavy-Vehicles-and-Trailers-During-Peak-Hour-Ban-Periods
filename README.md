# Automatic Detection of Heavy Vehicles and Trailers During Peak‑Hour Ban Periods

This repository contains a training exercise that uses computer vision to **automatically detect heavy vehicles (trucks, buses, trailers)** on highways and major urban corridors during peak‑hour ban periods. The project is framed as a small **urban planning and traffic management case study**, showing how AI can support city authorities in reducing congestion and improving traffic flow during rush hours.

---

## 1. Urban Planning Context

Many cities apply **time‑based restrictions** on heavy vehicles (e.g., morning and evening peak‑hour bans) to:

- Reduce congestion on key corridors.
- Improve travel times for commuters and public transport.
- Enhance safety and reduce conflicts between freight and passenger traffic.

However, these policies are often **hard to monitor** in real time. Manual checks by enforcement teams or occasional camera reviews are not enough to understand:

- How often heavy‑vehicle bans are actually violated.
- On which **roads and time windows** violations are concentrated.
- Whether the restrictions are really helping to **ease congestion**.

This project demonstrates how an **object detection model** can be a building block in a **smart urban traffic management system**, by automatically detecting heavy vehicles and trailers in peak‑hour traffic images.

---

## 2. Problem Statement

The core problem addressed is:

> How can we automatically detect heavy vehicles and trailers on urban highways and major roads during peak‑hour ban periods, to support monitoring, enforcement, and better urban traffic planning?

The exercise focuses on:

- Visual detection of heavy vehicles and trailers from camera images.
- Situations that mimic **rush hour** and high‑demand conditions.
- A pipeline that could, in a real deployment, feed data to planners and operators.

---

## 3. Objectives

Urban‑planning oriented objectives:

- Build a focused dataset of **traffic scenes from urban highways and main arterial roads** with mixed traffic.
- Annotate and train a model to detect:
  - `car` (background / comparison class)
  - `truck`
  - `bus`
  - `trailer` (or `truck_with_trailer` / `car_trailer`)
- Use the trained model to **estimate where and when heavy vehicles appear** in traffic images captured during typical peak hours.
- Discuss how such a system could:
  - Support **enforcement** of heavy‑vehicle bans.
  - Provide **evidence and statistics** for urban planners (e.g., corridors with frequent violations, before/after analysis of a policy).

---

## 4. Dataset Description

### 4.1 Urban Traffic Focus

The dataset is designed to reflect **urban mobility conditions**, not just generic driving scenes. Images are chosen to:

- Show **multi‑lane highways, ring roads, and major arterials** that connect or cross urban areas.
- Include **mixed traffic**: private cars, trucks, buses, and occasionally trailers.
- Represent **busy/peak‑hour situations**, with moderate to dense traffic that could lead to congestion or jams.

Sources can include:

- Open, license‑friendly stock images (e.g., highway traffic, rush hour, traffic jams).
- Small open datasets that provide traffic or surveillance views on major roads.

All sources should be documented in a `data/dataset_info.md` file with links and licensing notes.

### 4.2 Classes

The following object classes are used, chosen for their relevance to urban planning and freight policy:

- `car` – private passenger vehicles.
- `truck` – heavy goods vehicles / lorries.
- `bus` – public transport and coach buses.
- `trailer` – articulated trucks or vehicles with trailers  
  (in some setups this class may be named `truck_with_trailer` or `car_trailer`).

Optionally, classes can be aggregated into:

- `heavy_vehicle` (for all trucks and trailers)
- `public_transport` (for buses)

depending on the analysis needs.

### 4.3 Dataset Size and Splits

The dataset is intentionally **modest** in size, suitable for a course assignment:

- Total images: ~150–300  
- Train: ~70%  
- Validation: ~20%  
- Test: ~10%  

Each split contains a mix of:

- Different viewpoints (side, slight aerial, overpass).
- Varying traffic densities (free flow to near‑jam).
- Multiple appearances of heavy vehicles when possible.

---

## 5. Annotation and Preprocessing

### 5.1 Annotation

Images are annotated with **bounding boxes** for each visible vehicle:

- Draw a bounding box around each `car`, `truck`, `bus`, and `trailer`.
- Label consistently according to the chosen class list.
- Export to a detection‑friendly format (e.g., YOLO, COCO, or Pascal VOC).

Consistent labeling is important so that planners can trust the counts and class breakdowns in later analysis.

### 5.2 Preprocessing and Augmentation

To improve robustness to real‑world conditions (different cameras, weather, and lighting):

- **Resize** all images to a fixed resolution (e.g., 640×640).
- **Normalize** pixel values according to the model framework.
- Apply basic **augmentations**:
  - Horizontal flips (for different flow directions).
  - Brightness/contrast changes (day / evening / overcast).
  - Small rotations/crops (camera mounting differences).

These steps help simulate the variability in actual **urban surveillance cameras**.

---

## 6. Model Training

The project trains an **object detection model** (e.g., a YOLO variant) using the annotated dataset.

Typical configuration:

- Task: Object Detection
- Classes: `car`, `truck`, `bus`, `trailer`
- Input size: 416×416 or 640×640
- Epochs: ~50–100 (depending on convergence)
- Metrics:
  - mAP (mean Average Precision)
  - Precision and recall per class
  - Qualitative inspection of predictions on **peak‑like traffic scenes**

Training can be done on a cloud training platform or locally; the exact setup should be documented in `notebooks/` or a training script if included.

---

## 7. Urban Planning Use Case & Case Study Angle

This project can be described as a **prototype module** in a broader **Urban Traffic Management** or **Intelligent Transport System**. In a realistic deployment, the workflow might be:

1. **Cameras** on key urban corridors (ring roads, radial arterials, highway segments connecting to the city) capture frames during the day.
2. During defined **peak‑hour ban periods** (e.g., 7:00–9:00 and 16:30–18:30), the detection model runs on incoming frames.
3. For each frame or time window, the system counts:
   - Number of `truck`, `bus`, `trailer` detections.
   - Spatial distribution (which camera / corridor).
4. The results feed into:
   - A **dashboard** for operators (live or near‑real time alerts for violations).
   - **Daily/weekly statistics** for urban planners and policy makers, such as:
     - Corridors with the highest number of heavy vehicles during restricted hours.
     - Trends before and after the introduction of a new ban.
     - Evidence to adjust restriction times or designate alternative freight routes.

In your report, you can treat this as a **small case study** showing how:

- AI helps bridge the gap between **policy on paper** and **actual behavior on the network**.
- Planners and traffic engineers can base decisions on **measured data** about heavy‑vehicle presence during peak periods.
- Future work could integrate travel‑time, congestion, and safety metrics for a more complete planning tool.

---

## 8. Repository Structure

Example structure (adapt as needed):

```text
.
├── data/
│   ├── images/             # Traffic images (urban highways & major roads)
│   ├── labels/             # Bounding box annotations
│   └── dataset_info.md     # Sources, licenses, class definitions, split info
├── notebooks/
│   └── training.ipynb      # Optional: training / evaluation notebook
├── models/
│   └── best_model.*        # Exported trained model (if shared)
├── src/
│   ├── prepare_data.py     # (Optional) data prep scripts
│   └── infer.py            # (Optional) inference script on new images
├── README.md               # This file
└── requirements.txt        # Dependencies (if local code is included)
