---
name: pr-creation
description: >
  Creates pull requests following project conventions. Auto-generates a concise PR title and a
  structured What/Why description. ALWAYS invoke this skill whenever the user asks
  to create, open, raise, submit, push up, send out, file, or cut a pull request / PR - regardless
  of phrasing surrounding verbs, or compound instructions (e.g. "commit and open a PR", "push it up
  as a PR", "let's PR this", "create a pull request", "open a PR for this", "raise a PR", "cut a
  PR"). If the user's request includes any intent to put the current branch up for review on GitHub,
  this skill MUST run before `gh pr create` is called.
scope: individual
author: danyanos
---

# PR Creation Skill

Creates pull requests with standardized naming and description format.

## Unattended mode

When invoked by an automated caller (e.g. the `pr-loop` skill) that explicitly requests unattended
mode, one checkpoint changes:

- **Step 4 (Confirm with the user)**: skip entirely – go directly from step 3 (rebase) to step 5
  (push and create). The title/description generated in steps 2-3 are used as-is.

Every other step is unchanged. In particular, a rebase conflict in step 3 still stops the skill and
surfaces the failure rather than attempting to resolve it – that rule doesn't depend on whether a
human is watching.

## Instructions

### 1. Gather context

- **Diff**: the changes on the current branch versus the base branch
- **Conversation**: scope, decisions, and validation notes from the current session

### 2. Generate the PR title

Format: `{Brief description}` – no prefix or leading punctuation.

- Under 60 chars
- Derive from the diff and conversation directly

Example: `Remove dead mirrored_tables module and dbt-manifest pipeline`

### 3. Generate the PR description

The PR description is a roadmap for the diff, **not** a substitute for it. Aim for a 30-second
reviewer skim. If a reviewer wants implementation detail, they read the code.

#### Required sections

##### `## What`

A prose summary of what changed, written for a reviewer with **no prior context on this work** –
assume they weren't in any design discussion and work on a completely different track. GitHub's own
"Files changed" tab is already the bulleted, file-level view; this section exists to do what that
view can't: explain what the change *is*, in plain language, before the reviewer opens a single
file.

- 2-4 sentences. Describe the shape of the change at a conceptual level – what capability, resource,
  or behavior now exists that didn't before.
- Write it so someone unfamiliar with the internal names involved still understands the change.
  Spell out acronyms or internal terms on first use if an outside reviewer wouldn't know them.
- No file paths, filenames, or inline code/snippets. Describe the resulting capability or behavior,
  not the artifact that implements it – "adds request-level rate limiting" not "adds a `RateLimiter`
  class in `middleware.py`."
- Bullets are the exception, not the default – use them only when the PR bundles genuinely distinct,
  unrelated changes (e.g., a feature plus an unrelated dependency bump) that don't fit one
  narrative. Even then, each bullet should describe the change in a phrase, not just name the
  resource ("adds a read-only proxy role for cross-schema access", not "proxy role").

##### `## Why`

The business or product motivation. **Lead with the user need / outcome**, not the implementation
puzzle.

- 1-4 sentences. Keep it short.
- Good shape: *"Team X needs Y so they can Z. We chose A over B because [trade-off]."*
- Mention implementation rationale (provider version quirks, workarounds, library choices) **only
  if** it materially shaped the design and a reviewer needs it to evaluate the approach. Otherwise,
  leave it out – the diff and code comments are the right home for that.

#### Language

Write in industry-standard engineering terminology, not vocabulary specific to this conversation. If
a term, abbreviation, or framing only makes sense because of how the problem was discussed in this
session – rather than how an engineer unfamiliar with the session would name the change – replace it
with the standard term. A reviewer should not be able to tell the description was written by
relaying an agent conversation.

#### Optional sections (include only when relevant)

```
## Validation
[Testing/validation steps performed. Include screenshots the user shared.]

## Additional
[Anything else that doesn't fit above – e.g., closest existing analogs, follow-up work.]
```

#### What to leave out

These details belong in the code, the diff, or code comments – not the PR description:

- **Full Terraform / class / function names** with prefixes/suffixes
  (`${env}_BAPPS_DEPLOYMENT_ROLE`, `BappsServiceUserModule`). Use the category name (`deployment
  role`).
- **Grant taxonomies and privilege lists** (`CREATE USER + CREATE ROLE on account, USAGE WITH GRANT
  OPTION on warehouse`). The grant resources are right there in the diff.
- **Provider-version workarounds and mechanism explanations** (e.g., "the v0.73.0 provider doesn't
  expose `with_admin_option`, so we…"). Put this in a code comment next to the workaround, not in
  the PR body.
- **Side changes that aren't the feature** (CI script tweaks, lint fixes, dependency bumps that came
  along for the ride). Mention briefly in `## Additional` if non-obvious; otherwise omit.
- **Environment enumerations** (`ops → staging → dim1 → pmu → prod`) unless rollout order is itself
  the change being reviewed.

**Do not** mention AI, Claude, Claude Code, or any AI-assisted workflow in the title or description.

#### Worked example – verbose vs. tight

A multi-resource Snowflake infra PR.

❌ **Verbose (avoid):**

> ## What
>
> Provisions per-env Snowflake infrastructure for the BApps team: `${env}_BAPPS` database,
> `${env}_BAPPS_WH` XSMALL warehouse, `${env}_BAPPS_DEPLOYMENT_ROLE` (granted `CREATE USER` +
> `CREATE ROLE` on the account, `CREATE SCHEMA` + `USAGE WITH GRANT OPTION` on the database, `USAGE
> WITH GRANT OPTION` on the warehouse), `${env}_BAPPS_DATA_READER` proxy role bundling READ access
> to `BUILT_CORE` / `BUILT_ANALYTICS_MARTS` / `BUILT_ANALYTICS_PREP` (ownership transferred to the
> deployment role), and `${env}_BAPPS_SERVICE_USER` via `modules/service_user`. Also exempts
> ownership-grant resource types from the missing-grant detector since the v0.73.0 provider doesn't
> expose `enable_multiple_grants` on those resources.
>
> ## Why
>
> The BApps team needs a self-service surface for AI infrastructure without a Data Platform PR per
> change. Isolation is required so BApps cannot create or modify objects in any existing Built
> database/schema, and a dedicated warehouse is required for compute cost attribution. The
> proxy-role pattern works around v0.73.0's lack of `with_admin_option` on `snowflake_role_grants`,
> letting BApps re-grant the read bundle to their runtime roles without giving them the underlying
> access roles.

✅ **Tight (do this):**

> ## What
>
> This PR provisions isolated per-environment Snowflake infrastructure for the BApps team – a
> dedicated database and warehouse, a deployment role scoped to that database, and a read-only proxy
> role bundling access to the core/marts/prep schemas so BApps services can query existing data
> without holding direct grants on it.
>
> ## Why
>
> The BApps team needs the ability to create Snowflake infrastructure as they look to deploy
> applications for internal users. By creating a deployment role that gives them the ability to
> create resources in their own database, BApps can move quickly while still keeping guardrails
> around existing infrastructure. The dedicated warehouse (which BApps cannot create more of) gives
> us proper cost attribution.

The tight version is faster to read, ages better as the code changes, and answers the only two
questions a reviewer actually has: *what landed* and *why*.

### 4. Rebase onto `main`

1. `git fetch origin main`
2. `git log HEAD..origin/main --oneline` – is the feature branch behind?
3. If behind:
   - `git checkout main && git pull origin main`
   - `git checkout {FEATURE_BRANCH} && git rebase main`
4. On conflicts, stop and surface to the user. Do not auto-resolve.
5. Track whether a rebase happened – step 6 needs to know.

### 5. Confirm with the user

Show the generated title and description. Wait for approval before any push.

*Skipped entirely in unattended mode – see [Unattended mode](#unattended-mode).*

### 6. Push and create the PR

1. Push:
   - Normal: `git push -u origin {branch_name}`
   - After a rebase: `git push --force-with-lease origin {branch_name}`
2. Create the PR:

   ```bash
   gh pr create --base main --title "{title}" --body "$(cat <<'EOF'
   {description}
   EOF
   )"
   ```
3. On push failure, report the error. Do **not** force push.
4. Return the PR URL.
