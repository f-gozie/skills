# Skills

Claude skills I use every day, cleaned up for any project.

| Skill | What it does |
|---|---|
| [mockup](skills/mockup/SKILL.md) | Sets up a design system on first use, then designs screens as HTML boards that show every state, asks for approval, and keeps a record of what was approved and rejected. |
| [debate](skills/debate/SKILL.md) | Gives a hard decision to two or three independent agents (Claude and Codex), lets them argue, and returns one clear recommendation. |
| [checkpoint](skills/checkpoint/SKILL.md) | Ends a long session cleanly: updates the plan, writes a handoff note, commits only the right files, and gives you the line to start the next session. |

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

**Claude.ai or the Claude desktop app.** Zip one skill folder (for example `mockup/`) and upload it
under Settings > Capabilities > Skills.

## Update

```bash
cd ~/code/skills && git pull
```

## Notes

- `debate` works best with the [Codex CLI](https://github.com/openai/codex) installed. Without it,
  it uses two Claude models.
- `mockup` saves its work in your project under `design/`: `system.md`, `mockups/`, `approved/`.
- `checkpoint` uses your project's plans folder, or `docs/plans/` if there is none.
