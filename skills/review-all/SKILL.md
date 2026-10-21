---
name: review-all
description: The pre-PR review. Runs four reviewers on the same diff — a code review against the repo's own rules, a security review, a simplifier, and a reviewer from a different model family (Codex when the host is Claude, Claude when the host is Codex) — then runs the repo's real checks, verifies every finding against the code, applies only safe fixes, and writes the report into the active plan's reviews folder. Use before opening or updating a PR, or when the user says "review", "review this", "review the branch", "review my changes", "is this ready", or "/review-all". Not for judging findings someone else already wrote (verify those directly), and not for a design decision with no code yet (use debate).
argument-hint: "[branch|uncommitted|staged|<path>] [--no-cross]"
---

# Review all

Four reviewers look at the same diff at the same time. Then the main agent checks every finding
against the real code before it reaches the report. A finding is a lead, not a fact.

Two ideas make this work:

1. **A different model family reviews too.** A model that reviews its own work, or work from its
   own family, misses the same things it missed while writing. The fourth reviewer is always the
   other family.
2. **Nothing reaches the report unverified.** The main agent opens each cited line and confirms
   the problem is real and in scope.

**The repo is the source of truth, not this file.** Read the rules in `AGENTS.md` / `CLAUDE.md`,
any `standards/` files, `decisions.md`, and the CI config at review time. If they disagree with
this file, the repo wins.

## Arguments

| Arg | What gets reviewed |
|---|---|
| `branch` *(default)* | Everything on this branch that is not on the base branch, committed or not, plus new untracked files |
| `uncommitted` | Working-tree changes and new untracked files |
| `staged` | `git diff --cached` |
| `<path>` | Only that file or folder, inside the default scope |
| `--no-cross` | Skip the cross-model reviewer |

## Phase 1 — Scope and context

1. **Find the base branch:** `git symbolic-ref --short refs/remotes/origin/HEAD` (for example
   `origin/main`). If that fails, use `main`, else `master`.
2. **Build one diff file** for every reviewer, in a temp folder (`$TMPDIR/review-all/`):
   - `branch`: `git diff $(git merge-base HEAD <base>)` — this includes uncommitted changes.
   - `uncommitted`: `git diff HEAD`.
   - `staged`: `git diff --cached`.
   - For `branch` and `uncommitted`, add new files: for each path in
     `git ls-files --others --exclude-standard`, append `git diff --no-index /dev/null <path>`.
3. **Drop noise:** lock files, build output, vendored code, generated files, binaries. If a file
   is generated, review only that it was regenerated and not edited by hand.
4. **Read the rules:** `AGENTS.md`, `CLAUDE.md`, `standards/`, and `decisions.md` so a deliberate,
   recorded choice is not reported as a bug.
5. **Find the plan:** the `active` row in `plans/README.md` (see the `plans` skill). Note its
   `plan.md` for the plan check. No plan is fine for small work.

If the diff is empty, say so and stop.

## Phase 2 — Four reviewers, at the same time

Give each reviewer the same diff file, the list of changed files, the repo rules from Phase 1,
and the plan path. Every reviewer is **read-only** and only reports. Nobody edits files in this
phase, because edits would move the line numbers the other reviewers cite.

**How to run them:**

- **Claude Code:** start reviewers 1–3 as subagents (`Agent` tool, read-only instruction) and the
  cross-model command (Bash) in **one message**, so all four run at once.
- **A host without subagents (for example Codex):** start the cross-model command in the
  background first, then do reviewers 1–3 yourself, one after another, then collect the
  cross-model result.

Each reviewer returns findings as: file, line, severity (`critical` / `warning` / `info`),
category, title, and what to change.

### Reviewer 1 — Code review

- **Repo rules:** check the diff against every rule in `AGENTS.md`, `CLAUDE.md` and `standards/`.
  A broken hard rule is critical.
- **Correctness:** trace the changed functions. Look for wrong logic, missing error handling,
  empty and null cases, off-by-one, race conditions, and migrations that break old data.
- **The plan:** does the diff do what `plan.md` says? Report scope creep and missing items.
- **Tests:** new logic and every bug fix need a test that would fail without the change. Report
  missing tests and tests that cannot fail.
- **Fit:** the new code reads like the code around it (naming, comment density, patterns).

### Reviewer 2 — Security

- Secrets, keys or tokens in code, config or logs.
- Injection: SQL, shell commands, HTML/JS (XSS), file paths, unsafe deserialization, SSRF.
- Missing or wrong authentication and authorization checks, including on new endpoints.
- Personal or sensitive data in logs, errors, analytics or responses.
- Missing input validation where data enters the system.
- Unsafe defaults: CORS, cookies, permissions, new dependencies with known problems.

### Reviewer 3 — Simplifier

- Dead code, unused imports, duplication, needless layers or options added "for later".
- Long functions and deep nesting that a simpler shape would fix.
- Comments that repeat the code, and other signs of generated filler.
- **Report only.** The fixes are applied in Phase 6, after verification.

### Reviewer 4 — Cross-model (skip with `--no-cross`)

Use the model family that is **not** running this skill:

- **You are Claude →** use Codex. Check `which codex`.
- **You are Codex (or another non-Claude agent) →** use Claude. Check `which claude`.

Write one prompt file: a short line about the project, the repo's hard rules, this instruction,
and the diff.

> You are a senior engineer reviewing a git diff. Report only real, actionable findings tied to a
> changed file and line, as JSON per the schema. Check correctness, security, the repo rules
> above, tests, and needless complexity. Do not invent speculative risks or propose large
> rewrites. severity: critical = bug, security hole or broken rule; warning = should fix;
> info = minor.

The schema is `findings-schema.json`, in the same folder as this file.

```bash
# Codex as the second model (read-only, nothing saved)
codex exec --ephemeral -C "$(pwd)" -s read-only \
  --output-schema <skill-dir>/findings-schema.json \
  -o "$TMPDIR/review-all/cross.json" - < "$TMPDIR/review-all/prompt.md"

# Claude as the second model (read-only tools only)
claude -p --output-format json \
  --json-schema "$(cat <skill-dir>/findings-schema.json)" \
  --allowedTools Read Grep Glob --permission-mode dontAsk \
  < "$TMPDIR/review-all/prompt.md" > "$TMPDIR/review-all/cross-raw.json" \
  && jq '.structured_output' "$TMPDIR/review-all/cross-raw.json" > "$TMPDIR/review-all/cross.json"
```

Use a strong model at high effort if the person has not set one (for example
`codex exec -m <model> -c model_reasoning_effort="high"`, or `claude -p --model opus`).

**If the other CLI is missing, fails, times out, or returns no valid JSON:** write
"Cross-model review skipped: <reason>" in the report and continue. Never fail the whole review
because of it.

## Phase 3 — Run the real checks

Run the checks that CI runs, for the areas the diff touches. Read the list from the CI config
(`.github/workflows/`, or similar) and the repo's instructions. Do not guess commands.

- Typically: formatter check, linter, type check, the tests for the changed areas.
- **A red check is critical.** It blocks the PR.
- A check that cannot run here (missing tool, needs a service that is not up) is noted, not
  failed. Say what ran and what did not.

## Phase 4 — Verify every finding

For every critical and warning finding, from all four reviewers:

1. **Open the cited file and line.** Confirm the problem is real. If the line is not in the
   diff, drop it.
2. **Drop speculation.** If you cannot point to a concrete code path that goes wrong, mark it
   unverified and drop it or move it to info.
3. **Check the repo context.** A choice recorded in `decisions.md` or a rule file is not a bug.
4. **Look for siblings.** When a bug is real, search the diff and nearby code for the same
   pattern. Report every instance, not one.
5. **Check the fix.** A suggested fix must not break a check or a repo rule.

**Two model families agreeing is strong signal.** Mark a finding *cross-model confirmed* when the
cross-model reviewer and a Claude reviewer both raise it.

Count: verified, dropped, cross-model confirmed.

## Phase 5 — Consolidate

Merge the verified findings. Same file, same lines, same problem = one entry; keep the highest
severity. Sort critical → warning → info, and group by file.

## Phase 6 — Safe fixes only

Apply only fixes that cannot change behaviour, then re-run the affected checks:

- **Safe:** formatter and import-sort output, machine-applicable lint fixes, unused imports and
  dead code, small simplifications that keep the same behaviour (early return, rename of a local).
- **Never without asking:** security fixes, logic or error-handling changes, public API or
  contract changes, migrations, generated files, anything the verification did not confirm.

## Phase 7 — Write the report

If there is an active plan, write the report to `<plan-folder>/reviews/YYYY-MM-DD-<topic>.md`.
Create `reviews/` if it does not exist. If there is no plan, show the report in chat only. Do not
create a plan just to hold a review.

```markdown
# Review — <topic>

**Date:** YYYY-MM-DD · **Scope:** <branch|uncommitted|...> · **Files:** N
**Plan:** [plan.md](../plan.md) · **Cross-model:** <Codex|Claude> ran | skipped (<reason>)

## Checks
| Check | Result |
|---|---|
| <command> | pass / fail / not run (<why>) |

## Findings
**Verified:** V · **Dropped:** D · **Cross-model confirmed:** X

### Critical (fix before the PR)
- `path:line` — **[category]** what is wrong · why it matters · the fix

### Warnings (should fix)
- `path:line` — **[category]** ...

### Info
- ...

## Fixes applied
- ...

## Still to do
- [ ] ...

## Plan check
<matches the plan | differs: one line on what and why>
```

## Phase 8 — Present

1. In chat: the checks table, the finding counts, the critical items, and the report path.
2. Offer to fix the remaining items one at a time.
3. If everything is green and nothing critical or warning is left, say: "All checks pass. No
   critical or warning findings. Ready for a PR."

## Rules

- Reviewers never edit files. Only the main agent edits, and only in Phase 6.
- Report what ran and what did not. Never claim a check passed if it did not run.
- Do not commit or push as part of the review.
- Keep personal data out of the report.
