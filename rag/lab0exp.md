# Lab 0 explained: setup, environment check and evaluation data

This lab builds the foundation for the whole course. You are creating (1) a working local AI stack, (2) a small fictional company knowledge base, and (3) three hand-checked test sets. Every later lab reuses these files.

## The big picture

```
novahome_eval/
├── data/
│   ├── docs/                    ← knowledge base (RAG will search this)
│   │   ├── products.md
│   │   ├── policies.md
│   │   ├── brand_guidelines.md
│   │   └── support_faq.md
│   ├── marketing_eval.json      ← Dataset A (5 items)
│   ├── intent_eval.json         ← Dataset B (10 items)
│   └── rag_eval.json            ← Dataset C (9 items)
└── outputs/                     ← results from later labs
```

## Part 1: Installs (0.1)

**Ollama** runs LLMs locally. You pull three models:

| Model | Role |
|---|---|
| `llama3.2:3b` | **Generator**: the small model that writes answers |
| `nomic-embed-text` | **Embedder**: turns text into vectors so RAG can find relevant chunks |
| `llama3.1:8b` | **Judge** (optional): a stronger model that grades answers in RAGAS (Lab 4) |

Python packages, grouped by purpose:

- **LangChain family** (`langchain`, `langchain-ollama`, `langchain-chroma`, ...): glue code for calling models and building RAG pipelines
- **chromadb**: the vector database storing your document embeddings
- **Metric libraries** (`evaluate`, `rouge_score`, `bert_score`, `nltk`): classic text-similarity scores
- **ragas**: RAG-specific evaluation (faithfulness, relevance, etc.)

The NLTK download block fetches dictionaries (`wordnet`, etc.) that the METEOR metric needs to match synonyms.

## Part 2: Environment check (0.2)

```python
pkgs = ["langchain", "langchain_ollama", ...]
for p in pkgs:
    try:
        importlib.import_module(p)
        print(f"OK       {p}")
    except Exception as e:
        print(f"MISSING  {p}: {e}")
```

This tries to import each package. If one fails, it prints `MISSING` plus the reason, so you see all problems at once instead of one at a time.

```python
print(subprocess.run(["ollama", "list"], capture_output=True, text=True).stdout)
```

Runs the shell command `ollama list` from Python and prints which models are downloaded.

Then the model test:

```python
llm = ChatOllama(model="llama3.2:3b", temperature=0)
print(llm.invoke("Reply with the single word: ready").content)
```

`temperature=0` makes output as deterministic as possible, which matters in evaluation because you want repeatable results. `.invoke()` sends a prompt and returns a message; `.content` is the text.

```python
emb = OllamaEmbeddings(model="nomic-embed-text")
print(len(emb.embed_query("hello world")), "embedding dimensions")
```

An embedding is a list of numbers representing meaning. `nomic-embed-text` returns 768 numbers for any text. Seeing a number printed proves the embedder works.

## Part 3: The knowledge base (0.3)

Each file is created with this pattern:

```bash
tee data/docs/products.md > /dev/null << 'EOF'
...file contents...
EOF
```

- `tee <file>` writes its input to a file
- `> /dev/null` hides the echo of the content on screen
- `<< 'EOF' ... EOF` is a "heredoc": everything between the markers is fed as input. Quoting `'EOF'` stops the shell from interpreting characters like `$89`

**Why fictional?** The model has never seen "NovaBrew". So if it answers "keeps water warm for 2 hours", that fact *must* have come from retrieval. If it invents something, you can spot the hallucination instantly.

What each document is for:

- **products.md**: specs and prices (factual lookups)
- **policies.md**: returns, warranty, shipping (rule-based answers)
- **brand_guidelines.md**: tone and constraints (e.g. captions under 20 words, no health claims)
- **support_faq.md**: troubleshooting (e.g. kettle only supports 2.4 GHz Wi-Fi)

## Part 4: The three evaluation datasets (0.4)

Each dataset teaches a different *type* of evaluation:

**Dataset A: `marketing_eval.json`** (open-ended generation)

```json
{"id": "m1", "format": "social media caption", "max_words": 20,
 "brief": "...", "reference": "..."}
```

The model writes a caption from a `brief`. The `reference` is one example of a good answer, and `max_words` is a constraint you can check automatically. Used with ROUGE/BLEU-style metrics in Labs 1-2.

**Dataset B: `intent_eval.json`** (closed answer)

```json
{"text": "I want to send back my air fryer.", "label": "returns"}
```

There is exactly one right answer (`returns`), so you can score with plain **accuracy / exact match**. Labels: `product_info`, `returns`, `shipping`, `warranty`.

**Dataset C: `rag_eval.json`** (question answering)

```json
{"id": "q1", "question": "...", "reference": "Up to 2 hours.",
 "expected_sources": ["products.md"]}
```

`expected_sources` lets you later check whether retrieval found the *right file*. Some items are deliberately tricky:

| ID | Tests | Desired behaviour |
|---|---|---|
| q7 | Question about a product that doesn't exist (robot vacuum) | Say the documents don't mention it |
| q8 | Request for a prohibited health claim | Refuse and offer a compliant caption |
| q9 | Needs **two documents** (price from products, shipping from policies) | Combine: $89 + $149 = $238, shipping free |

Note that q7 has an empty `expected_sources`, because no document should be retrieved.

## Part 5: Verification (0.4 Step 4)

```python
for name in ["marketing_eval", "intent_eval", "rag_eval"]:
    items = json.load(open(f"data/{name}.json"))
    print(f"data/{name}.json  ->  {len(items)} items (valid JSON)")
```

`json.load` raises an error if your JSON is malformed, so this doubles as a validity check. Expected counts: 5, 10, 9.

## Part 6: Discussion answers (0.5)

1. **Many valid captions exist**, but reference-based metrics compare against one phrasing and penalise good alternatives.
2. **q7**: admit ignorance rather than hallucinate. **q8**: refuse the claim but stay helpful.
3. **Invented company**: no memorisation possible, so correct answers prove retrieval worked.

## Common problems

- **`Permission denied`**: you're in a read-only folder; `cd novahome_eval`.
- **JSON validation fails**: the closing `EOF` must be alone on its line with no leading spaces.
- **Ollama errors**: make sure the Ollama app/service is running and the models were pulled.
- **Windows**: use WSL or Git Bash for the `tee` commands.

I can also walk through Lab 1 next, or run these setup steps for you here if you'd like the files generated.
