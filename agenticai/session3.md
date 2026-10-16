# Section 3 Trainer Script: Model Families, Sizes & Capabilities

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 40 minutes

> **Before you start:** Names change, but the patterns stay. Use any current model families as live examples. Where this script cites models, they come from Section 2 so learners already know them.

---

## Timing Plan

| Slide | Topic | Time |
|---|---|---|
| 1 | Title and set-up | 1 min |
| 2 | What we will cover | 2 min |
| 3 | Model families and variants | 5 min |
| 4 | Parameter count: one measure of size | 4 min |
| 5 | Why parameter count alone is not enough | 4 min |
| 6 | Total vs. active parameters (MoE) | 5 min |
| 7 | Smaller vs. larger models | 5 min |
| 8 | Capabilities that matter for agents | 2 min |
| 9 | Reasoning, tool calling and structured output | 4 min |
| 10 | Context, coding and multimodality | 4 min |
| 11 | Select on task and system needs | 2 min |
| 12 | Key takeaways | 2 min |

> **Trainer note:** The timings in the deck's notes add up to about 43 minutes. The plan above trims slides 4 and 5 by one minute each and slide 1 to one minute, so everything fits in 40. Protect slides 6 and 9: they are the hardest ideas in the section.

---

## SLIDE 1: Title (1 min)

**SAY**
1. Welcome to Section 3: Model Families, Sizes and Capabilities.
2. We have 40 minutes.
3. In Section 2 we looked at **who** offers models.
4. Now we move to **how to read** a model family, its size, and its capabilities.
5. By the end, you should be able to look at a model name or a spec sheet and know what it tells you, and what it does not.

**ASK**
6. Quick one. When you see a name like "7B" or "120B" on a model, what do you think that number means?
7. **DO** Take two or three answers. Do not correct yet. Say "Hold that thought. We will come back to it on slide 4."

---

## SLIDE 2: What We Will Cover (2 min)

**SAY**
1. Here are four parts.
2. First, families and variants. What a family is, and what each variant is for.
3. **DO** Point to the second box.
4. Second, size. Parameter count, and total versus active parameters in mixture-of-experts models.
5. **DO** Point to the third box.
6. Third, small versus large. When each one is the right choice.
7. **DO** Point to the fourth box.
8. Fourth, agentic capabilities. Reasoning, tools, structured output, context, coding and multimodal.
9. (pause)
10. Notice how this ties back to Section 1.
11. There we saw the six jobs a model does inside an agent.
12. Today we look at which model properties make those jobs succeed or fail.

---

## SLIDE 3: Model Families and Variants (5 min)

**SAY**
1. Let's start with the idea of a **family**.
2. A family is a group of models that share an architecture and a training lineage.
3. Variants specialise the family for different jobs.
4. There are six kinds of variant to know.
5. **DO** Point to "Base vs instruct".
6. One: base versus instruct.
7. Base models predict text. They continue what you give them.
8. Instruct, or chat, models follow instructions. They are the usual choice for agents.
9. **DO** Point to "Reasoning variants".
10. Two: reasoning variants.
11. These spend extra tokens "thinking" before they answer.
12. They are better on planning and hard tasks. But they are slower and cost more.
13. **DO** Point to "Coding variants".
14. Three: coding variants.
15. Tuned for code generation, editing, and tool-driven developer workflows.
16. **DO** Point to "Multimodal variants".
17. Four: multimodal variants.
18. They accept images, audio or documents alongside text.
19. **DO** Point to "Small / distilled".
20. Five: small or distilled models.
21. Compact versions, built for speed, low cost and local use.
22. **DO** Point to "Size tiers".
23. Six: size tiers.
24. Most families ship small, medium and large models. They trade capability for cost and latency.
25. (pause)
26. Remember what we said in Section 2.
27. Every vendor has a top tier, a balanced tier and a fast tier. That is the same pattern.
28. Names change. The pattern stays.

**ASK**
29. Pick a family you use. Which variants does it offer?
30. **DO** Take one or two answers. Map each to the six kinds on the slide.

---

## SLIDE 4: Parameter Count, One Measure of Size (4 min)

**SAY**
1. Now back to the question from the start. What does "7B" mean?
2. Parameters are the **learned weights** inside the model.
3. Sizes are quoted in billions. So 7B means seven billion parameters.
4. (pause)
5. Here is the rule of thumb.
6. More parameters give more capacity. More room to store knowledge, and to handle complex tasks.
7. But there is a cost.
8. More parameters also mean more memory to hold the weights.
9. More compute for every token.
10. And a higher serving cost.
11. **DO** Point to the memory formula.
12. Now a calculation you can do in your head.
13. Memory is roughly parameters times bytes per parameter.
14. A 7 billion parameter model at 16-bit precision uses 2 bytes per parameter.
15. So 7 billion times 2 bytes is about 14 gigabytes.
16. (pause)
17. And that is **before** the KV cache and other overhead.
18. So real memory use is higher.
19. **DO** Point to the last line.
20. Last point, and it is important.
21. Treat parameter count as a rough guide. It is **not** a ranking of quality.

> **Trainer extras, only if asked**
> - At 8-bit, the same 7B model is about 7 GB. At 4-bit, about 3.5 GB. Section 4 covers this under quantisation.
> - KV cache is the memory the model uses to remember the tokens it has already processed. It grows with context length.

---

## SLIDE 5: Why Parameter Count Alone Is Not Enough (4 min)

**SAY**
1. So why is parameter count not enough? Four reasons.
2. **DO** Point to "Capability".
3. Capability. Training data quality, post-training and architecture matter as much as size.
4. **DO** Point to "Memory".
5. Memory. Precision, KV cache and context length all add to the weight footprint.
6. **DO** Point to "Speed".
7. Speed. Active parameters, hardware, batching and runtime drive how many tokens per second you get.
8. **DO** Point to "Cost".
9. Cost. Pricing and hardware needs depend on the provider, the deployment and your usage pattern.
10. (pause)
11. Let me give you two counter-examples.
12. First: a newer small model can outperform an older large one. Better data and training can beat raw size.
13. Second: two models of the **same size** can run at very different speeds. The runtime, the hardware and the architecture make the difference.
14. So size alone cannot tell you capability, memory, speed or cost.

**ASK**
15. Have you seen a smaller model beat a bigger one on your task?
16. **DO** Take one or two stories. If quiet, offer the counter-examples above again.

---

## SLIDE 6: Total vs. Active Parameters, Mixture-of-Experts (5 min)

> **Trainer note:** This is one of the hardest ideas in the section. Go slowly. Use the two boxes on the slide and one worked example.

**SAY**
1. This slide is about a distinction that confuses many people. Let's take it slowly.
2. **DO** Point to "Dense model".
3. First, a dense model.
4. Every parameter is used for every token.
5. So total parameters equals active parameters.
6. **DO** Point to "Mixture-of-Experts".
7. Now, mixture-of-experts. We call it MoE.
8. Inside the model there are many "experts".
9. A **router** picks a few experts for each token.
10. So only a fraction of the model does work per token.
11. That means active parameters are much smaller than total parameters.
12. (pause)
13. Here is the key sentence. Read it with me.
14. **DO** Point to the implication line.
15. Compute and speed follow **active** parameters.
16. Memory follows **total** parameters.
17. Why? Because all the weights must be stored. But only a fraction is computed per token.

**SAY: Worked example using models from Section 2**
18. Remember Mistral Small 4 from Section 2?
19. 119 billion total parameters. 6.5 billion active.
20. So to **store** it, you need memory for 119 billion parameters.
21. But to **run** each token, the compute is closer to a 6.5 billion model.
22. Another one: DeepSeek V4 Pro.
23. 1.6 trillion total. 49 billion active.
24. Huge memory needs. But far less compute per token than the total suggests.
25. (pause)
26. So when you read a spec, always ask two questions.
27. How many parameters in **total**? That tells me about memory.
28. How many **active**? That tells me about speed and compute.

**ASK**
29. A model has 100 billion total and 10 billion active parameters. Roughly which size does it feel like for speed, and which size for memory?
30. **DO** Wait. Answer: speed feels like about 10B, memory needs are those of 100B.

---

## SLIDE 7: Smaller vs. Larger Models (5 min)

**SAY**
1. Now the practical question. When do you pick small, and when do you pick large?
2. **DO** Point to the left column.
3. Smaller models fit when tasks are narrow or routine.
4. For example classification, extraction and routing.
5. They fit when latency and cost matter.
6. When data must stay local.
7. And when hardware is limited.
8. **DO** Point to the right column.
9. Larger models fit when tasks need multi-step reasoning.
10. When instructions are ambiguous.
11. When you need long context.
12. Or high tool-call reliability.
13. And when errors are costly.
14. (pause)
15. **DO** Point to the bottom line.
16. Here is the best part. You do not have to choose just one.
17. Many agents use **both**. A small model for the simple steps. A larger one for the hard decisions.
18. We will build on this idea in Section 5, with routing and mixed-model agents.
19. (pause)
20. A concrete example.
21. Think of a support agent.
22. Step one: classify the ticket. That is routine. A small model is fine.
23. Step two: decide how to resolve a tricky, ambiguous complaint. That needs planning. Use a larger model.
24. Same agent. Two different models.

**ASK**
25. Think of an agent step you have built, or would build. Is it a small-model step or a large-model step? Why?
26. **DO** Take two answers. Ask each person which constraint decided it: cost, latency, accuracy, or data location.

---

## SLIDE 8: Capabilities That Matter for Agents (2 min)

**SAY**
1. Now we move to capabilities. Six of them matter most for agents.
2. This is an overview slide. The next two slides go deeper.
3. **DO** Point to each item as you name it.
4. Reasoning and planning: breaking goals into steps and adapting.
5. Tool or function calling: choosing tools and generating valid arguments.
6. Structured output: reliable JSON or schema-conforming responses.
7. Context window: capacity, and long-context performance.
8. Coding: writing, reading and fixing code.
9. Multimodal: understanding images, documents and audio.
10. (pause)
11. Connect this back to Section 1.
12. Remember the six jobs: understand, plan, select tools, generate arguments, interpret outputs, respond.
13. These capabilities are what make those jobs work.

---

## SLIDE 9: Reasoning, Tool Calling and Structured Output (4 min)

> **Trainer note:** These three decide whether an agent loop works reliably. If you have a quick demo or a failure example ready, use it here.

**SAY**
1. These three capabilities decide whether an agent loop works at all. Let's go through each one.
2. **DO** Point to "Reasoning & planning".
3. Reasoning and planning.
4. The model decomposes tasks, tracks state, and recovers from errors.
5. Reasoning modes help. But they add latency and token cost.
6. **DO** Point to "Tool / function calling".
7. Tool calling.
8. The model picks the right tool and produces valid arguments.
9. It also has to **read the tool results correctly**.
10. And reliability varies a lot by model.
11. **DO** Point to "Structured output".
12. Structured output.
13. It is needed to pass data between components.
14. And please check schema adherence under real prompts. Not just once.
15. (pause)
16. A failure example to make it real.
17. The agent asks the model for a JSON object with three fields.
18. The model returns the JSON, but with a field missing, or the date in the wrong format.
19. The next tool call fails. The loop breaks.
20. One malformed output. One broken agent.
21. That is why we test these three under realistic conditions.

**ASK**
22. Which of the three has caused you the most pain? Reasoning, tool calling or structured output?
23. **DO** Take two answers.

---

## SLIDE 10: Context, Coding and Multimodality (4 min)

**SAY**
1. Now the other three capabilities.
2. **DO** Point to "Context window".
3. Context window.
4. Capacity is the **maximum** number of tokens the model can take in.
5. Long-context performance is **how well** the model uses them.
6. These are not the same thing.
7. And a larger context raises cost and latency.
8. **DO** Point to "Coding".
9. Coding.
10. This matters for code agents.
11. It also matters for agents that generate queries, scripts or tool code, even if they are not "coding agents".
12. **DO** Point to "Multimodal".
13. Multimodal.
14. It lets agents read screenshots, charts, PDFs or audio.
15. But support and quality differ widely between models.
16. (pause)
17. Let me stress the first point again.
18. A large context window is capacity, not proof of reliable use.
19. Models can lose information in the **middle** of long inputs.
20. So test long-context accuracy with your own documents.

---

## SLIDE 11: Select on Task and System Needs, Not Size (2 min)

**SAY**
1. Let's pull it together. How do you actually select?
2. **DO** Point to line 1.
3. Start from the **task**: its complexity, tool use, output format, context length and modality.
4. **DO** Point to line 2.
5. Then add the **system** constraints: latency, budget, data governance and hardware.
6. **DO** Point to line 3.
7. Then test the shortlisted models on representative tasks. Benchmarks are only a starting point.
8. **DO** Point to line 4.
9. And remember: the largest model is not always the best fit. And the smallest is not always good enough.
10. (pause)
11. One concrete example.
12. An extraction step: pull invoice numbers out of emails. A small model can handle it.
13. A planning step: decide how to resolve a multi-part customer dispute. That needs a larger model.
14. Match the model to the step.

---

## SLIDE 12: Key Takeaways (2 min)

**SAY**
1. Let's wrap up with five points.
2. **DO** Point to takeaway 1.
3. One: families offer variants for different jobs. Instruct, reasoning, coding, multimodal and small.
4. **DO** Point to takeaway 2.
5. Two: parameter count is one rough measure. It does not fix capability, speed, memory or cost.
6. **DO** Point to takeaway 3.
7. Three: MoE models separate total parameters, which drive memory, from active parameters, which drive compute.
8. **DO** Point to takeaway 4.
9. Four: choose by agentic capabilities and by constraints.
10. **DO** Point to takeaway 5.
11. Five: next is quantisation and efficient deployment.
12. In Section 4, we see how quantisation reduces memory, so larger models can run on modest hardware.
13. (pause)
14. Any questions before we move on?

---

## Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| What does "7B" mean? | Seven billion parameters, the learned weights of the model. |
| How much memory does a 7B model need? | About 14 GB at 16-bit (7B × 2 bytes), before KV cache and overhead. |
| Does more parameters mean a better model? | Not necessarily. Data quality, post-training and architecture matter as much as size. |
| What is a base model vs. an instruct model? | Base predicts text. Instruct follows instructions and is the usual agent choice. |
| What is a reasoning variant? | A model that spends extra tokens thinking before answering. Better on hard tasks, slower and costlier. |
| What is MoE? | Mixture-of-Experts. A router activates a few experts per token, so active parameters are far fewer than total. |
| Total vs. active parameters: which matters for what? | Total drives memory. Active drives compute and speed. |
| What is the KV cache? | Memory used to hold the tokens already processed. It grows with context length. |
| Is a big context window enough? | No. Capacity is not the same as accurate long-context use. Test with your own documents. |
| Should I always use the largest model? | No. Match the model to the task and constraints. Many agents mix small and large models. |
| What is distillation? | Training a smaller model to imitate a larger one, giving a compact model with much of its capability. |

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 1 min |
| 2. Agenda | 2 min |
| 3. Families and variants | 5 min |
| 4. Parameter count | 4 min |
| 5. Why size is not enough | 4 min |
| 6. Total vs. active (MoE) | 5 min |
| 7. Smaller vs. larger | 5 min |
| 8. Capabilities overview | 2 min |
| 9. Reasoning, tools, structured output | 4 min |
| 10. Context, coding, multimodal | 4 min |
| 11. Selection | 2 min |
| 12. Takeaways | 2 min |

If you run over, shorten discussion on slides 3 and 5, and keep slide 8 brisk. **Protect slides 6 and 9.**
