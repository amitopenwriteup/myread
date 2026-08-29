# Predictive Maintenance Workshop — Trainer Flow
### Load → Validate → EDA → Feature Engineer → SMOTE → Train

**Format:** Guided discovery. Each section states the *objective* and gives *clues* (function names, concepts, decisions to make) but not full working code. Participants write the implementation themselves.

---

## 0. Session Map (this doc covers Load → Validate; EDA and beyond follow in the next part)

```text
Load Data
    ↓
Validate (Pandera)
    ↓
   ... EDA, Feature Engineering, SMOTE, Training (next session part)
```

Tell participants up front: **"You will not copy-paste code today. You will make a decision, write one or two lines, run it, and look at the output before moving on."**

---

## 1. Load the Data (5 min)

**Objective:** Before doing anything clever, establish three plain facts: *what data do I have, how much of it, and what job does each piece play?*

**Talking point:** *"Nothing clever here — just load, print shapes, eyeball columns."*

### The logic, not the syntax

Loading data feels trivial, but there's a decision hiding inside it that shapes the entire rest of the day: **you're not loading one dataset, you're loading three datasets that will each play a different role in the pipeline.** If participants don't internalize this now, later steps (fit-on-train-only, SMOTE-train-only, evaluate-on-validation-only) will feel like arbitrary rules instead of consequences of this one decision.

```text
              Raw CSV files
                    │
    ┌───────────────┼───────────────┐
    ↓               ↓               ↓
 train.csv      current.csv     stress.csv
    │               │               │
    ↓               ↓               ↓
"Ground truth   "New data the    "Deliberately
 to learn from"  model will      harder / edge-
                 be scored on"   case conditions"
    │               │               │
    ↓               ↓               ↓
Fits: encoder,   Gets the SAME   Gets the SAME
scaler, SMOTE,   transformations transformations
the models       — never fits    — never fits
themselves       anything        anything
```

- **The core rule:** anything that *learns* a parameter from data (the label encoder's category mapping, the scaler's mean/std, SMOTE's synthetic neighbors, the model's weights) learns it from `train` **only**. `current` and `stress` only ever get *transformed* using what was already learned — they never get to influence that learning.
- **Why this matters:** if you let `current` or `stress` leak into fitting, you're no longer measuring "how will this model perform on data it hasn't seen" — you've quietly let it see the future.
- **The `CLASS_NAMES` mapping** you define here is a small instance of the same idea: define the "vocabulary" once, in one place, and reuse it everywhere downstream (EDA labels, per-class F1 reporting, MLflow logs) so it can never drift out of sync with itself.
- **Checking shapes and previewing rows** isn't about the numbers themselves — it's the cheapest possible trust check before you build anything on top of the data. If a file didn't load the way you expect, you want to know *now*, not three steps later when a model quietly trains on garbage.

**Discussion questions:**
- *"If you had a fourth file called `holdout.csv`, which of the three existing roles — train, current, or stress — would it most resemble, and why?"*
- *"What's the risk if you accidentally fit your encoder or scaler on `current` instead of `train`?"* (You'd be encoding based on categories/statistics from data the model is supposed to be evaluated on — a form of leakage, and if `current` ever has a category `train` didn't, the encoding could break or shift meaning entirely.)
- *"Why decide the role of each dataset *before* looking at the data, rather than deciding after you've explored it?"* (Otherwise it's tempting to redefine roles based on what's convenient after seeing the numbers — which reintroduces the leakage risk this structure exists to prevent.)

---

## 2. Validate with Pandera (10 min)

**Objective:** Before trusting a single row, ask: *is this data structurally the kind of data I told myself to expect?*

**Talking point:** *"Pandera is like airport security. It checks each passenger has a valid ticket and ID. It does not check whether 400 passengers from the same city all showing up on one flight is unusual. Valid ≠ normal."*

### The logic, not the syntax

A schema is a **contract you write down about your own assumptions**, so that instead of silently trusting the data, you make the assumptions explicit and testable.

```text
        Your assumptions about the data
                     │
                     ↓
         Write them down as a SCHEMA
      (one rule per column: type + range)
                     │
                     ↓
        Check each dataset against it
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     train        current       stress
        │            │            │
        ↓            ↓            ↓
   Expect PASS   Expect PASS   Might reveal
   (fail fast    (fail fast    something —
   if not)       if not)       check EVERY
                               row, not just
                               the first bad one
```

There are two categories of rule being written, and it's worth naming both explicitly:

- **Closed-set rules** — "this value must be one of a known, finite list" (e.g. `Type` must be `L`, `M`, or `H`; `Failure_Type` must be one of the five known codes). Anything outside the set is automatically wrong — there's no ambiguity.
- **Range rules** — "this value must fall inside a plausible physical envelope" (temperatures, torque, tool wear, rotational speed). This is where a real design decision hides: **should the bounds come from what the training data happens to contain, or from what the sensor is physically capable of measuring?** Data-derived bounds only prove "looks like what we've already seen." Spec-derived bounds prove "physically possible" — a stronger and more durable guarantee, because it doesn't need updating just because you collected slightly wider data next quarter.

**Why check `train` and `current` differently from `stress`:** for `train` and `current`, you expect a clean pass — if either fails, something upstream is broken and you want to know immediately (fail fast). For `stress`, you deliberately want the *complete* picture of every violation, not just the first one, because the whole point of stress-testing is to see the full extent of how far the data pushes against your assumptions.

**The twist that's the actual teaching moment of this section:** in this dataset, `stress.csv` **also passes** the schema cleanly. No violations. That's not a bug in the schema — it's the whole point. A schema checks that each row, individually, looks like a plausible reading. It says nothing about whether the *collection* of rows, as a population, resembles the population the model was trained on. `stress.csv` could be full of rows that are each perfectly valid on their own, while collectively representing an operating regime — e.g. far more high-torque, high-wear machines — the model has barely seen.

```text
Schema validation checks:      Schema validation CANNOT check:
"Is this one ticket valid?"    "Is this whole planeload of
                                passengers unusual compared
                                to every other flight this
                                airline has ever flown?"
```

### Where do the schema's numbers actually come from?

Before writing a single `Check.ge(...)` / `Check.le(...)`, someone had to look at the CSV and decide what "valid" means for each column. That step is invisible in the final code block, but it's the actual reasoning work — worth walking through explicitly.

```text
        Open the raw CSV
              │
              ↓
   For each column, ask two questions:
              │
   ┌──────────┴──────────┐
   ↓                      ↓
"What TYPE of value   "What RANGE / SET of
 is this, technically?" values is plausible?"
   │                      │
   ↓                      ↓
 text / integer /    look at train.describe(),
 float               .unique(), .min(), .max()
   │                      │
   └──────────┬───────────┘
              ↓
     Turn each answer into
     one Column(...) rule
```

Concretely, before this schema existed, the workflow looked like:

- **For `Type`:** run `train['Type'].unique()` on the CSV → it returns exactly `['L', 'M', 'H']`. That observed, closed set of three letters becomes `Check.isin(['L', 'M', 'H'])`. This is a case where the CSV *tells you* the exact rule — there's little judgment involved beyond "trust what's actually in the data."
- **For `Failure_Type`:** same idea — `train['Failure_Type'].unique()` shows only the codes `0` through `4`, matching the `CLASS_NAMES` dictionary you already built in the Load step. That's why the check is `Check.isin([0, 1, 2, 3, 4])`.
- **For the numeric sensor columns** (`Air temperature`, `Process temperature`, `Rotational speed`, `Torque`, `Tool wear`): here the CSV alone isn't enough — running `train['Air temperature'].describe()` gives you the min/max *of this particular sample*, but that's a narrower question than "what range is physically possible for this sensor." The bounds you saw in the schema (e.g. `295.0`–`305.0` for Air temperature, `3.0`–`80.0` for Torque) are a blend of **what the CSV's `.describe()` output showed** and **a bit of headroom added on top**, reasoning like: *"the training data ranges from about 296 to 304, and this is an ambient-temperature sensor, so a bound of 295–305 gives a small realistic margin without being so loose it stops catching genuine errors."*
- **For the dtypes** (`int64` vs `float`): this comes straight from how pandas already parsed the column when you ran `train.dtypes` or looked at `.head()` — `Rotational speed` and `Tool wear` load as whole numbers (`int64`), while the temperatures and Torque load with decimals (`float`).

**The general pattern to teach:** a schema is not invented from imagination — it's **reverse-engineered from a first honest look at the CSV**, then tightened using domain judgment about what's *physically* plausible versus merely what happened to appear in this one sample.

**Discuss:** *"If you only used `train['Torque'].min()` and `.max()` directly as your schema bounds — no added margin — what's the risk the very next batch of real data trips a false alarm?"* (Real sensors have natural variation; a schema with zero margin around a single sample's exact range will flag perfectly normal future readings as violations.)

**Discuss:** *"For `Type`, using `.unique()` to build the rule felt safe and direct. Would you trust `.min()`/`.max()` on `Torque` the same way, with zero adjustment? Why might categorical columns and numeric columns deserve different levels of trust in what the raw CSV shows you?"* (Categorical values are either present or not — there's no "just outside the range" case. Numeric ranges are continuous, so the exact sample min/max is really just one lucky/unlucky draw, not a hard boundary.)

**Discussion questions on the schema itself:**
- *"If `stress.csv` passes the exact same schema as `train.csv`, does that mean the two datasets are equivalent for modeling purposes? What could be different between them that a schema would never catch?"* (Distribution shift — same valid ranges, very different proportions/shape within those ranges.)


- *"Where should schema bounds come from — the training data's own min/max, or an external source like a sensor spec sheet? What failure mode does each protect against, and what failure mode does each miss?"*
- *"A new but legitimate Torque value shows up that's outside your current bounds — say the machine got upgraded and now reaches higher torque. Is that the schema's fault, or a sign the schema needs to evolve?"* (Schemas encode assumptions at a point in time — they need to be revisited, not treated as permanent truth.)
- *"What's a data quality problem a schema like this would never catch, even in principle?"* (Duplicated rows, a column shifted down by one row, missing values silently backfilled with a valid-looking default like `0`.)

**Checkpoint question to land the section:** *"Does `stress.csv` passing the schema check mean the data is normal?"* — **No. Valid ≠ normal.** Row-level structural correctness and population-level resemblance to what you've seen before are two different questions, answered by two different tools. Pandera answers the first. Detecting the second — "is this batch behaving like what we've seen before?" — is a job for drift detection (Evidently, in Session 2).

**Bridge to EDA:** *"So every row, individually, is plausible. Next we zoom out and actually look at the shape of the data as a whole — how many of each failure type, and whether features like Torque and Tool wear behave differently across failure types. That's where patterns a schema can't see start to become visible."*
