# claude-commands-skills

This is the *user-level* Claude configuration: personal, cross-project
slash commands and skills for Claude Code, shared across every machine
you use it on. It's distinct from a project's own `.claude/commands/` or
`.claude/skills/` folders, which are repo-specific, get committed to that
project's repo, and are meant to be shared with teammates on that project.
Nothing in this repo should be project-specific.

## How this stays in sync across machines

This repo lives on GitHub and is cloned locally as
`claude-commands-skills/`. Claude Code, however, looks for commands and
skills at fixed paths - `~/.claude/commands/` and `~/.claude/skills/` -
not wherever this repo happens to be cloned. To bridge that, each machine
has two OS-level directory junctions:

```
~/.claude/commands  →  claude-commands-skills/commands
~/.claude/skills    →  claude-commands-skills/skills
```

So editing a file through `~/.claude/commands/` and editing it through
this repo's `commands/` folder are the same file - the junction just
means Claude Code doesn't need to know this repo exists. Pull the repo,
and both `~/.claude/commands` and `~/.claude/skills` show the update
immediately, no copy step.

## Structure

- [`commands/`](commands/) - custom slash commands (`/name`, manually
  invoked). See [commands/README.md](commands/README.md) for how a
  command file works, first-time setup, and publishing changes.
- [`skills/`](skills/) - Claude skills (auto-invoked by Claude based on
  the task). See [skills/README.md](skills/README.md) for how a skill
  differs from a command and how syncing works on claude.ai vs. Claude
  Code.
