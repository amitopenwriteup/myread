# Section 6 Trainer Script: Conclusion & Q&A

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 10 minutes (about 5 minutes recap and bridge, 5 minutes Q&A)

> **Purpose of this section:** Bring the whole session together, point to what comes next, and give learners a safe space to ask questions. Keep the recap active. Make learners say the ideas, not you.

---

## Timing Plan

| Slide | Topic | Time |
|---|---|---|
| 1 | Title | 30 sec |
| 2 | Session recap | 2 min |
| 3 | Key factors for selecting models | 1 min 30 sec |
| 4 | What comes next | 1 min |
| 5 | Questions | 5 min |

> **Trainer note:** The deck's notes give about 6 minutes to the recap and bridge slides plus 5 for Q&A, which is slightly over. The plan above trims the recap so Q&A keeps its full 5 minutes. If the room is short of questions, finish early rather than stretching.

---

## Before You Start

- Keep a note of the questions parked during earlier sections. Offer to answer them now.
- Check the programme schedule for the **exact names and dates** of the upcoming C9 sessions. You will name them on slide 4.
- Have the Section 5 worksheet link or file ready, if you plan to share it.

---

## SLIDE 1: Title (30 sec)

**SAY**
1. Welcome to our final section: Conclusion and Q&A.
2. We have 10 minutes.
3. About five minutes to recap and look ahead.
4. And about five minutes for your questions.
5. Let's start by seeing what stuck.

---

## SLIDE 2: Session Recap (2 min)

> **Trainer note:** Before revealing each card, ask learners to name one idea from it. Cover the card text if you can, or move quickly so they speak first.

**ASK (retrieval round, one card at a time)**
1. Let's do this as a quick recall round. No notes.
2. **DO** Point to the first card, "Open-weight vs API", but do not read it.
3. What is the key difference between open-weight and API models?
4. **DO** Take one or two answers.
5. **SAY** Good. Here is the summary.
6. Open-weight offers control, but you take on deployment responsibility.
7. Proprietary or API models offer convenience, but also vendor dependency.
8. And remember: open-weight is not necessarily open source.

9. **DO** Point to the second card, "Families & sizes".
10. **ASK** What should we remember about model size?
11. **DO** Take one or two answers.
12. **SAY** Here is the summary.
13. Families offer variants for different jobs.
14. Parameter count is one rough measure only.
15. And mixture-of-experts separates **total** parameters from **active** parameters.
16. Total drives memory. Active drives compute.

17. **DO** Point to the third card, "Capabilities".
18. **ASK** Which capabilities matter most for agents?
19. **DO** Take one or two answers.
20. **SAY** Here is the summary.
21. Reasoning, tool calling, structured output, context handling, coding and multimodality.
22. Remember: tool calling and structured output can fail first, and they break agent loops.

23. **DO** Point to the fourth card, "Quantisation".
24. **ASK** What does quantisation give us, and what does it risk?
25. **DO** Take one or two answers.
26. **SAY** Here is the summary.
27. Lower precision cuts memory, and often improves speed.
28. But there may be quality loss, and it must be tested.
29. And you can estimate memory with one formula: parameters times bytes per parameter, plus cache and overhead.

**If learners are quiet, prompt with:**
- "Think of the 7B model. How much memory at FP16? At INT4?" (About 14 GB and 3.5 GB.)
- "Which would you test first, general chat or tool calls?" (Tool calls.)

---

## SLIDE 3: Key Factors for Selecting Models (1 min 30 sec)

**SAY**
1. Now the three factors for choosing a model. Together they are the heart of the session.
2. **DO** Point to "Task needs".
3. First, task needs.
4. Complexity, reasoning, tool calling, structured output, context and modality.
5. What does the **task** require?
6. **DO** Point to "System constraints".
7. Second, system constraints.
8. Latency, cost, data governance, hardware and deployment.
9. What does the **system** allow?
10. **DO** Point to "Evidence".
11. Third, evidence.
12. Representative tasks and system-level tests. Not benchmarks alone.
13. (pause)
14. Here is the sentence to take home.
15. **The decision is made on task and system needs, and then validated by testing.**
16. Remember the framework from Section 5.
17. Define. Constrain. Shortlist. Test. Decide.
18. And iterate. Test results can send you back to earlier steps.

---

## SLIDE 4: What Comes Next (1 min)

**SAY**
1. Last slide before questions. What comes next?
2. **DO** Point to bullet 1.
3. First: model choice is only **one component**.
4. Tools, orchestration and application logic shape agent behaviour too.
5. Remember the shaded box from Section 1. The model is the decision-maker, but not the whole agent.
6. **DO** Point to bullet 2.
7. Second: the upcoming C9 sessions cover agentic AI system design and implementation.
8. [Name the specific sessions and dates from the programme schedule.]
9. **DO** Point to bullet 3.
10. Third: apply today's framework when you choose models for those builds.
11. **DO** Point to bullet 4.
12. And fourth: revisit your choice.
13. Models, prices and your own requirements will all change.
14. Re-test when they do.

**SAY (suggested action)**
15. Before the next session, try this.
16. Take one agent you care about, and fill in the selection worksheet.
17. Define it. Constrain it. Shortlist two to four models.
18. Bring it with you, and we can use it in the build.

---

## SLIDE 5: Questions? (5 min)

**SAY**
1. That is the content.
2. **DO** Point to the topic line at the bottom.
3. We covered open-weight versus API, families and sizes, capabilities, quantisation and model selection.
4. Now, over to you.
5. What questions do you have? Anything from today, or anything we parked earlier.

**DO**
6. Wait at least five to eight seconds after asking. Silence is normal. Do not fill it quickly.
7. Take questions one at a time. Repeat each question so the whole room hears it.
8. If you do not know an answer, say so, and offer to find out.

**If questions dry up, prompt with:**
9. "Which constraint would decide the model for **your own project**?"
10. **DO** Take two or three answers. Link each back to a section.

**Other prompts if needed**
- "Would your project need an open-weight model, an API model, or both?"
- "What would be the first thing you test on your shortlist?"
- "Which part of today's session would you like to revisit?"

**SAY (closing, 30 seconds)**
11. Thank you, everyone.
12. To summarise in one line: choose the model from the task and the system, then prove it with testing.
13. See you at the next session.

---

## Anticipated Questions and Suggested Answers

### From Section 1 (foundation models and the agent loop)

| Question | Suggested answer |
|---|---|
| Is the agent the same as the model? | No. The model is the decision-maker. The agent also includes orchestration, memory, guardrails, tools and application logic. |
| Can good prompting fix a weak model? | It helps, but cannot create capabilities the model lacks, such as multi-step planning. |

### From Section 2 (landscape)

| Question | Suggested answer |
|---|---|
| Which is better, open-weight or API models? | Neither in general. Open-weight gives control and data residency, but you operate it. APIs give convenience and peak capability, but create vendor dependency. |
| Can I use any open-weight model commercially? | Not automatically. Licences differ per model. Read the licence file every time. |
| Which model names should I remember? | Remember the classes (frontier, balanced, fast; large, mid, small open-weight), not the names. The landscape changes monthly. |

### From Section 3 (families, sizes, capabilities)

| Question | Suggested answer |
|---|---|
| Does more parameters mean a better model? | Not necessarily. Data quality, post-training and architecture matter as much as size. |
| What is the difference between total and active parameters? | In a mixture-of-experts model, total drives memory and active drives compute and speed. |
| Is a big context window enough? | No. It is capacity, not proof of reliable use. Test long-context accuracy on your own documents. |

### From Section 4 (quantisation)

| Question | Suggested answer |
|---|---|
| How do I estimate memory? | Parameters × bytes per parameter, plus KV cache and overhead. A 7B model is about 14 GB at FP16, 7 GB at INT8, 3.5 GB at INT4. |
| Does quantisation always make a model faster? | Often, but it depends on hardware and kernel support. |
| Will quantisation hurt my agent? | It can, especially tool calls and JSON output, before general chat quality. Test on your real tasks. |

### From Section 5 (selection)

| Question | Suggested answer |
|---|---|
| Where do I start when choosing a model? | Define the task and success criteria. Then constrain, shortlist 2 to 4, test, and decide. |
| Why do constraints come first? | They often eliminate options outright, for example when data cannot leave your environment. |
| Can one agent use several models? | Yes. Route each step to the cheapest model that meets its requirements. |
| How often should I re-test? | Whenever models, prompts, tools, quantisation settings, prices or requirements change. |
| Should I trust benchmark scores? | Use them to shortlist. Decide using your own tasks and the full agent loop. |

### Forward-looking and practical

| Question | Suggested answer |
|---|---|
| Which model should I use for my project? | It depends on your task and constraints. Use the Section 5 worksheet, then test two to four candidates. |
| How do I keep up with new models? | Focus on classes and dimensions, not names. Recheck provider pages, licences and prices before each project. |
| What is covered in the next sessions? | Agentic AI system design and implementation. [Confirm names and dates from the programme schedule.] |

> **Trainer note:** If a question goes deep on something outside the course (for example, training a model from scratch), say: "Great question. Let's take it offline at the break," and write it down.

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 30 sec |
| 2. Recap | 2 min |
| 3. Key factors | 1 min 30 sec |
| 4. What comes next | 1 min |
| 5. Q&A | 5 min |

If you run over, keep the recall round to one answer per card. **Protect the Q&A time.**

---

## Whole-Session Cheat Sheet (for your own last look before closing)

| Section | One idea to leave in the room |
|---|---|
| 1. Foundation models in agentic AI | The model is the decision-maker, but only one part of the agent. Model sets the ceiling; system design decides how much you reach. |
| 2. Industry model landscape | Open-weight gives control and puts operations on you. APIs give convenience and create vendor dependency. Think in classes, not names. |
| 3. Families, sizes, capabilities | Parameter count is a rough guide. Total drives memory; active drives compute. Tool calling and structured output decide whether the loop works. |
| 4. Quantisation | Memory ≈ parameters × bytes per parameter. Lower precision saves memory, but must be tested on tool calls and JSON. |
| 5. Model selection | Define, constrain, shortlist, test, decide. Constraints often eliminate options first. One agent can use several models. |
| 6. Conclusion | Choose on task and system needs. Validate by testing. Revisit as things change. |
