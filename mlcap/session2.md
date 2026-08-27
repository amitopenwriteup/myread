# Trainer Readout — Session 2 (With Code + Full Explainers)

Use this as your speaking script. Code blocks are what you show on screen; the paragraphs around them are what you *say*. Deep-dive boxes are included for Optuna, MLflow Registry, Evidently, and SHAP.

---

## 1. Overview *(5 min)*

"Welcome back, everyone. Quick recap of Session 1: we loaded `train.csv`, `current.csv`, and `stress.csv`, validated them with Pandera, explored the class imbalance, engineered `Power_W` and `Temp_diff`, applied SMOTE to the training split only, and started training four candidate models — logging everything into MLflow.

Today we finish the story. We'll compare those four models and pick a winner, squeeze more performance out of it with Optuna, register it properly to a production registry, check whether it's going stale using Evidently, and finally explain its predictions with SHAP so an engineer actually trusts and can act on them. We'll close with grading expectations and a structured conclusion."

---

## 2. Model Comparison and Selection *(20 min)*

"Let's re-run and interpret the four-model comparison we set up last time."

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

        results[name] = {'macro_f1': macro_f1, 'weighted_f1': weighted_f1,
                          'accuracy': acc, 'per_class_f1': per_class}

print(f"{'Model':<22} {'Macro F1':>10} {'Weighted F1':>12} {'Accuracy':>10}")
print('-' * 58)
for name, r in results.items():
    print(f"{name:<22} {r['macro_f1']:>10.3f} {r['weighted_f1']:>12.3f} {r['accuracy']:>10.3f}")

best_name = max(results, key=lambda k: results[k]['macro_f1'])
print(f'\nBest model: {best_name} (macro F1 = {results[best_name]["macro_f1"]:.3f})')
```

"Here's the table you'll typically see:

| Model | Macro F1 | Weighted F1 | Accuracy |
|---|---|---|---|
| Logistic Regression | ~0.531 | — | — |
| LightGBM | ~0.730 | — | — |
| Random Forest | ~0.736 | — | — |
| **XGBoost** | **~0.748** | — | — |

XGBoost wins on macro F1. Notice all four models likely show accuracy above 98% — that's the imbalance trap we talked about in Session 1 rearing its head again. Accuracy tells you almost nothing here; macro F1 is what separates a genuinely useful model from one that's just good at predicting 'No Failure.'

**Why did Logistic Regression fall so far behind?** It's a linear model — it draws straight decision boundaries. Failure boundaries in physical systems (torque thresholds, thermal limits) are often non-linear and involve interactions between features. Tree-based models capture that naturally; logistic regression struggles."

---

## 3. Model Tuning and Registration *(20 min)*

```python
import optuna
import joblib

optuna.logging.set_verbosity(optuna.logging.WARNING)
mlflow.set_experiment('PredMaint_Optuna')

def objective(trial):
    params = {
        'n_estimators':      trial.suggest_int('n_estimators', 100, 500),
        'max_depth':         trial.suggest_int('max_depth', 3, 10),
        'learning_rate':     trial.suggest_float('learning_rate', 0.01, 0.3, log=True),
        'min_child_weight':  trial.suggest_float('min_child_weight', 1.0, 10.0),
        'subsample':         trial.suggest_float('subsample', 0.6, 1.0),
        'colsample_bytree':  trial.suggest_float('colsample_bytree', 0.6, 1.0),
        'gamma':             trial.suggest_float('gamma', 0.0, 2.0),
        'reg_alpha':         trial.suggest_float('reg_alpha', 1e-8, 1.0, log=True),
        'reg_lambda':        trial.suggest_float('reg_lambda', 1e-8, 5.0, log=True),
        'random_state': 42, 'eval_metric': 'mlogloss', 'verbosity': 0,
    }
    m = XGBClassifier(**params)
    m.fit(X_res, y_res)
    return f1_score(y_val, m.predict(X_val), average='macro')

study = optuna.create_study(direction='maximize',
                             sampler=optuna.samplers.TPESampler(seed=42))
study.optimize(objective, n_trials=30, show_progress_bar=False)

best_params = study.best_params
print(f'Optuna best macro F1 : {study.best_value:.4f}')
print(f'Best hyperparameters : {best_params}')
```

> **Extra explainer — Optuna, in plain language**
> "Think about tuning a car engine. You have nine dials — depth, learning rate, subsample ratio, and so on — and you don't know the right combination. You *could* try every possible combination (grid search), but with nine dials that's millions of combinations — way too slow.
>
> **Optuna** is smarter. It uses a strategy called **TPE (Tree-structured Parzen Estimator)** — in plain terms, it looks at which combinations worked well *so far*, builds a rough mental model of 'promising regions,' and spends more of its 30 trials exploring those regions instead of wasting trials on combinations it already suspects are bad. It's like a treasure hunter who, after finding gold in the north corner of a field, starts digging more carefully around that corner instead of randomly all over.
>
> We fix `seed=42` so the search is **reproducible** — anyone re-running this notebook gets the exact same 30 trials and the exact same answer."

```python
best_model = XGBClassifier(**best_params, random_state=42, eval_metric='mlogloss', verbosity=0)
best_model.fit(X_res, y_res)
y_pred_best     = best_model.predict(X_val)
final_macro_f1  = f1_score(y_val, y_pred_best, average='macro')

baseline_f1 = results['XGBoost']['macro_f1']
print(f'Baseline XGBoost macro F1 : {baseline_f1:.4f}')
print(f'Tuned XGBoost macro F1    : {final_macro_f1:.4f}')
print(f'Improvement                : {final_macro_f1 - baseline_f1:+.4f}')

with mlflow.start_run(run_name='XGBoost_Optuna_Best') as run:
    mlflow.log_params(best_params)
    mlflow.log_metric('macro_f1', round(final_macro_f1, 4))
    mlflow.sklearn.log_model(best_model, 'model',
                              registered_model_name='PredMaint_XGBoost',
                              input_example=X_val.iloc[:5])

client = mlflow.MlflowClient()
versions = client.search_model_versions("name='PredMaint_XGBoost'")
latest_v = sorted(versions, key=lambda v: int(v.version))[-1]
client.set_registered_model_alias('PredMaint_XGBoost', 'production', latest_v.version)
print(f'Model v{latest_v.version} promoted to production alias')

joblib.dump(best_model, 'best_model.pkl')
joblib.dump(le, 'label_encoder.pkl')
```

"Tuning takes us from a baseline macro F1 of about 0.748 to about 0.771 — roughly a +0.023 improvement. That may look small on paper, but remember: macro F1 improvements at this stage usually come from getting *slightly* better at the rare, expensive classes, which is exactly what we care about.

**What does 'registering' and 'production alias' actually mean?** MLflow's Model Registry is like a version control system for models — similar to how Git tracks code versions. Every time we register a model under the name `PredMaint_XGBoost`, it gets a new version number. The **alias** `production` is just a movable label — like a sticky note that says 'this is the one currently serving live traffic.' If next month someone trains a better version, they just move the sticky note to the new version number — nothing about the deployment code needs to change, because it always asks for 'whichever version has the production alias,' not a hardcoded version number."

---

## 4. Drift Detection and Monitoring *(20 min)*

```python
from evidently.legacy.report import Report
from evidently.legacy.metric_preset import DataDriftPreset

FEAT_COLS = ['Air temperature', 'Process temperature',
             'Rotational speed', 'Torque', 'Tool wear']

ref = train[FEAT_COLS]
cur = current[FEAT_COLS]

rpt_current = Report(metrics=[DataDriftPreset()])
rpt_current.run(reference_data=ref, current_data=cur)
rpt_current.save_html('drift_current.html')

rc = rpt_current.as_dict()['metrics'][0]['result']
print(f"Dataset drift detected : {rc['dataset_drift']}")
print(f"Drifted features       : {rc['number_of_drifted_columns']} / {rc['number_of_columns']}")
```

> **Extra explainer — Evidently and drift, in plain language**
> "In Session 1 we said: Pandera checks if a value is *valid*. Evidently checks if a *batch* looks statistically like what the model has seen before.
>
> Think of it like this: your doctor takes your temperature and it reads 37°C — perfectly valid, healthy even. But if your temperature *readings* over the last month have steadily crept from 36.5°C to 37.4°C, that trend itself is worth flagging, even though every single reading was 'valid.' That's drift — not about one bad value, but about the *shape* of the data shifting over time.
>
> Evidently compares two datasets statistically — a **reference** set (what 'normal' looked like, our `train.csv`) against a **current** set (what's happening now). For each feature, it runs a statistical test (often using something called the **Wasserstein distance**, which basically measures 'how much effort would it take to reshape one distribution into the other') and flags whether the shift is big enough to matter."

"For `current.csv` — our stable production batch — we typically see **no dataset drift detected**. That makes sense: it's business as usual, same operating conditions as training. No retraining needed here.

Now let's check the stress batch."

```python
from evidently.legacy.metrics.data_drift.dataset_drift_metric import DatasetDriftMetric
from evidently.legacy.metrics.data_drift.column_drift_metric import ColumnDriftMetric

str_batch = stress[FEAT_COLS]

rpt_stress = Report(metrics=[
    DatasetDriftMetric(),
    ColumnDriftMetric(column_name='Tool wear'),
    ColumnDriftMetric(column_name='Torque'),
    ColumnDriftMetric(column_name='Rotational speed'),
    ColumnDriftMetric(column_name='Air temperature'),
    ColumnDriftMetric(column_name='Process temperature'),
])
rpt_stress.run(reference_data=ref, current_data=str_batch)
rpt_stress.save_html('drift_stress.html')

rs = rpt_stress.as_dict()
ds = rs['metrics'][0]['result']
print(f"Dataset drift detected : {ds['dataset_drift']}")
print(f"Drifted features       : {ds['number_of_drifted_columns']} / {ds['number_of_columns']}")

print(f"\n{'Feature':<25} {'Drift':>8} {'Score':>8} {'Ref Mean':>10} {'Cur Mean':>10} {'Delta':>8}")
for m in rs['metrics'][1:]:
    r = m['result']
    col, det, score = r['column_name'], r['drift_detected'], r.get('drift_score', float('nan'))
    rm, cm = train[col].mean(), stress[col].mean()
    print(f"{col:<25} {str(det):>8} {score:>8.4f} {rm:>10.2f} {cm:>10.2f} {cm-rm:>+8.2f}")
```

"Here's what we typically see:

| Feature | Drift | Wasserstein Score | Delta |
|---|---|---|---|
| Tool wear | Yes | 0.645 | +41 min |
| Torque | Yes | 0.474 | +4.7 Nm |
| Rotational speed | Yes | 0.235 | −42 rpm |

This tells a physical story: under heavy load, machines run **slower** (lower rpm) but under **more torque**, and go longer **without maintenance breaks** (higher tool wear). This is real operational drift, not noise.

**Important distinction to repeat here:** `stress.csv` passed Pandera in Session 1 — every value was technically valid. But it clearly fails the drift check here. That's the whole point of running both tools — they catch two completely different failure modes."

---

## 5. Explainability and Insights *(10 min)*

```python
import shap
import numpy as np

best_model = joblib.load('best_model.pkl')
explainer  = shap.TreeExplainer(best_model)
X_explain  = train[FEATURES]
shap_vals  = explainer.shap_values(X_explain)
sv         = np.array(shap_vals)   # shape: (n_samples, n_features, n_classes)

feat_display = ['Type', 'Air Temp', 'Proc Temp', 'Rot Speed',
                'Torque', 'Tool Wear', 'Power (W)', 'Temp Diff']
failure_classes = {1: 'TWF', 2: 'HDF', 3: 'PWF', 4: 'OSF'}

fig, axes = plt.subplots(1, 4, figsize=(20, 5), sharey=True)
for ax, (ci, cn) in zip(axes, failure_classes.items()):
    means = np.abs(sv[:, :, ci]).mean(axis=0)
    sorted_idx = np.argsort(means)
    ax.barh([feat_display[i] for i in sorted_idx], means[sorted_idx], color='coral')
    ax.set_title(cn, fontweight='bold')

plt.tight_layout()
plt.savefig('shap_per_class.png', dpi=120, bbox_inches='tight')
plt.show()

print('Top SHAP driver per failure class:')
for ci, cn in failure_classes.items():
    means = np.abs(sv[:, :, ci]).mean(axis=0)
    top = np.argmax(means)
    print(f'  {cn}: {feat_display[top]} ({means[top]:.3f})')
```

> **Extra explainer — SHAP, in plain language**
> "Imagine asking a model 'why did you predict Overstrain Failure for this specific machine?' A plain model just shrugs and gives you the label. SHAP answers the 'why.'
>
> The idea comes from game theory. Picture the final prediction as a group project grade, and each *feature* (Torque, Tool wear, Temperature...) is a *team member*. SHAP asks: 'how much did each team member individually contribute to the final grade, accounting for how they interacted with each other?' It does this fairly — by testing combinations of features with and without each one included, and averaging the impact.
>
> The result: for every single row, and every single class, we get a number per feature — how much it pushed the prediction *up* or *down* for that class.
>
> **Why is the output 3-dimensional here?** `(n_samples, n_features, n_classes)` — because we have 5 possible classes, SHAP has to explain each class's score separately for every row. A feature like Torque might strongly push toward PWF while barely mattering for HDF. That's why **we never collapse this into one global ranking** — we always read it class by class, exactly like the four-panel chart we just built."

"Typical results:

| Class | Top SHAP Driver |
|---|---|
| TWF | Tool Wear |
| HDF | Air Temperature / Temp Diff |
| PWF | Torque |
| OSF | Torque & Tool Wear jointly |

**Engineering insight — Power_W for PWF:** even though we engineered `Power_W` specifically hoping it would explain power failures, raw `Torque` still dominates. This is a useful, humbling lesson: not every engineered feature outperforms the raw signal — SHAP is what tells us the truth, not our intuition.

**Engineering insight — Temp_diff for HDF:** this one *does* pay off. A *narrow* Temp_diff (not high absolute temperature) is what drives Heat Dissipation Failure — insufficient cooling headroom, not just 'it's hot.' That's a genuinely non-obvious finding you'd never get from looking at raw temperature alone."

---

## 6. Guidelines and Expectations *(10 min)*

"A few housekeeping and grading notes before we wrap up analysis:

- **Submission format:** submit the notebook (`.ipynb`) with every cell already executed — don't make the grader run it. Don't delete or reorder the markdown section headers; the structure is part of the grading rubric.
- **Grading weights:** Section 1 (Data/EDA) = 15 marks, Section 2 (Tracking/Selection/Tuning) = 15 marks, Section 3 (Drift) = 10 marks, Section 4 (Explainability) = 5 marks, Section 5 (Conclusions) = 5 marks. 50 marks total.
- **TWF partial credit rule:** TWF will likely end at F1 = 0.0 no matter how well you tune — it only has ~30 real historical samples. That's expected. **Full credit comes from correctly diagnosing this as a data scarcity problem**, not from artificially forcing the number up.
- **Stress batch expectation:** Section 3 is graded on *diagnosis quality* — did you correctly identify what shifted and why — not on whether your model scores well on `stress.csv`. Trying to 'fix' predictions on the stress batch misses the point of the exercise."

---

## 7. Q&A *(5 min)*

"Let's open it up. Common questions at this stage:

- *Why does a 30-trial Optuna search work when the search space has 9 dimensions?* — Because TPE isn't blind random search; it gets smarter with every trial by learning from what worked.
- *If drift is detected, do we always retrain immediately?* — Not necessarily — first check whether the drift is affecting features the model actually relies on for critical classes. We connect that dot explicitly in Section 3.3, using SHAP results.
- *Why is F1=0 for TWF acceptable at all?* — Because with 30 samples split across train/val, the model may never see a TWF example in validation, or too few to learn a real pattern. The honest, useful answer is 'we need more data,' not a fudged metric."

---

## 8. Conclusion *(5 min)*

"Let's bring it all together with the five key findings expected in your Section 5 write-up:

**1. Model selection:** XGBoost won the baseline comparison at macro F1 ≈ 0.748, ahead of Random Forest (~0.736), LightGBM (~0.730), and Logistic Regression (~0.531). After Optuna tuning, it improved to about 0.771.

**2. Accuracy is misleading:** every model scored above 98% accuracy, purely because 'No Failure' dominates the data. A model can look excellent on paper while missing every failure that actually matters — and each missed failure risks ₹8–15 lakh per hour of downtime. Macro F1 is the metric that actually reflects this business risk.

**3. The TWF problem:** F1 = 0.0 even after SMOTE and tuning, because there are only 30 real historical examples. This is a data engineering problem, not a modeling problem — the fix is targeted data collection for this failure type, not more hyperparameter tuning.

**4. Drift and maintenance scheduling:** the stress batch showed Tool wear drifting +41 min, Torque +4.7 Nm, and Rotational speed −42 rpm. These align most strongly with OSF and TWF risk. Maintenance intervals should be shortened specifically during high-torque, heavy-load periods.

**5. Actionable recommendation:** using the Condition → Risked failure class → Action format — *if Tool wear and Torque both move above their stable baseline together, OSF risk rises; trigger an earlier preventive maintenance check and temporarily lower the alert threshold.*

That completes the full pipeline — from raw sensor data all the way to an explainable, monitored, production-registered model. Great work, everyone."
