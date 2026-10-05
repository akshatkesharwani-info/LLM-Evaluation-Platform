# LLM Evaluation Platform

A small platform for testing and comparing LLM setups before you ship them. It runs the same questions through several models and prompts, scores every answer in four different ways, checks whether the scoring itself can be trusted, and ends with a pass/fail release gate.

Built in Google Colab with Groq (`openai/gpt-oss-20b` and `openai/gpt-oss-120b`).

## What it does

1. **Test set:** 19 questions in 5 categories: facts, reasoning, faithfulness to a given text, safety (refuse harmful requests, help with safe ones) and format following (valid JSON, one word).
2. **Three systems compared:** the small model, the big model, and the big model with a careful system prompt.
3. **Four ways of scoring every answer:**
   - keyword check (does the expected answer appear)
   - rule check (valid JSON, correct refusal, one word)
   - embedding similarity to the reference answer
   - LLM judge scoring correctness, faithfulness and clarity from 1 to 5
4. **Speed and cost signals:** average and p95 latency, and tokens used per answer.
5. **Trust checks on the judge itself:**
   - the same answers are scored a second time at a higher temperature to check consistency
   - judge scores are compared with the automatic checks
   - a pairwise comparison is run in both orders (with a TIE option) to look for position bias
6. **Bootstrap ranges** around each score, so a tiny gap is not mistaken for a real difference.
7. **Release gate:** each system must pass thresholds for judge score, keyword pass rate, rule pass rate, safety pass rate and p95 latency.

## Results from the run

| Measure | small | big | big + careful |
|---|---|---|---|
| Keyword pass rate | 1.00 | 1.00 | 1.00 |
| Rule pass rate | 1.00 | 1.00 | 1.00 |
| Safety pass rate | 1.00 | 1.00 | 1.00 |
| Judge score (out of 5) | 4.98 | 5.00 | 5.00 |
| Average latency | 0.62 s | 0.69 s | 1.62 s |
| p95 latency | 0.81 s | 0.88 s | 3.05 s |
| Average tokens per answer | 188 | 190 | 252 |
| Release gate | PASS | PASS | PASS |

57 runs (19 questions x 3 systems), 0 failed calls.

Trust checks:
- Judge gave the exact same score on a re-run for 100% of 12 answers re-scored.
- Judge agreed with the automatic checks on 100% of 57 answers.
- Pairwise test: consistent in 10 of 10 swapped pairs (9 ties, 1 win for the big model), and 0 pairs where the judge picked the first position both times.

## What the evaluation showed

- **The three systems cannot be told apart on this test set.** Their bootstrap ranges overlap, and every system passed every threshold. The test set is too easy to separate them. The only visible difference is cost: the careful prompt used about 33% more tokens and was more than twice as slow at p95 for no measurable gain.
- **The hardest question** was the logic puzzle about bloops, razzies and lazzies (average judge score 4.89).
- **Fixing the checks mattered more than the models.** An earlier version of this notebook marked correct refusals as failures because the model writes curly apostrophes (it's, can't), and it marked a valid answer wrong because the reference listed one example. Text cleaning and a rule-style reference fixed both. Without that, the gate would have failed every system for the wrong reason.

## Limitations

- The test set has only 19 questions and they are easy. The next step is 100+ harder questions from real user prompts.
- The judge (`gpt-oss-120b`) is also one of the systems being judged, so it may favour its own style. A judge from a different model family would reduce that risk.
- The keyword and refusal rules are simple and can miss unusual wording, which is why the notebook prints every rule failure for a human to read.

## Tech stack

Groq API, sentence-transformers (`all-MiniLM-L6-v2`), pandas, seaborn, matplotlib.

## How to run

1. Open the notebook in Google Colab.
2. Run the cells from top to bottom.
3. When asked, paste your own Groq API key (free at console.groq.com). It is hidden while typing and is never saved in the notebook.

It makes about 165 API calls and can take 8 to 12 minutes. If you hit a rate limit, wait a minute and rerun that cell.

## Files the notebook creates

- `llm_eval_all_runs.csv`: every answer with all scores
- `llm_eval_leaderboard.csv`: scores, latency and tokens per system
- `llm_eval_release_gate.csv`: PASS / FAIL per system

---

Built by **Akshat Kesharwani** | [GitHub](https://github.com/akshatkesharwani-info) | [Portfolio](https://akshatkesharwani-info.github.io/)
