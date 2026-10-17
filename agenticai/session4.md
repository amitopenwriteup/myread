# Section 4 Trainer Script: Quantisation & Efficient Model Deployment

**How to use this:** Read the lines under **SAY** out loud, one line at a time. Lines marked **DO** are actions, not words. Lines marked **ASK** are questions to the room. Pause where you see **(pause)**.

**Total time:** 35 minutes

> **Tone:** Keep this practical and conceptual. Avoid heavy maths. The only calculation learners need is: **memory ≈ parameters × bytes per parameter**.

---

## Before You Start: Demo Preparation (do this the day before)

The live demo on slide 8 only works if you choose everything in advance and put it on screen.

| Item | What to prepare | Your choice (fill in) |
|---|---|---|
| Model | A small open-weight model. State the exact name and size on screen | ____________ |
| Quantised build | A specific 4-bit or 8-bit file (e.g. GGUF Q4 or Q8) | ____________ |
| Higher-precision version | The FP16 version of the same model, for comparison | ____________ |
| Runtime | A local tool such as llama.cpp or Ollama | ____________ |
| Prompt 1 | A simple question, to compare answer quality | ____________ |
| Prompt 2 | A tool-call or JSON-output prompt, to test reliability | ____________ |
| Measures | Memory used and tokens per second, noted for each version | ____________ |

- Download the models **before** the session. Do not download live.
- Run both versions once beforehand so you know roughly what to expect.
- Have a backup: screenshots or a short recording, in case the demo fails live.

---

## Timing Plan

| Slide | Topic | Time |
|---|---|---|
| 1 | Title and link to Section 3 | 1 min |
| 2 | What we will cover | 1 min |
| 3 | What is quantisation and why use it | 4 min |
| 4 | Reduced precision vs. quantisation formats | 4 min |
| 5 | What can be quantised | 3 min |
| 6 | Precision and memory footprint | 4 min |
| 7 | Precision, performance and quality | 4 min |
| 8 | Demo: a small open-weight model, quantised | 6 min |
| 9 | Practical deployment considerations | 3 min |
| 10 | Quantisation is a trade-off | 3 min |
| 11 | Key takeaways | 2 min |

> **Trainer note:** The timings in the deck's notes add up to about 41 minutes. The plan above trims slides 2, 4, 5 and 9 slightly so everything fits in 35. Protect slides 6 (the memory arithmetic) and 8 (the demo).

---

## SLIDE 1: Title (1 min)

**SAY**
1. Welcome to Section 4: Quantisation and Efficient Model Deployment.
2. We have 35 minutes.
3. Let's link back to Section 3.
4. There we saw that larger models need more memory.
5. We also saw that memory follows **total** parameters, even in mixture-of-experts models.
6. So here is the practical problem.
7. What if the model you want does not fit on the hardware you have?
8. Quantisation is one of the most practical answers to that problem.

**ASK**
9. What hardware do you have access to? A laptop, a consumer GPU, a server GPU, or only cloud APIs?
10. **DO** Take three or four answers. Remember them. You will use them again on slide 9.

---

## SLIDE 2: What We Will Cover (1 min)

**SAY**
1. Four parts today.
2. First: why quantise. How it reduces memory and resource needs at inference.
3. **DO** Point to the second box.
4. Second: formats and targets. FP16 and BF16 versus INT8 and INT4. And what can be quantised: weights, activations and the KV cache.
5. **DO** Point to the third box.
6. Third: the trade-offs. Precision, memory, speed and output quality.
7. **DO** Point to the fourth box.
8. Fourth: deployment, and a demo. Local machines, consumer GPUs, and a quantised small model, live.
9. (pause)
10. I will keep this practical. No heavy maths.

---

## SLIDE 3: What Is Quantisation and Why Use It? (4 min)

**SAY**
1. Let's start with a simple definition.
2. Quantisation stores the model's numbers at **lower precision**. That means fewer bits per value.
3. (pause)
4. Here is an analogy.
5. Think of a photo.
6. If you save it at lower resolution, the file gets smaller.
7. It is faster to move around.
8. But you lose some detail.
9. Quantisation does the same thing to a model.
10. **DO** Point to bullet 2.
11. Why do it? Reason one: a smaller memory footprint.
12. That lets larger models fit on available GPUs or RAM.
13. **DO** Point to bullet 3.
14. Reason two: speed and cost.
15. There is less data to move for each token.
16. So inference is often faster, and serving costs less.
17. **DO** Point to bullet 4.
18. Reason three: it enables local, self-hosted and edge deployment.
19. And that helps with data governance.
20. Remember Section 2: open-weight models let data stay inside your own boundary.
21. Quantisation is what makes that possible on modest hardware.
22. **DO** Point to the last bullet.
23. Now the cost.
24. Some loss of numerical detail. And that may affect output quality.
25. (pause)
26. So the whole section comes down to one idea.
27. We trade a little precision for a lot of memory. The question is how much that trade costs **your** workload.

**Optional tie-in to Section 2**
28. Remember gpt-oss 120B, which fits on one 80 gigabyte GPU?
29. At 16-bit, 120 billion parameters would need around 240 gigabytes.
30. It fits because its weights are stored at much lower precision, roughly 4-bit.
31. That is quantisation making a large model practical.

---

## SLIDE 4: Reduced Precision vs. Quantisation Formats (4 min)

**SAY**
1. Now let's name the formats. Four of them.
2. **DO** Point to FP32.
3. FP32. Full precision, 32 bits per value.
4. It is common in training. It is rarely needed for inference.
5. **DO** Point to FP16 / BF16.
6. FP16 and BF16. Both are 16-bit floating point.
7. This is the standard inference baseline.
8. BF16 keeps FP32's range, but with less precision.
9. **DO** Point to INT8.
10. INT8. 8-bit integers, plus scaling.
11. About half the memory of FP16.
12. And usually only a small quality loss.
13. **DO** Point to INT4.
14. INT4. 4-bit integers, plus scaling.
15. About a quarter of the memory of FP16.
16. But the quality loss is more noticeable.
17. (pause)
18. There is one subtle point on this slide.
19. FP16 and BF16 are still **floating-point** formats. They are just lower precision than FP32.
20. INT8 and INT4 work differently.
21. They map values to small **integers**, and use scaling factors to convert back.
22. That is why we call INT8 and INT4 the quantisation formats.
23. And you will see names like GGUF Q4 or Q8. Those are file formats that package quantised models for local tools.

> **Trainer note:** If the room is not technical, skip the BF16 range detail. Say only: "FP16 and BF16 are the 16-bit baseline."

---

## SLIDE 5: What Can Be Quantised? (3 min)

**SAY**
1. Three things can be quantised.
2. **DO** Point to "Weights".
3. Weights.
4. This is the most common target.
5. It reduces the stored size of the model.
6. **DO** Point to "Activations".
7. Activations.
8. These are the intermediate values during computation.
9. They need hardware and kernel support.
10. And they are more sensitive to error.
11. **DO** Point to "KV cache".
12. The KV cache.
13. These are the stored attention keys and values.
14. It is the memory the model uses to remember the tokens it has already processed.
15. It matters for long contexts, and for many concurrent requests.
16. (pause)
17. In practice, many local setups quantise **weights only**.
18. Activation and KV-cache quantisation depend on your runtime and your hardware.

**ASK**
19. Why would the KV cache matter for an agent in particular?
20. **DO** Take one or two answers. Answer if needed: agents often carry long histories of tool calls and results, so the cache grows quickly.

---

## SLIDE 6: Precision and Memory Footprint (4 min)

> **Trainer note:** Work the arithmetic live on a whiteboard or in the chat. This is the one calculation learners should leave with.

**SAY**
1. This is the table I want you to remember.
2. The formula is simple.
3. **Memory equals parameters times bytes per parameter.**
4. Let's work it through.
5. **DO** Point to the FP16 / BF16 row.
6. FP16 and BF16 use 2 bytes per parameter.
7. A 7 billion parameter model: 7 billion times 2 is about 14 gigabytes.
8. A 70 billion parameter model: about 140 gigabytes.
9. **DO** Point to the INT8 row.
10. INT8 uses 1 byte per parameter.
11. So the 7B model is about 7 gigabytes. The 70B model is about 70 gigabytes.
12. **DO** Point to the INT4 row.
13. INT4 uses half a byte per parameter.
14. So the 7B model is about 3.5 gigabytes. The 70B model is about 35 gigabytes.
15. (pause)
16. Now read the small print at the bottom.
17. These numbers are approximate.
18. Real files are slightly larger. There are scaling factors. Some layers stay at higher precision.
19. And the KV cache and runtime overhead come **on top**.
20. So always leave headroom.

**ASK: Quick calculation**
21. You have a consumer GPU with 24 gigabytes of memory.
22. Does a 7B model at FP16 fit? Does a 70B model at INT4 fit?
23. **DO** Wait for answers.
24. **SAY** The 7B at FP16 is about 14 gigabytes. It fits, with room for the KV cache.
25. The 70B at INT4 is about 35 gigabytes. It does **not** fit on 24 gigabytes.
26. It would need something like 48 gigabytes, or a smaller model, or more quantisation with a quality cost.
27. (pause)
28. That is how you use this table. You work out what fits **before** you download anything.

---

## SLIDE 7: Precision, Performance and Quality (4 min)

**SAY**
1. Now the three-way trade-off. Memory, performance and quality.
2. **DO** Point to "Memory".
3. Memory.
4. Fewer bits per value gives a smaller footprint.
5. So bigger models, or longer contexts, can fit.
6. **DO** Point to "Performance".
7. Performance.
8. Less memory traffic often raises tokens per second.
9. But "often" is the key word. It depends on your hardware and your runtime.
10. **DO** Point to "Quality".
11. Quality.
12. Rounding error can reduce accuracy.
13. It can hurt instruction following.
14. And it can hurt tool-call reliability, especially at very low bit widths.
15. (pause)
16. Two warnings.
17. First: speed gains depend on hardware and kernel support. Lower precision does not always mean faster.
18. Second: quality loss is **not uniform**.
19. A model can still chat well, but start to fail on JSON output or tool arguments.
20. That is exactly the kind of failure that breaks an agent loop.
21. We saw in Section 3 how one malformed output can break the whole loop.

**ASK**
22. Why might tool calling degrade before general chat quality does?
23. **DO** Take one or two answers. Answer if needed: tool calls need exact formats, so small rounding errors show up as invalid arguments, while chat can stay fluent and approximate.

---

## SLIDE 8: Demo, a Small Open-weight Model, Quantised (6 min)

> **Trainer note:** Before this slide, check the prep table at the top. State the exact model, file and runtime on screen.

**SAY (set-up, 1 minute)**
1. Now let's see this live.
2. **DO** Point to the Model line.
3. The model today is: [say the exact name and size].
4. **DO** Point to the Format line.
5. The format is: [say the exact quantised build, for example GGUF Q4].
6. **DO** Point to the Runtime line.
7. And the runtime is: [say the tool, for example llama.cpp or Ollama].
8. **DO** Point to the Observe line.
9. We will watch three things.
10. Memory used. Tokens per second. And answer quality compared with the higher-precision version.

**DO (live demo, about 4 minutes)**
11. Load the quantised model. Note the memory used.
12. Run Prompt 1 on the quantised version. Note the tokens per second.
13. Run the **same** prompt on the FP16 version. Note memory, speed and answer.
14. Compare the two answers side by side.
15. Now run Prompt 2, the tool-call or JSON-output prompt, on both versions.
16. Ask the room: is the JSON valid? Are all the fields right?

**SAY (debrief, 1 minute)**
17. What did we see?
18. Memory: the quantised version used [say the number] compared with [say the number].
19. Speed: [say the result].
20. Quality: [say what you noticed].
21. Notice what we did not do.
22. We did not decide from the benchmark. We decided by running the **same prompts** on both.
23. And we included a tool-call prompt, because that is where agents are most sensitive.

> **If the demo fails:** Switch to your screenshots or recording. Say: "This is why we always test on our own setup. Things that work on a slide do not always work on a machine."

---

## SLIDE 9: Practical Deployment Considerations (3 min)

**SAY**
1. Now, where do you actually run these models? Three common settings.
2. **DO** Point to "Local machines".
3. Local machines.
4. CPU or unified-memory laptops can run small quantised models.
5. Speed is limited by memory bandwidth.
6. **DO** Point to "Consumer GPUs".
7. Consumer GPUs.
8. VRAM is the main limit.
9. Quantisation decides which model sizes fit.
10. And you must leave room for the KV cache.
11. **DO** Point to "Constrained environments".
12. Constrained environments.
13. Edge, on-premises or offline settings.
14. These favour small, heavily quantised models, and simple runtimes.

**ASK**
15. Earlier you told me what hardware you have.
16. Let's map it. Which model sizes and which quantisation levels would fit?
17. **DO** Use the table on slide 6. Go through two or three examples live.

---

## SLIDE 10: Quantisation Is a Trade-off, Not a Free Win (3 min)

**SAY**
1. Let's be honest about the limits.
2. **DO** Point to bullet 1.
3. The impact depends on the model, the quantisation method, the hardware and the workload.
4. **DO** Point to bullet 2.
5. Larger models usually tolerate lower precision better than very small ones.
6. **DO** Point to bullet 3.
7. So test on representative agent tasks.
8. Tool calls. JSON output. Long context.
9. **DO** Point to bullet 4.
10. And compare your options.
11. A smaller model at higher precision may beat a larger one at very low bit width.
12. (pause)
13. Here is an example.
14. Option A: a 70B model at very aggressive 2-bit. It fits, but may produce unreliable tool calls.
15. Option B: a 30B model at 8-bit. A similar memory footprint, and it may behave more reliably.
16. You do not know until you test.
17. Remember: tool calling and structured output can degrade **before** general chat quality does.

> **Trainer note:** The 70B and 30B pairing above is only an illustration of the pattern. Do not present it as a measured result.

---

## SLIDE 11: Key Takeaways (2 min)

**SAY**
1. Let's wrap up with five points.
2. **DO** Point to takeaway 1.
3. One: quantisation reduces memory and often improves speed, at some risk to quality.
4. **DO** Point to takeaway 2.
5. Two: FP16 and BF16 are reduced-precision floats. INT8 and INT4 are quantisation formats.
6. **DO** Point to takeaway 3.
7. Three: weights, activations and the KV cache can each be quantised.
8. **DO** Point to takeaway 4.
9. Four: memory is roughly parameters times bytes per parameter, plus cache and overhead.
10. **DO** Point to takeaway 5.
11. Five: validate on your own workload.
12. And next is Section 5: model selection for agentic systems.
13. There we bring together everything from Sections 1 to 4 into a selection framework.
14. (pause)
15. Any questions before we move on?

---

## Quick Answers if Questions Come Up

| Question | One-line answer |
|---|---|
| What is quantisation? | Storing a model's numbers at lower precision, with fewer bits per value. |
| Why quantise? | Smaller memory, often faster inference, lower cost, and it enables local and edge deployment. |
| What does it cost? | Some loss of numerical detail, which can affect quality. |
| What is the difference between FP16 and BF16? | Both are 16-bit floats. BF16 keeps FP32's range with less precision. |
| What are INT8 and INT4? | 8-bit and 4-bit integers with scaling factors. About half and a quarter of FP16 memory. |
| How do I estimate memory? | Parameters × bytes per parameter, plus KV cache and overhead. |
| How much memory does a 7B model need? | About 14 GB at FP16, 7 GB at INT8, 3.5 GB at INT4. |
| What is the KV cache? | Stored attention keys and values. It grows with context length and concurrent requests. |
| Is a quantised model always faster? | Often, but it depends on hardware and kernel support. |
| Does quantisation hurt all tasks equally? | No. Tool calls and structured output can degrade before general chat quality. |
| Is a bigger quantised model better than a smaller full-precision one? | Not necessarily. Test both on your own tasks. |
| What is GGUF? | A file format used to package quantised models for local tools such as llama.cpp. |
| Does MoE change the memory calculation? | Memory follows total parameters, so use the total count in the formula. |

---

## Pacing Reminder

| Slide | Target |
|---|---|
| 1. Title | 1 min |
| 2. Agenda | 1 min |
| 3. What and why | 4 min |
| 4. Formats | 4 min |
| 5. What can be quantised | 3 min |
| 6. Memory table | 4 min |
| 7. Trade-offs | 4 min |
| 8. Demo | 6 min |
| 9. Deployment | 3 min |
| 10. Trade-off, not free win | 3 min |
| 11. Takeaways | 2 min |

If you run over, shorten discussion on slides 5 and 9, and skip the BF16 detail on slide 4. **Protect slides 6 and 8.**
