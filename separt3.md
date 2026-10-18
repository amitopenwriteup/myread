# Simple Lab Explained (Trainer Guide)

Companion to `simple_lab.md`. It explains **what each step does, why it is there, how to read the output, and what to say**. All numbers are from real runs of the lab.

---

## 1. The lab in one minute

**The story:** A tiny help-desk has **15 notes** about Linux commands (the "chunks"). Users ask **12 questions**. We check three things:

1. **Does search find the right note?** (Part A, then retrieval metrics in B)
2. **If an answer is wrong, whose fault is it?** The finder or the writer? (A6)
3. **How do we put a number on it?** Precision, Recall, F1, AP, MRR, nDCG, kappa, BLEU, ROUGE (Part B)

**The two files:**

| File | What it is |
|---|---|
| `data.json` | All data: 15 chunks, 12 questions (with the right chunk and a key command), reference answers, three sets of generated answers (`llm`, `flawed`, plus a baseline built on the fly), and two raters' labels |
| `lab.py` | One program, many small commands. You never need to read it; this guide says what each command does |

**Time plan (60 min):** Setup 5 · Part A 25 · Part B 30.

---

## 2. Picture to keep in mind

```
 Question ─▶ [ SEARCH finds notes ] ─▶ [ PROMPT = question + notes ] ─▶ [ WRITER gives answer ]
                    │                                                           │
          Part A2–A3, B3 measure this                              Part A6, B5–B6 measure this
          (Precision, Recall, MAP, MRR)                            (diagnose, BLEU, ROUGE)
```

Say this early: **"Two jobs, two kinds of marks. We always mark the finder first."**

---

## 3. What is inside `lab.py` (so you can answer "how does it work?")

| Command | What happens under the hood |
|---|---|
| `search "text"` | Runs the question through three rankers and prints the top 3 chunk ids from each |
| `eval` | For every question: ranks all 15 chunks, then computes Precision@k, Recall@k, F1@k, Average Precision, Reciprocal Rank, and averages over 12 questions |
| `prompt q07` | Takes the top k chunks and pastes them under the rules and the question |
| `diagnose` | For every question asks two yes/no things: *is a relevant chunk in the top k?* and *does the answer contain the key command?* Then prints a verdict |
| `chunk` | Cuts one sample document into pieces of a given size and overlap, then checks if three commands survive whole |
| `metrics <ranking> <total>` | Computes every metric for any 0/1 list you type |
| `kappa` | Compares two raters' labels, corrected for chance |
| `toy "sentence"` | BLEU and ROUGE for one sentence against "the cat sat on the mat" |
| `text --answers X` | BLEU and ROUGE for all 12 answers, averaged |

**The three rankers:**

| Name | Plain meaning | How it scores |
|---|---|---|
| `tfidf` | Match the **words** (rare words count more) | Compares word-count vectors with cosine similarity |
| `lsa` | A small "meaning" model | Squeezes the same word counts into 8 numbers per text, so words that appear together get grouped |
| `hybrid` | Ask both, merge | **Reciprocal Rank Fusion:** each ranker gives `1 / (60 + rank)` to every chunk; scores are added |

> **Honest note for trainers:** `lsa` is a stand-in for real embeddings. It needs no download but is weak on 15 tiny chunks. Real embedding models do better on paraphrases; the lab teaches the *idea*.

---

## 4. Step-by-step explanation

### Setup (5 min)

**What it does:** Creates the two files and runs one check.
**Expected check line:**
```text
method=tfidf k=3  Precision=0.306  Recall=0.875  F1=0.450  MAP=0.850  MRR=0.878
```
**Say:** "If everyone sees this exact line, we're all starting from the same place."
**If it fails:** wrong folder (needs `data.json` in the same directory), or the virtual environment is not active.

---

### A1. Look at the data (2 min)

**What it does:** Nothing runs. People read the file.
**Point to make:** every question has a **right answer page** (`relevant`) and a **key command** (`key`). These two are our "answer key".
**Say:** "Without an answer key we cannot grade. Somebody had to create this by hand; that is the cost of evaluation."
**Key observation:** `q07` ("run a script every night") has **no important word in common** with its chunk (`c08`, "cron runs scheduled jobs").

---

### A2. Search and rank (5 min)

**Command:** `python lab.py eval --k 3 --verbose`
**What it does:** For each question, ranks all 15 chunks, shows the top 3 and the quality of the ranking.
**How to read a line:**
```text
q07  relevant=['c08']  top3=['c07', 'c02', 'c08']  AP=0.333 RR=0.33
      ^ the right chunk   ^ what search returned    ^ ranking quality
```
- `RR=0.33` means the right chunk is at rank 3 (1 ÷ 3).
- `AP=1.000` lines are hidden in the guide, since they are "perfect".

**Three imperfect questions:**

| Question | What went wrong | Why |
|---|---|---|
| `q07` | Right chunk at rank 3 | Wrong chunks contain the word "script" |
| `q11` | Right chunk (`c12`) at rank **5**, not even in the top 3 | "compress" vs "compressed" are different words to plain word matching |
| `q12` | One of two right chunks missed | The second right chunk (`c05`) is below the top 3 |

**Say:** "These three are our teaching cases for the whole lab. Remember them."

---

### A3. Lexical vs "meaning" search (6 min)

**Command 1:** `python lab.py search "run a script every night"`
```text
  tfidf   ['c07', 'c02', 'c08']
  lsa     ['c07', 'c08', 'c02']
  hybrid  ['c07', 'c02', 'c08']
```
**How to read it:** `c08` is the right answer. `lsa` ranks it 2nd, the others 3rd. A small improvement from "meaning", but nobody gets it first.

**Command 2:** `for m in tfidf lsa hybrid; do python lab.py eval --method $m --k 3; done`

| Method | Precision@3 | Recall@3 | MAP | MRR |
|---|---|---|---|---|
| tfidf | 0.306 | 0.875 | **0.850** | **0.878** |
| lsa | 0.333 | **0.917** | 0.808 | 0.808 |
| hybrid | 0.333 | **0.917** | 0.822 | 0.836 |

**What to explain:**
- `tfidf` is best at putting the right chunk **first** (MRR, MAP).
- `lsa` and `hybrid` are best at having the right chunk **somewhere in the top 3** (Recall@3).
- **No winner everywhere.** That is why real systems combine methods (slide 7).

**Common confusion:** "Why is `hybrid` not the best on everything?" Answer: fusing two lists averages their strengths *and* their mistakes. It is a safety net, not magic.

---

### A4. Chunking (3 min)

**Command:** `python lab.py chunk --size 45 --overlap 0`
**What it does:** Cuts a short "backup how-to" into 45-character pieces and checks whether three commands are still in one piece.
```text
chunk 1: 'To back up a folder, first run tar -czf backu'
chunk 2: 'p.tar.gz folder. Then copy it with scp backup'
...
'tar -czf backup.tar.gz'       intact? False
'crontab -e'                   intact? False
'30 2 * * * /opt/backup.sh'    intact? True
```
**Say:** "The command `tar -czf backup.tar.gz` is split across chunk 1 and chunk 2. A search for it would find *neither* chunk strongly."
**Fix:** `--overlap 25` repeats the last 25 characters in the next chunk. Now all three are `True`.
**Trade-off to say out loud:** overlap costs a little storage, saves facts. One huge chunk (`--size 300`) keeps everything but mixes topics.
**Slide link:** slide 9, "Chunking" knob.

---

### A5. The RAG prompt (3 min)

**Command:** `python lab.py prompt q07 --k 3`
**What it does:** Builds the exact text a RAG system would send to the writer (LLM).
**Parts of the prompt:** rules ("use only the context", "cite", "say not found") → numbered chunks → the question.
**Point to make:** the writer **only sees what is in this prompt**. With `--k 3`, `[3] (c08)` holds the answer. With `--k 1`, the cron chunk is gone, and the only honest answer is "not found".
**Say:** "If it isn't in the prompt, the model has to guess. That's where made-up answers come from."
**Slide link:** slide 8 (RAG flow) and slide 9 (k and prompt rules).

---

### A6. Diagnose (6 min), the most important step

**The rule:**

| Evidence in top k? | Answer correct? | Verdict |
|---|---|---|
| Yes | Yes | OK |
| Yes | No | **FIX GENERATION** |
| No | No | **FIX RETRIEVAL** |
| No | Yes | **LUCKY** (answer came from memory, not your documents) |

"Answer correct" is simple here: the answer must contain the key command (for example `kill -9`).

**Run 1:** `python lab.py diagnose --answers baseline --k 3`
```text
q07  evidence_in_top3=True   answer_ok=False -> FIX GENERATION
q11  evidence_in_top3=False  answer_ok=False -> FIX RETRIEVAL
```
- `q11`: the right chunk was never retrieved → **retrieval** problem. Better prompts cannot help.
- `q07`: the evidence *was* in the top 3, but the baseline writer copies only chunk #1. The label says "generation", but the **root cause is ranking** (right chunk was 3rd). **Great teaching moment:** a verdict is a first diagnosis, not the final one.

**Run 2:** `python lab.py diagnose --answers flawed --k 3`
```text
q05  ... answer_ok=False -> FIX GENERATION      (kill -1 instead of kill -9)
q06  ... answer_ok=False -> FIX GENERATION      (systemctl start instead of enable)
q11  evidence_in_top3=False  answer_ok=True -> LUCKY
```
**Say about LUCKY:** "The answer looks right, but the right chunk was never shown. The writer used its memory. Next time it may be wrong, and nobody will know why."

**Slide link:** slide 10 (How RAG fails), slide 35 (RAG triad: groundedness).

---

### B1. The running example (8 min)

**Command:** `python lab.py metrics 1,0,1,0,0,1,0,0,0,0 4 --grades 3,0,2,0,0,1,0,0,0,0`
**How to read the input:** ten results; `1` = relevant; relevant at ranks 1, 3, 6; **4 relevant exist in total**, so one was never found.
**Let people calculate first** (k = 5): P = 2/5 = 0.40, R = 2/4 = 0.50, F1 = 0.444.

| Number | Meaning in one line |
|---|---|
| **P@k** | Of the k shown, how many were relevant |
| **R@k** | Of all relevant ones, how many were in the k |
| **F1@k** | One mark that is low if either P or R is low |
| **AP = 0.542** | (1/1 + 2/3 + 3/6) ÷ 4. Rewards early finds, punishes misses |
| **RR = 1.0** | First relevant is at rank 1 |
| **nDCG = 0.943** | Uses grades 0–3; best results should be on top |

**Watch for:** "why divide by 4?" → the missing chunk counts against you.
**Pattern to point out in the table:** recall only goes up as k grows; precision tends to fall; **F1 peaks at k = 6** (0.600).

---

### B2. Break it on purpose (6 min)

**What it does:** Same command, different lists. People predict, then run.

| List | AP | RR | What it teaches |
|---|---|---|---|
| `0,0,0,1,...` (total 1) | 0.250 | 0.250 | A single late hit |
| `1,1,1,1,...` (total 4) | 1.000 | 1.000 | A perfect ranking |
| all zeros | 0.000 | 0.000 | Nothing found, still no crash |
| `0,0,0,0,1,1,1,1,...` | 0.365 | 0.200 | Same items as the perfect list, but late |

**Key lesson (last row):** its **Recall@10 is 1.0, the same as the perfect list**, but AP and RR are far lower. **Recall cannot see order.**
**nDCG demo:** best chunk (grade 3) at rank 6 → nDCG **0.449**, versus 0.943 when it was near the top.

---

### B3. Metrics on the real system (6 min)

**Command:** loop over `k = 1, 3, 5` and the three methods.

| k | Method | Precision | Recall | F1 |
|---|---|---|---|---|
| 1 | tfidf | **0.833** | 0.792 | **0.806** |
| 1 | lsa | 0.667 | 0.625 | 0.639 |
| 1 | hybrid | 0.750 | 0.708 | 0.722 |
| 3 | tfidf | 0.306 | 0.875 | 0.450 |
| 3 | lsa / hybrid | 0.333 | 0.917 | 0.483 |
| 5 | tfidf | 0.200 | 0.958 | 0.329 |
| 5 | lsa / hybrid | 0.217 | **1.000** | 0.353 |

**Three talking points:**
1. **k changes everything.** At k = 1 `tfidf` leads; at k = 5 `lsa`/`hybrid` find everything.
2. **Precision is small by design here.** Most questions have only **one** relevant chunk, so P@3 can never exceed 1/3. Do not read it as "the system is bad".
3. **Choose by the job.** A RAG prompt with 3 chunks cares about Recall@3 (is the evidence in the prompt?), so `lsa`/`hybrid` look better.

---

### B4. Rater agreement (4 min)

**Command:** `python lab.py kappa`
```text
agreement=0.80  by-chance=0.52  kappa=0.583
disagree on items: [4, 9, 14, 16]
```
**What it does:** Compares two raters on 20 items. Plain agreement is 80%. Kappa subtracts the agreement you would get by guessing.
**Say:** "Two people flipping coins would agree half the time. Kappa asks: how much *better than coin-flipping* are we?"
**Reading:** 0.58 is moderate. Four items to discuss, then fix the guidelines.
**Slide link:** slide 18.

---

### B5. BLEU and ROUGE on cat and mat (6 min)

**Command:** `python lab.py toy 2>/dev/null` (the `2>/dev/null` hides a harmless library warning)

| Score | Value | Where it comes from |
|---|---|---|
| BLEU-4, no smoothing | **0.000** | One zero (no matching 4-word group) kills the product |
| BLEU-4, smoothed | **0.380** | Smoothing stops one zero from wiping the score |
| ROUGE-1 F1 | 0.833 | 5 of 6 words covered |
| ROUGE-2 F1 | 0.600 | 3 of 5 word pairs covered |
| ROUGE-L F1 | 0.833 | 5 words in the same order |

**Let participants try their own sentences:** `python lab.py toy "your sentence" 2>/dev/null | tail -4`

| Sentence | BLEU | ROUGE-1 F1 | Lesson |
|---|---|---|---|
| `the feline rested on the rug` | 0.180 | 0.500 | **Same meaning, low score** |
| `the cat` | 0.135 | 0.500 (P 1.0, R 0.333) | Short → high precision, low recall |
| `the mat sat on the cat` | 0.427 | **1.000** | **Wrong meaning, perfect ROUGE-1** |

**Say:** "The checker counts matching words. It never reads for meaning."

---

### B6. Score all 12 answers (5 min)

**Command:** `python lab.py text --answers baseline|llm|flawed 2>/dev/null`

| Answers | BLEU | ROUGE-1 | ROUGE-2 | ROUGE-L |
|---|---|---|---|---|
| baseline (copy top chunk) | 0.136 | 0.431 | 0.230 | 0.349 |
| llm (good paraphrases) | **0.210** | **0.534** | **0.272** | **0.412** |
| flawed (llm + 2 wrong commands) | 0.197 | 0.509 | 0.242 | 0.387 |

**The punchline:** `flawed` has **two dangerous wrong commands**, yet its scores are only about 0.01 to 0.03 lower than `llm`. **The metrics barely noticed.**
**Say:** "That is why A6 (does the answer contain the key fact?) and human checks matter. Use BLEU/ROUGE as a smoke alarm for *change*, not as proof of *correctness*."
**Slide link:** slide 24.

---

## 5. Cheat sheet: reading every output

| You see | It means |
|---|---|
| `AP=1.000` | Right chunk(s) ranked at the very top |
| `RR=0.33` | First right chunk at rank 3 |
| Precision small, Recall large | Normal when each question has one right chunk |
| `FIX RETRIEVAL` | Right chunk never retrieved: change chunking, embeddings, k, or search method |
| `FIX GENERATION` | Right chunk was retrieved: change prompt, writer, or ranking |
| `LUCKY` | Right answer without evidence: not trustworthy |
| `intact? False` | A fact was cut between chunks |
| BLEU `0.000` for a short sentence | Missing smoothing, not a terrible answer |
| kappa < 0.6 | Raters need clearer guidelines |

---

## 6. Common questions from participants

| Question | Short answer |
|---|---|
| "Why is `lsa` so weak?" | It is a stand-in; with 15 short chunks it has little to learn from. Real embeddings are trained on huge text. |
| "Why isn't hybrid always best?" | It merges strengths **and** mistakes. Gains show up when the two rankers fail on *different* questions. |
| "Why is precision so low?" | Only one right chunk exists for most questions, so P@3 ≤ 1/3 by definition. |
| "Is `answer_ok` a real evaluation?" | It is a simple stand-in (does the key command appear?). Real projects use rubrics, humans, or groundedness checks. |
| "Can I use my own data?" | Yes. Replace the `corpus`, `queries` (with `relevant` and `key`) and `references` sections in `data.json`. |
| "Which metric should I report?" | Whichever matches the job: MRR for one answer, Recall@k for "don't miss anything", MAP or P@k for clean lists. Always say k. |

---

## 7. Facilitation tips

1. **Predict, then run.** Ask people to type a guess in chat before each command. Guessing makes the result memorable.
2. **Do not read code.** If someone asks, show the table in section 3.
3. **Use the three "problem questions" (`q07`, `q11`, `q12`) again and again.** Familiar examples beat new ones.
4. **Time-savers if you are behind:** skip A4 and B2's last two rows; keep A6, B1, B3 and B6.
5. **Checkpoint moments:** after Setup (paste the check line), after B3 (paste your Recall@3 for `hybrid` → expect 0.917), and after B6 (paste ROUGE-L for `llm` → expect 0.412).
6. **End with three sentences:** *Check retrieval first. Always say k. BLEU and ROUGE miss wrong facts.*
