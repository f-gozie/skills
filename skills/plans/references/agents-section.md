## Dev workflow

Plans, reviews and decisions live in `context/` (see the `plans` skill).

- **Where things are:** what is in progress → `context/plans/README.md`. Why something is the
  way it is → `context/decisions.md`. Conventions → `context/standards/`.
- **Plan first.** Work that touches more than one area, changes a data model or contract, or
  has a real design choice gets a plan folder in `context/plans/`. A person approves the plan
  before code is written. Small fixes need no plan.
- **The plan is the source of truth.** Tick items and add a progress line as work lands. If the
  plan turns out wrong, stop, update it and ask.
- **Checkable success criteria.** Every scope item has a test, command or visible result that
  proves it.
- **Review before every PR** with the `review-all` skill. The report goes in the plan's
  `reviews/` folder.
- **Handoffs only when work stops before it ships.** One file per session in the plan's
  `handoffs/` folder, never edited later.
- **Do not reopen a recorded decision** without a new reason.

### Hard rules

1. Never commit, push or deploy without a clear yes from a person.
2. Do exactly what was asked. Put other ideas in the plan's follow-ups.
3. Check state (`git status`, branch, running services) before risky commands.
4. Run the relevant tests before calling work done. Report failures as they are.
5. When a choice changes behaviour and the answer is not clear, ask.
6. Keep personal data out of plans, commits, PRs, fixtures and logs.

### How to verify

The `autopilot` skill reads this to check work in the running app. Fill it in from the repo;
delete it if nobody will use autopilot here.

- **Start the app:** `<command>` (URL or simulator target)
- **Test data:** `<command or script that makes a test account and data>`
- **Check stored results:** `<how to query the database or API>`
- **Tests per area:** `<area>: <command>`
- **Pass means:** `<what a person would check by hand>`
