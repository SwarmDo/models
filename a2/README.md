# SwarmDo-A2

**One open model, both skills: execution-verified *coding* and render-verified *visual coding*.**

> **Status: Churn-gate cleared.** The combined adapter is on [Hugging Face](https://huggingface.co/SwarmDo/SwarmDo-A2).
> A2 takes SwarmDo-A1's coding delta and fuses it — on the *same* 27B base — with a separately trained
> visual-coding delta, into a single checkpoint. Everything below is measured, not marketing; full numbers in
> [RESULTS.md](RESULTS.md), methods in [../docs/METHODOLOGY.md](../docs/METHODOLOGY.md).

---

## What it is

[SwarmDo-A1](../a1/) is the **coding** tier — a strong open base plus a harness that verifies patches by *running
the project's tests*. **SwarmDo-A2** folds a second, independently trained skill into that same model:

- **Coding** (from A1) — repository-level bug fixing, selected by execution against the project's own tests.
- **Visual coding** — given a chart, plot, or UI image, it writes the code to reproduce it, scored by *rendering
  the output and comparing to the target image* (an objective, un-bluffable reward). This delta comes from a
  **teacher-free flywheel**: the model self-generates render-verified training data and measurably improves from it.

Both deltas are combined into **one rank-48 LoRA adapter on `Qwen/Qwen3.6-27B`**, so A2 is a *single model that
carries both skills* — not two adapters you swap between.

### The headline: combining two skills degraded neither (the "churn gate")

Folding two independently trained skills into one model risks each eroding the other. We measured both halves,
paired and held-out, and **neither degraded** — see [RESULTS.md](RESULTS.md):

- **Visual moat preserved** — combined A2 vs the pure visual model: Δ −0.006, 95% CI [−0.016, +0.004], **p = 0.63**
  (not worse); A2 still beats the raw base 14 : 5.
- **Coding preserved** — A2 vs A1 on held-out SWE tasks, **execution-grounded** (real `FAIL_TO_PASS`): 18 vs 19
  solved, **McNemar p = 1.0**, 39/40 verdict agreement.

### Highlights

- **One model, both skills.** Code *and* vision in a single checkpoint, with measured proof that combining them
  costs neither.
- **Open weights, one GPU.** A LoRA on the Apache-2.0 `Qwen/Qwen3.6-27B` base; serve it yourself.
- **Execution- and render-verified, not vibes.** Coding wins are re-derived by running real tests (never
  self-graded — see the methodology note in RESULTS); visual wins are scored by rendering and comparing.
- **Honest about limits.** The coding churn-gate is small (rules out gross regression, underpowered for subtle);
  we say so. Negatives published, not buried.

## Quickstart — serve **merged** (important)

A2's adapter reaches the base's gated-DeltaNet layers, which **JIT-hang when served as a vLLM LoRA**. Serve A2 as
**merged** weights. The VL-aware merge keeps the vision tower and emits the image-processor config vLLM needs:

```bash
# weights: https://huggingface.co/SwarmDo/SwarmDo-A2  (merge_lora_vl.py is in that repo)
# 1) merge the A2 adapter into the base, preserving the vision tower
python merge_lora_vl.py --base Qwen/Qwen3.6-27B --lora ./ --out ./swarmdo-a2-merged

# 2) serve the merged model (multimodal + tool-calling)
vllm serve ./swarmdo-a2-merged --served-model-name swarmdo-a2 \
  --enable-auto-tool-choice --tool-call-parser qwen3_xml --reasoning-parser qwen3 \
  --enforce-eager --trust-remote-code
```

Then call it as an OpenAI-compatible chat endpoint (`model = "swarmdo-a2"`); pass an image via the standard
`image_url` content block for visual-coding.

### Serving notes

- Use `--tool-call-parser qwen3_xml` **with** `--enable-auto-tool-choice` (the `hermes` parser silently drops
  tool calls; without auto-tool-choice, tool calls 400).
- Serve once per process (the recurrent kernels dislike in-place restarts); eager mode avoids hybrid-attention
  graph-capture issues.
- **No GGUF for A2:** quantizing to a text GGUF drops the vision tower, making the visual half inert. For code-only
  local runs use the [A1 GGUF](https://huggingface.co/SwarmDo/SwarmDo-A1-GGUF); for A2's code+vision, serve merged.

## Intended use & limitations

**Intended:** one model for both repository-level bug fixing *and* image-to-code (chart/plot/UI → runnable code),
self-hosted.

**Limitations:** Python-centric agentic SWE + Python visual coding. As a single model A2 is ~neutral vs the base on
raw coding (the same "churn wall" A1 documents) — its value is carrying both skills at once with no measured loss,
plus the execution-verified system around it. The SWE churn-gate is 40 paired tasks (1 discordant pair): it rules
out *gross* coding regression, not a *subtle* one. Like all coding agents it can produce plausible-but-wrong
patches — **always run the project's tests against its output.** See [RESULTS.md](RESULTS.md).

## License

Apache-2.0 (see repository [LICENSE](../LICENSE)), consistent with the base model family.

## Citation

```bibtex
@software{swarmdo_a2_2026,
  title  = {SwarmDo-A2: One open model for execution-verified coding and render-verified visual coding},
  author = {SwarmDo},
  year   = {2026},
  url    = {https://github.com/SwarmDo/models}
}
```
