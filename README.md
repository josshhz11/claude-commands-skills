# Personal Claude Code Commands

Personal, cross-project custom slash commands for Claude Code. Lives at
`~/.claude/commands/` on every machine - anything here is available as
`/<name>` in **every** repo you open Claude Code in, no per-project setup.

This is the *user-level* command folder, distinct from a project's own
`.claude/commands/` (repo-specific, gets committed to that project's repo,
meant to be shared with teammates on that project). Nothing in here should
be project-specific - if a command needs to know something about a
particular repo, it figures that out at runtime (`git log`, reading files
in the repo, checking whether a `memory/` folder exists, etc.), it doesn't
hardcode it.

## How a custom slash command works

Each `.md` file in this folder becomes a command named after its filename -
`commit-pr.md` → `/commit-pr`.

A command file has two parts:

```markdown
---
description: One-line summary shown when you list commands
argument-hint: [optional hint shown while typing]
---

The actual prompt/instructions Claude follows when you run this command.
Write it exactly like you'd write instructions to Claude directly - numbered
steps, explicit dos/don'ts, whatever gets a reliable result.

Use $ARGUMENTS anywhere you want whatever the user typed after the command
name substituted in - e.g. `/commit-pr focus on the auth changes` makes
$ARGUMENTS become "focus on the auth changes".
```

Frontmatter fields worth knowing (all optional):
- `description` - shown in the command picker/list
- `argument-hint` - shown as a placeholder while typing the command
- `allowed-tools` - restricts which tools this command can use (omit to
  inherit whatever the current session already allows)
- `model` - pin this command to a specific model regardless of the
  session's default

Subfolders namespace commands: `git/commit.md` → `/git:commit`. Flat is
fine for now (just `commit-pr.md`) - reorganize into subfolders once
there are enough of these that a flat list gets messy.

## Creating a new command

1. Add a new `.md` file directly in this folder (or a subfolder for
   namespacing - see above). No build step, no restart - it's live the
   next time you invoke it by name in any Claude Code session, on this
   machine.
2. Test it in a real repo before committing - run `/<name>` and check the
   output actually does what you meant, same as sanity-checking any
   prompt.
3. Iterate directly on the file and re-test - each save takes effect
   immediately.

## First-time setup (once, on whichever machine you set this up on)

This folder isn't a git repo until you do this once. `gh` (GitHub's CLI)
isn't required - these steps work with plain `git` and the GitHub website.

```powershell
cd $env:USERPROFILE\.claude\commands
git init
git add -A
git commit -m "init: personal Claude Code slash commands"
git branch -M main
```

Then create an **empty** repo on GitHub (github.com/new) - name it
`claude-commands`, pick private or public, and leave every checkbox
unticked (no README/.gitignore/license - if GitHub creates any file at
all, its history won't match this local commit and the push below will be
rejected). Then:

```powershell
git remote add origin https://github.com/<your-username>/claude-commands.git
git push -u origin main
```

If you have `gh` installed and authenticated, the repo-creation step
collapses into the `git init`/`add`/`commit` above plus one line instead
of the website + remote-add dance:
```powershell
gh repo create claude-commands --private --source=. --remote=origin --push
```

## Publishing a change (from whichever machine you edited on)

```powershell
cd $env:USERPROFILE\.claude\commands
git add -A
git commit -m "add: <command-name> - <what it does>"
git push
```

## Pulling onto a new or different device

**First time on a machine** (the folder doesn't exist yet, or is empty):
```powershell
git clone https://github.com/josshhz11/claude-commands.git $env:USERPROFILE\.claude\commands
```

**Getting the latest, on a machine that already has this repo:**
```powershell
cd $env:USERPROFILE\.claude\commands
git pull
```
There's no auto-sync - do this before a session where you might rely on
the newest version of a command, same as pulling any other repo.

**If the folder already has unrelated files on that machine** (e.g. you'd
made a one-off command there before setting this up): move those files
aside first, clone into the now-empty folder, then decide whether to fold
the pre-existing ones back in as their own commit.

## Conventions for commands in this repo

- Every command should work from a **fresh session with no prior chat
  context** - pull whatever it needs from the filesystem/git at runtime
  (see `commit-pr.md` for the reasoning in full). Don't write a command
  that only works if Claude already "remembers" something from earlier in
  the conversation - a new session, or work committed outside Claude Code
  entirely, has none of that.
- Don't assume a specific project structure (e.g. a `memory/` folder with
  particular filenames inside it) - check whether something's there and
  adapt, never require it.
- Anything destructive or hard to reverse (pushing, opening a PR, deleting
  something) - draft it and stop; require an explicit go-ahead before
  actually running it. `commit-pr.md` follows this: stages and drafts a
  commit message, but never runs `git push` or `gh pr create` itself.

## Commands in this repo

- `commit-pr.md` (`/commit-pr`) - drafts a Conventional-Commits commit (or
  a small set of them) from the current repo's actual diff, then - once
  real commits exist on the branch - a PR summary from real commit
  history. Never pushes, never opens a PR on its own.
