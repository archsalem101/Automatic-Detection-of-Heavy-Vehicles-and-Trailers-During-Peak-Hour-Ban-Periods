# AECO Governance Checklist

## 1. Data Provenance

\* \*\*Source:\*\* [Where did the images come from? e.g., "Roboflow Universe

Public Dataset" or "Site Photos form Project X"]

\* \*\*Date Collection:\*\* [e.g., "June 2024"]

\* \*\*Owner:\*\* [Who owns the raw images?]

## 2. PII Handling (Privacy)

\* \*\*Faces/License Plates:\*\* [Are they present?]

\* \*\*Protection Strategy:\*\* [e.g., "Faces were blurred using Roboflow

Augmentation" OR "No PII present in dataset"]

## 3. Risk Statement

\* \*\*High-Impact False Negative:\*\* [What happens if we miss it? e.g.,

"Missed crack = Structural Failure"]

\* \*\*High-Impact False Positive:\*\* [What happens if we hallucinate? e.g.,

"False alarm = $50k unnecessary scaffolding"]

## 4. Human-in-the-Loop

\* \*\*Review Process:\*\* [Who makes the final decision? e.g., "This model is

for screening only. A Senior Engineer verifies all 'Crack' detections."]

## 5. License

\* \*\*Type:\*\* [MIT / Apache / Internal Use Only]