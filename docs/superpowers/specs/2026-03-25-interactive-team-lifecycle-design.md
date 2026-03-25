# Interactive Team Lifecycle Design

**Date:** 2026-03-25

## Goal
Add a dedicated `update-team` skill for iterative team changes and convert `init-team` plus `run-team` to interactive-only commands so team setup, execution, and maintenance all follow guided flows instead of argument-driven invocation.

## Approved Scope
- Add a new `update-team` skill for both Codex and Claude Code.
- Convert `init-team` to interactive-only usage.
- Convert `run-team` to interactive-only usage.
- Explicitly deprecate legacy argument-based usage for all three commands and return migration guidance instead of parsing old flags or role lists.
- Keep language selection mandatory at the start of each invocation, but collect it through the guided flow rather than command flags.
- Preserve project history during team updates while allowing role additions and removals.
- Archive removed roles instead of deleting their files.
- Update only the skill definitions and repository documentation for this change; do not regenerate or rewrite the example `.ai-team/` contents already present in the repository.

## Out of Scope
- Do not keep legacy argument-based examples in the README files as side-by-side reference material.
- Do not treat legacy arguments as prefilled defaults, partial input, or shortcuts inside any interactive flow.
- Do not modify existing example issues, prompts, profiles, worklogs, or other generated artifacts under the checked-in `.ai-team/` directory.

## Design Decisions

### Interactive-only command model
`init-team`, `run-team`, and `update-team` accept only bare command invocation:
- `/init-team`
- `/run-team`
- `/update-team`

Legacy forms such as `/init-team web-standard`, `/run-team all --task "..."`, `/run-team rd-1 --issue 001`, and any `--lang`, `--task`, or `--issue` flags are deprecated. The skills should not treat them as partial input or prefilled defaults. They should stop immediately and instruct the user to rerun the bare command.
README migration guidance should reflect the same policy: document bare-command usage only and describe older argument-based forms as deprecated.

### Dedicated `update-team` skill
Iterative team maintenance should not be merged into `init-team`. Initialization and in-flight team restructuring have different risk profiles and different preservation rules. `update-team` becomes the safe, explicit entry point for changing the team while keeping project artifacts intact.

### Operation-based update flow
`update-team` uses an incremental operation loop instead of a full desired-state replacement flow. The user edits the active team through repeated actions:
- add members
- remove members
- finish and apply

This keeps the workflow easy to reason about for small mid-project changes and makes the resulting diff obvious before writing files.

### Full refresh for active roles
After a team update, all active roles should have their `profiles/` and `prompts/` regenerated, not only newly added or directly modified roles. Team composition changes alter review relationships, escalation paths, and collaboration expectations, so a full active-team refresh is safer and more consistent.

### Archive removed roles by stable ID
Removed roles should be archived under `.ai-team/archive/roles/{id}/` rather than deleted. Each archived role keeps:
- its last profile
- its last prompt
- its full worklog directory
- an archive note with the removal date and context

This preserves references from old issues and lets users inspect historical team structure without reopening removed members as active roles.

### Role IDs are never reused
When a role type is added after earlier removals, the next active member gets the historical max suffix plus one. This avoids ambiguity in issues, worklogs, and archived references.

Examples:
- Remove `rd-2`, then later add another developer -> new ID is `rd-3`
- Remove `qa`, then later add QA again -> new ID is `qa-2`

## Command Flows

### `init-team`
`init-team` becomes a guided initializer with this sequence:
1. Ask for language preference.
2. Ask the user to choose a preset template or enter a custom composition.
3. Ask for project name.
4. Ask whether to customize role descriptions.
5. If `.ai-team/` already exists, offer only:
   - reinitialize and overwrite
   - switch to `update-team` for iterative changes
   - cancel
6. Show a summary preview and ask for final confirmation.

`merge` is removed from the interactive overwrite prompt because iterative updates now belong to `update-team`.

### `run-team`
`run-team` becomes a guided workflow launcher with this sequence:
1. Ask for language preference.
2. Ask whether to start a new task or continue an existing issue.
3. For a new task:
   - collect the task description
   - create a new issue
   - begin at PM requirement intake
4. For an existing issue:
   - list available issues
   - let the user choose one
   - infer the current stage from issue state
5. Offer only valid next actions for that issue:
   - continue to the next stage
   - rerun the current stage
   - apply targeted feedback to the current stage
6. Launch only the roles required by the current stage.

The user no longer chooses arbitrary role IDs at command invocation time. Role selection is controlled by the issue state and the documented stage workflow.

### `update-team`
`update-team` becomes a guided team-maintenance flow with this sequence:
1. Ask for language preference.
2. Validate that `.ai-team/` exists.
3. Display the current active team.
4. Enter an action loop:
   - add member(s)
   - remove member(s)
   - finish and review pending changes
5. Show the resulting team preview and the archive preview.
6. Ask for final confirmation before applying changes.
7. Apply changes:
   - rewrite `.ai-team/team.md`
   - rewrite `.ai-team/collaboration.md`
   - regenerate `profiles/` and `prompts/` for every active role
   - create worklog directories for newly added roles
   - archive removed roles under `.ai-team/archive/roles/{id}/`
   - append a team update entry to `.ai-team/project/changelog.md`

## Validation and Safety Rules

### Minimum runnable team for `run-team`
`run-team` should block execution unless the active team contains:
- at least one `pm`
- at least one `architect`
- at least one development role
- at least one `qa`

If any required role is missing, the skill should tell the user to fix the team through `update-team` before continuing.

### In-progress issue protection for `update-team`
Before removing an active role, `update-team` should inspect in-progress issues. If the role is still assigned in an unfinished stage or appears as the owner of pending work, the skill should block the removal and instruct the user to reassign or finish that work first.

### Historical issue preservation
Updating the team must not rewrite existing issue history. New members affect future tasks and future stage assignments, but old issue logs remain untouched. Archived roles remain discoverable through their archived files.

## Artifact Changes

### New archive structure
Add this preserved-history area:

```text
.ai-team/
└── archive/
    └── roles/
        └── {id}/
            ├── profile.md
            ├── prompt.md
            ├── worklog/
            └── archived.md
```

### Changelog updates
Each successful `update-team` run appends a structured entry to `.ai-team/project/changelog.md` under `Unreleased`, recording:
- added role IDs
- removed role IDs
- archived role IDs
- update date

### Team metadata
`team.md` should include a `Last Updated` field so team reconfiguration has a visible timestamp separate from the initial creation date.

## Files To Change
- `skills/codex/init-team/SKILL.md`
- `skills/claude-code/init-team/SKILL.md`
- `skills/codex/run-team/SKILL.md`
- `skills/claude-code/run-team/SKILL.md`
- `skills/codex/update-team/SKILL.md`
- `skills/claude-code/update-team/SKILL.md`
- `README.md`
- `README_zh.md`

## Verification Strategy
- Confirm `init-team`, `run-team`, and `update-team` are documented as interactive-only.
- Confirm legacy command examples with role lists and flags are removed from the READMEs and skill trigger sections.
- Confirm all three commands instruct the user to rerun the bare command when legacy arguments are supplied.
- Confirm `init-team` existing-directory handling now routes iterative changes to `update-team` instead of `merge`.
- Confirm `run-team` describes guided issue selection and stage-driven role dispatch rather than user-provided role lists.
- Confirm `update-team` documents the add/remove/apply loop, full active-role refresh, archive location, and non-reused IDs.
- Confirm `run-team` blocks when the active team lacks PM, architect, development, or QA coverage.
- Confirm `update-team` blocks role removal when unfinished issues still depend on that role.
