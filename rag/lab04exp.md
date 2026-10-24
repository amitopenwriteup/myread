# Lab 4 Explained: Evaluating the RAG Application

## The one-line story

> "Lab 3 built RAG. Lab 4 **measures** it, and more importantly tells you **which part broke**: retrieval or generation."

---

## 1. Why RAG needs more than "is the answer right?"

A wrong answer can come from two different places:

```
Question ─► [Retrieval] ─► Contexts ─► [Generation] ─► Answer
              ▲                           ▲
   Context Precision / Recall        Faithfulness / Answer Relevance /
   Context Relevance                 Answer Correctness
```

| If this is bad... | ...the problem is probably in |
|---|---|
| Context Recall / Context Relevance | **Retrieval** (chunking, embeddings, `k`) |
| Context Precision | **Ranking/noise** in retrieved chunks |
| Faithfulness | **Generation** going beyond the context |
| Answer Relevance | **Prompt** or model not addressing the question |
| Answer Correctness | Any of the above; use other metrics to locate it |

**Analogy:** an open-book exam. If the student fails, was it because they opened the wrong page (retrieval) or misread the right page (generation)? Different fix for each.

---

## 2. The files

| File | Purpose |
|---|---|
| `lab4_01_generate_and_retrieval_check.py` | Generate answers + free retrieval check (no judge) |
| `lab4_common.py` | RAGAS setup shared by scripts |
| `lab4_02_ragas_eval.py` | Run RAGAS metrics, save report |
| `lab4_03_read_report.py` | Read report row by row, flag weak scores |
| `lab4_04_failure_cases.py` | Two deliberate failure demos |
| `lab4_05_fallback_judge.py` | Hand-written judge if RAGAS fails |

Order of work: **cheap deterministic check first, expensive LLM judge second.**

---

## 3. Step 4.1 and 4.2: generate and do a retrieval sanity check

```python
for item in rag_eval:
    out = rag_answer(item["question"], k=3)
    records.append({**item, "answer": out["answer"],
                    "contexts": out["contexts"], "sources": out["sources"]})
df = pd.DataFrame(records)
```

Runs all 9 questions through the RAG app and keeps answer, contexts and sources next to the reference data.

```python
def retrieval_check(row):
    expected = set(row["expected_sources"])
    if not expected:                      # q7: nothing should be found
        return pd.Series({"hit": None, "all_found": None, "rr": None})
    retrieved = row["sources"]
    hit = any(s in expected for s in retrieved)
    all_found = expected.issubset(set(retrieved))
    rr = next((1 / (i + 1) for i, s in enumerate(retrieved) if s in expected), 0.0)
    return pd.Series({"hit": hit, "all_found": all_found, "rr": rr})
```

Because `rag_eval.json` has `expected_sources`, we can check retrieval without any AI judge:

| Metric | Plain meaning | Example |
|---|---|---|
| **Hit rate@k** | Did we retrieve at least one correct file? | Expected `policies.md`, got it → hit |
| **All-found rate** | Did we retrieve **all** needed files? | q9 needs `products.md` AND `policies.md` |
| **MRR** (reciprocal rank) | How high did the first correct file rank? | Rank 1 → 1.0, rank 2 → 0.5, rank 3 → 0.33 |

Out-of-scope question q7 returns `None` and is skipped in the averages.

**Caveat:** hitting the right *file* does not prove the right *chunk* was found. RAGAS checks that.

---

## 4. Step 4.3: `lab4_common.py` (RAGAS setup)

```python
judge_llm = LangchainLLMWrapper(ChatOllama(model="llama3.1:8b", temperature=0))
judge_emb = LangchainEmbeddingsWrapper(OllamaEmbeddings(model="nomic-embed-text"))
RUN_CONFIG = RunConfig(timeout=300, max_workers=2)
```

- **Judge LLM** = a model that grades answers. It should be at least as strong as the generator, so we use the 8B model to grade the 3B model.
- **Wrappers** let RAGAS use LangChain models.
- `max_workers=2` keeps parallel calls low because local models are slow.

```python
def to_ragas(df):
    return EvaluationDataset.from_list([
        {"user_input": r["question"],
         "retrieved_contexts": r["contexts"],
         "response": r["answer"],
         "reference": r["reference"]}
        for _, r in df.iterrows()
    ])
```

RAGAS needs four fields:

| RAGAS field | Comes from |
|---|---|
| `user_input` | the question |
| `retrieved_contexts` | chunks from Chroma |
| `response` | the generated answer |
| `reference` | our hand-written ground truth |

`all_metrics()` returns the list: Faithfulness, ResponseRelevancy, Context Precision, Context Recall, Answer Correctness, plus Context Relevance if the installed version has it (the `try/except ImportError` handles that).

---

## 5. Step 4.4: run the evaluation

```python
result = evaluate(dataset=to_ragas(df), metrics=all_metrics(),
                  llm=judge_llm, embeddings=judge_emb, run_config=RUN_CONFIG)
report = result.to_pandas()
report.to_csv(REPORT, index=False)
```

Scores every question on every metric and saves a CSV.

**Cached mode:** `python lab4_02_ragas_eval.py --cached` loads the saved CSV instead of re-running. Local judges are slow and sometimes return broken JSON (giving `NaN`), so run once before class and load the cache live.

---

## 6. Step 4.5: what each metric means (the core part)

| Metric | Simple explanation | Low score means |
|---|---|---|
| **Faithfulness** | Split answer into claims; is each claim supported by the context? | Hallucination or outside knowledge |
| **Answer Relevance** | Does the answer actually address the question? | Off-topic or evasive |
| **Context Precision** | Are useful chunks ranked at the top? | Good chunks buried under noise |
| **Context Recall** | Are the facts from the reference present in the contexts? | Key facts were never retrieved |
| **Context Relevance** | Is each retrieved chunk on-topic? | Retrieval brought noise |
| **Answer Correctness** | Does the answer agree with the ground truth? | Wrong or missing facts |

### Score ranges (rules of thumb)

| Range | Meaning | Action |
|---|---|---|
| 0.85 to 1.0 | Strong | Monitor |
| 0.60 to 0.85 | Acceptable | Read the lowest rows |
| 0.30 to 0.60 | Weak | Investigate the component |
| Below 0.30 | Failing | Fix before rollout |

### Caveats to say out loud

- The judge is an LLM, so scores vary between runs and judge models.
- Never compare scores across different judges.
- **A correct refusal can score low.** "I don't know based on the provided documents" on q7 is the *right* behaviour but may get low Answer Relevance. Metrics need business context.

---

## 7. Step 4.6: read the report row by row

```python
low = [c for c in score_cols if pd.notna(r[c]) and r[c] < 0.6]
print("  NEEDS ATTENTION (<0.6):", low if low else "none")
```

For each question it prints the answer, retrieval status, all scores, and which metrics fall below 0.6. It merges the RAGAS report with your retrieval check so you see everything in one place.

### Diagnosis flow (memorise this)

```
Context Recall low?   → yes → Retrieval missed the facts   → tune k / chunking / embeddings
        │ no
Faithfulness low?     → yes → Generator added unsupported claims → stronger grounding prompt, lower temperature
        │ no
Answer Relevance low? → yes → Answer misses the question   → improve prompt / format
        │ no
Correctness low?      → yes → Check reference quality or reasoning (e.g. q9 arithmetic)
```

Always check **retrieval first**, because if the facts were never fetched, the generator cannot be blamed.

---

## 8. Step 4.7: two deliberate failures

### Case 1: good retrieval, bad generation

```python
df_loose = build(("q4", "q7", "q8"), k=3, prompt=LOOSE_PROMPT)
```

Same retrieval, but the **loose prompt** ("one short, confident sentence"). Expect **Faithfulness drops**: the model invents answers instead of refusing.

### Case 2: bad retrieval, good generation

```python
df_bad = build(("q4", "q9"), k=1, chunk_size=120, chunk_overlap=0)
```

Strict prompt, but `k=1` and tiny chunks. Expect **Context Recall is low**, yet Faithfulness may stay **high** because the model honestly says "I don't know" or sticks to the little it has.

> This is the **faithful but wrong** case. Faithfulness alone would call it fine, so you need context and correctness metrics too.

### Summary matrix

| Scenario | Context Recall | Faithfulness | Correctness | Diagnosis |
|---|---|---|---|---|
| Healthy | High | High | High | Good |
| Good retrieval, bad generation | High | **Low** | Low | Fix prompt/model |
| Bad retrieval, honest generation | **Low** | High | Low | Fix retrieval |
| Bad retrieval, hallucinating | **Low** | **Low** | Low | Fix both; add refusal rule |

---

## 9. Step 4.8: fallback judge (build your own)

```python
FAITH_PROMPT = ChatPromptTemplate.from_template(
"""You are a strict fact-checker.
Context: {context}
Answer: {answer}
List each factual claim in the answer. For each, say SUPPORTED or UNSUPPORTED by the context.
Finish with one line: SCORE = <supported claims>/<total claims>."""
)
faith_chain = FAITH_PROMPT | judge | StrOutputParser()
```

This is basically **what Faithfulness does inside RAGAS**: list claims, check each against the context, compute supported/total. Writing it yourself shows there is no magic.

**Discussion:** what biases might this judge have? Verbosity preference, agreeing with fluent text, sensitivity to prompt wording.

---

## 10. Common errors

| Problem | Fix |
|---|---|
| `FileNotFoundError: lab4_rag_run.pkl` | Run `lab4_01_...py` first |
| `ModuleNotFoundError ... vertexai` | `pip install "langchain-community<0.4.2"` |
| `ImportError` from ragas | Version mismatch; check `pip show ragas` and adapt `lab4_common.py` |
| `NaN` scores / timeouts | Lower `max_workers`, raise `timeout`, use stronger judge, or `--cached` |
| `KeyError: 'context_recall'` | Column names differ; print `res2.columns` |
| `model "llama3.1:8b" not found` | `ollama pull llama3.1:8b` |

---

## 11. Learner challenge

Find a row where **Answer Relevance is low but the answer is actually right** (a correct refusal, e.g. q7). Fix: add a flag like `"should_refuse": true` to `rag_eval.json`, and score those rows with a separate rule ("did it refuse?") instead of relevance.

---

## 12. 30-second summary to say aloud

> "RAG can fail in retrieval or in generation, so one score is not enough. First we run a free check: did we retrieve the expected documents, using hit rate and MRR. Then RAGAS uses a stronger judge model to score faithfulness, relevance, context precision, context recall and correctness. We read results row by row and follow a flow: low context recall means fix retrieval, low faithfulness means fix the prompt. We proved it with two failures: a loose prompt hurts faithfulness, while bad chunking hurts recall but can still look faithful. One warning: a correct 'I don't know' can score badly, so metrics always need business context."

---

## Checkpoint

- [ ] Hit rate and MRR computed
- [ ] `outputs/lab4_ragas_report.csv` saved
- [ ] You can explain faithful-but-wrong vs unfaithful-but-right
- [ ] Diagnosis table filled for at least 3 rows

Want me to explain Lab 5 (compare, diagnose and improve) next?
