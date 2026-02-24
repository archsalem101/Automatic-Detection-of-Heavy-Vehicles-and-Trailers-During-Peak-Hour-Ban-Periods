# Automatic Detection of Heavy Vehicles and Trailers During Peak‑Hour Ban Periods

This repository contains a training exercise that uses computer vision to **automatically detect heavy vehicles (trucks, buses, trailers)** on highways and major roads during peak‑hour ban periods. The goal is to support traffic monitoring and help authorities enforce restrictions on heavy vehicles to **reduce congestion and avoid traffic jams** during rush hours.

---

## 1. Problem Statement

In many cities and highway networks, heavy vehicles are restricted or banned during specific **peak‑hour windows** to ease congestion and improve travel times. Manual monitoring of these rules using CCTV or roadside cameras is time‑consuming and unreliable.

This project explores how an **object detection model** can automatically:

- Detect heavy vehicles (trucks, trailers, buses) on highways and major roads.  
- Operate on images representative of **rush hour / peak traffic**.  
- Provide a foundation for future systems that could trigger alerts when banned vehicles are present during restricted hours.

---

## 2. Objectives

- Build a small but focused dataset of **traffic images** from highways and major roads, with visible heavy vehicles and mixed traffic.
- Annotate each image with bounding boxes and labels for:
  - `car`
  - `truck`
  - `bus`
  - `trailer` (or `truck_with_trailer` / `car_trailer`, depending on labeling scheme)
- Train an object detection model (via Roboflow / YOLO) to recognize these classes.
- Evaluate the model’s performance and discuss how it could be used to **monitor peak‑hour bans** and help avoid traffic jams.

---

## 3. Dataset Description

### 3.1 Source of Images

The images used in this exercise are collected from **open, license‑friendly sources** (e.g., free stock image sites) and/or small public traffic datasets. The focus is on:

- **Highways and major roads** with multiple lanes.  
- Scenes with **mixed traffic**: cars, trucks, buses, and sometimes trailers.  
- Images that resemble **rush‑hour conditions** or dense traffic flow (potential traffic jams).

> Note: If you clone this repository, check the `data/` folder or dataset links in this README to understand the exact image sources and licenses.

### 3.2 Classes

The following object classes are used:

- `car` – standard passenger vehicles.  
- `truck` – heavy trucks and lorries.  
- `bus` – public transport and coach buses.  
- `trailer` – articulated trucks or vehicles with trailers (can also be named `truck_with_trailer` / `car_trailer` depending on labeling).

You can adapt or merge these labels depending on your use case (for example, merging `truck` and `trailer` into a single `heavy_vehicle` class).

### 3.3 Dataset Size and Splits

For this assignment, the dataset is intentionally kept small but balanced:

- Total images: ~150–300  
- Train: ~70%  
- Validation: ~20%  
- Test: ~10%  

Each split contains a mix of scenes and classes, with a focus on having **enough examples of heavy vehicles** in different positions, scales, and lighting conditions.

---

## 4. Annotation and Preprocessing

### 4.1 Annotation

Images are annotated using **bounding boxes** around each vehicle instance. An annotation tool (e.g., Roboflow) is used to:

- Draw a bounding box around each `car`, `truck`, `bus`, and `trailer`.  
- Assign the correct class label.  
- Export the dataset in a detection‑compatible format (e.g., YOLO, COCO, or VOC).

### 4.2 Preprocessing and Augmentation

Before training, the dataset is processed with standard steps:

- **Resize** images to a fixed resolution (e.g., 640×640).  
- **Normalize** pixel values as required by the chosen model.  
- Apply basic **data augmentation** to improve robustness:
  - Random horizontal flip.  
  - Random brightness/contrast changes.  
  - Slight rotation or scaling.  

These augmentations help the model generalize to different perspectives, lighting conditions, and camera setups.

---

## 5. Model Training

This exercise uses an object detection model (e.g., YOLO‑based) trained via an online platform (such as Roboflow) or locally.

Typical training configuration:

- Task: **Object Detection**  
- Classes: `car`, `truck`, `bus`, `trailer`  
- Input size: 416×416 or 640×640  
- Epochs: ~50–100 (depending on dataset size and convergence)  
- Evaluation metrics:
  - mAP (mean Average Precision)  
  - Precision and recall per class  

After training, the model can be:

- Evaluated on the **test set**.  
- Tested on new highway / rush‑hour images to visually inspect its predictions.

---

## 6. Use Case: Monitoring Peak‑Hour Ban Periods

In a real deployment, the trained model could be integrated with:

- **CCTV or roadside cameras** on highways and major roads.  
- A system that knows the **current time and restricted hours**.  

The workflow could be:

1. Capture frames from traffic cameras during peak hours.  
2. Run the detection model to identify `truck`, `bus`, and `trailer`.  
3. If heavy vehicles are detected during a **ban window**, the system could:
   - Log the event.  
   - Trigger an alert to operators.  
   - Provide visual evidence (image with bounding boxes).

This assignment demonstrates the **computer vision component** of such a system: the ability to detect heavy vehicles in traffic scenes automatically.

---

## 7. Repository Structure

Example structure (adjust to match your actual files):

```text
.
├── data/
│   ├── images/          # Raw or processed images
│   ├── labels/          # Annotation files (YOLO/COCO/VOC)
│   └── dataset_info.md  # Notes about sources, licensing, and splits
├── notebooks/
│   └── training.ipynb   # Optional: example training notebook (if used)
├── models/
│   └── best_model.*     # Exported trained model (if shared)
├── README.md            # This file
└── requirements.txt     # Dependencies (if local training/inference is included)
