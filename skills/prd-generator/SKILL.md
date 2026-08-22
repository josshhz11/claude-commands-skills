---
name: prd-generator
description: Generates a Requirements doc, a numbered file-level Plan, and a per-phase Definition-of-Done checklist for a new feature or project. Use when the user wants to scope out a build before writing code, or explicitly asks for a PRD, project plan, roadmap, or spec.
---

When the user wants to plan a new feature or project, produce **three
separate artifacts**, never one combined document. Each is its own output
block, clearly labeled, ready to be saved as its own file.

## 1. Requirements (the what/why)

Use `templates/requirements-template.md` as the structure. Cover: purpose,
goals, explicit non-goals, constraints, and success criteria. Ask the user
for anything genuinely required to fill this in (scope, tech constraints,
timeline) rather than inventing specifics — but make a reasonable default
assumption and state it plainly rather than blocking on a clarifying
question when the answer doesn't change the shape of the doc.

## 2. Plan (numbered, file-level tasks)

Use `templates/plan-template.md`. Break the work into phases; within each
phase, numbered tasks concrete enough that Claude Code can execute one
without re-deriving intent — name the actual file(s) touched, not just a
vague description of the phase's goal. Order phases so each one produces
something that actually runs, not just partial scaffolding.

## 3. DoD / Checklist (per phase)

Use `templates/checklist-template.md`. Each phase gets binary, checkable
conditions — "the CLI reads a file and prints its contents" not "file
reading works well." A checklist item should be verifiable by running
something, not by judgment call.

## Output format

Three fenced markdown blocks in the same response, in this order:
Requirements, Plan, Checklist. Don't merge them, don't add a fourth
combined summary — the separation is the point, so each can be saved and
handed to Claude Code as its own file.

If the user's request is small enough that a full three-document split
would be overkill (e.g. a single well-scoped function), say so and offer
a lighter single-paragraph plan instead of forcing the template.