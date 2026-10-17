# What the lab does, in simple words

Think of a **small library with 17 notes** about Linux commands. You ask it 8 questions, like *"how do I run something at midnight?"*, and the lab shows how well it finds the right note and answers.

## The 6 steps

1. **Word search vs meaning search.** The same question goes to both. Word search (BM25) misses paraphrases but is perfect for exact codes like `E1043`.
2. **Hybrid.** Combine both lists, so what one misses the other may catch.
3. **Re-ranking (optional).** A smarter model re-orders the top results.
4. **Tiny RAG.** Shows the exact prompt an AI would get: question plus retrieved notes.
5. **Diagnose.** When an answer is wrong, the tool tells you why:
   - **Retrieval failure:** the right note was never found.
   - **Generation failure:** the right note was found, but the answer didn't use it.
6. **Check each stage.** Did we *find* the right note (recall)? Did we put it *first* (precision)?

## One-line summary

> The lab asks the same questions to different search methods, then shows where each one fails: finding, ranking, or answering.
