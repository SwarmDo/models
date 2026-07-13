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
- **Multimodal.** SwarmDo-A1 can debug from a *screenshot* of code or a broken UI, not just text —
  an axis most coding LLMs can't touch.
- **Measured, not demoed.** Every improvement claim is backed by a paired statistical test on a
  contamination-controlled held-out set. When a result is modest or null, we say so. See
  [docs/METHODOLOGY.md](docs/METHODOLOGY.md).

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
