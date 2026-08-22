---
description: Draft git add/commit/push command(s) with a single-line, properly-typed commit message (Adobe React-Spectrum PR naming convention) plus a PR summary carrying the detail. Stages, commits, and pushes only after explicit go-ahead - never runs `gh pr create` on its own.
argument-hint: [optional: extra context about what changed / what to focus on]
---

You are drafting a commit (or a small set of commits), a push to the
current branch's origin, and a PR summary for the CURRENT repository. The
add/commit/push commands only actually run once the user gives an explicit
go-ahead in this same conversation - never assume a prior run's approval
carries over. Never run `gh pr create` here under any circumstances, even
with a go-ahead - that's out of scope for this command entirely.

## Step 1 - Understand what actually changed

Run, in order:
- `git status`
- `git diff --stat` and `git diff` (working tree); also `git diff --cached
  --stat` / `git diff --cached` if anything is already staged
- `git log --oneline -15` - the repo's own recent commit-message
  conventions. If it already has a consistent style (even one that differs
  from the guidance below - e.g. always starting with "add:"/"update:"),
  match it rather than overriding it silently; if it's mixed or this is an
  early-history repo, use the type/scope/summary format in Step 2.
- `git branch --show-current`, and `git merge-base main HEAD` (fall back to
  `master` if `main` doesn't exist), then `git log <merge-base>..HEAD
  --oneline` - any commits already on this branch from before this
  invocation. Usually empty (most runs are drafting the first/next commit
  fresh) - if it's not empty, those real prior commits get included in
  Step 3's numbering too, oldest first, ahead of whatever's freshly
  drafted this run.
- If a `memory/` directory exists at the repo root: skim its most recently
  modified files (by mtime, e.g. `ls -lt memory` / `Get-ChildItem memory |
  Sort LastWriteTime -Descending` - this folder is commonly gitignored, so
  `git status`/`git log` won't surface it) for "why" context relevant to
  the current diff. Don't assume any particular filename inside it (this
  varies per project - some use `decisions_log.md`/`project_state.md`,
  others won't) - just read whatever's actually there. This is optional
  enrichment for the commit body/PR bullets, never a requirement - the
  diff and `git log` from above must be sufficient on their own even when
  no `memory/` folder exists.
- Deliberately NOT a source here: the current chat conversation. This
  command has to produce a correct result even in a brand-new session with
  no prior conversation (e.g. committing work from an earlier session, or
  work done outside Claude Code entirely) - git plus an optional
  `memory/` folder are the only inputs that are reliably present every
  time this runs.

If there's nothing to commit, say so plainly and stop here.

## Step 2 - Draft the staging + commit command(s)

- Default to one commit unless the diff clearly spans multiple unrelated
  concerns (e.g. an unrelated dependency bump mixed into a feature change).
  In that case, propose splitting into separate commits and say why, rather
  than silently squashing unrelated work together.
- Prefer targeted `git add <path> <path>` over a blanket `git add -A`/
  `git add .` when something in `git status` looks unrelated to the change
  being committed (a stray scratch file, unrelated config, a leftover debug
  script) - call these out explicitly rather than staging them silently.
- Commit message is **a single line only** - `git commit -m "<type>(<scope>):
  <summary>"`, nothing after it. No body, no bullets, no multi-line `-m`.
  All the "what changed and why" detail goes into the PR summary in Step 3
  instead, not into the commit message itself.
  - Type/scope/summary format - the Adobe React-Spectrum PR Naming Guide
    (https://github.com/adobe/react-spectrum/wiki/Pull-Request-Naming-Guide):
    - `<type>` - exactly one, whichever actually matches what the diff does
      (never a default/habitual choice):
      | Prefix | Meaning |
      |---|---|
      | `fix` | Fixing a bug |
      | `feat` | Adding a new feature |
      | `build` | Updates that affect the build system/process |
      | `chore` | Miscellaneous commits that do NOT affect the meaning of the code - whitespace, formatting, missing semicolons, typos within code, comment adjustments, etc. Not a catch-all for dependency or build changes - those are `bump`/`build` specifically. |
      | `docs` | A change to documentation only |
      | `test` | Adding or fixing existing tests |
      | `refactor` | A code change that neither fixes a bug nor adds a feature |
      | `ci` | Changes to CI config |
      | `localize` | Changes related to translations and localization |
      | `bump` | Increasing the version of some dependency |
      | `revert` | Undoing a previous commit |
    - `<scope>` (optional) - a noun giving context to the part of the
      codebase affected (e.g. `feat(virtualization)`, `fix(auth)`). Never
      an issue identifier. Only include it when it genuinely narrows down
      where the change lives - omit it for a change that touches the repo
      broadly or doesn't cleanly belong to one area.
    - `<summary>` - concise, readable at a glance, imperative mood ("allow
      useHref on synthetic links", not "allowed"/"allows useHref...").
      Examples straight from the guide: `fix: allow useHref on synthetic
      links`, `docs: fix typo in usePress docs`, `feat(virtualization): add
      support for custom collection renderers`.
  - Do NOT add a `Co-Authored-By` trailer yourself - the environment's own
    git convention already appends that automatically when the commit
    actually runs.

## Step 3 - Push

`git push origin <branch>`, where `<branch>` is the current branch from
Step 1's `git branch --show-current` - never hardcode a branch name.

Present this alongside Step 2's `git add`/`git commit` as one bundled set
of commands. All three (add, commit, push) wait on the same single
go-ahead from the user before actually *running* any of them - this is the
only thing in this command that waits for confirmation. The PR summary in
Step 4 does not (see below).

## Step 4 - PR summary, drafted in the same pass as Steps 2-3

Generate this immediately alongside Steps 2-3, in the same response - do
**not** wait for the commit/push drafted above to actually be run first.
The detailed "what changed and why" that used to live in the commit
message body now lives here instead, so this has to be available the
moment the commit is drafted, not gated behind the user separately
confirming and running it.

- Base each section on the same diff analysis from Step 1/2 for anything
  being drafted fresh this run. For any commits already on the branch from
  before this invocation (the `git log <merge-base>..HEAD --oneline` check
  in Step 1), use `git show --stat <sha>` (and `git show <sha>` if the stat
  alone doesn't make the real content clear) instead - don't guess their
  content from the commit message alone.
- Number sections sequentially in commit order - **never a git commit SHA**,
  even for real pre-existing commits where one's available. A SHA doesn't
  exist yet for a freshly-drafted commit, and mixing "real SHA" and
  "doesn't have one yet" formatting across sections would be inconsistent -
  plain sequential numbering works uniformly for both and doesn't create a
  dependency on the commit having actually happened.
- Output the PR summary as a single copy-pasteable block, wrapped in its own
  fenced code block (use four backticks for the outer fence so the `##`/`#`
  lines inside render as literal text, not as headings in the chat) - the
  same way the git commands in Step 2/3 are shown, so the user can copy the
  raw markdown as-is rather than getting it rendered. One `##` section per
  commit, oldest first, in exactly this format:

````
# PR Summary

## Commit 1: <what the commit does, a few words>

- <each real addition/change/removal in that commit, one bullet each>

## Commit 2: <...>

- ...
````

Keep bullets concrete and specific (name the actual file/function/behavior
that changed) - not generic restatements of the commit header.

## Notes

- `git add`/`git commit`/`git push` all wait for one explicit go-ahead
  before actually running (Step 3) - never run any of them speculatively.
  Never run `gh pr create` here at all, go-ahead or not - out of scope.
- Working across multiple repos in one session: re-run Step 1 fresh each
  time this command is invoked - don't assume a prior repo's `git status`
  still applies, and don't assume a go-ahead given for one repo carries
  over to another.

$ARGUMENTS
