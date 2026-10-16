# Day 1 Trainer Script: Foundation Models in Agentic AI

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 20 minutes

---

## SLIDE 1: Title (1 min)

**SAY**
1. Welcome, everyone.
2. This is Section 1 of 6 in our session on the Model Landscape for Agentic AI.
3. We have about 20 minutes together.
4. My goal is simple.
5. By the end, we should all share the same picture of how an agent uses a foundation model.
6. We will also pick up the set of dimensions we use to compare models for the rest of the session.

**ASK**
7. Before we start, a quick show of hands.
8. Who has already built or used an agent with tool calling?

**DO** Note how many hands go up. Many hands: move faster on slides 4 and 5. Few hands: slow down there.

**SAY**
9. Great, thank you. That helps me pitch this right.

---

## SLIDE 2: What We'll Cover (1 min)

**SAY**
1. Here is how we will use the 20 minutes.
2. Four short parts.
3. First, a quick recap of foundation models. About four minutes.
4. Second, how an agent actually uses a model. Six jobs the model does. About six minutes.
5. Third, what the model decides, and what the rest of the system decides. About four minutes.
6. Fourth, the dimensions we use to choose a model. About six minutes.
7. (pause)
8. Look at the line at the bottom.
9. By the end, you should be able to explain how an agent uses a model, and name the dimensions that drive model choice.
10. The last part matters most, because every later section builds on it.

---

## SLIDE 3: Recap, Foundation Models (2 to 3 min)

**SAY**
1. Let's start with a quick recap. You already know the basics, so I will keep this short.
2. A foundation model is pretrained at scale.
3. It is trained on broad data, so it learns general language, knowledge and reasoning patterns.
4. (pause)
5. Second idea: it is adapted, not rebuilt.
6. We do not train a new model for every task.
7. We take one model and prompt it, fine-tune it, or give it tools.
8. **DO** Point to the circle in the middle.
9. That is why one model can do chat and Q&A, summarising, image understanding, data extraction, coding, and agents.
10. The same model. Many uses.
11. (pause)
12. Third idea: from answers to actions.
13. With tool calling and an orchestration loop, a model can drive multi-step work.
14. That is the basis of agentic systems.
15. **DO** Point to "Agents".
16. Agents are just one use among many. But it is the one we care about today.

---

## SLIDE 4: The Model Sits Inside the Agent Loop (3 min)

**SAY**
1. This is the core picture of the whole section. Let me walk through it slowly.
2. **DO** Point to "User request".
3. It starts here. The user asks for something. The request and its context go to the model.
4. **DO** Point to "Foundation model".
5. The model understands the request, reasons about it, and decides what to do.
6. It might decide to call a tool. If so, it writes the tool call and its arguments.
7. **DO** Point to "Tools and APIs".
8. The tool runs. This could be a database, a search, an email service.
9. The tool's output goes back to the model.
10. **DO** Trace the arrow back.
11. Now the model decides again.
12. Does it need another tool? Or is it ready to give the final answer?
13. **DO** Point to "Response to user".
14. When it is done, the final answer goes back to the user.
15. (pause)
16. See the line at the bottom: the loop repeats until the model decides the task is done.
17. Now, the most important part of this slide.
18. **DO** Point to the shaded outer box.
19. Around the model sits orchestration, memory, guardrails and application logic.
20. So the model is the decision-maker.
21. But the model is not the whole agent.
22. Please remember that. We will come back to it.

---

## SLIDE 5: Six Jobs the Model Does (3 min)

**SAY**
1. Let's make this concrete with one example.
2. Imagine the user says: "Find last quarter's refund total and email it to finance."
3. Watch how many jobs the model does for that one request.
4. **DO** Point to job 1.
5. Job one: understand instructions. The model parses the goal, the constraints and the context. Here, the goal is a refund total, for last quarter, sent to finance.
6. **DO** Point to job 2.
7. Job two: reason and plan. It breaks the task into steps. First get the number. Then write the email. Then send it.
8. **DO** Point to job 3.
9. Job three: select tools. It chooses the database tool for step one.
10. **DO** Point to job 4.
11. Job four: generate arguments. It writes a valid query with the right date range. This must be well-formed, or the tool fails.
12. **DO** Point to job 5.
13. Job five: interpret tool outputs. It reads the result. Is it empty? Is it an error? Should it retry, or continue?
14. **DO** Point to job 6.
15. Job six: generate the response. It writes the final message, in the right format and tone for finance.
16. (pause)
17. Here is the key point.
18. If the model fails at any one of these six, the agent breaks.
19. That is why capabilities like reasoning, tool calling and structured output matter so much when we compare models.

**ASK (optional)**
20. Which of these six do you think a weaker model fails at first?
21. **DO** Take one or two answers. Common answers: planning, arguments, noticing errors.

---

## SLIDE 6: Model Choice vs. System Design (4 min)

**SAY**
1. Now an important question.
2. When an agent fails, whose fault is it? The model's? Or the system around it?
3. This slide splits the two.
4. **DO** Point to the left panel.
5. On the left is what the model drives.
6. The quality of reasoning and planning.
7. Choosing the right tool for each step.
8. Producing valid tool-call arguments.
9. Handling ambiguity and recovering from errors.
10. And using long context accurately.
11. **DO** Point to the right panel.
12. On the right is what the rest of the system drives.
13. Which tools exist, and what they are allowed to access.
14. Loop control, retries and stopping rules.
15. Memory, retrieval and context management.
16. Validation, guardrails and human approval.
17. And latency and cost budgets.
18. (pause)
19. Here is the nuance.
20. Guardrails, validation and retries can make up for some model weaknesses.
21. But not all of them.
22. If a model cannot plan multi-step tasks, better prompts alone will not rescue it.
23. And the reverse is true too.
24. A strong model inside a badly designed loop still fails.
25. **DO** Point to the bottom line.
26. So: a better model raises the ceiling. System design decides how much of that ceiling you actually reach.
27. Model selection is one part of agent design. It is not the whole of it.

**ASK**
28. Let's take two minutes.
29. Think of an agent failure you have seen.
30. Was it a model failure, or a system failure?
31. **DO** Take two or three answers. If quiet, offer examples:
    - Model failure: wrong tool chosen, or malformed output again and again.
    - System failure: no retry limit so it looped forever, or a tool had too much access with no approval step.
32. **SAY** Thank you. Notice how often it is a mix of both.

---

## SLIDE 7: Dimensions for Choosing a Model (4 min) — KEY SLIDE

> **Trainer note:** The slide title says "ten dimensions" but the grid shows twelve items. Say "these dimensions" aloud to avoid confusion. If asked, say: "The groups are the easy way to remember them."

**SAY**
1. This is the most important slide in this section.
2. I will leave it up during discussion, because every later section refers back to it.
3. We group the dimensions into three questions.
4. **DO** Point to the first column, Capability.
5. First question: what can the model do?
6. Task capability: can it do the actual job well?
7. Reasoning: planning and multi-step problem solving.
8. Tool calling: does it pick the right tools and form valid calls?
9. Structured output: does it reliably produce JSON that matches a schema?
10. Multimodality: can it handle text, images, audio and documents?
11. **DO** Point to the second column, Performance and economics.
12. Second question: how does it perform, and what does it cost?
13. Size and architecture: parameters, and whether it is dense or mixture-of-experts.
14. Context window: how much it can hold and actually use.
15. Latency: time to first token, and total time.
16. Cost: per token, per task, or per GPU hour.
17. **DO** Point to the third column, Deployment and governance.
18. Third question: where and how can we run it?
19. Hosting model: through an API, or self-hosted.
20. Data governance: privacy, data residency and compliance.
21. Hardware: the GPUs and memory we have available.
22. (pause)
23. Two kinds of dimension here.
24. Some are hard constraints. For example, "our data may not leave the organisation." That removes options immediately.
25. Others are trade-offs. For example, cost versus capability. We balance those.
26. **DO** Point to the bottom line.
27. Remember this sentence: no single model wins on every dimension. Selection is about trade-offs.
28. In Section 5 we turn all of this into a selection framework.

**Quick definitions, only if asked**
- Dense vs. mixture-of-experts: dense uses all parameters for every token; mixture-of-experts activates only a part, which can reduce compute.
- Time to first token: how long until the user sees anything. Total time: how long until the answer is complete.
- Context window: how much text goes in, and, more importantly, how well the model uses it.

---

## SLIDE 8: Same Dimensions, Different Priorities (3 min)

**SAY**
1. Now let's use the dimensions.
2. Same dimensions. Three very different agents.
3. **DO** Point to the first scenario.
4. First, a support triage agent.
5. It routes thousands of tickets a day and fills in a ticket form.
6. Top priorities: low latency, low cost per request, and reliable structured output.
7. What can it trade off? Deep reasoning.
8. **DO** Point to the second scenario.
9. Second, a research agent.
10. It reads long reports, runs many searches, and writes a synthesis.
11. Top priorities: strong reasoning, long context, and multi-step tool calling.
12. What can it trade off? Response speed.
13. **DO** Point to the third scenario.
14. Third, a regulated document agent.
15. It processes sensitive contracts inside the organisation.
16. Top priorities: data governance, self-hosted deployment, and fitting the hardware we have.
17. What can it trade off? Top-end capability.
18. (pause)
19. Same ten-plus dimensions, three completely different shopping lists.

**ASK**
20. Quick question for the room.
21. Which kind of model would you reach for in each case, and why?
22. **DO** Take one or two answers. Do not settle the debate.
23. **SAY** Good thinking. We will return to scenarios like these in Section 5, with explicit constraints.
24. One more idea to plant.
25. A single system can use different models for different tasks.
26. For example, a small, fast model to route requests, and a larger one for the hard reasoning.

---

## SLIDE 9: Key Takeaways (1 min)

**SAY**
1. Let's wrap up.
2. Foundation models are the reasoning engine of an agent, but only one component of the system.
3. Four takeaways.
4. **DO** Point to takeaway 1.
5. One: the model is used at every step. Understanding, planning, tool choice, arguments, reading results, and responding.
6. **DO** Point to takeaway 2.
7. Two: model choice sets the ceiling. Tools, orchestration and guardrails decide how much of it you reach.
8. **DO** Point to takeaway 3.
9. Three: choose using the dimensions, against the task and the constraints. Not by size or benchmark rank alone.
10. **DO** Point to takeaway 4.
11. Four: next up is Section 2, open-weight and proprietary models.
12. Now that we know what the model does and which dimensions matter, Section 2 surveys the landscape.
13. We will look at open-weight versus proprietary models, and how representative model families compare on these dimensions.
14. (pause)
15. Any questions before we move on?

---

## Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| Is the agent the same as the model? | No. The model decides; the system around it runs tools, holds memory and enforces rules. |
| Can better prompts fix a weak model? | They help, but cannot create capabilities the model lacks, like multi-step planning. |
| Should we always pick the biggest model? | No. It may be too slow, too costly, or not allowed for sensitive data. |
| Can one system use several models? | Yes. Different steps can use different models. |
| Why does structured output matter? | Tools need valid parameters. Malformed output breaks the loop. |
| Will we cover specific models? | Yes, from Section 2. Section 5 gives a selection framework. |

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 1 min |
| 2. Agenda | 1 min |
| 3. Recap | 2 to 3 min |
| 4. Agent loop | 3 min |
| 5. Six jobs | 3 min |
| 6. Model vs. system | 4 min |
| 7. Dimensions | 4 min |
| 8. Priorities | 3 min |
| 9. Takeaways | 1 min |

If you run over, shorten slide 3 and the discussion on slide 6. Never shorten slide 7.
