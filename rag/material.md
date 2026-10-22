# LLM Evaluation Demonstration
## Trainer Script (Teleprompter Format)

**How to use this script**

- Read the plain text out loud. It is written the way you speak.
- Text in **[BRACKETS]** is a stage cue. Do not read it aloud.
- Each line is one breath. Pause at every blank line.
- Times are guides. Total session: 3 hours, including a 10 minute break.

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

**[CLICK or POINT to right column]**

Now look at an LLM.
The output is open-ended text.
There are many acceptable answers.

Ask for a product description ten times.
You get ten different texts.
All of them could be right.

And the typical failure is different.
The text is fluent. It sounds confident. But it is wrong.

So what does good mean?
It is no longer a score above a threshold.
It is defined by the business objective.

**[ASK]** Has anyone here seen an LLM answer that sounded perfect but was wrong?
**[Take one or two answers. 1 minute.]**

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

Demonstration two is the RAG application.
Now we measure Faithfulness, Answer Relevance, and the Context metrics.

**[POINT to the bottom box]**

The goal today is not to memorise syntax.
It is to understand evaluation workflows, how to read the scores, and how to make design decisions from them.

---

# SECTION 2: EVALUATING A NON-RAG LLM APPLICATION

---

## SLIDE 7 | Use case: marketing content generation
**[5 min]**

Before any code, we state the business objective.

Our use case is marketing content generation.
The company wants accurate, on-brand copy, faster, with a consistent tone.

**[POINT to left cards]**

Quality means the copy is relevant to the brief, faithful to product facts, with the right tone and length.

The main risk is fluent text that drifts from the brief or invents product claims.

**[POINT to right flow, top to bottom]**

The flow is simple.
A product brief goes into a prompt template.
A locally hosted LLM generates the copy.
We compare it with approved reference copy.
Then we score it and review the results.

**[Switch to the notebook for a quick look at the brief and the prompt. 1 minute.]**

---

## SLIDE 8 | Building the evaluation workflow
**[5 min]**

Four steps build this workflow.

One, construct the prompt.
System instruction, product brief, tone and length rules.

Two, choose representative inputs.
Different products, audiences and channels, such as email, social and web.

Three, prepare reference outputs.
For each input, we need expected copy that marketing has written or approved.

Four, generate and score.
Run the model on every input and compute the metrics for each output.

**[PAUSE]**

One important point.
The reference is a design decision. It is not absolute truth.
A different approved reference would give different scores.

A small, carefully chosen set is enough to show the workflow.

**[DEMO: Run the generation cells in the notebook. 3 minutes.]**

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

Notice the direction.
Metrics at the top are strict about words.
Metrics at the bottom are closer to meaning.

We will now look at each one in detail.

---

## SLIDE 10 | Deep dive: Exact Match and Accuracy
**[4 min]**

Start with the simplest.

Exact Match is one if the output equals the reference, otherwise zero.
Accuracy is correct outputs divided by total outputs.

**[POINT to How it works]**

We normalise case and spacing, compare the strings, score pass or fail, and average.

**[POINT to Strengths]**

It is simple, fast and transparent.
Perfect for labels, short answers and tags.

**[POINT to Limits]**

But one extra word and the score is zero.
For marketing copy, Exact Match will be near zero even when the copy is excellent.

So use it for classification-style checks, not for free-form copy.

---

## SLIDE 11 | Deep dive: BLEU
**[5 min]**

BLEU comes from machine translation.

The formula looks heavy, so let me say it in plain words.
BLEU counts how many one to four word sequences in the output also appear in the reference.
That is precision.
Then it applies a brevity penalty, so very short outputs cannot cheat.

**[POINT to Strengths]**

It is cheap to run on large sets.
Good for tracking change between prompt versions.

**[POINT to Limits]**

But it punishes valid paraphrase.
If I write chilled instead of cold, BLEU sees a mismatch.
It knows nothing about meaning, facts or tone.

So treat BLEU as a wording-similarity signal.
Never as a quality verdict.

---

## SLIDE 12 | Deep dive: ROUGE
**[4 min]**

ROUGE is the mirror image of BLEU.

BLEU asks how much of the output is in the reference.
ROUGE asks how much of the reference is in the output.
That is recall.

ROUGE-1 counts shared words.
ROUGE-2 counts shared word pairs.
ROUGE-L finds the longest in-order match.

It is popular for summaries and long text.

The limit to remember is length.
A longer output can raise recall without adding any quality.
So always read ROUGE with length in mind.

---

## SLIDE 13 | Deep dive: METEOR
**[3 min]**

METEOR is a softer overlap metric.

It aligns words by exact form, by stem, and by synonym.
So running matches run, and strong matches powerful.

It combines precision and recall, but favours recall.
It also penalises matches that are scattered in the wrong order.

It often tracks human judgement better than BLEU.

The cost is that it depends on lexical resources and runs slower.
And like the others, it cannot check facts.

Use it when wording varies but meaning is close.

---

## SLIDE 14 | Deep dive: BERTScore
**[4 min]**

BERTScore moves from words to meaning.

It embeds every token of the output and the reference.
Then it matches each token to its most similar counterpart using cosine similarity.
The result is precision, recall and F1.

So two sentences with different words but the same idea score high.

The limits.
Scores cluster in a narrow high band, so small differences matter.
It depends on the embedding model.
And it checks similarity to the reference, not whether the facts are true.

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

**[Output B]**
Almost identical wording, but it says 48 hours instead of 24.
BLEU, ROUGE and BERTScore are all high.
But this copy is wrong, and it could mislead customers.

**[Output C]**
Generic copy that ignores the brief.
All the scores are low.

**[PAUSE]**

Output B is the lesson.
The metrics rewarded a factual error.

**[ASK]** Which of these three would you want to catch before publishing?

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

Pattern C. Right facts, off-brand voice.
Moderate scores everywhere.
No overlap metric captures voice.
Add a rubric or human review.

**[POINT to the bottom bar]**
Read several metrics together.
And always inspect real outputs.

---

## SLIDE 17 | Limits of traditional metrics
**[4 min]**

Let us summarise the limits.

They are tied to one reference, even when many answers are valid.
They reward surface wording rather than meaning.
They are blind to factual errors and brand voice.
And a score by itself has no business meaning.

**[POINT to right card]**

So what do we do with the results?
We improve the system.

Clarify the prompt and add examples.
Constrain length, format and tone.
Tune temperature and top-p.

**[POINT to the bottom bar]**
And keep the reference set as a regression test.
Re-score it after every change.

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

**[PAUSE]**

One rule.
Change one lever at a time.
Re-run the same reference set.
Otherwise you will not know what caused the change.

**[DEMO: Change temperature in the notebook, re-run, compare scores. 3 minutes.]**

**[Section 2 complete. Move to Section 3.]**

---

# SECTION 3: BUILDING A RAG PIPELINE

---

## SLIDE 19 | RAG pipeline architecture
**[6 min]**

Now we change the architecture.

A standalone LLM answers from its training data only.
RAG lets it answer from our documents.

There are two phases.

**[POINT to top row: Indexing]**

Indexing happens once.
One, document collection. Load and split documents into chunks.
Two, embedding generation. Turn each chunk into a vector.
Three, vector storage. Save the vectors in ChromaDB.

**[POINT to bottom row: Query time]**

Query time happens on every question.
Four, retrieval. Find the chunks closest to the question.
Five, context augmentation. Put those chunks into the prompt.
Six, response generation. The LLM answers from that context.

Keep this picture in your head.
Every box is a place where the system can fail.

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

Everything runs on this machine.
No data leaves it.

---

## SLIDE 21 | Indexing and retrieval parameters
**[5 min]**

These parameters decide how well retrieval works.
They are starting points, not rules.

Chunk size. Five hundred to a thousand characters.
Bigger chunks give more context, but more noise.

Chunk overlap. Ten to twenty percent.
It keeps a topic from being cut in half.

Top-k. Three to five chunks.
Higher k improves recall but lowers precision.

Embedding model. A compact sentence model.
Distance metric. Cosine similarity.

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

**[Right card: Query time]**
We create a retriever from the store with k equal to four.
We connect the Ollama model.
We retrieve the chunks for the question.
We build the prompt and call the model.

**[Say]** The code on the slide is simplified.
Exact imports can change between LangChain versions.
The notebook is the source of truth.

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

**[POINT to the bottom bar]**

But note the trade-off.
Retrieval improves grounding, and it adds new places to fail.
Chunking, retrieval and context can all go wrong.

**[DEMO: Ask one company-specific question to both. Compare answers. 2 minutes.]**

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

**[DEMO: Rephrase a question until retrieval fails. Show the retrieved chunks.]**

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

In every case, answers must be correct, grounded and traceable to a source.
A wrong answer here is not just a bad sentence.
It is a business risk.

---

## SLIDE 26 | Break
**[10 min]**

Let us take a ten minute break.

We resume with Section 4.
There we measure how well this RAG pipeline works.

**[Use the break to verify the notebook and RAGAS setup are ready.]**

---

# SECTION 4: EVALUATING THE RAG APPLICATION

---

## SLIDE 27 | Why RAG needs more than quality scores
**[5 min]**

Welcome back.

Look at the flow across the top.
Question, retriever, retrieved context, generator, answer.

Here is the key idea.
A RAG answer can be wrong for two different reasons.
Either retrieval failed.
Or generation failed.

A single answer-quality score cannot tell these apart.

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

---

## SLIDE 29 | The evaluation dataset
**[3 min]**

RAGAS needs four fields.

Question. The user query.
Contexts. The chunks the retriever returned.
Answer. What the RAG application generated.
Ground truth. The reference answer from experts.

**[POINT to the bottom bar]**
Build this set from real questions.
Then freeze it, so every run is comparable.

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

It needs no reference answer.
And it links directly to compliance and trust.

The limit.
It depends on the judge model.
And if the context itself is wrong, faithfulness cannot see that.

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

It catches off-topic and incomplete answers.
But it does not check truth.

**[POINT to the bottom bar]**
High relevance with low faithfulness means a confident, on-topic, ungrounded answer.
That is the dangerous combination.

---

## SLIDE 32 | Deep dive: Context Precision and Recall
**[5 min]**

These two metrics evaluate the retriever alone.

Context Precision.
Of the chunks we retrieved, are the relevant ones ranked near the top?
Low precision means noise.

Context Recall.
Split the reference answer into statements.
How many of them appear in the retrieved context?
Low recall means we missed needed information.

Use precision to guide ranking, re-ranking and top-k.
Use recall to guide chunking and top-k.

Both need an LLM judge. Recall also needs a reference answer.

**[POINT to the bottom bar]**
If recall is low, the answer cannot be right.
Fix retrieval before touching the prompt.

---

## SLIDE 33 | Deep dive: Answer Correctness
**[3 min]**

Answer Correctness is the headline number.

It blends two things.
Factual overlap between the answer and the reference.
And semantic similarity between them.

It gives one end-to-end view.
It rewards correct facts in different wording.
It is easy to trend across releases.

The limits.
It needs a trusted reference.
And it hides where the failure happened.

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

**[ASK]** If faithfulness were low and recall were high, where would you look?
**[Expected answer: the prompt or the generator.]**

**[DEMO: Run RAGAS on the real data and compare. 5 minutes if time allows.]**

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

Bottom right. Both poor.
Fix retrieval first.
Generation cannot recover missing context.

Then re-score the answers.

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

**[POINT to the right box]**
And link scores to business outcomes.
For compliance risk, set a high faithfulness threshold.
For support resolution time, track context recall on common tickets.
For brand risk, add tone review to the rubric.

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

**[POINT to the bottom right box]**
The workflow in one line.
Build, generate, evaluate, diagnose, improve.

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

**If you are running late, shorten these first**

1. Slides 12 and 13. Mention ROUGE and METEOR in one sentence each.
2. Slide 25. Skip it.
3. Slide 37. Merge with Slide 36.

**If you are running early, extend these**

1. Slide 15. Add your own example outputs.
2. Slide 22. Take more questions on the code.
3. Slide 34. Run RAGAS live on a second question set.

**Before you start, check**

- Ollama is running and the model is loaded.
- The ChromaDB folder exists, or the indexing cell has been run once.
- The evaluation dataset loads without errors.
- The RAGAS judge model responds.
- Replace the illustrative values on Slides 15 and 34 with your real notebook scores if you prefer.
