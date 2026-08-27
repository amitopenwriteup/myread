Here's the assignment content mapped into your two session outlines:

## Session 1 (90 min)

**1. Overview** *(5 min)*
Intro to the MLOps pipeline and toolchain — Pandera, MLflow, Optuna, Evidently, SHAP — and how they chain together (validate → train/track → tune/register → monitor → explain).

**2. Business Objective and Dataset Overview** *(20 min)*
- Business context: 10,000+ shop-floor machines, downtime cost of ₹8–15 lakh/hour
- Failure class table (0–4: No Failure, TWF, HDF, PWF, OSF)
- The three provided files: `train.csv` (6,993 rows, historical baseline), `current.csv` (1,499 rows, stable production), `stress.csv` (1,499 rows, heavy-load period)

**3. Data Loading, Validation, and EDA** *(35 min)*
- **1.1** Load train/current/stress, print shapes, preview `train.head()`
- **1.2** Define the Pandera `DataFrameSchema` (Type, Air/Process temperature ranges, Rotational speed, Torque, Tool wear, Failure_Type); validate train/current strictly, validate stress with `lazy=True`
- **1.3** EDA: class distribution of `Failure_Type`, Torque/Tool wear distributions by failure class, `Type` (L/M/H) distribution
- **1.4** Feature engineering: `Power_W` and `Temp_diff` derived features, grouped means by failure type

**4. Model Setup and Experiment Tracking** *(20 min)*
- **2.1** Feature list, encode `Type`, stratified 80/20 split, SMOTE on training split only (+ the "why SMOTE only on train" explanation)
- **2.2** (start) Train the 4 candidate models — Logistic Regression, Random Forest, XGBoost, LightGBM — and set up MLflow experiment `PredMaint_ModelSelection` for logging (params/metrics/per-class F1)

**5. Q&A** *(5 min)*

**6. Conclusion** *(5 min)*
Recap Section 1's outputs (validated data, engineered features, tracking setup) and preview what Session 2 covers.

---

## Session 2 (95 min)

**1. Overview** *(5 min)*
Recap where Session 1 left off (4 models logged); outline today's plan: selection → tuning → drift → explainability → wrap-up.

**2. Model Comparison and Selection** *(20 min)*
Finish **2.2**: comparison table across the 4 models (macro F1, weighted F1, accuracy, per-class F1), pick the best model by macro F1.

**3. Model Tuning and Registration** *(20 min)*
**2.3**: Optuna study (30 trials, `TPESampler(seed=42)`) tuning XGBoost, optimizing macro F1; register best model as `PredMaint_XGBoost` in MLflow Registry and promote to the `production` alias; report improvement over baseline XGBoost.

**4. Drift Detection and Monitoring** *(20 min)*
- **3.1** Evidently `DataDriftPreset` on `current.csv` vs `train.csv` → `drift_current.html`
- **3.2** Evidently on `stress.csv` with `ColumnDriftMetric` per feature → `drift_stress.html`, feature/Wasserstein/mean/delta table
- **3.3** Retraining decision: which features drifted, link to SHAP-driven failure class impact, retrain recommendation

**5. Explainability and Insights** *(10 min)*
- **4.1** SHAP `TreeExplainer` per failure class (TWF, HDF, PWF, OSF), `shap_per_class.png`, top driver per class
- **4.2** Engineering insight: `Power_W` vs raw Torque/Rotational speed for PWF, `Temp_diff` ranking for HDF, physical mechanisms

**6. Guidelines and Expectations** *(10 min)*
Submission requirements (executed `.ipynb`, keep section structure intact), grading breakdown by marks (Section 1: 15, Section 2: 15, Section 3: 10, Section 4: 5, Section 5: 5), and expectations for the write-up sections (e.g., TWF's F1=0.0 → data scarcity is full-credit insight even without fixing it).

**7. Q&A** *(5 min)*

**8. Conclusion** *(5 min)*
**Section 5.1** key findings: winning model + macro F1, why accuracy is misleading here, the TWF root cause, drift implications for maintenance scheduling, and the actionable SHAP-based recommendation (Condition → Risked failure class → Action).
