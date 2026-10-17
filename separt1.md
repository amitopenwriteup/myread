# Trainer Script · Part I: Search Systems Refresher

**Slides 3–11 · T+0:00 – T+0:30 · 30 min**
Semantic search: 10 min (slides 4–7) · Generative search / RAG: 20 min (slides 8–11)
# Trainer Script · Part I: Search Systems Refresher (with Newbie Examples)

**Slides 3–11 · T+0:00 – T+0:30 · 30 min**
Semantic search: 10 min (slides 4–7) · Generative search / RAG: 20 min (slides 8–11)

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Pause and let them answer in chat or on mic. Lines marked *(pause)* are short waits.
>
> **New in this version:** Each slide has a box called **🟢 Newbie example**. Use it right after the main explanation, or whenever you see confused faces (or silence in chat). Two pictures run through the whole script:
> - **The Library 📚** for search (slides 4–7)
> - **The Open-Book Exam 📝** for RAG (slides 8–11)

---

## Slide 3 · Part I Divider (≈1 min)

Alright, let's begin with Part I. This is a refresher on search systems, and it runs for the first thirty minutes.

Why start here? Because before we can talk about whether a search system is *good*, we all need the same vocabulary for what a search system *is*. Everything in the evaluation parts later today depends on this.

We'll split the half hour in two. The first ten minutes cover semantic search. The next twenty cover generative search, also called RAG.

**ASK:** Quick show of hands in chat. Who has built or worked on a search system before? Just type "yes" or "no". *(pause)*

Great, that gives me a feel for the room. Let's go.

> 🟢 **Newbie example: "You already use search every day"**
> When you type *"pizza near me"* in Google Maps, that is a search system. When you ask a chatbot *"what is our leave policy?"* and it writes an answer from your company documents, that is generative search (RAG).
> Today we are not learning how to *build* these. We are learning how to tell **whether they are good**, like a teacher checking whether a student's answers are right.

---

## Slide 4 · A search system is a pipeline (≈2 min)

Here's the first idea. A search system is not one thing. It's a pipeline of five stages.

The user types a **query**. It's usually short and often vague. Then the system **understands** it: it cleans it up, expands it, and works out what the person really wants. Next it **retrieves** candidate documents or chunks. Then it **ranks** them by how relevant it thinks they are. Finally it **presents** the result: snippets, an answer, citations.

Now look at the row underneath. Every one of these stages has its own way of failing.

If the query is ambiguous, we start off wrong. If the system misreads the intent, or misses a synonym, the understanding stage fails. If a relevant item is never retrieved, that's a *recall* problem. If it is retrieved but ranked too low, that's a *precision* problem. And if the snippet is misleading, the user gets the wrong answer even though everything before it worked.

The key point is this: when we evaluate, we must be able to point at the stage that failed. A single score that says "the system is bad" doesn't help anyone fix it.

**ASK:** Think of the last time a search gave you a bad result. Which of these five stages do you think failed? Type the stage name in chat. *(pause, read out two or three answers)*

> 🟢 **Newbie example: "The library 📚"**
> You walk up to a librarian and say: *"I want a book on saving money."*
>
> | Stage | What the librarian does | What can go wrong |
> |---|---|---|
> | **Query** | Listens to you | You were vague: money for what? |
> | **Understand** | Thinks "saving money, so budgeting or personal finance" | Misreads you and goes to the *investing* shelf |
> | **Retrieve** | Pulls 20 books off the shelves | The best book never gets pulled (**recall problem**) |
> | **Rank** | Stacks them best-first | The best book is at the *bottom* of the stack (**precision problem**) |
> | **Present** | Shows you the covers and a one-line summary | The summary is misleading, so you reject the right book |
>
> **Lesson:** If you only say "the librarian was bad", nobody knows what to fix. If you say "she pulled the right book but stacked it at the bottom", you know exactly where the problem is.

---

## Slide 5 · Lexical vs semantic (≈3 min)

Now let's look at how retrieval actually matches a query to a document. There are two big families.

On the left is **lexical** search. Think BM25 and TF-IDF. It counts shared words and gives more weight to rare ones. It's fast, it's exact, and it's easy to explain. It's excellent for IDs, names and error codes. But it misses paraphrases and synonyms.

On the right is **semantic** search. It uses dense embeddings. It turns text into vectors and compares them by angle. That lets it find paraphrases and related ideas, and it can work across different wording and even different languages. The downside is that it can retrieve things that look plausible but are wrong.

Let me make that concrete with the example at the bottom. The query is "run a script every night". The chunk says "cron runs scheduled jobs". Look at the two sentences. They share no words at all. A lexical system scores that as zero overlap. But a semantic system sees that they mean almost the same thing, so the similarity is high.

We'll use this exact cron example again in the lab, so keep it in mind.

**ASK:** For which kind of query would you still prefer lexical search? Give me an example. *(pause. Good answers: product codes, error messages, exact names.)*

Right. Neither approach wins everywhere, and that's why we'll combine them in a moment.

> 🟢 **Newbie example: "Two kinds of librarian"**
>
> Query: **"a book that teaches me to save money"**
> Book on the shelf: **"Personal Budgeting Guide"**
>
> | Librarian | How they think | Result |
> |---|---|---|
> | **Lexical librarian** (matches *words*) | Looks for the words "save" and "money" on the book title. Not there. | ❌ Misses the book |
> | **Semantic librarian** (matches *meaning*) | Understands "save money" and "budgeting" mean the same thing. | ✅ Finds it |
>
> But flip it. Query: **"Error E1043"**.
> The lexical librarian finds the exact page with "E1043". The semantic librarian may also hand you pages for E1044 and E1045, because they *look* similar. ❌
>
> **One line to remember:** *Lexical = match the words. Semantic = match the meaning. Each has a weakness the other covers.*

---

## Slide 6 · How semantic search works (≈2 min)

Let's see how semantic search works under the hood. There are two paths: one offline, one online.

The **offline** path builds the index. We **chunk**: we split documents into pieces. We **embed**: each piece becomes a vector. And we **index**: we store those vectors in a vector store, usually with approximate nearest-neighbour search so it's fast.

The **online** path happens when a user asks something. We **embed the query**, using the *same model* as before. That part is important. Then we find the **nearest neighbours**: the top-k chunks by similarity. And we return a **ranked list** with scores and chunk IDs.

The formula at the bottom is cosine similarity. It's the dot product of the two vectors divided by the product of their lengths. The result goes from minus one to one, and higher means closer in meaning. One thing to notice: cosine compares *direction*, not length.

**ASK:** Why do we have to use the same embedding model for the query and the documents? *(pause. Answer: otherwise the vectors live in different spaces and the comparison means nothing.)*

> 🟢 **Newbie example: "A map of meaning 🗺️"**
>
> Imagine every sentence is a **pin on a map**. Sentences with **similar meaning are placed close together**. Sentences with different meaning are far apart.
>
> ```
>   "save money" ●─● "budgeting guide"        ← close (same meaning)
>
>                              ● "cricket score"   ← far away
> ```
>
> - **Offline (done once):** The librarian reads *every* book and puts a pin on the map. This is *chunk → embed → index*.
> - **Online (every search):** When you ask something, she puts a pin for *your question* on the same map and looks at the **nearest pins**. These are your top-k results.
> - **Cosine similarity** is just "how close is my pin to that pin?" (1 = almost the same spot, 0 = unrelated).
>
> **Why the same model for both?** If books were pinned on a *Mumbai* map and your question on a *London* map, "close" would mean nothing. Same map = same embedding model.

---

## Slide 7 · Hybrid retrieval and re-ranking (≈2 min)

In real systems, people rarely pick just one method. Three practical ideas show up again and again.

First, **hybrid retrieval**. We run BM25 and dense retrieval in parallel, then merge the two lists. A simple way to merge is reciprocal rank fusion, or RRF. That way we cover both exact terms and paraphrases.

Second, **re-ranking**. A cross-encoder reads the query and the chunk *together* and rescores the top fifty to a hundred candidates. It's slower, but it gives much better precision at the top of the list.

Third, and this is why we're here today, **evaluate each stage**. Measure the recall of the first stage: did we fetch the right thing at all? And measure the precision of the final top-k: did we actually show it? One number hides which stage is weak.

**ASK:** If a system has high recall at the first stage but poor precision at the top of the final list, which stage would you look at first? *(pause. Answer: the re-ranker or the ranking step.)*

Good. That kind of thinking is exactly what we'll practise in the evaluation parts.

> 🟢 **Newbie example: "Two friends and a hiring round"**
>
> **Hybrid = ask two friends.**
> You ask two friends to recommend a restaurant. One matches your *exact words* (BM25), the other understands *what you mean* (dense). You merge their lists.
>
> | Restaurant | Friend A rank | Friend B rank |
> |---|---|---|
> | Spice Hub | 1 | 2 |
> | Pizza Point | 2 | 5 |
> | Dosa Corner | 6 | 1 |
>
> Spice Hub is high on **both** lists, so it ends up first. That is the idea of RRF: *if both friends like it, it wins.*
>
> **Re-ranking = the final interview.**
> 1000 resumes arrive. A quick screen picks 50 (fast, rough). Then you interview those 50 carefully (slow, accurate) and pick the best 5. You don't interview all 1000, because it would take forever.
>
> **Evaluate each stage = find who made the mistake.**
> The ideal candidate is "Asha".
>
> | What happened | Which stage is weak? |
> |---|---|
> | Asha was **not even in the 50** shortlisted | Stage 1 (**recall** is low) |
> | Asha was in the 50, but ended up **9th** in the final list | Re-ranking (**precision** at the top is low) |
>
> That is exactly the answer to the ASK: *high recall, poor top precision → look at the re-ranker.*

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

> 🟢 **Newbie example: "The open-book exam 📝"**
>
> You are allowed to open your textbook in the exam.
>
> | RAG step | In the exam |
> |---|---|
> | **Question** | "How many leaves does an employee get?" |
> | **Retriever** | You flip to the HR-policy pages (top-k chunks) |
> | **Prompt** | Rule: "Answer **only** from these pages" |
> | **LLM** | You write the answer in your own words |
> | **Answer + citation** | "18 days per year (HR Policy, page 4)" |
>
> **Why the four reasons make sense:**
> 1. **Fresher:** Policy changed? Swap the page in the textbook. No need to re-study everything.
> 2. **Grounded:** You can point to the page that backs your answer.
> 3. **Private data:** It works with your company's own book, which no public AI has read.
> 4. **⚠️ New risk:** You write so smoothly that the examiner *believes* you, even if you wrote "20 days" when the page says "18". A confident, well-written wrong answer is the hardest to catch. That is why we need evaluation.

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

> 🟢 **Newbie example: "Preparing for the open-book exam"**
>
> | Knob | In the exam analogy |
> |---|---|
> | **Chunking** | How you cut your notes into slips. One giant slip = too much to read. Tiny slips = a sentence loses its context ("it increases by 10%": *what* increases?). |
> | **Embedding model** | The *index* you use to find pages. A Hindi textbook needs a Hindi-aware index. |
> | **k and re-ranking** | How many pages you bring to the desk. 3 pages may miss it; 30 pages bury you in noise (and waste time). |
> | **Prompt rules** | The exam instructions: "answer only from the book", "give page numbers", "write 'not found' if unsure". |
> | **Generator model** | *Who* writes the answer. A strong student writes better, but takes longer and costs more. |
> | **Guardrails** | A safety rule: "if the book has no answer, don't guess. Say 'I don't know'." |
>
> **Quick matching for the two ASK questions:**
> - Answers feel **cut off** → the slips are too small, or you brought too few pages (**chunking / k**).
> - Model **invents things** → the instructions are too loose, or there is no "don't guess" rule (**prompt rules / guardrails**).
>
> **Why evaluate?** Suppose you change the chunk size and the answers *feel* better. Without a score on the same set of test questions, you can't tell whether it really improved or you just got lucky.

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

> 🟢 **Newbie example: "Why did I get the answer wrong in the exam?"**
>
> **Question:** *"What is the notice period for resignation?"*  
> **Correct answer in the book:** *60 days (page 12).*
>
> | Case | What happened | Where is the problem? |
> |---|---|---|
> | **A** | You opened the **wrong page** (page 7, about leave) and wrote "30 days" | 🔍 **Retrieval**: the right page never reached your desk. Better handwriting won't help. |
> | **B** | You opened the **right page** (page 12, which says 60) but wrote "30 days" | ✍️ **Generation**: the evidence was there, but you misread or invented. |
>
> **Same wrong answer, two different cures.**
>
> | Failure type | Everyday picture |
> |---|---|
> | Right chunk not in top-k | The page you needed is not on your desk |
> | Noisy chunks | Your desk is full of unrelated pages |
> | Stale documents | Your textbook is last year's edition |
> | Answer split across chunks | Half the answer is on page 12, half on page 40, and you only opened one |
> | Hallucination | You write something that is not in the book |
> | Ignores evidence | The right page is open, and you don't read it |
> | Wrong citation | You write "page 4" but the fact was on page 12 |
>
> **Habit to build:** Always first ask: *"Was the right page on the desk?"*

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

> 🟢 **Newbie example: "Checking a booklist vs checking an essay"**
>
> - **Search = the librarian hands you a list of 5 books.** A teacher checks *each book*: relevant ✅ or not ❌. Easy to count. That is why we can use **Precision, Recall, MAP, MRR**.
> - **RAG = the student writes a full paragraph.** The teacher now has to ask: Is it *correct*? Is it *backed by the book*? Is anything *missing*? Harder to check, so we use **BLEU/ROUGE** (compare with a model answer), **human ratings**, and **groundedness** checks.
>
> | | Search | RAG |
> |---|---|---|
> | What we get | List of books | A written paragraph |
> | Teacher's question | "Is each book relevant?" | "Is the whole answer right and supported?" |
> | Answer key needed | Which books are relevant (labels) | A model answer or marking rubric |
> | Typical mistake | Right book missing or at the bottom | Smooth-sounding but wrong sentence |

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask | One-line newbie picture |
|---|---|---|---|---|
| 3 | Part I divider | 1 min | Who has built a search system? | "pizza near me" is search |
| 4 | Pipeline & failure points | 2 min | Which stage failed in your bad search? | Librarian: listen, understand, pull, stack, show |
| 5 | Lexical vs semantic | 3 min | When would you still prefer lexical? | Match words vs match meaning |
| 6 | Embed, index, compare | 2 min | Why the same embedding model? | Pins on one map |
| 7 | Hybrid + re-ranking | 2 min | High recall, poor top precision: where to look? | Two friends, then a final interview |
| 8 | What is RAG | 4 min | Confident but wrong AI answer you've seen? | Open-book exam |
| 9 | Six design choices | 5 min | Which knob for cut-off answers / made-up answers? | How you prepare for the exam |
| 10 | How RAG fails | 5 min | Case A vs Case B: where is the fix? | Wrong page vs misread page |
| 11 | Bridge to Part II | 3 min | What is still unclear? | Checking a booklist vs an essay |

---

### Tips for using the newbie examples

1. **Don't read every box.** Pick the one that matches the room: if people look lost, use it; if they nod, skip it and save time.
2. **Let participants extend the analogy.** Ask: "In the library picture, what would a *re-ranker* be?" (A senior librarian who re-checks the top 20 books.)
3. **Keep the two pictures consistent.** Library = *finding* things (slides 4–7). Open-book exam = *answering* from what was found (slides 8–11). It is the same story, in two parts.
4. **Time check:** If you are running behind, skip the example boxes on slides 6 and 9 first; they are the least critical.
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
