# 231FA04023-MLOPS-FEAST-SKILLGAP

## Student Details
- **Name:** RAJESH
- **Register Number:** `231FA04023`
- **Section:** `15`

> Replace the placeholders with your actual details before uploading to GitHub.

## Problem Statement
The curriculum-industry skill-gap problem identifies differences between skills covered by an academic curriculum and skills demanded by industry. This project converts the 100-row skill-gap dataset from the previous activity into a simple Feast-based feature store and uses Feast features for ML training and online prediction.

## Dataset
**Number of skills:** 100

**Columns:**
- `Skill_Name`
- `Curriculum_Coverage`
- `Industry_Demand`
- `Job_Frequency`
- `Expert_Importance`
- `Gap_Status`

**Target:** `Gap_Status`, encoded as `Gap → 1` and `Aligned → 0`.

The entries were created in the previous curriculum-industry skill-gap activity. Feast-compatible IDs, timestamps and engineered features are added without replacing the original records.

## Feature Engineering

| Feature | Meaning |
|---|---|
| `Curriculum_Coverage` | Original curriculum coverage score |
| `Industry_Demand` | Original industry demand score |
| `Job_Frequency` | Original job-frequency score |
| `Expert_Importance` | Original expert-importance score |
| `skill_gap` | `Industry_Demand - Curriculum_Coverage` |
| `demand_ratio` | Industry demand relative to curriculum coverage |
| `demand_minus_job` | Industry demand minus job frequency |
| `importance_gap` | Expert importance minus curriculum coverage |
| `industry_pressure` | Industry demand + job frequency + expert importance |
| `coverage_pressure` | Curriculum coverage + expert importance |
| `high_gap` | Target encoding: 1 = Gap, 0 = Aligned |

Example:
```text
skill_gap = Industry_Demand - Curriculum_Coverage
```

## Feast Architecture

```text
Original Dataset
      ↓
Feature Engineering
      ↓
Parquet Offline Data
      ↓
Feast FeatureView
      ↓
 ┌─────────────────────┐
 ↓                     ↓
Historical Features   Materialization
 ↓                     ↓
Model Training       Online Store
                       ↓
                  Online Retrieval
                       ↓
                    Prediction
```

## Implementation

### Entity
The Feast entity is `skill_id`. It uniquely identifies each skill record (`S001` to `S100`).

### Data Source
The engineered dataset is stored as:
```text
data/skill_gap_features.parquet
```
Feast reads it using a `FileSource`. `event_timestamp` supports point-in-time historical retrieval.

### FeatureView
The FeatureView is named:
```text
skill_gap_features
```
and contains all features listed above.

### Registration
Feast definitions are registered using:
```bash
feast apply
```

### Historical Retrieval
```python
store.get_historical_features(
    entity_df=entity_df,
    features=feature_refs
).to_df()
```

### Model
The notebook compares Logistic Regression, Random Forest, Extra Trees, HistGradientBoosting, SVM and KNN using Stratified 5-Fold Cross-Validation. Random Forest and Extra Trees are hyperparameter-tuned with `GridSearchCV`. The best validated model is evaluated on a separate 25% held-out test set.

### Materialization
```bash
feast materialize-incremental 2026-08-17T23:59:59
```
This copies required feature values from the offline source into the local SQLite online store.

### Online Retrieval
```python
store.get_online_features(
    features=feature_refs,
    entity_rows=[{"skill_id": "S005"}]
).to_dict()
```

## Results

### Historical Feature Output
The notebook displays the output of `get_historical_features()`, containing `skill_id`, the Feast features and `high_gap`.

### Model Accuracy
Copy the exact value printed by the notebook:
```text
HELD-OUT TEST ACCURACY (%): <COPY ACTUAL VALUE FROM COLAB>
```

Do not manually claim 85% unless the validated run actually produces 85% or higher.

### Online Feature Output
The notebook retrieves online features for:
```text
S005
S007
S015
```

### One Final Prediction
Copy one actual result from Colab:
```text
Skill: <COPY FROM COLAB>
Entity: <COPY FROM COLAB>
Actual: <COPY FROM COLAB>
Predicted: <COPY FROM COLAB>
```

# Required Analysis

## 1. What is the entity in your Feast implementation?
The entity is `skill_id`, which uniquely identifies each skill.

## 2. List the features stored in your FeatureView.
`Curriculum_Coverage`, `Industry_Demand`, `Job_Frequency`, `Expert_Importance`, `skill_gap`, `demand_ratio`, `demand_minus_job`, `importance_gap`, `industry_pressure`, `coverage_pressure`, and `high_gap`.

## 3. Explain how one feature was calculated.
`skill_gap = Industry_Demand - Curriculum_Coverage`. A positive value indicates industry demand is greater than curriculum coverage.

## 4. What is the difference between your original dataset and the feature dataset?
The original dataset contains the six original columns. The feature dataset retains the original information and adds `skill_id`, `event_timestamp`, engineered features, and the encoded target needed by Feast and the ML workflow.

## 5. What is the purpose of the offline store?
It stores historical feature data and is used for historical retrieval and creation of training datasets. This project uses Parquet through Feast `FileSource`.

## 6. What is the purpose of the online store?
It stores materialized feature values for fast retrieval during online inference. This project uses SQLite.

## 7. What is the purpose of `feast apply`?
It registers and updates the Feast entity, data source and FeatureView in the Feast registry.

## 8. What does materialization do?
It transfers required feature values from the offline source into the online store so they can be retrieved during prediction.

## 9. What is the advantage of retrieving features through Feast instead of manually calculating them separately during training and prediction?
Feast centralizes feature definitions and lets the same definitions be used for historical training and online inference. This reduces duplicated feature-engineering code and helps prevent training-serving inconsistency.

## 10. State two limitations of your current dataset.
1. **Dataset size:** only 100 skill records, so it may not represent the complete industry skill landscape.
2. **Limited industry evidence:** it does not contain detailed job-posting text, company information, dates, salary information, source URLs or historical trends.

## 11. State two ways your feature store could be improved when more curriculum and industry evidence becomes available.
1. Add curriculum documents, job postings, company requirements, expert surveys and industry reports as additional evidence sources.
2. Add time-varying features such as job-posting frequency, salary demand, emerging-skill trends, curriculum update frequency and skill-demand growth.

## Original Dataset vs Feature Dataset

```text
Original Dataset
       ↓
Feature Engineering
       ↓
Feature Dataset
       ↓
Parquet Offline Data
       ↓
Feast FeatureView
```

## Project Structure

```text
<RegisterNumber>MLOps-Feast-SkillGap/
├── README.md
├── requirements.txt
├── feature_engineering.py
├── feature_store.py
├── feature_store.yaml
├── CLA_1_MLOps_Feast_SkillGap_HighAccuracy.ipynb
└── data/
    └── skill_gap_dataset_100.csv
```

## How to Run
1. Open the supplied `.ipynb` in Google Colab.
2. Upload `skill_gap_dataset_100.csv`.
3. Run cells from top to bottom.
4. Confirm `skill_gap_features` appears after `feast apply`.
5. Run historical retrieval.
6. Run model comparison and tuning.
7. Record the actual held-out accuracy.
8. Run materialization.
9. Run online feature retrieval and final prediction.
10. Copy the actual results into this README.

## GitHub Repository

Required repository name:
```text
231FA04023-MLOps-Feast-SkillGap
```

## Conclusion
This project demonstrates feature engineering, Feast entity and data-source creation, FeatureView registration, historical retrieval, ML training, materialization, online retrieval and prediction using a centralized feature-store workflow.
