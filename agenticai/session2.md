# Section 2 Trainer Script: Industry Model Landscape (Open-weight and Proprietary Models)

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 45 minutes
**Reference date on the deck:** 14 October 2026

> **Before you start:** The landscape moves monthly. Verify model version names, licences and prices on the day. Stress **classes** of model over **names**. Prepare the live demo (slide 7) in advance: the same task set, three models, a blank scorecard.

---

## Timing Plan

| Slide | Topic | Time |
|---|---|---|
| 1 | Title and roadmap | 1 min |
| 2 | Open-weight vs. proprietary | 9 min |
| 3 | Open-weight is not open source | 7 min |
| 4 | Proprietary families and tiers | 3 min |
| 5 | Open-weight families to know | 3 min |
| 6 | Positioning by class of model | 4 min |
| 7 | Comparing models in practice (live demo) | 12 min |
| 8 | Why benchmarks alone do not decide | 5 min |
| 9 | Key takeaways | 1 min |

> **Trainer note:** The notes in the deck give slightly more than 45 minutes if every slide runs to its own suggested time (slides 4, 5 and 6 together are meant to fit inside the 10-minute "families" block). The plan above fits everything into 45. Protect slide 7, the live demo.

---

## SLIDE 1: Title (1 min)

**SAY**
1. Welcome back. This is Section 2 of 6: the Industry Model Landscape.
2. We have 45 minutes.
3. Today we look at open-weight and proprietary models.
4. Here is the plan.
5. First, how we access models. Then, open-weight versus open-source, and licences.
6. Then the main model families.
7. Then a live demo where we compare models in practice.
8. And finally, why benchmarks alone are not enough.
9. (pause)
10. One warning before we begin.
11. This landscape changes every month.
12. So I will focus on **classes** of model, not on specific names.
13. The names on the slides are accurate as of 14 October 2026. Please check them again before you use them.

**ASK**
14. Quick question. Which models are you using or testing today?
15. **DO** Take two or three answers. Note whether they are mostly API or open-weight. It tells you where to spend time.

---

## SLIDE 2: Open-weight vs. Proprietary Models (9 min)

**SAY**
1. We start with the most basic question: how do you get access to a model?
2. There are two main routes.
3. **DO** Point to the left column.
4. Open-weight models: you can download the weights, under a licence.
5. **DO** Point to the right column.
6. Proprietary, or API-accessed models: you use them through a hosted API or an app. The weights stay private.
7. Let's go through the table row by row.
8. **DO** Point to the Access row.
9. Access. Open-weight: weights are downloadable under a licence. Proprietary: hosted API or app, and the weights stay private.
10. **DO** Point to the Deployment row.
11. Deployment. Open-weight runs on your servers, on cloud GPUs, or even on a local machine. Proprietary runs in the provider's cloud. Some are also available through the major cloud platforms.
12. **DO** Point to the Customisation row.
13. Customisation. With open-weight you can fine-tune, quantise, and control the serving stack. With proprietary you work mainly through prompting and tools. Fine-tuning exists only where the provider offers it.
14. **DO** Point to the Data governance row.
15. Data governance. With open-weight, data can stay inside your own boundary. With proprietary, data goes to the provider, and it is covered by contract.
16. **DO** Point to the Cost row.
17. Cost. Open-weight costs you GPUs, operations and engineering time. Proprietary is pay-per-token, with no infrastructure to run.
18. **DO** Point to the Vendor dependency row.
19. Vendor dependency. Open-weight means less model lock-in, but more operational duty. Proprietary means you are exposed to price changes, version changes and deprecations.
20. (pause)
21. Now the key message.
22. Neither is better in general.
23. Open-weight moves control to you. But it also moves the operating burden to you.
24. APIs give you instant access to the strongest models. But they create dependency on the vendor's pricing and release cycle.
25. And one more point.
26. You do not have to choose only one.
27. A hybrid design is common: one agent system calls an API model for hard reasoning, and a self-hosted model for sensitive or high-volume steps.

**ASK**
28. Think about your own organisation.
29. Which row of this table would worry you most? Data governance? Cost? Lock-in?
30. **DO** Take two or three answers. Link each back to the row on the slide.

---

## SLIDE 3: Open-weight Is Not Open Source (7 min)

**SAY**
1. This slide clears up one of the most common mix-ups.
2. People say "open" for everything. But there are four different levels.
3. **DO** Point to level 1.
4. Level one: proprietary or API-only. No weights are released. You access it through an API or an app. Examples: Claude, GPT and Gemini.
5. **DO** Point to level 2.
6. Level two: open-weight with a custom licence. The weights are downloadable, but with use terms and limits. Examples: Llama 4, and some Qwen models.
7. **DO** Point to level 3.
8. Level three: open-weight with a permissive licence. Apache 2.0 or MIT. Few restrictions on use. Examples: gpt-oss and DeepSeek V4.
9. **DO** Point to level 4.
10. Level four: fully open source. You get the weights, plus the training code, the data and the recipe. Examples are OLMo-style releases.
11. **DO** Point to the arrow at the bottom.
12. As you move from one to four, you get more access to the weights, more licence freedom, and more transparency.
13. (pause)
14. Now read the sentence at the bottom with me.
15. "Open-weight means you can get the weights. It does not mean you can use them freely."
16. Attribution, user-count thresholds and usage policies can all apply.

**SAY (licence examples, as of October 2026)**
17. A few examples, so you see how much it varies.
18. gpt-oss is Apache 2.0, with an added usage policy.
19. DeepSeek V4 is MIT.
20. Llama 4 uses the Llama Community licence. It requires attribution, and a separate licence above 700 million monthly users.
21. Qwen licences vary by model. Some are Apache 2.0. Some are community or model-specific.
22. Mistral varies too. Small 4 and Large 3 are Apache 2.0. Medium 3.5 has a custom licence.
23. (pause)
24. The lesson is simple.
25. Licences differ per **model**, not per **family**.
26. So read the licence file every single time.

> **Trainer note:** The licence details come from provider pages and secondary summaries. Confirm them on the day.

**ASK**
27. Quick check. If a model is open-weight, can you automatically use it in a commercial product?
28. **DO** Wait for answers. The right answer: not automatically. Check the licence.

---

## SLIDE 4: Proprietary Families and Their Tiers (3 min)

**SAY**
1. Now let's look at the main families, starting with proprietary models.
2. Three vendors on this slide. And one pattern repeats in all of them.
3. **DO** Point to Anthropic Claude.
4. Anthropic's Claude, in the Claude 5 generation.
5. Fable 5.1 is the top tier, with extra safeguards.
6. Opus 5.5 is high capability.
7. Sonnet 5.5 is the balanced option.
8. Haiku 5.5 is fast and low cost.
9. **DO** Point to OpenAI GPT.
10. OpenAI's GPT, in the GPT-6 generation.
11. Astra is the most capable, with computer use.
12. Sol is for complex coding and agents.
13. Luna is low cost, for high volume.
14. **DO** Point to Google Gemini.
15. Google's Gemini: the 3.x series, with Gemini 4 in preview.
16. The Pro tier is the flagship for reasoning.
17. The Flash tiers are fast and low cost.
18. Gemini 4 is in limited preview only.
19. (pause)
20. See the pattern?
21. Every vendor offers a top model, a balanced model, and a cheap fast model.
22. Tier names differ by vendor. So think in classes: **top, balanced, fast and low-cost**.
23. Context windows on the tiers shown are around one million tokens. Please confirm exact figures on each provider's model page.
24. And prices get cut often. Do not memorise them.

> **Trainer note:** GPT-6 Astra launched 3 September, and Sol and Luna on 22 September, per press coverage. Gemini details come from secondary sources. Confirm on the vendors' pages.

---

## SLIDE 5: Open-weight Families to Know (3 min)

**SAY**
1. Now the open-weight side. Six families to know.
2. **DO** Point to gpt-oss.
3. gpt-oss, from OpenAI, under Apache 2.0.
4. It comes in 20 billion and 120 billion parameter versions. Both are mixture-of-experts, with 128K context.
5. The 20B fits laptop-class memory. The 120B fits one 80 gigabyte GPU.
6. **DO** Point to DeepSeek V4.
7. DeepSeek V4, under MIT.
8. The Pro version has 1.6 trillion total parameters, with 49 billion active.
9. The Flash version has 284 billion total, with 13 billion active.
10. It has 1 million context. This is data-centre scale.
11. **DO** Point to Qwen 3.x.
12. Qwen 3.x, from Alibaba. The licence varies.
13. It ranges from small dense models up to huge mixture-of-experts models. 256K context.
14. Check each model's licence.
15. **DO** Point to Mistral Small 4.
16. Mistral Small 4, under Apache 2.0.
17. 119 billion total, 6.5 billion active. Multimodal, 256K context. Its focus is enterprise and EU-sovereign use.
18. **DO** Point to Llama 4.
19. Llama 4, from Meta, under the Llama Community licence.
20. Scout is 109 billion, and Maverick is 400 billion. Both have 17 billion active. Multimodal.
21. Attribution and scale-based conditions apply.
22. **DO** Point to Gemma 4.
23. Gemma 4, from Google, under Apache 2.0.
24. Small models for laptops and edge devices. A good fit for local and constrained hardware.
25. (pause)
26. Here is the pattern to notice.
27. Most large open-weight models are **mixture-of-experts**.
28. So their active parameters are far lower than their total parameters.
29. Section 3 explains this properly. For now, just notice it.
30. One more mention: GLM from Z.ai and Kimi from Moonshot lead several open-weight rankings. But they are data-centre scale.

---

## SLIDE 6: Positioning by Class of Model (4 min)

**SAY**
1. Now let's step back from names and look at classes.
2. This table is a qualitative map. It is **not** benchmark scores.
3. Dots show the typical profile of each class. More dots means stronger.
4. **DO** Point to the Frontier API row.
5. Frontier API: top marks for reasoning, coding, multimodal and tool use. But the slowest. And not self-hostable.
6. **DO** Point to the Balanced API row.
7. Balanced API: strong coding and tool use, good speed. A very common starting point for agents.
8. **DO** Point to the Fast, low-cost API row.
9. Fast, low-cost API: lower on reasoning, but the best on speed.
10. **DO** Point to the Large open-weight row.
11. Large open-weight, multi-GPU: strong reasoning and coding. Self-hostable, but at data-centre scale.
12. **DO** Point to the Mid open-weight row.
13. Mid open-weight, one GPU: middle across the board. Self-hostable on a single GPU.
14. **DO** Point to the Small open-weight row.
15. Small open-weight, laptop or edge: weaker on capability, but fast. Runs locally.
16. (pause)
17. Three takeaways from this table.
18. Frontier APIs lead on peak capability, but are the slowest and the priciest.
19. Fast API models and small open-weight models win on speed and cost.
20. The open-weight classes are the only ones you can self-host.
21. And the gaps are narrowing quickly. Mid-size open-weight models now handle many agentic tasks.
22. Read the footnote: these are typical profiles for each class, not scores for any single model.
23. Individual models vary. Validate on your own tasks.

**ASK**
24. Which row would you start from for your own use case?
25. **DO** Take two or three answers. Ask "why that row?" each time.

---

## SLIDE 7: Comparing Models in Practice (12 min including live demo)

**SAY**
1. Now the practical part. Let's compare models properly.
2. **DO** Point to the three model boxes.
3. We will compare three models.
4. Model A: an API model, top tier.
5. Model B: an API model, fast tier.
6. Model C: an open-weight model, self-hosted.
7. **DO** Point to "Representative tasks".
8. We test them on representative tasks.
9. That means real prompts, tools and documents from your use case.
10. **DO** Point to the Scorecard.
11. And we fill in a scorecard with six measures.
12. Task-specific response quality.
13. Instruction following.
14. Tool-call and structured-output reliability.
15. Context capacity and long-context accuracy.
16. Latency: first token and total.
17. And cost per completed task.
18. (pause)
19. Now the rule that makes the comparison fair.
20. **DO** Point to the bottom line.
21. Fix the conditions. Same prompts, same tools, same settings.
22. Repeat the runs. At least five each.
23. Record tokens and timings, then compute cost.

**DO: Live demo (about 6 to 7 minutes)**
24. Run the same task through all three models. Use the same prompt, the same tool definitions, and the same temperature or reasoning effort.
25. Fill in the scorecard live, where everyone can see.
26. Ask the room to call out what they notice about speed, formatting and tool use.

**SAY: Cost per task (about 3 minutes)**
27. Now let's do the cost maths. Cost matters in agents because every loop step is a call.
28. The formula is on the board.
29. Cost per task equals input tokens times input price, plus output tokens times output price, divided by one million. And you sum it over every call in the agent loop.
30. Here is a worked example with hypothetical prices.
31. A top-tier model: 5 dollars per million tokens in, 25 dollars per million out.
32. A fast-tier model: 50 cents in, 2 dollars 50 out.
33. Take a task with 10 calls, in total 80 thousand tokens in and 5 thousand tokens out.
34. The top-tier model costs about 53 cents per task.
35. The fast-tier model costs about 5 cents per task.
36. Now scale it. For 100,000 tasks.
37. That is roughly 52,500 dollars versus 5,250 dollars.
38. (pause)
39. Same job. Ten times the price.
40. Two more points.
41. Latency compounds across sequential calls. Ten calls in a row means ten waits.
42. And check long-context accuracy with **your own** documents.
43. A large window is capacity. It is not proof of reliable use.

> **Trainer note on the maths (so you can answer if asked):**
> Top tier: 80,000 × $5 / 1M = $0.40, plus 5,000 × $25 / 1M = $0.125, total about $0.525.
> Fast tier: 80,000 × $0.50 / 1M = $0.04, plus 5,000 × $2.50 / 1M = $0.0125, total about $0.0525.
> These prices are hypothetical, for teaching only.

**ASK**
44. Which model would you pick for a high-volume triage task, and which for a complex research task? Why?
45. **DO** Take two answers. Connect them to the scorecard measures.

---

## SLIDE 8: Why Benchmarks Alone Do Not Decide (5 min)

**SAY**
1. You will see benchmark tables everywhere. So let's be clear about what they can and cannot tell you.
2. **DO** Point to the left column.
3. What benchmarks **do** tell you.
4. A rough capability tier.
5. Progress across model generations.
6. And they are a useful filter for building a shortlist.
7. **DO** Point to the right column.
8. What benchmarks **cannot** tell you.
9. Fit with your tasks, prompts and tools.
10. Tool-call and schema reliability in your agent.
11. Latency and cost under your load.
12. Data-governance and deployment fit.
13. And how vendor-reported scores were run.
14. (pause)
15. Let me explain why that last one matters.
16. Scores are often vendor-reported, and different vendors use different test setups.
17. Tests can saturate, meaning everyone scores near the top, so they stop separating models.
18. Tests can also leak into training data.
19. And none of them measure your agent's tool definitions, your latency budget, your cost, or your governance constraints.
20. **DO** Point to the line at the bottom.
21. So remember this sentence: use benchmarks to shortlist. Use your own tasks to decide.
22. We return to this in Section 5, where we validate models on representative tasks.

**ASK**
23. Has a benchmark ever pointed you to the wrong model for your task?
24. **DO** Take one or two stories. If quiet, give an example: a model that ranks high overall but fails on your JSON schema.

---

## SLIDE 9: Key Takeaways (1 min)

**SAY**
1. Let's wrap up.
2. Open-weight brings control and data residency, but shifts operations to you.
3. APIs trade control for convenience and peak capability.
4. **DO** Point to takeaway 1.
5. One: open-weight is not open source. Read the licence of every model.
6. **DO** Point to takeaway 2.
7. Two: think in classes. Frontier, balanced and fast. Large, mid and small open-weight. Names change monthly.
8. **DO** Point to takeaway 3.
9. Three: compare on your own tasks under fixed conditions. Benchmarks only shortlist.
10. **DO** Point to takeaway 4.
11. Four: next is Section 3, Families, sizes and capabilities.
12. In Section 3 we look inside the families.
13. We look at parameter counts, total versus active parameters, and capabilities such as reasoning, tool calling and context handling.
14. (pause)
15. Any questions before we move on?

---

## Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| Which is better, open-weight or proprietary? | Neither in general. Open-weight gives control and data residency; APIs give convenience and peak capability. |
| Is open-weight the same as open source? | No. Open-weight means downloadable weights, often with licence limits. Fully open source also releases training code, data and recipe. |
| Can I use any open-weight model commercially? | Not automatically. Read the licence file for that exact model. |
| Why do licences differ inside one family? | Licences are set per model, not per family (e.g. Qwen and Mistral). |
| Do I have to pick just one type? | No. Hybrid designs are common: API for hard reasoning, self-hosted for sensitive or high-volume steps. |
| Which model name should I remember? | Remember the classes, not the names. Names and versions change monthly. |
| What does "mixture-of-experts" mean here? | Only part of the model is active per token, so active parameters are much lower than total. Section 3 covers it. |
| Is a bigger context window always better? | No. It is capacity, not proof of reliable use. Test with your own documents. |
| Can I trust benchmark scores? | Use them to shortlist only. Decide using your own tasks. |
| Why is cost per task so different from price per token? | Agents make many calls per task, and each call has input and output tokens. |

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 1 min |
| 2. Open-weight vs. proprietary | 9 min |
| 3. Not open source | 7 min |
| 4. Proprietary families | 3 min |
| 5. Open-weight families | 3 min |
| 6. Class positioning | 4 min |
| 7. Live demo | 12 min |
| 8. Benchmarks | 5 min |
| 9. Takeaways | 1 min |

If you run over, trim discussion on slides 2 and 6, and read fewer model details on slide 5. **Never cut the live demo on slide 7.**
