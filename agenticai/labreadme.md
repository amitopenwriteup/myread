# Lab Trainer Script: Running, Measuring and Selecting Quantised Models for Agents

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Companion file:** `Lab_Quantised_Models_for_Agents_Corrected.md` (the learner handout)
**Total time:** 90 minutes
**Links to:** Sections 3, 4 and 5 of the session (sizes and capabilities, quantisation, model selection)

---

## 1. Lab at a Glance

| Item | Detail |
|---|---|
| **Goal** | Learners run one small model at two quantisation levels, measure memory, speed and reliability, and use the evidence to choose a model for an agent |
| **Core lesson** | Quantisation trades memory and speed against possible quality loss. The only way to know the cost for **your** agent is to test it |
| **Hardest idea** | In Part 3, schema-constrained output makes JSON *valid* almost every time, but does not make it *correct* |
| **Key slide references** | Section 4 memory table (parameters × bytes per parameter); Section 5 selection framework |

### Timing Plan

| Part | Topic | Time |
|---|---|---|
| Intro | Goals and how the lab works | 3 min |
| Part 0 | Setup | 12 min |
| Part 1 | Predict memory before you measure | 10 min |
| Part 2 | Run the models and measure | 18 min |
| Part 3 | Test tool calling and structured output | 20 min |
| Part 4 | Model-selection workshop | 19 min |
| Wrap-up | Share-out, deliverable, cleanup | 8 min |

> **Trainer note:** The handout gives Part 0 as 15 minutes. Here it is 12, because you will start the downloads before the intro (see below) and use the waiting time for the intro and Part 1 reading. Protect Part 3 and Part 4.

---

## 2. Before the Session (Trainer Preparation)

Do these the day before.

| Task | Why |
|---|---|
| Run the whole lab yourself on a VM similar to the learners' | You will know the real file sizes, speeds and failures |
| Record your own numbers: `ollama list` sizes, `ollama ps` memory, tokens per second, test scores | You need a reference, and a fallback if a learner's machine fails |
| Confirm the model tags exist on the Ollama library page | Tags can change |
| Confirm your Ollama version supports the `format` option | Part 3 uses structured outputs. Update Ollama and the `ollama` Python package if needed |
| Decide whether learners use the 3B or the 1B builds | Use 1B for machines with 8 GB of RAM or less |
| Ask learners to start Ollama install and model downloads **before** the session if possible | The two downloads are about 5 GB and are the slowest step |
| Prepare a shared place for results (a table on screen, shared sheet, or chat) | Makes the share-out in Part 4 and the wrap-up quick |
| Prepare a backup: your own result files and screenshots | A learner whose machine fails can still do Parts 3 and 4 with your numbers |

---

## 3. Intro (3 min)

**SAY**
1. Welcome to the lab.
2. For the next 90 minutes, you will do hands-on what we discussed in Sections 3, 4 and 5.
3. You will run one small open-weight model, in two quantised builds, on your own machine.
4. You will measure memory, speed and quality.
5. You will test the two things agents depend on most: tool calling and structured output.
6. And at the end, you will make and defend a model-selection decision.
7. (pause)
8. Remember the big idea from Section 4.
9. Quantisation is a trade-off, not a free win.
10. In this lab, you will see that trade-off with your own numbers.

**SAY (how the lab works)**
11. The handout has five parts, Part 0 to Part 4.
12. Each part ends with a **checkpoint**. Do not move on until you pass it.
13. Each part also has a **reflection** question. Those matter more than the code.
14. Work at your own pace in Parts 0 to 3. In Part 4 you work in pairs.
15. If you get stuck for more than a couple of minutes, ask. Many problems here have simple causes.
16. (pause)
17. One rule: write your results in the tables as you go. You will need them in Part 4 and for the write-up.

**DO** Make sure everyone has the handout open and has started the Ollama install and model downloads.

---

## 4. Part 0: Setup (12 min)

**Purpose:** Get a working local inference setup. This is also a small lesson in running a model on your own infrastructure.

### Explaining the steps

**SAY**
1. In Part 0 we build the environment.
2. First, check your machine. Cores, memory, disk.
3. These are the same dimensions we used in Section 1: hardware matters.
4. **DO** Show `nproc`, `free -h`, `df -h /`.
5. Next, install the tools we need with `apt`.
6. Then install Ollama.
7. **DO** Show the `curl ... | sh` line.
8. A quick safety point.
9. Piping a script from the internet into `sh` runs it with your permissions.
10. On a lab machine, that is fine. In production, download the script and read it first.
11. (pause)
12. The installer does three things.
13. It installs the binary, creates a service, and starts it.
14. So after installing, you check the service. `systemctl status ollama`.
15. You want to see "active (running)".
16. Then check the API with `curl` on port 11434.
17. This is the same interface our Python code will call.
18. (pause)
19. A warning.
20. Do **not** change `OLLAMA_HOST`.
21. By default, Ollama listens only on your own machine. That is what we want.
22. Ollama has no login. If you open it to the network, anyone who can reach your machine can use it.
23. (pause)
24. Next, a Python virtual environment, and `pip install ollama`.
25. Then pull the two builds.
26. **DO** Point to the two tags.
27. Same model. Same size. Two different quantisation levels.
28. Q4_K_M is the smaller, more compressed build.
29. Q8_0 is the larger, higher-precision build.
30. If your machine has 8 GB of RAM or less, use the 1B tags from the handout. Then use those names everywhere.
31. Last, run the quick test.
32. `ollama run` with "Say hello in one sentence".
33. That also loads the model into memory, so later runs start faster.

**ASK**
34. Checkpoint. Does `ollama list` show both models, and did the greeting work?
35. **DO** Ask for a show of hands. Help anyone who is stuck.

### What learners should see

- `ollama --version` prints a version number.
- `systemctl status ollama` shows `active (running)`.
- `curl http://localhost:11434/api/tags` returns JSON.
- `ollama list` shows two models. The Q8 file is clearly larger than the Q4 file, roughly 1.5 to 2 times.

### Common problems in Part 0

| Symptom | Likely cause | Fix |
|---|---|---|
| `connection refused` | Service not running | `sudo systemctl start ollama` |
| Port already in use | Service and `ollama serve` both running | Stop one of them |
| Model not found / 404 | Wrong tag, or not pulled | Compare with `ollama list` and the library page, then re-pull |
| `pip: command not found` | pip not installed | `sudo apt install -y python3-pip` |
| No `(lab-env)` in the prompt | Environment not activated | `source lab-env/bin/activate`, or re-create with `--clear` |
| Download very slow | Network | Move on to Part 1 while it finishes |

> **If behind:** Tell anyone still downloading to start Part 1 on paper. Part 1 needs no software.

---

## 5. Part 1: Predict Memory Before You Measure (10 min)

**Purpose:** Practise the one calculation from Section 4, and see why estimates are always a lower bound.

**SAY**
1. Before we measure anything, we predict.
2. Remember the rule from Section 4.
3. **Memory ≈ parameters × bytes per parameter.**
4. Our model has about 3 billion parameters.
5. **DO** Point to the table in the handout.
6. Fill in the three rows.
7. FP16 uses 2 bytes per parameter. INT8 uses 1. INT4 uses half a byte.
8. Take two minutes. Do it on paper first.

**DO** Wait two minutes. Then reveal the expected answers.

**SAY (reveal)**
9. FP16: about 6 gigabytes.
10. INT8: about 3 gigabytes.
11. INT4: about 1.5 gigabytes.
12. Now compare with your `ollama list` sizes.
13. Your Q4 file is probably around 2 gigabytes. Your Q8 file is probably around 3.4.
14. So the real files are **larger** than our estimates.

**ASK (reflection)**
15. Why is the real file bigger than the estimate?
16. **DO** Take two or three answers.

**SAY (answer, if needed)**
17. Four reasons.
18. One. The model has slightly more than three billion parameters.
19. Two. The Q4_K_M build is mixed precision. It averages a little over 4 bits per weight, not exactly 4.
20. Three. Some layers are kept at higher precision.
21. Four. Scaling factors and metadata add a little.
22. (pause)
23. And now the bigger question.
24. **What else uses memory when the model runs?**
25. **DO** Wait for answers.
26. **SAY** The KV cache, activations, and runtime overhead.
27. So the file size is a **floor**, not the total.
28. We will see the real total in Part 2, with `ollama ps`.

> **Trainer note:** The numbers here are approximate and depend on the exact release. Check them against your own `ollama list` before the session.

---

## 6. Part 2: Run the Models and Measure (18 min)

**Purpose:** Get real measurements of memory and speed, and look at quality side by side.

### Explaining `bench.py`

**SAY**
1. Now we measure. Create `bench.py` from the handout.
2. Let me explain what it does before you run it.
3. **DO** Walk through the code.
4. It takes the model name from the command line.
5. It has three prompts. One short explanation, one coding task, one customer-support task.
6. Why those three? They test explanation, code, and a realistic agent-style task.
7. For each prompt, it calls `ollama.chat`.
8. **DO** Point to `temperature: 0`.
9. Temperature zero makes the output as repeatable as possible, so we compare the two builds fairly.
10. **DO** Point to `eval_count` and `eval_duration`.
11. These are the runtime's own counters. Tokens generated, and time spent generating, in nanoseconds.
12. Tokens divided by seconds gives tokens per second.
13. Note: this measures **generation** only. Model loading time is not included. So it is a fair speed measure.
14. (pause)
15. Run it for both builds. Use `tee`, so you see the output and also save it.
16. **DO** Show the two commands.

### Explaining memory measurement

**SAY**
17. While the model is loaded, or right after the run, open a second terminal.
18. Run `ollama ps`.
19. It shows the loaded model, its **total memory size**, and how it is split between CPU and GPU.
20. This total includes working memory like the KV cache. So it is larger than the file size.
21. Use `free -h` or `htop` too, if you like.

### Scoring quality

**SAY**
22. Now score quality, from 1 to 5, for each of the three answers.
23. Use the rubric in the handout. Five is correct, complete and clear. Three is mostly correct with gaps. One is wrong or unusable.
24. If you can, ask your partner to shuffle the files, so you do not know which build wrote which answer.
25. Be honest. Do not expect a big difference.

**ASK (reflection)**
26. Is the speed difference what you expected?
27. Is any quality difference visible, or do the outputs look similar?
28. **DO** Take three or four answers. Collect speeds on a shared table if you can.

**SAY (framing)**
29. Three points to remember.
30. One. Your numbers depend on your hardware. Someone with a GPU will see very different speeds from someone on a CPU.
31. Two. Three prompts is a tiny sample.
32. Three. Casual chat quality often looks the same across builds. That is exactly why Part 3 tests something harder.

### Common problems in Part 2

| Symptom | Likely cause | Fix |
|---|---|---|
| `KeyError: 'eval_count'` | Different client version | Print the full response and check field names |
| Very slow, or machine freezes | Model too large for free memory, swapping | Use the 1B builds, close other applications |
| First prompt much slower | Model loading | Normal. Speed is measured on generation only |
| `ollama ps` shows nothing | Model already unloaded | Run it right after a run, or rerun the script |

> **What to expect (hedged):** Q4 is usually smaller and often faster than Q8. The size of the speed gain varies a lot by hardware. On some CPU-only machines it can be large, on others small. Do not promise a specific result.

---

## 7. Part 3: Test Tool Calling and Structured Output (20 min)

**Purpose:** Test the two capabilities agents depend on most, and learn why a naive test can hide problems.

> **Trainer note:** This is the most important part of the lab. Go slowly through the explanation of the JSON test.

### Framing

**SAY**
1. Remember Section 3 and Section 4.
2. Tool calling and structured output can fail **before** casual chat quality does.
3. And one malformed output can break a whole agent loop.
4. So now we test them directly.

### Explaining `tools_test.py` (about 8 minutes including run)

**SAY**
5. Create `tools_test.py`.
6. First, it defines one tool: `get_order_status`. It takes an `order_id`, which is a string.
7. Then four test cases.
8. **DO** Point to the cases.
9. Three questions that **should** call the tool, each with a different order ID.
10. And one question that should **not** call any tool: "What is the capital of France?"
11. That last case matters.
12. A reliable agent must know **when not to use a tool** as well as when to use one.
13. For each case, the script checks four things.
14. Was a tool called, or not, as expected?
15. Was it the right tool?
16. Was exactly one call made?
17. And was the `order_id` argument exactly right?
18. It prints PASS or FAIL, and the total.
19. Run it for both builds.

**ASK**
20. Did either build fail a case?
21. **DO** Take answers. For any failure, ask: was it a wrong tool, a wrong argument, an unneeded call, or no call at all?

### Explaining `json_test.py` (about 10 minutes including run)

**SAY**
22. Now structured output. This test needs careful explanation.
23. The task is simple. Classify a customer review as JSON.
24. Two fields: `sentiment`, which is positive, negative or neutral, and `urgent`, true or false.
25. There are five reviews, each with a **known correct answer**.
26. **DO** Point to the `reviews` list with expected labels.
27. (pause)
28. Now the key idea. The script runs in **two modes**.
29. **DO** Point to the `mode` argument.
30. **Prompt mode.** We ask for JSON in the prompt. The model must obey on its own.
31. **Schema mode.** We pass a JSON schema. Ollama then **forces** the output to match it.
32. And each answer is scored two ways.
33. **Valid:** does it parse, and does it match the schema?
34. **Correct:** are the labels actually the right ones?
35. (pause)
36. Why do we need both modes and both scores?
37. Because of this.
38. In schema mode, the output **structure** is forced. So "valid" will be almost 100 percent for any model.
39. That tells you nothing about quantisation.
40. But the **content** is not forced. The model still has to judge the sentiment and the urgency correctly.
41. So "valid" can be perfect while "correct" is not.
42. **Constraining the format forces the structure, not the judgement.**
43. (pause)
44. Run all four combinations. Two builds, two modes.
45. **DO** Show the four commands.

> **If a learner gets an error about `format`:** Ollama or the Python client is too old for structured outputs. Update both: re-run the install script, and `pip install --upgrade ollama`.

### Record and reflect

**SAY**
46. Fill in the five-row table.
47. Tool-call score out of four. JSON valid and correct, in both modes.

**ASK (reflection 1)**
48. Compare prompt mode with schema mode. Why is "valid" so much easier in schema mode?
49. **DO** Take two answers. Target: the format is enforced by the runtime, not left to the model.

**ASK (reflection 2)**
50. Does schema mode also guarantee "correct"?
51. **DO** Wait. Target: no. It forces the shape, not the judgement.

**ASK (reflection 3)**
52. We used five or four test cases. What would you do to get trustworthy numbers?
53. **DO** Take answers. Target: many more cases, include edge cases and failures, repeat runs, test in the full agent loop.

**SAY (closing the part)**
54. Here is the lesson.
55. If both builds pass everything, that does **not** prove they are equal. It may mean your test is too easy.
56. If one build fails, ask exactly **how** it failed. That tells you what to fix, or whether to change the model.
57. (pause)
58. This is exactly why Section 5 said: validate with representative tasks, and run the full agent loop.

### What to expect (hedged)

- Schema mode should give near 100% "valid" for both builds.
- Differences, if any, are most likely to show up in tool-call arguments and in prompt-mode JSON, or in the "correct" score.
- At 3B, Q4 and Q8 may score the same on these small tests. If so, use that to discuss why a larger and harder test set is needed. **Do not promise learners a specific outcome.**

### Common problems in Part 3

| Symptom | Likely cause | Fix |
|---|---|---|
| No `tool_calls` at all | Model or version does not return tool calls in this format | Check the model supports tools, and that the `tools` list matches the example |
| Error about `format` | Ollama or client too old | Update both |
| Prompt-mode output has extra text or code fences | Model ignoring "ONLY JSON" | Counts as INVALID. This is a realistic failure. Discuss it |
| Results differ between runs | Small nondeterminism even at temperature 0 | Normal. Rerun, and note it |

---

## 8. Part 4: Model-Selection Workshop (19 min)

**Purpose:** Use measured evidence and the Section 5 framework to make a defensible decision.

### Setting up the scenario

**SAY**
1. Now we turn data into a decision. Work in pairs.
2. Here is the scenario.
3. A company wants an internal agent that answers staff questions about HR policies.
4. It must look up policy documents using a tool.
5. It must return answers as JSON for a web interface.
6. It must respond in about five seconds.
7. All data must stay on company servers.
8. And the only hardware is one workstation with a consumer GPU.
9. (pause)
10. Use the framework from Section 5.
11. Define. Constrain. Shortlist. Test. Decide.

### Step by step

**SAY**
12. Step 4.1: task requirements.
13. Complexity, reasoning, tool calling, structured output, context length, modality.
14. Three minutes.
15. Step 4.2: system constraints.
16. Latency, cost, data governance, hardware and deployment.
17. Two minutes.
18. Step 4.3: the decision matrix.
19. Three options: a small model at Q4, a small model at Q8, and a larger hosted API model.
20. Score each from 1 to 5, and use **your lab results** for tool-call and structured-output reliability.
21. Seven minutes for steps 4.1 to 4.3 in total, then four minutes for step 4.4.
22. Step 4.4: write your decision in two or three sentences.
23. Say what you would test next before committing.

**DO** Walk around the room. Listen for reasoning, not just scores. Prompt pairs who only give totals: "Why a 2 there?"

### Share-out (about 4 minutes)

**SAY**
24. Step 4.5: present to another pair, or to the room.
25. I want to hear **reasoning**, not just the total.

**ASK**
26. Which option did you pick, and which criterion decided it?
27. **DO** Take two or three pairs. Look for these ideas, and prompt if they are missing.

### Discussion points to draw out

| Idea | How to prompt |
|---|---|
| The hosted API model fails the data-governance requirement outright | "Did anyone score it low on governance? What does that do to the total?" This shows constraints come before capability |
| Q4 vs. Q8 is a trade of memory and speed against possible reliability | "What in **your** results supports your choice?" |
| A small sample is weak evidence | "How confident are you, with five test cases?" |
| A split design may be best | "Could a small model handle routine questions and a larger self-hosted model the hard ones?" |
| Name what you would test next | "What is the first thing you would test before committing?" |

> **Trainer note:** There is no single right answer. A good answer is **justified by the constraints and by measured evidence**, and names what to test next.

---

## 9. Wrap-Up (8 min)

**SAY**
1. Let's pull it together.
2. In Part 1, we predicted memory. The real total is always bigger than the file size.
3. In Part 2, we measured speed and memory, and learned that speed gains depend on hardware, and that casual quality often looks similar.
4. In Part 3, we tested tool calling and structured output.
5. And we learned that schema mode forces structure, not correctness.
6. In Part 4, we made a decision using constraints first, then evidence.
7. (pause)
8. The one sentence to take away.
9. **Quantisation is a trade-off. Decide with your own tests, on your own tasks, under your own constraints.**

**SAY (deliverable)**
10. For the write-up, submit one page.
11. Your completed tables from Parts 1 to 3.
12. Your decision matrix from Part 4.
13. And one paragraph answering: when is a quantised small model the right choice for an agent, and when is it not?

**ASK**
14. Any questions before we clean up?

**SAY (cleanup)**
15. Last, clean up.
16. Remove the models with `ollama rm`, and run `deactivate` to leave the Python environment.

**SAY (optional extensions)**
17. If you finish early, or want to continue at home, there are four extensions.
18. Try a Q2 or Q3 build and see where tool-call reliability breaks down.
19. Compare a larger model at Q4 with a smaller model at Q8.
20. Use a long pasted document and watch latency and memory change. That is the KV cache.
21. Or enlarge the test sets to 30 or more cases, and report pass rates.

---

## 10. Answers to the Reflection Questions (for the trainer)

| Part | Reflection question | Suggested answer |
|---|---|---|
| 1 | Why is the real file larger than the estimate? | Slightly more than 3B parameters, mixed precision in Q4_K_M (a little over 4 bits per weight), some layers at higher precision, scaling factors and metadata |
| 1 | What else uses memory at run time? | KV cache, activations, runtime overhead |
| 2 | Is the speed difference what you expected? | Often Q4 is faster, but the size of the gain depends on hardware and runtime |
| 2 | Is any quality difference visible? | Often small on casual prompts at temperature 0. Three prompts is a small sample |
| 3 | Why is "valid" easier in schema mode? | The runtime forces the output to match the schema |
| 3 | Does schema mode guarantee "correct"? | No. It forces structure, not judgement |
| 3 | How would you get trustworthy numbers? | Many more cases, edge cases and failures, repeated runs, full agent loop |
| 4 | When is a quantised small model the right choice? | Narrow or routine tasks, tight latency or cost, data must stay local, limited hardware, and tests show tool calls and JSON stay reliable |
| 4 | When is it not? | Multi-step reasoning, ambiguous instructions, high cost of errors, or tests show unreliable tool calls |

---

## 11. Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| Why compare Q4 and Q8 of the same model? | Same model and size, so the only change is precision. That isolates the effect of quantisation |
| Why temperature 0? | It makes outputs as repeatable as possible, so the two builds are compared fairly |
| Why is `ollama ps` bigger than the file size? | It includes working memory such as the KV cache, not only the weights |
| Why is the first run slow? | The model is loading into memory. Speed is measured on generation only, so it is not affected |
| Is Q4 always faster? | Often, but it depends on hardware and runtime |
| Why test "no tool" in the tool test? | A reliable agent must know when **not** to call a tool |
| Why use both prompt mode and schema mode? | They measure different things. Prompt mode tests obedience. Schema mode tests the content |
| If both builds pass, are they equal? | Not necessarily. Your test may be too easy. Add harder and more cases |
| Can we use this method for bigger models? | Yes. The method is the same, but memory needs grow, so check with the Section 4 table |
| Why would a hosted API be ruled out in Part 4? | Data must stay on company servers. Constraints can eliminate options before capability is compared |
| Why is the sample so small? | For time. Real validation needs many more cases, including failures |

---

## 12. Pacing Reminder

| Part | Target |
|---|---|
| Intro | 3 min |
| Part 0: Setup | 12 min |
| Part 1: Predict memory | 10 min |
| Part 2: Measure | 18 min |
| Part 3: Tools and JSON | 20 min |
| Part 4: Selection | 19 min |
| Wrap-up | 8 min |

If you run over, shorten the share-out in Part 4 and the reflection in Part 2. **Protect Part 3 and Part 4.** If many learners are stuck on setup, hand them your own result files so they can still do Parts 3 and 4.
