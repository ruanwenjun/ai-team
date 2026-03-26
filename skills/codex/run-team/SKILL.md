---
name: run-team
description: Use when launching AI team members to work on tasks, selecting a role to execute on an issue with an AI team initialized by init-team
---

# Run AI Team

## Overview

Launch a single team role on a selected issue. The user picks which issue and which role to run — there is no enforced execution order, no stage gates, and no approval flow.

**Prerequisite:** `.ai-team/` directory must exist (created by `init-team`).

**Interactive-only:** This skill accepts only the bare command `/run-team`.

If the user supplies any legacy role list, issue number, task text, or language flag inline, stop immediately, explain that the old form is deprecated, and tell them to rerun the bare command.

---

## Guided Flow

Follow this sequence in order:

1. Validate that `.ai-team/` exists and that `roles/` contains at least one role directory with a `prompt.md`.
2. List existing issues from `.ai-team/project/issues/`, or offer to create a new one.
   - New issue: collect task description, scan for the highest issue number (by directory name), increment, create `issues/{num}-{slug}/issue.md` with only the task description.
   - Existing issue: let the user choose one.
3. List all roles from `.ai-team/roles/`, showing which roles already have output in this issue.
   - If a role's subdirectory exists under the issue directory, mark it as "已有产出" (has output).
4. User picks one role.
5. Load role context and execute in the current conversation.

---

## Role Selection Display

```
Issue: #001 - user-login
团队角色：
  1. pm (Project Manager)
  2. architect (Architect) — 已有产出
  3. rd-1 (Developer)
  4. rd-2 (Developer) — 已有产出
  5. qa (QA Engineer)

请选择要运行的角色：
```

---

## Role Context Loading

Before executing, load context in this order:

1. **Role prompt** — `roles/{id}/prompt.md`
2. **Role profile** — `roles/{id}/profile.md`
3. **Role knowledge base** — `roles/{id}/templates/` and `roles/{id}/notes/` (relevant files)
4. **Issue content** — `issues/{num}-{slug}/issue.md` (user-maintained, read-only)
5. **Other roles' output** — `issues/{num}-{slug}/{other-id}/` (read-only reference)
6. **Collaboration rules** — `collaboration.md`

---

## Role Execution

1. Announce: "Now acting as {role-id} ({role-name})."
2. Read issue content and other roles' existing output as input.
3. Execute work per own prompt definition.
4. Write all output to `issues/{num}-{slug}/{id}/`.
5. Update `roles/{id}/profile.md` (learning record).
6. Optionally update `roles/{id}/templates/` or `roles/{id}/notes/` (accumulate reusable assets).
7. Display list of files produced.

No stage gate, no approval flow. User decides next step.

---

## Read/Write Rules

**Write:** Each role can only create and modify files in:
- `roles/{id}/` — own global knowledge base (prompt, profile, templates, notes)
- `issues/{num}-{slug}/{id}/` — own output in current issue

**Read:** Each role can read:
- `roles/*/` — all roles' knowledge bases
- `issues/{num}-{slug}/*` — all files in current issue
- `collaboration.md`, `team.md`, `project/README.md`

**One-line summary:** Globally readable, write only to own directories.

---

## Error Handling

| Condition | Action |
|---|---|
| Legacy argument-based invocation | Tell the user the form is deprecated and they must rerun bare `/run-team`. |
| `.ai-team/` directory missing | Tell the user: "No `.ai-team/` directory found. Run `init-team` first to set up your AI team." |
| `.ai-team/` exists but `roles/` missing or empty | Report the issue and suggest re-running `init-team`. |
| No issues exist and user wants to continue | Tell the user no issues found and offer to create a new one. |
| Selected role has no `prompt.md` | Report the missing file and suggest checking `roles/{id}/`. |
| Role execution fails | Report the failure and ask the user how to proceed. |
