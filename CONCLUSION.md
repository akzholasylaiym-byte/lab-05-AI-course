# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 changed the results the most. It was the only layer that raised the number of correct answers.

Layer 1 did. Bare: 16 correct / 4 wrong / 4 no answer; layer 1: 18 / 6 / 0 (out of 24). It turned all 4 “no answer” cases into answers (2 correct, 2 wrong), so it changed what the agent can do, because it can look again after an error. Layer 2 gave identical numbers. Layer 3 (18 / 4 / 2) added no correct answers; it only turned 2 wrong answers (q18, q22) into “no answer”, so it changed what the agent admits, not what it can do.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

q18 ends as none (UNVERIFIED) under layer 3. The model fetched the Jambyl cpi_yoy series (2022 starts at 16.1), then called calculate with (16.1 - 10.6)/100, getting 0.055. The /100 is wrong because values are already in percent. The model then answered 0.55, a number no tool returned. Grounding correctly rejected it, but the model repeated the same call and the same answer, because grounding cannot say which number the tool produced or that the formula is wrong.

I would add a repair message quoting the last tool value and forbidding identical repeated calls, plus a tool-description note that values are already in percent. Cost: a few dozen extra tokens per call (my estimate), one more model call when it fires, and a risk of a grounded but still wrong answer.
## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

I would ship the fixed workflow: 13/15 vs 9/15, 15 vs 26 model calls, 3,852 vs 19,835 tokens (about 5x fewer), 15.9 s vs 48.8 s (about 3x faster). On rank questions (q07–q11) it got 5/5 vs 3/5. With 15 questions the accuracy intervals overlap, so the decision rests mainly on the clear cost gap and the fixed question shape. I would choose the agent for open-ended questions or ones using region aliases (q21, q22).

---

I wrote this file myself, without any AI tool.
