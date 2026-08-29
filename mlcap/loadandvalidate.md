# Predictive Maintenance Workshop — Trainer Flow
### Load → Validate → EDA → Feature Engineer → SMOTE → Train

**Format:** Guided discovery. Each section states the *objective* and gives *clues* (function names, concepts, decisions to make) but not full working code. Participants write the implementation themselves.

---

## 0. Session Map

```text
Load Data
    ↓
Validate (Pandera)
    ↓
EDA (distributions, Torque, Tool wear)
    ↓
Feature Engineering (Power_W, Temp_diff)
    ↓
Encode + Split + SMOTE
    ↓
Train multiple models + MLflow
    ↓
Compare & select best model
```

Tell participants up front: **"You will not copy-paste code today. You will make a decision, write one or two lines, run it, and look at the output before moving on."**

---

## 1. Load the Data (5 min)

**Objective:** Confirm you have three datasets and know their shape before trusting anything in them.

**Talking point:** *"Nothing clever here — just load, print shapes, eyeball columns."*

### 1a. Silence the noise first

```python
import warnings
warnings.filterwarnings('ignore')
```

- This isn't lazy coding — it's a deliberate choice to stop library warnings (deprecation notices, version mismatches, etc.) from cluttering the notebook output during a live walkthrough.
- **Discuss:** *"What's the risk of doing this in production code versus in a workshop notebook?"* (In production you'd want to see and log warnings — silencing them can hide a real problem. Here, it's a presentation choice.)

### 1b. Imports

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

- `pandas` → loading and manipulating tabular data.
- `numpy` → numerical operations you'll need later (e.g. the `2π` in the power formula).
- `matplotlib.pyplot` → the plotting you'll do in the EDA step.
- **Discuss:** *"Why import all three now instead of importing them only when first used?"* (Convention: declare dependencies up front so anyone reading the notebook top-to-bottom knows what's needed before running anything.)

### 1c. Load three CSVs into three roles

```python
train   = pd.read_csv('data/train.csv')
current = pd.read_csv('data/current.csv')
stress  = pd.read_csv('data/stress.csv')
```

- Three files, three **different jobs** in the pipeline — this is worth spelling out explicitly, because participants will keep coming back to this distinction all session:
  - `train` → used to fit the label encoder, the scaler, SMOTE, and the models themselves.
  - `current` → represents "live" or "recent" data the trained model will be scored against. It gets the *same* transformations as train, but never influences fitting.
  - `stress` → a deliberately harder/edge-case dataset used to test how the pipeline behaves outside normal conditions (this is the one that will matter a lot in Session 2 on drift).
- **Discuss:** *"If you fit your label encoder or scaler on `current` or `stress` instead of `train`, what could go wrong?"* (Data leakage / inconsistent encoding — a category seen in `stress` but not `train` could break the encoder, or worse, silently produce a different numeric mapping.)

### 1d. Confirm shapes before doing anything else

```python
print(f'train  : {train.shape}')
print(f'current: {current.shape}')
print(f'stress : {stress.shape}')
```

- `.shape` returns `(rows, columns)`. This is the cheapest possible sanity check — if a file failed to load correctly (wrong delimiter, truncated download, wrong path), the shape will immediately look wrong.
- **Discuss:** *"What would you expect if `stress.csv` has a very different row count from `train.csv`? Is that automatically a problem?"* (Not necessarily — a stress-test dataset is often smaller/targeted by design. But it's worth noticing and asking why.)

### 1e. Build the class-name lookup once

```python
CLASS_NAMES = {0: 'No Failure', 1: 'TWF', 2: 'HDF', 3: 'PWF', 4: 'OSF'}
```

- This dictionary is created **once**, here, at the top of the notebook — and reused in every later section (EDA labels, per-class F1 reporting, MLflow logging). Point this out explicitly: *"You'll see this same dictionary again in three more sections today."*
- **Discuss:** *"Why define this now instead of inline each time you need a label?"* (Single source of truth — if the mapping ever changes, you only update it in one place.)

### 1f. Eyeball the data

```python
train.head()
```

- Not a rigorous check — a human sanity check. Are the column names what you expect? Do the value ranges look plausible at a glance (e.g. temperatures in the 300s, not 3000s)?

**Checkpoint questions for participants:**
- *"Do all three files have the same columns? Are the row counts what you expected?"*
- *"Looking at `.head()`, does anything look obviously wrong — a shifted column, an unexpected data type, a suspicious value?"*
- *"Which of these three datasets do you expect to be the 'cleanest', and which do you expect to cause problems later? Why?"*

---

## 2. Validate with Pandera (10 min)

**Objective:** Define a schema — a rulebook for what a "valid" row looks like — and check each dataset against it.

**Talking point:** *"Pandera is like airport security. It checks each passenger has a valid ticket and ID. It does not check whether 400 passengers from the same city all showing up on one flight is unusual. Valid ≠ normal."*

### 2a. Imports

```python
import pandera as pa
from pandera import Column, DataFrameSchema, Check
```

- `DataFrameSchema` is the container for your rulebook; `Column` describes one field's expected type and constraints; `Check` expresses a specific rule (range, membership, etc.).

### 2b. Define one rule per column

```python
schema = DataFrameSchema({
    'Type':                Column(str,     Check.isin(['L', 'M', 'H'])),
    'Air temperature':     Column(float,   [Check.ge(295.0), Check.le(305.0)]),
    'Process temperature': Column(float,   [Check.ge(305.0), Check.le(315.0)]),
    'Rotational speed':    Column('int64', [Check.ge(1000),  Check.le(2900)]),
    'Torque':              Column(float,   [Check.ge(3.0),   Check.le(80.0)]),
    'Tool wear':           Column('int64', [Check.ge(0),     Check.le(253)]),
    'Failure_Type':        Column('int64', Check.isin([0, 1, 2, 3, 4])),
})
```

Walk through the **two kinds of rules** being mixed here:
- **Type/category rules** — `Type` must be one of exactly three letters (`Check.isin`). `Failure_Type` must be one of exactly five known codes. These are "closed set" checks: anything outside the set is automatically wrong.
- **Range rules** — temperatures, speed, torque, and tool wear each get a `Check.ge` (greater-or-equal) and `Check.le` (less-or-equal) pair, defining a plausible physical envelope for that sensor. These bounds should come from domain knowledge (what the sensor is physically capable of reading), not just "whatever the training data happens to contain."
- **Discuss:** *"Where would these lower/upper bounds realistically come from — the training data's min/max, or the sensor's engineering spec sheet? What's the difference in what each protects you from?"* (Training-data bounds only guarantee "looks like what we've seen"; spec-sheet bounds guarantee "physically plausible," which is a stronger and more useful check.)
- **Discuss:** *"What happens if a genuinely new but valid Torque value — say the machine got upgraded and now tops out at 90 instead of 80 — comes in? Is that the schema's fault, or does it just mean the schema needs updating?"* (Schemas are living documents — they should be revisited when the underlying system changes, not treated as permanent truth.)

### 2c. Validate train and current — the "expected pass" cases

```python
schema.validate(train);   print('PASS: train.csv')
schema.validate(current); print('PASS: current.csv')
```

- `schema.validate(df)` raises an exception immediately on the **first** violation found (fail-fast mode) — that's fine here because you expect these two to be clean.
- **Discuss:** *"Why might it make sense to fail fast on `train` and `current`, but not on `stress`?"* (For train/current, a violation is a signal something's badly wrong upstream and you want to stop immediately. For stress, you want the full picture of *how* it deviates, not just the first thing that trips the schema.)

### 2d. Validate stress — the "collect everything" case

```python
try:
    schema.validate(stress, lazy=True)
    print('PASS: stress.csv (no schema violations)')
except pa.errors.SchemaErrors as e:
    print(f'Schema violations in stress.csv: {len(e.failure_cases)} rows')
```

- `lazy=True` changes the behavior: instead of stopping at the first bad row, Pandera checks *everything* and collects all failures into `e.failure_cases` if any exist.
- The `try/except` structure means: attempt full validation, and if `SchemaErrors` is raised, report how many rows failed rather than crashing the notebook.
- **The twist to sit with:** in this dataset, `stress.csv` actually **passes** cleanly too — same schema, no violations. That's the deliberately counter-intuitive moment of this section.
- **Discuss:** *"If `stress.csv` passes the exact same schema as `train.csv`, does that mean the two datasets are equivalent for modeling purposes? What could be different between them that a schema would never catch?"* (Distribution shift — e.g. `stress` could contain proportionally far more high-torque, high-wear records, all individually within valid range, but collectively representing a very different operating regime. Schema checks per-row plausibility; they say nothing about the population's shape.)
- **Discuss:** *"What's a concrete example of two datasets that are both 100% schema-valid but you'd still be worried about using one to evaluate a model trained on the other?"*

**Checkpoint question:** *"Does `stress.csv` passing the schema check mean the data is normal? Why or why not?"* — Answer to land on: **Valid ≠ normal.** Schema validation checks row-level structural correctness (is this a real ticket and ID); it says nothing about whether the batch as a whole resembles what the model has seen before. That second kind of check — population-level drift detection — is a different tool's job (Evidently, in Session 2).

**Bridge to EDA:** *"So we know every row is individually plausible. Next we actually look at the shape of the data — how many of each failure type, and whether features like Torque and Tool wear behave differently across failure types. That's where patterns Pandera can't see start to show up."*

---

## 3. Exploratory Data Analysis (20 min)

**Objective:** Answer three questions before touching a model:
1. How many examples exist per failure class? (imbalance check)
2. Does Torque differ across failure types?
3. Does Tool wear differ across failure types?

**Layout clue:** One figure, three subplots side by side (`plt.subplots(1, 3, figsize=(16, 4))`).

### 3a. Class distribution (`axes[0]`)
- Use `value_counts().sort_index()` on `Failure_Type`.
- Map class IDs to names via `CLASS_NAMES` before labeling the bar chart.
- Also print count **and** percentage of total for each class — percentage is what makes imbalance obvious.

**Checkpoint question:** *"Which class dominates? What percentage of the data is 'No Failure'? What does that imply for a model that just guesses the majority class?"*

### 3b. Torque by failure type (`axes[1]`)
- First filter out `Failure_Type == 0` — you're only comparing *actual failures* here, not failures vs. non-failures.
- Loop over each remaining class, pull that class's `Torque` values, and plot an overlaid histogram per class.
- Add a legend and title.

### 3c. Tool wear by failure type (`axes[2]`)
- Same pattern as 3b, but with the `Tool wear` column.

**Wrap-up:**
- `plt.tight_layout()` before saving to avoid overlapping labels.
- Save with `plt.savefig(...)`, then `plt.show()`.

**Checkpoint question:** *"Do any failure types show a visibly different Torque or Tool wear distribution from the others? If two classes' histograms look identical, is that feature likely to help separate them?"*

---

## 4. Feature Engineering (20 min)

**Objective:** Create two new features with physical meaning: `Power_W` and `Temp_diff`.

### 4a. `Power_W`
- Physical relationship: **Power = Torque × Angular velocity**.
- Angular velocity needs converting from RPM to rad/s: `RPM × 2π / 60`.
- So: `Power_W = Torque × (Rotational speed × 2π / 60)`.
- Units check: Nm × rad/s ≈ Watts — that's why the feature is named with a `_W` suffix.

### 4b. `Temp_diff`
- Simple subtraction: `Process temperature − Air temperature`.
- Ask participants: *"Why might the difference matter more than either temperature alone?"* (Two machines can have the same process temp but very different deltas from ambient — that delta may signal something the raw values don't.)

### 4c. Make it reusable
- You'll need to apply the **same** two calculations to `train`, `current`, and `stress`. Don't repeat the formula three times — write one function that takes a DataFrame and returns it with the new columns added.
- Inside that function, **copy** the input DataFrame before modifying it (`.copy()`) — don't mutate the caller's original object.

**Checkpoint question:** *"If you apply this function to `current` and `stress`, will the columns be named and computed identically to `train`? Why does that consistency matter later?"*

### 4d. Sanity-check the new features
- Filter out `Failure_Type == 0` again.
- Group by `Failure_Type`, compute `.mean()` of `Power_W` and `Temp_diff` per group.
- Convert the numeric class IDs to names (via `CLASS_NAMES`) in the printed/displayed summary.

**Checkpoint question:** *"Does any failure type show a noticeably higher average Power_W or Temp_diff? Remember: a difference in averages is a hint, not proof — the model will decide if it's actually useful."*

---

## 5. Encode, Split, and Balance (20 min)

### 5a. Encode `Type`
- `Type` is categorical text (`L`, `M`, `H`) — models need numbers.
- Fit a label encoder **on the training data only**, then apply that same fitted encoder to `current` and `stress`. Do not re-fit on each dataset.

### 5b. Define X and y
- Decide which columns are inputs (`X`): machine Type (encoded), both temperatures, rotational speed, torque, tool wear, and your two engineered features.
- Target (`y`): `Failure_Type`.

### 5c. Train/validation split
- Split 80/20.
- Use **stratification** on `Failure_Type` so both sets preserve the class proportions.
- Fix a random seed for reproducibility.

**Checkpoint question:** *"If you didn't stratify, could your validation set end up with zero examples of a rare failure class? What would that do to your evaluation?"*

### 5d. Diagnose imbalance
- Count examples per class in the training split (not the full dataset — you already saw the full picture in EDA).

### 5e. SMOTE — training data only
- Concept: SMOTE generates *synthetic* minority-class examples by interpolating between existing neighbors of the same class — it doesn't just duplicate rows.
- **Critical rule:** apply SMOTE after the split, to `X_train`/`y_train` only. Never touch the validation set.
- Ask participants to state *why* out loud before coding: *"If I SMOTE the validation set too, what am I no longer measuring?"* (Real-world performance — you'd be evaluating against synthetic data that doesn't exist in reality.)
- Store the balanced output as something like `X_res`, `y_res`.

**Checkpoint question:** *"After SMOTE, print the class counts again. Are the minority classes now closer in size to the majority class?"*

---

## 6. Train Multiple Models + Track with MLflow (25 min)

**Objective:** Train several classifiers on the balanced training data, evaluate each on the untouched validation set, and log everything so models are comparable later.

### 6a. Candidate models
Build a collection (e.g. a dict or list) containing:
- **Logistic Regression** — wrap it in a `Pipeline` with a `StandardScaler` first, since your features are on very different scales (temperature ~300, torque ~50, power in the thousands). Use `class_weight='balanced'` as a second layer of imbalance handling.
- **Random Forest** — an ensemble of many trees (e.g. `n_estimators=100`) voting on the final prediction. Good at capturing non-linear relationships.
- **XGBoost** — sequential boosting: each tree tries to correct the previous tree's mistakes. Use `mlogloss` as the multiclass evaluation metric.
- **LightGBM** — another efficient gradient-boosting algorithm, good on tabular data.

### 6b. Loop, don't repeat
- Write one `for` loop over your model collection instead of four copy-pasted training blocks.

### 6c. Inside the loop, for each model:
1. Start an MLflow run (one run = one experiment for one model).
2. Fit on `X_res` / `y_res` (the SMOTE-balanced training data).
3. Predict on `X_val` → this gives `y_pred`.
4. Compare `y_pred` against `y_val` to compute:
   - **Accuracy** — fraction of correct predictions overall.
   - **Macro F1** — F1 per class, averaged with equal weight per class. This is your key metric here, because it won't let the model hide behind strong "No Failure" performance.
   - **Weighted F1** — F1 per class, averaged weighted by class size.
   - **Per-class F1** — F1 for each individual failure type, so you can see exactly which failures the model struggles with.
5. Log parameters and all four metrics to MLflow.
6. Log the trained model itself (with an `input_example`) so you can reload it later without retraining.
7. Save the results into a Python `results` dictionary keyed by model name.

**Checkpoint question before they code the loop:** *"Why do we care about Macro F1 more than plain Accuracy for this dataset specifically?"* (Because ~90% of the data is "No Failure" — a model can get high accuracy while being useless at detecting rare failures. Macro F1 forces equal weighting across classes.)

---

## 7. Compare and Select (10 min)

**Objective:** Turn the `results` dictionary into a comparison and pick a model.

**Clues:**
- Convert `results` into a small table (e.g. a DataFrame) with one row per model and columns for Accuracy / Macro F1 / Weighted F1.
- Sort by Macro F1, not Accuracy, given the imbalance discussion above.
- Look at per-class F1 for your top 1–2 candidates — a model with slightly lower overall Macro F1 but much better performance on a dangerous rare failure (e.g. PWF) might be the more useful choice in a real predictive-maintenance setting.

**Closing discussion prompt:** *"If you had to deploy one model tomorrow, which would you pick, and what's the one number you'd defend that choice with?"*

---

## Full Mental Model (for the whiteboard)

```text
Load → Validate → EDA
                    │
                    ↓
         Feature Engineering
        (Power_W, Temp_diff)
                    │
                    ↓
        Encode Type (fit on train only)
                    │
                    ↓
          Select X (inputs) and y (target)
                    │
                    ↓
        Train/Validation Split (stratified)
             │                    │
             ↓                    │
           SMOTE                  │
     (train only, never val)      │
             ↓                    ↓
     Balanced Training      Untouched Validation
             │                    │
      ┌──────┼─────┬──────┬───────┘
      ↓      ↓      ↓      ↓
     LR     RF   XGBoost LightGBM
      │      │      │      │
      └──────┴──────┴──────┘
               ↓
         Predict on X_val
               ↓
   Accuracy / Macro F1 / Weighted F1 / Per-class F1
               ↓
             MLflow
               ↓
       Compare → Select best model
```

**Core principle to repeat throughout the session:**
> Don't memorize the formula or the Python syntax first. Understand what information you have, what new information could be useful, and how to derive it — then translate that reasoning into code.

---

## Extra Discussion Topics (use as time allows / for faster groups)

### On Load & Validate
- *"`train`, `current`, and `stress` play three different roles in this pipeline. If you had a fourth dataset called `holdout`, which of the three existing roles would it most resemble, and why?"*
- *"Pandera checks range and category — what's a data quality problem it would **never** catch?"* (e.g. duplicated rows, a column that's technically valid but was accidentally shifted one row down, missing values that got silently filled with a valid-looking default like `0`.)
- *"Should schema bounds ever come from the data itself (e.g. `train['Torque'].min()`), or always from an external source of truth? What's the failure mode of each approach?"*
- *"If `schema.validate(train)` fails on day one of a real deployment, what's your first move — patch the schema, or investigate the data source? How do you decide which?"*

### On EDA
- *"The Torque and Tool wear histograms are overlaid per class. What would you do differently if you had 20 failure types instead of 4 — would overlaid histograms still be readable?"* (Leads toward small multiples / faceted plots as an alternative.)
- *"The bar chart shows class imbalance, but it doesn't show *within-class* variance. What plot would show you whether the 'No Failure' class is itself a tight cluster or a wide spread?"* (Box plot, violin plot.)
- *"EDA here only look at Torque and Tool wear individually. What might a scatter plot of Torque vs. Tool wear, colored by Failure_Type, reveal that two separate histograms cannot?"* (Interaction effects — a failure type might only be distinguishable by a *combination* of two features, not either alone.)
- *"Why filter out `Failure_Type == 0` before the Torque/Tool wear plots, but not before the class-distribution bar chart?"* (The bar chart's whole purpose is to show the imbalance including the majority class; the histograms are about distinguishing failure types from *each other*, where the dominant "No Failure" class would visually drown out the others.)

### On Feature Engineering
- *"Power_W and Temp_diff are physically motivated. What's a purely statistical feature (not physically motivated) you could construct from these same columns, and would you trust it as much?"* (e.g. a rolling average or a ratio like Torque/Tool wear — harder to sanity-check because there's no physical unit backing it up.)
- *"If you added a third feature, `Wear_rate = Tool wear / (some time or cycle count)`, what new column would you need that isn't currently in the dataset?"*

### On Split & SMOTE
- *"SMOTE interpolates between existing minority examples. What could go wrong if two 'neighboring' minority examples are actually mislabeled or are outliers?"* (SMOTE would happily generate synthetic examples *between* two bad points, amplifying a labeling error rather than correcting it.)
- *"Besides SMOTE, name another way to handle class imbalance without generating synthetic data."* (`class_weight='balanced'`, undersampling the majority class, collecting more real minority-class data, anomaly-detection framing instead of multiclass classification.)
- *"We stratify the train/validation split by Failure_Type. Should we also stratify by `Type` (L/M/H)? What would that protect against?"*

### On Training & MLflow
- *"Why compare four different model families instead of just tuning one model harder?"* (Different algorithms make different assumptions — linear boundaries vs. tree splits — and with limited time, breadth of approach often beats depth of tuning on a single one.)
- *"MLflow logs the model itself, not just its score. Why does that matter operationally, beyond just picking a 'winner' today?"* (Reproducibility — you can reload the exact model later for inference, audit, or comparison against a future retrained version, without needing to retrain from scratch.)
- *"If two models have nearly identical Macro F1 but very different per-class F1 patterns — one is great at PWF but weak on HDF, the other is the reverse — how would you decide, and what business context would you need to ask about first?"*
- *"What's missing from this evaluation that you'd want before trusting a model in production?"* (e.g. confusion matrix, calibration check, latency/inference cost, testing on `stress.csv` specifically, confidence intervals via cross-validation rather than a single split.)
