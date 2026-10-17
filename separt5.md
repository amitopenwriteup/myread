# Trainer Script · Part V: Doubt Resolution

**Slides 39–40 · T+2:45 – T+3:00 · 15 min**
Open Q&A (lab blockers first, then concepts) · Recap and next steps

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Lines marked *(pause)* are short waits. Before this part, collect the questions that came up in chat during the session. Answer **lab blockers first**, then **concept questions**.

---

## Slide 39 · Open Q&A (≈10 min)

We've reached the last fifteen minutes. This part is for **your** questions. We'll use ten minutes for questions, and five for a recap.

Here's how we'll do it. I've been collecting your questions in chat throughout the session. I'll answer them in two rounds. First, anything that **blocked you in the lab**, because those hold people up. Then **concept questions**, like metrics, biases or design choices.

**ASK:** Is anyone still stuck on the lab, for example a setup error or numbers that don't match? Type "lab" and the step number in chat. *(pause about 30 seconds and take these first)*

Thanks. Let me go through those now. *(answer lab questions. If you see a pattern, such as the same install error, fix it once for everyone.)*

Now the concept questions. **ASK:** What's one thing from today that you'd like explained again? *(pause and read out the chat questions in order of how many people raised them)*

If you need a quick way to answer, here's a short guide to the most likely questions.

- **"Which metric should I use?"** It depends on the user's goal. One right answer: MRR or precision at one. Everything relevant: recall at k. A clean list: precision at k or MAP. Graded relevance: nDCG.
- **"Why is my precision low even though the system seems good?"** Precision at k is capped when a query has fewer than k relevant items. Look at recall, MAP and MRR as well.
- **"Is BLEU or ROUGE enough for RAG?"** No. They measure overlap, not truth. Add groundedness checks, semantic similarity, LLM-as-judge calibrated on human labels, and human review on a sample.
- **"A RAG answer is wrong. Where do I start?"** Check retrieval first. Was the evidence retrieved? If no, fix retrieval. If yes, fix generation.
- **"How many queries do I need in a golden set?"** No fixed number. Start small, as in our lab, make sure it covers your intents, languages and topics, and then grow and version it over time.
- **"Can I use an LLM as a judge?"** Yes, with a clear rubric, and calibrated on human labels. Be aware it prefers longer and self-similar answers.

*(Use these only when a question matches. Read the answer in your own words if it helps.)*

**ASK:** Did that answer your question? If not, type "more" and I'll add detail. *(pause)*

---

## Slide 40 · Key takeaways and next steps (≈5 min)

Let's close with the main messages and where to go next.

On the left, five things to remember.

**One:** evaluate retrieval and generation **separately**. A bad answer can come from either one.

**Two:** always state **k and the dataset**. A score without them is hard to interpret.

**Three:** BLEU and ROUGE measure **overlap, not truth**. Use them as regression signals.

**Four:** **humans and users** remain the ground truth.

**Five:** watch **bias, drift and ethics** continuously, not once.

On the right, tools to explore for further practice.

For **retrieval metrics**: `ir_measures`, `pytrec_eval` and `ranx`.
For **text metrics**: `sacrebleu`, `rouge-score` and BERTScore.
For **RAG evaluation**: RAGAS, TruLens and ARES.
For **benchmarks**: BEIR, MTEB and the TREC tracks.
And for **today's workshop**: the file `rag_eval_workshop.md`, which you can re-run any time.

One small note. Tools change quickly, so please check the current documentation for each before you pick a specific version.

**ASK:** Before we finish, type one thing you will do differently when you evaluate a search or RAG system next. *(pause. Read out three or four answers.)*

**ASK:** Which tool from the list will you try first? *(pause and read out a few)*

Thank you all for your time and your questions today. If you think of something later, the lab file is the best place to start. Have a great rest of your day.

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask |
|---|---|---|---|
| 39 | Open Q&A | 10 min | Anyone stuck on the lab? What should I explain again? |
| 40 | Takeaways and tools | 5 min | One thing you will do differently? Which tool first? |

**Five takeaways:** separate retrieval and generation · state k and dataset · overlap is not truth · humans are ground truth · watch bias, drift, ethics
