# claude-commands-skills

This is the *user-level* Claude configuration: personal, cross-project
slash commands and skills for Claude Code, shared across every machine
you use it on. It's distinct from a project's own `.claude/commands/` or
`.claude/skills/` folders, which are repo-specific, get committed to that
project's repo, and are meant to be shared with teammates on that project.
Nothing in this repo should be project-specific.

## Structure

```
claude-commands-skills/
├── README.md ← this file
├── commands/ ← custom slash commands, see commands/README.md
└── skills/ ← Claude skills, see skills/README.md
```

## How this stays in sync across machines

This repo lives on GitHub (`josshhz11/claude-commands-skills`) and is
cloned locally to `~/dev/claude-commands-skills/`. Claude Code, however,
looks for commands and skills at fixed paths — `~/.claude/commands/` and
`~/.claude/skills/` — not wherever this repo happens to be cloned. Each
machine bridges that with two OS-level directory junctions:

```
~/.claude/commands → ~/dev/claude-commands-skills/commands
~/.claude/skills → ~/dev/claude-commands-skills/skills
```

A junction makes the two paths the same folder on disk — editing through
either path edits the same files, and `git` run from either path sees the
same repo. Once set up, `git pull` in the repo folder updates both
`~/.claude/commands` and `~/.claude/skills` immediately, with no copy step.

## First-time setup on a new machine

```powershell
# 1. Clone the repo
git clone https://github.com/josshhz11/claude-commands-skills.git "$env:USERPROFILE\dev\claude-commands-skills"

# 2. Create the two junctions (no admin rights needed)
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\commands" -Target "$env:USERPROFILE\dev\claude-commands-skills\commands"
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills" -Target "$env:USERPROFILE\dev\claude-commands-skills\skills"

# 3. Verify — both should list the real subfolder contents
Get-ChildItem "$env:USERPROFILE\.claude\commands"
Get-ChildItem "$env:USERPROFILE\.claude\skills"
```

If `~/.claude/commands` or `~/.claude/skills` already exist as real
folders (not junctions) on that machine — e.g. an old pre-restructure
setup — move or delete them first; `New-Item -ItemType Junction` won't
overwrite an existing folder at that path.

## Publishing a change (from whichever machine you edited on)

```powershell
cd $env:USERPROFILE\dev\claude-commands-skills
git add -A
git commit -m "add: <name> - <what it does>"
git push
```

## Pulling the latest (on any machine, before relying on something new)

```powershell
cd $env:USERPROFILE\dev\claude-commands-skills
git pull
```
No auto-sync — do this before a session where you might rely on a
recently added/changed command or skill. Because both `~/.claude/commands`
and `~/.claude/skills` are junctions into this folder, the pulled changes
appear there automatically.

## Working with a skill on claude.ai (web)

Claude Code picks up skill changes on the next `git pull`, no extra step.
claude.ai has no direct access to this git repo, so after changing a
skill, zip that skill's folder specifically (not the whole `skills/`
directory) and re-upload it via **Settings → Features**:

```powershell
cd $env:USERPROFILE\dev\claude-commands-skills
git pull
cd skills\<skill-name>
tar -a -c -f ..\..\<skill-name>.zip *
cd ..\..
```
Note: use `tar`, not `Compress-Archive` — on Windows, `Compress-Archive`
sometimes writes nested-folder paths with backslashes instead of forward
slashes, which claude.ai's upload validator rejects as "invalid
characters" in the path. `tar` (built into Windows 10/11) writes correct
forward-slash paths. Verify the zip's contents if in doubt:
```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zip = [System.IO.Compression.ZipFile]::OpenRead("<skill-name>.zip")
$zip.Entries | ForEach-Object { $_.FullName }
$zip.Dispose()
```
Then upload `<skill-name>.zip`. Only needed for skills you actually changed.

## Details

- [`commands/README.md`](commands/README.md) — how a command file works,
  writing conventions, commands currently in this repo.
- [`skills/README.md`](skills/README.md) — commands vs. skills, folder
  shape, skills currently in this repo.