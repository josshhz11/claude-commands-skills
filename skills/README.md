# Skills

Personal, cross-project Claude skills. This folder is synced onto
`~/.claude/skills/` on every machine via an OS-level directory junction
(see the [repo root README](../README.md) for how).

## Commands vs. skills

A **command** (`commands/`) is manually invoked - you type `/name` and
Claude runs that exact prompt.

A **skill** is invoked automatically. Claude reads every skill's `name`
and `description` up front and decides on its own, based on what you're
asking for, whether a skill's instructions are relevant to the current
task - you never type its name directly. Write the `description` as a
precise trigger, not just a summary: it's the only thing Claude sees
before deciding to load the rest.

## Folder shape

Each skill is its own subfolder:

```
skill-name/
├── SKILL.md              (required - frontmatter + instructions)
└── ...                   (optional - templates, scripts, reference docs)
```

`SKILL.md` starts with frontmatter (`name`, `description`, and any other
supported fields) followed by the instructions Claude follows once the
skill is invoked. Supporting files (templates, helper scripts, reference
material) live alongside it and are pulled in by the instructions as
needed - see [`prd-generator/`](prd-generator/) for the shape.

## Claude Code vs. claude.ai

Skills work identically on both surfaces once installed - same
`SKILL.md` format, same invocation model. Getting a change live differs:

- **Claude Code**: picks up changes automatically on the next `git pull`
  (via the `~/.claude/skills` junction). No extra step.
- **claude.ai**: has no direct access to this git repo. After any change
  to a skill, zip that skill's folder and re-upload it via
  **Settings → Features** to push the update.

## Skills in this repo

- `prd-generator/` - stub, not yet filled in.
