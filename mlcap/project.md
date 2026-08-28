# MLOps Assignment — Trainer Guide
## Predictive Maintenance Classification with Local MLOps

This trainer guide merges the **assignment brief**, the **stage-wise tasks**, and the
**reference solution logic** into one flow, with **interactive checkpoints** inserted
between sections. Use the checkpoints as live discussion prompts (cold-call, think-pair-share,
or a quick poll) before revealing the model answer to students.

> How to run a checkpoint: ask the question, give students 60–90 seconds to answer on paper
> or in chat, take 2–3 verbal answers, *then* reveal the "Model Answer" box.

---

## 1. Problem Statement

A heavy-equipment manufacturer runs 10,000+ machines on the shop floor. Each machine
streams sensor readings (type, air temperature, process temperature, rotational speed,
torque, tool wear). When a machine fails, production halts — at a cost of roughly
**Rs. 8–15 lakh per hour of downtime**. The current process is reactive: technicians
respond *after* a failure happens.

The assignment asks students to replace that reactive process with a **local MLOps loop**:

`validate → train → track → tune → monitor → explain → decide`

Concretely, the workflow must:
- validate incoming sensor data before it enters the pipeline
- compare and track candidate models
- tune and register the best model
- monitor incoming batches for drift
- explain model predictions
- recommend retraining when monitoring shows risk

### ✅ Checkpoint 1 — Framing the problem
**Ask students:** "Why does the case study insist on a *local* MLOps loop instead of asking
you to deploy a live API? What is this assignment actually testing?"

<details>
<summary>Model Answer</summary>

The assignment is testing whether a student understands the **full lifecycle of a model in
production** — not just training accuracy. A live API would add engineering overhead
(serving, auth, latency) that is irrelevant to the actual learning goal, which is:
*can you validate data, track experiments, detect when a model has gone stale, and justify
a retraining decision with evidence?* Local MLflow + Evidently + SHAP lets every one of
those skills be graded without needing infrastructure.
</details>

---

## 2. Assignment Goals

By the end of the assignment, a student should be able to:

- validate tabular data with **Pandera**
- handle class imbalance correctly (with **SMOTE**)
- compare multiple models fairly using **MLflow**
- tune the strongest model with **Optuna**
- detect drift with **Evidently**
- interpret a multiclass tree model with **SHAP**
- connect all of the above technical outputs to an engineering decision

This is not "build the best classifier." It is "prove you understand the loop."

### ✅ Checkpoint 2 — Tool mapping
**Ask students:** "Match each tool to the pipeline stage it belongs to: Pandera, MLflow,
Optuna, Evidently, SHAP."

<details>
<summary>Model Answer</summary>

| Tool | Stage | One-line role |
|---|---|---|
| Pandera | Validate | Checks schema/range validity of incoming data |
| MLflow | Track / Register | Logs runs, metrics, and promotes the winning model |
| Optuna | Tune | Searches hyperparameters to maximize a chosen metric |
| Evidently | Monitor | Detects distribution shift (drift) after deployment |
| SHAP | Explain | Attributes each prediction to feature contributions |

</details>

---

## 3. Files Provided & What the Data Represents

Provided to students (already built, not to be re-created):

- `train.csv` — 6,993 historical labelled sensor readings (the baseline)
- `current.csv` — 1,499 readings from a *stable* post-deployment batch
- `stress.csv` — 1,499 readings from a *heavy-load* post-deployment batch
- `requirements.txt` — pinned environment
- Starter notebook with full section scaffolding + TODOs
- Grading rubric (CSV)

Each dataset plays a distinct role:

| Dataset | Role | Expected story |
|---|---|---|
| `train.csv` | Model training + validation baseline | Ground truth distribution |
| `current.csv` | Simulated normal production batch | Mostly stable vs. train |
| `stress.csv` | Simulated heavy-load production batch | **Valid but distributionally shifted** |

The key subtlety: `stress.csv` is built to be **operationally valid** — it will pass
Pandera's schema checks — but it is **statistically drifted**, which only Evidently will
surface. That gap between "valid" and "unchanged" is one of the core lessons.

### ✅ Checkpoint 3 — Why three CSVs, not one?
**Ask students:** "Why does this assignment give you three separate files instead of one
big dataset you split yourself? What would be lost if `current.csv` and `stress.csv` were
just random samples cut from `train.csv`?"

<details>
<summary>Model Answer</summary>

The three files simulate **three different points in time / operating conditions**:
past (training), present-normal (current), and present-stressed (stress). If `current`
and `stress` were random cuts of `train`, they would have the *same underlying
distribution* as the training data by construction — there would be nothing to detect.
Drift detection only makes sense when the "new" data can plausibly differ from the
"reference" data. Separating the files also mirrors real deployment: you train once on
historical data, then repeatedly receive new batches you did *not* control the sampling of.
</details>

### ✅ Checkpoint 4 — Validity vs. drift
**Ask students:** "If `stress.csv` passes every Pandera check, does that mean the model
will perform fine on it? Why or why not?"

<details>
<summary>Model Answer</summary>

No. **Validity ≠ stationarity.** Pandera only checks that each value is *individually*
plausible (e.g., torque is between 3–80 Nm). It says nothing about whether the
*distribution* of values has shifted from what the model was trained on. A batch can be
100% schema-valid and still be operationally different enough (e.g., systematically
higher torque and tool wear) that the model's learned decision boundaries no longer fit.
That's exactly why the pipeline needs a second, separate check — Evidently — for
distributional drift.
</details>

---

## 4. Stage 1 — Data Loading, Schema Validation & EDA *(15 marks)*

**Goal:** Prove the data is safe to use before modeling it.

**Tasks:**
- load all three CSVs, print shapes, preview `train`
- fix integer dtypes *before* validating (`Rotational speed`, `Tool wear`, `Failure_Type`)
- define a Pandera `DataFrameSchema` for the 7 raw columns with correct types/ranges
- validate `train` and `current` cleanly; validate `stress` with `lazy=True` and inspect failures
- plot class distribution, and `Torque`/`Tool wear` by failure class
- engineer `Power_W = Torque × RPM × 2π/60` and `Temp_diff = Process temp − Air temp`
- print grouped means of the engineered features by `Failure_Type`

**Common mistakes:** validating before fixing dtypes; forgetting to validate `stress`;
computing engineered features on only one dataframe.

### ✅ Checkpoint 5 — What is Pandera, really?
**Ask students:** "In one sentence, what is Pandera? What role does it play that a simple
`assert` statement doesn't?"

<details>
<summary>Model Answer</summary>

**Pandera is a data validation library for pandas DataFrames** — it lets you declare a
schema (expected column types, ranges, allowed categories) once, and then check any
DataFrame against it in a structured, reusable, and reportable way. Compared to scattered
`assert` statements, Pandera gives you: (1) a single declarative source of truth for what
"valid data" means, (2) the ability to check *all* violations at once with `lazy=True`
instead of stopping at the first failure, and (3) a structured failure report (which rows,
which columns, which check) instead of a crashed script.
</details>

### ✅ Checkpoint 6 — Reading the schema table
**Ask students:** "Why is `Rotational speed` cast to `int64` before validation, but
`Torque` stays a `float`? What would happen if you validated before calling `fix_dtypes`?"

<details>
<summary>Model Answer</summary>

`Rotational speed` and `Tool wear` are physically whole-number quantities (rpm, minutes),
and the schema declares them as `int64`. If the CSV loads them as `float64` (common when a
column has any NaN or gets inferred loosely), a Pandera schema expecting `int64` will
reject the *entire column on type alone*, even if every value is numerically in range.
`Torque` is genuinely continuous, so it correctly stays `float`. Skipping `fix_dtypes` before
validation is Common Mistake #1 in the guide — it produces false-positive schema failures
that have nothing to do with the actual data quality.
</details>

---

## 5. Stage 2 — Experiment Tracking & Model Selection *(15 marks)*

**Goal:** Compare models fairly under class imbalance, then tune and register the winner.

**Tasks:**
- encode `Type` (fit on `train`, transform `current`/`stress` with the *same* encoder)
- stratified 80/20 split (`random_state=42`)
- apply **SMOTE** to the training split *only*, with `k_neighbors=3`
- train & log 4 models to MLflow (Logistic Regression, Random Forest, XGBoost, LightGBM)
- log per-run: model name, `macro_f1`, `weighted_f1`, `accuracy`, and per-class F1
- pick the winner by **macro F1**, not accuracy
- run a 30-trial Optuna study (TPE, seed=42) tuning XGBoost for `macro_f1`
- register the tuned model in the MLflow Model Registry and promote to `production`

### ✅ Checkpoint 7 — What is SMOTE and why here?
**Ask students:** "The training set is ~96.7% 'No Failure'. Explain what SMOTE does and
why it's necessary here. Then explain why it's applied *after* the train/validation split,
not before."

<details>
<summary>Model Answer</summary>

**SMOTE (Synthetic Minority Over-sampling Technique)** creates *synthetic* examples of
minority classes by interpolating between real minority samples and their nearest
neighbors, rather than just duplicating existing rows. It's necessary here because a
model trained on 96.7% "No Failure" data can hit 96.7% accuracy by never predicting a
failure at all — SMOTE forces the model to actually learn failure patterns during training.

It must be applied **after** the split (fit on `X_train`/`y_train` only) because applying
it *before* splitting would let synthetic points derived from a validation-set sample leak
into training, or vice versa — inflating validation performance in a way that doesn't
reflect real-world deployment. This is Common Mistake #2 in the rubric.
</details>

### ✅ Checkpoint 8 — Why `k_neighbors=3`?
**Ask students:** "SMOTE's default `k_neighbors` is 5. Why does this assignment explicitly
set it to 3?"

<details>
<summary>Model Answer</summary>

SMOTE needs at least `k_neighbors` real samples of a class to interpolate between. The
rarest class here (**TWF**) has only ~30 real samples *before* the split, and fewer still
in the 80% training portion. With the default `k=5`, SMOTE can fail outright or produce
poor synthetic points for very small classes. Lowering `k_neighbors=3` is the minimum
practical adjustment that lets SMOTE still function on the smallest class — it doesn't
solve data scarcity, it just keeps the algorithm from breaking.
</details>

### ✅ Checkpoint 9 — Why macro F1 over accuracy?
**Ask students:** "All four models are expected to score above 98% accuracy. Why is that
number almost meaningless here, and what should you report instead?"

<details>
<summary>Model Answer</summary>

Accuracy is dominated by the majority class. Since "No Failure" is ~96.7% of the data, a
model that predicts "No Failure" for *every single row* still scores ~96.7% accuracy while
being operationally useless — it would never flag a real failure. **Macro F1** averages the
F1 score across all five classes *with equal weight*, so a model can't hide poor
performance on rare failure types behind a high accuracy number. In a predictive
maintenance setting, missing a rare failure is exactly the expensive mistake accuracy is
blind to — which is why macro F1 (not accuracy, not even weighted F1) is the selection
metric in the rubric.
</details>

---

## 6. Stage 3 — Drift Detection & Monitoring *(10 marks)*

**Goal:** Show how the deployed model behaves once new data starts arriving.

**Tasks:**
- sanity-check with a basic stat first (e.g., mean `Rotational speed`, `current` vs `stress`)
- run Evidently's `DataDriftPreset` with `train` as reference vs `current` → save `drift_current.html`
- run per-column `ColumnDriftMetric` with `train` as reference vs `stress` → save `drift_stress.html`
- report drift detected / not, per-feature Wasserstein scores, and mean deltas
- make a retraining recommendation using the chain: **drifted feature → affected failure class → decision**

### ✅ Checkpoint 10 — What is "drift," and how is it different from invalid data?
**Ask students:** "Define data drift in your own words. Why did Stage 1 (Pandera) already
tell you `stress.csv` was 'fine,' and Stage 3 (Evidently) is telling you something
different?"

<details>
<summary>Model Answer</summary>

**Data drift** is a change in the statistical distribution of incoming data relative to
the data a model was trained on — the values are still individually plausible, but their
overall pattern (mean, spread, correlations) has shifted. Pandera checks *row-level
validity* (is this torque value physically possible?); Evidently checks *distribution-level
similarity* (does the whole column of torque values still look like training data?). A
batch can pass one check and fail the other — that's exactly the `stress.csv` case: every
value is legal, but as a population it has moved (e.g., `Tool wear` up ~41 min, `Torque` up
~4.7 Nm on average), which is invisible to a per-row schema check.
</details>

### ✅ Checkpoint 11 — Making the retraining call
**Ask students:** "Suppose `Tool wear` and `Torque` both show significant drift in
`stress.csv`. Using the SHAP results from Stage 4, which failure class(es) should you
worry about, and what's your recommendation?"

<details>
<summary>Model Answer</summary>

Cross-referencing SHAP: `Tool wear` is the top driver for **TWF**, and `Torque` +
`Tool wear` jointly drive **OSF**. Since both features have drifted upward in `stress.csv`,
the evidence chain is: *drifted Torque & Tool wear → elevated OSF (and to a lesser extent
TWF) risk → retrain.* The recommended action isn't just "retrain on stress.csv alone" —
it's to retrain on a **mixed dataset** (historical + recent stress-like data) so the model
covers both the original and the new heavy-load operating regime, rather than overfitting
to one.
</details>

---

## 7. Stage 4 — Explainability & Insights *(5 marks)*

**Goal:** Explain the tuned model's predictions in engineering terms, per failure class.

**Tasks:**
- load `best_model.pkl`, run `shap.TreeExplainer` on the training set
- build a 4-panel bar chart of mean |SHAP| per feature, one panel per failure class
  (TWF, HDF, PWF, OSF) → save `shap_per_class.png`
- name the top driver for each class and explain the physical mechanism

### ✅ Checkpoint 12 — Why not one global SHAP ranking?
**Ask students:** "Why does this assignment insist on four *separate* SHAP panels
(one per failure class) instead of one overall feature-importance ranking?"

<details>
<summary>Model Answer</summary>

In a multiclass model, a feature's SHAP value is class-specific — a feature can push
strongly *toward* one class while being irrelevant (or even pushing *away* from) another.
Collapsing everything into one global ranking would average this away and hide real
mechanisms: e.g., `Tool wear` matters most for TWF, `Temp_diff`/`Air temperature` matter
most for HDF, and `Torque` dominates PWF. A single combined ranking would blur these into
a generic "these five features matter somewhat," which is not an engineering-usable
insight. Reading per-class is what lets you say "*this specific* condition raises *this
specific* failure risk."
</details>

---

## 8. Stage 5 — Conclusions *(5 marks)*

**Goal:** Summarize the assignment as a production engineering case, referencing actual
numbers from the student's own run.

**Required in the write-up:**
1. Which model won and why (reference macro F1 numbers)
2. Why accuracy is misleading here
3. Root cause of TWF's poor F1 (data scarcity, ~30 real samples) — full credit even if
   tuned TWF F1 stays at 0.0
4. What drifted in the stress batch and what it implies operationally
5. One actionable, evidence-based recommendation

### ✅ Checkpoint 13 — The TWF trap
**Ask students:** "A student tunes hard with Optuna but TWF's F1 stays exactly 0.0. Did
they fail the assignment? What should their conclusion say?"

<details>
<summary>Model Answer</summary>

No — this is explicitly graded as a **correct diagnosis, not a required fix**. TWF has
only ~30 real historical examples. SMOTE can rebalance the *training set count*, but it
cannot manufacture the real-world diversity of a failure mode that was barely observed —
synthetic points interpolated from 20-odd real examples don't capture the true variety of
how tool-wear failures actually occur. The correct, full-credit conclusion names this as a
**data scarcity problem**, not a modeling failure, and proposes a data-engineering fix
(targeted TWF data collection, cost-sensitive training, or anomaly-style detection for
that class) rather than more hyperparameter tuning.
</details>

### ✅ Checkpoint 14 — Full-loop recap (wrap-up quiz)
**Ask students to answer rapid-fire, one line each:**
1. What does Pandera check that Evidently doesn't?
2. What does Evidently check that Pandera doesn't?
3. Why fit SMOTE after the split, not before?
4. Why is macro F1 preferred over accuracy here?
5. Why read SHAP per class instead of globally?

<details>
<summary>Model Answers</summary>

1. Pandera checks **row-level validity** — type, range, allowed categories.
2. Evidently checks **distribution-level similarity** to a reference dataset — drift.
3. Fitting SMOTE before the split leaks synthetic information derived from what should be
   held-out validation data, inflating apparent performance.
4. Because the dataset is highly imbalanced (~96.7% one class); accuracy can look great
   while completely missing rare failures, which is the opposite of what matters
   operationally.
5. Because a feature's contribution is class-specific in a multiclass model — global
   averaging hides which condition drives which specific failure type.

</details>

---

## 9. Rubric Summary *(Total: 50 marks)*

| Section | Marks |
|---|---|
| 1. Data Loading, Validation & EDA | 15 |
| 2. Experiment Tracking & Model Selection | 15 |
| 3. Drift Detection & Monitoring | 10 |
| 4. Explainability & Insights | 5 |
| 5. Conclusions | 5 |

Full line-item breakdown is in `MLOps_Assignment_rubrics.csv`.

---

## 10. Facilitator Notes

- Run Checkpoints 3–4 (three CSVs / validity vs. drift) **before** Stage 1 coding starts —
  they set up the whole assignment's mental model.
- Run Checkpoints 7–9 (SMOTE, macro F1) **right after** students see the class imbalance
  numbers in their own EDA — the concept lands much better once they've seen 96.7% with
  their own eyes.
- Checkpoint 13 (TWF trap) is worth flagging explicitly to anxious students *before* they
  start Stage 2 — many will assume a 0.0 F1 means they did something wrong.
- If a class is short on time, the four checkpoints that most reliably separate students
  who understand the loop from students who just ran the notebook are: **4, 7, 10, 13**.
