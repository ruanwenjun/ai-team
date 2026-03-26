---
name: run-team
description: Use when launching AI team members to work on tasks, guiding issue-based stage execution, or managing iterative task acceptance with an AI team initialized by init-team
---

# Run AI Team

## Overview

Launch team members through a guided, issue-driven workflow. This skill creates or resumes issues, dispatches only the roles needed for the current stage, and keeps user approval gates between PM intake, architect planning, implementation, QA, architect final review, and PM acceptance.

When the current platform exposes relevant external Superpowers skills, roles may use them as an enhancement to the existing AI Team stage workflow. If those external skills are unavailable, continue the normal AI Team workflow and do not simulate a local fallback.

**Prerequisite:** `.ai-team/` directory must exist (created by `init-team`).

**Interactive-only:** This skill accepts only the bare command `/run-team`.

If the user supplies any legacy role list, issue number, task text, or language flag inline, stop immediately, explain that the old form is deprecated, and tell them to rerun the bare command.

---

## Trigger and Guided Launch Flow

### Supported Command

- `/run-team`

### Guided Flow

Follow this sequence in order:

1. Ask for language preference before any stage work begins.
2. Validate that `.ai-team/` exists and that required team files are present.
3. Validate the active team contains at least one pm, at least one architect, at least one development role, and at least one qa.
4. If any required role category is missing, stop and tell the user to fix the team through `update-team` before continuing.
5. Ask whether to `start a new task` or `continue an existing issue`.
6. For a new task:
   - collect the task description
   - create a new issue
   - begin at PM requirement intake
7. For an existing issue:
   - list available issues
   - let the user choose one
   - infer the current stage from the issue state
8. Offer only valid next actions for that issue:
   - `continue to the next stage`
   - `rerun the current stage`
   - `apply targeted feedback to the current stage`
9. Display the roles involved in the current stage with their current status (`pending`, `in-progress`, `done`), and let the user pick one `pending` role to run in the current conversation.
   - If only one `pending` role exists, auto-select it (with confirmation).
   - Selecting an `in-progress` role requires explicit takeover or rerun confirmation.
10. Launch the selected role in the current conversation.

### Legacy Command Handling

Treat any argument-based form that includes role lists, task text, issue selection, or language flags as deprecated.

Do not parse them as partial input or defaults. Tell the user to rerun the bare command.

### Language Preference Resolution

- Always ask for language preference before the first stage starts.
- You may recommend a default based on the current user input.
- Recommendation never replaces the explicit language question.
- The selected language applies only to newly generated or appended content in the current invocation.

---

## Operating Modes

### Single-Session Mode

Used when one conversation advances the issue through one or more stages, one role at a time.

1. Launch only one role per invocation; the user picks which role to run.
2. After the role completes, show the role-progress report.
3. If all roles in the stage are `done`, auto-generate a stage summary report.
4. Stop and wait for explicit user approval before starting the next stage.

### Multi-Session Mode

Used when the user runs `/run-team` in separate terminals or separate conversations while continuing the same team workflow.

1. Each session still starts from the bare `/run-team` command.
2. The user selects the relevant existing issue through the guided flow.
3. Coordination remains file-based through issue files, per-role stage status, and worklogs.
4. Within the current stage, each assigned role is tracked individually as `pending`, `in-progress`, or `done`.
5. A session should claim only `pending` roles by default. Re-running or taking over an `in-progress` role requires an explicit user choice.
6. No stage may be skipped; later stages still require explicit user approval after every role in the current stage is `done`.

### Default Stage Order

The documented workflow is always:

1. PM requirement intake and clarification.
2. Architect planning, decomposition, and assignment.
3. Implementation by the assigned developer roles.
4. QA review by the assigned QA role.
5. Architect final technical review.
6. PM acceptance.

Each handoff requires explicit user approval before the next stage starts. If the user asks for changes, relaunch only the role(s) in the current stage or the specific stage being revised.

If architect assigns multiple developers in the implementation stage, the implementation stage stays open until every assigned developer role is `done`. QA cannot begin until every assigned developer role is `done` and the user explicitly approves moving forward.

### Role Selection

Before launching a stage, display the roles involved and let the user pick one role to run.

**Display format example:**

```
当前阶段：Implementation
涉及角色：
  1. rd-1 (Developer) — pending
  2. rd-2 (Developer) — pending

请选择要运行的角色：
```

If only one `pending` role exists, auto-select it and confirm with the user.

**Execution rules:**

When running a role in the current session:

1. Read the role's prompt, profile, issue content, collaboration guidelines.
2. Load the relevant Superpowers skill via the Skill tool (if available).
3. Announce: "Now acting as {role-id} ({role-name})."
4. Before starting the role's actual work, update the issue file to mark that role `in-progress`.
5. Follow the role's prompt and Superpowers skill instructions to execute the work.
6. Write the worklog entry and update the issue file.
7. Mark the role `done` in the issue file.
8. Update the role's profile.
9. If unfinished roles remain in the current stage, present the remaining role statuses and stay in the current stage. If no unfinished roles remain, present the full stage summary and enter the stage gate.

### Role-Level Stage State

Track each assigned role inside the current stage using this lifecycle:

- `pending` — not started yet; claimable by any session
- `in-progress` — claimed by one session; not claimable by another session unless the user explicitly requests a rerun or takeover
- `done` — role finished its work for the current stage

Before launching any role, re-read the issue file and verify the role is still in the expected state. If another session already changed it, stop, refresh the current stage state, and show the user the latest role statuses.

**Stage gate behavior** — enter the stage gate only after every role assigned to the current stage is `done`. A single-role run should not close the stage unless it completed the last unfinished role.

### Optional External Superpowers Usage

When the current platform exposes the relevant external Superpowers skill, roles should use it for the matching stage:

- PM requirement intake -> `brainstorming`
- Architect planning -> `writing-plans`
- Implementation -> `test-driven-development`
- QA sign-off -> `verification-before-completion`

If the skill is not exposed, continue the normal AI Team stage workflow. Do not invent a local replacement.

### Bug-Oriented Implementation Rule

If the task, stage feedback, or issue history shows the implementation stage is addressing a bug, regression, or production issue:

- the assigned developer roles should use external `systematic-debugging` before implementation when that skill is available
- the debugging evidence should be summarized in the issue and worklog before the implementation summary
- if `systematic-debugging` is unavailable, continue the normal AI Team debugging and implementation flow

---

## Issue Management

### New Issue Creation

When the user chooses to start a new task:

1. Scan `.ai-team/project/issues/` for the highest existing issue number (by directory name).
2. Increment by one.
3. Create the issue directory at `.ai-team/project/issues/{number}-{slug}/`.
4. Create the issue file at `.ai-team/project/issues/{number}-{slug}/issue.md`.

### Existing Issue Resume

When the user chooses to continue an existing issue:

1. List the existing issue directories from `.ai-team/project/issues/`.
2. Ask the user to choose one.
3. Read the issue file at `.ai-team/project/issues/{number}-{slug}/issue.md`.
4. Infer the current stage and valid next actions from the recorded stage status and progress.

### Issue File Structure

```markdown
# {number} - {title}

## Task
{Original task description from user}

## Status: in-progress

## Assigned: pm, architect

## Stage Status
- PM requirement intake: approved
- Architect planning: approved
- Implementation: in-progress
- QA review: blocked until implementation is approved
- Architect final review: pending
- PM acceptance: pending

## Stage Role Status

### Stage 1 - PM requirement intake
- pm: done

### Stage 2 - Architect planning
- architect: done

### Stage 3 - Implementation
- rd-1: done
- rd-2: pending

### Stage 4 - QA review
- qa: pending

## Progress

### Stage 1 - PM requirement intake - {date}
- **pm** (model: claude-opus-4-6): {requirement clarification summary}
- **User Feedback**: approved for architect planning

### Stage 2 - Architect planning - {date}
- **architect** (model: claude-opus-4-6): {task breakdown and role assignment summary}
- **Assigned Developers**: rd-1
- **Assigned QA**: qa
- **User Feedback**: approved for implementation

### Stage 3 - Implementation - {date}
- **rd-1** (model: claude-opus-4-6): {implementation summary}
- **Stage Progress**: rd-1 done, rd-2 pending

## Status: in-progress
```

### Rules

- Each stage records overall stage status, per-role stage status, agent work summaries, model used, stage-specific assignments, and user feedback.
- When continuing an existing issue, read the full issue file and inject its history into agent context.
- A stage may advance only after every role in its current `Stage Role Status` block is `done` and the user approves that stage.
- In multi-session mode, mark roles `in-progress` before launch so other sessions can avoid double-claiming them.
- Race condition caveat: in multi-session mode, two sessions may update the same issue file concurrently. Re-read the issue file before claiming a role, keep updates append-only where possible, and refresh role status after any collision.

---

## Role Context Assembly

Before executing a role, assemble the following context in the current conversation:

1. **Role prompt** from `.ai-team/prompts/{id}.md`, plus **file attribution rules** (see File Attribution Rules below)
2. **Role profile** from `.ai-team/profiles/{id}.md` (read for context; the role will update it after completing work)
3. **Full issue content** (task description + all stage history)
4. **Collaboration guidelines** from `.ai-team/collaboration.md`
5. **Superpowers skill content** (if available — see below)

When continuing an existing issue, the full issue history is loaded so the role has complete context continuity across stages.

### Superpowers Skill Loading

Before executing a role, load the relevant Superpowers skill for the current stage using the Skill tool.

**Stage-to-skill mapping:**

| Stage | Skill to load | Skill tool name |
|-------|--------------|-----------------|
| PM requirement intake | brainstorming | `superpowers:brainstorming` |
| Architect planning | writing-plans | `superpowers:writing-plans` |
| Implementation | test-driven-development | `superpowers:test-driven-development` |
| Implementation (bug/regression) | systematic-debugging | `superpowers:systematic-debugging` |
| QA sign-off | verification-before-completion | `superpowers:verification-before-completion` |

If the Skill tool invocation fails or the skill is not installed, continue with the normal AI Team workflow — do not block the stage.

---

## Workflow Execution

Execute these steps in order:

1. **Validate environment:**
   - Confirm `.ai-team/` directory exists. If not, tell the user to run `init-team` first.
   - Confirm key files exist: `.ai-team/team.md`, `.ai-team/prompts/`, `.ai-team/profiles/`, and `.ai-team/project/issues/` (issues are directories, not files).
   - Confirm the active team contains at least one pm, at least one architect, at least one development role, and at least one qa.
   - If the team is incomplete, stop and tell the user to fix the team through `update-team`.
2. **Resolve language and issue intent:**
   - Ask for language preference.
   - Ask whether to start a new task or continue an existing issue.
3. **Create or read issue:**
   - For a new task: collect task description, scan for the highest issue number (by directory name), increment, create the issue directory, and write `issue.md` inside it.
   - For an existing issue: list existing issue directories, read the selected `issue.md`, and infer the current stage plus the per-role state for that stage.
4. **Determine valid next action:**
   - If the current stage still has `pending` roles, offer only `run a remaining role`, `rerun/take over a role`, or `apply targeted feedback to the current stage`.
   - If the current stage has only `in-progress` and `done` roles, offer only `wait for in-progress roles`, `rerun/take over a role`, or `apply targeted feedback to the current stage`.
   - If every role in the current stage is `done`, offer only `continue to the next stage`, `rerun the current stage`, or `apply targeted feedback to the current stage`.
   - Use the issue state to decide which stage is eligible to run next.
5. **Display roles and select one:**
   - Show the roles involved in the current stage together with their current status (`pending`, `in-progress`, `done`).
   - Let the user pick one `pending` role to run. If only one `pending` role exists, auto-select it with confirmation.
   - Selecting an `in-progress` role must be treated as an explicit takeover or rerun.
   - See "Role Selection" above for the display format.
6. **Load Superpowers skill for the current stage:**
   - Determine which Superpowers skill maps to the current stage (see the stage-to-skill mapping table in Superpowers Skill Loading).
   - Use the **Skill tool** to load the relevant skill content in the main conversation.
   - If the Skill tool invocation fails or the skill is not installed, proceed without it.
7. **Execute the selected role in the current conversation:**
   - Claim the role by marking it `in-progress` in the issue file before execution.
   - Read the role's prompt, profile, issue content, and collaboration guidelines.
   - Announce "Now acting as {role-id} ({role-name})."
   - Follow the role's prompt and loaded Superpowers skill to execute the work directly in the current conversation.
8. **Role execution:**
   - Each role writes a worklog entry to the current issue directory at `.ai-team/project/issues/{number}-{slug}/{id}-stage-{n}-{stage-name}.md`.
   - Each role updates the issue file (`issue.md` in the same directory) by appending work summary, listing all files created or modified under the current stage's Progress section, and marking its role status `done`.
   - Each role updates its own profile at `.ai-team/profiles/{id}.md`.
   - Each role reports the model it is running on (best-effort).
9. **Collect results:**
   - If unfinished roles remain in the stage, generate a role-progress report showing which roles are `done`, `pending`, and `in-progress`.
   - If every role in the stage is `done`, generate the full stage summary report and update the issue file with the stage results.
10. **Prompt user to review:**
   - After each role run, display the files written by that role and the current role-status table for the stage.
   - After the current stage completes, display a checklist of all files in the issue directory and updated profiles for each agent in that stage.
11. **Enter the stage gate:**
   - Wait for explicit user approval before launching the next stage, but only after every role in the current stage is `done`.

---

## Model Reporting (Best-Effort)

Model identification is best-effort and may not always be accurate.

- **Attempt detection** from the runtime environment or API response metadata.
- **Inject into subagent context** if detected, so agents can include it in their worklogs.
- **Fallback:** the agent self-reports its model name. This may be inaccurate — note this caveat in output.
- **Report locations:**
  - startup announcement
  - worklog entry
  - summary report

---

## File Attribution Rules

Inject these rules into each agent's context. Agents must follow them when creating or modifying files.

### Single-Role Files

Add a metadata comment header at the top of the file, using the file's native comment syntax:

- **Markdown/HTML:** `<!-- Generated by: rd-1 (Developer) | Issue: #001 | Date: 2026-03-25 -->`
- **JavaScript/TypeScript:** `// Generated by: rd-1 (Developer) | Issue: #001 | Date: 2026-03-25`
- **Python:** `# Generated by: rd-1 (Developer) | Issue: #001 | Date: 2026-03-25`
- **CSS:** `/* Generated by: rd-1 (Developer) | Issue: #001 | Date: 2026-03-25 */`
- **Shell:** `# Generated by: rd-1 (Developer) | Issue: #001 | Date: 2026-03-25`

### Multi-Role Files

When multiple agents modify the same file, use boundary comments around each agent's section:

- **JavaScript/TypeScript:**
  ```
  // BEGIN rd-1 (Developer) | 2026-03-25
  ...code...
  // END rd-1
  ```
- **Python:**
  ```
  # BEGIN rd-1 (Developer) | 2026-03-25
  ...code...
  # END rd-1
  ```
- **HTML/Markdown:**
  ```
  <!-- BEGIN rd-1 (Developer) | 2026-03-25 -->
  ...content...
  <!-- END rd-1 -->
  ```

### No-Comment Files

For files that do not support comments (JSON, binary, images, etc.), record the attribution in the worklog entry only. Do not modify these files to add attribution.

---

## Summary Report

Generated after each role run in single-session mode. When a stage still has unfinished roles, present a role-progress report instead of a final stage summary.

### Structure

- **Per agent:** What they did, files produced or modified, model used.
- **Stage role status:** Which roles are `done`, `pending`, or `in-progress`.
- **Overall:** Unresolved issues, risks, or items needing attention.
- **Next steps:** Suggested actions if applicable.

If every role in the current stage is `done`, present the summary report directly to the user in the conversation, then enter the stage gate. Otherwise present the role-progress report and keep the workflow in the current stage.

---

## Stage Gate and Acceptance Flow

An iterative loop within the current conversation (single-session mode only):

1. Enter the stage gate only when every role in the current stage is `done`.
2. Present the current stage summary report to the user.
3. Interpret the user's response using LLM judgment:
   - **Approval** for the current stage marks that stage complete. If another stage remains, wait for the user to continue to the next stage. If the current stage is PM acceptance, mark the issue status as `done` and end the workflow.
   - **Rerun request** keeps the issue at the same stage, resets the selected role(s) from `done` or `in-progress` back to `pending`, and relaunches only those role(s).
   - **Targeted feedback** appends feedback to the issue file, resets the affected role(s) to `pending`, and relaunches only the role(s) in the current stage or the stage the feedback applies to.
4. Selective relaunch rules:
   - Relaunched agents receive the full issue history including the new feedback.
   - Results from earlier approved stages remain untouched.
   - If one of several assigned developers needs revision, keep the workflow in the implementation stage until every assigned developer role returns to `done` and the user approves the implementation stage as a whole.
5. No extra commands are needed; the skill keeps the issue context active within the loop.
6. The loop continues until PM acceptance is explicitly approved after architect final review.

---

## Worklog Entry Template

Each agent writes a worklog entry to `.ai-team/project/issues/{number}-{slug}/{id}-stage-{n}-{stage-name}.md`:

```markdown
# {Issue Number} - {Brief Description}

- **Role**: {role name} ({id})
- **Model**: {model name}
- **Date**: {date}
- **Issue**: #{issue number}

## What Was Done
- {summary of work performed}

## Files Modified
- `path/to/file.js` — {what was changed}

## Issues Encountered
- {any problems or blockers, or "None"}

## Notes
- {any additional context}
```

---

## Error Handling

Handle these error cases with clear, actionable messages:

| Condition | Action |
|---|---|
| Legacy argument-based invocation | Tell the user the form is deprecated and they must rerun bare `/run-team`. |
| `.ai-team/` directory missing | Tell the user: "No `.ai-team/` directory found. Run `init-team` first to set up your AI team." |
| `.ai-team/` exists but key files missing (`team.md`, `prompts/`, `profiles/`) | Report the specific missing files and suggest re-running `init-team`. |
| Active team missing required coverage | Tell the user the team must include at least one pm, at least one architect, at least one development role, and at least one qa; direct them to `update-team`. |
| Existing issue selection is invalid | List existing issue directories in `.ai-team/project/issues/` and ask the user to pick one. |
| No claimable roles remain in the current stage | Show the current role-status table and tell the user the stage is waiting on `in-progress` roles or ready for approval if all roles are `done`. |
| Target role is already `in-progress` in another session | Explain that the role is currently claimed, then offer only wait, explicit takeover, or targeted feedback. |
| Concurrent issue update changes role status during claim | Re-read the issue file, show the refreshed role-status table, and ask the user which remaining role to run now. |
| Stage cannot advance yet | Explain which earlier approval or unfinished work is still blocking the next stage. |
| Role execution fails | Report the failure, record it in the issue file, and ask the user how to proceed. |
