# The workflow

How these skills fit together to take a feature from idea to merged PR with a coding agent.
Read this first. The skills do the steps; this page explains why the steps exist.

## The loop

```text
plan → approve → build → review → PR → close
                   ↑                 |
                   └── checkpoint ───┘   (only if the work stops before it ships)
```

1. **Plan.** The agent writes a plan in the repo: the problem, what is in and out of scope,
   checkable success criteria, and the approach. (`plans`)
2. **Approve.** You read the plan and say yes, or change it. No code before this. For a hard
   design choice, let two models argue it first. (`debate`) For a new screen, approve a mockup
   first. (`mockup`)
3. **Build.** The agent follows the plan and logs progress in it as each step lands. If the plan
   turns out wrong, it stops and asks.
4. **Review.** Four reviewers: code against the repo's rules, security, a simplifier, and a model
   from a different family. Every finding is checked against the real code. (`review-all`)
5. **PR.** You approve every commit and push.
6. **Close.** The plan gets an Outcome section and is marked done. (`plans`)

## Hands-off

`autopilot` runs the whole loop without you. You approve once, at the start, and it plans,
builds, reviews, verifies in the running app with screenshots, opens the PR and fixes CI. It
decides alone, debates the calls it is not sure of, and parks anything at a hard stop (deploys,
production, secrets, data deletion) in a **Waiting on you** list at the top of its report. It
reads how to run and check the app from the **How to verify** section of your `AGENTS.md`.

## What lives where

```text
context/
  decisions.md        # why things are the way they are; append-only
  standards/          # optional: conventions per area
  plans/
    README.md         # the registry: every plan and its status
    2026-10-21-<slug>/
      plan.md         # the one living document for this work
      reviews/        # review reports
      handoffs/       # only if the work spans sessions
```

- **The plan is the source of truth, not the chat.** Chats get compacted and lost. A new session,
  a teammate or a reviewer reads the plan folder and sees the whole story: plan, reviews, outcome.
- **Status lives only in the registry.** Older setups put `[WIP]` or `[DONE]` in filenames.
  Nobody renames files reliably, so those folders fill with stale "work in progress".
- **Decisions are written once,** with the reason and the rejected options. This stops the agent
  (and you) from reopening a settled question in every session.

## One session or many?

**One session per feature is the default.** Current strong models can plan, build, review and
ship a normal feature in one session. Then there is no handoff; the plan's Outcome section is the
record.

**Hand off only when work stops before it ships:** the context is nearly full, the day ends, or
something urgent comes in. `checkpoint` updates the plan and writes one handoff file. The next
session reads it and continues with no re-explaining.

**Use phases on purpose with weaker or cheaper models, or very large features.** Give each
session one phase of the plan, and checkpoint between phases. A model that loses track of long
work does well with a short, clear contract.

## The rules

These go in your repo's `AGENTS.md` (the `plans` skill adds them):

1. Never commit, push or deploy without a clear yes.
2. Do exactly what was asked. Other ideas go in the plan's follow-ups.
3. Check state before risky commands.
4. Run the tests before calling work done. Report failures as they are.
5. When a choice changes behaviour and the answer is not clear, ask.
6. Keep personal data out of plans, commits and logs.

## What is not here

**Mindmaps or file indexes.** A hand-kept map of the codebase made sense when agents were slow at
finding things. Now an agent searches the repo in seconds, and a hand-kept index goes out of date
and costs a commit in every change. Keep only what the code cannot tell an agent: the reasons
(`decisions.md`), the conventions (`standards/`), and an optional one-page `architecture.md`.

## Start in a repo

1. Install the skills (see the README).
2. In the repo, ask the agent: "set up plans in this repo".
3. For the next feature: "plan <the feature>".
