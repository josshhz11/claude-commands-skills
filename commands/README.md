# Commands

Custom slash commands for Claude Code. Manually invoked: you type
`/<name>` and Claude runs the instructions in that file, in the current
repo/session.

This is the *user-level* command folder — see the [repo root
README](../README.md) for how it's synced onto `~/.claude/commands/` via
a directory junction, and how to set that up on a new machine.

Distinct from a project's own `.claude/commands/` (repo-specific, gets
committed to that project's repo, meant to be shared with teammates on
that project). Nothing in here should be project-specific — if a command
needs to know something about a particular repo, it figures that out at
runtime (`git log`, reading files in the repo, checking whether a
`memory/` folder exists, etc.), it doesn't hardcode it.

## How a custom slash command works

Each `.md` file in this folder becomes a command named after its filename
— `commit-pr.md` → `/commit-pr`.

```markdown
---
description: One-line summary shown when you list commands
argument-hint: [optional hint shown while typing]
---

The actual prompt/instructions Claude follows when you run this command.
Write it exactly like you'd write instructions to Claude directly —
numbered steps, explicit dos/don'ts, whatever gets a reliable result.

Use $ARGUMENTS anywhere you want whatever the user typed after the command
name substituted in.
```

Frontmatter fields (all optional): `description`, `argument-hint`,
`allowed-tools` (restricts which tools this command can use — omit to
inherit the session's), `model` (pin this command to a specific model).

Subfolders namespace commands: `git/commit.md` → `/git:commit`. Flat is
fine until a flat list gets messy.

## Creating a new command

1. Add a `.md` file directly in this folder (or a namespacing subfolder).
   No build step, no restart — live the next time you invoke it by name,
   on this machine.
2. Test it in a real repo before committing.
3. Iterate directly on the file — each save takes effect immediately.

## Publishing / pulling

```powershell
cd $env:USERPROFILE\dev\claude-commands-skills
git add -A
git commit -m "add: <command-name> - <what it does>"
git push
```
```powershell
cd $env:USERPROFILE\dev\claude-commands-skills
git pull
```
No auto-sync — pull before a session where you might rely on the newest
version. See the [repo root README](../README.md) for first-time junction
setup on a new machine.

## Conventions for commands in this repo

- Every command should work from a **fresh session with no prior chat
  context** — pull whatever it needs from the filesystem/git at runtime.
  Don't write a command that only works if Claude already "remembers"
  something from earlier in the conversation.
- Don't assume a specific project structure (e.g. a `memory/` folder with
  particular filenames) — check whether something's there and adapt,
  never require it.
- Anything destructive or hard to reverse (pushing, opening a PR, deleting
  something) — draft it and stop; require an explicit go-ahead before
  actually running it.

## Commands in this repo

- `commit-pr.md` (`/commit-pr`) — drafts a Conventional-Commits commit (or
  a small set of them) from the current repo's actual diff, then — once
  real commits exist on the branch — a PR summary from real commit
  history. Never pushes, never opens a PR on its own.