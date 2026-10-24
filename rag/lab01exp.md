# Lab 1 Explained: Build the Non-RAG Marketing Assistant

## The one-line story

> "We build a small AI that writes marketing captions, then check whether its output is actually good."

No document search (RAG) here. The model writes only from its own knowledge plus the short brief you give it.

---

## 1. Business context (explain this first)

NovaHome's marketing team wants an assistant to draft **social captions** and **email subject lines**.

| What the business wants | What we must check |
|---|---|
| Save copywriter time | Output is usable with light editing |
| Stay on-brand | Right tone, word limit, vocabulary |
| Never mislead customers | No invented specs, no health claims |

**Key idea:** Before picking any metric, decide what "good" means. Metrics come after.

---

## 2. The 4 files and what each does

| File | Simple purpose |
|---|---|
| `lab1_common.py` | The "engine": prompts and a helper to build the chain |
| `lab1_generate_marketing.py` | Asks the AI to write captions |
| `lab1_rule_checks.py` | Automatic checks (word limit, banned words) |
| `lab1_intent.py` | A second task: classify customer messages |

Flow:

```
common.py  →  generate_marketing.py  →  rule_checks.py
          ↘  intent.py
```

---

## 3. `lab1_common.py`: the engine

```python
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
```

Three building blocks: **prompt**, **model**, **parser**.

### The marketing prompt

```python
PROMPT_V1 = ChatPromptTemplate.from_messages([
    ("system",
     "You are a marketing copywriter for NovaHome Appliances. "
     "Write in a friendly, confident tone. "
     "Never exceed {max_words} words. "
     "Do not make medical or health claims. "
     "Reply with the copy only, no quotation marks and no explanations."),
    ("human", "Write a {format} for this brief:\n{brief}"),
])
```

- **system message** = the rules and role you give the AI.
- **human message** = the actual task.
- `{max_words}`, `{format}`, `{brief}` are **blanks** filled in per item. It is like a mail-merge template.
- "Reply with the copy only" stops the model from adding chatty extras like "Sure! Here's your caption:".

### The intent prompt

```python
INTENT_PROMPT = ChatPromptTemplate.from_messages([
    ("system",
     "Classify the customer message into exactly one of: product_info, returns, shipping, warranty. "
     "Reply with the label only, in lowercase, with no punctuation."),
    ("human", "{text}"),
])
```

Same idea, but the AI must answer with **one label only**. The strict wording matters because we will compare the answer to the correct label.

### The helper function

```python
LABELS = ["product_info", "returns", "shipping", "warranty"]

def make_chain(prompt, temperature=0.7, model="llama3.2:3b"):
    llm = ChatOllama(model=model, temperature=temperature)
    return prompt | llm | StrOutputParser()
```

The important line is:

```python
prompt | llm | StrOutputParser()
```

The `|` (pipe) works like an assembly line:

```
Fill the template  →  Send to model  →  Convert reply to plain text
     (prompt)            (llm)              (StrOutputParser)
```

| Part | Role |
|---|---|
| `ChatPromptTemplate` | Fills in the blanks so the prompt is reusable and versionable |
| `ChatOllama` | The local model |
| `StrOutputParser` | Turns the model's message object into a normal string |

**Temperature** = creativity dial:
- `0.7` for captions (some variety is fine)
- `0.0` for classification (we want the same answer every time)

---

## 4. `lab1_generate_marketing.py`: generate captions

```python
import json
import pandas as pd
from lab1_common import PROMPT_V1, make_chain

marketing_eval = json.load(open("data/marketing_eval.json"))
chain_v1 = make_chain(PROMPT_V1, temperature=0.7)
```

Loads the 5 test briefs and builds the chain.

```python
rows = []
for item in marketing_eval:
    out = chain_v1.invoke({
        "format": item["format"],
        "max_words": item["max_words"],
        "brief": item["brief"],
    }).strip()
    rows.append({**item, "prediction": out})
```

For each brief:
1. Fill the template blanks.
2. Call the model with `.invoke()`.
3. `.strip()` removes extra spaces or newlines.
4. Save the original item **plus** the new `prediction`. (`{**item, ...}` copies all fields and adds one.)

```python
df_v1 = pd.DataFrame(rows)
df_v1["pred_words"] = df_v1["prediction"].str.split().str.len()
df_v1.to_csv("outputs/lab1_marketing_v1.csv", index=False)
```

- Puts the results into a table.
- Counts words by splitting on spaces.
- Saves to CSV so Lab 2 can score it.

```python
for r in rows:
    print(f"[{r['id']}] {r['format']}")
    print("  REF :", r["reference"])
    print("  PRED:", r["prediction"], "\n")
```

Prints reference (human-written example) next to prediction (AI output) so you can compare by eye.

**Run:** `python lab1_generate_marketing.py`

---

## 5. Read outputs before any metric

Fill this table by hand:

| id | On-brand? | Within word limit? | Invented facts? | Would you publish it? |
|---|---|---|---|---|
| m1 | | | | |
| m2 | | | | |
| ... | | | | |

**Why?** Human judgment is the ground truth. Automated metrics are only attempts to imitate it.

---

## 6. `lab1_rule_checks.py`: automatic rule checks

```python
df = pd.read_csv("outputs/lab1_marketing_v1.csv")

df["within_limit"] = df["pred_words"] <= df["max_words"]
df["says_cheap"] = df["prediction"].str.contains("cheap", case=False)
df["health_claim"] = df["prediction"].str.contains(
    r"\b(cure|treat|prevent|allerg|flu|asthma)", case=False, regex=True)
```

Three simple checks, each giving True/False:

| Check | What it tests |
|---|---|
| `within_limit` | Is the word count within the allowed maximum? |
| `says_cheap` | Does it use the banned word "cheap"? |
| `health_claim` | Does it contain risky health words? (regex pattern) |

Notes:
- `case=False` ignores capital letters.
- The `\b` in the regex means "start of a word".
- `allerg` catches both "allergy" and "allergen".

```python
checks = df[["id", "pred_words", "within_limit", "says_cheap", "health_claim"]]
print(checks.to_string(index=False))
checks.to_csv("outputs/lab1_marketing_v1_checks.csv", index=False)
```

Shows a clean summary table and saves it.

**Teaching point:** Many important quality rules are **rule-based checks**, not similarity scores. Real evaluation mixes both.

---

## 7. `lab1_intent.py`: classification (the closed-answer task)

```python
intent_eval = json.load(open("data/intent_eval.json"))
intent_chain = make_chain(INTENT_PROMPT, temperature=0.0)
```

Loads 10 customer messages, with temperature 0 for consistency.

```python
rows = []
for item in intent_eval:
    pred = intent_chain.invoke({"text": item["text"]}).strip()
    rows.append({**item, "prediction": pred})
```

For each message, the AI predicts a label, and we store it beside the correct label.

```python
df_intent = pd.DataFrame(rows)
df_intent.to_csv("outputs/lab1_intent.csv", index=False)

print(df_intent.to_string(index=False))
print("\nMismatches (raw string comparison):")
print(df_intent[df_intent["prediction"] != df_intent["label"]].to_string(index=False))
```

Prints everything, then shows only the rows where `prediction != label`.

### The important lesson

When something mismatches, ask: **is the model wrong, or is the format wrong?**

| Case | Example | Type of failure | How to fix |
|---|---|---|---|
| Wrong meaning | Predicts `shipping` for a return question | **Model error** | Better prompt or model |
| Right meaning, wrong format | `Returns.` instead of `returns` | **Format error** | Tighten the prompt or clean the output |

They need different fixes, so evaluation must separate them.

---

## 8. Why two different tasks?

| | Marketing captions | Intent classification |
|---|---|---|
| Correct answers | Many | Exactly one |
| Best evaluation | Human review, rules, similarity metrics | Accuracy / Exact Match |
| Temperature | 0.7 | 0.0 |

This explains discussion question 2: **Exact Match works for intents but not captions**, because many different captions can all be good.

---

## 9. Discussion answers (ready to say out loud)

1. **Ambiguous messages:** e.g. "Can I get a refund if I opened the box?" could be `returns` or `product_info`. A human labeller might disagree.
2. **Exact Match for intents only:** intents have one correct label; captions have many valid wordings.
3. **"The label is returns":** Exact Match says it **failed** (strings differ). A looser check, such as "does the output contain the right label?", says it **passed**. So the metric you choose changes the verdict.

---

## 10. How to explain it in 30 seconds

> "We built a small pipeline: a prompt template, a local model and an output parser, chained together with `|`. We ran it on 5 marketing briefs and saved the outputs. Then we checked them in two ways: by eye, and with simple rules like word limit and banned words. We also built a second task, intent classification, where there is one correct answer so we can use accuracy. The big lesson is that you should look at the outputs and define 'good' before picking metrics."

---

## Common errors

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: lab1_common` | Run from inside `novahome_eval/`, and make sure the first `tee` block ran |
| `FileNotFoundError: data/marketing_eval.json` | Redo Lab 0 dataset steps |
| `Connection refused` | Start Ollama (`ollama serve`) |
| `SyntaxError` after paste | Closing `EOF` must be alone on its line |

---

## Learner challenge

Add a tricky 6th brief to `marketing_eval.json`, for example: *"NovaAir Purifier: helps people with allergies breathe easier."* This tempts the model to make a health claim. **Predict** what happens, then run the generator and rule check to see whether the `health_claim` check catches it.

---

Want me to explain Lab 2 (scoring with ROUGE, BLEU, METEOR, BERTScore) in the same style?
