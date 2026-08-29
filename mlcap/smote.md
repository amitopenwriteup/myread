Let's forget "code" for a second and just think of this as a **kitchen recipe**. Then I'll show you the matching code line right after each step.

## The story version

Imagine you have a big list of machines. Each machine has some numbers written next to it (temperature, speed, torque...) and a label saying what happened to it (Broke down? Fine? Which way did it break?).

Problem: almost every machine says "Fine." Only a tiny handful say "broke down this specific way." If you only show a student 10,000 "Fine" examples and 5 "Broke down" examples, the student will just learn to always guess "Fine" — because that's right almost every time, but useless.

So we do 6 simple things, in order:

---

### 1. Turn words into numbers
The machine only understands numbers, not words like "Low/Medium/High". So we swap:
- Low → 0
- Medium → 1
- High → 2

```python
le = LabelEncoder()
train['Type_enc'] = le.fit_transform(train['Type'])
```
This line looks at the training list, learns "Low=0, Medium=1, High=2", and writes those numbers in a new column.

```python
current['Type_enc'] = le.transform(current['Type'])
stress['Type_enc']  = le.transform(stress['Type'])
```
These two lines just **reuse the same rule** on two other lists. We don't relearn it — we use the exact same 0/1/2 mapping everywhere, so it stays consistent.

---

### 2. Pick "the clues" and "the answer"
We tell the computer: here are 8 columns you're allowed to look at (the clues). And here's the column with the real answer (what actually happened).

```python
FEATURES = ['Type_enc', 'Air temperature', 'Process temperature',
            'Rotational speed', 'Torque', 'Tool wear', 'Power_W', 'Temp_diff']

X = train[FEATURES]      # the clues
y = train['Failure_Type']  # the answer
```

---

### 3. Split into "practice" and "test"
You never want to practice and take the test on the exact same questions — that's cheating, sort of. So we cut the list into two piles:

```python
X_train, X_val, y_train, y_val = train_test_split(
    X, y, test_size=0.20, stratify=y, random_state=42)
```
- 80% → `X_train`/`y_train` → the practice pile
- 20% → `X_val`/`y_val` → the test pile, kept aside, untouched

`stratify=y` just means: make sure both piles have a similar mix of "Fine" vs "Broke down" — not all the rare cases stuck in one pile by accident.

`random_state=42` just means: split it the same way every single time we run this, instead of randomly differently each time.

---

### 4. Fix the "too few rare examples" problem
Here's the actual fix. This tool invents new, realistic-looking practice examples for the rare cases, so the model gets more to learn from.

```python
sm = SMOTE(random_state=42, k_neighbors=3)
```
Think of `SMOTE` as a machine that says: *"give me two real rare examples that look alike, and I'll invent a third one that's a blend of them."*

`k_neighbors=3` just answers: *"how many similar examples should I compare against before blending?"* We use a small number like 3 because some rare classes only have a handful of real examples — you can't compare against 5 similar friends if only 5 exist total.

```python
X_res, y_res = sm.fit_resample(X_train, y_train)
```
This actually does the inventing. It only touches the **practice pile** (`X_train`/`y_train`) — never the test pile. The result, `X_res`/`y_res`, is your new practice pile: same real examples, plus lots of new invented ones for the rare cases.

---

### 5. Check it worked
Just a printout to double check the rare cases actually grew.

```python
print('Class distribution after SMOTE:')
for cls, cnt in sorted(pd.Series(y_res).value_counts().items()):
    print(f'  {cls} {CLASS_NAMES[cls]}: {cnt}')
```

Read this like a sentence: "count how many of each answer type are in the new practice pile, and print it in a readable way." `CLASS_NAMES[cls]` just turns the number back into the word (0 → "No Failure") so the printout is readable instead of showing confusing numbers.

---

## The one thing to remember, in a single sentence

**We invent extra practice examples only for the rare failure types, only in the practice pile — never in the test pile — so the model actually learns to recognize rare failures instead of just always guessing "Fine."**
