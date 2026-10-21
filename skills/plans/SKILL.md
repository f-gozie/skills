---
name: plans
description: A plan-first way to build features with a coding agent. One folder per piece of work holds its plan, review reports and session handoffs; a registry shows what is active; a decision log keeps the why. Use to set it up in a repo ("set up plans here", "add plans to this repo"), to start a feature ("plan this", "new plan", "let's build X"), to pick up work ("where are we", "resume", "continue the plan"), to record a decision ("log this decision", "we decided X"), and to close a plan when it ships ("close the plan", "mark it done"). Not for one-line fixes, questions, or work with no plan worth keeping.
---

# Plans

A small, plain-file system that keeps a feature's thinking next to its code, so any session (or
any person) can see what is being built, why, and where it stands.

The ideas:

- **The plan is the source of truth, not the chat.** Chats get compacted, cleared and lost. The
  plan file survives all of that.
- **One folder per piece of work.** Plan, reviews and handoffs live together, so a reader follows
  plan → review → outcome in one place.
- **Status lives in one place:** the registry table. Never in filenames.
- **Decisions are written down once,** with the reason and the rejected options, and are not
  reopened without a new reason.
- **A person approves the plan before code is written,** and approves every commit, push and
  deploy.

## Layout

```text
<root>/                         # context/ by default; see "Find the root"
  decisions.md                  # append-only decision log (D1, D2, ...)
  architecture.md               # optional: system shape, in one page
  standards/                    # optional: conventions per area (api.md, testing.md, ...)
  plans/
    README.md                   # the registry: every plan, its status, one line
    YYYY-MM-DD-<slug>/
      plan.md                   # the one living document for this work
      reviews/                  # one report per review run: YYYY-MM-DD-<topic>.md
      handoffs/                 # only if the work spans sessions: YYYY-MM-DD-NN-<slug>.md
      tasks/                    # only if the plan ships as several PRs: one how-to per PR
      notes/                    # optional: research and option studies behind a decision
```

Only `plan.md` is required in a plan folder. Create the other folders when something goes in them.

Templates are in `references/`: `plan-template.md`, `registry-template.md`,
`decisions-template.md`, `agents-section.md`.

## Find the root

1. If the repo's `AGENTS.md` or `CLAUDE.md` names a plans location, use it.
2. Else, if `context/plans/`, `docs/plans/` or `plans/` exists, use its parent.
3. Else, use `context/`.

Say which root you use the first time.

## Does this need a plan?

Make a plan folder when the work does one of these:

- touches more than one area (for example API and UI);
- changes a data model, a public contract or a migration;
- has a design choice a reviewer could disagree with;
- will probably take more than one sitting.

Do not make one for a bug fix with a clear cause, a copy change, a dependency bump or a small UI
tweak. Do the work and say why no plan was needed.

## Session shape

**Default: one session per feature.** A strong model can plan, build, review and ship a feature
in one session. In that case there is no handoff. The plan's **Outcome** section, written when
the plan closes, is the record.

**Write a handoff only when work stops before it ships:** the context is nearly full, the day
ends, or you switch to something else. Use the `checkpoint` skill for this.

**Split into phases on purpose** when the feature is large or the model is weaker. Weaker models
lose track of long work, so give them less of it: one phase per session, with a checkpoint
between phases. The plan's phase list is the contract between sessions.

The plan file also protects a single session. When the harness compacts the chat, the agent
re-reads `plan.md` and continues from the progress log, not from memory.

## Actions

### Set up (once per repo)

1. Find the root. Create `<root>/plans/README.md` from `references/registry-template.md` and
   `<root>/decisions.md` from `references/decisions-template.md`. Do not overwrite existing files.
2. Add the section in `references/agents-section.md` to `AGENTS.md`, with the real root. Create
   `AGENTS.md` if there is none. Claude Code reads `CLAUDE.md` and Codex reads `AGENTS.md`, so
   make `CLAUDE.md` import the other: add `@AGENTS.md` at its top, or create a `CLAUDE.md` that
   holds only that line.
3. Do not invent `architecture.md`, `standards/` or the **How to verify** section. Offer them,
   and write them only from what the code and the person say. Fill in How to verify only from
   commands that exist in the repo (scripts, `package.json`, `Makefile`, CI config); otherwise
   leave its placeholders.
4. If the repo has an older layout (flat plan files with a status in the name, a separate
   reviews folder, mindmaps), report what you found. Do not move or delete anything without
   approval. Moving old files is optional; new work can use the new layout from today.

### Start a plan

1. Read `decisions.md` and any `standards/` file for the area. Read the code you will change.
2. Create `plans/YYYY-MM-DD-<slug>/plan.md` from `references/plan-template.md`. Fill in the
   problem, scope, out of scope, success criteria and approach.
3. **Success criteria must be checkable:** a test, a command or something a person can see.
   "Make it work" is not a criterion. "A test for an empty list fails before the change and
   passes after it" is.
4. Add a row to the registry with status `active`.
5. **Stop and ask for approval of the plan.** Ask questions only where the answer changes what
   gets built. Use the `debate` skill if a design choice is hard and the person agrees. Use the
   `mockup` skill if the work adds or changes a screen.
6. After approval, record any decisions in `decisions.md` and link them from the plan.

### Build

- Work through the plan in order. Tick items and add a line to **Progress** when a step lands,
  not at the end of the session.
- If the build shows that the plan is wrong, stop. Update the plan, say what changed and why,
  and ask before you go on. Do not drift from the plan without a note.
- Do only what the plan says. Put new ideas in **Follow-ups**.
- Before a PR, run the `review-all` skill. It writes its report into this plan's `reviews/`.

### Resume

1. Read the registry. Find the `active` plan. If there are more than one, ask which.
2. Read `plan.md`, then the newest file in `handoffs/` if there is one, then the decisions the
   plan links.
3. Check `git status`, the branch and any open PR. Compare them with what the plan says.
4. Report in a few lines: where the work stands, what is next, and anything that does not
   match. Then continue.

### Record a decision

Append to `decisions.md`: the next number, a title, the decision, **why**, and the **rejected**
options with one line each. Never rewrite an old entry. If a decision changes, add a new entry
that says which old one it replaces. Link the entry from the plan.

### Close a plan

When the work ships (merged, or released if release is part of the scope):

1. Fill in the plan's **Outcome**: what shipped, the PR, any change from the approved plan,
   follow-ups that remain, and checks that still need a person (for example a test on a real
   phone).
2. Set the registry status to `done`. Keep the folder; it is the history.
3. If the work made a decision that future work must respect, make sure it is in
   `decisions.md`.

Use `paused` for work that stops with no plan to continue soon, and `dropped` for work that
will not continue. Write one line in the plan that says why.

## Rules

- Never commit, push or deploy without a clear yes from the person.
- Do exactly what the plan says; no scope creep.
- Keep personal data out of every file this workflow writes: no customer names, emails or real
  user content. Use made-up labels.
- Plans go in the plans folder, never in the agent's own default plan location.
- If the registry and a plan disagree, the plan wins; fix the registry.
