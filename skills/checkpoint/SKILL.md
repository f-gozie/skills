---
name: checkpoint
description: End-of-session ritual for plan-driven work that spans several sessions. Updates the source-of-truth plan, writes a handoff note, optionally commits and pushes, and gives the exact message to paste into the next session. Detects implementation vs design/refinement mode (override with `impl` / `design`; skip git with `--no-push` / `--no-commit`). Run when context is filling up and you want to park the session so a fresh one continues with no re-explaining. Triggers on "checkpoint", "/checkpoint", "park the session", "hand this off".
---

# Checkpoint

Park a session so a fresh one can pick up with zero re-explaining.

The working model: **one source-of-truth plan** per effort, **a handoff note** at each session
boundary, and usually **one branch or PR** that collects the work.

Two modes, detected automatically (override with `impl` or `design`):

- **Implementation** — building a plan in code: phases, tests, a PR.
- **Design / refinement** — iterating a plan and mockups: decisions, rejected directions, open
  questions, often with little code.

The steps are the same for both. Only Step 1 and Step 2 change emphasis.

Optional arguments: free notes about what happened, `impl` / `design`, `--no-commit`, `--no-push`.

---

## Step 0 — Context and mode

- **Find the plans directory.** Look for the project's existing one (`docs/plans/`, `plans/`,
  `context/plans/`, ...). Check the repo's agent instructions (`CLAUDE.md`, `AGENTS.md`) for a stated
  location or naming rule. If there is none, use `docs/plans/` and say so.
- **Find the source-of-truth plan** for this effort. If more than one plan could be it, ask.
- **Branch and PR:** `git branch --show-current`; look for an open PR on it
  (`gh pr list --head <branch>`). No PR yet is fine.
- **Detect the mode** from `git status`, `git diff --stat`, and the conversation:
  - mostly code and tests changed → **implementation**;
  - mostly mockups, plan edits, and rounds of feedback → **design**;
  - both → **mixed**: do both branches below.
- **Mixed working tree:** `git status --short` may show changes that are not part of this effort,
  and secrets such as `.env` files. Only this effort's files get committed. **Never
  `git add .`, `git add -A`, or a whole directory.**

## Step 1 — Update the plan

Put this session's progress into the plan, so the plan stays the source of truth and not the chat.
Record decisions and new facts. Do not reopen what is already decided.

**Implementation:**

- Mark phases done / in progress / not started.
- Update the status line: latest commit, PR, what remains.

**Design / refinement:**

- **Decisions locked** this session, with the reason.
- **Directions explored and rejected** — the do-not-revive list, so the next session does not bring
  back a killed idea.
- **Open questions** still being worked out.
- **Current mockup** — which file is canonical and which are superseded.
- The next focus.

## Step 2 — Write the handoff

Write `<plans-dir>/<YYYY-MM-DD>-handoff-<effort>.md` with today's date. It must stand alone: a fresh
session reads it first, then the plan.

**Always include:**

- What to read first: plan, spec, key files, mockups.
- The working model (one plan, one PR, sessions with handoffs).
- Current state.
- Git state and the safe commit flow from Step 4, so the next session does not rediscover it.
- What this session changed.
- Next steps.
- Gotchas and environment notes (servers, ports, flags, tools that misbehave).
- Links: PR, plans, mockups.

**Implementation emphasis:** what was built (files, modules), test status, bugs found and fixed, the
next phase, and any hooks already left in the code for it.

**Design emphasis:**

- Decisions locked this session, and why.
- **Rejected directions (do not revive)** — the most important thing to carry forward.
- Mockups created or updated, which is canonical, and **how to view them** (the preview command or
  server and port, plus any known tooling limits).
- What is still open, and the next refinement focus.
- Voice and brand rules in play, so copy stays consistent.

## Step 3 — Update memory (if the harness has it)

If the agent has a persistent memory system, update the effort's entry with status, the PR, and this
handoff's path. Pointers only — do not copy plan content into memory. Skip this step if there is no
memory system.

## Step 4 — Commit and push (skip with `--no-commit`)

Follow the repo's own rules first. If the repo requires approval before commits or pushes, show the
staged file list and the message, and wait for a clear yes.

Stage **only** this effort's files: the plan update, the handoff, and the session's work (code and
tests, or mockups).

**Stage precisely:**

- List this effort's changed and new files explicitly. Leave out unrelated work and every `.env` or
  secret file.
- `git add <explicit paths>`. If a filename contains glob characters such as `[` or `]`, use
  `GIT_LITERAL_PATHSPECS=1 git add ...`.

**Safety check (hard stop):** `git diff --cached --name-only` must list only this effort's files.
Search it for `.env`, `secret`, `key`, and names from unrelated work. If anything unexpected is
staged, **stop and report. Do not commit.** If you cannot tell whether a file belongs, ask.

**Commit through pre-commit hooks.** Formatting hooks often change staged files and abort the first
commit. On a tree with unrelated unstaged changes, the hook's own stash can also conflict with its
fixes. A flow that works on a mixed tree:

1. `git diff --cached --name-only > /tmp/staged.txt`
2. `git stash push --keep-index` — parks unrelated unstaged changes
3. `git commit -F <msg-file>` — hooks may reformat and abort
4. `GIT_LITERAL_PATHSPECS=1 git add --pathspec-from-file=/tmp/staged.txt` — re-stage formatted files
5. `git commit -F <msg-file>` — lands
6. `git stash pop` — restores the unrelated changes

On a clean tree, skip the stash: commit, and if hooks reformat, re-add and commit again.

**Verify:** hooks can drop files silently. `git show --stat HEAD` must contain everything expected and
nothing unrelated. `git status` must show nothing from this effort left uncommitted.

**Message:** follow the repo's commit convention (for example `feat(...)`, `docs(...)`).

**Push** (skip with `--no-push`): `git push` to the existing branch so it stacks onto the PR. If there
is no PR yet, say so and offer to open one.

## Step 5 — Report

- New commit(s) and the PR link, or "no PR yet, branch `X`".
- Working tree state: only unrelated changes remain; nothing from this effort is uncommitted.
- One line to paste into the next session:
  **"Read `<handoff path>`, then the plan, and continue with `<next phase or focus>`."**

---

## Rules

- Do exactly this; no scope creep.
- Never commit secrets or unrelated work. When unsure, stop and ask.
- The plan is the source of truth; the handoff is continuity on top of it.
- For design work, the rejected-directions list and the current-mockup pointer are required.
