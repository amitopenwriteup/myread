**# Trainer Script · Part IV: Common Challenges & Emerging Trends

**Slides 31–38 · T+2:15 – T+2:45 · 30 min**
Ambiguity, intent, bias: 15 min · User-centric metrics, explainability, ethics: 15 min

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Pause and let them answer in chat or on mic. Lines marked *(pause)* are short waits.

---

## Slide 31 · Part IV Divider (≈1 min)

Welcome back from the lab. Up to now we've asked *how* to measure a search system. In this part we ask something harder: **what can go wrong with the measurement itself?**

We have two blocks of fifteen minutes. First, the common challenges: ambiguity, intent, changing data, and bias. Second, emerging trends: user-centric metrics, explainable search, and ethics.

**ASK:** From the lab, did any score surprise you? Type it in chat in one line. *(pause, read out one or two)*

Hold on to that feeling. It's exactly what this part is about.

---

## Slide 32 · Ambiguity (≈3 min)

Let's start with ambiguity. One query can have several valid answers.

Take the query "python". Is the user looking for the **programming language**? The **snake**? Or the **comedy troupe**? All three are valid. The query doesn't say.

What does this do to evaluation? Raters can disagree, and they can be perfectly right to disagree. So labels become noisy, and metrics shift depending on which rater you happened to ask.

What can we do? Use **graded labels**, not just yes or no. Use **multiple references**. Build **intent-aware test sets**. And in the product itself, **ask a clarifying question**.

**ASK:** Think of another ambiguous query in your own domain. Type it in chat with its possible meanings. *(pause, read out two or three. Examples: "apple", "java", "jaguar", "bank".)*

**ASK:** For one of those queries, how would you label results if two raters disagree? *(pause. Answers: graded labels, a third rater, separate labels per intent.)*

---

## Slide 33 · Intent and dynamicity (≈4 min)

Two more challenges. Both are about a target that keeps moving.

First, **intent**. The same words can come with different goals. An **informational** query: "how does cron work". A **navigational** query: "ufw docs", where the user wants one specific page. A **task** query: "fix permission denied on file", where the user wants to get something done.

If we only report one overall score, we hide these differences. A system can be great at navigational queries and weak at task queries, and the average hides it. So **report metrics per intent, not just overall.**

Second, **dynamicity**. Documents are added, edited and removed. Query language and topics drift. Models and prompts get updated. So a test set that was perfect last year slowly goes out of date.

The fix: **refresh the labels, version the test set, and monitor continuously.**

**ASK:** If you froze your test set a year ago and never touched it, what could silently go wrong? *(pause. Answers: new documents are never tested, removed documents still count as relevant, new query types are missing.)*

**ASK:** Which of the three intent types would be hardest for your system? Type "info", "nav" or "task". *(pause and look at the spread)*

---

## Slide 34 · Six evaluation biases (≈7 min)

Now the part I find most important: bias. There are six ways that your scores can mislead you. Let me go through them one by one.

**Position bias.** Users and raters tend to favour whatever is at the top. The fix is to randomize the order when people judge.

**Annotator bias.** A rater's background and tiredness shape their labels. Use clear guidelines and measure agreement.

**Pooling or selection bias.** This is a classic. If only the results shown by the old system were ever judged, then a new system that returns *different* results will look wrong, because nobody has labelled them. The fix is to judge a union of results from several systems.

**Metric bias.** Every metric has a preference. ROUGE favours length and copying. BLEU favours exact wording.

**Judge bias.** LLM judges tend to prefer longer answers, and answers in their own style. Calibrate them on human labels.

**Coverage bias.** The test set may miss languages, topics or user groups. Audit your results by slice.

**ASK:** Which of these six has the biggest risk in your own project? Type the name in chat. *(pause and read out a few)*

**ASK:** A new system gets a lower score than the old one, but its results are mostly new items nobody has labelled. Which bias is that? *(pause. Answer: pooling or selection bias.)*

---

## Slide 35 · User-centric metrics (≈4 min)

Now we move to emerging trends, and we start with a simple shift. From system-centric relevance to **user outcomes.** The question changes from "was this result relevant?" to "**did this actually help?**"

Look at the left side. Four outcomes matter to users. **Task success**: did they finish? **Effort**: how much time and how many reformulations? **Trust**: is the confidence of the system calibrated? **Satisfaction**: ratings and return rate.

On the right is the **RAG triad**. It's a popular way to break RAG quality into three automated checks.

**Context relevance**: is the retrieved context relevant to the query? That's query to chunks.
**Groundedness**: is the answer supported by the retrieved chunks? That's answer to chunks.
**Answer relevance**: does the answer actually address the question? That's answer to query.

The nice thing is that each side points to a different part of the pipeline, so when one is low, you know where to look.

**ASK:** A RAG answer scores high on answer relevance but low on groundedness. What does it tell you? *(pause. Answer: it answers the question, but with claims not supported by the retrieved evidence. A hallucination risk.)*

**ASK:** And if context relevance is low? *(pause. Answer: retrieval is bringing the wrong chunks. Fix retrieval first.)*

---

## Slide 36 · Explainable search (≈3 min)

Next trend: explainable search. The idea is simple. **Show why an answer appeared.**

Look at the example on the slide. The question: "How do I run a script every night?" The answer says to add a line with `crontab -e`, for example `30 2 * * * /opt/backup.sh`, which runs at 02:30 every day.

Notice three things, each one a feature. **Citations**: each claim links to its source chunk. Here that's marker one, chunk c08. **Evidence highlights**: we show the matching passage itself, "cron runs scheduled jobs. Edit your schedule with crontab -e". And **abstain and confidence**: the system shows a confidence level and an evidence match score, and it says "not found" when the evidence is weak.

Why does this matter? It helps users **calibrate trust**, and it helps developers **debug**. And remember, explanations need to be evaluated too. Do the citations actually support the claim they're attached to?

**ASK:** Would you trust an answer more with a citation? Does a citation guarantee the answer is right? *(pause. Answer: no. A citation can be wrong or irrelevant, so we must check that it supports the claim.)*

**ASK:** When should a system answer "not found" instead of guessing? *(pause. Answers: when evidence is weak, in high-stakes domains, when retrieval confidence is low.)*

---

## Slide 37 · Ethics in evaluation (≈3 min)

Last emerging topic: ethics. Evaluation isn't only about accuracy. There are six questions to keep in the plan, from the start.

**Fairness.** Report metrics by language, region and user group. An average can hide a group that is badly served.

**Privacy.** Protect query logs and personal data, both in the corpus and in the test sets.

**Safety.** Test for harmful or misleading answers in high-stakes domains such as health, legal and finance, where a hallucination does real harm.

**Transparency.** Disclose that answers are AI-generated, and what their limits are.

**Rights.** Respect the copyright and licensing of the content you index.

**Cost.** Weigh the compute and energy use of large-scale evaluation.

**ASK:** In which domain would a hallucinated answer be the most harmful? Type one in chat. *(pause and read out two or three)*

**ASK:** Which of the six is most often forgotten in real projects? *(pause. Common answers: privacy of logs, fairness by slice, cost.)*

---

## Slide 38 · Evaluation checklist (≈5 min)

Let's finish Part IV with something you can take back to work. A six-step checklist.

**One:** define the task and the users. **Two:** build a labelled golden set. **Three:** score retrieval and generation separately. **Four:** add human review on a sample. **Five:** check for bias and slice by group. **Six:** report k, dataset and version, and monitor over time.

If you do only these six things, you'll avoid most of the mistakes we've discussed today.

**ASK:** Which step is the hardest for your team? Type the number in chat. *(pause and read out the spread)*

**ASK:** Which step could you start on next week? *(pause. Let two people answer.)*

Thank you. In the last part we'll answer your questions and recap.

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask |
|---|---|---|---|
| 31 | Part IV divider | 1 min | Did any lab score surprise you? |
| 32 | Ambiguity | 3 min | Another ambiguous query? |
| 33 | Intent and dynamicity | 4 min | What breaks in a frozen test set? |
| 34 | Six biases | 7 min | Unlabelled new results: which bias? |
| 35 | User-centric metrics, RAG triad | 4 min | High answer relevance, low groundedness? |
| 36 | Explainable search | 3 min | Does a citation guarantee correctness? |
| 37 | Ethics | 3 min | Most harmful domain for hallucination? |
| 38 | Evaluation checklist | 5 min | Hardest step for your team? |
**
