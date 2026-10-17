# Trainer Script · Part II: Standard Methods for Evaluating Search Systems (with Newbie Examples)

**Slides 12–24 · T+0:30 – T+1:30 · 60 min**
Relevance metrics (P, R, F1, MAP): 20 min · Manual / task-oriented evaluation: 10 min · Generation metrics (BLEU, ROUGE): 30 min

> **How to use this:** Read the plain text aloud as it is. Lines marked **ASK** are questions for the participants. Pause and let them answer in chat or on mic. Lines marked *(pause)* are short waits. Where a calculation is shown, give people a few seconds to try it before you reveal the answer.
>
> **New in this version:** Each slide has a box called **🟢 Newbie example**. Use it right after the main explanation, or whenever the room goes quiet. Two pictures run through this part:
> - **The Mango Market 🥭** for ranked-list metrics (slides 13–17). Same numbers as the running example: 10 mangoes in a row, 4 ripe ones exist, you found 3.
> - **The Classroom 🏫** for text metrics (slides 20–24). Comparing a student's answer with the teacher's model answer.

---

## Slide 12 · Part II Divider (≈1 min)

Welcome to Part II. This is the theory core of today, and it runs for one hour.

We'll cover three things. First, relevance metrics: Precision, Recall, F1 and MAP, for judging a ranked list. Second, manual and task-oriented evaluation, where humans and real users tell us what numbers can't. Third, generation metrics, BLEU and ROUGE, for judging written answers.

One thing that will make this easier. We use just **one running ranked list** and **one running sentence pair** throughout. You'll see both again in the lab, computed by code. So you don't need to keep learning new examples. You only need to learn these two well.

**ASK:** Before we start, which metric names from today's agenda have you already used? Type them in chat. *(pause)*

> 🟢 **Newbie example: "Metrics are just marks on a report card"**
> Part I was about *building* search. Part II is about *giving it marks*, like a teacher grading a student.
> - Marks for a **list of results** → Precision, Recall, F1, MAP (slides 13–17)
> - Marks given by **humans** → manual and task-based (slides 18–19)
> - Marks for a **written answer** → BLEU, ROUGE (slides 20–24)
>
> Don't worry about the names. Each one answers one plain question, and we'll use one example for each half.

---

## Slide 13 · Two layers, two modes (≈3 min)

Let's start with the big picture. We evaluate a RAG system in **two layers**, and then **end to end**.

The **retrieval layer** is judged against relevance labels: did we find the right chunks? The **generation layer** is judged against reference answers: did we write the right answer? And **end to end**, it's judged by humans and users: did this actually help?

There are also two **modes**. **Offline evaluation** uses a test collection: a fixed set of queries, labelled relevant chunks, and reference answers. It's repeatable and fast, and it's great for regression tests. **Online evaluation** uses real users: clicks, task success, A/B tests. It's more realistic, but it's slower and noisier, and you need traffic.

We start with offline, because it's repeatable and cheap. You can run it every time you change something.

**ASK:** You've just changed the chunk size. Would you check that offline or online first? Why? *(pause. Answer: offline, because it's fast and repeatable. Online comes later for confirmation.)*

> 🟢 **Newbie example: "A restaurant kitchen 🍽️"**
>
> | Layer | In the restaurant | What we check |
> |---|---|---|
> | **Retrieval** | The person who fetches ingredients | Did they bring the *right* ingredients? |
> | **Generation** | The chef who cooks the dish | Is the dish cooked *correctly* from those ingredients? |
> | **End to end** | The customer eating | Did the customer enjoy the meal? |
>
> Why separate them? If the dish is bad, you need to know: *wrong ingredients* (retrieval) or *bad cooking* (generation)?
>
> | Mode | In the restaurant |
> |---|---|
> | **Offline** | You test the recipe in the kitchen with a fixed set of test orders. Fast, repeatable, free. |
> | **Online** | You serve real customers and watch their reactions. Real, but slow, noisy, and risky. |
>
> **Rule:** Taste-test in the kitchen first (offline). Then confirm with real customers (online).

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

> 🟢 **Newbie example: "Picking ripe mangoes 🥭"**
>
> A fruit seller lines up **10 mangoes** for you (rank 1 to 10). In the whole market there are **4 ripe mangoes**. The seller's row contains 3 of them (positions 1, 3, 6). The 4th ripe one is at a different stall and **never made it to the row**.
>
> ```
> Rank:   1    2    3    4    5  |  6    7    8    9   10
>         🥭   ·    🥭   ·    ·  |  🥭   ·    ·    ·    ·
>       ripe     ripe              ripe          (🥭 = ripe, · = raw)
>       |<------ you pick the first 5 ------>|
> ```
>
> You only take the **first 5** mangoes home. 2 of them are ripe.
>
> | Question | Plain meaning | Calculation |
> |---|---|---|
> | **Precision@5** | "Of the 5 I took, how many are ripe?" | 2 ÷ 5 = **0.40** |
> | **Recall@5** | "Of all 4 ripe mangoes in the market, how many did I get?" | 2 ÷ 4 = **0.50** |
>
> **Easy way to remember:**
> - **Precision = how *clean* is my bag** (few raw ones inside).
> - **Recall = how *complete* is my bag** (did I get all the ripe ones?).
>
> **ASK answer:** Take all 10 → 3 ripe → Precision 3/10 = 0.30, Recall 3/4 = 0.75. A bigger bag catches more ripe ones (recall ↑) but carries more raw ones (precision ↓).
>
> **Common mistake to warn about:** Recall is divided by *all ripe mangoes in the market (4)*, not by the 3 you happened to find.

---

## Slide 15 · F1 and the cut-off k (≈4 min)

That leads straight to this slide. Look at the chart. As k grows from one to ten, the recall line can only go up or stay flat. Precision tends to fall.

So how do we get one number? That's **F1**. It's two times P times R, divided by P plus R. It's the **harmonic mean**, which means it's low if *either* precision or recall is low. You can't hide a weak one behind a strong one.

For our running example, at k equals five: P is 0.40 and R is 0.50, so F1 is **0.444**.

And look at the line at the bottom: **report k with every score.** A score of 0.4 means nothing unless I know whether it was measured at top three, top five or top ten.

**ASK:** If a teammate says "our precision is 0.9", what is the first thing you'd ask them? *(pause. Answer: at what k, and on which dataset.)*

> 🟢 **Newbie example: "The seller who shows only one mango"**
>
> | Seller | What they do | Precision | Recall |
> |---|---|---|---|
> | **Careful seller** | Shows just **1** mango, a perfectly ripe one | **1.00** (very clean) | **0.25** (found 1 of 4) |
> | **Greedy seller** | Dumps **all 100** mangoes in your bag, including all 4 ripe ones | **0.04** (mostly raw) | **1.00** (got all) |
> | **Balanced seller** | Shows 5 mangoes, 2 ripe | 0.40 | 0.50 |
>
> Neither extreme is useful. **F1 gives one mark that is low if *either* is low.**
>
> Example, extreme case: Precision = 1.0, Recall = 0.01
> - Plain average = (1.0 + 0.01) ÷ 2 = **0.505**. Looks okay, but it is misleading!
> - F1 = 2 × 1.0 × 0.01 ÷ 1.01 = **0.02**. Honest: this system is bad.
>
> **Running example:** F1@5 = 2 × 0.40 × 0.50 ÷ (0.40 + 0.50) = 0.40 ÷ 0.90 = **0.444**.
>
> **Why always say k?** "Precision 0.9" could mean "0.9 on a bag of 1 mango" (easy) or "0.9 on a bag of 20 mangoes" (hard). Without k, the number is meaningless.

---

## Slide 16 · Rank-aware metrics (≈4 min)

So far, precision at five treats ranks one and five the same. But users care a lot about order. Rank-aware metrics fix that.

First, **Average Precision, or AP**. For each relevant result we find, we take the precision at that rank. Then we divide by the total number of relevant items, including the one we missed. For our list: precision at rank 1 is 1 out of 1. At rank 3, it's 2 out of 3. At rank 6, it's 3 out of 6. Add those up and divide by 4 relevant items. We get **0.542**. AP rewards early hits and penalizes misses. And **MAP**, mean average precision, is simply AP averaged over all queries.

Second, **MRR, Mean Reciprocal Rank**. It's one divided by the rank of the *first* relevant result. In our list the first relevant result is at rank 1, so the score is 1.0. It's ideal when there's a single correct answer.

Third, **nDCG**. It's for **graded** relevance, where labels run from zero to three instead of just yes or no. It also discounts results lower in the list.

**ASK:** Suppose the first relevant result sat at rank 4 instead of rank 1. What would the reciprocal rank be? *(pause. Answer: 1/4 = 0.25.)*

> 🟢 **Newbie example: "Order matters: the mango queue"**
>
> You eat mangoes one by one from the row. **Finding a ripe one early feels better** than finding it late. Precision@5 can't see that, but AP can.
>
> **AP, step by step** (ripe at positions 1, 3, 6; 4 ripe exist in total):
>
> | Ripe mango found at… | Mangoes tasted so far | Ripe among them | Precision at that moment |
> |---|---|---|---|
> | Position 1 | 1 | 1 | 1/1 = 1.000 |
> | Position 3 | 3 | 2 | 2/3 = 0.667 |
> | Position 6 | 6 | 3 | 3/6 = 0.500 |
> | The 4th ripe one | never found | — | counts as **0** |
>
> **AP** = (1.000 + 0.667 + 0.500 + 0) ÷ 4 = 2.167 ÷ 4 = **0.542**
> The missing 4th mango pulls the score down. Early finds help, missed ones hurt.
>
> **MAP** = do the same for many queries and take the average.
>
> **MRR = "how fast did I get my first good one?"**
>
> | First ripe mango at rank | Reciprocal rank |
> |---|---|
> | 1 | 1/1 = **1.00** |
> | 2 | 1/2 = **0.50** |
> | 4 | 1/4 = **0.25** |
> | 10 | 1/10 = **0.10** |
>
> Great when there is **one right answer** (like finding an ATM). You just care how soon you hit it.
>
> **nDCG = grades, not just yes/no.** Hotels: 5-star = grade 3, 3-star = grade 1, closed hotel = 0. nDCG says: *put the 5-star hotels at the top.* A 3-star at rank 1 and a 5-star at rank 8 scores worse than the other way round.

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

> 🟢 **Newbie example: "Which mark for which job?"**
>
> | Everyday situation | What really matters | Metric |
> |---|---|---|
> | "Find the **nearest ATM**" | The first result must be right | **MRR / Precision@1** |
> | "Find **every clause** in the contract that mentions penalties" (a missed one can cost money) | Don't miss anything | **Recall@k** |
> | "Browse **10 shirt suggestions**" | Every shown shirt should be good | **Precision@k / MAP** |
> | "Show **5-star hotels** before 3-star ones" | Best items first | **nDCG** |
> | "One number to **compare two versions** while tuning" | A balanced summary | **F1@k / MAP** |
>
> **For RAG:** the LLM only reads the top few chunks, like an open-book exam where you can bring only 3 pages. So you need both: the right page *must be in the 3* (recall@3) and the 3 should not be full of junk (precision).

---

## Slide 18 · Manual evaluation (≈5 min)

Now we move to the human side. Metrics are only as good as their labels, and humans create those labels.

A good manual evaluation has five steps. **Sample**: pick representative queries. **Guidelines**: define what each rating level means. **Judge**: use two or more raters per item. **Agreement**: measure it with Cohen's kappa and resolve conflicts. **Analyse**: look per query type and find patterns.

Agreement matters most. If two raters disagree a lot, the labels aren't trustworthy, and every metric built on them is shaky.

For judging answers, a simple rubric works well. Rate each answer from one to five on four things. **Correctness**: is it factually right? **Groundedness**: is it supported by the retrieved text? **Completeness**: does it cover the whole question? **Clarity**: is it concise and readable?

**ASK:** Two raters give the same answer a 5 and a 2 for correctness. What do you do? *(pause. Answers: check the guidelines, discuss and resolve, add a third rater. Then mention the guidelines may be unclear.)*

> 🟢 **Newbie example: "Two teachers marking the same answer"**
>
> **Question:** "What is the capital of Australia?" **Student answer:** "Canberra, which is Australia's capital and largest city."
>
> | Teacher | Correctness | Why |
> |---|---|---|
> | Teacher A | 5 | "Canberra is right." |
> | Teacher B | 2 | "Canberra is the capital, but not the *largest* city (that's Sydney), so part is wrong." |
>
> Both are partly right! The disagreement shows the **rating guide was unclear** (does one wrong detail make the answer a 2 or a 4?). That's why you need **guidelines**, **two or more raters**, and a **measure of agreement**.
>
> **Cohen's kappa in one line:** "How much do the two teachers agree, *after removing agreement that could happen by pure luck*?" (If two people guess a coin toss, they'd agree 50% of the time. That doesn't count.) Low kappa → fix the guidelines before trusting any label.
>
> **The 4-point rubric, as a school report:**
>
> | Criterion | Plain question | 1 (bad) → 5 (good) |
> |---|---|---|
> | **Correctness** | Is it true? | Wrong facts → all facts right |
> | **Groundedness** | Is it backed by the retrieved page? | Made up → every claim has evidence |
> | **Completeness** | Does it cover the whole question? | Half answered → fully answered |
> | **Clarity** | Is it easy to read? | Confusing → crisp |

---

## Slide 19 · Task-oriented evaluation (≈5 min)

Manual rating asks "is this answer good?" Task-oriented evaluation asks something bigger: **did the user finish the job?**

We look at four kinds of signals. **Success**: the task completion rate. **Time**: how long to a correct answer. **Effort**: reformulations and clicks per task. **Rating**: satisfaction, such as CSAT or thumbs up and down.

Two standard tools. **Side-by-side**: raters see two systems' answers to the same query and pick the better one. It's fast and sensitive. **A/B testing**: you split live traffic between two versions and compare task success and engagement. It needs enough traffic.

The trade-off: it's the most realistic signal, but it's costly, slow and subjective. So use a small, well-designed sample, give real users realistic tasks, and measure outcomes, not just relevance.

**ASK:** Which would you choose for a small internal tool with only twenty users: side-by-side or A/B testing? Why? *(pause. Answer: side-by-side, because A/B needs traffic.)*

> 🟢 **Newbie example: "Did you actually book the train ticket? 🚆"**
>
> A booking website shows very relevant search results, but you still **can't buy a ticket**. Relevance metrics say "great!". You say "useless!". Task-oriented evaluation measures *your* result:
>
> | Signal | In the ticket example |
> |---|---|
> | **Success** | Did you finish booking? (yes/no) |
> | **Time** | How many minutes did it take? |
> | **Effort** | How many times did you re-type the search or click back? |
> | **Rating** | Did you tap 👍 or 👎 at the end? |
>
> **Two ways to compare versions:**
>
> | Tool | Everyday picture | Use when |
> |---|---|---|
> | **Side-by-side** | Two chefs cook the same dish; a taster picks the better one | Few users: quick and sensitive |
> | **A/B test** | Half the customers get menu A, half get menu B, then compare sales | Lots of traffic |
>
> **ASK answer:** With only 20 users, an A/B test would not have enough people to show a clear difference, so use side-by-side.

---

## Slide 20 · Generation metrics: the idea (≈4 min)

Time for the second running example, and it's a short one. A **reference**, which is the human answer: "the cat sat on the mat". And a **generated** answer from the system: "the cat is on the mat".

Both metrics we cover compare these two texts by counting overlapping **n-grams**, which are runs of words. They just look from opposite directions.

**BLEU** is the **precision** view. Of the n-grams the system produced, how many appear in the reference? It was born in machine translation.

**ROUGE** is the **recall** view. Of the n-grams in the reference, how many did the system recover? It was born in summarization.

And here's the warning, the line at the bottom. Both reward **word overlap** with the reference, **not meaning**. Keep that in your head for the next ten minutes.

**ASK:** Give me a sentence with the same meaning as "the cat sat on the mat" but a very low word overlap. *(pause. Example: "the feline rested on the rug." Say that a human sees the same meaning, but BLEU and ROUGE will not.)*

> 🟢 **Newbie example: "The classroom: student vs teacher 🏫"**
>
> The teacher wrote the model answer: **"the cat sat on the mat"**
> A student wrote: **"the cat is on the mat"**
>
> The teacher uses a simple *word-matching* checker with **two questions**:
>
> | Checker | Question it asks | Everyday meaning |
> |---|---|---|
> | **BLEU** (precision view) | "Of the words **the student wrote**, how many are in the teacher's answer?" | *Is everything you wrote correct?* |
> | **ROUGE** (recall view) | "Of the words **in the teacher's answer**, how many did the student cover?" | *Did you cover everything needed?* |
>
> **n-grams** are just groups of words:
>
> | Type | Example from "the cat is on the mat" |
> |---|---|
> | 1-gram (single words) | the · cat · is · on · the · mat |
> | 2-gram (pairs) | "the cat" · "cat is" · "is on" · "on the" · "the mat" |
> | 3-gram (triples) | "the cat is" · "cat is on" · "is on the" · "on the mat" |
>
> **The big warning:** the checker only counts *matching words*. It does **not understand meaning**.
> - "the feline rested on the rug" means the **same thing**, but shares almost no words, so it gets a **low score**. A human would give full marks.

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

> 🟢 **Newbie example: "Counting matching words, step by step"**
>
> Teacher: `the cat sat on the mat`  ·  Student: `the cat is on the mat`
>
> | Group size | Student's groups | Found in teacher's answer? | Score |
> |---|---|---|---|
> | **1 word** | the, cat, is, on, the, mat | the ✅ cat ✅ is ❌ on ✅ the ✅ mat ✅ | **5 / 6** |
> | **2 words** | the cat ✅ · cat is ❌ · is on ❌ · on the ✅ · the mat ✅ | 3 match | **3 / 5** |
> | **3 words** | the cat is ❌ · cat is on ❌ · is on the ❌ · on the mat ✅ | 1 match | **1 / 4** |
> | **4 words** | the cat is on ❌ · cat is on the ❌ · is on the mat ❌ | 0 match | **0 / 3** |
>
> **Why does the final score become 0?** BLEU *multiplies* the four scores (roughly). In multiplication, **one zero kills everything**: 0.83 × 0.60 × 0.25 × **0** = 0. Yet the student's answer is nearly perfect, so we use **smoothing** (a small fix so zero doesn't wipe the score) → about **0.38**.
>
> **Brevity penalty: the "Hi" trick.**
> Suppose the student writes just **"the"**. All 1 of the student's words match the teacher's answer: precision = 100%! Silly. So BLEU **cuts the score** when the answer is much shorter than the reference. Without this rule, answering with one safe word would win.

---

## Slide 22 · ROUGE (≈6 min)

Now the same sentence pair, through ROUGE. Look at the chart first: BLEU smoothed is 0.38, while ROUGE-1 F1 is 0.833, ROUGE-2 F1 is 0.6, and ROUGE-L F1 is 0.833. Same sentences, very different numbers.

**ROUGE-1** counts unigram overlap. The reference has six words, and five were recovered. So recall is 5 out of 6, and F1 is 0.833.

**ROUGE-2** counts bigram overlap. The reference has five bigrams, and three were recovered. Recall is 3 out of 5, and F1 is 0.600.

**ROUGE-L** uses the longest common subsequence. Here that's "the cat on the mat", five words. It respects word order but doesn't need the words to be adjacent. F1 is 0.833.

Notice how BLEU-4 collapses to zero while ROUGE stays informative. That tells you the metrics are not interchangeable.

**ASK:** The generated answer only said "the cat". What would happen to ROUGE recall? To BLEU precision? *(pause. Answer: ROUGE recall drops a lot, because most of the reference is missing. BLEU precision stays high, but the brevity penalty would hit.)*

> 🟢 **Newbie example: "A coverage checklist"**
>
> Teacher's answer words: **the · cat · sat · on · the · mat** (6 items). The student's answer: *the cat is on the mat*.
>
> | ROUGE type | What it checks | Calculation | Score |
> |---|---|---|---|
> | **ROUGE-1** | Teacher's **single words** covered | 5 of 6 covered (missed "sat") | 5/6 → F1 **0.833** |
> | **ROUGE-2** | Teacher's **word pairs** covered | 3 of 5 pairs (the cat, on the, the mat) | 3/5 → F1 **0.600** |
> | **ROUGE-L** | Longest run of words in the **same order** (gaps allowed) | "the cat … on the mat" = 5 words | F1 **0.833** |
>
> **ROUGE-L in plain words:** "How many words appear in the same order?" Teacher: *the cat **sat** on the mat*. Student: *the cat **is** on the mat*. Skip the one different word, and 5 words line up.
>
> **Compare the two checkers on the same sentences:**
>
> | Metric | Score |
> |---|---|
> | BLEU (smoothed) | 0.38 |
> | ROUGE-1 | 0.833 |
> | ROUGE-2 | 0.600 |
> | ROUGE-L | 0.833 |
>
> Different checkers, different marks. **Always say which one you used.**
>
> **ASK answer (student wrote only "the cat"):**
> - ROUGE (coverage): covers 2 of 6 teacher words = **0.33**, a poor mark for missing most of the answer.
> - BLEU (correctness): both words are right = 100% precision, but the **brevity penalty** reduces it (the answer is far too short).

---

## Slide 23 · BLEU vs ROUGE (≈3 min)

Let's put them side by side.

BLEU is about the **precision** of generated n-grams. ROUGE is about the **recall** of reference n-grams, and it's also reported as F1.

BLEU comes from machine translation. ROUGE comes from summarization.

BLEU has variants from BLEU-1 to BLEU-4, plus smoothed versions. ROUGE has ROUGE-1, ROUGE-2 and ROUGE-L.

Use BLEU when **wording should match closely**, like translation or templated output. Use ROUGE when **content coverage** matters, like summaries and answers.

And watch out. BLEU gives zero for short text without smoothing. ROUGE rewards copying and long outputs. Also, both need multiple references to be fair, since there is more than one good way to say something.

**ASK:** Which would you pick to check whether a generated summary covered the main points of a report? *(pause. Answer: ROUGE, because coverage matters.)*

> 🟢 **Newbie example: "Two questions a teacher asks"**
>
> | | **BLEU** | **ROUGE** |
> |---|---|---|
> | The teacher's question | "Is **everything you wrote** correct?" | "Did you write **everything needed**?" |
> | Good for | Translation, fixed phrasing | Summaries, answers (coverage) |
> | Easy to fool by | Writing very little | Writing a lot or copying the whole book |
>
> **Fooling BLEU:** write one safe word → every word is right → high precision (that's why the brevity penalty exists).
> **Fooling ROUGE:** copy the *entire textbook* → every teacher word is covered → high recall.
>
> **Multiple references:** "the cat sat on the mat" and "a cat was sitting on the rug" are both fine answers. If you only keep one model answer, a good student can lose marks just for using different words. So keep **several** reference answers when possible.
>
> **ASK answer:** A summary should *cover the main points* → **ROUGE**.

---

## Slide 24 · Limits of overlap metrics (≈4 min)

Let me finish with a warning from our lab, and you'll see these numbers again soon.

We compared two systems. One **copies the top retrieved chunk** as the answer. The other produces **LLM-style, paraphrased** answers. On BLEU, ROUGE-1 and ROUGE-L, the LLM-style answers win. But on **ROUGE-2**, the copy-paste baseline scores 0.293 and the LLM-style answers only 0.270. The baseline wins that one, even though its answers are less direct.

Overlap is not understanding.

So what should we add? **Semantic similarity**, such as BERTScore. **Groundedness or faithfulness checks** against the retrieved chunks. **LLM-as-judge** with a clear rubric, calibrated on human labels. And **human review** on a sample.

The rule: use BLEU and ROUGE as regression signals, not as the final word.

**ASK:** An answer is fluent, overlaps heavily with the reference, but contains one wrong number. Would BLEU or ROUGE catch it? *(pause. Answer: probably not. That's why we add human review and groundedness checks.)*

Good. Next we go to the lab and see this for ourselves.

> 🟢 **Newbie example: "The checker that can be tricked"**
>
> **Case 1: Wrong, but scores high.**
>
> | | Sentence |
> |---|---|
> | Teacher | "The meeting is at **5 PM** in room 3." |
> | Student | "The meeting is at **6 PM** in room 3." |
>
> Almost every word matches, so BLEU and ROUGE give a **very high score**. But the answer is **wrong**: someone shows up an hour late!
>
> **Case 2: Right, but scores low.**
>
> | | Sentence |
> |---|---|
> | Teacher | "The cat sat on the mat." |
> | Student | "The feline rested on the rug." |
>
> Same meaning, almost no matching words → **low score**.
>
> **Case 3: Our lab result.** A student who **copies the textbook line** can beat a student who **explains in their own words** on ROUGE-2 (0.293 vs 0.270). The copier isn't a better answerer. The checker just rewards matching words.
>
> **So what do we add?**
>
> | Extra check | Plain meaning |
> |---|---|
> | **BERTScore** | Compares *meaning*, not just words |
> | **Groundedness check** | "Is every claim backed by the retrieved page?" (would catch 5 PM vs 6 PM) |
> | **LLM-as-judge** | An AI grader with a clear rubric, tested against human graders |
> | **Human review** | A person reads a sample |
>
> **Take-home line:** BLEU and ROUGE are a *smoke alarm*: handy to tell you "something changed" after an update, but not a replacement for someone checking the room.

---

### Trainer quick reference

| Slide | Topic | Time | Question to ask | One-line newbie picture |
|---|---|---|---|---|
| 12 | Part II divider | 1 min | Which metrics have you used? | Metrics = marks on a report card |
| 13 | Two layers, offline vs online | 3 min | Chunk size change: offline or online? | Ingredients vs cooking vs customer; kitchen test vs real diners |
| 14 | Precision and recall | 5 min | P@10 and R@10 for the same list? | Ripe mangoes: clean bag vs complete bag |
| 15 | F1 and cut-off k | 4 min | "Precision is 0.9": what do you ask? | Seller who shows just one mango |
| 16 | AP, MAP, MRR, nDCG | 4 min | First relevant at rank 4: RR? | Finding the ripe mango early; 5-star hotels first |
| 17 | Choosing the metric | 3 min | Which metric fits your use case? | ATM = MRR, contract clauses = recall |
| 18 | Manual evaluation | 5 min | Raters give 5 and 2: what now? | Two teachers marking one answer |
| 19 | Task-oriented evaluation | 5 min | Side-by-side or A/B for 20 users? | Did you actually book the ticket? |
| 20 | Generation metrics idea | 4 min | Same meaning, low overlap sentence? | Student vs teacher, word matching |
| 21 | BLEU | 7 min | Why a brevity penalty? | Matching words; one zero kills the product; the "Hi" trick |
| 22 | ROUGE | 6 min | Answer is only "the cat": effect? | Coverage checklist |
| 23 | BLEU vs ROUGE | 3 min | Which for summary coverage? | "Is it all correct?" vs "Is it all there?" |
| 24 | Limits of overlap | 4 min | Wrong number, high overlap: caught? | 5 PM vs 6 PM: the checker can be tricked |

---

### Cheat sheet: all numbers from the running examples

| Item | Value |
|---|---|
| Relevant ranks (list of 10) | 1, 3, 6 (a 4th relevant item never retrieved) |
| Precision@5 / Recall@5 | 2/5 = **0.40** / 2/4 = **0.50** |
| Precision@10 / Recall@10 | 3/10 = **0.30** / 3/4 = **0.75** |
| F1@5 | **0.444** |
| AP | (1/1 + 2/3 + 3/6) ÷ 4 = **0.542** |
| MRR (this list) | **1.0** (first relevant at rank 1) |
| BLEU n-gram precisions | 5/6, 3/5, 1/4, 0/3 |
| BLEU-4 unsmoothed / smoothed | **0.00** / about **0.38** |
| ROUGE-1 / ROUGE-2 / ROUGE-L (F1) | **0.833** / **0.600** / **0.833** |
| Lab: ROUGE-2 copy baseline vs LLM-style | **0.293** vs **0.270** |

---

### Tips for using the newbie examples

1. **Use the mango picture only for lists (slides 13–17).** Switch to the classroom picture for text (slides 20–24) so people don't mix them up.
2. **Let people calculate first.** For the mango example, give 30 seconds and ask them to type Precision@5 before you reveal it.
3. **Don't read every table.** Pick one example per slide. If the room is confident, skip the box and save time.
4. **If you run behind:** shorten slides 17 and 23 (reference slides) to one minute each, and protect slides 14, 16, 21 and 24.
5. **Link forward:** the lab computes the same numbers (P@5 = 0.40, AP = 0.5417, BLEU 0.38, ROUGE 0.833/0.600/0.833) with code. Tell participants they will "see these same mangoes and cats in the terminal".
