---
name: commit-norms
description: >
  Git commit conventions: when it's okay to commit, commit message style, and how to propose
  squashing commits. ALWAYS invoke this skill before running `git commit` for the first time in a
  session, and before proposing any commit squash — regardless of whether the user has explicitly
  asked about commit conventions. This skill MUST run before any `git commit` command is executed.
---

# Commit norms

## When to commit

- Never commit unprompted. Either the user has explicitly asked you to commit, or you have asked
  for and received approval first.
- This rule applies unconditionally, including when invoked by other skills or automated callers
  — there is no unattended-mode exception.
- To ask: propose the exact files being committed and the commit message, then wait for approval
  before running `git commit`.

## Commit messages

- Use imperative mood: "Add foo", "Fix bar", "Refactor baz" — not "Added" or "Fixing".
- Be clear and concise. Each message should explain the change being made.
- One logical change per commit — don't bundle unrelated edits into a single commit.

## Squashing

- When a branch has many small commits, or a PR contains a large number of commits, proactively
  propose a squash.
- Every squash proposal must include:
  1. **Which commits to squash** (list them by short hash + subject).
  2. **Why** they belong together (e.g., "all fix-ups for the same logic", "iterative tweaks to
     one function").
  3. **The proposed resulting commit message** — so the user can accept with a single "yes".
