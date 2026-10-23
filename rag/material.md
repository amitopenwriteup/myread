# LLM Evaluation Demonstration
## Trainer Script (Teleprompter Format) | Expanded Edition

**How to use this script**

- Read the plain text out loud. It is written the way you speak.
- Text in **[BRACKETS]** is a stage cue. Do not read it aloud.
- Text marked **[EXTRA]** is added talking material: analogies, examples, stories, questions. It is optional. Drop it first if you run late.
- Each line is one breath. Pause at every blank line.
- Times are guides. Total session: 3 hours, including a 10 minute break. Extra material adds roughly 15 to 20 minutes in total, so trim as noted in the Trainer Quick Notes.
- Numbers in worked examples are illustrative. Replace them with your own notebook scores if you prefer.

| Section | Slides | Time |
|---|---|---|
| Opening | 1 to 3 | 5 min |
| 1. Introduction to LLM evaluation | 4 to 6 | 15 min |
| 2. Evaluating a non-RAG application | 7 to 18 | 55 min |
| 3. Building a RAG pipeline | 19 to 25 | 40 min |
| Break | 26 | 10 min |
| 4. Evaluating the RAG application | 27 to 39 | 50 min |
| 5. Conclusion and Q and A | 40 to 42 | 10 min |

---

# OPENING

---

## SLIDE 1 | Title
**[2 min]**

Good morning, everyone.
Welcome to this session on LLM evaluation.

Over the next three hours, we will answer one question.
How do we know whether an LLM application is actually good?

We will use one business use case.
We will evaluate it twice.
First as a standalone LLM.
Then as a RAG application.

Our tools are LangChain, ChromaDB, Ollama and RAGAS.
Everything runs locally.

**[EXTRA]**
Let me explain why "locally" matters.
Many companies cannot send internal documents to an outside service.
Legal, privacy and cost reasons all get in the way.
So running everything on one machine is not just convenient for a demo.
It is close to what many real teams have to do.

**[EXTRA: quick poll, 1 minute]**
Raise your hand if you have built something with an LLM.
Keep it up if you have measured how good it is, with numbers.
Most hands go down at the second question.
That gap is exactly what today is about.

**[CLICK]**

---

## SLIDE 2 | Session agenda
**[2 min]**

Here is the plan for the three hours.

Section one is a twenty minute introduction.
Why is evaluating an LLM different from evaluating a normal machine learning model?

Section two is the longest, fifty-five minutes.
We evaluate a standalone LLM using BLEU, ROUGE, METEOR, BERTScore and Exact Match.

Section three is forty minutes.
We build a RAG pipeline with LangChain and ChromaDB.

Section four is fifty minutes.
We evaluate that RAG pipeline with RAGAS metrics.

Section five is ten minutes for wrap-up and questions.

There is a ten minute break after section three.

**[POINT to the right-hand cards]**
Three hours, two demonstrations, one use case.

**[EXTRA]**
Notice the shape of the day.
First we measure a simple system.
Then we make the system more complex.
Then we measure again, and we need better tools, because there are more places to fail.
That is a pattern you will meet again and again in real projects.

**[EXTRA: ground rules, 30 seconds]**
Please ask questions as we go.
If something is unclear, it is probably unclear for three other people too.
I will park deep technical detours until the end, so we stay on schedule.

---

## SLIDE 3 | What you will take away
**[1 min]**

By the end, you should be able to do eight things.

Read this list quickly, do not go line by line.

You will understand how industry evaluates LLM applications.
You will see the pipelines for both non-RAG and RAG.
You will know why the metrics differ by architecture.
You will be able to interpret the scores.
And most important, you will connect the business objective to the evaluation design.

If you remember one thing from today, remember number five.
The business goal decides the metric.

**[EXTRA]**
Here is a way to test yourself at the end of the day.
Someone gives you a new LLM project.
Before they show you any code, can you ask the right three questions?
What is the business goal?
What does a bad answer cost?
What are the parts of the system that can fail?
If you can ask those, you have learned the main lesson.

**[CLICK to Section 1]**

---

# SECTION 1: INTRODUCTION TO LLM EVALUATION

---

## SLIDE 4 | Why LLM evaluation is different
**[6 min]**

Let us start with a comparison.

In traditional machine learning, the output is a label or a number.
Spam or not spam.
Price of a house.

There is one correct answer.
So we use accuracy, precision, recall, F1.
Failure is simple. The prediction is wrong.

**[EXTRA]**
Think of a spam filter.
For each email, the answer is yes or no.
You can count how many it got right.
Anyone can check the result, and two people will agree on it.

**[CLICK or POINT to right column]**

Now look at an LLM.
The output is open-ended text.
There are many acceptable answers.

Ask for a product description ten times.
You get ten different texts.
All of them could be right.

**[EXTRA]**
Here is an analogy.
Evaluating a classifier is like marking a multiple-choice exam.
Evaluating an LLM is like marking an essay.
With the essay, two good teachers can give different marks.
And you need a marking guide, a rubric, to be fair.
Much of today is about building that rubric with numbers.

And the typical failure is different.
The text is fluent. It sounds confident. But it is wrong.

**[EXTRA]**
This is the dangerous part.
A wrong house price looks wrong.
A wrong paragraph looks perfectly fine.
Fluent language lowers our guard.
People trust text that reads smoothly.

So what does good mean?
It is no longer a score above a threshold.
It is defined by the business objective.

**[ASK]** Has anyone here seen an LLM answer that sounded perfect but was wrong?
**[Take one or two answers. 1 minute.]**

**[EXTRA: if nobody answers, tell this one]**
A well-known real case is a lawyer who used a chatbot to prepare a court filing.
The chatbot produced case citations that looked real, with names, courts and dates.
Some of those cases did not exist.
The writing was fluent and confident, and that is why it slipped through.
No accuracy score on a label set would have caught that.

---

## SLIDE 5 | One metric is never enough
**[5 min]**

Because text has many qualities, one number is never enough.

Here are six dimensions.

Relevance. Does it answer what was asked?
Correctness. Are the facts right?
Fluency. Is the language natural?
Tone and brand fit. Does it sound like us?
Grounding. Is every claim supported by a source?
Safety. Is it free of harmful content?

A text can score well on fluency and fail on correctness.
A single score hides that trade-off.

**[EXTRA: one example per dimension, say quickly]**
Relevance. You ask about the refund policy and it explains shipping.
Correctness. It says the warranty is two years, but it is one.
Fluency. The grammar is broken or the phrasing is robotic.
Tone. A luxury brand writes copy that sounds like a street-market shout.
Grounding. It adds a claim that appears in no company document.
Safety. It produces content that is offensive or leaks private data.

**[EXTRA]**
Notice that these can fail independently.
A text can be fluent, relevant, on-tone and still invent a fact.
That is why a dashboard with several numbers beats one grand total.
Think of a car dashboard.
You do not want one light called "car health".
You want fuel, temperature and oil pressure separately.

**[PAUSE]**

Read the line at the bottom with me.
Metric choice follows the business objective and the system architecture.

We will return to this sentence many times today.

---

## SLIDE 6 | Two demonstrations, one application
**[4 min]**

This is the storyline for the whole session.

Demonstration one is a standalone LLM.
We evaluate it against reference outputs.
We use Accuracy, Exact Match, BLEU, ROUGE, METEOR and BERTScore.

**[POINT to the arrow]**

Then we add retrieval.
The application now has new parts, so it has new ways to fail.

**[EXTRA]**
A good way to picture this is a student.
The standalone LLM is a student taking a closed-book exam.
It answers from memory.
RAG is the same student taking an open-book exam.
Now two things can go wrong.
They can look up the wrong page.
Or they can look up the right page and still write a poor answer.
Evaluation has to tell those two apart.

Demonstration two is the RAG application.
Now we measure Faithfulness, Answer Relevance, and the Context metrics.

**[POINT to the bottom box]**

The goal today is not to memorise syntax.
It is to understand evaluation workflows, how to read the scores, and how to make design decisions from them.

**[EXTRA]**
Libraries change every few months.
Function names will change.
But the questions you ask of a score will not change.
That is the part worth keeping.

---

# SECTION 2: EVALUATING A NON-RAG LLM APPLICATION

---

## SLIDE 7 | Use case: marketing content generation
**[5 min]**

Before any code, we state the business objective.

Our use case is marketing content generation.
The company wants accurate, on-brand copy, faster, with a consistent tone.

**[EXTRA]**
Why did we choose marketing for the first demo?
Because everyone understands what good copy looks like.
You do not need domain knowledge to judge the output.
That lets us focus on the metrics, not the subject matter.

**[POINT to left cards]**

Quality means the copy is relevant to the brief, faithful to product facts, with the right tone and length.

The main risk is fluent text that drifts from the brief or invents product claims.

**[EXTRA]**
Let me make the risk concrete.
Suppose the model writes "waterproof" for a product that is only splash-resistant.
That is not a style problem.
It can lead to complaints, returns, even legal trouble in regulated markets.
So even in marketing, a factual slip has a real price.

**[POINT to right flow, top to bottom]**

The flow is simple.
A product brief goes into a prompt template.
A locally hosted LLM generates the copy.
We compare it with approved reference copy.
Then we score it and review the results.

**[Switch to the notebook for a quick look at the brief and the prompt. 1 minute.]**

**[EXTRA: while showing the prompt]**
Notice the prompt has three parts.
A role: you are a copywriter for this brand.
The facts: the product brief, which the model must not go beyond.
The rules: tone, length and format.
Most of what we tune later lives in these three parts.

---

## SLIDE 8 | Building the evaluation workflow
**[5 min]**

Four steps build this workflow.

One, construct the prompt.
System instruction, product brief, tone and length rules.

Two, choose representative inputs.
Different products, audiences and channels, such as email, social and web.

**[EXTRA]**
"Representative" is the key word.
If all your test inputs are easy, your scores will look great and mean little.
Include a short product, a long product, a product with many specs, and one tricky case.
For example, a product with a number that is easy to get wrong, like a battery life or a discount.

Three, prepare reference outputs.
For each input, we need expected copy that marketing has written or approved.

Four, generate and score.
Run the model on every input and compute the metrics for each output.

**[PAUSE]**

One important point.
The reference is a design decision. It is not absolute truth.
A different approved reference would give different scores.

**[EXTRA]**
Imagine two marketers each write a perfect tagline for the same bottle.
Score one against the other and you will not get a hundred percent.
So a low score does not always mean bad copy.
It can mean the reference is just one of many good answers.

A small, carefully chosen set is enough to show the workflow.

**[EXTRA]**
In real projects, teams start with thirty to a hundred well-chosen examples.
They grow the set whenever a new failure is found in production.
Every bug becomes a new test case.
That is how an evaluation set becomes valuable over time.

**[DEMO: Run the generation cells in the notebook. 3 minutes.]**

**[EXTRA: during the demo]**
While it runs, point out three things.
How long each generation takes on local hardware.
That the outputs differ in wording each time.
And that we store the output next to the reference so we can compare them side by side.

---

## SLIDE 9 | Traditional metrics at a glance
**[4 min]**

Here are the six traditional metrics in one table.

Read the table top to bottom, one line each.

Accuracy. Share of outputs judged correct.
Exact Match. Output identical to the reference. Very strict.
BLEU. N-gram precision. Punishes paraphrase.
ROUGE. N-gram recall. Shows how much of the reference is covered.
METEOR. Adds stems and synonyms.
BERTScore. Compares meaning using embeddings.

**[EXTRA]**
An n-gram is just a run of n words in a row.
One word is a unigram, two words a bigram, and so on.
Most of these metrics are really doing careful counting of matching word runs.

Notice the direction.
Metrics at the top are strict about words.
Metrics at the bottom are closer to meaning.

**[EXTRA]**
Think of it as a ladder.
The bottom rung asks: are these the exact same characters?
Higher up asks: do they share words? Then: do they share word forms and synonyms?
At the top it asks: do they mean the same thing?
Each step up is more forgiving, and more expensive to compute.

We will now look at each one in detail.

---

## SLIDE 10 | Deep dive: Exact Match and Accuracy
**[4 min]**

Start with the simplest.

Exact Match is one if the output equals the reference, otherwise zero.
Accuracy is correct outputs divided by total outputs.

**[POINT to How it works]**

We normalise case and spacing, compare the strings, score pass or fail, and average.

**[EXTRA: tiny example]**
Reference: "Order today and save 15 percent."
Output: "Order today and save 15 percent!"
One exclamation mark in place of a full stop, and Exact Match gives zero.
That is why normalisation, like removing punctuation and lowercasing, matters.

**[POINT to Strengths]**

It is simple, fast and transparent.
Perfect for labels, short answers and tags.

**[EXTRA]**
Good fits are things like: is the sentiment positive or negative, which category is this ticket, what is the invoice number.
There is one right string, and you can check it.

**[POINT to Limits]**

But one extra word and the score is zero.
For marketing copy, Exact Match will be near zero even when the copy is excellent.

So use it for classification-style checks, not for free-form copy.

**[EXTRA]**
A useful trick.
Even in free-form copy, you can use Exact Match on parts of the output.
For example, check that the discount "15 percent" appears, or that the product name is spelled correctly.
That is a small, rule-based check, and it catches exactly the factual slips the fancy metrics miss.

---

## SLIDE 11 | Deep dive: BLEU
**[5 min]**

BLEU comes from machine translation.

The formula looks heavy, so let me say it in plain words.
BLEU counts how many one to four word sequences in the output also appear in the reference.
That is precision.
Then it applies a brevity penalty, so very short outputs cannot cheat.

**[EXTRA: why the brevity penalty exists]**
Imagine the output is just one word, and that word is in the reference.
Precision would be a perfect one hundred percent.
Without a penalty, a tiny output could win.
The brevity penalty says: if you are much shorter than the reference, you lose points.

**[EXTRA: tiny example, pick numbers loosely]**
Reference: "keeps drinks cold for 24 hours".
Output: "keeps drinks chilled for 24 hours".
Most words match, but "chilled" does not match "cold".
The bigrams and trigrams that include that word also fail.
So the score drops sharply for one synonym.
That is why BLEU feels harsh on paraphrase.

**[POINT to Strengths]**

It is cheap to run on large sets.
Good for tracking change between prompt versions.

**[POINT to Limits]**

But it punishes valid paraphrase.
If I write chilled instead of cold, BLEU sees a mismatch.
It knows nothing about meaning, facts or tone.

So treat BLEU as a wording-similarity signal.
Never as a quality verdict.

**[EXTRA]**
In practice, BLEU values for good marketing copy are often low, because creative writing varies so much.
So do not panic at a score like 0.2 or 0.3.
What matters is the trend across versions, and whether the numbers agree with your own reading of the outputs.

---

## SLIDE 12 | Deep dive: ROUGE
**[4 min]**

ROUGE is the mirror image of BLEU.

BLEU asks how much of the output is in the reference.
ROUGE asks how much of the reference is in the output.
That is recall.

**[EXTRA]**
Here is a simple memory trick.
BLEU is about Being precise: is what I said in the reference?
ROUGE is about coverage: did I cover what the reference said?

ROUGE-1 counts shared words.
ROUGE-2 counts shared word pairs.
ROUGE-L finds the longest in-order match.

**[EXTRA]**
ROUGE-1 is forgiving, because single words match easily.
ROUGE-2 is stricter, because the pair has to match in order.
ROUGE-L checks whether the main structure of the sentence is kept, even with gaps in between.

It is popular for summaries and long text.

The limit to remember is length.
A longer output can raise recall without adding any quality.
So always read ROUGE with length in mind.

**[EXTRA]**
Picture a student who writes three pages for a one-page question.
By sheer volume, they will mention most of what the examiner wanted.
Recall looks high.
But the answer is padded, and nobody wants to read it.
That is why we compare ROUGE with output length, and often look at the F-measure, which balances both.

---

## SLIDE 13 | Deep dive: METEOR
**[3 min]**

METEOR is a softer overlap metric.

It aligns words by exact form, by stem, and by synonym.
So running matches run, and strong matches powerful.

It combines precision and recall, but favours recall.
It also penalises matches that are scattered in the wrong order.

**[EXTRA]**
The "wrong order" part is called a fragmentation penalty.
If the matched words appear as one smooth run, the penalty is small.
If they are scattered all over, the penalty is bigger.
That gives METEOR some sense of fluency, which BLEU and ROUGE lack.

It often tracks human judgement better than BLEU.

The cost is that it depends on lexical resources and runs slower.
And like the others, it cannot check facts.

**[EXTRA]**
Those lexical resources are mostly English.
So for other languages, METEOR support can be weaker.
Check that before you rely on it.

Use it when wording varies but meaning is close.

---

## SLIDE 14 | Deep dive: BERTScore
**[4 min]**

BERTScore moves from words to meaning.

It embeds every token of the output and the reference.
Then it matches each token to its most similar counterpart using cosine similarity.
The result is precision, recall and F1.

**[EXTRA]**
Here is the picture.
An embedding turns a word, in its sentence, into a list of numbers.
Words with similar meaning in similar contexts end up with similar numbers.
So "cold" and "chilled" are close together, even though the letters are different.
Cosine similarity measures how close the arrows point.

So two sentences with different words but the same idea score high.

The limits.
Scores cluster in a narrow high band, so small differences matter.
It depends on the embedding model.
And it checks similarity to the reference, not whether the facts are true.

**[EXTRA]**
Because the scores sit between roughly 0.85 and 0.95 for most reasonable text, a drop from 0.92 to 0.89 can be meaningful.
Some libraries offer "baseline rescaling" to stretch the range and make differences easier to read.
Also note that "24 hours" and "48 hours" are very close in embedding space.
That is exactly why BERTScore can reward a wrong number.

It is the best traditional metric for paraphrase.
But pair it with a factual check.

---

## SLIDE 15 | Worked example: three outputs, one reference
**[6 min]**

Let us make this concrete.

**[READ the reference aloud]**
Our new EcoBottle keeps drinks cold for 24 hours. Order today and save 15 percent.

**[Output A]**
It uses different words, but the same facts.
BLEU is low, because the wording changed.
BERTScore is high, because the meaning is the same.
This is good copy.

**[EXTRA: a possible Output A text you can read out]**
"Meet the EcoBottle: your drink stays chilled for a full day. Buy now and take 15 percent off."
Almost every word is different from the reference.
But a customer reads exactly the same offer.

**[Output B]**
Almost identical wording, but it says 48 hours instead of 24.
BLEU, ROUGE and BERTScore are all high.
But this copy is wrong, and it could mislead customers.

**[EXTRA: a possible Output B text]**
"Our new EcoBottle keeps drinks cold for 48 hours. Order today and save 15 percent."
Only one number changed.
To a human, that is a serious error.
To a word-overlap metric, it is a tiny difference.

**[Output C]**
Generic copy that ignores the brief.
All the scores are low.

**[EXTRA: a possible Output C text]**
"Stay hydrated with our premium range of products. Visit our store for great deals."
Nothing wrong with the grammar.
But no product, no number, no offer.

**[PAUSE]**

Output B is the lesson.
The metrics rewarded a factual error.

**[ASK]** Which of these three would you want to catch before publishing?

**[EXTRA: expected answer and follow-up]**
Most people say B.
Ask: which one would the metrics have flagged?
The answer is C.
So the metrics catch the output that is obviously off, and miss the one that is dangerously close.

**[Mention]** These are qualitative readings. We will see the real scores in the notebook.

---

## SLIDE 16 | When metrics disagree
**[4 min]**

Three patterns you will see again and again.

Pattern A. Same meaning, new wording.
BLEU is low, BERTScore is high.
The copy is acceptable. BLEU simply understates quality.

Pattern B. Close wording, wrong claim.
BLEU and ROUGE are high, but one product fact is wrong.
Overlap hides factual errors.
Add fact checks or an LLM judge.

**[EXTRA]**
An LLM judge is a second model that reads the output, plus the facts, and answers a question like "does this text contradict the brief?"
We will use exactly this idea in Section 4, so keep it in mind.

Pattern C. Right facts, off-brand voice.
Moderate scores everywhere.
No overlap metric captures voice.
Add a rubric or human review.

**[EXTRA]**
A rubric is a short checklist, for example: friendly tone yes or no, no slang, ends with a call to action.
Even a three-line rubric, applied consistently, tells you things BLEU never can.

**[POINT to the bottom bar]**
Read several metrics together.
And always inspect real outputs.

**[EXTRA]**
My personal rule is simple.
Before you trust any number, read ten real outputs.
If the numbers and your reading disagree, trust your reading first, then find out why.

---

## SLIDE 17 | Limits of traditional metrics
**[4 min]**

Let us summarise the limits.

They are tied to one reference, even when many answers are valid.
They reward surface wording rather than meaning.
They are blind to factual errors and brand voice.
And a score by itself has no business meaning.

**[EXTRA]**
That last point deserves a moment.
Suppose I tell you the BLEU score is 0.31.
Is that good?
It depends on the task, the references, the length, and the version before it.
A score only has meaning compared against something: a previous run, a baseline, or a business threshold.

**[POINT to right card]**

So what do we do with the results?
We improve the system.

Clarify the prompt and add examples.
Constrain length, format and tone.
Tune temperature and top-p.

**[POINT to the bottom bar]**
And keep the reference set as a regression test.
Re-score it after every change.

**[EXTRA]**
Regression test means: after a change, did anything that used to work get worse?
Developers do this for normal software.
The reference set plays the same role for your LLM application.

---

## SLIDE 18 | Levers for improving the application
**[5 min]**

Here are the five levers, in a table.

Prompt instructions.
Change the task, tone and length rules when outputs miss the brief.

Few-shot examples.
Show the model approved copy when the voice is inconsistent.

Temperature.
Lower gives stable scores. Higher gives varied wording.

Top-p.
Lower gives safer, more predictable text.

Max tokens.
Controls length, so it affects ROUGE recall and the BLEU penalty.

**[EXTRA: simple intuition for temperature and top-p]**
Temperature is how adventurous the model is when choosing the next word.
At zero it nearly always takes the most likely word.
At a high value it will sometimes take a surprising one.
Top-p limits the pool of words the model may pick from, to the most likely ones that add up to a chosen probability.
So low top-p means a short, safe list of candidates.

**[EXTRA: what to expect]**
For marketing, you may want a little variety, so a moderate temperature.
For scoring runs, use a low temperature, so that a change in score reflects a change in your system, not random luck.

**[PAUSE]**

One rule.
Change one lever at a time.
Re-run the same reference set.
Otherwise you will not know what caused the change.

**[EXTRA]**
If you change the prompt and the temperature at once, and the score goes up, which one helped?
You cannot tell.
That is the same discipline as a science experiment: change one variable.

**[DEMO: Change temperature in the notebook, re-run, compare scores. 3 minutes.]**

**[EXTRA: during the demo]**
Run the same input twice at a high temperature and show the scores differ.
That makes the point that some score movement is just noise.
Then say: this is why we want enough examples, and why we avoid reading too much into a small score change.

**[Section 2 complete. Move to Section 3.]**


---

# SECTION 3: BUILDING A RAG PIPELINE

---

## SLIDE 19 | RAG pipeline architecture
**[6 min]**

Now we change the architecture.

A standalone LLM answers from its training data only.
RAG lets it answer from our documents.

**[EXTRA]**
Why not just train the model on our documents?
Fine-tuning is slow, costly, and the knowledge gets frozen at training time.
When a policy changes next week, you would have to train again.
With RAG, you update the document, re-index it, and the system knows the new policy.
RAG also lets you show the source of each answer.

There are two phases.

**[POINT to top row: Indexing]**

Indexing happens once.
One, document collection. Load and split documents into chunks.
Two, embedding generation. Turn each chunk into a vector.
Three, vector storage. Save the vectors in ChromaDB.

**[EXTRA]**
Think of a library.
Indexing is the librarian cataloguing every book by topic, so that books on similar subjects sit near each other.
The catalogue is built once, and updated when new books arrive.

**[POINT to bottom row: Query time]**

Query time happens on every question.
Four, retrieval. Find the chunks closest to the question.
Five, context augmentation. Put those chunks into the prompt.
Six, response generation. The LLM answers from that context.

**[EXTRA]**
At query time, the same librarian takes your question, walks to the right shelf, brings back a few pages, and the writer composes an answer using only those pages.

Keep this picture in your head.
Every box is a place where the system can fail.

**[EXTRA: say slowly, one failure per box]**
Loading can miss a file or garble a table in a PDF.
Chunking can cut a sentence in half.
Embedding can place two related ideas far apart.
Storage and search can return near-matches that are not truly relevant.
Augmentation can stuff the prompt with too much text.
And generation can still ignore the context.
Six boxes, six ways to fail. This is why Section 4 needs more than one metric.

---

## SLIDE 20 | Tools behind each stage
**[4 min]**

Here is the tool for each stage.

LangChain loaders and splitters prepare the chunks.
A Hugging Face embedding model creates the vectors.
ChromaDB stores and searches them.
A LangChain retriever returns the top-k chunks.
A prompt template and chain combine question and context.
And a local LLM served with Ollama writes the answer.

**[EXTRA]**
Let me give each one a one-line role.
LangChain is the glue: it connects the steps in a chain.
The embedding model is the translator from text to numbers.
ChromaDB is a database that is good at "find me the nearest vectors".
Ollama is the engine that runs the language model on your own machine.
None of these is special. Each can be swapped for another tool with the same job.

Everything runs on this machine.
No data leaves it.

**[EXTRA]**
This design also makes experiments fair.
You can swap the embedding model, keep everything else the same, and measure the difference.
We will do exactly that kind of single-change experiment in Section 4.

---

## SLIDE 21 | Indexing and retrieval parameters
**[5 min]**

These parameters decide how well retrieval works.
They are starting points, not rules.

Chunk size. Five hundred to a thousand characters.
Bigger chunks give more context, but more noise.

**[EXTRA]**
Imagine cutting a book into index cards.
Cards that are too small hold half a thought, so the answer is split across cards.
Cards that are too large carry a lot of unrelated text, and the model has to dig for the useful sentence.

Chunk overlap. Ten to twenty percent.
It keeps a topic from being cut in half.

**[EXTRA]**
Overlap means the end of one chunk repeats at the start of the next.
So if an important sentence falls on the boundary, it still appears whole in at least one chunk.
The cost is a slightly bigger index and some repeated text.

Top-k. Three to five chunks.
Higher k improves recall but lowers precision.

**[EXTRA]**
Top-k is how many chunks we fetch.
Fetch more, and you are more likely to include the needed one. That is recall.
But you also bring more irrelevant ones, which adds noise and uses up prompt space. That is lower precision.
This trade-off is one you will see in the scores.

Embedding model. A compact sentence model.
Distance metric. Cosine similarity.

**[EXTRA]**
A bigger embedding model usually retrieves better, but is slower and needs more memory.
Pick the smallest one that meets your quality bar.

**[PAUSE]**

We will tune these in Section 4, using the evaluation scores.
That is the whole point of measuring.

---

## SLIDE 22 | Pipeline code walkthrough
**[10 min]**

Now the code.
**[SWITCH to the notebook and run cell by cell. Use this slide as a map.]**

**[Left card: Indexing]**
We load the documents from a folder.
We split them with a recursive character splitter.
Chunk size eight hundred, overlap one hundred.
Then we embed the chunks and store them in Chroma.

**[EXTRA: what to point at while running]**
After loading, print how many documents and how many pages came in.
After splitting, print the number of chunks and show two or three of them.
Ask the audience: does this chunk make sense on its own?
That simple look often reveals a problem before any metric does.
The recursive splitter tries paragraph breaks first, then sentences, then words.
That is why it usually respects natural boundaries better than a plain cut every N characters.

**[Right card: Query time]**
We create a retriever from the store with k equal to four.
We connect the Ollama model.
We retrieve the chunks for the question.
We build the prompt and call the model.

**[EXTRA: what to point at]**
Print the retrieved chunks for one question before generating the answer.
Show the audience the raw evidence the model will see.
Then show the prompt template, and read the instruction aloud:
"Answer only from the context below. If the answer is not in the context, say you do not know."
That sentence matters a great deal for faithfulness, and we will come back to it.

**[Say]** The code on the slide is simplified.
Exact imports can change between LangChain versions.
The notebook is the source of truth.

**[EXTRA: if something breaks live]**
Stay calm, and treat it as part of the lesson.
Read the error message aloud, because most errors are version or path problems.
Check that Ollama is running, the model name matches, and the Chroma folder exists.
Say: this is exactly what the last slide, on using an LLM as a partner, is for.

---

## SLIDE 23 | Standalone LLM versus RAG
**[4 min]**

Same questions, two applications.

The standalone LLM answers from training data.
It can sound confident while outdated or invented.
It has no link to company documents.
But it is simple to build.

RAG answers from retrieved company content.
It is grounded and traceable.

**[EXTRA]**
Traceable means that for any answer, you can point to the chunk and the source document it came from.
When a user asks "where did this come from?", you have an answer.
For compliance and trust, this is a big advantage.

**[POINT to the bottom bar]**

But note the trade-off.
Retrieval improves grounding, and it adds new places to fail.
Chunking, retrieval and context can all go wrong.

**[EXTRA]**
RAG also adds cost: more components to run, more latency per question, and more things to maintain.
So RAG is not always the right choice.
If the task does not need company knowledge, like the marketing copy in Demonstration one, a standalone model may be simpler and good enough.

**[DEMO: Ask one company-specific question to both. Compare answers. 2 minutes.]**

**[EXTRA: how to pick the demo question]**
Choose a fact that only exists in your documents, such as a leave policy number of days or a specific escalation step.
The standalone model will either refuse, or invent a plausible answer.
Highlight the invented answer: it will look just as confident as the correct one from RAG.
Ask the audience how they would tell the difference without seeing the document.

---

## SLIDE 24 | Successful and unsuccessful retrieval
**[5 min]**

Let me show both outcomes.

**[Left]**
Successful retrieval.
The question matches a section of a policy document.
The right chunks come back near the top.
The answer uses the correct content.

**[Right]**
Unsuccessful retrieval has four common causes.
The question wording differs from the document wording.
A chunk splits a topic in the middle.
Relevant chunks rank below irrelevant ones.
Or the relevant document does not exist at all.

**[EXTRA: one example per cause]**
Wording mismatch. The user asks about "time off for a new baby" and the document says "parental leave".
Embeddings help here, but they are not perfect.
Split topic. The leave policy lists eligibility in one chunk and the number of days in the next, and only one is retrieved.
Wrong ranking. A general HR overview ranks above the specific leave section because it mentions "leave" more often.
Missing document. The answer simply is not in the knowledge base.
In that last case, the best behaviour is for the system to say "I could not find this".

**[DEMO: Rephrase a question until retrieval fails. Show the retrieved chunks.]**

**[EXTRA: during the demo]**
Start with a question that works.
Then replace key terms with synonyms, step by step, and watch the retrieved chunks drift.
Ask the audience to predict when it will break.
This makes the abstract idea of retrieval failure visible.

**[POINT to the bottom line]**
These are the failures that RAG evaluation metrics are built to expose.

---

## SLIDE 25 | Enterprise knowledge assistants
**[3 min]**

Why does this matter outside a notebook?

Four examples.
An HR policy assistant for leave and benefits.
A support knowledge base that helps agents resolve tickets.
Compliance question and answer over regulations.
Sales enablement for product facts and pricing rules.

**[EXTRA: what a wrong answer costs in each]**
HR. An employee takes leave they are not entitled to, or misses leave they are.
Support. An agent gives a wrong fix, and the customer calls back angry.
Compliance. A wrong reading of a regulation can mean a fine.
Sales. A wrong discount rule is a quoted price the company has to honour or walk back.

In every case, answers must be correct, grounded and traceable to a source.
A wrong answer here is not just a bad sentence.
It is a business risk.

**[EXTRA]**
Notice this connects to our key sentence.
Because the risk is higher in compliance than in sales emails, the thresholds we set on the scores should be higher too.
The business goal decides the metric, and also the passing mark.

---

## SLIDE 26 | Break
**[10 min]**

Let us take a ten minute break.

We resume with Section 4.
There we measure how well this RAG pipeline works.

**[Use the break to verify the notebook and RAGAS setup are ready.]**

**[EXTRA: a question to leave with the audience]**
While you have your coffee, think about this.
If our RAG assistant gives a wrong answer, how would you find out whether the search or the writing was to blame?
We will answer that right after the break.


---

# SECTION 4: EVALUATING THE RAG APPLICATION

---

## SLIDE 27 | Why RAG needs more than quality scores
**[5 min]**

Welcome back.

**[EXTRA]**
Before the break I left you with a question.
Who is to blame when a RAG answer is wrong: the search or the writing?
Hold your answer for a minute.

Look at the flow across the top.
Question, retriever, retrieved context, generator, answer.

Here is the key idea.
A RAG answer can be wrong for two different reasons.
Either retrieval failed.
Or generation failed.

A single answer-quality score cannot tell these apart.

**[EXTRA]**
Think of a doctor who gets a wrong diagnosis.
Either the lab tests were wrong, or the doctor misread correct tests.
If you only know the diagnosis was wrong, you do not know whether to fix the lab or retrain the doctor.
Same here.

**[POINT to the two middle boxes]**

So we measure in layers.
Retrieval quality. Context Precision, Context Recall, Context Relevance.
Generation quality. Faithfulness, Answer Relevance.

**[POINT to the bottom bar]**
And end to end, Answer Correctness compares the final answer with the reference.

---

## SLIDE 28 | RAGAS metrics explained
**[4 min]**

RAGAS is an industry-standard framework for this.

**[EXTRA]**
RAGAS stands for Retrieval Augmented Generation Assessment.
It is open source, and a popular starting point.
Many teams also use other tools with similar ideas, so the concepts matter more than the package name.
Note that most of its metrics use a language model as a judge, so the scores can vary a little between runs and judge models.

Read the middle column as questions.

Faithfulness. Are the claims supported by the context?
Answer Relevance. Does it answer the question that was asked?
Context Precision. Are relevant chunks ranked near the top?
Context Recall. Did retrieval find everything the reference needs?
Context Relevance. How much of the context is useful?
Answer Correctness. How close is the final answer to the reference?

**[POINT to the right column]**
Notice the layer each one checks.
Generation, retrieval, or end to end.
That column is how we will diagnose problems.

**[EXTRA: memory aid]**
Faithfulness: "Did you make things up?"
Answer relevance: "Did you answer my question?"
Context precision: "Is the useful stuff at the top?"
Context recall: "Did you find everything needed?"
Correctness: "Is the final answer right?"
Five plain questions. You can ask them about any RAG system.

---

## SLIDE 29 | The evaluation dataset
**[3 min]**

RAGAS needs four fields.

Question. The user query.
Contexts. The chunks the retriever returned.
Answer. What the RAG application generated.
Ground truth. The reference answer from experts.

**[EXTRA]**
Notice that three of the four fields come from running the system.
Only the ground truth is written by humans.
That is the expensive part, so spend your expert time there.

**[POINT to the bottom bar]**
Build this set from real questions.
Then freeze it, so every run is comparable.

**[EXTRA]**
Where do real questions come from?
Support tickets, search logs, emails to HR, questions from new joiners.
Include easy questions, hard questions, questions with no answer in the documents, and questions with wording that differs from the documents.
Questions with no answer are especially valuable, because they test whether the system admits it does not know.

**[DEMO: Show one row of the dataset in the notebook.]**

---

## SLIDE 30 | Deep dive: Faithfulness
**[4 min]**

Faithfulness checks for hallucination.

An LLM judge splits the answer into separate claims.
Each claim is checked against the retrieved context.
Faithfulness is supported claims divided by total claims.

Example.
The answer makes five claims.
Four are in the context.
Faithfulness is point eight.

**[EXTRA: a concrete walk-through]**
Question: what is the leave policy for new parents?
Answer: "Employees get 26 weeks of paid leave. It starts up to two weeks before the due date. Fathers get two weeks. A doctor's note is required. Leave can be extended with approval."
The judge breaks this into five claims and checks each one against the retrieved text.
Suppose the context never mentions a doctor's note.
Then that claim is unsupported, and the score is four out of five.
Notice that the claim might be true in real life.
Faithfulness asks only: is it supported by the context we gave the model?

It needs no reference answer.
And it links directly to compliance and trust.

The limit.
It depends on the judge model.
And if the context itself is wrong, faithfulness cannot see that.

**[EXTRA]**
A weak judge model can be too lenient or too strict.
So for important decisions, read a sample of the judge's verdicts yourself.

**[POINT to the bottom bar]**
Low faithfulness with good contexts points to the prompt or the generator.

---

## SLIDE 31 | Deep dive: Answer Relevance
**[4 min]**

Answer Relevance asks a different question.
Not is it true, but does it fit what was asked.

The method is clever.
An LLM writes questions that the answer would satisfy.
We embed those questions.
Then we compare them with the original question.

If the generated questions look like the original, the answer is on topic.

**[EXTRA: an example]**
Original question: "How many days of casual leave do I get?"
Answer A: "You get 12 days of casual leave per year."
Questions an LLM would guess from A: "How many days of casual leave per year?" Very close to the original, so the score is high.
Answer B: "Leave must be applied for in the HR portal, and your manager approves it."
Questions guessed from B: "How do I apply for leave?" Different from the original, so the score is lower.
B may be true and useful, but it did not answer what was asked.

It catches off-topic and incomplete answers.
But it does not check truth.

**[POINT to the bottom bar]**
High relevance with low faithfulness means a confident, on-topic, ungrounded answer.
That is the dangerous combination.

**[EXTRA]**
This is the one to worry about most in compliance settings.
The answer looks perfect: it addresses the question, in fluent language.
Only the faithfulness score tells you it is not backed by the documents.

---

## SLIDE 32 | Deep dive: Context Precision and Recall
**[5 min]**

These two metrics evaluate the retriever alone.

Context Precision.
Of the chunks we retrieved, are the relevant ones ranked near the top?
Low precision means noise.

**[EXTRA: example]**
We retrieve four chunks, in this order: irrelevant, relevant, irrelevant, relevant.
Both relevant chunks were found, but the first slot, the most important one, is wasted.
Precision here is lower than for the order relevant, relevant, irrelevant, irrelevant.
Ranking matters because models pay most attention to what comes first, and long prompts dilute the signal.

Context Recall.
Split the reference answer into statements.
How many of them appear in the retrieved context?
Low recall means we missed needed information.

**[EXTRA: example]**
The reference answer has four statements.
Only three can be found in the retrieved chunks.
Recall is three out of four, which is point seven five.
The missing statement means the generator has nothing to build that part of the answer from, so it will either skip it or invent it.

Use precision to guide ranking, re-ranking and top-k.
Use recall to guide chunking and top-k.

Both need an LLM judge. Recall also needs a reference answer.

**[POINT to the bottom bar]**
If recall is low, the answer cannot be right.
Fix retrieval before touching the prompt.

**[EXTRA]**
Notice the tension between the two.
Raising top-k tends to improve recall and lower precision.
Re-ranking, which reorders the retrieved chunks with a stronger model, is a popular way to get both.

---

## SLIDE 33 | Deep dive: Answer Correctness
**[3 min]**

Answer Correctness is the headline number.

It blends two things.
Factual overlap between the answer and the reference.
And semantic similarity between them.

**[EXTRA]**
In RAGAS, the factual part works at the level of statements: which statements in the answer are also in the reference, which are extra, and which are missing.
The semantic part is an embedding similarity between the two texts.
The two are combined with weights, so correct facts in a different style still score well.

It gives one end-to-end view.
It rewards correct facts in different wording.
It is easy to trend across releases.

The limits.
It needs a trusted reference.
And it hides where the failure happened.

**[EXTRA]**
If the reference answer is outdated or wrong, correctness will punish a right answer.
So review references regularly, especially when policies change.

**[POINT to the bottom bar]**
Use correctness for the headline.
Then read the other metrics to diagnose.

---

## SLIDE 34 | Reading a scorecard
**[5 min]**

Let us practise reading a result.

**[Say]** These values are illustrative.
We will compare them with our real run.

Faithfulness point nine one.
Answer relevance point eight eight.
The generator is behaving well.

But context recall is point six two.
That is the weakest bar.
Needed facts are not reaching the prompt.

Precision is moderate.
Some noise gets through.

So what is the next step?
Not the prompt.
Tune chunking and top-k, then re-score.

**[EXTRA: why this surprises people]**
The natural reaction to a weak system is to rewrite the prompt.
Here that would waste time.
The generator is already faithful to what it receives.
It is simply not receiving enough.
Reading the scorecard saves you from fixing the wrong thing.

**[EXTRA: a possible experiment to describe]**
Change chunk size from 800 to 1000 and k from 4 to 6.
Re-run the same frozen questions.
If recall goes up to, say, 0.75 and precision drops a little, you have learned the trade-off for your own data.
Then decide whether the extra noise is acceptable.

**[ASK]** If faithfulness were low and recall were high, where would you look?
**[Expected answer: the prompt or the generator.]**

**[EXTRA: a second quick question]**
What if all the retrieval and generation scores look fine, but correctness is low?
**[Expected answer: suspect the reference answers or the source documents, as Slide 38 says.]**

**[DEMO: Run RAGAS on the real data and compare. 5 minutes if time allows.]**

**[EXTRA: during the demo]**
Sort the results by the lowest score and open the worst two or three questions.
Show the question, the retrieved chunks, the answer and the ground truth together.
Averages tell you how you are doing, but individual bad rows tell you what to fix.

---

## SLIDE 35 | Reading the scores
**[4 min]**

A simple rule of thumb for score bands.

Point eight to one is strong.
Safe to ship, keep monitoring.

Point five to point seven nine needs review.
Inspect the weakest queries.

Below point five is failing.
Find the root cause before release.

**[POINT to the four small cards]**
And a low score has a typical meaning.
Low faithfulness, the model adds unsupported claims.
Low answer relevance, off-topic or incomplete.
Low context precision, relevant chunks buried in noise.
Low context recall, needed information never retrieved.

**[Say]** These bands are only a rule of thumb.
Set real thresholds from the business risk of a wrong answer.

**[EXTRA: how to set a threshold]**
Ask: what does one wrong answer cost, and how many wrong answers can we live with?
A casual internal FAQ bot might accept faithfulness around 0.8.
A compliance assistant may demand 0.95 or higher, plus human review for what falls below.
Then agree the numbers with the business owner, and write them down before you test, not after.

**[EXTRA]**
Also remember that with a small question set, one or two questions can move an average a lot.
Look at the spread, not just the mean.

---

## SLIDE 36 | Retrieval and generation fail independently
**[4 min]**

This grid is the most useful picture in Section 4.

Top left. Good retrieval, good generation.
A healthy system.
Keep monitoring.

Top right. Good retrieval, poor generation.
The context is right but the answer is weak.
Fix the prompt, the model or the settings.

Bottom left. Poor retrieval, good-looking generation.
A fluent answer on weak context.
Fix chunking, embeddings or top-k.

**[EXTRA]**
This box is the sneakiest.
The answer reads well, so a casual reviewer approves it.
Only the context scores reveal that the foundation is weak.

Bottom right. Both poor.
Fix retrieval first.
Generation cannot recover missing context.

Then re-score the answers.

**[EXTRA]**
An easy way to remember the order: fix the input before the output.
If the model is given the wrong pages, even a perfect writer will produce a poor answer.

---

## SLIDE 37 | Evaluating standalone LLM versus RAG
**[4 min]**

Now compare the two evaluations side by side.

Main focus.
Standalone checks response quality against reference text.
RAG checks retrieval quality plus response quality.

Metrics.
BLEU, ROUGE, METEOR and BERTScore for standalone.
Faithfulness, relevance and the context metrics for RAG.

Grounding.
For a standalone model, facts cannot be checked without source context.
For RAG, they are checked against the retrieved context.

And the business question changes.
From does the copy fit the brief and tone,
to can staff trust answers from company documents.

**[EXTRA]**
One more difference.
In Demonstration one, the references are the only source of truth, so overlap metrics are the main tool.
In Demonstration two, the retrieved context is also a source of truth, so we can check claims against it without needing a perfect reference.
That is a big step forward, because it lets us detect hallucination even when we have no gold answer.

**[EXTRA]**
And the metrics can be combined.
Nothing stops you from using BERTScore on RAG answers, or an LLM judge on marketing copy.
Choose the tool to fit the question you need answered.

---

## SLIDE 38 | Locating the source of a failure
**[5 min]**

This table is your debugging guide.

Read the score pattern, then find the likely origin.

Low context recall.
Look at chunking, embeddings, top-k.
Resize chunks, raise k, try a stronger embedding model.

Low context precision.
Look at ranking and noise.
Add re-ranking or metadata filters, reduce k.

Low faithfulness with good contexts.
Look at generation and prompt design.
Tell the model to answer only from the context.

Low answer relevance.
Clarify the task, rewrite the query, add examples.

Low correctness while other scores are high.
Suspect the source documents or the reference answers.

**[EXTRA: one quick scenario to practise]**
Scenario: recall 0.9, precision 0.5, faithfulness 0.9, relevance 0.9.
Ask the room what to try.
Expected answer: retrieval finds the right information, but with too much noise, so try re-ranking or a smaller k.
Then give a second one: recall 0.5, everything else 0.9. Expected: fix chunking, k or embeddings.

**[EXTRA]**
Metadata filters mean restricting the search by tags, such as department, document type or date.
For example, only search HR documents for an HR question.
That removes a whole class of irrelevant chunks.

**[Say]** Take a photo of this slide. You will use it.

---

## SLIDE 39 | Evaluation-driven improvement loop
**[5 min]**

Evaluation is not a one-time gate.
It is a loop.

Measure. Score a fixed question set.
Diagnose. Locate retrieval or generation faults.
Change. Adjust the prompt, chunks or retrieval.
Re-evaluate. Compare with the previous run.

**[EXTRA]**
Keep a simple log: date, what changed, scores before and after.
After a few iterations, that log is the story of how your system improved, and a record you can show stakeholders.

**[POINT to the right box]**
And link scores to business outcomes.
For compliance risk, set a high faithfulness threshold.
For support resolution time, track context recall on common tickets.
For brand risk, add tone review to the rubric.

**[EXTRA]**
Evaluation should continue after launch.
Documents change, users ask new things, and models get updated.
Sample real questions every week, add the failures to your frozen set, and re-run.
Scores that were fine in March can drift by September without anyone touching the code.

**[Say]** A score is only useful if someone owns the decision it supports.

**[Section 4 complete. Move to the conclusion.]**

---

# SECTION 5: CONCLUSION AND Q AND A

---

## SLIDE 40 | Key takeaways
**[4 min]**

Let me close with five takeaways.

One. Evaluation is multi-dimensional.
No single score captures quality.

Two. Reference metrics have limits.
BLEU, ROUGE, METEOR and BERTScore miss facts and tone.

Three. RAG adds new dimensions.
Measure retrieval and generation separately.

Four. Metrics follow the objective.
Start from the business goal.

Five. Evaluation drives improvement.
Scores guide fixes.

**[EXTRA: tie each takeaway to a moment from today]**
One: the dashboard analogy.
Two: Output B, the wrong 48 hours that scored high.
Three: the scorecard where recall was the weak bar and the prompt was fine.
Four: the same product facts need different thresholds in a sales email and a compliance assistant.
Five: the loop, with a frozen question set and a log of changes.

**[POINT to the bottom right box]**
The workflow in one line.
Build, generate, evaluate, diagnose, improve.

**[EXTRA: suggested next steps for the audience]**
Take one real use case from your own work.
Write down the business goal and the cost of a wrong answer.
Collect twenty real questions with expert answers.
Run the pipeline and the scores once, and read the worst three rows.
That small exercise is worth more than any tool comparison.

---

## SLIDE 41 | Use LLMs as a productivity partner
**[3 min]**

One last practical tip.
Use an LLM as your own learning partner.

For knowledge discovery.
Ask it to explain context recall using an HR assistant as the example.

For code and debugging.
Paste the cell and the full error. Ask for the cause and a fix.

For tool troubleshooting.
Share versions and the exact message from LangChain, ChromaDB or Ollama.

**[EXTRA: tips for good prompts]**
Include your goal, your versions, the exact error text, and what you already tried.
Ask for an explanation as well as a fix, so you learn why it broke.
Ask it to quiz you on a metric once you think you understand it.

**[EXTRA: a good irony]**
Notice that everything we discussed today applies to this advice too.
An answer from an LLM about your code is fluent and confident, and can still be wrong.
So evaluate it: run the code, check the documentation, and test the fix.

**[POINT to the bottom bar]**
But always verify AI-generated code and explanations against official documentation.

---

## SLIDE 42 | Questions and discussion
**[3 min or more]**

Thank you.

The floor is open.
Questions on metric choice.
Questions on RAG design.
Questions on debugging your own setup.

If we run out of time, I will take questions after the session.

**[EXTRA: if the room is quiet, seed with one of these]**
How many examples do I need in my evaluation set?
Answer: start with thirty to fifty good ones, and grow it from real failures.
Can I use a small local model as the RAGAS judge?
Answer: yes, but weaker judges are less reliable, so spot-check their verdicts.
Should I use human review or automated metrics?
Answer: both. Automated metrics for speed and trends, human review for tone, edge cases and calibration.
When is RAG not the right choice?
Answer: when the task does not depend on company knowledge, or the knowledge is small and stable enough to fit directly in the prompt.

**[Closing line]**
Remember. Evaluation choices follow the business objective and the application architecture.

Thank you, everyone.

---

# TRAINER QUICK NOTES

**Timing checkpoints**

| Clock | You should be at |
|---|---|
| 0:20 | End of Section 1 (Slide 6) |
| 1:15 | End of Section 2 (Slide 18) |
| 1:55 | End of Section 3, start break (Slide 26) |
| 2:05 | Back from break, Section 4 starts (Slide 27) |
| 2:55 | End of Section 4 (Slide 39) |
| 3:00 | Wrap-up and Q and A |

**About the extra material**

The EXTRA blocks add roughly 15 to 20 minutes if you read all of them.
Plan to use about half. Suggested priority, highest first:

1. Slide 15 example texts (A, B, C) and the follow-up question.
2. Slide 19 failure-per-box list.
3. Slide 27 doctor analogy.
4. Slide 30 faithfulness walk-through.
5. Slide 34 "why fix recall, not the prompt".
6. Slide 38 practice scenarios.

**If you are running late, shorten these first**

1. Slides 12 and 13. Mention ROUGE and METEOR in one sentence each, and skip their EXTRA blocks.
2. Slide 25. Skip it.
3. Slide 37. Merge with Slide 36.
4. All EXTRA blocks on Slides 1 to 3, 9 and 20.

**If you are running early, extend these**

1. Slide 15. Add your own example outputs.
2. Slide 22. Take more questions on the code.
3. Slide 34. Run RAGAS live on a second question set.
4. Slide 42. Use the seed questions.

**Before you start, check**

- Ollama is running and the model is loaded.
- The ChromaDB folder exists, or the indexing cell has been run once.
- The evaluation dataset loads without errors.
- The RAGAS judge model responds.
- Replace the illustrative values on Slides 15 and 34 with your real notebook scores if you prefer.
- Have one company-specific question ready for the Slide 23 demo, and one that you know will fail retrieval for Slide 24.
