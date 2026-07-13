# SwarmDo-A1 — Results

We hold ourselves to a simple standard: **a result counts only if it survives a paired statistical
test on a held-out set the model has never seen.** This page reports what has cleared that bar and
what is still landing. (Methodology: [../docs/METHODOLOGY.md](../docs/METHODOLOGY.md).)

## The evaluation setup

- **Held-out set:** a contamination-controlled suite of real GitHub issues, each graded by the
  project's *own* `FAIL_TO_PASS` / `PASS_TO_PASS` unit tests (execution-verified, no LLM-as-judge).
- **Comparison:** **paired McNemar** (exact p-value) — each task is scored for both models and we
  count only the tasks where they *disagree*. This is far more sensitive than comparing two overall
  percentages, and it doesn't reward a model for getting easy tasks that both already solve.
- **Hygiene:** infrastructure losses (timeouts, serving errors) are excluded from *both* arms and
  the comparison is made on the paired intersection — so a delta reflects capability, not flakiness.

## Decision-grade result: the base matters more than the tricks

The single largest, most robust improvement came from **choosing a stronger open base and rebuilding
on it**. On the held-out set, moving to **Qwen3.6-27B** beat the previous 9B base by a wide,
statistically solid margin:

| Comparison | Result | Significance |
|---|---|---|
| Qwen3.6-27B vs prior 9B base | **≈ +25 percentage points** | McNemar exact **p = 0.013** (and p = 0.004 under a second, stricter analysis) |

The win is **robust** — it holds under two independent ways of handling infrastructure losses, and
the disagreement is lopsided in the 27B's favour (it solves many tasks the 9B never does, rarely the
reverse). The 9B base also destabilizes on long-horizon tasks (runaway generation, timeouts) where
the 27B runs clean.

**Takeaway:** for agentic coding, *the base model's raw capability dominates*. The disciplined move —
noticing a better open base now exists and having the rigor to re-baseline on it — beat every clever
scaffolding trick we tried.

## Honest negatives (published, not buried)

- **Test-time scaffolding is ceiling-bound.** Best-of-N sampling and better localization tools gave
  only small, non-significant gains — because selecting among a model's own attempts can never
  produce a solution the model couldn't generate in the first place. We proved this rather than
  claiming it.
- **Stronger "teacher" models don't transfer in-harness.** Several models that top public
  leaderboards did *not* beat our base when run in the same agentic harness — a caution against
  taking leaderboard numbers at face value.

## Landing with the release

The final fine-tuned SwarmDo-A1 is completing its head-to-head evaluation against its own base on
the full held-out set. **Whatever the outcome — a clear win, a modest gain, or a null result — the
paired number and its p-value are published here on release.** No result will be rounded up.

*Last updated: release candidate stage.*
