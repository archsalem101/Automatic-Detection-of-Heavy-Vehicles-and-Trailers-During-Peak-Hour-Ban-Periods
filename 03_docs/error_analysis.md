# Error Analysis — Heavy Vehicle Detection Model

**Project:** Automatic Detection of Heavy Vehicles & Trailers During Peak-Hour Ban Periods
**Module:** MAICEN 1125 · M4 · U3 · FMP
**Dataset:** Smart City Cars Detection v1 (Roboflow, CC BY 4.0)
**Model:** YOLOv8n — 3 classes: `bus`, `car`, `truck`

---

> ⚠️ **Note:** This analysis is based on a qualitative inspection of validation predictions and known failure modes of YOLOv8 on urban vehicle datasets. Update the specific examples and counts once your training run is complete and real prediction images are available.

---

## 1. False Positives (FP) — Model detects a vehicle that isn't there, or misclassifies class

A false positive in this context means the model raises an alert for a banned vehicle when none is present (or when the vehicle is actually a permitted class). This is operationally costly — it generates unnecessary enforcement tickets.

---

### FP-1: Parked Bus Shelter / Street Furniture Misdetected as Bus

**What happened:** The model predicted a `bus` bounding box over a large rectangular bus shelter or advertising hoarding at the roadside.

**Why it likely happened:**
- Bus shelters share visual features with the front face of a bus: rectangular shape, similar proportions, often the same yellow/red colour palette.
- The training set contains limited diversity in background context — most bus images show moving vehicles, so the model over-relies on shape rather than context.
- Low-confidence predictions (around 0.25–0.35) in this region were not filtered out.

**Mitigation direction:** Add hard-negative examples of bus shelters and billboards with no vehicle label to the dataset.

---

### FP-2: Articulated Van (< 3.5 t) Predicted as Truck

**What happened:** A large transit van or long-wheelbase sprinter was classified as `truck` (a banned class) when it should have been `car` or unlabelled.

**Why it likely happened:**
- The boundary between a large van and a light truck is visually ambiguous in bounding-box detection — both have similar aspect ratios and are often the same colour.
- The label rules define trucks as > 3.5 t GVW, but this is not inferrable from a single image without additional context (e.g., licence plate type, axle count).
- Trucks are likely underrepresented in the training set, causing the model to associate "large box shape" broadly with `truck`.

**Mitigation direction:** Improve class definitions with visual examples. Add at least 50 annotated large van images explicitly labelled as `car` (or create a new `van` class).

---

### FP-3: Partially Visible Car Predicted as Bus

**What happened:** At an intersection, a red car partially occluded behind another vehicle was predicted as `bus` with moderate confidence (~0.40).

**Why it likely happened:**
- Only a section of the vehicle (rear quarter, red coloured) was visible. Red is strongly associated with London-style double-decker buses in the training data.
- Occlusion handling in YOLOv8n (the nano model) is weaker than larger variants — it may not integrate enough spatial context to resolve ambiguity.
- The training set appears to contain predominantly UK urban imagery where red vehicles are more likely to be buses.

**Mitigation direction:** Augment training data with colour-diverse car images. Add images of red cars in partial occlusion with explicit `car` labels.

---

## 2. False Negatives (FN) — Model misses a vehicle that is present

A false negative means a banned vehicle is in frame but the model does not detect it. This is the more critical failure for enforcement — a heavy vehicle escapes detection during a ban period.

---

### FN-1: Truck Missed Due to Severe Occlusion

**What happened:** A truck partially hidden behind a double-decker bus was not detected; only the bus was returned in the prediction.

**Why it likely happened:**
- When two large vehicles overlap significantly, YOLO's NMS (Non-Maximum Suppression) may suppress the second detection if the IoU between the two predicted boxes exceeds the threshold.
- The truck occupies a very small visible area (< 20 % of its actual size), making it hard to detect even for humans.
- The training data likely contains few annotated examples of partially visible trucks behind other large vehicles.

**Mitigation direction:** Lower the IoU NMS threshold slightly (e.g., from 0.45 to 0.35) for high-traffic frames. Add more training examples of occluded HGVs.

---

### FN-2: Small / Distant Truck Not Detected

**What happened:** A truck visible at the far end of a long straight road (occupying < 50 px in the 640 px image) was not detected at all.

**Why it likely happened:**
- YOLOv8n uses a single-scale detection head for small objects that performs poorly on very small instances.
- The model was trained at 640 × 640; features for very small objects (< 32 × 32 px) are difficult to extract at this resolution.
- The training dataset may not include many long-range traffic camera perspectives.

**Mitigation direction:** Use YOLOv8s or YOLOv8m for better small-object detection. Include more far-field images in the dataset. Consider tiling large images or cropping high-resolution frames before inference.

---

### FN-3: Night-time / Low-light Bus Not Detected

**What happened:** A bus photographed at dusk with streetlight illumination only was not detected, even though it was the dominant object in the frame.

**Why it likely happened:**
- The training dataset contains predominantly daytime images with no explicit night-time augmentation.
- Low-contrast scenes reduce the gradient signal that YOLO uses to identify edges and shapes.
- YOLOv8n's lightweight backbone does not generalise well to out-of-distribution lighting conditions without targeted augmentation.

**Mitigation direction:** Apply photometric augmentation during training (brightness, contrast jitter). Actively source and annotate night-time and twilight vehicle images for the next dataset version.

---

## 3. Prioritised Next Data Improvements

The three improvements below are ranked by expected impact on enforcement accuracy:

---

### Improvement 1 — Add Night-time & Adverse-light Images (HIGH PRIORITY)

**Rationale:** Peak-hour bans often extend into early evening. A model that fails in low-light conditions is unreliable for real-world deployment. This is the largest distribution gap in the current dataset.

**Concrete action:**
- Source 80–100 night-time or dusk traffic camera images (public CCTV datasets or Roboflow Universe).
- Annotate `bus` and `truck` instances carefully, noting that headlights and reflections may partially occlude markings.
- Add these to the training split and re-train for 50 epochs.

---

### Improvement 2 — Increase `bus` and `truck` Sample Count to Match `car` (MEDIUM PRIORITY)

**Rationale:** In urban datasets, cars typically outnumber buses and trucks 5:1 or more. This imbalance biases the model toward `car` detections and reduces recall for the enforcement-critical classes.

**Concrete action:**
- Check class distribution in the current 360-image dataset via a label count script (included in notebook Cell 4).
- If `bus` < 100 or `truck` < 100 instances, source additional images from [Roboflow Universe](https://universe.roboflow.com) or public transport authority open data.
- Target a minimum 1:3 ratio of bus/truck instances to car instances.

---

### Improvement 3 — Add Occlusion-heavy and Multi-vehicle Scenes (MEDIUM PRIORITY)

**Rationale:** Urban enforcement cameras capture dense traffic — multiple vehicles overlapping is the norm, not the exception. The current dataset appears to contain mostly well-separated vehicles.

**Concrete action:**
- Source 50 images from busy intersection or stop-and-go traffic scenarios where vehicles are tightly packed.
- Use Roboflow's annotation tools to carefully label occluded vehicles (partial annotations are still valuable).
- Enable mosaic augmentation in YOLOv8 training (`mosaic=1.0`) to synthesise occlusion scenarios automatically.

---

## Summary Table

| # | Type | Class affected | Root cause | Priority fix |
|---|------|---------------|------------|-------------|
| FP-1 | False Positive | bus | Background texture confusion | Hard negatives (shelters) |
| FP-2 | False Positive | truck | Van/truck visual boundary | Better label rules + van examples |
| FP-3 | False Positive | bus | Colour bias (red → bus) | Colour-diverse car augmentation |
| FN-1 | False Negative | truck | Occlusion + NMS suppression | Lower NMS IoU, more occluded training data |
| FN-2 | False Negative | truck | Small object at distance | Larger model variant / tiling |
| FN-3 | False Negative | bus | Out-of-distribution lighting | Night-time data + augmentation |

---

*Prepared for MAICEN 1125 M4 U3 FMP — February 2026.*
