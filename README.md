# Skills

Just a bunch of skills I use day to day. Take whatever's useful.

| Skill | What it does |
|---|---|
| [plans](skills/plans/SKILL.md) | Plan-first feature work: one folder per feature for its plan, reviews and handoffs, a registry of what is active, and a decision log. Sets itself up in a repo. |
| [review-all](skills/review-all/SKILL.md) | The pre-PR review: code against the repo's rules, security, a simplifier, and a reviewer from another model family. Verifies every finding and writes the report into the plan. |
| [autopilot](skills/autopilot/SKILL.md) | Hands-off delivery: takes a plan or a request to a reviewed, verified PR with screenshots while you are away. Debates the calls it is unsure of and parks hard stops for you. |
| [checkpoint](skills/checkpoint/SKILL.md) | Ends a long session cleanly: updates the plan, writes a handoff note, commits only the right files, and gives you the line to start the next session. |
| [mockup](skills/mockup/SKILL.md) | Sets up a design system on first use, then designs screens as HTML boards that show every state, asks for approval, and keeps a record of what was approved and rejected. |
| [debate](skills/debate/SKILL.md) | Gives a hard decision to two or three independent agents (Claude and Codex), lets them argue, and returns one clear recommendation. |

`plans`, `review-all`, `autopilot` and `checkpoint` work as a set. [WORKFLOW.md](WORKFLOW.md) explains how
they fit together. The others work on their own.

## Install

Clone the repo:

```bash
git clone https://github.com/f-gozie/skills.git ~/code/skills
```

**Claude Code, all projects.** Link the skills you want into your user skills folder:

```bash
mkdir -p ~/.claude/skills && ln -s ~/code/skills/skills/mockup ~/.claude/skills/mockup
```

A symlink means `git pull` in the repo updates the skill. Copy the folder instead if you want to
change it for yourself.

**Claude Code, one project.** Copy a skill folder into `<project>/.claude/skills/`.

**Codex, all projects.** Link the skill into your Codex skills folder:

```bash
mkdir -p ~/.codex/skills && ln -s ~/code/skills/skills/plans ~/.codex/skills/plans
```

**Claude.ai or the Claude desktop app.** Zip one skill folder (for example `mockup/`) and upload it
under Settings > Capabilities > Skills.

## Update

```bash
cd ~/code/skills && git pull
```

## Notes

- `debate` works best with the [Codex CLI](https://github.com/openai/codex) installed. Without it,
  it uses two Claude models.
- `review-all` uses the model family that is not running the review: Codex from Claude Code, and
  [Claude Code](https://claude.com/claude-code) from Codex. Without the other CLI, it skips that
  reviewer and says so.
- `plans` keeps its files under `context/` by default: `plans/`, `decisions.md`, and optional
  `standards/`. It uses your repo's existing plans folder if there is one.
- `mockup` saves its work in your project under `design/`: `system.md`, `mockups/`, `approved/`.
- `checkpoint` writes handoffs into the active plan's `handoffs/` folder.
