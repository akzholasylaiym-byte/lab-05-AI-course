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

Bare: 16 correct / 4 wrong / 4 no answer (out of 24)
Layer 1: 18 correct / 6 wrong / 0 no answer (out of 24)
Layer 1 turned all 4 “no answer” cases into answers: 2 became correct and 2 became wrong. It changed what the agent can do, not just what it admits, because it gave the agent a way to look again after an error and recover some answers it would otherwise have missed. Layer 2 gave exactly the same numbers as layer 1, so it changed nothing measurable. Layer 3 (18 / 4 / 2) did not add any correct answers. It only turned 2 wrong answers (q18, q22) into “no answer”, so it changed what the agent admits, not what it can do. It makes the system more honest, but not more capable.


## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

One failure the full harness did not fix
Question: q18. Under layer 3 it ends as none (the answer was marked UNVERIFIED), so the harness refuses it rather than giving a wrong answer. The trace marks the first thing that went wrong as “ungrounded”.
What happened in the trace: the model called get_series for Jambyl, indicator cpi_yoy, 2022 to 2025, and got the series back (it starts with 16.1 for 2022). It then called calculate with (16.1 - 10.6)/100, which returned 0.055. The first mistake is the /100: the values are already in percent, so the difference in percentage points is simply 5.5 and needs no division. The second mistake came when the model wrote its answer: it gave 0.55 percentage points, which is not even the 0.055 that the tool returned.
Grounding did work here. The harness replied that 0.55 is not in any tool output and told the model to compute it with calculate. But the model did not change anything: it repeated the identical calculate call, got 0.055 again, and after the harness told it to stop calling tools it wrote 0.55 again. The harness told the model that the answer was unsupported, but not which number the tool actually produced, and it could not tell the model that the formula was wrong. Grounding only checks that a number appears in a tool output, so it cannot catch a wrong formula, and a repeated mistake just ends in UNVERIFIED.
What I would add: a better repair message. When grounding rejects an answer, the harness should quote the exact value from the last tool result (here 0.055) and forbid repeating an identical tool call. I would also add one line to the calculate/get_series tool description saying that values are already in percent, so a percentage-point difference needs no division by 100.
Cost: a longer prompt on every call (a few dozen extra tokens), plus one more model call in the cases where the repair message fires. It also adds a new failure mode: the model may now copy 0.055 straight from the tool output and give a grounded but still wrong answer, because grounding cannot detect the bad /100. The unit hint is also tuned to this one case and may not generalise.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

Workflow or agent for the monthly ranking?
I would ship the fixed workflow. On the 15 fixed questions it scored 13/15 versus 9/15 for the agent. It used 15 model calls versus 26, 3,852 tokens versus 19,835 (about 5x fewer) and 15.9 s versus 48.8 s (about 3x faster). On the rank questions (q07–q11), which are the “highest and lowest region” type Dala Insight wants, the workflow got 5/5 and the agent 3/5 (q08 and q11 wrong).
One caution: with only 15 questions the 95% intervals for accuracy overlap a lot (about [0.62, 0.96] for the workflow and [0.36, 0.80] for the agent), so the accuracy gap alone is not conclusive. The cost gap (5x tokens, 3x time) is clear, and the decision rests mainly on that plus the fixed shape of the monthly question. The alias weakness (q21, q22) matters little here, because in a ranking the region names come from the tool output, not from the question. I would choose the agent instead for open-ended questions, or ones where the user names a region by an alias or an unusual name, where the fixed workflow has no step that fits.

---

I wrote this file myself, without any AI tool.
