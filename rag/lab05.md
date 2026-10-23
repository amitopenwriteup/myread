# Trainer Guide: LLM Evaluation Labs 0 to 5 (Teleprompter Script)

**Workshop:** LLM Evaluation Demonstration (3 hours) | **Deck:** `LLM_Evaluation_Demonstration_Detailed.pptx` (42 slides)
**Stack:** LangChain, ChromaDB, Ollama, RAGAS | **Use case:** NovaHome Appliances (fictional)

**Conventions:** plain lines are read aloud. `[DO]` is an action. `[ASK]` is a question to the room. `[PAUSE]` means wait. `[DRAFT]` marks content written without the lab file, to confirm.

---

## 0. Schedule at a glance

| Perform after slide | Lab | Live time | Section |
|---|---|---|---|
| 6 (Two demonstrations, one application) | Lab 0: Setup, environment check, evaluation data | 10 min | 1 Introduction |
| 8 (Building the evaluation workflow) | Lab 1: Non-RAG marketing assistant | 20 min | 2 Non-RAG app |
| 16 (When metrics disagree) | Lab 2: Score with traditional NLP metrics [DRAFT] | ~35 min (estimate) | 2 Non-RAG app |
| 25 (Enterprise knowledge assistants) | Lab 3: Build the RAG pipeline | 40 min | 3 RAG pipeline |
| 38 (Locating the source of a failure) | Lab 4: Evaluate the RAG application | 50 min | 4 RAG evaluation |
| 39 (Evaluation-driven improvement loop) | Lab 5: Compare, diagnose, improve [DRAFT] | Take-home | 4 RAG evaluation |

Slides 1 to 5 and 40 to 42 have no lab. The break is on slide 26. Live lab time is about 155 of 180 minutes, so keep slide time short.

### Before the session (pre-work)

1. Install Ollama, then pull `llama3.2:3b`, `nomic-embed-text` and (optional) `llama3.1:8b`.
2. Create the virtual environment and install the packages from Lab 0 section 0.1.
3. Download the NLTK data (`wordnet`, `punkt`, `punkt_tab`, `omw-1.4`) for METEOR.
4. Record the exact RAGAS version with `pip show ragas` and share a pinned `requirements.txt`.
5. Run the full Lab 4 RAGAS evaluation once so `outputs/lab4_ragas_report.csv` exists for live use.
6. Open a terminal inside `novahome_eval/` with the environment active and Ollama running.

### Known differences between the slides and the labs

| Topic | Slides say | Labs say |
|---|---|---|
| Embedding model | Hugging Face model (slide 20) | `nomic-embed-text` via Ollama |
| Chunk size / overlap | 800 / 100 (slide 22) | 350 / 50 |
| Top-k | 4 (slide 22) | 3 |
| Generator | `llama3` (slide 22) | `llama3.2:3b` |
| Reference field | `ground_truth` (slide 29) | `reference` |
| Score bands | 0.80 / 0.50 (slide 35) | 0.85 / 0.60 / 0.30 (Lab 4) |
| Example product | EcoBottle (slide 15) | NovaHome kettle, purifier, air fryer |

Say once: slide values are illustrative; the labs use the lab-file settings.

---

## Lab 0: Setup, environment check and evaluation data

**After slide 6 | 10 min live | Guided**

### Objectives
- Confirm Ollama models and Python packages work.
- Create the document collection and three evaluation datasets used by every later lab.

### Opening script
Now we move from slides to the keyboard. This first lab is only setup and takes about ten minutes. The installs are already done, so we only check that everything works.

Everything today uses one fictional company called NovaHome. We invented it on purpose. A model cannot have memorised NovaHome's prices or policies. So if an answer is right, it came from our documents. If it is wrong, we spot the invention straight away.

### Steps

**0.2 Environment check**
- `[DO]` In a notebook inside `novahome_eval/`, run the package check cell and `ollama list`.
- Say: Every package should say OK and Ollama should list our models. Fix anything marked MISSING now.
- `[DO]` Run the model test (`ChatOllama` reply "ready", then `OllamaEmbeddings` length).
- Say: The generator should answer with the word ready. The embedding model returns a few hundred numbers; that is the vector the RAG labs store.

**0.3 Folders and knowledge base**
- `[DO]` `mkdir -p data/docs outputs`, then run four `tee` blocks: `products.md`, `policies.md`, `brand_guidelines.md`, `support_faq.md`.
- Say: `tee` writes a file from the text we paste. The quotes around EOF stop the shell changing characters such as the dollar sign in a price. These four files are our company knowledge.

**0.4 Evaluation datasets**
- `[DO]` Run the three `tee` blocks: `marketing_eval.json` (5 items), `intent_eval.json` (10 items), `rag_eval.json` (9 items).
- Say: Marketing has five briefs with reference captions. Intent has ten customer messages with one correct label each. RAG has nine questions with reference answers and the document each answer should come from.
- Say: Look at questions seven, eight and nine. Seven asks about a robot vacuum we do not sell. Eight asks the assistant to claim the purifier prevents the flu, which the brand rules forbid. Nine needs facts from two documents plus a sum.

**Verify**
- `[DO]` `ls -R data`, then run the JSON check script.
- Expected: 4 files in `data/docs/`; `intent_eval.json` 10 items; `marketing_eval.json` 5 items; `rag_eval.json` 9 items.

### Discussion (5 min) `[ASK]`
1. Why is a single reference a problem for marketing copy? *Reference metrics reward similarity to one phrasing, so valid alternatives score poorly.*
2. What do we want for q7 and q8? *q7: admit the documents contain no robot vacuum. q8: refuse the health claim and offer a compliant alternative.*
3. Why invent a company? *The model cannot have memorised it, so correct answers can only come from retrieval, and hallucinations are easy to spot.*

### Checkpoint
- [ ] Ollama models respond
- [ ] `data/docs/` has 4 files
- [ ] The three JSON datasets exist

### If it goes wrong
- `Permission denied`: you are in a read-only folder; `cd` into `novahome_eval/`.
- JSON check fails: the closing `EOF` must be alone on its line with no leading spaces.

---

## Lab 1: Build the non-RAG LLM application (marketing assistant)

**After slide 8 | 20 min live | Demonstration plus guided coding**

### Business context
NovaHome's marketing team wants an assistant that drafts social captions and email subject lines from short briefs.

| Business objective | Implication for evaluation |
|---|---|
| Save copywriter time | Outputs usable with light editing |
| Stay on brand | Tone, length and vocabulary rules respected |
| Never mislead customers | No invented specs, no health claims |

### Objectives
- Build a prompt, model and parser chain with LangChain and a local Ollama model.
- Generate outputs for representative inputs.
- Create the output table later labs score.
- Build a closed-answer task (intent classification) for Accuracy and Exact Match.

### Files created
`lab1_common.py`, `lab1_generate_marketing.py`, `lab1_rule_checks.py`, `lab1_intent.py`

### Opening script
NovaHome's marketing team wants an assistant that drafts captions and subject lines from short briefs. They want to save copywriter time, stay on brand and never mislead customers. Before we choose any metric, write down what good means. `[PAUSE]` Take one minute as a group. We will come back to this list.

### Steps

**1.1 The generation chain**
- `[DO]` Create `lab1_common.py` with `tee`.
- Say: A chain has three parts joined by a pipe: a prompt template, a local model and a parser that returns plain text. The prompt says be friendly, stay under the word limit, avoid health claims and return the copy only. Temperature 0.7 adds some randomness, as a copywriter would have.

| Component | Role |
|---|---|
| `ChatPromptTemplate` | Turns structured inputs into a prompt; versionable |
| `ChatOllama` | Local model; temperature controls randomness |
| `StrOutputParser` | Converts the model message to a string |
| `|` | LangChain Expression Language pipe |

**1.2 Generate outputs**
- `[DO]` Create and run `lab1_generate_marketing.py` (`python lab1_generate_marketing.py`).
- Say: For each brief we print the reference and the model's caption. Read them yourselves before any score appears.
- Output: `outputs/lab1_marketing_v1.csv`.

**1.3 Human review, then rule checks**
- `[DO]` Fill in the table together: on brand, within word limit, invented facts, would you publish it (rows m1 to m5).
- `[DO]` Create and run `lab1_rule_checks.py`.
- Say: Your judgement is the ground truth metrics try to copy. Word limit, the banned word "cheap" and forbidden health words need no similarity score. Real evaluation suites mix both kinds.
- Output: `outputs/lab1_marketing_v1_checks.csv`.

**1.4 Intent classification**
- `[DO]` Create and run `lab1_intent.py`.
- Say: One right answer per message: product info, returns, shipping or warranty. Temperature is zero so the answer is stable.
- Say: For any mismatch ask: is the model wrong, or is the format wrong? "Returns." instead of `returns` is the right idea in the wrong shape. The two failures have different fixes.
- Output: `outputs/lab1_intent.csv`.

### Discussion `[ASK]`
1. Which customer messages are genuinely ambiguous? Could two labellers disagree?
2. Why use Exact Match for intents but not for captions?
3. If the model returns "The label is returns", did it fail? Which metric says yes, which says no?

### Optional challenge
Add a sixth brief to `data/marketing_eval.json` that tempts a health claim. Predict the result, then re-run both scripts.

### Checkpoint
- [ ] The four Lab 1 scripts exist
- [ ] `outputs/lab1_marketing_v1.csv` and `outputs/lab1_intent.csv` exist
- [ ] The group wrote its "what good looks like" list

### If it goes wrong
- `ModuleNotFoundError: lab1_common`: wrong folder, or the first `tee` block was not run.
- `Connection refused`: start Ollama with `ollama serve`, check `ollama list`.

### Bridge
Slides 9 to 16 cover the metrics. Lab 2 then scores these outputs.

---

## Lab 2: Score with traditional NLP metrics `[DRAFT]`

**After slide 16 | ~35 min (estimate) | Guided coding**

> The Lab 2 file was not available. This section follows slides 9 to 16 and the Lab 1 outputs. Match it to your Lab 2 file before presenting.

### Objectives
- Score Lab 1's intent outputs with Accuracy and Exact Match.
- Score Lab 1's captions against references with BLEU, ROUGE, METEOR and BERTScore.
- Read disagreements between metrics against real outputs.

### Opening script
You have read the outputs and you know the five metrics. Now we put numbers on Lab 1. We score the captions against their reference captions and the intent labels against the correct labels.

### Steps (proposed)
**2.1 Intent accuracy and exact match.** Score `outputs/lab1_intent.csv`. Accuracy is correct outputs over total outputs. Normalise case and spacing first (slide 10). Say: exact match marks "Returns." wrong even though the meaning is right.

**2.2 BLEU, ROUGE and METEOR.** Score `outputs/lab1_marketing_v1.csv` against `reference`. METEOR needs the NLTK data from Lab 0. Say: a good caption in different words scores low on BLEU; that is the metric being strict, not the copy being bad.

**2.3 BERTScore.** Add it to the same table. Say: it compares meaning, so paraphrases score high, and scores sit in a narrow high band, so compare rows, not absolute values.

**2.4 Read the disagreements.** Sort by the gap between BLEU and BERTScore and read the top rows. This is slide 16 on real data:
- BLEU low, BERTScore high: acceptable copy, BLEU understates it.
- Close wording but a wrong fact: overlap hides the error. A caption saying eight hours instead of two still scores high on every similarity metric.
- Right facts, off-brand voice: needs a rubric or human review.

**2.5 Return to the "what good looks like" list.** `[PAUSE]` Which items did any metric capture? Word limit and banned words came from rule checks. Tone and invented facts needed a human.

### Discussion `[ASK]`
1. Where was BLEU misleading, and what would you use instead?
2. Which brand rules can no similarity metric check?
3. What would catch invented product facts? *A fact check or an LLM judge, which is where Section 4 goes.*

### Bridge
Similarity is not truth. To check facts against source text we need retrieval, so next we build a RAG pipeline.

---

## Lab 3: Build a RAG pipeline with LangChain and ChromaDB

**After slide 25 (before the break on slide 26) | 40 min live | Demonstration plus guided coding**

### Objectives
- Walk through documents, chunking, embeddings, vector store, retrieval, augmentation and generation.
- Package the pipeline as `rag_app.py` so later labs can vary its settings.
- Compare standalone and RAG answers.
- Observe successful and unsuccessful retrieval.

```
Question -> Embed -> Search Chroma -> Top-k chunks -> Prompt (context + question) -> LLM -> Answer
Documents -> Split -> Embed -> Store
```

### Files created

| File | Step | Purpose |
|---|---|---|
| `lab3_01_load_and_chunk.py` | 3.1, 3.2 | Load and split documents |
| `lab3_02_embed_store.py` | 3.3 | Embed chunks, store in ChromaDB |
| `lab3_03_retrieval.py` | 3.4 | Similarity search and distances |
| `lab3_04_augmented_generation.py` | 3.5 | Paste context into a prompt and generate |
| `rag_app.py` | 3.6 | Reusable module for Labs 4 and 5 |
| `lab3_test_rag_app.py` | 3.6 | Quick module test |
| `lab3_05_compare.py` | 3.7 | Standalone vs RAG on all questions |
| `lab3_06_failures.py` | 3.8 | Four failure demonstrations |

### Opening script
Customers now ask factual questions: prices, policies, specs, brand rules. A standalone model has never seen our documents, so it guesses. RAG finds the right text at question time and pastes it into the prompt. The model never learns the documents; we only hand it the relevant pieces each time.

### Steps

**3.1 and 3.2 Load and chunk**
- `[DO]` Create and run `lab3_01_load_and_chunk.py`.
- Say: Each file becomes a document: text plus metadata. The source filename lets us check which file an answer came from. We split into chunks of 350 characters with 50 overlap.

| Choice | Effect |
|---|---|
| Smaller chunks | Precise retrieval, but facts may be split |
| Larger chunks | More context per hit, but more noise and cost |
| Overlap | Protects sentences that cross a boundary |

**3.3 Embeddings and ChromaDB**
- `[DO]` Create and run `lab3_02_embed_store.py`.
- Say: Each chunk becomes a vector that captures meaning. ChromaDB stores vectors, text and metadata in `chroma_db/`.

**3.4 Retrieval**
- `[DO]` Create and run `lab3_03_retrieval.py`.
- Say: We ask how long the kettle keeps water warm and get the three closest chunks. The score is a distance, so lower is closer. Do not compare distances across embedding models or collections.

**3.5 Augmented generation**
- `[DO]` Create and run `lab3_04_augmented_generation.py`.
- Say: Here is the context we send and the answer. The instruction says answer only from the context, and if it is missing, reply exactly: "I don't know based on the provided documents."

**3.6 Package it**
- `[DO]` Create `rag_app.py`, then create and run `lab3_test_rag_app.py`.
- Say: Evaluation needs the retrieved contexts as well as the answer, so `rag_answer()` returns the answer, the contexts and the sources.

**3.7 Standalone versus RAG**
- `[DO]` Run `lab3_05_compare.py`; output is `outputs/lab3_plain_vs_rag.csv`.

| Question | What to look for |
|---|---|
| Where does the plain model invent facts? | Prices, warranty length, shipping speeds |
| Where does RAG fix this? | Single-fact lookups (q1, q2, q5, q6) |
| Where does RAG still struggle? | q9 (two documents plus arithmetic), q7 and q8 (out-of-scope and rule-based) |

**3.8 Four deliberate failures**
- `[DO]` Run `lab3_06_failures.py`.

| Failure | Lesson |
|---|---|
| A: out-of-scope (q7) | Vector search always returns something; only the prompt's refusal rule protects us |
| B: two documents (q9, k=1 vs k=3) | With k=1 only one document can be retrieved |
| C: tiny chunks (120/0) | A fact is cut away from the heading that gives it meaning |
| D: loose prompt | Confident one-liners invite confident hallucination |

- Say: We saw these by reading. In Lab 4 we make them measurable.

**3.9 Enterprise link.** Say: HR bots, helpdesks, compliance assistants and sales tools use this pattern. A fluent wrong answer is worse than "I don't know."

### Learner challenge
Add `data/docs/faq_shipping_exceptions.md` containing a rule that contradicts `policies.md`. Delete `chroma_db/`, re-run `lab3_05_compare.py`, and describe what the assistant does. Why is this an evaluation problem and not only a retrieval problem?

### Checkpoint
- [ ] `rag_answer()` returns `answer`, `contexts` and `sources`
- [ ] `outputs/lab3_plain_vs_rag.csv` exists
- [ ] You observed at least three failure types

### If it goes wrong
- `model "nomic-embed-text" not found`: `ollama pull nomic-embed-text`.
- Odd or empty results after changing documents: delete `chroma_db/` and re-run; collections only fill when empty.
- `ModuleNotFoundError: rag_app`: run from `novahome_eval/` and make sure the `rag_app.py` block was run.

---

## Lab 4: Evaluating the RAG application

**After slide 38 | 50 min live | Demonstration plus guided coding**

### Objectives
- Run a fast, deterministic retrieval check with no judge model.
- Run RAGAS metrics and produce a per-question report.
- Interpret score ranges.
- Show good-retrieval/bad-generation and the reverse.
- Locate the failing component for each bad answer.

| If this is bad... | ...the problem is probably in |
|---|---|
| Context Recall / Context Relevance | Retrieval (chunking, embeddings, `k`) |
| Context Precision | Ranking and noise in retrieved chunks |
| Faithfulness | Generation going beyond the context |
| Answer Relevance | Prompt design or the question not being addressed |
| Answer Correctness | Any of the above; use the others to locate the cause |

### Files created
`lab4_01_generate_and_retrieval_check.py`, `lab4_common.py`, `lab4_02_ragas_eval.py`, `lab4_03_read_report.py`, `lab4_04_failure_cases.py`, `lab4_05_fallback_judge.py`

### Opening script
A RAG answer can be wrong for different reasons, and each needs a different fix. Retrieval may not have fetched the facts. The generator may have added claims. We use metrics to find which part failed.

### Steps

**4.1 and 4.2 Generate, then a free retrieval check**
- `[DO]` Run `lab4_01_generate_and_retrieval_check.py`.
- Say: We know which document should hold each answer, so we can compute hit rate, all-found rate and MRR with no judge model. They are fast and deterministic, so we run them first.

| Metric | Meaning |
|---|---|
| Hit rate@k | At least one expected document retrieved |
| All-found rate | Every expected document retrieved (matters for q9) |
| MRR | How high the first relevant document ranks |

- Say: A document-level hit does not prove the right chunk came back; the RAGAS context metrics check that.

**4.3 RAGAS setup**
- `[DO]` Show `lab4_common.py` briefly.
- Say: RAGAS needs the question (`user_input`), the retrieved contexts, the response and our reference. A judge model, `llama3.1:8b`, scores them. RAGAS changes between releases, so if an import fails, check `pip show ragas`.

**4.4 Run the evaluation**
- `[DO]` `python lab4_02_ragas_eval.py --cached`
- Say: A local judge is slow, so I ran this before the session and saved the report. Later I will re-run one or two questions live.

**4.5 Reading the scores**

| Metric | High score means | Low score means |
|---|---|---|
| Faithfulness | Every claim is supported by context | Hallucination or outside knowledge |
| Answer Relevance | Answer addresses the question | Off-topic, evasive or incomplete |
| Context Precision | Useful chunks come first | Relevant chunks buried under noise |
| Context Recall | Retrieval fetched what was needed | Key facts never retrieved |
| Context Relevance | Retrieved text is on-topic | Retrieval brought in noise |
| Answer Correctness | Agrees with ground truth | Disagrees or is missing facts |

| Range (lab heuristics) | Interpretation |
|---|---|
| 0.85 to 1.0 | Strong: monitor |
| 0.60 to 0.85 | Acceptable: read the lowest rows |
| 0.30 to 0.60 | Weak: investigate the component |
| Below 0.30 | Failing: fix before rollout |

- Say: Slide 35 uses 0.80 and 0.50. Both are rules of thumb; set your own thresholds from business risk.
- Caveats: judge scores vary between runs and judge models; never compare across judges; a correct refusal can score low on Answer Relevance.

**4.6 Read the report row by row**
- `[DO]` Run `lab4_03_read_report.py`; output is `outputs/lab4_report_with_answers.csv`.
- `[DO]` For each row with a score below 0.6, fill in: id, lowest metric, retrieval OK?, likely failing component, proposed fix.
- Decision flow: low Context Recall means retrieval (tune `k`, chunking, embeddings); else low Faithfulness means the generator (strengthen the grounding prompt, lower temperature); else low Answer Relevance means the prompt; else low Correctness means check the reference or the reasoning (for example, the arithmetic in q9).

**4.7 Two deliberate failure cases**
- `[DO]` Run `lab4_04_failure_cases.py`.
- Case 1 (loose prompt, good retrieval): documents are found (hit = True) but Faithfulness drops.
- Case 2 (`k=1`, tiny chunks, strict prompt): Context Recall is low while Faithfulness can stay high. The answer is faithful but wrong.

| Scenario | Context Recall | Faithfulness | Correctness | Diagnosis |
|---|---|---|---|---|
| Healthy | High | High | High | Good |
| Good retrieval, bad generation | High | Low | Low | Fix prompt or model |
| Bad retrieval, honest generation | Low | High | Low | Fix retrieval |
| Bad retrieval, hallucinating generation | Low | Low | Low | Fix both; add a refusal rule |

- Say: That second case is why we need context metrics and correctness together.

**4.8 Fallback LLM-as-judge**
- `[DO]` Run `lab4_05_fallback_judge.py` (q1 and q3).
- `[ASK]` What biases might this judge have? *Verbosity, agreeing with fluent text, sensitivity to prompt wording.*

### Learner challenge
Find a row where Answer Relevance is low but the answer is right (a correct refusal). Propose a "should refuse" flag in `data/rag_eval.json` so correct refusals count as successes.

### Checkpoint
- [ ] `outputs/lab4_retrieval_check.csv` exists
- [ ] `outputs/lab4_ragas_report.csv` saved
- [ ] You can explain faithful-but-wrong versus unfaithful-but-right
- [ ] Diagnosis table done for at least three rows

### If it goes wrong
- `FileNotFoundError: outputs/lab4_rag_run.pkl`: run `lab4_01_generate_and_retrieval_check.py` first.
- Import error mentioning `langchain_community.chat_models.vertexai`: `pip install "langchain-community<0.4.2"` (try `<0.4`).
- `ImportError` from `ragas`: check the version and adapt `lab4_common.py`.
- NaN scores or timeouts: lower `max_workers`, raise `timeout`, use a stronger judge, or load `--cached`.
- `KeyError: 'context_recall'`: print `res2.columns` and adjust.
- `model "llama3.1:8b" not found`: `ollama pull llama3.1:8b`.

---

## Lab 5: Compare, diagnose and improve `[DRAFT]`

**After slide 39 | Take-home (not in the 3-hour timing)**

> The Lab 5 file was not available. This follows slide 39 and the settings earlier labs said would be tuned (`k`, chunk size, prompt). Confirm against your Lab 5 file.

### Objectives
- Apply the loop: measure, diagnose, change, re-evaluate.
- Change one setting and compare scores against the Lab 4 baseline.

### Opening script
Slide 39 is the improvement loop: measure, diagnose, change, re-evaluate. Lab 5 is that loop on your own laptop, and it is homework if we run out of time.

### Steps (proposed)
1. **Measure.** Keep the Lab 4 report as the baseline for the same nine questions.
2. **Diagnose.** Pick the weakest row and name the failing component using the Lab 4 decision flow.
3. **Change one thing.** For example `k` (1, 3 or 5), `chunk_size` and `chunk_overlap` through `rag_answer()`, or the prompt (`GROUNDED_PROMPT` against `LOOSE_PROMPT`).
4. **Re-evaluate.** Re-run the same questions and compare with the baseline.
5. **Record.** Write down what moved, what did not, and whether anything got worse.

Say: Change one thing, re-measure, record the result. If you change two things at once you will not know which one helped.

---

## Closing script (after slide 40)

1. No single metric is enough. Combine similarity, correctness, grounding and rule checks.
2. Choose metrics from the business goal and the system architecture.
3. Similarity metrics miss wrong facts. Grounding metrics catch them.
4. Read the rows, not just the averages.
5. Scores guide fixes to prompts, retrieval and system design.

Workflow recap: Build, Generate, Evaluate, Diagnose, Improve. Thank you. Let's take your questions.
