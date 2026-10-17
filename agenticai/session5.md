# Section 5 Trainer Script: Model Selection for Agentic AI Systems

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 30 minutes

> **Purpose of this section:** This is where Sections 1 to 4 come together. Capabilities, size, deployment and quantisation stop being facts and become **inputs to a decision**. Keep the tone practical.

---

## Before You Start

- Have the Section 1 dimensions slide in mind. You will refer back to it.
- Think of one example agent that learners know, for the slide 4 rating exercise. A simple one works well, such as an HR-policy assistant or an invoice-processing agent.
- Keep your notes from Section 4 on the hardware learners have. You will use them on slide 5 and slide 8.

---

## Timing Plan

| Slide | Topic | Time |
|---|---|---|
| 1 | Title and link to Sections 1 to 4 | 1 min |
| 2 | What we will cover | 1 min |
| 3 | A practical selection framework | 3 min |
| 4 | Task requirements | 3 min |
| 5 | System constraints | 3 min |
| 6 | Key trade-offs | 4 min |
| 7 | Scenario 1: customer-support agent | 4 min |
| 8 | Scenario 2: on-premises contract review | 4 min |
| 9 | Different tasks, different models | 2 min |
| 10 | Validate with representative tasks | 3 min |
| 11 | Key takeaways | 2 min |

> **Trainer note:** The timings in the deck's notes add up to about 36 minutes. The plan above trims slides 3, 4, 5, 6, 9 and the two scenarios slightly to fit 30. Protect the two scenario slides (7 and 8): they are where learners practise the framework.

---

## SLIDE 1: Title (1 min)

**SAY**
1. Welcome to Section 5: Model Selection for Agentic AI Systems.
2. We have 30 minutes.
3. This is the section where everything comes together.
4. In Section 1 we learned what the model does inside an agent, and the dimensions we compare.
5. In Section 2 we saw open-weight and proprietary models.
6. In Section 3 we looked at families, sizes and capabilities.
7. In Section 4 we saw how quantisation changes what fits on your hardware.
8. (pause)
9. Now we turn all of that into a **decision**.
10. By the end, you should be able to walk from a task description to a tested, defensible model choice.

---

## SLIDE 2: What We Will Cover (1 min)

**SAY**
1. Four parts.
2. First, a selection framework. From task requirements to a tested decision.
3. **DO** Point to the second box.
4. Second, requirements and constraints. What the task needs, and what the system allows.
5. **DO** Point to the third box.
6. Third, trade-offs. Capability, cost, latency, size and ownership.
7. **DO** Point to the fourth box.
8. Fourth, scenarios and validation. Two worked examples, mixed-model agents, and testing.
9. (pause)
10. The scenarios are where it becomes real. So please think of your own use case as we go.

---

## SLIDE 3: A Practical Selection Framework (3 min)

**SAY**
1. Here is the framework. Five steps.
2. **DO** Point to step 1.
3. Step one: **Define**. Define the task and the success criteria.
4. What does "good" look like for this agent? Be specific.
5. **DO** Point to step 2.
6. Step two: **Constrain**. Latency, cost, governance and hardware.
7. These are the limits of the system.
8. **DO** Point to step 3.
9. Step three: **Shortlist**. Two to four candidate models.
10. Not twenty. Two to four.
11. **DO** Point to step 4.
12. Step four: **Test**. Test on representative tasks.
13. **DO** Point to step 5.
14. Step five: **Decide** and monitor in production.
15. (pause)
16. Now look at the line at the bottom.
17. "Iterate: test results may change the shortlist or the constraints."
18. This is not a straight line.
19. Maybe testing shows that your cheap model cannot call tools reliably. Then you go back to the shortlist.
20. Or maybe testing shows that the latency budget was unrealistic. Then you go back to the constraints.
21. That is normal. It is how the process is supposed to work.

**Memory hook**
22. Define. Constrain. Shortlist. Test. Decide.

---

## SLIDE 4: Task Requirements (3 min)

**SAY**
1. Step one was to define the task. Here are six questions to help.
2. **DO** Point to "Task complexity".
3. Task complexity.
4. Is the task routine and well defined? Or open-ended and multi-step?
5. **DO** Point to "Reasoning".
6. Reasoning.
7. Does it need planning? Or is a direct answer enough?
8. **DO** Point to "Tool calling".
9. Tool calling.
10. How many tools are there? How complex are the arguments? How costly are mistakes?
11. **DO** Point to "Structured output".
12. Structured output.
13. Is a strict schema required by downstream components?
14. **DO** Point to "Context window".
15. Context window.
16. What are the typical **and worst-case** input sizes?
17. Include the history and any retrieved documents.
18. **DO** Point to "Multimodality".
19. Multimodality.
20. Are there images, documents or audio, as input or output?
21. (pause)
22. You will recognise these. They are the capabilities from Section 3.

**ASK: Quick rating exercise (about 1 minute)**
23. Take [your example agent].
24. Rate it quickly on each card: low, medium or high need.
25. **DO** Take answers for each of the six cards. Write them in a simple grid on a whiteboard or in chat.
26. **SAY** Now you have a requirements profile. Keep it. We will use it.

---

## SLIDE 5: System Constraints (3 min)

**SAY**
1. Requirements say what the task needs.
2. Constraints say what the **system allows**.
3. Four of them.
4. **DO** Point to "Latency".
5. Latency.
6. Is it an interactive response, or a background job?
7. And remember: agent loops multiply the per-call delay.
8. **DO** Point to "Cost".
9. Cost.
10. Per-token or hardware cost, multiplied by calls per task, multiplied by traffic volume.
11. **DO** Point to "Data governance".
12. Data governance.
13. Can data leave your environment? Think about regulation, residency and contractual limits.
14. **DO** Point to "Hardware & deployment".
15. Hardware and deployment.
16. What GPUs and memory are available? What quantisation options? And does your team have the ability to operate models?
17. (pause)
18. Here is the important point.
19. **Constraints often eliminate options before capability is even compared.**
20. If data cannot leave the building, hosted APIs are out. Full stop.
21. No benchmark changes that.

**Quick illustration of the latency multiplier**
22. Suppose each model call takes one second, and the agent makes four calls.
23. That is four seconds, not one.
24. If your target is three seconds, you already have a problem.
25. So latency has to be judged across the whole loop. Not per call.

---

## SLIDE 6: Key Trade-offs (4 min)

**SAY**
1. Now the trade-offs. This table summarises the whole course so far.
2. **DO** Point to the Capability vs. cost row.
3. Capability versus cost.
4. You gain better quality on hard tasks.
5. You give up: higher spend per call.
6. **DO** Point to the Capability vs. latency row.
7. Capability versus latency.
8. You gain deeper reasoning and bigger models.
9. You give up: slower responses and longer loops.
10. **DO** Point to the Model size vs. infrastructure row.
11. Model size versus infrastructure.
12. You gain stronger capacity.
13. You give up: more memory, more GPUs, more operations effort.
14. **DO** Point to the Open-weight row.
15. Open-weight.
16. You gain control, privacy and customisation.
17. You give up: you run, secure and scale it yourself.
18. **DO** Point to the Proprietary / API row.
19. Proprietary or API.
20. You gain a fast start and managed scaling.
21. You give up: vendor dependency, and data-sharing terms.
22. (pause)
23. Notice something.
24. **No option wins on every row.**
25. This is what we said at the very start, in Section 1.
26. No single model wins on every dimension. Selection is about trade-offs.

**ASK**
27. Which row matters most for **your** use case?
28. **DO** Take three or four answers. Connect each to a row.

---

## SLIDE 7: Scenario 1, Customer-Support Agent (4 min)

> **Trainer note:** Do not read the right-hand panel immediately. Read the requirements first, and ask learners what they would choose. Then reveal the reasoning.

**SAY (requirements, 1 minute)**
1. Let's apply the framework. Scenario one.
2. **DO** Point to the left panel.
3. A customer-support agent.
4. It looks up orders and issues refunds, using tools.
5. It must return JSON.
6. Replies are needed in about three seconds.
7. Volume is high every day.
8. It is cost-sensitive.
9. And the data is not highly regulated.

**ASK**
10. Before I show the reasoning: which class of model would you try first, and why?
11. **DO** Take two answers. Do not comment yet.

**SAY (reasoning, 2 minutes)**
12. **DO** Point to the right panel.
13. Here is the reasoning.
14. Most support intents are **routine**. So a small or mid-tier hosted model is a good fit.
15. But it needs reliable tool calling.
16. Unclear or high-risk cases, such as a large refund, should **escalate** to a larger model.
17. And what do we test? Tool-call accuracy, and schema adherence.
18. (pause)
19. Let's check it against the constraints.
20. Latency of about three seconds, with high volume and cost sensitivity, points to a small or mid-tier model.
21. Data is not highly regulated, so a hosted API is allowed.
22. JSON output and tools mean reliable tool calling is a must-have.
23. So: a mixed design. Small or mid-tier model for routine work. Larger model for escalation.

**ASK**
24. What would you test **first**?
25. **DO** Take one or two answers. Good answers: JSON validity on messy customer messages, correct refund-tool arguments, behaviour when the order lookup fails.

---

## SLIDE 8: Scenario 2, On-Premises Contract Review (4 min)

**SAY (requirements, 1 minute)**
1. Scenario two. Very different.
2. **DO** Point to the left panel.
3. An on-premises contract review agent.
4. Documents **cannot leave the company network**.
5. The contracts are long.
6. The task is extraction plus summary.
7. Results are needed within minutes, not seconds.
8. And there is one on-prem GPU server.

**ASK**
9. Which constraint decides this one?
10. **DO** Wait. Answer: data governance. The documents cannot leave the network.

**SAY (reasoning, 2 minutes)**
11. **DO** Point to the right panel.
12. Here is the reasoning.
13. Data cannot leave. So **open-weight, self-hosted**.
14. Use quantisation to fit a capable model in GPU memory.
15. And test long-context accuracy, and extraction quality, on real contracts.
16. (pause)
17. Now link it to Section 4.
18. Suppose the server has a single GPU with 80 gigabytes of memory.
19. A 70 billion parameter model at 16-bit needs about 140 gigabytes. It does not fit.
20. At INT8, about 70 gigabytes. It barely fits, with almost no room for the KV cache.
21. At INT4, about 35 gigabytes. It fits comfortably, and leaves room for long contracts.
22. And long contracts mean a **large KV cache**. So you must leave headroom.
23. (pause)
24. Notice what drives the decision here. It is not latency. It is governance and context length.

> **Trainer note:** The 80 GB GPU is an illustration. Adjust the numbers to match your learners' hardware, using the memory table from Section 4. The estimates are approximate and exclude overhead.

**SAY (contrast, 30 seconds)**
25. Compare it with scenario one.
26. Scenario one: latency and cost drove the choice.
27. Scenario two: governance and context length drove the choice.
28. Same framework. Different winners.

---

## SLIDE 9: Different Tasks, Different Models (2 min)

**SAY**
1. One more idea. You do not have to pick one model for the whole agent.
2. **DO** Point to the centre box.
3. In the middle is the agent orchestrator, or router.
4. **DO** Point to the small model.
5. A small model handles simple steps: classify, extract and route.
6. **DO** Point to the large model.
7. A large model handles the hard steps: plan, reason and decide.
8. **DO** Point to the multimodal model.
9. A multimodal model reads images and PDFs.
10. (pause)
11. Read the rule at the bottom with me.
12. "Each step uses the cheapest model that meets its requirements."
13. Routing keeps cost and latency down.
14. And it reserves the strong models for the hard steps.
15. Remember scenario one. Small or mid-tier for routine tickets. Larger for escalations. That is routing.
16. We build the orchestrator itself in the next sections.

---

## SLIDE 10: Validate with Representative Tasks (3 min)

**SAY**
1. We have a shortlist. Now, step four: test.
2. **DO** Point to bullet 1.
3. Build a small test set from **real** tasks.
4. Include edge cases, and include failures.
5. **DO** Point to bullet 2.
6. Measure four things.
7. Task success.
8. Tool-call and schema validity.
9. Latency.
10. And cost per task.
11. **DO** Point to bullet 3.
12. Run the **full agent loop**, not isolated prompts.
13. Why? Because errors compound across steps.
14. A model that is 95 percent right at each of ten steps is only about 60 percent right overall.
15. **DO** Point to bullet 4.
16. And re-test when anything changes.
17. A new model version. A new prompt. A new tool. Or new quantisation settings.
18. (pause)
19. Remember what we said in Section 2.
20. Benchmarks show general ability. They do not show fit for your prompts, your tools and your data.

> **Trainer note on the 95 percent figure:** 0.95 to the power of 10 is about 0.60. It is an illustration of compounding errors, not a measured rate.

**ASK**
21. What would you put in a test set for your own agent?
22. **DO** Take two answers. Remind them to include at least one failure case.

---

## SLIDE 11: Key Takeaways (2 min)

**SAY**
1. Let's wrap up.
2. **DO** Point to takeaway 1.
3. One: start from task requirements and system constraints. Then shortlist and test.
4. **DO** Point to takeaway 2.
5. Two: every choice trades capability against cost, latency, infrastructure and control.
6. **DO** Point to takeaway 3.
7. Three: open-weight means more ownership. API models mean vendor dependency.
8. **DO** Point to takeaway 4.
9. Four: one agent can use several models for different steps.
10. **DO** Point to takeaway 5.
11. Five: next is agentic AI system design and implementation.
12. We now know **which model** to use.
13. Next we look at **how to build the agent around it**.
14. (pause)
15. Any questions before we move on?

---

## Optional Handout: One-Page Selection Worksheet

Share this with learners to use on their own agent.

| Step | Question | Your answer |
|---|---|---|
| 1. Define | What is the task? What does success look like? | |
| | Task complexity (routine / multi-step) | |
| | Reasoning need (low / medium / high) | |
| | Tool calling (how many tools, how costly are mistakes) | |
| | Structured output (strict schema?) | |
| | Context (typical and worst-case size) | |
| | Multimodality (images, documents, audio?) | |
| 2. Constrain | Latency target (interactive / background) | |
| | Budget (per task, per month) | |
| | Data governance (can data leave?) | |
| | Hardware and team (GPUs, ability to operate) | |
| 3. Shortlist | 2 to 4 candidate models | |
| 4. Test | Test set (real tasks, edge cases, failures) | |
| | Measures (success, schema validity, latency, cost) | |
| 5. Decide | Chosen model(s) and what to monitor | |

---

## Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| What is the first step in selecting a model? | Define the task and success criteria. Start from requirements, not model names. |
| Why do constraints come before comparing capability? | They often eliminate options outright, for example when data cannot leave your environment. |
| How many models should I shortlist? | Two to four. |
| Is selection a one-time decision? | No. It is iterative, and you re-test when models, prompts, tools or settings change. |
| Why test the full agent loop? | Errors compound across steps, and isolated prompts hide that. |
| When should I choose open-weight? | When data governance, control or customisation come first, and you can operate it. |
| When should I choose an API model? | When you want a fast start and managed scaling, and data-sharing terms are acceptable. |
| Can one agent use several models? | Yes. Route each step to the cheapest model that meets its requirements. |
| How do I handle latency in agent loops? | Judge it across the whole loop, not per call. Each call adds delay. |
| Should I trust benchmarks? | Use them to shortlist. Decide using your own tasks. |
| What if the best model is too costly? | Consider a smaller model, routing, or quantisation, and test whether quality holds on your tasks. |

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 1 min |
| 2. Agenda | 1 min |
| 3. Framework | 3 min |
| 4. Task requirements | 3 min |
| 5. System constraints | 3 min |
| 6. Trade-offs | 4 min |
| 7. Scenario 1 | 4 min |
| 8. Scenario 2 | 4 min |
| 9. Mixed-model agents | 2 min |
| 10. Validation | 3 min |
| 11. Takeaways | 2 min |

If you run over, shorten the rating exercise on slide 4 and the discussion on slide 6. **Protect slides 7 and 8.**
