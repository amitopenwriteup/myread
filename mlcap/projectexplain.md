# Problem Statement: Predictive Maintenance Classification

## Background

A heavy-equipment manufacturer operates 10,000+ machines on its shop floor. Each machine continuously streams six sensor readings (Type, Air temperature, Process temperature, Rotational speed, Torque, Tool wear). The current maintenance process is **reactive** — technicians respond only *after* a machine fails. Every hour of unplanned downtime costs the company ₹8–15 lakh.

The company wants to move from reactive to **predictive** maintenance: catch early warning signs in sensor data so a failure can be anticipated and prevented, rather than repaired after the fact.

## Objective

Design and build a complete MLOps pipeline that:

1. **Validates** incoming sensor data automatically before it's trusted (data quality gate)
2. **Trains and tracks** a multi-class model that predicts *which* failure type is developing, not just "will it fail"
3. **Tunes** the model to maximize performance on rare, high-cost failure classes — not just overall accuracy
4. **Monitors** the deployed model over time to detect when real-world operating conditions have shifted away from what the model was trained on
5. **Explains** individual predictions so maintenance engineers trust and can act on the model's output, class by class

## Data Provided

| File | Rows | Description |
|---|---|---|
| `data/train.csv` | 6,993 | Historical labelled sensor readings — baseline "normal" |
| `data/current.csv` | 1,499 | Recent stable production batch |
| `data/stress.csv` | 1,499 | Heavy-load production period — potentially drifted |

**Target variable — `Failure_Type`:**

| Code | Name |
|---|---|
| 0 | No Failure |
| 1 | TWF — Tool Wear Failure |
| 2 | HDF — Heat Dissipation Failure |
| 3 | PWF — Power Failure |
| 4 | OSF — Overstrain Failure |

## Constraints & Known Challenges

- The dataset is **heavily imbalanced** — ~97% of readings are "No Failure."
- One failure class (**TWF**) has only ~30 real historical samples — a data scarcity problem no amount of tuning can fully solve.
- `stress.csv` is engineered to be **schema-valid but distributionally different** — it will pass data-quality checks while still representing a shifted operating regime.
- The business needs **explainability**, not just a black-box prediction — every alert must be traceable to a physical cause an engineer can act on.

## Deliverables

1. A validated, feature-engineered dataset (with derived `Power_W` and `Temp_diff` signals)
2. A tracked comparison of 4 candidate models, with the best one tuned and registered to production
3. Drift reports comparing current vs. stress conditions against the training baseline
4. Class-by-class SHAP explanations tying features to specific failure types
5. A written recommendation: should the model be retrained, and what should the maintenance team do differently?

---

# Trainer Questionnaire

Use these questions live while walking through the problem statement — pause after each block, let people answer or discuss before revealing the "intended answer" tied to the solution.

## Block A — Framing the Business Problem

1. Why is "reactive maintenance" expensive for this company? Can anyone put a number on it?
2. If a maintenance engineer gets an alert saying "this machine will fail," what's missing from that alert that would make it actually useful?
3. Why do we need to predict the *type* of failure (TWF, HDF, PWF, OSF) instead of just a binary "will fail / won't fail"?

## Block B — Data Validation (Pandera)

4. What could go wrong on a shop floor with 10,000 machines if we *don't* validate sensor data before feeding it to a model?
5. If a batch of data passes every schema check we define, does that guarantee it's safe to trust for prediction? Why or why not?
6. `stress.csv` is designed to pass validation. What do you think that tells us about the difference between "valid" and "normal" data?

## Block C — Class Imbalance & SMOTE

7. If 97% of the data is "No Failure," what's the easiest (and most useless) way a model could get 97% accuracy?
8. Why might we want to balance the training data but leave the validation data untouched?
9. Can synthetic oversampling (SMOTE) create genuinely *new* information about a failure type, or does it just help the model learn the existing pattern better?

## Block D — Model Selection & Metrics

10. Why is accuracy a poor metric to judge success here? What metric would you propose instead, and why?
11. If two models have similar overall accuracy but very different macro F1 scores, which one would you trust more for this use case, and why?
12. What's the real operational cost of a model with high accuracy but poor recall on OSF (Overstrain Failure)?

## Block E — Hyperparameter Tuning

13. Why tune hyperparameters *after* picking a model family, rather than tuning all four models equally first?
14. What's the risk if we tune only on the validation set and never check against a truly separate holdout?

## Block F — Drift Detection

15. If the model was trained on `train.csv` and performs well on `current.csv`, does that guarantee it will perform well on `stress.csv`? Why might it not?
16. Which sensor readings do you predict will shift the most under heavy-load conditions, and why (think physically — what happens to a machine under stress)?
17. If drift is detected, does that automatically mean we should retrain? What else should we consider first?

## Block G — Explainability (SHAP)

18. Why isn't a single "most important feature overall" ranking useful when we have five different failure types?
19. If SHAP shows that `Tool wear` is the top driver for TWF, but `Torque` is the top driver for PWF, what does that tell an engineer about how to *act differently* for each alert?
20. Why might a derived feature like `Power_W` matter less than expected once we can see its actual SHAP importance?

## Block H — The TWF Problem (Data Scarcity)

21. If a failure class only has 30 real examples, what are the actual options available to us — and which ones are genuine fixes versus band-aids?
22. Is it acceptable for a model to have F1 = 0 on a class, as long as we can explain *why* and propose a concrete next step? What would that next step look like here?

## Block I — Closing Synthesis

23. If you had to explain this entire pipeline to a plant manager in three sentences, what would you say?
24. Which single stage of this pipeline do you think has the highest business risk if skipped — validation, tracking, tuning, drift monitoring, or explainability? Defend your choice.
