# Governance Checklist

**Project:** Automatic Detection of Heavy Vehicles & Trailers During Peak-Hour Ban Periods
**Module:** MAICEN 1125 · M4 · U3 · FMP Group Assignment
**Model:** YOLOv8n — Classes: `bus`, `car`, `truck`
**Date:** February 2026

---

## 1. Data Provenance

| Question | Answer |
|----------|--------|
| Where did the data come from? | Roboflow Universe — "Smart City Cars Detection" v1, assembled from publicly available urban traffic camera imagery |
| Who collected / annotated it? | Project team (FMP group), annotated via Roboflow annotation tools |
| Is the source documented? | ✅ Yes — dataset URL: https://app.roboflow.com/salem-w0yva/smart-city-cars-detection-thzvu/1 |
| Is the data licence clear? | ✅ Yes — Creative Commons Attribution 4.0 International (CC BY 4.0) |
| Is the dataset version pinned? | ✅ Yes — Version 1 (exported 26 February 2026) |
| Can a third party reproduce the exact dataset? | ✅ Yes — Roboflow public project, downloadable via API with version lock |

**Summary:** The dataset is fully traceable, publicly accessible, and published under a permissive licence that allows reuse with attribution.

---

## 2. PII Handling (Personally Identifiable Information)

| Item | Status | Action required |
|------|--------|-----------------|
| Faces visible in training images | ⚠️ Possible | Blur or crop all identifiable faces before any public release of images |
| Licence plates visible in images | ⚠️ Possible | Anonymise all licence plates in images before public release |
| Driver / passenger identity data | ✅ Not collected | Model outputs class label + bounding box only; no personal data stored |
| GPS / location metadata in images | ✅ Removed | Roboflow export strips EXIF metadata including GPS coordinates by default |
| Tracking of individual vehicles across frames | ✅ Not performed | Each image is processed independently; no persistent vehicle IDs |
| Data subject consent | ✅ Not required for research | Images are of vehicles in public spaces; no reasonable expectation of privacy applies to vehicles on public roads under UK law |

**GDPR / UK DPA 2018 note:** This is a research prototype. Any deployment in a live enforcement context requires a documented legal basis under Article 6 GDPR (e.g., legitimate interest or legal obligation), a Data Protection Impact Assessment (DPIA), and licence plate anonymisation for non-evidential outputs.

---

## 3. Risk Statement

### False Negative Risk (missed detection) — **HIGH**
A heavy vehicle (bus or truck) operating during a ban period is not detected by the model → no enforcement alert is raised.

**Why this is the primary risk:** The stated purpose of the system is to catch ban violations. A missed detection is a direct failure of that objective. Heavy vehicles cause disproportionate road wear and congestion; systematic under-detection undermines the entire use case.

**Likelihood:** Medium-High. The model is trained on only 360 images. Night-time, occluded, and distant vehicles are known blind spots.

**Impact:** High — ban period violations go unpunished; road safety and congestion goals not met.

**Mitigation:** Prioritise recall over precision in model tuning. Lower confidence threshold for `bus` and `truck`. Add night-time and occlusion-heavy training data. Mandate human review of all low-confidence detections.

---

### False Positive Risk (wrong detection) — **MEDIUM**
A permitted vehicle (e.g., a car or large van) or a non-vehicle object is classified as a banned heavy vehicle → an unwarranted enforcement alert is triggered.

**Why this matters:** An unjust alert, if acted on without human review, could result in a wrongful penalty notice — a legal and reputational risk for the deploying authority.

**Likelihood:** Medium. Visual ambiguity between large vans and light trucks is a known model weakness.

**Impact:** Medium — erroneous alerts cause inconvenience and erode public trust, but can be mitigated by human review.

**Mitigation:** Always require human-in-the-loop review before issuing any penalty. Hard-negative examples (large vans explicitly labelled as `car`) can improve precision.

---

### Other Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| Model deployed outside intended camera angles or lighting | Medium | Document scope; re-test before deploying on new camera hardware |
| Dataset colour/brand bias causing systematic misses of minority vehicle types | Low–Medium | Audit class distribution; add underrepresented vehicle types |
| Model presented as definitive without qualification | Medium | All outputs must be clearly labelled as "AI-assisted, requires human review" |

---

## 4. Human-in-the-Loop

**Requirement:** This model must **not** be the sole decision-maker for any enforcement action.

| Stage | Human role |
|-------|-----------|
| Annotation | Human annotators labelled all bounding boxes; reviewed in Roboflow |
| Threshold tuning | Human decision on confidence threshold (currently 0.25) |
| Alert review | A qualified enforcement officer reviews any model-flagged frame before issuing a penalty |
| Edge case escalation | Low-confidence detections (< 0.40) are always escalated for manual review |
| Periodic model audit | Model performance should be re-evaluated every 6 months against new data |

**Rationale (from lecture notes — LS4):** Object detection models trained on small datasets have known failure modes in distribution-shifted conditions (lighting, weather, occlusion). A human-in-the-loop requirement is the primary safeguard against systematic enforcement errors.

---

## 5. Licence Statement

### Code & Notebooks
**MIT License** — free to use, modify, and distribute with attribution.

### Dataset
**Creative Commons Attribution 4.0 International (CC BY 4.0)**
Source: Roboflow Smart City Cars Detection v1
You must credit the dataset and link to the licence when reusing.

### Pre-trained Backbone Weights (YOLOv8n)
**GNU AGPL-3.0** (Ultralytics open-source licence)
Commercial deployment without open-sourcing derived work requires an Ultralytics Enterprise Licence.
See: https://github.com/ultralytics/ultralytics/blob/main/LICENSE

### Summary Table

| Component | Licence | Commercial use allowed? |
|-----------|---------|------------------------|
| Notebooks / code | MIT | ✅ Yes |
| Dataset images + labels | CC BY 4.0 | ✅ Yes (with attribution) |
| YOLOv8 pre-trained weights | AGPL-3.0 | ⚠️ Open-source only; enterprise licence needed for closed commercial use |
| Fine-tuned weights (this project) | AGPL-3.0 (inherited) | ⚠️ Same as above |

---

## Sign-off

- [x] Data provenance documented and dataset version pinned
- [x] PII risks identified; anonymisation required before public image release
- [x] Risk statement written; false negative identified as primary risk
- [x] Human-in-the-loop requirement defined for all enforcement decisions
- [x] All licences documented and compatible with intended use
- [ ] DPIA completed *(required before live enforcement deployment — out of scope for this research prototype)*
- [ ] Licence plate anonymisation implemented *(required before public image release)*

---

*Prepared for MAICEN 1125 M4 U3 FMP — February 2026.*
