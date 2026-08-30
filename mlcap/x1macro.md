**Macro F1** is a way to score a multi-class model that treats every class as equally important, regardless of how many examples that class has.

To get there, break it into three layers:

### 1. Precision and Recall (per class)
For one specific class — say **TWF**:
- **Precision** = "Of all the times the model predicted TWF, how often was it actually TWF?"
- **Recall** = "Of all the real TWF cases, how many did the model actually catch?"

### 2. F1 score (per class)
F1 combines precision and recall into one number — specifically their *harmonic mean*, which punishes a big gap between the two more than a simple average would. A model with precision=1.0 but recall=0.0 (never predicts TWF at all, but on the rare occasion it does, it's right) still gets F1=0, not 0.5. That harshness is intentional — it stops you from gaming the score by being cautious in one direction.

### 3. Macro = average the F1 scores across all classes, unweighted
Once you have an F1 score for each class — No Failure, TWF, HDF, PWF, OSF — **macro F1 just averages those five numbers, giving each class the same vote**, no matter how many rows it has.

```
Macro F1 = (F1_NoFailure + F1_TWF + F1_HDF + F1_PWF + F1_OSF) / 5
```

That's the key detail: **No Failure** might have thousands of rows and **TWF** might have ~30, but in the macro F1 average, they count *equally* — one-fifth each.

### Why this matters for your dataset specifically
Contrast it with accuracy or weighted F1:
- **Accuracy** just counts "correct guesses across all rows" — so the 9,700 "No Failure" rows drown out the 30 "TWF" rows completely. A model that's clueless about every rare failure can still hit 98%+ accuracy.
- **Weighted F1** averages the per-class F1 scores too, but weights each class by how many rows it has — so it still lets the huge "No Failure" class dominate the number, just less bluntly than accuracy does.
- **Macro F1** refuses to let class size matter at all. If the model completely misses TWF (F1=0.0), that zero drags the macro average down hard — even though TWF is a tiny fraction of the data.

That's exactly why, in your notebook, XGBoost's macro F1 of 0.748 is meaningful even while every model scores >98% accuracy: macro F1 is the only one of the three that's actually being honest about how well the model handles the rare failure types — which is the entire point of a predictive maintenance model.
