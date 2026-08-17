---
description: Draft git add/commit command(s) with a properly-typed commit message (Adobe React-Spectrum PR naming convention), then a PR summary from real commit history. Drafts only - never pushes, never opens a PR.
argument-hint: [optional: extra context about what changed / what to focus on]
---

You are drafting a commit (or a small set of commits) and a PR summary for
the CURRENT repository, then stopping. Never push and never run
`gh pr create` unless the user explicitly asks in a later message - this
command's job ends at giving them text to review.

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
  `master` if `main` doesn't exist) - needed for Step 3.
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
- Commit message format - the Adobe React-Spectrum PR Naming Guide
  (https://github.com/adobe/react-spectrum/wiki/Pull-Request-Naming-Guide):
  - Header: `<type>(<scope>): <summary>` - `(<scope>)` is optional,
    `<type>` and `<summary>` are not.
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
    codebase affected (e.g. `feat(virtualization)`, `fix(auth)`). Never an
    issue identifier. Only include it when it genuinely narrows down where
    the change lives - omit it for a change that touches the repo broadly
    or doesn't cleanly belong to one area.
  - `<summary>` - concise, readable at a glance, imperative mood ("allow
    useHref on synthetic links", not "allowed"/"allows useHref...").
    Examples straight from the guide: `fix: allow useHref on synthetic
    links`, `docs: fix typo in usePress docs`, `feat(virtualization): add
    support for custom collection renderers`.
  - Body (when the header alone doesn't explain it): a few bullet points of
    what changed and why, matching the voice/density of this repo's own
    recent commits from Step 1 if it has an established one.
  - Do NOT add a `Co-Authored-By` trailer yourself - the environment's own
    git convention already appends that automatically when the commit
    actually runs; don't duplicate it in the drafted text.

Present the exact `git add ...` and `git commit -m "..."` command(s) you're
proposing, then STOP and wait for the user to confirm before running
anything. Do not run `git add`/`git commit` until they say to.

## Step 3 - PR summary, from real commit history

Only after commit(s) actually exist on this branch (either just made in
Step 2, or already sitting there from earlier in this branch's life):

- `git log <merge-base>..HEAD --oneline` - every commit unique to this
  branch, oldest first.
- For each one, `git show --stat <sha>` (and `git show <sha>` if the stat
  alone doesn't make the real content clear) - don't infer content from the
  commit message alone, verify it.
- Produce the summary in exactly this format, one `##` section per commit,
  oldest first:

```
# PR Summary

## <short sha>: <what the commit does, a few words>

- <each real addition/change/removal in that commit, one bullet each>
```

Keep bullets concrete and specific (name the actual file/function/behavior
that changed) - not generic restatements of the commit header.

## Notes

- Never push (`git push`) and never run `gh pr create` here - draft only.
- Working across multiple repos in one session: re-run Step 1 fresh each
  time this command is invoked - don't assume a prior repo's `git status`
  still applies.

$ARGUMENTS
