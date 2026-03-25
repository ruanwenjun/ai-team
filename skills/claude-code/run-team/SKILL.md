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
9. Display the roles involved in the current stage and let the user choose a run mode:
   - **Single role** — pick one role, run it in the current conversation (no subagent)
   - **All roles — parallel subagents** — dispatch all roles as subagents concurrently (existing behavior)
   - **All roles — sequential in current session** — run each role one by one in the current conversation, pausing for user confirmation between each role
10. Launch according to the chosen run mode.

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

Used when one conversation advances the issue through one or more stages.

1. Launch only the roles for the current stage, not the whole workflow at once.
2. If concurrent Agent calls are unsupported, fall back to sequential execution for that stage.
3. After the stage completes, auto-generate a stage summary report.
4. Stop and wait for explicit user approval before starting the next stage.

### Multi-Session Mode

Used when the user runs `/run-team` in separate terminals or separate conversations while continuing the same team workflow.

1. Each session still starts from the bare `/run-team` command.
2. The user selects the relevant existing issue through the guided flow.
3. Coordination remains file-based through issue files and worklogs.
4. No stage may be skipped; later stages still require explicit user approval.
5. Launch only the roles required by the issue's current valid stage.

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

### Role Selection and Run Modes

Before launching a stage, display the roles involved and let the user choose how to run them.

**Display format example:**

```
当前阶段：Implementation
涉及角色：rd-1 (Developer), rd-2 (Developer)

运行方式：
1. 选择单个角色 — 在当前会话中运行
2. 全部角色 — 并行子 agent
3. 全部角色 — 逐个在当前会话中运行
```

**Run mode details:**

| Mode | How it works |
|------|-------------|
| Single role | User picks one role. Its prompt, profile, issue content, collaboration guidelines, and Superpowers skill are loaded into the current conversation. The orchestrator acts as that role directly — no subagent. After this role completes, enter the stage gate. |
| All roles — parallel subagents | Dispatch all roles via the Agent tool concurrently (existing behavior). |
| All roles — sequential in session | Run each role one by one in the current conversation. After each role completes its work and writes its worklog/issue update, pause and wait for user confirmation before switching to the next role. After the last role completes, enter the stage gate. |

**Current-session execution rules:**

When running a role in the current session (single role or sequential mode):

1. Read the role's prompt, profile, issue content, collaboration guidelines.
2. Load the relevant Superpowers skill via the Skill tool (if available).
3. Announce: "Now acting as {role-id} ({role-name})."
4. Follow the role's prompt and Superpowers skill instructions to execute the work.
5. Write the worklog entry and update the issue file, same as a subagent would.
6. Update the role's profile.
7. Announce completion and present the stage summary.

**Stage gate behavior is the same regardless of run mode** — after all roles in the current stage have completed, wait for explicit user approval before proceeding to the next stage.

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

- Each stage records agent work summaries, model used, stage-specific assignments, and user feedback.
- When continuing an existing issue, read the full issue file and inject its history into agent context.
- Race condition caveat: in multi-session mode, two sessions may update the same issue file concurrently. Keep updates append-only to minimize conflicts.

---

## Subagent Context Injection

Each subagent receives the following context, assembled in this order:

1. **Role prompt** from `.ai-team/prompts/{id}.md`, plus **file attribution rules** (see File Attribution Rules below — inject alongside the prompt at launch time)
2. **Role profile** from `.ai-team/profiles/{id}.md` (read for context; agent will update it after completing work)
3. **Full issue content** (task description + all stage history)
4. **Collaboration guidelines** from `.ai-team/collaboration.md`
5. **Superpowers skill content** (if available — see Superpowers Skill Injection below)

When continuing an existing issue, the full issue history is injected so the agent has complete context continuity across stages.

### Superpowers Skill Injection

Subagents cannot invoke the Skill tool themselves. The orchestrator (main conversation) must load the relevant Superpowers skill content and inject it into each subagent's prompt.

**Before dispatching a subagent**, the orchestrator should:

1. Determine which Superpowers skill maps to the current stage (see Optional External Superpowers Usage above).
2. Use the **Skill tool** to load the skill content in the main conversation (e.g., invoke `superpowers:brainstorming` for PM intake).
3. Include the loaded skill instructions in the subagent's prompt, wrapped in a clear section header:

```
## Superpowers Skill: {skill-name}
{full skill content loaded by the Skill tool}
```

4. Tell the subagent in its prompt: "Follow the Superpowers Skill instructions above as your primary workflow for this stage."

**Stage-to-skill mapping for injection:**

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
   - For an existing issue: list existing issue directories, read the selected `issue.md`, and infer the current stage from its status.
4. **Determine valid next action:**
   - Offer only `continue to the next stage`, `rerun the current stage`, or `apply targeted feedback to the current stage`.
   - Use the issue state to decide which stage is eligible to run next.
5. **Display roles and choose run mode:**
   - Show the roles involved in the current stage.
   - Let the user choose: single role, all roles parallel subagents, or all roles sequential in session.
   - See "Role Selection and Run Modes" above for the full display format and mode details.
6. **Load Superpowers skill for the current stage:**
   - Determine which Superpowers skill maps to the current stage (see the stage-to-skill mapping table in Superpowers Skill Injection).
   - Use the **Skill tool** to load the relevant skill content in the main conversation.
   - If the Skill tool invocation fails or the skill is not installed, proceed without it.
7. **Execute according to run mode:**
   - **Parallel subagents mode:**
     - For each role, read its prompt, profile, issue content, and collaboration guidelines.
     - Include the loaded Superpowers skill content in the subagent's prompt under a `## Superpowers Skill: {name}` header.
     - Tell the subagent to follow the Superpowers Skill instructions as its primary workflow.
     - Assemble the full context and launch the agent via the Agent tool.
   - **Single role or sequential mode (current session):**
     - Read the role's prompt, profile, issue content, and collaboration guidelines.
     - Announce "Now acting as {role-id} ({role-name})."
     - Follow the role's prompt and loaded Superpowers skill to execute the work directly in the current conversation.
     - For sequential mode: after each role completes, pause and wait for user confirmation before starting the next role.
8. **Role execution (all modes):**
   - Each role writes a worklog entry to the current issue directory at `.ai-team/project/issues/{number}-{slug}/{id}-stage-{n}-{stage-name}.md`.
   - Each role updates the issue file (`issue.md` in the same directory) by appending work summary and listing all files created or modified under the current stage's Progress section.
   - Each role updates its own profile at `.ai-team/profiles/{id}.md`.
   - Each role reports the model it is running on (best-effort).
8. **Collect results:**
   - Gather all agent outputs.
   - Generate a stage summary report.
   - Update the issue file with the stage results.
9. **Prompt user to review:**
   - After the current stage completes, display a checklist of all files in the issue directory and updated profiles for each agent in that stage.
10. **Enter the stage gate:**
   - Wait for explicit user approval before launching the next stage.

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

Generated after the current stage completes (single-session mode only).

### Structure

- **Per agent:** What they did, files produced or modified, model used.
- **Overall:** Unresolved issues, risks, or items needing attention.
- **Next steps:** Suggested actions if applicable.

Present the summary report directly to the user in the conversation, then enter the stage gate.

---

## Stage Gate and Acceptance Flow

An iterative loop within the current conversation (single-session mode only):

1. Present the current stage summary report to the user.
2. Interpret the user's response using LLM judgment:
   - **Approval** for the current stage marks that stage complete. If another stage remains, wait for the user to continue to the next stage. If the current stage is PM acceptance, mark the issue status as `done` and end the workflow.
   - **Rerun request** keeps the issue at the same stage and relaunches only the role(s) in that stage.
   - **Targeted feedback** appends feedback to the issue file and relaunches only the role(s) in the current stage or the stage the feedback applies to.
3. Selective relaunch rules:
   - Relaunched agents receive the full issue history including the new feedback.
   - Results from earlier approved stages remain untouched.
   - If one of several assigned developers needs revision, keep the workflow in the implementation stage until the user approves the implementation stage as a whole.
4. No extra commands are needed; the skill keeps the issue context active within the loop.
5. The loop continues until PM acceptance is explicitly approved after architect final review.

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
| Stage cannot advance yet | Explain which earlier approval or unfinished work is still blocking the next stage. |
| Agent call fails | Report the failure, record it in the issue file, and ask the user how to proceed. |
