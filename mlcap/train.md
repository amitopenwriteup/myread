Yes. Let's continue with the **same learner-focused approach**: first understand the objective and decision-making, then the learner can write the Python code themselves.

# Objective

At this stage, we have already:

```text
Raw data
   ↓
Feature engineering
   ↓
Encode categorical data
   ↓
Train/validation split
   ↓
SMOTE on training data
```

Now our objective is:

> **Train several different ML models, compare their performance, track the experiments, and identify which model is best for predicting machine failure type.**

Instead of assuming one algorithm is best, we test several:

* Logistic Regression
* Random Forest
* XGBoost
* LightGBM

The overall flow is:

```text
Balanced Training Data
        ↓
 ┌──────┼────────┬─────────┐
 ↓      ↓        ↓         ↓
Logistic Random  XGBoost  LightGBM
Regression Forest
 ↓      ↓        ↓         ↓
       Predictions
            ↓
      Evaluate models
            ↓
     Compare F1 / Accuracy
            ↓
        MLflow
            ↓
      Select best model
```

---

# Step 1: Import MLflow

MLflow is used to **track your ML experiments**.

Imagine you train four models.

Without MLflow, you might manually write:

```text
Logistic Regression → F1 = 0.72
Random Forest       → F1 = 0.84
XGBoost             → F1 = 0.87
LightGBM            → F1 = 0.86
```

As your project grows, this becomes difficult to manage.

MLflow records these results automatically.

Think of MLflow as an **experiment notebook for machine learning**.

It can record:

* Which model was used
* Model parameters
* Accuracy
* F1 score
* Per-class performance
* The trained model itself

---

# Step 2: Import evaluation metrics

You import:

* **F1 score**
* **Accuracy**

These are used to answer:

> How well did each model predict the failure types?

### Accuracy

Accuracy asks:

> Out of all predictions, how many were correct?

For example:

```text
100 predictions
85 correct
15 incorrect
```

Accuracy = 85%.

---

# Step 3: Why use F1 score?

This is especially important for your problem because you have **multiple failure classes and class imbalance**.

A model could have high accuracy simply because it predicts the majority class very well.

For example:

```text
No Failure = 90% of data
```

A model that predicts "No Failure" most of the time could achieve around 90% accuracy while being terrible at detecting rare failures.

That's why you also use **F1 score**.

F1 considers both:

* Precision
* Recall

So it gives a better indication of whether the model is actually detecting the failure classes.

---

# Step 4: Why `macro_f1`?

You calculate:

```text
Macro F1
```

This means:

> Calculate F1 separately for every class and then give every class equal importance when averaging.

Imagine:

| Class      |   F1 |
| ---------- | ---: |
| No Failure | 0.98 |
| TWF        | 0.80 |
| HDF        | 0.70 |
| PWF        | 0.60 |
| OSF        | 0.50 |

Macro F1 treats each class equally.

This is useful because **rare failures matter too**.

You don't want your model to look good only because it performs well on `No Failure`.

---

# Step 5: What is weighted F1?

You also calculate:

```text
Weighted F1
```

Here, classes with more samples have more influence on the final score.

So:

```text
Macro F1
→ every class has equal importance

Weighted F1
→ bigger classes have more influence
```

For your predictive-maintenance problem, **Macro F1 is often particularly useful when you care about performance across all failure categories**, especially minority failures.

---

# Step 6: Define the classes

You have:

```text
0
1
2
3
4
```

as the failure classes you're evaluating.

The code stores these in `CLASS_LIST`.

The purpose is to make sure the per-class F1 calculation evaluates the classes in a known order.

For example:

```text
0 → No Failure
1 → TWF
2 → HDF
3 → PWF
4 → OSF
```

The exact names come from your `CLASS_NAMES` dictionary.

---

# Step 7: Prepare several models

Now comes the main idea.

Instead of training just one algorithm, you create a collection of candidate models.

Conceptually:

```text
models_to_run

├── Logistic Regression
├── Random Forest
├── XGBoost
└── LightGBM
```

Then your program can train all four automatically.

This is called **model selection**.

---

# Step 8: Logistic Regression

The first model is **Logistic Regression**.

Although its name contains "Regression", it can be used for classification.

Your problem is:

```text
Machine measurements
       ↓
Failure Type
```

so Logistic Regression can act as a classification algorithm.

### Why StandardScaler?

The Logistic Regression model uses:

```text
StandardScaler
```

before training.

Why?

Your features have very different scales.

For example:

```text
Temperature → around 300
Torque      → around 50
Tool wear   → around 100
Power       → thousands
```

A scaler transforms the numerical features to a more comparable scale.

Conceptually:

```text
Raw features
     ↓
Standardization
     ↓
Logistic Regression
```

The `Pipeline` ensures this happens automatically in the correct order.

---

# Step 9: Why `class_weight='balanced'`?

Some of your failure types are rare.

`class_weight='balanced'` tells the model:

> Pay more attention to classes that have fewer training examples.

So the model doesn't treat:

```text
No Failure
```

as overwhelmingly more important than a rare failure.

This is another approach to dealing with imbalance.

---

# Step 10: Random Forest

The second candidate is **Random Forest**.

Instead of building one decision tree, Random Forest builds many trees.

Conceptually:

```text
                Data
                  ↓
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     Tree 1     Tree 2     Tree 3
       ↓          ↓          ↓
       └──────────┼──────────┘
                  ↓
             Final prediction
```

Your setting:

```text
n_estimators = 100
```

means:

> Build 100 decision trees.

Random Forest is useful because it can capture **non-linear relationships** between machine measurements and failures.

---

# Step 11: XGBoost

The third model is **XGBoost**.

XGBoost is a gradient-boosting algorithm.

Instead of building independent trees like Random Forest, boosting builds trees sequentially, where later trees try to improve the mistakes made by previous trees.

Conceptually:

```text
Tree 1
  ↓
Find mistakes
  ↓
Tree 2 tries to improve them
  ↓
Tree 3 improves further
  ↓
...
  ↓
Final model
```

Your code specifies:

```text
100 trees
```

through `n_estimators`.

`mlogloss` is used as the evaluation/loss metric during the model's training process for the multiclass problem.

---

# Step 12: LightGBM

The fourth candidate is **LightGBM**.

It is another gradient-boosting algorithm, designed to be efficient and powerful on tabular datasets.

Conceptually:

```text
Training data
     ↓
Gradient boosting
     ↓
Many decision trees
     ↓
Final prediction
```

So now you're comparing four different approaches:

| Model               | Main idea                       |
| ------------------- | ------------------------------- |
| Logistic Regression | Linear classification           |
| Random Forest       | Many independent decision trees |
| XGBoost             | Sequential boosting             |
| LightGBM            | Efficient gradient boosting     |

---

# Step 13: Loop through all models

Instead of writing the same training code four times, you put the models into a collection.

Then you loop through:

```text
Logistic Regression
       ↓
train → predict → evaluate → save results

Random Forest
       ↓
train → predict → evaluate → save results

XGBoost
       ↓
train → predict → evaluate → save results

LightGBM
       ↓
train → predict → evaluate → save results
```

This is why the `for` loop is used.

---

# Step 14: Start an MLflow experiment

For every model, you create an MLflow **run**.

Think of a run as:

> One experiment where I trained one model.

For example:

```text
MLflow Experiment
       │
       ├── Run 1 → Logistic Regression
       ├── Run 2 → Random Forest
       ├── Run 3 → XGBoost
       └── Run 4 → LightGBM
```

This makes model comparison much easier.

---

# Step 15: Train the model

The model is trained using:

```text
X_res
y_res
```

Remember where those came from?

Earlier:

```text
X_train + y_train
       ↓
     SMOTE
       ↓
X_res + y_res
```

So the model is trained on the **SMOTE-balanced training data**.

---

# Step 16: Make predictions

After training, the model has never seen the validation data.

Now give it:

```text
X_val
```

and ask:

> What failure type do you predict?

The result is:

```text
y_pred
```

So:

```text
X_val
  ↓
Trained model
  ↓
y_pred
```

Then we compare:

```text
y_pred
vs
y_val
```

because `y_val` contains the actual answers.

---

# Step 17: Calculate Accuracy

Now calculate:

> How many predictions were correct?

This gives you:

```text
accuracy
```

---

# Step 18: Calculate Macro F1

Calculate F1 for every class:

```text
No Failure → F1
TWF        → F1
HDF        → F1
PWF        → F1
OSF        → F1
```

Then average them equally.

That gives:

```text
macro_f1
```

This is one of your most important metrics.

---

# Step 19: Calculate Weighted F1

Again calculate F1 for each class, but this time give larger classes more influence.

This gives:

```text
weighted_f1
```

---

# Step 20: Calculate F1 for each individual class

The code also calculates:

```text
per_class
```

This is extremely useful.

Instead of only knowing:

```text
Overall Macro F1 = 0.82
```

you can see:

```text
No Failure → 0.96
TWF        → 0.81
HDF        → 0.78
PWF        → 0.70
OSF        → 0.65
```

Now you know **which failure types the model struggles with**.

This is particularly valuable in predictive maintenance because a model with good overall performance might still perform poorly on a particular dangerous or rare failure.

---

# Step 21: Store results in MLflow

For each model, MLflow records things such as:

```text
Model = Random Forest
Accuracy = 0.91
Macro F1 = 0.82
Weighted F1 = 0.90
```

It also records the F1 score for individual failure types.

So MLflow becomes your experiment history.

---

# Step 22: Save the trained model

The code also tells MLflow to save the trained model.

This is important because you don't just want to know:

> "XGBoost scored 0.87."

You also want to keep the actual trained XGBoost model so that you can use it later for predictions.

The `input_example` gives MLflow an example of what kind of input the model expects.

---

# Step 23: Store results in a Python dictionary

Finally, you maintain a `results` dictionary.

Conceptually:

```text
results

Logistic Regression
    ↓
    macro_f1
    weighted_f1
    accuracy

Random Forest
    ↓
    macro_f1
    weighted_f1
    accuracy

XGBoost
    ↓
    macro_f1
    weighted_f1
    accuracy

LightGBM
    ↓
    macro_f1
    weighted_f1
    accuracy
```

This makes it easy to compare the models later.

---

# The complete mental flow

This is what you should understand **before writing the code**:

```text
                 TRAINING DATA
                       │
                       ↓
                Feature Engineering
                       │
                       ↓
                Encode Type
                       │
                       ↓
                 Select X and y
                       │
                       ↓
                  Train / Val Split
                       │
              ┌────────┴────────┐
              ↓                 ↓
         Training            Validation
              ↓                 │
            SMOTE               │
              ↓                 │
       Balanced Training        │
              │                 │
      ┌───────┼────────┬────────┤
      ↓       ↓        ↓        ↓
     LR      RF      XGBoost   LightGBM
      ↓       ↓        ↓        ↓
      └───────┼────────┴────────┘
              ↓
          Predictions
              ↓
       Compare with y_val
              ↓
      ┌───────┼─────────────┐
      ↓       ↓             ↓
   Accuracy Macro F1    Weighted F1
              ↓
       Per-class F1
              ↓
            MLflow
              ↓
       Compare models
              ↓
         Select best
```

# What the learner should be able to say

Before coding, the learner should be able to explain:

> **"I have balanced training data and untouched validation data. I want to compare several classification algorithms. For each model, I train it on the balanced training data, predict the validation data, calculate accuracy and different types of F1 scores, record those results in MLflow, and save the trained model. Finally, I compare the models and select the one that performs best, paying particular attention to Macro F1 because my failure classes are imbalanced."**

That's the **reasoning that should come first**. The Python code is simply implementing those decisions.
