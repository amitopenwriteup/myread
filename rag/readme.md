# Trainer Guide: LLM Evaluation Workshop

A script you can follow section by section. Each lab has the same layout:

- **Say:** talking points, in the order to deliver them
- **Do:** what to run or show
- **Expect:** the output you should see (verify this in your own dry run; small local models vary)
- **Ask:** questions to put to learners
- **Watch for:** common problems and how to handle them
- **Bridge:** one line that leads into the next part

> Expected outputs are typical, not guaranteed. A 3B local model changes between runs. **Always do a full dry run** and note what *your* environment produces, then adjust the "Expect" lines.

---

## 0. Before the Session

### One week before

- [ ] Ask your delivery contact to share the workshop folder, the Lab 0 setup instructions and the `ollama pull` commands with learners in advance (the session brief requires this)
- [ ] Tell learners to complete **Lab 0 sections 0.1 and 0.2** before the session so the live time is not spent on installs

### Day before: full dry run

Run every lab end to end on the machine you will teach from.

```bash
# 1. models
ollama pull llama3.2:3b && ollama pull nomic-embed-text && ollama pull llama3.1:8b

# 2. environment (note the pins)
pip install "langchain-community<0.4.2"      # avoids the ragas/vertexai import error
pip show ragas langchain-community | grep -E "Name|Version"

# 3. create data (Lab 0), then run in order
python lab1_generate_marketing.py && python lab1_rule_checks.py && python lab1_intent.py
python lab2_closed_answer.py && python lab2_generation_metrics.py && python lab2_metric_disagreement.py && python lab2_prompt_v2.py
python lab3_02_embed_store.py
python lab3_05_compare.py && python lab3_06_failures.py
python lab4_01_generate_and_retrieval_check.py && python lab4_02_ragas_eval.py
python lab4_04_failure_cases.py
python lab5_01_standalone_vs_rag.py && python lab5_02_experiments.py
```

### Cache these so you can show results instantly

| Slow step | Cached output | Live command |
|---|---|---|
| RAGAS report | `outputs/lab4_ragas_report.csv` | `python lab4_02_ragas_eval.py --cached` |
| Failure cases | `outputs/lab4_case1_loose_prompt.csv`, `lab4_case2_bad_retrieval.csv` | open the CSVs |
| Standalone vs RAG | `outputs/lab5_standalone_vs_rag.csv` | `python lab5_01_standalone_vs_rag.py --cached` |
| Experiments | `outputs/lab5_experiments.csv` | `python lab5_02_experiments.py --cached` |

### Record your own "golden" results

Keep a note of what you saw for: Lab 1 intent mismatches, Lab 2 candidate B and C scores, Lab 3 failures A to D, Lab 4 lowest-scoring rows, Lab 5 experiment table. You will quote them during discussion.

### Room and tooling checklist

- [ ] Terminal font large; two windows ready (one for running scripts, one for viewing files)
- [ ] `outputs/` folder open so you can show CSVs
- [ ] An AI assistant open in a browser tab (see "Using AI live" below)
- [ ] The deck `LLM_Evaluation_Workshop.pptx` open for Sections 1 and 5
- [ ] Backup: a second copy of the whole folder with all outputs already generated

### Using AI live (required by the session brief)

Weave at least three short AI moments into the day. Suggested ones:

| Moment | What to demonstrate |
|---|---|
| Lab 0 or Lab 3 | Paste an Ollama connection error and ask for likely causes |
| Lab 2 | Ask the assistant to explain why BLEU is 0.0 on a short sentence |
| Lab 4 | If you hit the `langchain_community ... vertexai` import error, paste the traceback and show how the assistant (or a search) finds the version pin. This is a real issue learners may meet. |
| Lab 5 | Paste the experiment table and ask which component to fix first, then **critique its answer** against the playbook |

Always model the habit: *read the suggestion, check it against documentation or output, then run it.*

---

## 1. Section 1: Introduction to LLM Evaluation (20 min)

**Timing:** 10 min talk (slides 1 to 3) + 10 min Lab 0

### 1.1 Opening (3 min)

**Say**
- Introduce yourself and the agenda (slide 2: five sections, one break after Section 2).
- "Today is about *how to judge* an LLM app, not about LangChain syntax. You will see the code, but the thinking matters more."
- "We will evaluate one application twice: first without RAG, then with RAG."

**Ask:** "Who has built an LLM prototype? How did you decide it was good?" Take two answers. Typical answers: "I read some outputs" or "it looked right". Use this to motivate the session.

### 1.2 Why LLM evaluation is different (5 min, slide 3)

**Say**
1. Traditional ML: one label per input, compare to ground truth, compute accuracy or F1.
2. LLM apps: many valid outputs for one input, so a single reference is one opinion.
3. Quality is multi-dimensional: factual accuracy, tone, safety, usefulness, format.
4. Fluent text can be confidently wrong. Fluency is not correctness.
5. Therefore we combine: similarity metrics, correctness checks, grounding checks, rule checks and human review.

**Ask:** "A caption is grammatical, on-brand, and states the wrong price. Which of BLEU, ROUGE or accuracy tells you?" Answer: none reliably. Park this; you will prove it in Lab 2.

### 1.3 Metric recap (2 min, README cheat sheet)

Show the metric cheat sheet in the README. Do not teach each metric yet; say "we will meet each one on real outputs." Highlight the columns *Needs reference?* and *Needs contexts?*.

### 1.4 Roadmap (included in the above)

Draw the two-step plan: **Version 1 (no RAG)** with BLEU, ROUGE, METEOR, BERTScore, Accuracy/EM; **Version 2 (RAG)** with Faithfulness, Answer Relevance, Context Precision, Context Recall, Context Relevance, Answer Correctness.

---

## 2. Lab 0: Setup, Environment Check and Data (10 min, inside Section 1)

**Goal:** everyone can run the scripts and has the same data.

**Say**
- "All names and numbers in our documents are fictional on purpose. A model cannot have memorised them, so correct answers must come from retrieval, and hallucinations are easy to spot."
- "We create files with `tee` and heredocs so everyone gets identical content without copy-paste errors in notebooks."

**Do**
1. Show the environment check block (0.2). Run it. Everyone should see `OK` lines and the `ollama list` output.
2. Run the quick model test (a one-word reply and an embedding length).
3. Run the `mkdir` command, then paste the four document `tee` blocks. Show one file with `cat data/docs/policies.md` to demonstrate what was created.
4. Paste the three dataset `tee` blocks. Run the verification step.

**Expect:** 4 files in `data/docs/`; `intent_eval.json` (10 items), `marketing_eval.json` (5), `rag_eval.json` (9).

**Ask:** the three discussion questions in 0.5. Keep to 3 minutes. Key answers: references capture one phrasing; q7 should be refused as out-of-scope, q8 should refuse the health claim; invented company means retrieval is the only source of truth.

**Watch for**
| Problem | Handling |
|---|---|
| A learner pasted the closing `EOF` with leading spaces | File is cut off; `EOF` must be on its own line starting at column 1. Re-paste. |
| Ollama not running | `ollama serve` in another terminal |
| Pip import errors for ragas | Use the pin in the setup block; this is also a good teaching moment |
| Windows PowerShell users | Use WSL or Git Bash; `tee` heredocs will not work in plain PowerShell |

**Bridge:** "We have a company, documents and test sets. First application: marketing copy with no retrieval."

---

## 3. Section 2: Non-RAG Application (55 min)

### Lab 1: Build the non-RAG app (20 min)

**Say (3 min, before any code)**
- "Before choosing a metric, decide what *good* means for the business." Show the business-objective table.
- Ask the room to call out criteria: on-brand, within 20 words, no invented specs, no health claims. Write them on a flip chart or slide. You will revisit this list in Lab 2.

**Do**
1. Create `lab1_common.py` (1.1). Walk through the four parts: prompt template, model, parser, the pipe. One sentence each.
2. Explain temperature 0.7: "creative, but variable."
3. Create and run `lab1_generate_marketing.py` (1.2). Read two outputs aloud next to their references.
4. Human evaluation (1.3): give learners 2 minutes to fill the table for m1 to m5 on their own screens.
5. Create and run `lab1_rule_checks.py`. Point at `within_limit`, `says_cheap`, `health_claim`.
6. Create and run `lab1_intent.py` (1.4).

**Expect**
- Captions are fluent but often different from the references. Possible issues: extra hashtags or emojis, quotation marks, going over the word limit, adding specs not in the brief (check with your dry run).
- Intent classification is mostly right. If you see mismatches, check whether they are real errors or format errors such as `Returns.` with capital letter or full stop.

**Ask**
- "Look at your table. Would you publish m2? Why or why not?" (Surfaces brand and fact issues.)
- "Why is Exact Match fine for intents but not captions?"
- "If the model says `The label is returns`, did it fail?" Strict Exact Match says yes; a human says no. Preview Lab 2.

**Watch for**
- Slow generation on laptops: run the generation script while you talk through the table.
- Learners who change `temperature` and get different outputs: welcome it, and tie it to "stochastic outputs make small differences noisy."

**Bridge:** "We have outputs and a human opinion. Now let's see what automated metrics say."

### Lab 2: Traditional NLP metrics (35 min)

**Say (2 min)**
- "These metrics all compare to a reference. They measure similarity, not truth."

**Do and explain**

1. **Metric helpers (2.1).** Create `lab2_metrics_common.py`. Mention the first run downloads data.
2. **Closed answers (2.2), 6 min.** Run `lab2_closed_answer.py`.
   - Explain raw EM vs normalised EM vs Accuracy using the printed numbers.
   - **Expect:** if raw EM < normalised EM, it is a **format** problem, not a model problem. Use the interpretation table.
3. **Generation metrics (2.3), 8 min.** Run `lab2_generation_metrics.py`. Teach one metric at a time using the "How to read each score" table:
   - BLEU: precision of n-grams; 0 or tiny on short texts.
   - ROUGE: coverage of the reference words.
   - METEOR: adds stems and synonyms.
   - BERTScore: embedding similarity; looks high for everything on the same topic.
   - Point at a row where BLEU is low but you would still publish the caption.
4. **The disagreement demo (2.4), 10 min. This is the key teaching moment.** Run `lab2_metric_disagreement.py`. Walk through candidates A to D in order:
   - A near-copy: all high ("metrics reward copying").
   - B paraphrase: lexical metrics drop, BERTScore holds (good copy penalised).
   - **C wrong fact (8 hours vs 2 hours): scores stay high.** Pause. Ask: "Would you ship C?" Then say: "Not one of these metrics can see the error. That is why Part 2 exists."
   - D off-topic: lower, but BERTScore still not low (same domain).
5. **Improve and re-measure (2.5), 8 min.** Run `lab2_prompt_v2.py`.
   - State two hypotheses: lower temperature, few-shot examples.
   - Show the comparison table and the **leakage warning**: the few-shot examples are m3 and m4, so scores on those items are inflated; use the "excluding leaked items" table for an honest comparison.
   - Show the rule-check table for v2: did the prompt change break a rule?
6. **Discuss (2.6), 3 min.** Use the three questions and suggested answers.

**Expect:** exact numbers vary; what matters is the *pattern* (C close to A; B low on BLEU). If your dry run did not show the pattern, say so honestly and explain the cause (smoothing, tokenisation).

**Ask**
- "Which metric would you put on a marketing dashboard?" (Probably BERTScore or ROUGE-L as a regression signal, plus rule checks, plus human review.)
- "With five examples, is a 0.03 gain real?" (No. Small samples plus randomness.)

**Watch for**
- BERTScore download hanging: have the model cached beforehand.
- NLTK `LookupError`: the `punkt_tab` and `wordnet` downloads are in `lab2_metrics_common.py`; run once with internet.
- Learners over-interpreting absolute numbers: repeat "compare, don't worship the number."

**AI moment:** ask the assistant "why is my BLEU 0.0 on a short sentence?" and show how smoothing resolves it.

**Bridge (then break, 10 min):** "Our metrics say these captions look fine, yet none can verify a fact. After the break, we give the model documents."

---

## Break (10 min)

Use it to: let slow runs finish, confirm everyone has `lab1_common.py`, and run `python lab3_02_embed_store.py` yourself so Chroma is warm.

---

## 4. Section 3: RAG Pipeline (40 min)

### Lab 3: Build a RAG pipeline (40 min)

**Say (3 min)**
- "Customers now ask factual questions. A plain LLM guesses; RAG hands it the right text at question time."
- Draw the pipeline: Documents, Chunk, Embed, Store, Retrieve, Generate (slide 5).
- Emphasise: **the model never learns the documents; we paste retrieved text into the prompt.**

**Do and explain**

| Step | Time | What to say | Run |
|---|---|---|---|
| 3.1/3.2 Load and chunk | 5 min | A `Document` is text plus metadata. The `source` metadata lets us check later which file an answer came from. Smaller chunks give precise hits but can split facts; larger chunks add noise; overlap protects boundaries. | `lab3_01_load_and_chunk.py` |
| 3.3 Embed and store | 5 min | Each chunk becomes a vector. Chroma stores vectors, text and metadata in `chroma_db/`. | `lab3_02_embed_store.py` (show the `Chunks stored` count and vector length) |
| 3.4 Retrieval | 5 min | Chroma returns a **distance**, so lower is closer. Do not compare distances across models. | `lab3_03_retrieval.py` |
| 3.5 Augmented generation | 5 min | Show the full context sent to the model, then the answer. Point at the grounding instruction in the system prompt. | `lab3_04_augmented_generation.py` |
| 3.6 Package it | 4 min | "Evaluation needs the contexts as well as the answer, so we wrap everything in `rag_answer()` returning answer, contexts and sources." Open `rag_app.py` briefly. | create `rag_app.py`, run `lab3_test_rag_app.py` |
| 3.7 Compare | 5 min | Standalone vs RAG on nine questions. | `lab3_05_compare.py` |
| 3.8 Failures | 8 min | See below. | `lab3_06_failures.py` |

**Expect**
- q1: retrieval returns `products.md`; the RAG answer says 2 hours; the plain model guesses something else.
- Plain model invents warranty lengths, prices or shipping times.
- q9 (two documents plus arithmetic) is difficult for a 3B model.

**Run the failures one at a time and say the lesson aloud**

| Failure | Say |
|---|---|
| A: out-of-scope (robot vacuum) | "Vector search always returns something. Only the prompt can say 'I don't know'." |
| B: two documents, k=1 vs 3 | "With k=1 a complete answer is impossible; the facts live in two files." |
| C: tiny chunks | "A fact is cut off from the heading that gives it meaning." |
| D: loose prompt | "Without a grounding rule, the model answers confidently anyway." |

**Ask**
- "Where does the plain model invent facts?" Let learners find examples in the compare output.
- "Where does RAG still struggle, and why?"
- Learner challenge (optional, 2 min): add a contradicting document. "Why is this an evaluation problem, not only a retrieval problem?" (The system retrieves both rules and may pick the wrong one; faithfulness to the context does not mean correctness.)

**Watch for**
- Chroma collection already filled with older chunks after you change documents: delete `chroma_db/`.
- Embedding calls slow on first run: warm up beforehand.
- Learners worried that the plain model "got it right" once: ask for a second run and a different question; luck is not reliability.

**Connect to the enterprise (1 min):** HR bots, IT helpdesks, support assistants, compliance Q&A. "A fluent wrong answer is worse than 'I don't know'."

**AI moment:** ask the assistant to explain `chunk_size` and `chunk_overlap` with a short example, or how to convert Chroma distance to similarity.

**Bridge:** "We can *see* these failures by reading outputs. In the next section we make them measurable."

---

## 5. Section 4: Evaluating the RAG Application (50 min)

**Timing:** Lab 4 about 35 min + Lab 5 (5.1 to 5.3) about 15 min

### Lab 4: RAG evaluation (35 min)

**Say (3 min, slide 6)**
- "A RAG answer can fail in different places, and each needs a different fix."
- Draw: Question to Retrieval to Contexts to Generation to Answer. Retrieval metrics on the left (Context Precision, Context Recall, Context Relevance); generation metrics on the right (Faithfulness, Answer Relevance, Answer Correctness).

**Do and explain**

1. **Generate and retrieval check (4.1, 4.2), 6 min.** Run `lab4_01_generate_and_retrieval_check.py`.
   - Say: "Run the cheap deterministic check first. No judge model, no cost."
   - Explain hit rate, all-found rate and MRR with the printed table. Highlight q9, where `all_found` should be the interesting one.
   - Caveat: a document-level hit does not prove the right chunk was found.
2. **RAGAS setup (4.3), 4 min.** Create `lab4_common.py`. Explain the four fields: `user_input`, `retrieved_contexts`, `response`, `reference`. Mention that RAGAS APIs change between versions and that a judge LLM is itself an LLM.
3. **Run the evaluation (4.4), 4 min.** Start `lab4_02_ragas_eval.py` (or load `--cached`). Say what it does while it runs: "It asks a judge model to split answers into claims, compare them to contexts and references."
4. **Metrics and ranges (4.5), 6 min.** Teach with the table; for each metric give a one-line question:
   - Faithfulness: "Is every claim supported by the context?"
   - Answer Relevance: "Does it address the question?"
   - Context Precision: "Are useful chunks ranked first?"
   - Context Recall: "Did we fetch everything the reference needs?"
   - Context Relevance: "Is the retrieved text on topic?"
   - Answer Correctness: "Does it agree with the ground truth?"
   - Give the heuristic ranges, then the caveats: judges vary; do not compare across judge models; **a correct refusal can score low on Answer Relevance**.
5. **Read the report row by row (4.6), 6 min.** Run `lab4_03_read_report.py`. Pick the lowest-scoring row and walk the decision flow: Recall low? Faithfulness low? Relevance low? Correctness low?
   - Learners fill the diagnosis table for three rows (2 min).
6. **Failure demonstrations (4.7), 6 min.** Show `lab4_case1_loose_prompt.csv` and `lab4_case2_bad_retrieval.csv` (or run the script if time permits).
   - Case 1: retrieval right, Faithfulness drops. Say: "good retrieval, bad generation: fix the prompt or model."
   - Case 2: Recall low, Faithfulness may be high. Say: "**faithful but wrong**. The model honestly used poor context. Fix retrieval."
   - Show the summary matrix.
7. **Fallback judge (4.8), 2 min (optional).** Show the hand-written fact-checker to demystify what Faithfulness does and discuss judge biases (verbosity, agreeing with fluent text, prompt sensitivity).

**Expect:** q1, q2, q5, q6 scores are generally high; q9 and the refusal questions q7 and q8 are the interesting ones. NaN values are possible with a local judge; explain that this is a tooling limitation, not a result.

**Ask**
- "Faithfulness 0.95, Correctness 0.2. Where do you look first?" (Context Recall and the retrieved chunks.)
- "A refusal got Answer Relevance 0.1. Is the system wrong?" (No, the metric lacks business context; add a 'should refuse' flag.)
- "Can the judge be the same model as the generator?" (Possible, but weaker; bias risk. Prefer a stronger or different judge.)

**Watch for**
- **Import error** `No module named 'langchain_community.chat_models.vertexai'`: version mismatch between ragas and langchain-community. Fix with `pip install "langchain-community<0.4.2"` (try `<0.4` if needed). Turn it into an AI-debugging demo.
- Very slow judge: use cached results and re-run only one or two questions live.
- Column-name differences between RAGAS versions: print `res.columns` and adjust.

**Bridge:** "We have scores for the RAG app. Did RAG actually help versus the plain model, and what would we fix next?"

### Lab 5: Compare, diagnose, improve (5.1 to 5.3, 15 min)

**5.1 Standalone vs RAG (5 min)**
- **Say:** "Faithfulness and context metrics need contexts, so the standalone model can only be compared on answer-level metrics."
- **Do:** show `lab5_01_standalone_vs_rag.py --cached`.
- **Expect:** RAG far ahead on Answer Correctness for fact lookups (q1, q2, q5, q6); the BERTScore gap is much smaller; q9 may fail for both.
- **Say:** "The metric you choose decides whether you notice the improvement. BERTScore alone would have understated the value of RAG."

**5.2 Diagnosis playbook (5 min)**
- **Do:** show the table, then run the four practice cases as a quick poll (30 seconds each).
- **Answers:** (1) retrieval failure with an honest generator; (2) good retrieval, unfaithful generator; (3) good answers, noisy ranking; (4) metric artefact for a correct refusal.

**5.3 Improvement loop (5 min)**
- **Say:** "Change one thing at a time, re-measure, record."
- **Do:** show `lab5_02_experiments.py --cached`. Fill in the experiment log together.
- **Ask:** "Did increasing k help recall? What did it cost in latency and tokens? With nine questions, which differences would you trust?" (Few. Grow the test set, repeat runs.)
- **AI moment:** paste the table into an AI assistant, ask which component to fix first, then compare its answer with the playbook.

**Watch for:** running all four experiments live will take too long; use cached results and `--limit=5` if you want to show mechanics.

---

## 6. Section 5: Conclusion and Q&A (10 min)

**Do (6 min, slide 7 and Lab 5 parts 5.4 and 5.5)**
1. Business link (5.4): "Can the assistant answer customers unsupervised? Evidence: Faithfulness and Correctness on policy and price questions, plus refusal behaviour."
2. Workflow recap: business goal, references and rule checks, choose metrics for the architecture, run and read rows, diagnose, iterate.
3. Standalone vs RAG table: main risk, core metrics, debug question.
4. Five takeaways (read them; ask learners to say one in their own words).
5. Exit ticket: ask the three questions aloud or put them in chat.

**Q&A (4 min)**

Prepared answers for frequent questions:

| Question | Answer |
|---|---|
| How big should the evaluation set be? | Start with 30 to 50 carefully chosen questions covering easy, hard, out-of-scope and adversarial cases; grow to hundreds. Report spread across repeated runs. |
| Can the judge be the generator? | Possible but biased; prefer a stronger or different judge, and spot-check with humans. |
| Which single metric should we track? | None. Track one or two per component (for example Context Recall, Faithfulness) plus rule checks and periodic human review. |
| How do I evaluate multi-turn chats? | Evaluate per turn with conversation context, plus task-level success. It is an extension of the same ideas; RAGAS and other frameworks support multi-turn samples. |
| Local vs hosted judge? | Hosted judges are usually more reliable and faster; local judges are free and private but slower and noisier. |
| When do I stop tuning? | When the metrics tied to the business risk meet an agreed threshold on a representative, held-out set. |

**Close:** point learners to the optional take-home deliverable (one-page evaluation report) and the README folder.

---

## 7. If You Are Running Behind

| Remaining time | Cut |
|---|---|
| 5 minutes short | Skip the fallback judge (4.8) and the Lab 3 learner challenge |
| 10 minutes short | Show Lab 2 prompt improvement (2.5) from cached output only; show Lab 4 failure cases (4.7) from CSVs |
| 15 minutes short | Also skip Lab 5 experiments (5.3), and describe the improvement loop verbally using the table |
| Environment broken for a learner | Pair them with a neighbour; give them the CSVs from `outputs/` so they can still read results |

**Never cut:** the Lab 2 candidate C demonstration, Lab 3 failure demonstrations, the Lab 4 faithful-but-wrong case, and the five takeaways. They carry the main learning.

---

## 8. Common Failure Reference

| Where | Symptom | Fix |
|---|---|---|
| Lab 0 to 5 | `SyntaxError` after pasting a `tee` block | Closing `EOF` must be alone on its line, column 1 |
| Lab 0 | Plain PowerShell | Use WSL or Git Bash |
| Lab 1 | Connection refused | `ollama serve`; `ollama list` |
| Lab 2 | NLTK `LookupError` | Re-run once with internet; downloads happen in `lab2_metrics_common.py` |
| Lab 2 | `evaluate.load` slow | Pre-download before the session |
| Lab 3 | Strange retrieval after editing documents | Delete `chroma_db/` and rebuild |
| Lab 4 / 5 | `No module named ... vertexai` | `pip install "langchain-community<0.4.2"` (or `<0.4`) |
| Lab 4 / 5 | `NaN` scores or timeouts | Lower `max_workers`, raise timeout, use a stronger judge, or `--cached` |
| Lab 4 / 5 | `KeyError` for a metric column | Print `res.columns`; names differ by RAGAS version |
| Lab 5 | Experiments take very long | `--limit=5`, or load `--cached` |

---

## 9. Teaching Principles to Keep in Mind

1. **Start from the business goal**, then pick metrics.
2. **Show failures on purpose.** Bad scores teach more than good ones.
3. **Read rows, not just averages.** Do this live every time a table appears.
4. **Say what a metric cannot do** every time you introduce it.
5. **Compare, never worship numbers.** Thresholds in the guide are heuristics.
6. **Model good AI habits:** use an assistant for ideas and debugging, then verify against output and documentation.
7. **Be honest about uncertainty:** if your dry-run output differs from this guide, trust your output and explain why.

---

## 10. Quick Timeline (print this)

| Clock | Part | Content |
|---|---|---|
| 0:00 | Section 1 | Intro talk (10 min) + Lab 0 (10 min) |
| 0:20 | Section 2 | Lab 1 (20 min) |
| 0:40 | Section 2 | Lab 2 (35 min) |
| 1:15 | Break | 10 min |
| 1:25 | Section 3 | Lab 3 (40 min) |
| 2:05 | Section 4 | Lab 4 (35 min) |
| 2:40 | Section 4 | Lab 5, parts 5.1 to 5.3 (15 min) |
| 2:55 | Section 5 | Conclusion and Q&A (10 min) |
| 3:05 | End | The break is the only extra time beyond the 3-hour content plan |
