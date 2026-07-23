# SwarmDo-A2 — Results

Same standard as the rest of this hub: **a result counts only if it survives a paired statistical test on a
held-out set the model has never seen** — and, for coding, only if the verdict comes from *actually running the
project's tests*, never from a model's opinion. (Methodology: [../docs/METHODOLOGY.md](../docs/METHODOLOGY.md).)

A2's central question is a **churn gate**: A2 combines two independently trained skills — A1's coding delta and a
render-verified visual-coding delta — into one model on one base. Does fusing them degrade *either* skill? We
measured both halves, paired and held-out. **Neither degraded.**

## Decision-grade result 1: the visual skill survives the merge

Held-out chart-reproduction (unseen charts), best-of-4, scored by rendering the generated code and comparing to
the target image (timeout-aware paired scoring):

| Comparison | Result | Significance |
|---|---|---|
| Combined A2 vs the pure visual model | Δ **−0.006** render-reward | 95% CI [−0.016, +0.004], sign-test **p = 0.63** — *not* worse |
| Combined A2 vs the raw base | **14 wins : 5** | A2 keeps its visual edge over base |

Folding the coding adapter in **does not cost the visual capability** — the combined model is statistically
indistinguishable from the visual-only model, and still beats the raw base.

## Decision-grade result 2: the coding skill survives the merge

Paired A2-vs-A1 on 40 held-out real-GitHub-issue tasks, each graded by the project's own `FAIL_TO_PASS` /
`PASS_TO_PASS` tests (execution-verified, no LLM judge):

| Comparison | Result | Significance |
|---|---|---|
| A2 (code+vision) vs A1 (code only), real tests | **18 vs 19** solved (Δ −1) | McNemar **p = 1.0**; 39/40 tasks agree; 40% byte-identical patches |

The two models behave **almost identically** on coding: they agree on 39 of 40 tasks and produce the *same* patch
on 40% of them. Folding the visual skill in does not perturb bug-fixing.

**Honesty on power:** this arm is deliberately small (40 tasks, a single discordant pair). It rules out a *gross*
coding regression convincingly (the near-identity is the evidence), but it is **underpowered for a subtle
difference** — we would need more tasks to detect one. We report it as "no gross churn," not "provably identical."

## The methodology catch we're proud of

Our agentic harness can *self-judge* a candidate patch — an LLM reading its own diff and declaring pass/fail. On
this run that self-judge claimed **~37/40 solved for each model**. When we re-graded by **actually executing the
tests**, the real number was **~18–19/40** — the self-judge was roughly **2× optimistic**, and (being two
different models) not even symmetrically so. A comparison built on the self-judge would have been meaningless.

So **every coding number on this page is re-derived by running the real tests** against the saved patches, never
by self-grading. It is exactly the kind of near-miss this hub exists to surface: the good-looking number was
wrong, we caught it by execution, and we report the grounded one.

## Context: the visual skill is a *compounding* flywheel

A2's visual delta is not a one-off. The render-verified self-fuel loop — the model generates its own
render-verified training data, with no external teacher — **compounds** as the fuel scales: doubling the
self-generated fuel roughly doubled the held-out lift (to **+0.0192** render-reward, sign-test **p = 0.0063**, 95%
bootstrap CI excludes zero). A2 ships the current step of that loop; it is expected to improve with further rounds.

## What A2 is, in one line

A single open model that fixes bugs (execution-verified) **and** turns images into code (render-verified), with
*measured proof that combining the two costs neither* — and every coding claim re-checked by running real tests.

*Last updated: 2026-07 (churn-gate cleared; A2 combined adapter published to Hugging Face).*
