# Skills

Personal, cross-project Claude skills — auto-invoked by Claude based on
the task, not manually typed. See the [repo root README](../README.md)
for how this folder is synced onto `~/.claude/skills/` via a directory
junction, and how to set that up on a new machine.

## Commands vs. skills

A **command** (`commands/`) is manually invoked — you type `/name`.

A **skill** is invoked automatically. Claude reads every skill's `name`
and `description` up front and decides on its own whether it's relevant
to the current task — you never type its name directly. Write the
`description` as a precise trigger, not just a summary: it's the only
thing Claude sees before deciding to load the rest of the skill.

## Folder shape

```
skill-name/
├── SKILL.md              (required - frontmatter + instructions)
└── ...                   (optional - templates, scripts, reference docs)
```


`SKILL.md` starts with frontmatter (`name`, `description`, any other
supported fields), followed by the instructions Claude follows once
invoked. Supporting files live alongside it and are pulled in by the
instructions as needed — see [`prd-generator/`](prd-generator/).

## Claude Code vs. claude.ai

Same `SKILL.md` format, same invocation model on both. Getting a change
live differs:

- **Claude Code** — picks up changes automatically on the next `git pull`
  (via the `~/.claude/skills` junction). No extra step.
- **claude.ai** — no direct access to this git repo. After changing a
  skill, zip *that skill's folder specifically* and re-upload it via
  **Settings → Features**:
```powershell
  cd $env:USERPROFILE\dev\claude-commands-skills
  git pull
  Compress-Archive -Path skills\<skill-name>\* -DestinationPath "<skill-name>.zip" -Force
```
  Upload one skill = one zip; don't bundle multiple skills into one
  upload, and only re-upload skills you actually changed.

## Creating a new skill

1. New subfolder under `skills/`, named after the skill.
2. Add `SKILL.md` with `name` + `description` frontmatter, plus the
   instructions.
3. Add any supporting templates/scripts alongside it.
4. Test in Claude Code first (live immediately, no upload needed) before
   bothering to zip/upload to claude.ai.

## Skills in this repo

- `prd-generator/` — generates a Requirements doc, a numbered file-level
  Plan, and a per-phase DoD checklist for a new project, as three
  separate artifacts rather than one combined document.
