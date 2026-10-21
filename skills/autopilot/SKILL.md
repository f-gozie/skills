---
name: autopilot
description: Hands-off delivery. Takes a plan or a request to a reviewed, verified pull request without the person — plans, builds, tests, reviews, verifies in the browser or simulator with screenshots, opens the PR, fixes CI, and ends with a short report. Decides alone, debates low-confidence calls, and parks anything at a hard stop for the person. Use when the user says "autopilot", "/autopilot", "take this all the way", "ship this while I'm away", or "I'll be back later, get it to a PR". Not for work the person wants to steer step by step (use plans), and never for deploys or releases.
argument-hint: "<plan-path | \"request\"> [--merge]"
---

# Autopilot

Run a feature from plan to finished pull request while the person is away. Stop only at the hard
stops below. Decide everything else here, and write each decision down so the person can check it
later.

This skill drives the others: `plans` for the plan folder, `review-all` for the review, `debate`
for hard calls, `mockup` for new screens, and `checkpoint` if the run must stop early.

## Input

- A plan path, or a request in quotes. With a request, write the plan first (Phase 0).
- `--merge`: squash-merge the PR when CI is green and nothing blocks it. Without it, stop at the PR.

## What running this approves

Running autopilot is the person's approval, **on this run's own branch only**, for:

- commits, pushes of that branch, opening and updating its PR, and CI fixes on it;
- a worktree, a local test stack, a simulator or browser for verification;
- with `--merge` only: one squash merge of that PR after CI is green.

It approves nothing in the hard stops.

## Hard stops

Never pass these alone:

- Deploys, releases, app store or over-the-air updates, and merges without `--merge`.
- Pushes to the default branch or to a branch that another person or session owns.
- Production data, production config, secrets, infrastructure.
- Migrations that delete or rewrite user data; pricing; removing a public API.
- Real spend beyond a few test calls (paid APIs, AI evals).
- Anything the repo's `AGENTS.md` / `CLAUDE.md` forbids.

At a hard stop, add the item to the plan's **Waiting on you** list, keep going with everything that
does not depend on it, and put it first in the report.

## Run settings

- Work on a new branch (`feat/<slug>` or `fix/<slug>`) from the latest default branch. If the
  main checkout has other changes or other sessions use it, work in a git worktree.
- If the host can keep the machine awake, ask it to for the run.
- Check free disk before large builds. Remove only what this run created.

---

## Phase 0 — Intake

1. Read the plan, or write one with the `plans` skill. The person is away, so do not stop for
   approval: the request is the approval. Add the registry row as `active`.
2. Add two sections to the plan if they are missing:
   - **Decisions made alone** — a table: decision · options · chosen · confidence (high / medium /
     low) · how (alone / debate) · reversible?
   - **Waiting on you** — hard stops, delegated designs, checks only a person can do.
3. Read `decisions.md` and the repo rules. Do not reopen a recorded decision.

## Phase 1 — Design gate

For every visible change (screen, component, email, page):

- An approved design covers it → build exactly that.
- No approved design → run the `mockup` skill. Pick the option that its own criteria favour, mark
  the record **"Decided under delegation — needs confirmation"**, add it to **Waiting on you**, and
  continue.

## Phase 2 — Contracts

Write the API, type and event shapes into the plan before code, so parts can be built in parallel
against fixed shapes. A shape that is hard to undo (stored data, public API) needs high confidence
or a debate.

## Phase 3 — Build

- Follow the plan in order. If the host has subagents, give independent parts to them in
  parallel, each with a self-contained brief: plan path, contract, files, tests to add, what not to
  touch.
- Tests for all new logic, and a failing-first test for every bug fix. No scope creep.
- After each phase: run that area's tests, tick the plan, add a progress line.
- A failure that survives two fix attempts goes to a debate. If the debate does not solve it,
  park it in **Waiting on you** and continue.

## Phase 4 — Review

1. Run `review-all branch`.
2. Fix every confirmed critical and warning finding. A finding the two model families disagree on
   goes to a debate.
3. Run the review once more on the fixes. Keep what was found and fixed for the PR body.

## Phase 5 — Verify

Read the repo's **How to verify** section in `AGENTS.md`: how to start the app, how to seed test
data, and what counts as a pass. Never guess these commands.

1. Write `scenarios.md` in the plan folder: one scenario for each user-visible behaviour and each
   state of each new screen (empty, loading, error, success, light and dark, small and large
   screens), with its pass criteria.
2. Start the app. Drive it with the browser or simulator tools the host has.
3. For each scenario: set up the data, act in the UI, take a screenshot, and check the result
   where it is stored (database or API), not only on screen. Look at every screenshot yourself.
4. A failed scenario is a bug: fix it, re-run that scenario, and say in the report what
   verification found.
5. Write `verify-report.md` in the plan folder: scenario · result · evidence · screenshot. Save
   screenshots in `<plan-folder>/screenshots/`. Delete screenshots of layouts you replaced.
6. Checks the tools cannot do (a real device, sound, haptics, real push, speed on slow hardware) go
   to **Waiting on you**.
7. Stop what you started.

**If the repo has no How to verify section** and you cannot find a safe way to run the app, run the
tests only. Put the visual checks in **Waiting on you**, and suggest a How to verify section in the
report.

## Phase 6 — Ship

1. Fill in the plan's **Outcome**. Keep the registry row `active` until the person's checks pass.
2. Make logical commits: one concern each, in the repo's commit style. Check each commit after the
   pre-commit hooks run.
3. Push the branch and open the PR. The body has:
   - **Summary**;
   - **Needs your confirmation**: delegated designs, and decisions below high confidence;
   - **Screenshots**: a table per area, with images linked from the branch;
   - what changed, what the review found and fixed, the test plan with real counts, and how it
     rolls out.
4. Wait for CI (`gh pr checks <n> --watch`). Fix failures on the branch, push, repeat.
5. With `--merge`, and only when CI is green and nothing in **Waiting on you** blocks the merge:
   squash-merge. Never deploy or release.

## Phase 7 — Report

Write the report into the plan and send it as the final message:

```markdown
**<feature>: PR <link> <ready for review | merged>.**

**Waiting on you** (first): hard stops, delegated designs, checks for a person.
**Done:** what ships, in user terms.
**Decided alone:** each decision, its confidence, and whether it was debated.
**Verified:** N scenarios pass (link verify-report.md); what verification caught and fixed.
**Tests and CI:** counts per suite; CI state.
**Not done:** anything parked, and why.
```

If the host can send files, send the key screenshots.

---

## When to debate without asking

Run the `debate` skill when:

- confidence in a decision is below high and the plan does not settle it;
- the two model families disagree on a review finding;
- two fixes for the same failure did not work;
- a design option is a close call under delegation.

Give the debaters facts only. Record the outcome in **Decisions made alone** with "debate". A debate
that stays split on a choice that is hard to undo becomes a hard stop.

## When things fail

- Never end the run on one blocker. Park it, finish the rest, report it first.
- Never guess infrastructure commands or model names. Look them up in the repo or in earlier runs.
- If a tool or service is down (the other model's CLI, the simulator, Docker), do less and say so:
  one model family for review, tests only for verification.
- If the run must stop early (disk, auth, a hard stop that blocks everything), keep the plan
  current, push what is committed, run `checkpoint`, and report where it stands.
