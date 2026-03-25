---
name: update-team
description: Use when iteratively changing an existing AI team, adding or removing members safely, or refreshing team prompts and profiles without losing project history
---

# Update AI Team

## Overview

Safely update an existing `.ai-team/` without rewriting historical project artifacts. This skill is for iterative team maintenance after `init-team` has already created the workspace.

**Prerequisite:** `.ai-team/` directory must exist.

**Interactive-only:** This skill accepts only the bare command `/update-team`.

If the user supplies any legacy arguments, flags, or inline role changes, stop immediately and tell them the old form is deprecated and they must rerun the bare command.

---

## Trigger and Invocation Rules

### Supported Command

- `/update-team`

### Deprecated Legacy Forms

Treat all non-bare forms as deprecated, including examples such as:

- `/update-team add rd`
- `/update-team remove qa`
- `/update-team --lang zh`

Do not parse these as partial input or prefilled defaults. Show migration guidance and wait for the user to rerun `/update-team`.

### Language Preference

1. Ask for language preference at the start of every invocation.
2. If the current user input suggests a default language, you may recommend it.
3. Recommendation never replaces the explicit question. The user must still choose the language for this run.

---

## Guided Flow

Execute the following sequence in order:

1. Ask for language preference.
2. Confirm `.ai-team/` exists. If it does not, tell the user to run `init-team` first.
3. Read and display the current active team from `.ai-team/team.md`.
4. Enter an action loop with only these choices:
   - `add member`
   - `remove member`
   - `finish and review pending changes`
5. After every action, refresh the pending team preview before asking for the next action.
6. When the user chooses `finish and review pending changes`, show:
   - the resulting active team preview
   - the archive preview for any removed roles
   - any blocked removals that were prevented by safety checks
7. Ask for final confirmation before applying changes.
8. Apply changes only after explicit confirmation.

---

## Team Update Operations

### Add Member

When adding a member:

1. Ask for the role type to add.
2. Determine the next available stable role ID.
3. Add the member to the pending team state.
4. Mark the member as requiring a new worklog directory.

### Remove Member

When removing a member:

1. Ask which active role ID should be removed.
2. Validate that the role currently exists in the active team.
3. Inspect in-progress issues before allowing the removal.
4. If unfinished work still depends on that role, block the removal and explain what must be reassigned or completed first.
5. If removal is allowed, move the role into the pending archive set rather than deleting it from history.

### Finish and Review Pending Changes

When the user asks to review pending changes:

1. Show the pending active team after additions and removals.
2. Show the archive destinations for removed roles.
3. Show which active roles will be regenerated.
4. Show which new worklog directories will be created.
5. Show the changelog summary that will be appended on apply.

---

## Role ID Allocation

### Role IDs are never reused

When a role type is added after earlier removals, continue from the historical maximum suffix rather than filling gaps.

Examples:

- remove `rd-2`, then add another developer -> new ID is `rd-3`
- remove `qa`, then add QA again -> new ID is `qa-2`

For single-instance roles, preserve the original unsuffixed ID for the first active member and use suffixed IDs for later re-additions when history requires it.

---

## Safety Rules

### In-progress issues

Before removing any active role, inspect `.ai-team/project/issues/` for unfinished work.

Block removal if the role:

- is explicitly assigned in a stage that is not complete
- appears to own pending implementation or review work
- is still needed for the current stage of an unfinished issue

If blocked, instruct the user to reassign or finish that work before retrying `update-team`.

### Historical Preservation

Do not rewrite existing issue history, worklogs, or archived references. Team updates affect future work only.

### Full Refresh for Active Roles

After any confirmed update, regenerate `profiles/` and `prompts/` for every active role, not just added or directly modified ones.

---

## Apply Phase

After final confirmation, apply the team update in this order:

1. Rewrite `.ai-team/team.md`
2. Rewrite `.ai-team/collaboration.md`
3. Regenerate `.ai-team/profiles/{id}.md` for every active role
4. Regenerate `.ai-team/prompts/{id}.md` for every active role
5. Create `.ai-team/worklog/{id}/` for newly added roles
6. Archive removed roles under `.ai-team/archive/roles/{id}/`
7. Append a structured update entry to `.ai-team/project/changelog.md`

The `team.md` output should include a `Last Updated` field in addition to the original creation date.

---

## Archive Rules

Archive removed members under:

```text
.ai-team/archive/roles/{id}/
```

Each archived role keeps:

- `profile.md`
- `prompt.md`
- `worklog/`
- `archived.md`

The archive note should capture the removal date and enough context to explain why the role left the active team.

---

## Changelog Update

Each successful run appends a team update entry under `Unreleased` in `.ai-team/project/changelog.md` that records:

- added role IDs
- removed role IDs
- archived role IDs
- update date

---

## Validation Checklist

Before applying changes, confirm all of the following:

- `.ai-team/` exists
- language preference is explicitly chosen for this invocation
- every removal passed the in-progress issues safety check
- every added member received a non-reused stable ID
- the archive preview matches the pending removals
- the user explicitly approved the final preview

If any item fails, stop and resolve it before writing files.
