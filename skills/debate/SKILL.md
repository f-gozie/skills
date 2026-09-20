---
name: debate
description: Pit independent AI agents against each other as adversarial reviewers on a hard architecture, design, or product decision, a debugging impasse, or when you need an unbiased second opinion or fresh ideas. By default one Claude subagent and one Codex CLI agent (a different model family), with a third debater on complex calls. They reason from the same neutral, facts-only brief, argue across rounds, and the main agent referees and writes the conclusion. Trigger when the user says "debate", "argue this", "two agents", "second opinion", "stress-test this decision", "which approach", "I'm stuck", "red-team this", or wants a confident, non-over-engineered call.
---

# Debate

Run independent reasoners against the same question and let them argue. The value comes from
**independence** and **no steering**: each forms its own view from the same facts, they rebut each
other, and you (the main agent) referee and synthesize. Use it when a decision matters and you want
confidence, not a coin flip.

## When to use

- **Architecture, design, or product decisions** with real trade-offs or several valid options.
- **Debugging impasses** — two independent root-cause diagnoses, then argue.
- **Stuck** — two different approaches proposed, then critiqued.
- **Sanity checks** on something that works but might be over-engineered or wrong.

Skip it for small or easily reversed choices. It costs several agent runs.

## The cast

**Default: two debaters from different model families.**

- **Claude** — via the `Agent` tool (`subagent_type: "general-purpose"`). Pin a strong model
  (for example `model: "opus"`).
- **Codex** — the external Codex CLI, a different model lineage. Check it exists: `which codex`.
  Run it **read-only**. It uses the model in `~/.codex/config.toml`; for debates, prefer a strong
  model at high reasoning effort, pinned if needed:
  `codex exec -m <model> -c model_reasoning_effort="high" ...`

**Add a third debater on complex calls** (another `Agent` run on a different strong model, for
example `model: "fable"`) when:

- the decision is a **one-way door** — data migrations, public API shape, pricing, a vendor
  commitment;
- there are **three or more live options**;
- the call spans **several surfaces at once** (backend, web, mobile) or leans on product judgment;
- round 1 comes back **split with both sides well argued** — add the third rather than
  tie-breaking a 1–1.

Two is the default. Say which you chose and why in the synthesis.

**If Codex is not installed:** use two `Agent` runs on *different* models. Last resort: two agents on
the same model with different default stances (one "ship the simplest thing", one "find what
breaks"). Independence and adversarial framing matter more than the exact models.

## The rule: neutrality

The whole method fails if you bias the debaters.

- **One brief, byte-identical for every debater.** No extra hints for anyone.
- **No steering.** Do not say which option you or the user prefer. Present options even-handedly.
- **Facts, not opinions.** Real constraints, the actual question, file paths to read, and the genuine
  options.
- **Set a quality bar, not an answer.** For example: "be decisive, avoid over-engineering, flag
  assumptions, optimize for X". This shapes rigor, not the conclusion.

## Protocol

**Phase 0 — Gather facts and size the cast.** Read the relevant code and context so the brief is
correct. Find the real question and the genuine options. Decide two or three debaters now, from the
facts, before seeing anyone's answer.

**Phase 1 — Write the neutral brief** to a temp file (for example `/tmp/<topic>-brief.md`):

- *Context*: what the system is and what matters (speed, simplicity, cost, ...).
- *The question*: plain and neutral.
- *The options*: even-handed. For debugging: symptoms and what is already ruled out. For ideas: the
  goal and constraints, with no preferred direction.
- *Ground truth*: exact file paths to read.
- *The ask*: verdict, critique, concrete pitfalls, a decisive recommendation.
- *Quality bar*: opinionated, concrete, decisive, anti-over-engineering, assumptions flagged.

**Phase 2 — Round 1, independent, in parallel.** In one message, start every debater:

- Claude: `Agent(subagent_type: "general-purpose", model: "opus", prompt: <brief> + "Read these files for ground truth. Do not edit anything.")`
- Codex: `codex exec --cd <repo-abs-path> --sandbox read-only < /tmp/<topic>-brief.md`
- Third debater, if used: the same `Agent` call on another model.

**Phase 3 — Map the field.** List where they agree and where they disagree. Agreement between
independent debaters means high confidence. Disagreements become the round-2 agenda. With three
debaters, a 2–1 split is not settled; a lone dissent with the better argument can still win.

If you started with two and round 1 split with strong arguments on both sides, add the third now. Give
it the **round-1 brief**, not the rebuttal, so its first view stays independent.

**Phase 4 — Round 2, rebuttal.** Write a round-2 brief: the agreements ("do not relitigate") and each
open disagreement with every side stated fairly and **without naming which model said it**. Send it
to all debaters in fresh runs with full context. Each must argue the strongest case, then commit to
one answer per disagreement.

**Phase 5 — Synthesize.** Write the conclusion for the user:

- The cast (two or three) and the one-line reason.
- What they agreed on, and why that is high confidence.
- How each disagreement resolved — who conceded, where they met.
- Anything still split: **you break the tie, with explicit reasoning** tied to the project's values.
  Do not average.
- A clear, concrete recommendation.

**Phase 6 — Act (optional).** Implement, or record the decision (an ADR or decision log) so the
reasoning survives.

## Discipline

- **Two rounds is usually enough.** Run a third only if a real disagreement is still live and
  solvable.
- **Every debater stays read-only.** Codex: `--sandbox read-only`. Claude agents: "do not edit".
- **Never more than three debaters.** Past that you are averaging, not refereeing.
- **Force decisions.** Every round ends with each debater committing. Reject "it depends".
- **Show disagreement honestly.** Present a real split and your tie-break; do not hide it.
- **Report the conclusion, not the transcripts.** Quote a line only when it carries the argument.
- **You are the referee, not a debater.** Stay neutral until the synthesis, then own the call.

## Reference invocations

```bash
which codex
codex exec --cd /abs/repo --sandbox read-only < /tmp/topic-brief.md
```

```
Agent(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "<the identical brief>\n\nRead <files> for ground truth. Do not edit anything. Be decisive; end with a clear recommendation."
)
```

Start the Codex run (Bash) and the Claude agents in the **same message** so they run at the same time.
