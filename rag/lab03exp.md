# Lab 3 Explained: Build a RAG Pipeline

## The one-line story

> "A normal LLM has never seen NovaHome's documents, so it guesses. RAG looks up the right text first, pastes it into the prompt, and then asks the model to answer from it."

**RAG = Retrieval-Augmented Generation**

---

## 1. The big picture

```
Question ─► Embed ─► Search Chroma ─► Top-k chunks ─► Prompt (context + question) ─► LLM ─► Answer
                         ▲
Documents ─► Split ─► Embed ─► Store
```

There are two phases:

| Phase | When | Steps |
|---|---|---|
| **Indexing** (prepare) | Once | Load docs → split into chunks → embed → store in Chroma |
| **Querying** (answer) | Every question | Embed question → find closest chunks → build prompt → LLM answers |

Simple analogy: **an open-book exam**. The model is the student, Chroma is the library, and retrieval is finding the right page before answering.

---

## 2. The 8 files

| File | Purpose |
|---|---|
| `lab3_01_load_and_chunk.py` | Load documents and split into chunks |
| `lab3_02_embed_store.py` | Embed chunks and save to ChromaDB |
| `lab3_03_retrieval.py` | Search and view distances |
| `lab3_04_augmented_generation.py` | Paste context into a prompt and answer |
| `rag_app.py` | **Reusable module** for Labs 4 and 5 |
| `lab3_test_rag_app.py` | Quick test of the module |
| `lab3_05_compare.py` | Plain LLM vs RAG on all questions |
| `lab3_06_failures.py` | Four deliberate failures |

---

## 3. Step 3.1 and 3.2: load and chunk

```python
docs = [Document(page_content=p.read_text(), metadata={"source": p.name})
        for p in sorted(Path("data/docs").glob("*.md"))]
```

Reads each `.md` file into a `Document` object: **text + metadata**. The `source` metadata (e.g. `policies.md`) lets us check later *which file* an answer came from.

```python
splitter = RecursiveCharacterTextSplitter(chunk_size=350, chunk_overlap=50)
chunks = splitter.split_documents(docs)
```

Cuts each document into pieces of about 350 characters, with 50 characters shared between neighbours.

| Choice | Effect |
|---|---|
| Smaller chunks | Precise, but a fact may be split apart |
| Larger chunks | More context, but more noise |
| Overlap | Protects sentences that fall on a boundary |

Why chunk at all? The model prompt is limited, and searching small pieces finds the exact relevant part.

Chunk size is a **setting we tune in Lab 5**.

---

## 4. Step 3.3: embeddings and ChromaDB

```python
embeddings = OllamaEmbeddings(model="nomic-embed-text")

vectorstore = Chroma(
    collection_name="novahome_350_50",
    embedding_function=embeddings,
    persist_directory="chroma_db",
)

if vectorstore._collection.count() == 0:
    vectorstore.add_documents(chunks)
```

- An **embedding** turns text into a list of numbers that captures meaning. Similar meanings give nearby vectors.
- **Chroma** stores vector + text + metadata, and saves to the `chroma_db/` folder.
- The `if count() == 0` check stops duplicates when you re-run the script.
- The collection name `novahome_350_50` encodes the chunk settings, so different settings get separate collections.

The peek code prints the IDs, metadata and vector size (768 dimensions) so you can see what is inside.

---

## 5. Step 3.4: retrieval

```python
query = "How long does the NovaBrew keep water warm?"

for doc, score in vectorstore.similarity_search_with_score(query, k=3):
    print(f"distance={score:.3f} | {doc.metadata['source']}")
```

- Embeds the question and finds the **k = 3** closest chunks.
- The score is a **distance**, so **lower = closer**. Do not compare distances across different embedding models.

```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
```

A **retriever** is a standard wrapper around search, which can be plugged into chains.

---

## 6. Step 3.5: augmented generation

```python
GROUNDED_PROMPT = ChatPromptTemplate.from_messages([
    ("system",
     "You are a NovaHome assistant. Answer ONLY using the context below. "
     "If the context does not contain the answer, reply exactly: "
     "\"I don't know based on the provided documents.\" Be concise."),
    ("human", "Context:\n{context}\n\nQuestion: {question}"),
])
```

This is the heart of RAG. Two key parts:

1. `{context}` gets the retrieved text.
2. The system rule: **answer only from context, otherwise say you don't know.**

```python
ctx_docs = retriever.invoke(query)
context = "\n\n".join(d.page_content for d in ctx_docs)
rag_chain.invoke({"context": context, "question": query})
```

Retrieve → join chunks into one string → fill the prompt → ask the model.

**Key point:** the model never "learns" the documents. We just paste text into the prompt each time.

---

## 7. Step 3.6: `rag_app.py` (the reusable module)

Why package it? Labs 4 and 5 need to call RAG many times with different settings.

### Three prompts

| Prompt | Purpose |
|---|---|
| `GROUNDED_PROMPT` | The good one: answer only from context |
| `LOOSE_PROMPT` | Deliberately weak: "confident one short sentence" (to show failures) |
| `PLAIN_PROMPT` | No context at all (the non-RAG baseline) |

### `get_vectorstore`

```python
@lru_cache(maxsize=None)
def get_vectorstore(chunk_size=350, chunk_overlap=50):
    name = f"novahome_{chunk_size}_{chunk_overlap}"
    ...
```

- Builds or opens the collection for the given chunk settings.
- `@lru_cache` remembers the result, so it is not rebuilt on every call.

### `rag_answer`, the main function

```python
def rag_answer(question, k=3, prompt=GROUNDED_PROMPT, chunk_size=350, chunk_overlap=50, temperature=0.0):
    vs = get_vectorstore(chunk_size, chunk_overlap)
    docs = vs.as_retriever(search_kwargs={"k": k}).invoke(question)
    contexts = [d.page_content for d in docs]
    llm = ChatOllama(model=GEN_MODEL, temperature=temperature)
    answer = (prompt | llm | StrOutputParser()).invoke(
        {"context": "\n\n".join(contexts), "question": question}).strip()
    return {"question": question, "answer": answer, "contexts": contexts,
            "sources": [d.metadata["source"] for d in docs]}
```

It returns **four things**: question, answer, **contexts**, and **sources**.

Why return contexts and sources? **Evaluation needs them.** To judge RAG you must check not only the answer, but *what was retrieved*.

Tunable knobs: `k`, `prompt`, `chunk_size`, `chunk_overlap`, `temperature`.

### `plain_answer`

The same LLM with **no retrieval**, as a baseline for comparison.

---

## 8. Step 3.7: plain vs RAG

```python
rows.append({"id": item["id"], "question": q,
             "plain": plain_answer(q),
             "rag": rag_answer(q)["answer"],
             "reference": item["reference"]})
```

Runs all 9 questions through both approaches and prints them next to the reference.

| Question | What to look for |
|---|---|
| Where does plain invent facts? | Prices, warranty length, shipping speeds |
| Where does RAG help? | Single-fact lookups (q1, q2, q5, q6) |
| Where does RAG still struggle? | q9 (two documents + arithmetic), q7/q8 (out-of-scope, rule-based) |

---

## 9. Step 3.8: four deliberate failures

| Failure | Code idea | Lesson |
|---|---|---|
| **A. Out-of-scope** | Ask about a robot vacuum | Vector search **always returns something**, even if irrelevant. Only the prompt's "I don't know" rule protects you. |
| **B. Two documents** | q9 with `k=1` vs `k=3` | With `k=1`, only one file can be retrieved, so a complete answer is impossible. |
| **C. Tiny chunks** | `chunk_size=120, chunk_overlap=0` | A fact gets cut away from its heading and loses meaning (e.g. "3-year warranty" without knowing which product). |
| **D. Loose prompt** | `LOOSE_PROMPT` on the robot vacuum | "Confident one-sentence" style invites confident hallucination. |

**Bridge to Lab 4:** we can *see* these failures by reading, but evaluation metrics must **measure** them across hundreds of questions.

---

## 10. Why this matters in real companies

RAG appears in HR bots, IT helpdesks, support assistants and compliance Q&A. The shared rule:

> **A fluent wrong answer is worse than "I don't know."**

---

## 11. Common errors

| Problem | Fix |
|---|---|
| `Connection refused` | Start Ollama (`ollama serve`) |
| `model "nomic-embed-text" not found` | `ollama pull nomic-embed-text` |
| Odd results after editing docs | Delete `chroma_db/` and re-run (collections only fill when empty) |
| `ModuleNotFoundError: rag_app` | Run from `novahome_eval/` and create `rag_app.py` first |
| `SyntaxError` after paste | Closing `EOF` must be alone on its line |

---

## 12. Learner challenge

Add a fifth doc that **contradicts** `policies.md` (e.g. a different express price), delete `chroma_db/`, re-run the comparison.

**Why is this an evaluation problem, not just retrieval?** Retrieval works perfectly: it finds relevant text. But the *content* conflicts, so the model may pick either answer. You need evaluation that checks correctness against a trusted reference, not just whether retrieval "worked."

---

## 13. 30-second summary to say aloud

> "RAG gives the model an open book. We split our documents into chunks, turn them into vectors, and store them in ChromaDB. When a question arrives, we find the closest chunks, paste them into a prompt, and tell the model to answer only from that context. We packaged this into one function that returns the answer plus the retrieved contexts and sources, because evaluation needs all three. Then we broke it on purpose: out-of-scope questions, answers spread across two documents, bad chunk sizes, and a loose prompt. Those failures are exactly what Lab 4 will measure."

---

## Checkpoint

- [ ] `rag_app.py` exists and returns `answer`, `contexts`, `sources`
- [ ] `outputs/lab3_plain_vs_rag.csv` exists
- [ ] You saw at least three failure types

Want me to explain Lab 4 (evaluating the RAG app with RAGAS) next?
