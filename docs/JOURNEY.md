# The SwarmDo Journey

*How we build open coding models — the public story. For the reusable methods, see
[METHODOLOGY.md](METHODOLOGY.md).*

---

## The mission

Build an open-source AI that fixes bugs in real software — hand it a bug report and a whole
codebase, and it finds the problem and writes a fix that passes the project's own tests, on its own.

Plenty of such models exist. Our goal is narrower and harder: take a strong open base and make it
**measurably, honestly better** — and prove the improvement with the rigor a scientist would demand,
not with a flattering demo.

That discipline — *honestly* — turned out to be the through-line of the whole project.

## Chapter 1 — The uncomfortable discovery

Early on, our numbers looked good. Then we looked harder at *how* we were grading ourselves and
found the evaluation was contaminated: some "problems" the model solved had effectively leaked their
answers, and some runs were quietly losing tasks to timeouts and miscounting them. The impressive
numbers were partly an illusion.

Most projects move on from that quietly. We did the opposite — threw out the flawed scoreboard and
rebuilt it: a clean, sealed set of problems the model had never seen, graded by the projects' real
unit tests, and a **paired statistical test** that compares two models problem-by-problem and tells
you honestly whether a difference is real or just luck. Everything since is measured on that
scoreboard.

## Chapter 2 — The big win was a better foundation

The single largest improvement didn't come from a clever trick. It came from **changing the base.**
We rebuilt on a stronger, newer open model (27B parameters, single-GPU servable) — and on the honest
scoreboard the jump was large and statistically solid: roughly **+25 percentage points**, the kind
of result that survives scrutiny.

The lesson worth keeping: sometimes the best move isn't a new invention — it's noticing that a better
starting point now exists, and having the discipline to rebuild on it.

## Chapter 3 — The ceiling that scaffolding couldn't break

Next we tried the obvious things: let the model take several attempts and pick the best; give it
better tools to locate bugs. These helped a little — but we showed, with the math, that they **can't
break through a ceiling.** Picking the best of several tries can only ever choose from what the model
was already capable of producing. If it can't solve a problem in *any* attempt, no amount of clever
selecting conjures a solution.

So we packaged that honestly: the re-base win is real and significant; the scaffolding tricks are
promising but too small to claim. Then we turned to the only thing that can actually raise the
ceiling — **making the model genuinely smarter through training.**

## Chapter 4 — Teaching the model from its own near-misses

Take the problems the model *fails*. For each, we know the correct fix (these bugs were mined from
real merged fixes). We write a tiny, vague hint — one sentence, pointing at the concept, never
giving the answer — and let the model try again. A surprising share of the time, that gentle nudge
is enough to solve a problem it had completely failed.

Then the key step: we **remove the hint** and train the model on its own successful attempt, so it
learns to reach that solution *without* the hint next time. The model effectively teaches itself,
using its past failures plus a whisper of guidance — no smarter "teacher" model required.

Two traps we caught along the way (both documented in [METHODOLOGY.md](METHODOLOGY.md)):

- **A hinted model talks about the hint** while solving. Train on that and you accidentally teach it
  to rely on a hint that won't exist at test time — a silent poison. We measured it, and built a
  filter to strip it out.
- **A subtle training bug was freezing most of the layers that matter** for this architecture's
  reasoning — training only a fraction of the model. We found it, verified it against the actual
  model internals, and fixed it.

## Chapter 5 — A research swarm, and the value of ruling things out

Alongside the hands-on work, we run swarms of AI research agents — many at a time — combing the
scientific literature and the far corners of the web for ideas that might raise the ceiling. They
argue against each other, and an adversarial pass kills the ideas that sound good but don't hold up.

They've produced concrete wins (one found the layer-freezing bug above). They've also done something
rarer: **honestly ruled things out.** Repeated searches for a "magic formula from an unrelated field"
came back with a confident null — the space is mostly seductive analogies, and the ideas that survive
are useful *measuring tools*, not breakthroughs. Knowing where the treasure *isn't* saves everyone
who comes after from digging there.

## The theme: honesty as a competitive advantage

The tempting version of this project overclaims — polishes the demo, quotes the flattering number,
buries the caveats. We keep choosing the harder version: clean scoreboards, paired statistics,
killing our own pretty ideas when the evidence says to, and reporting a modest or null result as
faithfully as a win.

That discipline is *why* the one big result is trustworthy — and it's why, whatever any given
experiment says, we can tell you exactly what we learned and exactly how much to believe it.
