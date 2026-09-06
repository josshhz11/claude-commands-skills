---
description: Append a dated entry to this repo's decisions-log memory file and update its project-state memory file, based on the code diff/git history and/or whatever was just done in this session.
argument-hint: [optional: what changed / what to focus on, if it's not obvious from the diff or this conversation]
---

You are updating the CURRENT repository's own persistent memory files — a
decisions log (a chronological, append-only record of real incidents/
decisions/fixes) and a project-state summary (a living, reverse-
chronological snapshot of where things stand) — to reflect whatever has
changed in the code, or been decided/done, since the log's last entry.
These are informal continuity notes for a future session to read, not
polished documentation for other people — write them the way the existing
entries in each file are already written, not in a more formal voice.

This command must work from a **fresh session with no prior chat
context** (e.g. catching up memory for work done outside Claude Code, or
in an earlier session that's gone) just as well as from a session that
already has the relevant work fresh in it — see Step 2.

## Step 1 - Find the two memory files

Don't assume a fixed location or exact filename - conventions vary per
project.

1. Look for a `memory/` directory at the repo root first. If it doesn't
   exist, search the repo root itself, then one level down.
2. Within wherever it's found, identify:
   - **the decisions log** - a file whose name matches `decision(s)?[_-]?log`
     (case-insensitive, underscore/hyphen/no-separator all count -
     `decisions_log.md`, `decision-log.md`, `decisionslog.md` all match)
   - **the project state file** - a file whose name matches
     `project[_-]?state` similarly
3. If either pattern matches more than one file, or matches none, **stop
   and ask** which file to use (or whether to create one) rather than
   guessing - don't invent a memory structure the project doesn't already
   have.
4. These files can be large. Don't read either one in full:
   - Decisions log: read the last ~150-200 lines (or `grep` for the last
     few section headers) - you only need the most recent entry's date,
     topic, and exact formatting conventions (heading style, bold labels
     like `**What happened:**`/`**Fixed:**`, how file/function names are
     quoted, `---` separators between entries, etc.).
   - Project state: read from the top through the first one or two
     existing entries - you need the "last updated" line (or equivalent)
     and the surrounding style, not the whole history below it.

## Step 2 - Work out what actually changed or happened

Use whichever of these actually apply - both, if both do:

- **This conversation's own recent context**, if there is any. Real
  decisions, mistakes, refusals, user choices, and reasoning routinely
  produce no code diff at all (a rejected approach, a security/policy
  boundary held, a feature explicitly deferred, a bug root-caused to
  something outside the repo) - a diff can't show any of that, so when
  this session actually did the work being logged, treat the conversation
  itself as the primary source for the *why*, and use git only to ground
  the *what* precisely (exact file/function names, line numbers, test
  counts) rather than paraphrasing from memory.
- **Git, always** - `git status`, `git diff`/`git diff --cached` (working
  tree), and `git log --oneline` since whatever point the decisions log's
  last entry left off at (parse that entry's date, then `git log
  --since=<that date> --oneline`, adjusting for entries authored on a
  different clock than the repo's own commit timestamps - if the log's
  own dates and `git log`'s commit dates disagree, as they may in a
  project that back/forward-dates its narrative, trust the decisions
  log's own last-entry date as the boundary and use `git log` only to
  enumerate *what* changed, not to re-date anything). This is the only
  source available in a cold session with no prior conversation, and it's
  also how you get exact specifics (a real test count from actually
  running the suite if the project has one and code - not just docs -
  changed; a real file/function name; a real branch/PR number from `git
  log`/`git branch --show-current`) even when the conversation already
  told you the gist.
- **$ARGUMENTS**, if given - extra context/focus the user is pointing you
  at directly.

If neither the conversation nor git/the working tree turns up anything
that isn't already reflected in the decisions log's last entry, say so
plainly and stop - don't manufacture an entry for nothing.

## Step 3 - Append to the decisions log

Match whatever heading/style conventions Step 1 found in the existing
file (don't impose a different one), with one deliberate, standardized
addition going forward: put the date directly under the entry's title,
not buried at the end.

```markdown
---

### <Concise title - what happened or what changed, not a vague label>
**Date:** YYYY-MM-DD

<Body, in whatever style the existing entries already use - typically
short bolded labels (**What happened:**, **Fixed:**, **Also:**, **Why:**)
introducing a few sentences each, backtick-quoting real file/function/
branch names, past tense, one paragraph per distinct point rather than a
wall of text. Be concrete and specific - name the actual file, function,
bug, decision, or number, the same way the entries you read in Step 1 do.
If a fix/change has a branch or PR number, name it. If tests were added
or a suite was run, give the real count, not an approximate one.>
```

- Compute `YYYY-MM-DD` from the current system date unless the
  conversation's own stated "current date" clearly differs from it (some
  projects run on a fictional/forward-dated timeline for their narrative
  memory - if Step 1's existing entries are consistently dated against a
  fictional "today" mentioned earlier in this conversation rather than
  the real system clock, match that, don't silently switch conventions
  mid-file).
- One entry per distinct concern - if Step 2 turned up several unrelated
  things (e.g. a bug fix and a separate feature), write separate entries
  for each, in chronological order, each with its own `---` separator,
  rather than merging them into one.
- Append at the end of the file (assume chronological, oldest-first,
  ordering unless Step 1's read of the file clearly shows otherwise).

## Step 4 - Update the project state file

Match whatever ordering Step 1 found (most project-state files are
reverse-chronological - newest content right after a "Last updated"-style
line near the top, older entries further down; if this one isn't, follow
whatever ordering it does use instead).

1. Update the "last updated" line/date at the top to today's date (same
   date used in Step 3), with a short parenthetical pointing at what
   changed and, if there's more than a sentence of detail, a pointer back
   to the decisions log rather than duplicating it in full.
2. Insert a new paragraph (right after the "last updated" line, if this
   file is newest-first) summarizing the same work more concisely than
   the decisions-log entry - a few sentences per concern, not the full
   narrative. This is a *summary*, not a duplicate of Step 3's entry -
   cross-reference the decisions log for the full account rather than
   repeating it.
3. Don't touch anything below the new insertion - older content stays
   exactly where it already was.

## Notes

- These are local, informal notes (often gitignored on purpose in a
  project that keeps personal/portfolio-style detail out of its public
  repo) - editing them is a plain, easily-reversible file write, not a
  git operation, so this command just writes directly and reports what it
  added; it doesn't need a separate go-ahead the way pushing or opening a
  PR would.
- Never fabricate a specific (a test count, a file name, a branch/PR
  number, a root cause) that isn't actually grounded in something from
  Step 2 - if a detail isn't known precisely, describe it at the level of
  precision that's actually justified rather than inventing a number.
- Don't create `memory/`, a decisions-log file, or a project-state file
  from scratch if none exist - report that back to the user and let them
  decide whether/how they want one started, rather than guessing at a
  structure and format this project has never used.

$ARGUMENTS
