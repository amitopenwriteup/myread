# Trainer Script · Part I: Search Systems Refresher

**Slides 3–11 · T+0:00 – T+0:30 · 30 min**
Semantic search: 10 min (slides 4–7) · Generative search / RAG: 20 min (slides 8–11)

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Pause and let them answer in chat or on mic. Lines marked *(pause)* are short waits.

---

## Slide 3 · Part I Divider (≈1 min)

Alright, let's begin with Part I. This is a refresher on search systems, and it runs for the first thirty minutes.

Why start here? Because before we can talk about whether a search system is *good*, we all need the same vocabulary for what a search system *is*. Everything in the evaluation parts later today depends on this.

We'll split the half hour in two. The first ten minutes cover semantic search. The next twenty cover generative search, also called RAG.

**ASK:** Quick show of hands in chat. Who has built or worked on a search system before? Just type "yes" or "no". *(pause)*

Great, that gives me a feel for the room. Let's go.

---

## Slide 4 · A search system is a pipeline (≈2 min)

Here's the first idea. A search system is not one thing. It's a pipeline of five stages.

The user types a **query**. It's usually short and often vague. Then the system **understands** it: it cleans it up, expands it, and works out what the person really wants. Next it **retrieves** candidate documents or chunks. Then it **ranks** them by how relevant it thinks they are. Finally it **presents** the result: snippets, an answer, citations.

Now look at the row underneath. Every one of these stages has its own way of failing.

If the query is ambiguous, we start off wrong. If the system misreads the intent, or misses a synonym, the understanding stage fails. If a relevant item is never retrieved, that's a *recall* problem. If it is retrieved but ranked too low, that's a *precision* problem. And if the snippet is misleading, the user gets the wrong answer even though everything before it worked.

The key point is this: when we evaluate, we must be able to point at the stage that failed. A single score that says "the system is bad" doesn't help anyone fix it.

**ASK:** Think of the last time a search gave you a bad result. Which of these five stages do you think failed? Type the stage name in chat. *(pause, read out two or three answers)*

---

## Slide 5 · Lexical vs semantic (≈3 min)

Now let's look at how retrieval actually matches a query to a document. There are two big families.

On the left is **lexical** search. Think BM25 and TF-IDF. It counts shared words and gives more weight to rare ones. It's fast, it's exact, and it's easy to explain. It's excellent for IDs, names and error codes. But it misses paraphrases and synonyms.

On the right is **semantic** search. It uses dense embeddings. It turns text into vectors and compares them by angle. That lets it find paraphrases and related ideas, and it can work across different wording and even different languages. The downside is that it can retrieve things that look plausible but are wrong.

Let me make that concrete with the example at the bottom. The query is "run a script every night". The chunk says "cron runs scheduled jobs". Look at the two sentences. They share no words at all. A lexical system scores that as zero overlap. But a semantic system sees that they mean almost the same thing, so the similarity is high.

We'll use this exact cron example again in the lab, so keep it in mind.

**ASK:** For which kind of query would you still prefer lexical search? Give me an example. *(pause. Good answers: product codes, error messages, exact names.)*

Right. Neither approach wins everywhere, and that's why we'll combine them in a moment.

---

## Slide 6 · How semantic search works (≈2 min)

Let's see how semantic search works under the hood. There are two paths: one offline, one online.

The **offline** path builds the index. We **chunk**: we split documents into pieces. We **embed**: each piece becomes a vector. And we **index**: we store those vectors in a vector store, usually with approximate nearest-neighbour search so it's fast.

The **online** path happens when a user asks something. We **embed the query**, using the *same model* as before. That part is important. Then we find the **nearest neighbours**: the top-k chunks by similarity. And we return a **ranked list** with scores and chunk IDs.

The formula at the bottom is cosine similarity. It's the dot product of the two vectors divided by the product of their lengths. The result goes from minus one to one, and higher means closer in meaning. One thing to notice: cosine compares *direction*, not length.

**ASK:** Why do we have to use the same embedding model for the query and the documents? *(pause. Answer: otherwise the vectors live in different spaces and the comparison means nothing.)*

---

## Slide 7 · Hybrid retrieval and re-ranking (≈2 min)

In real systems, people rarely pick just one method. Three practical ideas show up again and again.

First, **hybrid retrieval**. We run BM25 and dense retrieval in parallel, then merge the two lists. A simple way to merge is reciprocal rank fusion, or RRF. That way we cover both exact terms and paraphrases.

Second, **re-ranking**. A cross-encoder reads the query and the chunk *together* and rescores the top fifty to a hundred candidates. It's slower, but it gives much better precision at the top of the list.

Third, and this is why we're here today, **evaluate each stage**. Measure the recall of the first stage: did we fetch the right thing at all? And measure the precision of the final top-k: did we actually show it? One number hides which stage is weak.

**ASK:** If a system has high recall at the first stage but poor precision at the top of the final list, which stage would you look at first? *(pause. Answer: the re-ranker or the ranking step.)*

Good. That kind of thinking is exactly what we'll practise in the evaluation parts.

---

## Slide 8 · RAG: retrieve, then generate (≈4 min)

Now let's move to generative search. This is the second half of Part I, and we'll spend twenty minutes here.

The idea is called **RAG**, retrieval-augmented generation. The flow is simple. A **question** comes in. A **retriever** fetches the top-k chunks. We build a **prompt** that contains the question, the evidence, and some rules. An **LLM** writes the answer from that evidence. And the **answer** comes back, ideally with citations.

Why do people do this? Four reasons are on the slide.

One, **fresher knowledge**. You update the index, not the model.
Two, **grounded answers**. The evidence is right there in the prompt, so it can be cited.
Three, **private data**. You can use internal documents without retraining anything.

And the fourth one is marked as a *new risk*. Fluent text can hide wrong or unsupported claims. A generated answer *sounds* confident even when it's not supported by the evidence. That's the main reason evaluating RAG is harder than evaluating classic search.

**ASK:** Have you seen an AI assistant give a confident answer that turned out to be wrong? What happened? Share one example in chat. *(pause, read out one or two)*

That's the risk we'll learn to measure.

---

## Slide 9 · Six design choices (≈5 min)

Every RAG system is built from choices, and each choice moves your evaluation numbers. Here are six of them. Think of each as a knob.

**Chunking.** Size, overlap and structure. If chunks are too big, the meaning gets diluted. If they're too small, you lose context.

**Embedding model.** Domain fit and language coverage have a big impact on recall.

**k and re-ranking.** Retrieving more chunks raises recall, but it also adds noise and cost.

**Prompt rules.** Instructions like "use only the context", "cite your sources", and "say when you're unsure".

**Generator model.** Its size, decoding temperature and context length.

**Guardrails.** Abstain when the evidence is weak, and filter unsafe output.

Here's the main message. Evaluation is how you know whether turning a knob actually helped. Without it, you're guessing. In today's lab we'll turn two of these knobs: the retrieval method and the generator.

**ASK:** If answers feel "cut off" or missing context, which of these six knobs would you suspect first? *(pause. Likely answers: chunking, or k.)*

**ASK:** And if the model keeps making things up beyond the documents? *(pause. Likely answers: prompt rules, guardrails.)*

---

## Slide 10 · How RAG fails (≈5 min)

Now the most useful slide in this section: how RAG fails.

There are two buckets.

**Retrieval failures.** The relevant chunk isn't in the top-k. Or noisy, off-topic chunks got in. Or the documents are stale or missing. Or the answer is split across several chunks and we only got part of it.

**Generation failures.** The model hallucinates, which means it makes unsupported claims. Or it ignores the evidence it was given. Or the answer is incomplete or too wordy. Or the citation or format is wrong.

And here's the diagnostic question I want you to remember. *Was the evidence retrieved?*

If the answer is **no**, fix retrieval. If the answer is **yes**, fix generation.

A wrong answer is not always the LLM's fault. If the right chunk was never retrieved, no amount of prompt tuning will fix it. So we always check retrieval first.

**ASK:** Let me give you two cases. Case A: the correct paragraph was not in the retrieved chunks, and the model gave a wrong answer. Case B: the correct paragraph *was* retrieved, but the model still gave a wrong answer. Where do you fix each one? *(pause)*

Case A is a retrieval problem. Case B is a generation problem. Same symptom, different cure.

---

## Slide 11 · Bridge to Part II (≈3 min)

Let me close Part I with a bridge to what comes next. This table is our roadmap for the next hour.

Look at the first row: output. Semantic search gives you a **ranked list** of documents or chunks. Generative search gives you a **written answer**, often with sources.

That changes what we judge. For search, we judge **each result**: is it relevant or not? For RAG, we judge the **whole answer**: is it correct, grounded and complete?

It also changes the ground truth. For search, we need **relevance labels per query**. For RAG, we need a **reference answer or a human rubric**.

So the metrics differ. For ranked lists we'll use Precision, Recall, F1, MAP and MRR. For generated answers we'll use BLEU, ROUGE, human ratings and groundedness.

And the risk is different too. Search can *miss* or *mis-rank* results. RAG can produce text that is *fluent but wrong or unsupported*.

**ASK:** Before we move on: what's one thing from Part I that is still unclear? Type it in chat, and we'll pick it up in the doubt session at the end. *(pause 30 seconds)*

Thank you. That sets our shared vocabulary. Next, Part II: the standard methods for evaluating all of this. Let's start with how to measure a ranked list.

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask |
|---|---|---|---|
| 3 | Part I divider | 1 min | Who has built a search system? |
| 4 | Pipeline & failure points | 2 min | Which stage failed in your bad search? |
| 5 | Lexical vs semantic | 3 min | When would you still prefer lexical? |
| 6 | Embed, index, compare | 2 min | Why the same embedding model? |
| 7 | Hybrid + re-ranking | 2 min | High recall, poor top precision: where to look? |
| 8 | What is RAG | 4 min | Confident but wrong AI answer you've seen? |
| 9 | Six design choices | 5 min | Which knob for cut-off answers / made-up answers? |
| 10 | How RAG fails | 5 min | Case A vs Case B: where is the fix? |
| 11 | Bridge to Part II | 3 min | What is still unclear? |
