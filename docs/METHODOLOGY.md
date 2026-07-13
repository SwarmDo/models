# Methodology

The reusable methods behind SwarmDo models. These are the parts we think are worth adopting whether
or not you use our weights.

---

## 1. Decision-grade evaluation for agentic coding

Most agentic-coding evaluations quietly mislead. Ours is built to be *decision-grade* — trustworthy
enough to bet a research direction on.

**Contamination-controlled held-out set.** Real GitHub issues the model has never seen, each graded
by the repository's own `FAIL_TO_PASS` / `PASS_TO_PASS` unit tests. Grading is **execution-verified**
— the tests either pass or they don't. No LLM-as-judge, no fuzzy text similarity (both of which we
found inflate scores).

**Paired McNemar, not two percentages.** To compare model A against model B, score *both* on every
task and look only at the tasks where they *disagree*. The exact McNemar test on those discordant
pairs tells you whether the difference is real. This is dramatically more sensitive than comparing
two overall resolve-rates, and it refuses to reward a model for easy tasks both already solve.

**Infrastructure-loss hygiene.** Timeouts and serving errors are excluded from *both* arms, and the
comparison is made on the paired intersection. Otherwise a flaky endpoint masquerades as a capability
difference. We also require two independent analyses (different loss-handling rules) to agree before
claiming a verdict.

**Why it matters:** this methodology is what let us tell a *real* +25-point improvement from the
statistical noise that several plausible-looking "improvements" turned out to be.

## 2. The base dominates — re-baseline aggressively

Our largest, most robust gain came not from a training trick but from **rebuilding on a stronger open
base model** the moment one became available (≈ +25 points, paired-significant). Test-time
scaffolding (best-of-N, better localization) gave only small, non-significant gains by comparison.

Practical rule: *before* investing in clever inference-time or training tricks, check whether a
better base now exists and re-baseline on it. The base's raw capability is the single biggest lever.

## 3. Selection is ceiling-bound; only training raises the ceiling

Best-of-N sampling and re-ranking can only choose among attempts the model already produces. If the
model fails a problem in *every* attempt, selection cannot fix it. We treat the **union of tasks
solved across many attempts** as the true ceiling, and recognize that raising it requires *training*,
not selection. This reframing keeps effort pointed at what can actually move the number.

## 4. Hindsight-Hint Distillation (HHD) — learning from near-misses

To add *new* solvable tasks (raise the ceiling), for each task the model currently fails:

1. We know the correct fix (bugs are mined from real merged fixes).
2. A one-sentence conceptual **hint** is written from that fix — pointing at the idea, forbidden from
   naming the change.
3. The model re-attempts *with the hint prepended*, in its own native format.
4. Keep only attempts that **pass the real tests**.
5. **Strip the hint** and fine-tune on the model's own successful trajectory.

The supervisor is the correct fix plus an execution oracle — never a "smarter" model — so it sidesteps
the trap that stronger teachers don't transfer in-harness.

**Trap we caught and fixed:** a hinted model *verbalizes the hint* while solving ("the hint suggests
…"). Training on those turns teaches reliance on a hint that won't exist at test time — a silent
poison. Naive prompt-stripping is insufficient; you must filter out the trajectory segments that
reference the hint at all. We measured this leak in nearly every recovered trajectory before filtering.

## 5. Reach the whole architecture when fine-tuning

Standard low-rank fine-tuning targets the usual attention projections. On a hybrid
**gated-DeltaNet linear-attention** model, those defaults **silently skip the layers that do most of
the sequence mixing** — freezing the majority of the reasoning machinery. Fixing the adapter to reach
those layers (while keeping the recurrence kernels frozen) was one of our most concrete wins. Lesson:
verify *which* modules your fine-tune is actually training against the real model internals — don't
trust the defaults on a non-standard architecture.

## 6. Serving hybrid linear-attention models

Hard-won operational notes for gated-DeltaNet / linear-attention hybrids:

- Verify tool-call round-trips **before** an evaluation — the wrong parser can *silently* drop tool
  calls and turn a capable model into an apparent failure.
- These models' recurrent kernels are sensitive to repeated in-place restarts; serve once per
  process rather than kill-and-restart.
- Long agentic contexts cost **prefill tokens**, not growing KV-cache memory (the recurrent state is
  fixed-size) — which changes what "context optimization" even means for these models.

---

*These methods emerged from building SwarmDo-A1. We publish them because the honest-evaluation gap in
agentic coding is real, and better shared than hoarded.*
