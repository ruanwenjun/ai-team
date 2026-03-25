---
name: run-team
description: Use when launching AI team members to work on tasks, dispatching role-specific agents, or managing iterative task acceptance with an AI team initialized by init-team
---

# Run AI Team

## Overview

Launch team members as subagents to work on tasks. Creates issues, dispatches agents with role-specific prompts, collects results, and manages an architect-led, user-gated stage loop until PM acceptance is complete.

**Prerequisite:** `.ai-team/` directory must exist (created by `init-team`).

---

## Trigger and Argument Parsing

### Command Formats

- `/run-team pm,architect --task "implement user login"` — start the gated intake and planning chain for a task
- `/run-team all --task "..."` — start all roles listed in `.ai-team/team.md`
- `/run-team pm,architect` — interactive mode, ask the user for a task description
- `/run-team rd-1 --issue 001` — continue working on an existing issue

### Rules

- **Task is always required.** If `--task` is not provided, prompt the user interactively.
- **Project name** is read from `.ai-team/team.md`.
- **Role IDs** are comma-separated, matching IDs defined in `.ai-team/team.md`.
- **`all`** expands to every role listed in `.ai-team/team.md`.
- **`--issue {number}`** resumes an existing issue instead of creating a new one. The full issue history is injected into each agent's context for continuity.

---

## Operating Modes

### Single-Session Mode

Used when multiple roles are launched in one command within a single conversation.

1. Launch only the roles for the current stage, not the whole workflow at once.
2. If concurrent Agent calls are unsupported, fall back to sequential execution for that stage.
3. After the stage completes, auto-generate a stage summary report.
4. Stop and wait for explicit user approval before starting the next stage.

### Multi-Session Mode

Used when roles are launched in separate terminals or separate commands.

1. Each terminal runs `/run-team {role} --task "..." ` or `/run-team {role} --issue {number}` independently.
2. Coordination is file-based: agents check issue files for updates and write worklogs independently.
3. No automatic summary — the user reviews files in `.ai-team/project/` manually.
4. The same stage order and approval gates still apply; do not start a later stage until the previous stage has explicit user approval.
5. In a new issue, start with `pm` (or `pm,architect` in one terminal) rather than launching developers or QA directly. Later stages should resume the approved issue with `--issue {number}`.

### Default Stage Order

The documented workflow is always:

1. PM requirement intake and clarification.
2. Architect planning, decomposition, and assignment.
3. Implementation by the assigned developer roles.
4. QA review by the assigned QA role.
5. Architect final technical review.
6. PM acceptance.

Each handoff requires explicit user approval before the next stage starts. If the user asks for changes, relaunch only the role(s) in the current stage or the specific stage being revised.
If architect assigns multiple developers in the implementation stage, they all work within the same gated stage. QA cannot begin until every assigned developer has completed their work and the user explicitly approves moving forward.

### Mode Detection

There is no explicit flag. The mode is determined by usage pattern:

- Multiple roles in one command = single-session mode.
- Separate commands in separate terminals = multi-session mode.

**Note:** Issue numbering uses scan-and-increment. A race condition is possible in multi-session mode when two terminals create issues simultaneously, but this is rare in practice.

---

## Issue Management

### Auto-Creation

Scan `.ai-team/project/issues/` for the highest existing issue number and increment by one. Create the issue file at `.ai-team/project/issues/{number}-{slug}.md` (e.g., `001-implement-user-login.md`).

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
- Implementation: ready after architect assignment
- QA review: pending architect assignment
- Architect final review: pending
- PM acceptance: pending

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
- **rd-2** (model: claude-sonnet-4-6): {implementation summary if assigned}
- **User Feedback**: approved for QA review

## Status: done
```

### Rules

- Each stage records: agent work summaries, model used, stage-specific assignments, and user feedback.
- When `--issue` is used, read the existing file and inject its full content into agent context.
- Race condition caveat: in multi-session mode, two agents may update the same issue file concurrently. Keep updates append-only to minimize conflicts.

---

## Subagent Context Injection

Each subagent receives the following context, assembled in this order:

1. **Role prompt** from `.ai-team/prompts/{id}.md`, plus **file attribution rules** (see File Attribution Rules below — inject alongside the prompt at launch time)
2. **Role profile** from `.ai-team/profiles/{id}.md` (read for context; agent will update it after completing work)
3. **Full issue content** (task description + all stage history)
4. **Collaboration guidelines** from `.ai-team/collaboration.md`

When `--issue` is used, the full issue history is injected so the agent has complete context continuity across stages.

---

## Workflow Execution

Execute these steps in order:

1. **Validate environment:**
   - Confirm `.ai-team/` directory exists. If not, tell the user to run `init-team` first.
   - Confirm each specified role ID exists in `.ai-team/team.md`. If a role is not found, show available roles.
2. **Create or read issue:**
   - Without `--issue`: scan `.ai-team/project/issues/` for the highest number, increment, create a new issue file.
   - With `--issue`: read the existing issue file. If not found, list existing issues.
3. **Launch agents:**
   - For each role in the active stage: read its prompt (`.ai-team/prompts/{id}.md`), profile (`.ai-team/profiles/{id}.md`), issue content, and collaboration guidelines (`.ai-team/collaboration.md`).
   - Assemble the full context and launch the agent via the Agent tool.
4. **Agent execution:**
   - Each agent executes the task from its role perspective.
   - Each agent writes a worklog entry to `.ai-team/worklog/{id}/`.
   - Each agent updates the current issue file — appending work summary and listing all files created/modified with paths under the current stage's Progress section.
   - Each agent updates its own profile at `.ai-team/profiles/{id}.md` — adding new skills learned, updating growth areas, and logging what was learned in the Learning Log table.
   - Each agent reports the model it is running on (best-effort).
5. **Collect results:**
   - Gather all agent outputs.
   - Generate a stage summary report.
   - Update the issue file with the stage's results.
6. **Prompt user to review:** After the current stage completes, display a checklist of files to review for each agent in that stage:
   ```
   ✅ {id} 已完成工作，请检查：
   - Issue: `.ai-team/project/issues/{issue-file}`
   - Worklog: `.ai-team/worklog/{id}/{worklog-entry}`
   - Profile: `.ai-team/profiles/{id}.md`
   - 产出文件: {list of files created/modified}
   ```
7. **Enter the stage gate** and wait for explicit user approval before launching the next stage.

---

## Model Reporting (Best-Effort)

Model identification is best-effort and may not always be accurate.

- **Attempt detection** from the runtime environment or API response metadata.
- **Inject into subagent context** if detected, so agents can include it in their worklogs.
- **Fallback:** the agent self-reports its model name. This may be inaccurate — note this caveat in output.
- **Report locations:**
  - Startup announcement (when agents are launched)
  - Worklog entry (per agent)
  - Summary report (per agent row)

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

For files that do not support comments (JSON, binary, images, etc.), record the attribution in the worklog only. Do not modify these files to add attribution.

---

## Summary Report

Generated after the current stage completes (single-session mode only).

### Structure

- **Per agent:** What they did, files produced or modified, model used.
- **Overall:** Unresolved issues, risks, or items needing attention.
- **Next steps:** Suggested actions if applicable.

Present the summary report directly to the user in the conversation, then enter the acceptance flow.

---

## Stage Gate and Acceptance Flow

An iterative loop within the current conversation (single-session mode only):

1. **Present** the current stage summary report to the user.
2. **Interpret** the user's response using LLM judgment:
   - **Approval** for the current stage ("approved", "looks good", "LGTM", "ship it", etc.) — mark the current stage complete and, if another stage remains, launch only the next stage after the user explicitly approves moving on. If the current stage is PM acceptance, mark the issue status as `done` and end the workflow.
   - **Targeted feedback** ("architect revise the assignment", "qa recheck this") — append feedback to the issue file, relaunch only the role(s) in the current stage or the stage the feedback applies to.
   - **Multi-role feedback** ("rd-1 and qa recheck this") — append feedback to the issue, relaunch only the specified role(s) for that stage.
3. **Selective relaunch rules:**
   - Only the specified stage or role(s) are relaunched. Results from earlier stages remain untouched.
   - Relaunched agents receive the full issue history including the new feedback.
   - If one of several assigned developers needs revision, keep the workflow in the implementation stage until the user approves the implementation stage as a whole.
   - Users can add roles for the current stage or the next stage, as long as the role exists in `.ai-team/team.md`. Validate before launching.
4. **No extra commands needed** — the skill maintains the current issue context throughout the loop.
5. **Loop continues** until PM acceptance is explicitly approved after architect final review.

---

## Worklog Entry Template

Each agent writes a worklog entry to `.ai-team/worklog/{id}/{issue-number}-stage-{n}.md`:

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
| `.ai-team/` directory missing | Tell the user: "No `.ai-team/` directory found. Run `init-team` first to set up your AI team." |
| `.ai-team/` exists but key files missing (`team.md`, `prompts/`, `profiles/`) | Report the specific missing files and suggest re-running `init-team`. |
| Role not found in team | Show available roles from `.ai-team/team.md` and ask the user to correct the input. |
| Issue not found with `--issue` | List existing issues in `.ai-team/project/issues/` and ask the user to pick one. |
| All specified roles are invalid (zero valid count) | Display error: "No valid roles specified." and show available roles. |
| Agent tool call fails | Report the failure, record it in the issue file, and ask the user how to proceed. |

---

## Future: Commander Mode

> **Not in v1.** Planned for a future release.

Commander mode introduces an orchestrator agent:

- `/run-team --commander pm` — the PM agent acts as orchestrator.
- The PM reads the task, decomposes it into subtasks, and dynamically dispatches other agents.
- Other agents report back to the PM, who synthesizes results and presents them to the user.

This is noted here for design awareness only. Do not implement commander mode logic.
