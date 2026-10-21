# Plans

Every plan in this repo, newest first. Find the `active` plan, then read its `plan.md` and the
newest file in its `handoffs/` folder, if it has one.

| Plan | Status | What |
|---|---|---|
| | | |

Status is one of `active`, `paused`, `done`, `dropped`. This table is the only place a plan's
status is kept. One-off fixes need no plan.

## A plan folder

```text
YYYY-MM-DD-<slug>/
  plan.md       # the one living document: scope, checks, approach, progress, outcome
  reviews/      # one report per review run
  handoffs/     # only when work spans sessions; one file per session, never edited later
  tasks/        # only when the plan ships as several PRs; one how-to per PR
  notes/        # optional research behind a decision
```

The reasons behind choices are in `../decisions.md`.
