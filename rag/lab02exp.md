# Lab 2 Explained: Scoring with Traditional NLP Metrics

## The one-line story

> "Lab 1 made the AI write things. Lab 2 gives those outputs **numbers**, and then shows why those numbers can be misleading."

---

## 1. Big picture

| Task type | Example | Metrics used |
|---|---|---|
| **Closed answer** (one right answer) | Intent classification | Accuracy, Exact Match |
| **Open-ended** (many right answers) | Marketing captions | BLEU, ROUGE, METEOR, BERTScore |

```
Lab 1 CSVs  →  score them  →  read the numbers  →  improve prompt  →  re-score
```

## 2. The 5 files

| File | Purpose |
|---|---|
| `lab2_metrics_common.py` | Loads all metrics, defines one `score_generation()` helper |
| `lab2_closed_answer.py` | Accuracy + Exact Match for intents |
| `lab2_generation_metrics.py` | BLEU/ROUGE/METEOR/BERTScore on captions |
| `lab2_metric_disagreement.py` | Hand-made examples showing where metrics fail |
| `lab2_prompt_v2.py` | Better prompt, re-score, rule checks |

---

## 3. `lab2_metrics_common.py`: the scoring toolbox

```python
import evaluate
bleu = evaluate.load("bleu")
rouge = evaluate.load("rouge")
meteor = evaluate.load("meteor")
bertscore = evaluate.load("bertscore")
```

Loads four metric tools from Hugging Face's `evaluate` library. First run downloads files, so do it with internet beforehand.

```python
def score_generation(preds, refs):
    bs = bertscore.compute(predictions=preds, references=refs,
                           lang="en", model_type="distilbert-base-uncased")
    for i, (p, r) in enumerate(zip(preds, refs)):
        b = bleu.compute(predictions=[p], references=[[r]], smooth=True)["bleu"]
        rg = rouge.compute(predictions=[p], references=[r])
        m = meteor.compute(predictions=[p], references=[r])["meteor"]
        rows.append({...})
    return pd.DataFrame(rows)
```

- Takes a list of **predictions** (AI text) and **references** (human text).
- Scores **each pair** and returns a table, one row per sample.
- BERTScore runs **once as a batch** because loading its model is slow.
- `smooth=True` for BLEU prevents a score of 0 on short sentences when no 4-word phrase matches.
- BLEU wants `[[r]]` (a list of lists) because it supports multiple references.

---

## 4. `lab2_closed_answer.py`: Accuracy and Exact Match

```python
def normalise(s):
    return str(s).strip().lower().strip(".!\"' ")
```

Cleans a string: remove spaces, lowercase, strip punctuation. So `"Returns."` becomes `returns`.

```python
em_raw  = exact.compute(predictions=preds, references=refs)["exact_match"]
em_norm = exact.compute(predictions=[normalise(p) for p in preds], references=refs)["exact_match"]
accuracy = np.mean([normalise(p) == r for p, r in zip(preds, refs)])
```

| Score | Meaning |
|---|---|
| **EM raw** | Strict: output must match the label character for character |
| **EM normalised** | Same, after cleaning case/punctuation |
| **Accuracy** | Share of correct labels (same number here, just the usual name for classification) |

### How to read the results

| Observation | Meaning |
|---|---|
| Raw EM **much lower** than normalised | Model understands the task but formats badly. Fix the **prompt/parser** |
| Both low | Real **classification errors**. Look at what gets confused (e.g. warranty vs returns) |

Tip: for more detail, use per-class precision/recall (`sklearn.metrics.classification_report`).

---

## 5. `lab2_generation_metrics.py`: scoring captions

```python
df = pd.read_csv("outputs/lab1_marketing_v1.csv")
scores_v1 = score_generation(df["prediction"].tolist(), df["reference"].tolist())
report_v1 = pd.concat([df[["id", "prediction"]], scores_v1], axis=1)
```

Loads Lab 1's outputs, scores each caption against its reference, and prints per-sample and mean scores.

### What each metric means (simple)

| Metric | Plain-English meaning | Typical value here |
|---|---|---|
| **BLEU** | How many of the AI's word-groups appear in the reference | Often below 0.2, even for good text |
| **ROUGE-1** | How many reference words the AI reused | Above ~0.5 = lots of word reuse |
| **ROUGE-2** | Same, for two-word pairs | Lower than ROUGE-1 |
| **ROUGE-L** | Longest shared word sequence (respects order) | Between the two |
| **METEOR** | Like BLEU but accepts stems and synonyms | More forgiving |
| **BERTScore F1** | Compares **meaning** using embeddings | Usually 0.85 to 0.95 for any same-topic text |

**Warning:** BERTScore looks high for almost everything, so compare *differences between systems*, not the absolute number. (`rescale_with_baseline=True` spreads scores out.)

---

## 6. `lab2_metric_disagreement.py`: the star demo

One reference, four hand-written candidates:

```python
reference = "Meet the NovaBrew Smart Kettle: control it from your phone and keep your tea warm for 2 hours."
```

| Candidate | Text idea | What happens | Lesson |
|---|---|---|---|
| **A. Near copy** | Reference + "!" | All metrics very high | Metrics reward copying |
| **B. Good paraphrase** | "Run the NovaBrew from your phone... two hours." | BLEU/ROUGE low, METEOR/BERTScore higher | Good text, but punished by word-matching |
| **C. Fluent but WRONG** | Same sentence, "**8 hours**" instead of 2 | All metrics **still high** | Metrics cannot detect false facts |
| **D. Off-topic** | Talks about the air fryer | BERTScore still fairly high, others low | Same domain fools embeddings |

### The key message

> **These metrics measure similarity to a reference, not truth, brand compliance, or usefulness.**

Candidate C is the dangerous one: it reads perfectly and scores high, but tells customers the wrong thing. That is why we need **faithfulness/correctness metrics** (coming in later labs).

Use these metrics as **regression signals** ("did we get worse?"), not quality guarantees.

### Limitations to mention

- Many valid outputs exist, but there is only one reference
- Cannot verify facts
- Ignore tone, style, safety
- Noisy on short text
- References are expensive to write by hand

---

## 7. `lab2_prompt_v2.py`: improve and re-measure

Two hypotheses:

1. Temperature 0.7 causes too much variation, so try **0.2**
2. Model ignores our style, so add **few-shot examples** (show approved copy)

```python
PROMPT_V2 = ChatPromptTemplate.from_messages([
    ("system",
     "... Mention the exact product name and one or two concrete specs from the brief. "
     "Do NOT invent specs or make health claims. Reply with the copy only.\n\n"
     "Examples of approved copy:\n"
     "- Dinner made easy: the NovaChef 5L Air Fryer packs 8 one-touch presets...\n"
     "- Launch week: save 15% on the NovaChef 5L Air Fryer"),
    ("human", "Write a {format} for this brief:\n{brief}"),
])
```

The `generate()` helper loops over all briefs, then:

```python
preds_v2 = generate(PROMPT_V2, temperature=0.2)
scores_v1 = score_generation(df_v1["prediction"].tolist(), refs)
scores_v2 = score_generation(preds_v2, refs)
```

Scores both versions and builds a comparison table with a `delta` column (v2 minus v1).

### Data leakage trap

The few-shot examples are the references for **m3 and m4**. So v2 can just copy them and get inflated scores.

```python
keep = [0, 1, 4]   # m1, m2, m5 only
honest = pd.DataFrame({"v1": scores_v1.iloc[keep].mean(), "v2": scores_v2.iloc[keep].mean()})
```

The script prints a second, **honest comparison** excluding m3 and m4. In real projects, use examples that are *not* in your test set.

### Rule checks again

```python
rules["within_limit"] = rules["pred_words"] <= rules["max_words"]
rules["says_cheap"] = ...contains("cheap")
rules["health_claim"] = ...contains(r"\b(cure|treat|prevent|allerg|flu|asthma)")
```

**Why?** A prompt change can raise similarity scores while breaking a rule (for example, going over the word limit). Always read the metric table and the rule table together.

---

## 8. Discussion answers

1. **How to catch candidate C?** Add a fact-checking step: compare the claim against the product docs (faithfulness check, LLM-as-judge, or RAG-based verification), plus human review.
2. **One dashboard metric?** Something like BERTScore or ROUGE-L as a trend signal, but humans are still needed for tone, truth and brand fit.
3. **0.03 improvement on 5 examples?** Not trustworthy. The sample is tiny and generation is random, so it is likely noise. Use more data and repeat runs.

---

## 9. Learner challenge

Add a candidate that scores **higher on BLEU than B** but is **worse copy**, for example a clumsy sentence that copies many reference words in a stiff way. Lesson: **optimising for a metric is not the same as optimising for quality** (Goodhart's law).

---

## 10. Common errors

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: lab1_common` | Run Lab 1 first; run from `novahome_eval/` |
| `FileNotFoundError: outputs/lab1_marketing_v1.csv` | Run `python lab1_generate_marketing.py` |
| `LookupError` wordnet/punkt_tab | Re-run once with internet to download NLTK data |
| `evaluate.load` hangs | Needs internet on first use; pre-download |
| `SyntaxError` after paste | Closing `EOF` must be alone on its line |

---

## 11. 30-second summary you can say aloud

> "We scored the AI's outputs with standard metrics. For intents, accuracy and exact match work because there is one right answer. For captions, we used BLEU, ROUGE, METEOR and BERTScore, which compare against a reference. Then we proved they can be fooled: a fluent sentence with a wrong fact still scores high. So these metrics are good for tracking changes, but not for deciding quality. We improved the prompt, re-measured honestly by excluding leaked examples, and checked the rules too."

---

## Checkpoint

- [ ] 5 Lab 2 files exist
- [ ] You have per-sample and mean scores for v1
- [ ] You can explain why candidate C is dangerous
- [ ] You compared v1 vs v2 and checked rules

Want me to explain Lab 3 (Build the RAG pipeline) next in the same style?
