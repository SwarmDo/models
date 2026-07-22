# SwarmDo-A1

**An open, self-hostable, multimodal coding agent for real-world software engineering.**

> **Status: Release candidate.** Weights and download links are published here on final release.
> Everything below is measured, not marketing — including the parts that didn't work. Full numbers
> in [RESULTS.md](RESULTS.md); how we measure in [../docs/METHODOLOGY.md](../docs/METHODOLOGY.md).

---

## What it is

SwarmDo-A1 is not just a checkpoint — it's an **open, self-hostable coding *system*** with three parts,
each earning its place by a measured result:

1. **A strong, deliberately-chosen open base** — **Qwen3.6-27B**, a single-GPU-servable model with a
   hybrid **gated-DeltaNet linear-attention** architecture and native **vision**. Choosing and
   re-basing on it was, by a wide margin, the single biggest measured win we found (see Results).
2. **A hardened agentic harness that verifies by *running the code*** — it samples several candidate
   patches and selects among them using the project's own tests, not a model's opinion. This
   execution-grounded selection is where the system beats the raw base decisively.
3. **A differentiated capability the base + verifier unlock together: exec-grounded *visual coding*** —
   given an image (a chart, a UI, a diagram), the model writes code to reproduce it, and we score
   that code by *rendering it and comparing to the target*. An objective reward you can't bluff.

It is designed to be run agentically — given tools to read, search, and edit a repository, it works
a bug to a test-verified patch over many steps.

### Highlights

- **Open weights, one GPU.** Fits a single 80GB accelerator. Self-host it; no external API required.
- **Execution-verified, not vibes.** The system's advantage comes from selecting patches that *pass
  tests*, and every headline is a paired statistical test on a held-out set — no cherry-picked demos.
- **Multimodal, and it matters.** The vision path is preserved through fine-tuning, and it's
  **measurably load-bearing** on visual-coding tasks (not a silent passenger — we tested that).
- **Honest about its limits.** We publish the negatives too — including that single-model fine-tuning
  *alone* gave no significant gain. The value is the *system*, and we say so plainly.

Ships alongside **[SwarmDo](https://github.com/SwarmDo/swarmdo)** — the plugin that provides the swarm
orchestration, hooks, and agent-memory the system runs inside.

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
image as vision tokens and reasons over it alongside the text. For visual-coding (image → code), the
model returns runnable code you can execute and diff against the target.

### Serving notes (learned the hard way)

- Use `--tool-call-parser qwen3_xml` (**not** `hermes`) — the hermes parser silently drops this
  model's tool calls.
- Serve in eager mode if you hit hybrid-attention graph-capture issues; the model's recurrent
  kernels are sensitive to repeated in-place restarts — serve once per process.
- The base is linear-attention with a fixed-size recurrent state, so long agentic contexts cost
  *prefill tokens*, not growing KV-cache memory.
- **On-device / cheap-GPU:** the architecture is supported today by MLX (4-bit, ~16GB, vision kept),
  GGUF (llama.cpp/Ollama), and GPTQ-Int4 vLLM (single 24GB GPU).

## Intended use & limitations

**Intended:** autonomous and assisted repository-level bug fixing and code editing, and — as the
differentiating axis — **exec-grounded visual coding** (image → runnable code, screenshot debugging,
UI/diagram comprehension).

**Limitations:** A1 is specialized for Python-centric agentic SWE and evaluated primarily there.
Like all coding agents it can produce plausible-but-wrong patches; **always run the project's tests
against its output.** It is not a general chat assistant. See [RESULTS.md](RESULTS.md) for exactly
what is and isn't proven.

## License

Apache-2.0 (see repository [LICENSE](../LICENSE)), consistent with the base model family.

## Citation

```bibtex
@software{swarmdo_a1_2026,
  title  = {SwarmDo-A1: An open, execution-verified multimodal coding system},
  author = {SwarmDo},
  year   = {2026},
  url    = {https://github.com/SwarmDo/models}
}
```
