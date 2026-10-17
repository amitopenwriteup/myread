# Trainer Script · Part II: Standard Methods for Evaluating Search Systems

**Slides 12–24 · T+0:30 – T+1:30 · 60 min**
Relevance metrics (P, R, F1, MAP): 20 min · Manual / task-oriented evaluation: 10 min · Generation metrics (BLEU, ROUGE): 30 min

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Pause and let them answer in chat or on mic. Lines marked *(pause)* are short waits. Where a calculation is shown, give people a few seconds to try it before you reveal the answer.

---

## Slide 12 · Part II Divider (≈1 min)

Welcome to Part II. This is the theory core of today, and it runs for one hour.

We'll cover three things. First, relevance metrics: Precision, Recall, F1 and MAP, for judging a ranked list. Second, manual and task-oriented evaluation, where humans and real users tell us what numbers can't. Third, generation metrics, BLEU and ROUGE, for judging written answers.

One thing that will make this easier. We use just **one running ranked list** and **one running sentence pair** throughout. You'll see both again in the lab, computed by code. So you don't need to keep learning new examples. You only need to learn these two well.

**ASK:** Before we start, which metric names from today's agenda have you already used? Type them in chat. *(pause)*

---

## Slide 13 · Two layers, two modes (≈3 min)

Let's start with the big picture. We evaluate a RAG system in **two layers**, and then **end to end**.

The **retrieval layer** is judged against relevance labels: did we find the right chunks? The **generation layer** is judged against reference answers: did we write the right answer? And **end to end**, it's judged by humans and users: did this actually help?

There are also two **modes**. **Offline evaluation** uses a test collection: a fixed set of queries, labelled relevant chunks, and reference answers. It's repeatable and fast, and it's great for regression tests. **Online evaluation** uses real users: clicks, task success, A/B tests. It's more realistic, but it's slower and noisier, and you need traffic.

We start with offline, because it's repeatable and cheap. You can run it every time you change something.

**ASK:** You've just changed the chunk size. Would you check that offline or online first? Why? *(pause. Answer: offline, because it's fast and repeatable. Online comes later for confirmation.)*

---

## Slide 14 · Precision and recall (≈5 min)

Now our running example. Look at the ten boxes. They're a ranked list: rank one on the left, rank ten on the right. The teal boxes are relevant results.

Here's the setup. There are **four** relevant chunks in total. Our system found three of them, at ranks **1, 3 and 6**. The fourth, called d11, was **never retrieved at all**. Now we look only at the **top five**.

Two questions. How clean is what we show? And how much did we find?

**Precision at k** answers the first one. It's the number of relevant items in the top k, divided by k. In the top five we have two relevant ones, so precision at five is 2 out of 5, which is **0.40**.

**Recall at k** answers the second. It's the number of relevant items in the top k, divided by *all* relevant items that exist. That's 2 out of 4, so recall at five is **0.50**.

Same list, two different views. Precision looks at what we showed. Recall looks at what we could have found.

**ASK:** What would precision at ten and recall at ten be for this same list? Work it out and type it in chat. *(pause about 30 seconds)*

The top ten contains three relevant results. So precision at ten is 3 out of 10, which is 0.30. Recall at ten is 3 out of 4, which is 0.75. Notice what happened: recall went up, precision went down.

---

## Slide 15 · F1 and the cut-off k (≈4 min)

That leads straight to this slide. Look at the chart. As k grows from one to ten, the recall line can only go up or stay flat. Precision tends to fall.

So how do we get one number? That's **F1**. It's two times P times R, divided by P plus R. It's the **harmonic mean**, which means it's low if *either* precision or recall is low. You can't hide a weak one behind a strong one.

For our running example, at k equals five: P is 0.40 and R is 0.50, so F1 is **0.444**.

And look at the line at the bottom: **report k with every score.** A score of 0.4 means nothing unless I know whether it was measured at top three, top five or top ten.

**ASK:** If a teammate says "our precision is 0.9", what is the first thing you'd ask them? *(pause. Answer: at what k, and on which dataset.)*

---

## Slide 16 · Rank-aware metrics (≈4 min)

So far, precision at five treats ranks one and five the same. But users care a lot about order. Rank-aware metrics fix that.

First, **Average Precision, or AP**. For each relevant result we find, we take the precision at that rank. Then we divide by the total number of relevant items, including the one we missed. For our list: precision at rank 1 is 1 out of 1. At rank 3, it's 2 out of 3. At rank 6, it's 3 out of 6. Add those up and divide by 4 relevant items. We get **0.542**. AP rewards early hits and penalizes misses. And **MAP**, mean average precision, is simply AP averaged over all queries.

Second, **MRR, Mean Reciprocal Rank**. It's one divided by the rank of the *first* relevant result. In our list the first relevant result is at rank 1, so the score is 1.0. It's ideal when there's a single correct answer.

Third, **nDCG**. It's for **graded** relevance, where labels run from zero to three instead of just yes or no. It also discounts results lower in the list.

**ASK:** Suppose the first relevant result sat at rank 4 instead of rank 1. What would the reciprocal rank be? *(pause. Answer: 1/4 = 0.25.)*

---

## Slide 17 · Choosing the metric (≈3 min)

There is no single best metric. The right one depends on what the user needs. Let me go through the table.

If the user needs **one correct answer**, like QA or the top result for RAG, use MRR or precision at one. Only the first hit matters.

If the user needs **everything relevant**, like legal or compliance search, use recall at k. A missed item is costly.

If the user wants a **clean, browsable list**, use precision at k or MAP. Every shown item counts.

If relevance has **grades**, use nDCG. Highly relevant items should come first.

And if you need **one summary number for tuning**, use F1 at k or MAP.

For RAG specifically, the top few chunks feed the LLM. So both the retriever's recall at k and the precision at small k matter.

**ASK:** Take a use case from your own work. Which metric from this table fits it best? Type the use case and the metric. *(pause, read out two or three)*

---

## Slide 18 · Manual evaluation (≈5 min)

Now we move to the human side. Metrics are only as good as their labels, and humans create those labels.

A good manual evaluation has five steps. **Sample**: pick representative queries. **Guidelines**: define what each rating level means. **Judge**: use two or more raters per item. **Agreement**: measure it with Cohen's kappa and resolve conflicts. **Analyse**: look per query type and find patterns.

Agreement matters most. If two raters disagree a lot, the labels aren't trustworthy, and every metric built on them is shaky.

For judging answers, a simple rubric works well. Rate each answer from one to five on four things. **Correctness**: is it factually right? **Groundedness**: is it supported by the retrieved text? **Completeness**: does it cover the whole question? **Clarity**: is it concise and readable?

**ASK:** Two raters give the same answer a 5 and a 2 for correctness. What do you do? *(pause. Answers: check the guidelines, discuss and resolve, add a third rater. Then mention the guidelines may be unclear.)*

---

## Slide 19 · Task-oriented evaluation (≈5 min)

Manual rating asks "is this answer good?" Task-oriented evaluation asks something bigger: **did the user finish the job?**

We look at four kinds of signals. **Success**: the task completion rate. **Time**: how long to a correct answer. **Effort**: reformulations and clicks per task. **Rating**: satisfaction, such as CSAT or thumbs up and down.

Two standard tools. **Side-by-side**: raters see two systems' answers to the same query and pick the better one. It's fast and sensitive. **A/B testing**: you split live traffic between two versions and compare task success and engagement. It needs enough traffic.

The trade-off: it's the most realistic signal, but it's costly, slow and subjective. So use a small, well-designed sample, give real users realistic tasks, and measure outcomes, not just relevance.

**ASK:** Which would you choose for a small internal tool with only twenty users: side-by-side or A/B testing? Why? *(pause. Answer: side-by-side, because A/B needs traffic.)*

---

## Slide 20 · Generation metrics: the idea (≈4 min)

Time for the second running example, and it's a short one. A **reference**, which is the human answer: "the cat sat on the mat". And a **generated** answer from the system: "the cat is on the mat".

Both metrics we cover compare these two texts by counting overlapping **n-grams**, which are runs of words. They just look from opposite directions.

**BLEU** is the **precision** view. Of the n-grams the system produced, how many appear in the reference? It was born in machine translation.

**ROUGE** is the **recall** view. Of the n-grams in the reference, how many did the system recover? It was born in summarization.

And here's the warning, the line at the bottom. Both reward **word overlap** with the reference, **not meaning**. Keep that in your head for the next ten minutes.

**ASK:** Give me a sentence with the same meaning as "the cat sat on the mat" but a very low word overlap. *(pause. Example: "the feline rested on the rug." Say that a human sees the same meaning, but BLEU and ROUGE will not.)*

---

## Slide 21 · BLEU (≈7 min)

Let's compute BLEU on our pair. We check 1-grams, 2-grams, 3-grams and 4-grams in the generated sentence.

**1-grams**: the generated text has six words, and five of them appear in the reference: the, cat, on, the, mat. Precision is 5 out of 6.

**2-grams**: five pairs in the generated text. Three of them match: "the cat", "on the", "the mat". Precision is 3 out of 5.

**3-grams**: four triples, and one matches, "on the mat". Precision is 1 out of 4.

**4-grams**: three of them, and **none** match. Precision is 0 out of 3.

BLEU takes the geometric mean of these four precisions and multiplies by a **brevity penalty**, so that very short outputs can't cheat. Now see the problem. One zero in that product makes everything zero. So BLEU-4 without smoothing is **0.00**, even though the sentences are close. With smoothing, we get about **0.38**.

The lesson: on short texts, always use smoothing, and say which BLEU you used.

**ASK:** Why do you think BLEU needs a brevity penalty? What would happen without one? *(pause. Answer: a very short output with a few matching words would get a perfect precision.)*

---

## Slide 22 · ROUGE (≈6 min)

Now the same sentence pair, through ROUGE. Look at the chart first: BLEU smoothed is 0.38, while ROUGE-1 F1 is 0.833, ROUGE-2 F1 is 0.6, and ROUGE-L F1 is 0.833. Same sentences, very different numbers.

**ROUGE-1** counts unigram overlap. The reference has six words, and five were recovered. So recall is 5 out of 6, and F1 is 0.833.

**ROUGE-2** counts bigram overlap. The reference has five bigrams, and three were recovered. Recall is 3 out of 5, and F1 is 0.600.

**ROUGE-L** uses the longest common subsequence. Here that's "the cat on the mat", five words. It respects word order but doesn't need the words to be adjacent. F1 is 0.833.

Notice how BLEU-4 collapses to zero while ROUGE stays informative. That tells you the metrics are not interchangeable.

**ASK:** The generated answer only said "the cat". What would happen to ROUGE recall? To BLEU precision? *(pause. Answer: ROUGE recall drops a lot, because most of the reference is missing. BLEU precision stays high, but the brevity penalty would hit.)*

---

## Slide 23 · BLEU vs ROUGE (≈3 min)

Let's put them side by side.

BLEU is about the **precision** of generated n-grams. ROUGE is about the **recall** of reference n-grams, and it's also reported as F1.

BLEU comes from machine translation. ROUGE comes from summarization.

BLEU has variants from BLEU-1 to BLEU-4, plus smoothed versions. ROUGE has ROUGE-1, ROUGE-2 and ROUGE-L.

Use BLEU when **wording should match closely**, like translation or templated output. Use ROUGE when **content coverage** matters, like summaries and answers.

And watch out. BLEU gives zero for short text without smoothing. ROUGE rewards copying and long outputs. Also, both need multiple references to be fair, since there is more than one good way to say something.

**ASK:** Which would you pick to check whether a generated summary covered the main points of a report? *(pause. Answer: ROUGE, because coverage matters.)*

---

## Slide 24 · Limits of overlap metrics (≈4 min)

Let me finish with a warning from our lab, and you'll see these numbers again soon.

We compared two systems. One **copies the top retrieved chunk** as the answer. The other produces **LLM-style, paraphrased** answers. On BLEU, ROUGE-1 and ROUGE-L, the LLM-style answers win. But on **ROUGE-2**, the copy-paste baseline scores 0.293 and the LLM-style answers only 0.270. The baseline wins that one, even though its answers are less direct.

Overlap is not understanding.

So what should we add? **Semantic similarity**, such as BERTScore. **Groundedness or faithfulness checks** against the retrieved chunks. **LLM-as-judge** with a clear rubric, calibrated on human labels. And **human review** on a sample.

The rule: use BLEU and ROUGE as regression signals, not as the final word.

**ASK:** An answer is fluent, overlaps heavily with the reference, but contains one wrong number. Would BLEU or ROUGE catch it? *(pause. Answer: probably not. That's why we add human review and groundedness checks.)*

Good. Next we go to the lab and see this for ourselves.

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask |
|---|---|---|---|
| 12 | Part II divider | 1 min | Which metrics have you used? |
| 13 | Two layers, offline vs online | 3 min | Chunk size change: offline or online? |
| 14 | Precision and recall | 5 min | P@10 and R@10 for the same list? |
| 15 | F1 and cut-off k | 4 min | "Precision is 0.9": what do you ask? |
| 16 | AP, MAP, MRR, nDCG | 4 min | First relevant at rank 4: RR? |
| 17 | Choosing the metric | 3 min | Which metric fits your use case? |
| 18 | Manual evaluation | 5 min | Raters give 5 and 2: what now? |
| 19 | Task-oriented evaluation | 5 min | Side-by-side or A/B for 20 users? |
| 20 | Generation metrics idea | 4 min | Same meaning, low overlap sentence? |
| 21 | BLEU | 7 min | Why a brevity penalty? |
| 22 | ROUGE | 6 min | Answer is only "the cat": effect? |
| 23 | BLEU vs ROUGE | 3 min | Which for summary coverage? |
| 24 | Limits of overlap | 4 min | Wrong number, high overlap: caught? |
