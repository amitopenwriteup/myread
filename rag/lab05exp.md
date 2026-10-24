# Lab 4 (Condensed): Evaluating the RAG App

This is the same Lab 4 as before, so here is a shorter version built around one worked example, which is easier to explain to others.

## The core idea

> RAG has two parts: **retrieval** (find the text) and **generation** (write the answer). A bad answer can come from either, so we score each part separately.

Think of an open-book exam. A student fails either because they opened the **wrong page** (retrieval) or **misread the right page** (generation).

---

## Step 1: The free check (no AI judge)

Each question in `rag_eval.json` lists `expected_sources`, so we can check retrieval with plain Python.

```python
def retrieval_check(row):
    expected = set(row["expected_sources"])
    if not expected:                       # q7: nothing should be found
        return pd.Series({"hit": None, "all_found": None, "rr": None})
    retrieved = row["sources"]
    hit = any(s in expected for s in retrieved)
    all_found = expected.issubset(set(retrieved))
    rr = next((1 / (i + 1) for i, s in enumerate(retrieved) if s in expected), 0.0)
    return pd.Series({"hit": hit, "all_found": all_found, "rr": rr})
```

| Metric | Question it answers | Example |
|---|---|---|
| **hit** | Did we get at least one correct file? | Expected `policies.md`, got it: True |
| **all_found** | Did we get every needed file? | q9 needs both `products.md` and `policies.md` |
| **rr** | How high did the first correct file rank? | Rank 1 = 1.0, rank 2 = 0.5, rank 3 = 0.33 |

The average of `rr` across questions is **MRR**. It is fast, free and deterministic, so always run it first.

**Limit:** it checks the *file*, not the *chunk*, so it can say "hit" even when the wrong paragraph was retrieved.

---

## Step 2: The AI judge (RAGAS)

```python
judge_llm = LangchainLLMWrapper(ChatOllama(model="llama3.1:8b", temperature=0))
RUN_CONFIG = RunConfig(timeout=300, max_workers=2)
```

A bigger model (8B) grades the smaller one (3B). Low `max_workers` keeps the local machine from choking.

RAGAS needs four fields per question:

```python
{"user_input": question,
 "retrieved_contexts": contexts,   # what Chroma returned
 "response": answer,               # what the model said
 "reference": ground_truth}        # your hand-written answer
```

Then one call scores everything:

```python
result = evaluate(dataset=to_ragas(df), metrics=all_metrics(),
                  llm=judge_llm, embeddings=judge_emb, run_config=RUN_CONFIG)
```

---

## Step 3: The six metrics, as plain questions

| Metric | Plain question | Checks |
|---|---|---|
| **Context Recall** | Did we fetch all the facts needed? | Retrieval |
| **Context Precision** | Are the good chunks at the top? | Retrieval |
| **Context Relevance** | Is each chunk on-topic? | Retrieval |
| **Faithfulness** | Is every claim in the answer backed by the context? | Generation |
| **Answer Relevance** | Does the answer address the question? | Generation |
| **Answer Correctness** | Does it match the ground truth? | Overall |

Rough score bands: **0.85+** strong, **0.6 to 0.85** acceptable, **0.3 to 0.6** weak, **below 0.3** failing.

---

## Step 4: Worked example (q4)

> **Question:** "How long is the warranty on the NovaAir purifier and does it cover filters?"
> **Reference:** "3-year warranty. Filters are not covered."

| Scenario | What happens | Scores | Diagnosis |
|---|---|---|---|
| **Healthy** | Retrieves the warranty chunk, answers correctly | All high | Good |
| **Loose prompt** | Right chunk retrieved, but the model adds "and it covers accidental damage" | Recall high, **Faithfulness low** | Fix the prompt |
| **Tiny chunks, k=1** | Retrieves "3-year warranty" with no heading, model says "I don't know" | **Recall low**, Faithfulness high, Correctness low | Fix retrieval |

The last row is the important trap: **faithful but wrong**. The model was honest, but it never saw the facts. Faithfulness alone would call this a pass.

---

## Step 5: The diagnosis flow

```
Context Recall low?    → fix retrieval (k, chunk size, embeddings)
Faithfulness low?      → fix the prompt, lower temperature
Answer Relevance low?  → fix prompt or output format
Correctness low?       → check the reference or reasoning (e.g. q9 arithmetic)
```

Always check retrieval first. If the facts were never fetched, the generator cannot be blamed.

The report-reading script automates part of this:

```python
low = [c for c in score_cols if pd.notna(r[c]) and r[c] < 0.6]
print("NEEDS ATTENTION (<0.6):", low if low else "none")
```

---

## Step 6: The fallback judge

If RAGAS breaks, a prompt does the same job:

```python
FAITH_PROMPT = ChatPromptTemplate.from_template(
"""You are a strict fact-checker.
Context: {context}
Answer: {answer}
List each factual claim. Say SUPPORTED or UNSUPPORTED.
Finish with: SCORE = <supported>/<total>."""
)
```

This is essentially what **Faithfulness** does internally: split into claims, check each, compute supported ÷ total. There is no magic.

---

## Three warnings to mention

1. **Judges are LLMs**, so scores vary between runs and between judge models. Never compare across judges.
2. **Correct refusals can score low.** "I don't know" on q7 is the right behaviour but may get low Answer Relevance. Fix: add `"should_refuse": true` to the dataset and score those rows with a refusal check.
3. **`NaN` scores** usually mean a timeout or malformed JSON from the local judge. Lower `max_workers`, raise `timeout`, or run once beforehand and load the cache with `--cached`.

---

## 30-second script

> "RAG can fail in two places, so we measure both. First a free check: did we retrieve the expected files, using hit rate and MRR. Then RAGAS has a stronger model score recall, precision, faithfulness, relevance and correctness. We follow a flow: low recall means fix retrieval, low faithfulness means fix the prompt. The key trap is an answer that is faithful but wrong, because the facts were never retrieved."

---

Ready for **Lab 5** (compare, diagnose and improve) whenever you paste it.
