This is an **EDA (Exploratory Data Analysis)** code block. Its purpose is to understand the **distribution of machine failures** and see whether **Torque** and **Tool wear** differ between failure types.

Here's what each part does.

### 1. Create 3 plots side by side

```python
fig, axes = plt.subplots(1, 3, figsize=(16, 4))
```

Creates one figure containing **3 plots in a row**:

```text
┌─────────────────┬─────────────────┬─────────────────┐
│ Class            │ Torque          │ Tool Wear       │
│ Distribution     │ by Failure Type │ by Failure Type │
└─────────────────┴─────────────────┴─────────────────┘
```

`axes[0]`, `axes[1]`, and `axes[2]` refer to the three plots.

---

### 2. Count each failure type

```python
vc = train['Failure_Type'].value_counts().sort_index()
```

This counts how many records belong to each `Failure_Type`.

For example:

```text
Failure_Type
0    9000
1     500
2     300
3     150
```

If your `CLASS_NAMES` looks something like:

```python
CLASS_NAMES = {
    0: 'No Failure',
    1: 'TWF',
    2: 'HDF',
    3: 'PWF',
    4: 'OSF',
    5: 'RNF'
}
```

then the numbers are converted into meaningful names.

---

### 3. Plot failure distribution

```python
axes[0].bar(
    [CLASS_NAMES[k] for k in vc.index],
    vc.values,
    color='steelblue'
)
```

This creates a **bar chart** showing how many samples belong to each failure class.

For example:

```text
Samples
9000 | ███████████████████ No Failure
 500 | █                    TWF
 300 | █                    HDF
 150 | █                    PWF
```

This is important because it tells you whether your dataset is **balanced or imbalanced**.

If `No Failure` has 9,000 samples while a failure type has only 50, your model may struggle with that minority class.

---

### 4. Print class percentages

```python
print('Class distribution:')
for k, v in vc.items():
    print(f'  {k} {CLASS_NAMES[k]:<15}: {v:>5} ({v/len(train)*100:.2f}%)')
```

This prints both the **count and percentage**.

For example:

```text
Class distribution:
  0 No Failure     :  9000 (90.00%)
  1 TWF            :   500 (5.00%)
  2 HDF            :   300 (3.00%)
  3 PWF            :   150 (1.50%)
  4 OSF            :    50 (0.50%)
```

The calculation:

```python
v / len(train) * 100
```

means:

> What percentage of the entire training dataset belongs to this class?

---

## 5. Keep only failure records

```python
failures = train[train['Failure_Type'] > 0]
```

This is an important line.

It removes class `0`, which presumably means **No Failure**.

So now:

```text
train
├── No Failure → removed
├── TWF        → kept
├── HDF        → kept
├── PWF        → kept
├── OSF        → kept
└── RNF        → kept
```

The next two plots therefore compare **only actual failures**.

---

## 6. Analyze Torque

```python
for cls_id, cls_name in CLASS_NAMES.items():
    if cls_id == 0: continue
```

Loops through each failure class but skips `0` (No Failure).

Then:

```python
failures[failures['Failure_Type'] == cls_id]['Torque']
```

selects the Torque values belonging to one particular failure type.

For example:

```text
TWF → Torque values
HDF → Torque values
PWF → Torque values
OSF → Torque values
```

Then:

```python
axes[1].hist(..., bins=20)
```

creates a **histogram**.

So the second graph asks:

> **Does Torque have a different distribution for different failure types?**

If the distributions are very different, Torque may be a useful feature for predicting failure type.

genui{"learning_viz":{"type_id":"TORQUE"}}

---

## 7. Analyze Tool Wear

The third section does essentially the same thing:

```python
failures[failures['Failure_Type'] == cls_id]['Tool wear']
```

but instead of Torque, it examines:

```text
Tool wear
```

Then:

```python
axes[2].hist(...)
```

creates histograms for each failure type.

So the question here is:

> **Does Tool Wear have a different distribution for different failure types?**

For example, you might discover:

```text
Tool wear

TWF  █████████████
HDF  ████
PWF  ████████
OSF  ███████████████
```

That could indicate that **Tool wear is associated with certain failure modes**.

---

## 8. Make the plots readable

```python
axes[1].set_title('Torque by Failure Type')
axes[1].legend()
```

adds the title and legend.

Same for Tool Wear:

```python
axes[2].set_title('Tool Wear by Failure Type')
axes[2].legend()
```

---

## 9. Save the EDA figure

```python
plt.tight_layout()
plt.savefig('eda_distributions.png', dpi=120, bbox_inches='tight')
plt.show()
```

`plt.tight_layout()` prevents labels from overlapping.

`plt.savefig(...)` saves the complete figure as:

```text
eda_distributions.png
```

And:

```python
plt.show()
```

displays it in your notebook.

---

# In simple terms

Your entire EDA code is answering **3 questions**:

| Plot                             | Question                                                  |
| -------------------------------- | --------------------------------------------------------- |
| **1. Failure Type Distribution** | How many examples do I have of each class?                |
| **2. Torque by Failure Type**    | Does Torque behave differently for different failures?    |
| **3. Tool Wear by Failure Type** | Does Tool Wear behave differently for different failures? |

### Why this matters for your ML model

You're trying to determine whether your input features contain useful information for predicting:

```text
Machine measurements
       ↓
Torque
Tool wear
Temperature
Speed
etc.
       ↓
ML model
       ↓
Failure Type
```

This EDA helps you understand the dataset **before training the model**. In particular, the first plot identifies **class imbalance**, while the Torque and Tool Wear plots help you visually inspect whether those features have different patterns across failure classes.
