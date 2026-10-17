# Trainer Script · Part III: Hands-On Lab

**Slides 25–30 · T+1:30 – T+2:15 · 45 min · Linux VM**
Data preparation: 10 min · Retrieval evaluation (P, R): 15 min · Generation evaluation (BLEU, ROUGE): 20 min, including the 5-min wrap-up (slide 30)

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Lines marked **CHECKPOINT** are moments to have everyone paste a result in chat before moving on. Lines marked *(pause)* are short waits. Everyone follows `rag_eval_workshop.md` on a Linux VM. Before starting, make sure `pip install` worked for everyone.

---

## Slide 25 · Part III Divider (≈1 min)

Now we get hands-on. For the next forty-five minutes, we stop talking about metrics and actually compute them.

The plan: first, ten minutes to set up and look at the data. Then fifteen minutes on retrieval evaluation, with precision and recall. Then twenty minutes on generation evaluation, with BLEU and ROUGE.

Everyone is working on a Linux VM, and everyone follows the same file, `rag_eval_workshop.md`. If you get stuck at any point, type "stuck" in chat and say which step. We'll sort it out together.

**ASK:** Is everyone connected to the VM? Type "ready" in chat when your terminal is open. *(pause until most people confirm)*

---

## Slide 26 · Lab setup (≈5 min)

Let's set up. You need Python 3.9 or higher, about 2 GB of RAM, and internet access for pip.

First, create a virtual environment and activate it. The command is:

`python3 -m venv .venv && source .venv/bin/activate`

Then install the libraries:

`pip install numpy scikit-learn sacrebleu rouge-score`

There's also an optional line for sentence-transformers. Install it if you want to try embeddings in the exercises later. It's heavier, so skip it if your machine is slow.

You can copy these commands exactly as written from the markdown file. Remote participants, please paste them from there.

Now look at the project layout. There's a **data** folder with the corpus, queries, references and generated answers. A **src** folder with four scripts: retrieve, eval_retrieval, generate_baseline and eval_generation. And a **runs** folder for rankings and generated answers.

The dataset is small on purpose: **15 chunks and 12 queries.** It's small enough that you can verify the results by hand.

**CHECKPOINT:** Type "installed" in chat once `pip install` finishes without errors. If you see an error, paste the last two lines of it. *(pause and help anyone who is stuck before moving on)*

---

## Slide 27 · The four data assets (≈5 min)

Before we run anything, let's understand the data. Four assets drive every metric we compute today.

First, **reference chunks**, in `corpus.json`. These are 15 knowledge-base chunks about Linux administration. This is the search space.

Second, **relevance labels**, in `queries.json`. It has 12 queries, and for each one, the chunk IDs that are relevant. This is the ground truth for retrieval.

Third, **reference answers**, in `references.json`. One human-written answer per query. This is the ground truth for generation.

Fourth, **generated texts**: `generated_llm.json`, plus an extractive baseline built from the retrieval output. These are the systems under test.

An easy way to remember it. Reference chunks answer "what should be found." Reference answers answer "what should be said." And generated texts are what the system actually produced.

And one important point: labels are created **before** systems are compared. Otherwise we'd be biased by what the system gave us.

**ASK:** Open `queries.json` and look at query one. Which chunk IDs are marked relevant? Paste them in chat. *(pause and read out a few)*

**ASK:** Open `corpus.json`. Is the relevant chunk what you expected for that query? *(pause. Let two or three people respond.)*

---

## Slide 28 · Retrieval evaluation (≈15 min)

Now let's run retrieval. There are two commands.

First, build the ranking with TF-IDF:

`python src/retrieve.py --method tfidf`

This ranks every chunk for every query and saves the result in the runs folder.

Second, score it:

`python src/eval_retrieval.py --run runs/tfidf.json --k 3 --verbose`

The `--k 3` means we evaluate the top three. And `--verbose` prints the results per query.

Go ahead and run both. *(pause for two or three minutes)*

**CHECKPOINT:** Paste your **Precision at 3** in chat. *(pause and compare. Everyone should see the same numbers.)*

You should see these four numbers: **Precision at 3 is 0.361. Recall at 3 is 0.875. MAP is 0.909. MRR is 1.000.**

Recall is high, MAP is high, MRR is perfect. So the right chunk is nearly always found, and often at rank one. But precision is only 0.361. Why?

Here's the explanation. Most queries have **one** relevant chunk. If you look at the top three, you can have at most one relevant result out of three, so precision at three is **capped at one third**. Precision depends on how many relevant items exist.

Now scroll through the verbose output. Look at queries **q09, q11 and q12**. That's where average precision drops. MRR is 1.0, so the first relevant result is always at rank one. The drop comes from other relevant chunks that were ranked lower or missed.

**ASK:** Why is a precision of 0.361 not a sign of a bad system here? *(pause. Answer: most queries have only one relevant chunk, so the maximum possible P@3 is about 0.33 for them.)*

**ASK:** Look at q09, q11 and q12. Which relevant chunks were ranked low or missed, and why? Compare the words in the query with the words in those chunks. *(pause. Let people explore. A likely theme is little word overlap, which is where a lexical method struggles.)*

Good observation. We'll test that idea in the exercises at the end.

---

## Slide 29 · Generation evaluation (≈15 min)

Now we move from retrieval to generation. We'll do the toy example first, then both systems.

Start with the toy example, the same cat and mat sentences from the slides:

`python src/toy_metrics.py`

Run it. *(pause two minutes)*

**CHECKPOINT:** Do your numbers match the slide? BLEU-4 without smoothing should be zero, and smoothed BLEU around 0.38. ROUGE-1 should be 0.833. Type "match" in chat if they do.

Good. Now the real systems. First, build the baseline answers from the TF-IDF ranking:

`python src/generate_baseline.py runs/tfidf.json`

Then score the baseline:

`python src/eval_generation.py --gen runs/generated_baseline.json`

And score the LLM-style answers:

`python src/eval_generation.py --gen data/generated_llm.json`

Run all three. *(pause about five minutes)*

**CHECKPOINT:** Paste your **ROUGE-L** for both systems in chat. *(pause)*

You should see this. The copy-the-top-chunk baseline: BLEU 0.104, ROUGE-1 0.488, ROUGE-2 0.293, ROUGE-L 0.437. The LLM-style answers: BLEU 0.171, ROUGE-1 0.603, ROUGE-2 0.270, ROUGE-L 0.500.

Now let's discuss. There are three questions on the slide.

**ASK:** Is the copy-paste baseline really better? It wins on ROUGE-2. Why do you think that happens? *(pause. Answer: it copies text directly from the source, so it reproduces exact word pairs from the reference. The LLM paraphrases, so exact bigrams are lost even when the meaning is right.)*

**ASK:** Open the generated answers. Can you find one that is correct but has a low score? Paste the query ID in chat. *(pause and let people look)*

**ASK:** What would these metrics miss? For example, a hallucination or an omission. *(pause. Answer: a single wrong word in an otherwise matching answer barely changes the score. Missing a key fact only slightly lowers recall.)*

This links back to the limits slide in Part II. Overlap is not understanding.

---

## Slide 30 · Exercises and stretch goals (≈5 min)

To wrap up the lab, here are four exercises. If we're short on time, they can be done in breakouts or as homework. Details are in the markdown under each block.

**Exercise one: semantic versus lexical.** Run retrieval again with `--method embed`, and compare per-query results with TF-IDF. Which queries improved?

**Exercise two: add a hard query.** Write a query that has no word overlap with its answer, like our cron example, and compare both retrievers.

**Exercise three: wrong but similar.** In one of the answers, change `kill -9` to `kill -1`. The answer is now wrong, but looks almost the same. How much do the scores move? I expect you'll be surprised how little.

**Exercise four: hybrid plus a real LLM.** Blend TF-IDF with embeddings, or generate answers with a local model through Ollama.

**ASK:** Which exercise would you like to try first? Type 1, 2, 3 or 4 in chat. *(pause, then note the popular one)*

Great. Please give your VM a rest for now. In the next section we'll move away from the lab and talk about the challenges of measuring all this.

---

### Trainer quick reference

| Slide | Topic | Time | Checkpoint / question |
|---|---|---|---|
| 25 | Part III divider | 1 min | Type "ready" when connected |
| 26 | Setup | 5 min | Type "installed" |
| 27 | Four data assets | 5 min | Which chunks are relevant for query 1? |
| 28 | Retrieval evaluation | 15 min | Paste P@3. Expect 0.361 / R@3 0.875 / MAP 0.909 / MRR 1.000 |
| 29 | Generation evaluation | 15 min | Paste ROUGE-L. Baseline 0.437, LLM-style 0.500 |
| 30 | Exercises | 5 min | Which exercise first? |

**Key numbers to keep at hand**

| System | BLEU | ROUGE-1 | ROUGE-2 | ROUGE-L |
|---|---|---|---|---|
| Copy top chunk | 0.104 | 0.488 | 0.293 | 0.437 |
| LLM-style | 0.171 | 0.603 | 0.270 | 0.500 |
