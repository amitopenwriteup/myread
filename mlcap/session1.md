# Trainer Readout — Session 1 (With Code + Extra Concept Explainers)

Use this as your speaking script. Code blocks are what you show on screen; the paragraphs around them are what you *say*. I've also added deep-dive boxes for SMOTE and SHAP so you can explain them simply if someone asks.

---

## 1. Overview *(5 min)*

"Good morning/afternoon everyone, welcome to Session 1.

We're building a full MLOps pipeline for predictive maintenance, using five tools: **Pandera** (data validation), **MLflow** (experiment tracking), **Optuna** (tuning), **Evidently** (drift monitoring), and **SHAP** (explainability). Today is about getting the data right — loading, validating, exploring, engineering features, and starting our tracking setup."

---

## 2. Business Objective and Dataset Overview *(20 min)*

"A factory runs 10,000+ machines, each streaming six sensor readings. Every hour of downtime costs ₹8–15 lakh. Instead of reacting after failure, we want to predict it early.

We're given three files, sitting in a `data/` folder:

- **`data/train.csv`** — ~6,993 historical, labelled readings (our baseline of 'normal')
- **`data/current.csv`** — ~1,499 readings from a recent, stable production run
- **`data/stress.csv`** — ~1,499 readings from a heavy-load period

And five failure classes we'll track throughout both sessions:

| Code | Name | Meaning |
|---|---|---|
| 0 | No Failure | Machine healthy |
| 1 | TWF | Tool Wear Failure |
| 2 | HDF | Heat Dissipation Failure |
| 3 | PWF | Power Failure |
| 4 | OSF | Overstrain Failure |

Keep this table close — we'll return to it constantly, especially in SHAP."

---

## 3. Data Loading, Validation, and EDA *(35 min)*

### 3a. Load the data

"First, just load and look."

```python
import warnings
warnings.filterwarnings('ignore')
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

train   = pd.read_csv('data/train.csv')
current = pd.read_csv('data/current.csv')
stress  = pd.read_csv('data/stress.csv')

print(f'train  : {train.shape}')
print(f'current: {current.shape}')
print(f'stress : {stress.shape}')

CLASS_NAMES = {0: 'No Failure', 1: 'TWF', 2: 'HDF', 3: 'PWF', 4: 'OSF'}
train.head()
```

"Nothing clever here — just confirming we have three files (`train.csv`, `current.csv`, `stress.csv`), checking their sizes, and eyeballing the columns before we trust any of it."

### 3b. Validate with Pandera

"Now we define a rulebook — a schema — for what a 'valid' row looks like."

```python
import pandera as pa
from pandera import Column, DataFrameSchema, Check

schema = DataFrameSchema({
    'Type':                Column(str,     Check.isin(['L', 'M', 'H'])),
    'Air temperature':     Column(float,   [Check.ge(295.0), Check.le(305.0)]),
    'Process temperature': Column(float,   [Check.ge(305.0), Check.le(315.0)]),
    'Rotational speed':    Column('int64', [Check.ge(1000),  Check.le(2900)]),
    'Torque':              Column(float,   [Check.ge(3.0),   Check.le(80.0)]),
    'Tool wear':           Column('int64', [Check.ge(0),     Check.le(253)]),
    'Failure_Type':        Column('int64', Check.isin([0, 1, 2, 3, 4])),
})

schema.validate(train);   print('PASS: train.csv')
schema.validate(current); print('PASS: current.csv')

try:
    schema.validate(stress, lazy=True)
    print('PASS: stress.csv (no schema violations)')
except pa.errors.SchemaErrors as e:
    print(f'Schema violations in stress.csv: {len(e.failure_cases)} rows')
```

"`train.csv` and `current.csv` pass cleanly — no surprise there. But watch `stress.csv` — it *also* passes. I want you to sit with that for a second, because it feels counterintuitive.

**Here's the plain-language takeaway:** Pandera is like airport security — it checks whether each passenger has a valid ticket and ID. It does *not* check whether it's unusual that 400 passengers from the same city suddenly all show up on one flight. That second kind of check — 'is this batch behaving like what we've seen before?' — is a completely different job, and that's what Evidently does in Session 2. Valid ≠ normal. Remember that phrase."

### 3c. Explore the data (EDA)

```python
fig, axes = plt.subplots(1, 3, figsize=(16, 4))

vc = train['Failure_Type'].value_counts().sort_index()
axes[0].bar([CLASS_NAMES[k] for k in vc.index], vc.values, color='steelblue')
axes[0].set_title('Failure Type Distribution (train)')

print('Class distribution:')
for k, v in vc.items():
    print(f'  {k} {CLASS_NAMES[k]:<15}: {v:>5} ({v/len(train)*100:.2f}%)')

failures = train[train['Failure_Type'] > 0]
for cls_id, cls_name in CLASS_NAMES.items():
    if cls_id == 0: continue
    axes[1].hist(failures[failures['Failure_Type'] == cls_id]['Torque'], alpha=0.6, label=cls_name, bins=20)
axes[1].set_title('Torque by Failure Type'); axes[1].legend()

for cls_id, cls_name in CLASS_NAMES.items():
    if cls_id == 0: continue
    axes[2].hist(failures[failures['Failure_Type'] == cls_id]['Tool wear'], alpha=0.6, label=cls_name, bins=20)
axes[2].set_title('Tool Wear by Failure Type'); axes[2].legend()

plt.tight_layout()
plt.savefig('eda_distributions.png', dpi=120, bbox_inches='tight')
plt.show()
```

"Three quick questions answered here:
1. Is the data balanced? No — about 97% of readings are 'No Failure.' Hold onto this number, it explains a decision we make in a few minutes.
2. How do Torque and Tool Wear differ across failure types? The histograms show it visually — for example, high Tool Wear values cluster around TWF.
3. What's the L/M/H machine type split? Just a quick sanity check on categorical balance.

This chart gets saved as `eda_distributions.png` — we'll reference it again before drift and SHAP sections in Session 2, so it's worth remembering what 'normal' looked like here."

### 3d. Feature Engineering

```python
def engineer_features(df):
    df = df.copy()
    df['Power_W']   = df['Torque'] * df['Rotational speed'] * 2 * np.pi / 60
    df['Temp_diff'] = df['Process temperature'] - df['Air temperature']
    return df

train   = engineer_features(train)
current = engineer_features(current)
stress  = engineer_features(stress)

summary = train.groupby('Failure_Type')[['Power_W', 'Temp_diff']].mean().round(2)
summary.index = [CLASS_NAMES[i] for i in summary.index]
print(summary)
```

"We create two new columns using basic physics, applied to *all three* files — train, current, and stress:

- **Power_W** = Torque × angular velocity. Physically, this is how much mechanical work the machine is doing per second.
- **Temp_diff** = Process temperature minus Air temperature. This tells us how much 'cooling headroom' the machine has.

We print the average of each, grouped by failure type. This gives us early, cheap hints — for example, which failure types run at unusually high power. We're not proving anything yet, just forming hypotheses. SHAP, in Session 2, will confirm or challenge these hunches with hard numbers."

---

## 4. Model Setup and Experiment Tracking *(20 min)*

### 4a. Split and SMOTE

```python
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
from imblearn.over_sampling import SMOTE

le = LabelEncoder()
train['Type_enc']   = le.fit_transform(train['Type'])
current['Type_enc'] = le.transform(current['Type'])
stress['Type_enc']  = le.transform(stress['Type'])

FEATURES = ['Type_enc', 'Air temperature', 'Process temperature',
            'Rotational speed', 'Torque', 'Tool wear', 'Power_W', 'Temp_diff']

X = train[FEATURES]
y = train['Failure_Type']

X_train, X_val, y_train, y_val = train_test_split(
    X, y, test_size=0.20, stratify=y, random_state=42)

sm = SMOTE(random_state=42, k_neighbors=3)
X_res, y_res = sm.fit_resample(X_train, y_train)

print('Class distribution after SMOTE:')
for cls, cnt in sorted(pd.Series(y_res).value_counts().items()):
    print(f'  {cls} {CLASS_NAMES[cls]}: {cnt}')
```

> **Extra explainer — SMOTE, in plain language**
> "Imagine a classroom of 100 students where 97 always pass and only 3 ever fail. If I build a system that just says 'everyone passes' every time, I'd be right 97% of the time — but completely useless for the thing I actually care about: catching the failures.
>
> That's our data. About 97% of readings are 'No Failure.' If we train on this as-is, a model can get amazing accuracy while never learning what a real failure looks like.
>
> **SMOTE (Synthetic Minority Oversampling Technique)** fixes this by looking at the *few* real examples of each rare failure class and generating new, synthetic examples that sit 'in between' the real ones — mathematically similar but not exact duplicates. It's like saying: 'I only have 3 examples of a failing student, but based on their pattern, here are 20 plausible variations to learn from.'
>
> **Critical rule — we only apply this to the training set.** The validation set stays exactly as it came, imbalance and all, because that's what real-world deployment looks like. If we balanced the validation set too, our performance numbers would lie to us — they'd look great on paper but fail in production, where failures really are rare."

### 4b. Train and log models with MLflow

```python
import mlflow
import mlflow.sklearn
from sklearn.metrics import f1_score, accuracy_score
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from xgboost import XGBClassifier
from lightgbm import LGBMClassifier

mlflow.set_tracking_uri('sqlite:///mlflow.db')
mlflow.set_experiment('PredMaint_ModelSelection')

CLASS_LIST = [0, 1, 2, 3, 4]
results = {}

models_to_run = {
    'LogisticRegression': Pipeline([
        ('scaler', StandardScaler()),
        ('lr', LogisticRegression(max_iter=2000, random_state=42, class_weight='balanced'))
    ]),
    'RandomForest': RandomForestClassifier(n_estimators=100, random_state=42, class_weight='balanced'),
    'XGBoost':      XGBClassifier(n_estimators=100, random_state=42, eval_metric='mlogloss', verbosity=0),
    'LightGBM':     LGBMClassifier(n_estimators=100, random_state=42, class_weight='balanced', verbose=-1),
}

for name, model in models_to_run.items():
    with mlflow.start_run(run_name=name):
        model.fit(X_res, y_res)
        y_pred      = model.predict(X_val)
        macro_f1    = f1_score(y_val, y_pred, average='macro')
        weighted_f1 = f1_score(y_val, y_pred, average='weighted')
        acc         = accuracy_score(y_val, y_pred)
        per_class   = f1_score(y_val, y_pred, average=None, labels=CLASS_LIST)

        mlflow.log_param('model', name)
        mlflow.log_metric('macro_f1', round(macro_f1, 4))
        mlflow.log_metric('weighted_f1', round(weighted_f1, 4))
        mlflow.log_metric('accuracy', round(acc, 4))
        for i, cn in CLASS_NAMES.items():
            mlflow.log_metric(f'f1_{cn.replace(" ","_")}', round(per_class[i], 4))
        mlflow.sklearn.log_model(model, 'model', input_example=X_val.iloc[:5])

        results[name] = {'macro_f1': macro_f1, 'weighted_f1': weighted_f1, 'accuracy': acc}
```

"We're training four models — Logistic Regression, Random Forest, XGBoost, LightGBM — all on the SMOTE-balanced training set, then evaluating on the untouched, imbalanced validation set (that's deliberate — see the SMOTE note above).

Every run gets a folder inside MLflow: which model it was, its macro F1, weighted F1, accuracy, and the F1 score for each of the five classes individually. Today's job is just getting this loop running and logged cleanly. We compare and crown a winner at the start of Session 2."

---

> ## Extra explainer — SHAP, in plain language *(preview for Session 2)*
> "Someone might ask — why do we even need SHAP if the model already gives predictions?
>
> Think of the model as a doctor who says 'this patient has condition X' but refuses to explain *why*. That's not good enough in a factory — if the model says 'this machine will have an Overstrain Failure,' the maintenance engineer needs to know: *is it because of the torque? The tool wear? Something else?*
>
> **SHAP (SHapley Additive exPlanations)** answers that. For every single prediction, it tells you how much *each individual feature* pushed the model toward or away from a particular class. For example, it might say: 'For this row, high Torque pushed the prediction toward PWF by +0.4, while normal Tool Wear pushed it away by -0.1.'
>
> Two things to remember for later:
> 1. Because we have *five* failure classes, SHAP doesn't give one ranking — it gives a *separate* ranking of important features *per class*. A feature like Torque might matter a lot for PWF but barely matter for HDF. We never collapse this into one 'global winner' feature — we read it class by class.
> 2. This is also why our engineered features (`Power_W`, `Temp_diff`) matter — SHAP will tell us whether our physics-based engineering actually paid off, or whether the raw sensor values were just as good on their own.
>
> We'll compute and visualize this properly in Session 2, once we have a final tuned model — but it's good to plant this idea now, since we already started forming hypotheses about power and temperature during today's EDA."

---

## 5. Q&A *(5 min)*

"Let's pause for questions. Common ones at this stage:

- *Why does `stress.csv` pass Pandera validation if it's meant to look different?* — Because Pandera checks value *validity*, not statistical *normality*. Evidently, in Session 2, checks the second thing.
- *Why SMOTE only on the training set, never validation?* — Because we want our performance numbers to reflect the real, imbalanced world our model will actually be deployed into.
- *What's the point of Power_W and Temp_diff if we already have Torque, Speed, and Temperature?* — They combine raw signals into physically meaningful quantities that may capture patterns the raw values miss individually. SHAP will confirm whether that bet paid off.

Take a moment for the room's actual questions here."

---

## 6. Conclusion *(5 min)*

"Let's recap. We loaded three files — `train.csv`, `current.csv`, `stress.csv` — and understood why predictive maintenance matters: catching failures before they cost ₹8–15 lakh per hour. We built a Pandera schema and learned that *valid data isn't the same as normal data* — a lesson that pays off directly in Session 2's drift section. We explored the data and found it's heavily imbalanced toward 'No Failure.' We engineered two physics-based features, Power_W and Temp_diff, and formed some early hypotheses about which failure types they might explain. And we set up SMOTE — balancing only the training data, never the validation data — before training and logging four models into MLflow.

In Session 2, we'll compare these four models and pick a winner, tune it with Optuna, register it to production, check it for drift with Evidently, and finally open it up with SHAP to explain exactly why it predicts what it predicts. See you there."
