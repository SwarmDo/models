# SwarmDo-A1 — Results

We hold ourselves to a simple standard: **a result counts only if it survives a paired statistical
test on a held-out set the model has never seen** — or, for visual work, an *executable* one (render
the code, compare to the target). This page reports what has cleared that bar and what is still
landing. (Methodology: [../docs/METHODOLOGY.md](../docs/METHODOLOGY.md).)

## How we measure

- **Held-out set:** a contamination-controlled suite of real GitHub issues, each graded by the
  project's *own* `FAIL_TO_PASS` / `PASS_TO_PASS` unit tests (execution-verified, no LLM-as-judge).
- **Comparison:** **paired McNemar** (exact p-value) — each task is scored for both models and we
  count only the tasks where they *disagree*. Far more sensitive than comparing two overall
  percentages, and it doesn't reward a model for easy tasks both already solve.
- **Hygiene:** infrastructure losses (timeouts, serving errors) are excluded from *both* arms and the
  comparison is made on the paired intersection — so a delta reflects capability, not flakiness. (We
  hold this line strictly: one visual result below was nearly mis-reported because 4 eval *timeouts*
  looked like model failures — we caught it and corrected it. Discipline over a good-looking number.)

## Decision-grade result 1: the base matters more than the tricks

The single largest, most robust improvement came from **choosing a stronger open base and rebuilding
on it**. On the held-out set, moving to **Qwen3.6-27B** beat the previous 9B base by a wide,
statistically solid margin:

| Comparison | Result | Significance |
|---|---|---|
| Qwen3.6-27B vs prior 9B base | **≈ +25 percentage points** | McNemar exact **p = 0.013** (p = 0.004 under a stricter analysis) |

Robust under two independent ways of handling infrastructure losses, and the disagreement is lopsided
in the 27B's favour. **Takeaway:** for agentic coding, *the base model's raw capability dominates* —
the disciplined move (spotting a better open base and re-baselining on it) beat every clever
scaffolding trick we tried.

## Decision-grade result 2: verify by running the tests

The system's own contribution over the raw base is **execution-grounded selection** — sampling
several candidate patches and keeping the ones that *pass the tests*:

| Comparison | Result | Significance |
|---|---|---|
| Base + exec-verified selection vs base alone | **36 vs 24 tasks solved** (+50%) | McNemar **p = 0.0005**, regression-free |

This is where "run the code, don't trust the vibes" pays off — and it's the honest description of what
A1 adds on top of a strong base.

## Decision-grade result 3: an exec-grounded visual-coding capability

A1 keeps its base's **vision** and pairs it with the same execution-verification idea, on a new axis:
**image → code**. Give it a chart; it writes matplotlib code; we *render that code and score it against
the original image*. On a 150-image chart-reproduction benchmark:

| Test | Result | Significance |
|---|---|---|
| Does the image actually drive the output? (image-on vs image-off) | image-on solves **15** tasks image-off solves **0** | McNemar **p = 6e-05** |
| Continuous render-reward lift from seeing the image | **+0.088** | 95% CI [0.078, 0.097] — excludes zero |

The vision path is **measurably load-bearing**, not decoration. And the model **fuels its own
improvement loop**: from 150 charts it self-generates render-verified-good code on **63%** at a strict
quality bar — with *no external teacher* — the raw material for a self-improving flywheel.

## Honest negatives (published, not buried)

- **Single-model fine-tuning, alone, gave no significant gain.** Fine-tuning one checkpoint trades a
  small quality gain against a robustness cost (a "churn wall" we see on both the coding and visual
  axes). This is *why* A1 is framed as a **system** (strong base + execution-verified selection),
  not a magic checkpoint. We measured this rather than hiding it.
- **Test-time scaffolding is ceiling-bound.** Best-of-N and better localization tools alone gave
  small, non-significant gains — selecting among a model's own attempts can't produce a solution it
  couldn't generate. We proved it rather than claiming it.
- **Stronger "teacher" models don't transfer in-harness.** Several leaderboard-topping models did
  *not* beat our base in the same agentic harness — a caution against face-value leaderboard numbers.
- **The visual flywheel *builds and trains* but doesn't yet *compound*.** One small round of self-fuel
  training is roughly neutral on unseen charts (marginally positive when it doesn't time out). Making
  it compound needs more fuel and iteration — an open, honest next step, not a claimed win.

## Landing with the release

Weights, download links, and any final head-to-head numbers are published here on release. **Whatever
the outcome — a clear win, a modest gain, or a null result — the paired number and its p-value are
published here. No result is rounded up.**

*Last updated: release candidate stage (2026-07).*
