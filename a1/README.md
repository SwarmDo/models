# SwarmDo-A1

**An open, self-hostable, multimodal coding agent for real-world software engineering.**

> **Status: Release candidate.** The final head-to-head evaluation is completing; download links
> and the confirmed headline number are published here the moment it lands. The methodology and
> the one decision-grade result behind A1 are documented below and in [RESULTS.md](RESULTS.md).

---

## What it is

SwarmDo-A1 is a coding agent built on **Qwen3.6-27B** — an open, single-GPU-servable model with a
hybrid **gated-DeltaNet linear-attention** architecture and native **vision** support. On top of
that base we apply a targeted fine-tune (a low-rank adapter that reaches the architecture's
sequence-mixing layers, not just the standard attention projections) aimed squarely at the failure
mode that bounds agentic coding: **producing the *right fix*, not just finding the right file.**

It is designed to be run agentically — given tools to read, search, and edit a repository, it works
a bug to a verified patch over many steps.

### Highlights

- **Open weights, one GPU.** Fits a single 80GB accelerator. Self-host it; no external API required.
- **Multimodal.** Debug from a screenshot of code or a broken UI, reproduce a chart, read a diagram —
  the vision path is preserved through fine-tuning.
- **Honestly evaluated.** Improvements are validated with a paired statistical test on a
  contamination-controlled held-out set, not a cherry-picked demo.

## Quickstart

SwarmDo-A1 serves through [vLLM](https://github.com/vllm-project/vllm) as an OpenAI-compatible
endpoint.

> Model weights: **published on Hugging Face with the release** (link added here on launch).
> Until then, the recipe below documents exactly how A1 is served.

```bash
# 1. Serve the model (single 80GB GPU)
vllm serve SwarmDo/SwarmDo-A1 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_xml \
  --reasoning-parser qwen3

# 2. Call it like any OpenAI chat endpoint
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "SwarmDo/SwarmDo-A1",
    "messages": [{"role": "user", "content": "Fix the off-by-one in average(): return sum(xs)/(len(xs)+1)"}]
  }'
```

**Multimodal use** — pass an image the standard way (`image_url` content block); the model reads the
image as vision tokens and reasons over it alongside the text.

### Serving notes (learned the hard way)

- Use `--tool-call-parser qwen3_xml` (**not** `hermes`) — the hermes parser silently drops this
  model's tool calls.
- Serve in eager mode if you hit hybrid-attention graph-capture issues; the model's recurrent
  kernels are sensitive to repeated in-place restarts — serve once per process.
- The base is linear-attention with a fixed-size recurrent state, so long agentic contexts cost
  *prefill tokens*, not growing KV-cache memory.

## Intended use & limitations

**Intended:** autonomous and assisted repository-level bug fixing, code editing, and — uniquely —
visual coding tasks (screenshot debugging, UI/diagram comprehension).

**Limitations:** A1 is specialized for Python-centric agentic SWE and evaluated primarily there.
Like all coding agents it can produce plausible-but-wrong patches; **always run the project's tests
against its output.** It is not a general chat assistant.

## License

Apache-2.0 (see repository [LICENSE](../LICENSE)), consistent with the base model family.

## Citation

```bibtex
@software{swarmdo_a1_2026,
  title  = {SwarmDo-A1: An open multimodal coding agent},
  author = {SwarmDo},
  year   = {2026},
  url    = {https://github.com/SwarmDo/models}
}
```
