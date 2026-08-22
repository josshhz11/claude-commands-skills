---
description: Draft and create a new branch following the <type>/<YY.MM>_<slug> convention (default type: feature).
argument-hint: [type:] <short description of the work>
---

Create a new branch following this convention:

<type>/<YY>.<MM>_<slug>

- `<type>` — defaults to `feature`. If $ARGUMENTS starts with a recognized
  type followed by a colon (`bugfix:`, `hotfix:`, `refactor:`, `docs:`,
  `test:`, `chore:`), use that type instead and strip it from the
  description.
- `<YY>.<MM>` — current year/month, two digits each, computed at runtime
  from the system date — never reuse a date mentioned earlier in the
  conversation.
- `<slug>` — the remaining description, lowercase, words separated by
  hyphens (not underscores — that's reserved for the date/slug boundary),
  specific enough to be meaningful on its own: `feature/26.08_prd-generator-skill`,
  not `feature/26.08_updates`.

## Steps

1. Compute `<YY>.<MM>` and build the slug from $ARGUMENTS as above.
2. Run `git status`, `git branch --show-current`, and
   `git branch --list "<type>/<YY>.<MM>_*"` to confirm the starting point
   and check nothing with this exact name exists yet this month.
3. If an identical branch name already exists, say so and stop — don't
   silently append a suffix.
4. Otherwise, run `git checkout -b <type>/<YY>.<MM>_<slug>` directly — no
   go-ahead needed. Creating a local branch is safe and easily reversible,
   unlike the push/PR-gated actions in `commit-pr.md`, so this one doesn't
   wait for confirmation.
5. Confirm with `git branch --show-current` and report the branch name.

$ARGUMENTS