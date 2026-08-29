# Trainer Guide: How to Walk Learners Through the Code Sections
### (Session 2 — Predictive Maintenance Pipeline)

This guide is for the *trainer*, not the learner. For every code block in the readout, it gives you:
- **How to approach it** — the pedagogical sequence (what to show, what to hide, what to ask first)
- **Prompt(s) to pose to learners** — actual questions you say out loud or drop in chat/whiteboard
- **What a good answer sounds like**, so you can tell if the room actually understood it
- **A misconception to pre-empt**, because these are the errors learners make every year

The general rhythm for *every* code block: **predict → reveal → interrogate → connect**. Don't just read code top to bottom. Cover the output, ask learners to predict it, then reveal and ask them to explain the gap between guess and reality.

---

## Section 2 — Model Comparison Code

### How to approach it
Don't start with the code. Start with the *table*. Show learners the four models and their accuracy numbers (hide macro F1 first) and let them almost fall into the imbalance trap themselves — that's more memorable than you warning them upfront.

**Suggested sequence:**
1. Show only the `accuracy` column of the results table. Ask the prompt below.
2. Reveal `macro_f1` and `weighted_f1` side by side. Ask learners to explain the disagreement.
3. Now walk the code top to bottom, pausing at `average='macro'` vs `average='weighted'` vs `average=None`.
4. Close with the "why did Logistic Regression fall behind" discussion.

### Prompts to give learners
- *"All four models score above 98% accuracy. Based on that number alone, which one would you ship to production — and what does your gut tell you might be wrong with that decision?"*
- *"Look at the `f1_score` calls in the code — there are three variants: `macro`, `weighted`, and `None`. Before I tell you, guess: which one would go down the most if the model completely failed to predict the rarest class, and why?"*
- *"If I told you Logistic Regression scored 0.531 macro F1 while XGBoost scored 0.748, what does that gap tell you about the *shape* of the decision boundary the failures require?"*

### What a good answer sounds like
Learner should articulate that accuracy is dominated by the majority class (`No Failure`), so a model that never predicts a rare failure class can still score >98%. Macro F1 treats every class equally regardless of size, which punishes that behavior — that's *why* it's the metric to optimize here, not accuracy.

### Misconception to pre-empt
Learners often think "weighted F1" is the "fair" one because it sounds more sophisticated. Clarify: weighted F1 still lets large classes dominate the score — it just reweights by class frequency instead of ignoring it. **Macro is the only one that gives rare failure classes equal say.**

---

## Section 3a — Optuna Hyperparameter Tuning

### How to approach it
This code block is dense (9 hyperparameters). Don't explain every parameter individually — that's a losing battle. Instead, anchor the whole block to one question: *"what is `objective(trial)` actually being asked to do?"* Once that clicks, the specific hyperparameters are just details learners can look up later.

**Suggested sequence:**
1. Ask learners to read only the `return` line of `objective()` before anything else.
2. Ask them to describe, in one sentence, what Optuna is optimizing *for*.
3. Then reveal the `study.optimize(objective, n_trials=30, ...)` line and connect it back.
4. Use the treasure-hunter/TPE analogy only *after* they've attempted their own explanation — don't lead with the analogy, let them build their own first.

### Prompts to give learners
- *"Before reading the parameter grid, just look at the last line of `objective()`. In your own words: what is Optuna trying to maximize, and against which dataset?"*
- *"We only run 30 trials for a 9-dimensional search space. A grid search over the same 9 dials at even 5 values each would need over a million runs. What does that tell you about what TPE must be doing differently from grid search?"*
- *"Why do we fix `seed=42` here? What would break if we didn't?"*

### What a good answer sounds like
Optuna calls `objective(trial)` repeatedly, each time trying a new combination of the 9 hyperparameters, training an XGBoost model on `X_res`/`y_res`, and scoring it with macro F1 on the *validation* set (`X_val`/`y_val`) — never on the training set itself. TPE uses the results of earlier trials to bias later trials toward promising regions, rather than sampling blindly, which is why 30 trials suffice. Seeding it makes the 30 trials — and therefore the final answer — reproducible across reruns.

### Misconception to pre-empt
Learners often think Optuna is testing on the training set the same way `.fit()` does, and conflate "best trial score" with "training accuracy." Be explicit: the objective function's score comes from `X_val`, which is *held out* — that's the whole point.

---

## Section 3b — MLflow Registration & Production Alias

### How to approach it
This is a conceptual block disguised as code — the actual logic (`log_params`, `log_metric`, `log_model`) is straightforward MLflow API calls. The real teaching moment is `set_registered_model_alias`. Spend most of your time there.

**Suggested sequence:**
1. Quickly walk the `mlflow.start_run()` block — this should feel familiar from Session 1.
2. Stop at `client.set_registered_model_alias(...)`. Ask the prompt below *before* giving the Git/sticky-note analogy.
3. Ask learners to imagine a deployment script and how it would reference the model — this is where the "alias vs hardcoded version number" idea lands.

### Prompts to give learners
- *"If we register three more model versions next month, and none of the deployment code changes, how is that possible — what part of this code makes that true?"*
- *"What's the practical difference between writing `model_version=7` in your deployment script versus `alias='production'`? Imagine you're on-call at 2am and a bad model just got promoted — which setup lets you roll back faster?"*

### What a good answer sounds like
Learner should recognize `production` as a *pointer* that can be reassigned to any version number, so promoting a new model is just moving the alias — not touching deployment code, not redeploying. Rolling back is just re-pointing the alias to the previous version.

### Misconception to pre-empt
Learners sometimes think the alias *is* the version — clarify it's a separate, movable label on top of an immutable, auto-incrementing version number. The version never changes; only what the alias points to changes.

---

## Section 4 — Evidently Drift Detection

### How to approach it
This section has a built-in contrast (stable `current.csv` vs `stress.csv`) — use it. Don't reveal both results at once; run them one at a time so learners form an expectation for the second based on the first.

**Suggested sequence:**
1. Run/show the `current.csv` drift check first. Confirm: no drift. Ask learners *why that makes sense* before moving on.
2. Before running the stress batch, ask learners to predict which of the 5 features will drift the most, and in which direction.
3. Reveal the stress table. Compare predictions to the actual deltas.
4. Explicitly return to Session 1's Pandera check and ask the connecting prompt.

### Prompts to give learners
- *"`current.csv` shows no dataset drift. Given what you know about how this batch was generated, why is that the expected — even boring — result?"*
- *"Before I show you the stress batch numbers: physically, if a machine is under heavier load, what do you expect happens to rotational speed, torque, and tool wear? Write down a direction (up/down) for each before we check."*
- *"Every single row in `stress.csv` passed Pandera's validation in Session 1 — no nulls, no out-of-range values, nothing technically wrong. Yet Evidently flags heavy drift here. How can both of those be true at the same time?"*

### What a good answer sounds like
Under heavier load: torque up, rotational speed down, tool wear up (longer stretches between maintenance) — this matches the actual table. On the Pandera-vs-Evidently question: Pandera checks *row-level validity* (is this one value plausible?), Evidently checks *distributional shape over a batch* (does this whole batch of values still resemble the training distribution?). A value can be perfectly valid and still be part of a batch that's drifted.

### Misconception to pre-empt
Learners often assume "passed validation" means "safe to trust the model's predictions on this data." Make the distinction explicit and blunt: **valid ≠ familiar to the model.**

---

## Section 5 — SHAP Explainability

### How to approach it
This is the section where learners most often try to shortcut to "just tell me the important feature." Resist that. The teaching goal is that importance is *class-specific*, not global — the 3D shape of `sv` is the whole lesson.

**Suggested sequence:**
1. Ask learners to guess the shape of `shap_vals` *before* revealing the `# shape:` comment.
2. Once revealed, ask why there's a `classes` dimension at all — connect back to this being a multi-class (5-class) problem, not binary.
3. Walk the four-panel chart. For each panel, ask learners to name the top bar *before* scrolling to the printed "Top SHAP driver" summary.
4. Land on the two "engineering insight" callouts (`Power_W` for PWF, `Temp_diff` for HDF) as the payoff — this is where SHAP earns its keep versus intuition.

### Prompts to give learners
- *"We engineered `Power_W` specifically because we suspected it would explain Power Failure (PWF) better than raw features. SHAP says raw `Torque` still dominates. What does that tell you about the risk of trusting your own feature-engineering instincts without checking?"*
- *"For Heat Dissipation Failure (HDF), the top driver is `Temp_diff`, not absolute temperature. What's the physical difference between 'it's hot' and 'the temperature difference is narrow' — and why would a machine care more about the second one?"*
- *"Why does the code loop over `failure_classes` and build four separate bar charts instead of one global importance ranking? What would we lose if we collapsed it into a single chart?"*

### What a good answer sounds like
`shap_vals` is `(n_samples, n_features, n_classes)` because with 5 classes, SHAP must attribute contribution to *each class's* score separately — a feature can matter hugely for one failure type and barely register for another. Collapsing to one global ranking would hide that a feature like Torque dominates PWF/OSF but not HDF. On the engineering insights: a narrow `Temp_diff` means low cooling headroom even at moderate absolute temperature — that's a genuinely non-obvious, physically meaningful finding, and it's a caution against assuming an engineered feature is automatically better just because it was designed with a failure mode in mind.

### Misconception to pre-empt
Learners frequently want a single "most important feature" answer for the whole model. Explicitly forbid that framing here: **there is no single global ranking in a multi-class SHAP analysis worth trusting — only per-class rankings.**

---

## Quick-Reference: One-Line Prompt Per Section (for a fast-paced cohort)

If you're short on time, use just these five prompts as checkpoints — one per code section:

1. **Model comparison:** "Why can a model score 98% accuracy and still be useless?"
2. **Optuna:** "What is `objective(trial)` actually measuring, and on which dataset?"
3. **MLflow registry:** "How does an alias let you change the production model without touching deployment code?"
4. **Evidently:** "How can a batch pass every row-level validation check and still show drift?"
5. **SHAP:** "Why is there no single 'most important feature' for this model?"

If a learner can answer all five unprompted by the end of the session, they've understood the pipeline — not just run the code.
