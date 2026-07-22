<div align="center">

# SwarmDo Models

**Open, self-hostable coding agents — built in public, measured honestly.**

[Website](https://swarmdo.com) · [Model Cards](#released-models) · [Methodology](docs/METHODOLOGY.md) · [The Journey](docs/JOURNEY.md)

</div>

---

SwarmDo builds open large language models for **agentic software engineering** — models that
take a real bug report and a whole codebase and produce a fix that passes the project's own
tests, autonomously.

This repository is the **public home** for those models: model cards, download links, reproduce
guides, and the findings behind them. It is intentionally curated — the day-to-day research
mess lives elsewhere; what you'll find here is what's stable enough to build on.

## Released models

| Model | Base | Size | Modality | Status | Card |
|-------|------|------|----------|--------|------|
| **SwarmDo-A1** | Qwen3.6-27B | 27B | Text + Vision | Release candidate | [a1/](a1/) |

*Future models (A2, B1, …) will be added as sibling directories with their own cards.*

## What makes these models different

- **Genuinely open & self-hostable.** Built on open, Apache-2.0-licensed base weights that fit a
  single 80GB GPU. No API lock-in — you can serve the model yourself.
- **Execution-verified, not vibes.** The systems earn their gains by *running the code* — selecting
  patches that pass the project's own tests. On the held-out set this beats the raw base decisively
  (+50% more issues solved, paired McNemar p = 0.0005).
- **Multimodal — and it's load-bearing.** SwarmDo-A1 can debug from a *screenshot* and reproduce a
  chart as runnable code, scored by *rendering it and comparing to the target* — an objective,
  un-bluffable reward. We tested that the image genuinely drives the output (p = 6e-05), and the
  model self-generates its own render-verified training data. An axis most coding LLMs can't touch.
- **Measured, not demoed — negatives included.** Every claim is a paired statistical test on a
  contamination-controlled held-out set; when a result is modest, null, or we *nearly* mis-read one,
  we say so. See [docs/METHODOLOGY.md](docs/METHODOLOGY.md).

## Repository layout

```
a1/                 SwarmDo-A1 — model card, quickstart, results
docs/
  JOURNEY.md        How these models were built (the public story)
  METHODOLOGY.md    Reusable methods: decision-grade eval, hindsight-hint distillation, union-ceiling
LICENSE             Apache-2.0
```

## License

Code and model weights released under **Apache-2.0** (see [LICENSE](LICENSE)), matching the base
model family. Documentation is free to quote with attribution.

---

<div align="center">
<sub>Part of the <a href="https://swarmdo.com">SwarmDo</a> project.</sub>
</div>
