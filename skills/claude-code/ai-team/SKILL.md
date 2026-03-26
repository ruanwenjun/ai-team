---
name: ai-team
description: Manage AI agent teams — initialize new teams, run roles on issues, or update team composition. Subcommands: init, run, update, status, stop. Use when setting up AI team roles, launching team members on tasks, or modifying team membership.
---

# AI Team Management

## Overview

Unified command for managing AI agent teams with role-specific prompts, capability profiles, collaboration guidelines, and working directories. Platform-agnostic — works with any AI agent tool.

## Subcommand Routing

This skill accepts exactly one subcommand:

| Command | Purpose |
|---------|---------|
| `/ai-team init` | Initialize a new AI team |
| `/ai-team run` | Launch a role on an issue (or re-select to switch) |
| `/ai-team update` | Add/remove team members |
| `/ai-team status` | Show active role and issue in this session |
| `/ai-team stop` | Exit role mode and clear session state |

### Invocation Rules

- Only the forms above are accepted.
- `/ai-team` with no subcommand: show the subcommand table and ask the user to pick one.
- `/ai-team` with unrecognized subcommand: show the table and ask again.
- Legacy forms (`/init-team`, `/run-team`, `/update-team`) are deprecated. If detected, inform the user and show the new form.

Parse the subcommand, then jump to the corresponding section below.

---
---

# Subcommand: init

## Overview

Quickly scaffold a complete AI agent team with role-specific prompts, capability profiles, collaboration guidelines, and working directories.

---

## Trigger and Invocation Rules

### Interactive-only Command Model

`/ai-team init` is interactive-only and accepts only bare invocation.

Legacy forms that append templates, custom compositions, or inline flags are deprecated. If the user provides a legacy form:

1. Stop immediately.
2. Do not treat the extra input as partial state or defaults.
3. Tell the user the old form is deprecated.
4. Instruct them to rerun `/ai-team init`.

### Team Composition Format

During the guided flow, custom combinations still use `{count}{abbreviation}` comma-separated. Examples:
- `1pm,1architect,2rd,1qa` — PM, architect, 2 developers, 1 QA
- `1architect,1fe,1be,1qa` — architect, 1 frontend, 1 backend, 1 QA

### Input Validation Rules

Apply these validation rules before generation:

1. **Invalid template name**: Show the available preset templates table and prompt the user to re-select.
2. **Malformed custom combo** (e.g., `2,rd` or `rd2`): Show the correct format `{count}{abbreviation}` with examples, and request re-input.
3. **`.ai-team/` already exists**: Prompt the user with three options:
   - **Reinitialize and overwrite** — delete the existing `.ai-team/` directory and regenerate everything
   - **Switch to `/ai-team update`** — stop initialization and direct the user to the dedicated team-maintenance flow
   - **Cancel** — abort without changes
4. **Role count 0 or negative**: Silently ignore that role entry.
5. **Invalid language choice**: Show the accepted values `zh` and `en`, then request a valid selection.

### Language Preference Resolution

- Always ask for language preference before any other setup question.
- You may recommend a default based on the current user input, but the user must still explicitly choose.
- Auto-detection only selects the recommended default. It never skips the language question.

---

## Preset Team Templates

| Template | Composition | Use Case |
|----------|------------|----------|
| `web-standard` | 1pm, 1architect, 2rd, 1qa | Standard web project |
| `mobile-app` | 1pm, 1architect, 2rd, 1qa, 1designer | Mobile application |
| `data-pipeline` | 1pm, 1architect, 2rd, 1de | Data engineering project |
| `fullstack` | 1pm, 1architect, 1fe, 1be, 1qa | Full-stack with frontend/backend split |
| `minimal` | 1pm, 1architect, 1rd, 1qa | Small project / quick validation |

---

## Built-in Role Types

| Abbr | Role | Core Responsibilities |
|------|------|----------------------|
| `pm` | Project Manager | Requirement intake and clarification, scope definition, acceptance criteria |
| `architect` | Architect | Task decomposition, architecture design, project README/changelog maintenance, dispute review in `project/decisions/` |
| `rd` | Developer | Code implementation, code review, technical feedback on assigned tasks |
| `qa` | QA Engineer | Test case design, automated testing, defect tracking |
| `designer` | Designer | UI/UX design, interaction specs, visual mockups |
| `de` | Data Engineer | Data pipelines, ETL, data quality |
| `fe` | Frontend Engineer | Frontend architecture, UI implementation, performance optimization |
| `be` | Backend Engineer | Backend architecture, API design, database design |

**Custom roles**: Any abbreviation not in the list above is treated as a custom role. Generate appropriate responsibilities, skills, system prompt, and collaboration relationships based on the role name. For example, `devops` would get DevOps-oriented content.

---

## Directory Structure Generation

Generate the following `.ai-team/` directory structure. Every file listed below MUST be created.

```
.ai-team/
├── team.md                     # Team overview (member list, project info)
├── collaboration.md            # Collaboration guidelines (dynamically generated)
├── roles/                      # Each role owns everything about itself
│   ├── {id}/
│   │   ├── prompt.md           # Behavior definition (identity, responsibilities, work style)
│   │   ├── profile.md          # Self-learning record (abilities, preferences, growth)
│   │   ├── templates/          # Reusable assets (initially empty)
│   │   │   └── .gitkeep
│   │   └── notes/              # Experience notes (initially empty)
│   │       └── .gitkeep
│   └── ...
├── project/                    # Project documents
│   ├── README.md               # Project overview (goals, tech stack, architecture)
│   ├── issues/                 # Issue tracking (each issue is a directory)
│   │   └── .gitkeep
│   ├── decisions/              # Architecture Decision Records (ADR)
│   │   └── .gitkeep
│   └── changelog.md            # Changelog
```

### Naming Rules

- **Single role of a type**: use the abbreviation directly with no number — `architect`, `pm`, `qa`, `designer`
- **Multiple roles of the same type**: append a number — `rd-1`, `rd-2`

---

## File Templates

### team.md

Generate this file at `.ai-team/team.md`:

```markdown
# {Project Name} - Team Overview

## Project Info
- Project Name: {project name}
- Created: {date}
- Last Updated: {date}
- Team Size: {count}

## Team Members

| ID | Role | Status | Prompt | Profile |
|----|------|--------|--------|---------|
| pm | Project Manager | active | [roles/pm/prompt.md](roles/pm/prompt.md) | [roles/pm/profile.md](roles/pm/profile.md) |
| architect | Architect | active | [roles/architect/prompt.md](roles/architect/prompt.md) | [roles/architect/profile.md](roles/architect/profile.md) |
| rd-1 | Developer | active | [roles/rd-1/prompt.md](roles/rd-1/prompt.md) | [roles/rd-1/profile.md](roles/rd-1/profile.md) |
| rd-2 | Developer | active | [roles/rd-2/prompt.md](roles/rd-2/prompt.md) | [roles/rd-2/profile.md](roles/rd-2/profile.md) |
| qa | QA Engineer | active | [roles/qa/prompt.md](roles/qa/prompt.md) | [roles/qa/profile.md](roles/qa/profile.md) |

## Quick Links
- [Collaboration Guidelines](collaboration.md)
- [Project Documentation](project/README.md)
- [Issue Tracking](project/issues/)
- [Changelog](project/changelog.md)
```

Populate the Team Members table with the actual roles for the team being generated. Use the naming rules above for IDs.

### project/README.md

Generate this file at `.ai-team/project/README.md`:

```markdown
# {Project Name}

## Goals
{To be provided by user or filled in later}

## Tech Stack
{To be provided by user or filled in later}

## Architecture Overview
{To be discussed and documented by the team}

## Milestones
{To be planned by architect}
```

### project/changelog.md

Generate this file at `.ai-team/project/changelog.md`:

```markdown
# Changelog

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added
- Team initialized ({date})
```

Replace `{date}` with the current date in `YYYY-MM-DD` format.

### roles/{id}/profile.md

Generate one profile per team member at `.ai-team/roles/{id}/profile.md`:

```markdown
# {Role Name} - {ID}

## Basic Info
- Role: {e.g., Developer}
- ID: {e.g., rd-1}
- Status: active

## Core Responsibilities
- {Generated based on role type — use the Core Responsibilities from the Built-in Role Types table}

## Current Skills
### Strengths
- {Default skills generated based on role type}

### Areas to Develop
- {Empty or generated suggestions based on role type}

## Growth Direction
- {Growth path suggestions based on role type}

## Work Preferences
- {Empty — to be filled by the agent during work}

## Learning Log
| Date | What I Learned | Applied To |
|------|---------------|------------|
| {date} | {skill or insight gained} | {task or context} |
```

Generate intelligent, role-appropriate content for each section. For custom roles, infer appropriate content from the role name.

**Note:** The Learning Log table and all profile sections are updated by the agent after completing work (see Self-Learning in the prompt template).

### roles/{id}/prompt.md

Generate one system prompt per team member at `.ai-team/roles/{id}/prompt.md`:

```markdown
# System Prompt - {Role Name} ({ID})

## Identity
You are the {Role Name} (ID: {ID}) on the {Project Name} project team.

## Responsibilities
{Core responsibilities from profile}

## Behavioral Guidelines
- {Generated based on role type}

## Working Directories
- Your global knowledge base: `.ai-team/roles/{ID}/` — your prompt, profile, templates, and notes
- Your issue workspace: `.ai-team/project/issues/{issue-dir}/{ID}/` — your output for each issue
- Project docs: `.ai-team/project/` — refer to issues and decision records
- Collaboration guidelines: `.ai-team/collaboration.md` — follow team collaboration rules

## Read/Write Rules
- **Write only:** `roles/{ID}/` (your knowledge base) and `issues/{issue-dir}/{ID}/` (your issue output)
- **Read all:** all roles' knowledge bases (`roles/*/`), all issue files (`issues/{issue-dir}/*`), `collaboration.md`, `team.md`, `project/README.md`
- Never modify other roles' files or the issue.md file (maintained by user)

## Collaboration
- {Interaction patterns with other roles on this specific team}
- Regularly check for `@{your-ID}` mentions in team files

## Mandatory Superpowers Skills
You MUST invoke the designated Superpowers skill for your role before producing any work output. This is not optional — it is a core part of your workflow. If the skill invocation fails or is unavailable, report the failure and ask the user how to proceed.

- {Generated based on the Mandatory Superpowers Mapping table below}

Record useful skill experiences in your knowledge base for future reference.

## Self-Learning
After completing work, you MUST update your profile at `.ai-team/roles/{ID}/profile.md`:
- **Strengths**: Add new skills or tools you used successfully
- **Areas to Develop**: Update based on challenges you encountered
- **Growth Direction**: Adjust based on what you learned
- **Work Preferences**: Record any effective patterns or approaches you discovered

You may also update your knowledge base:
- `roles/{ID}/templates/` — save reusable templates, checklists, or patterns
- `roles/{ID}/notes/` — record experience notes, skill preferences, or insights

This is not optional. Self-reflection and knowledge base updates are part of your workflow.

Before finishing, ask yourself:
- Did I create anything reusable? (template, checklist, pattern) → save to `templates/`
- Did I learn something non-obvious? (gotcha, insight, preference) → save to `notes/`

## Prompt Evolution
When you discover a better way to approach your work (new patterns, improved processes, better collaboration strategies), you may update your own `prompt.md` to reflect these improvements. This is how you grow stronger over time. Only modify sections that genuinely benefit from the change — don't change for the sake of changing.

## Default Output
{Role-specific default output guidance}
```

**Behavioral Guidelines** examples by role type:
- **PM**: Capture and clarify user requirements, document scope decisions and acceptance criteria
- **Architect**: Break down requirements into implementation-ready tasks, design architecture, maintain `project/README.md` and `project/changelog.md`, review disputes in `project/decisions/`
- **RD**: Write clear commit messages, document architectural decisions in `project/decisions/`, follow code review process
- **QA**: Document reproduction steps for every defect, write automated test cases, verify fixes before closing issues
- **Designer**: Provide interaction specs with every mockup, document design rationale, iterate based on feedback
- **DE**: Document data lineage, write data quality checks, maintain pipeline documentation
- **FE**: Optimize for performance and accessibility, document component APIs, coordinate with BE on API contracts
- **BE**: Design RESTful APIs with clear documentation, write database migration scripts, maintain API versioning

**Mandatory Superpowers Mapping** by role type:

| Role | Required Skill | When |
|------|---------------|------|
| **PM** | `/brainstorming` | Before producing requirements — explore intent, constraints, and edge cases |
| **Architect** | `/plan` | Before producing design or task breakdown — create structured implementation plan |
| **RD** | `/tdd` | Before writing implementation — write tests first, then implement |
| **RD** (bug/regression) | `/systematic-debugging` then `/tdd` | Debug first, then TDD for the fix |
| **FE** | `/tdd` | Before writing implementation — write tests first, then implement |
| **FE** (bug/regression) | `/systematic-debugging` then `/tdd` | Debug first, then TDD for the fix |
| **BE** | `/tdd` | Before writing implementation — write tests first, then implement |
| **BE** (bug/regression) | `/systematic-debugging` then `/tdd` | Debug first, then TDD for the fix |
| **QA** | `/verify` | Before finalizing test report — run verification loop |
| **Designer** | `/brainstorming` | Before producing design specs — explore design directions and constraints |
| **DE** | `/tdd` | Before writing pipeline code — write tests first, then implement |

Each role MUST invoke its designated skill. This is enforced, not optional. Generate the prompt section content from this table based on the role type.

**Default Output** guidance by role type:
- **PM**: Write requirements documents in `issues/{issue-dir}/pm/`. Default: `requirements.md` with feature descriptions and acceptance criteria.
- **Architect**: Write architecture and task documents in `issues/{issue-dir}/architect/`. Default: `design.md` for architecture, `task-breakdown.md` for task decomposition.
- **RD**: Write implementation records in `issues/{issue-dir}/{id}/`. Default: `worklog.md` documenting what was done and files changed.
- **QA**: Write test reports in `issues/{issue-dir}/qa/`. Default: `test-report.md` with test cases, results, and defects found.
- **Designer**: Write design specs in `issues/{issue-dir}/designer/`. Default: `design-spec.md` with mockups, interaction flows, and design rationale.
- **DE**: Write pipeline documentation in `issues/{issue-dir}/de/`. Default: `pipeline-doc.md` with data flow, quality checks, and lineage.
- **FE**: Write implementation records in `issues/{issue-dir}/fe/`. Default: `worklog.md` documenting what was done and files changed.
- **BE**: Write implementation records in `issues/{issue-dir}/be/`. Default: `worklog.md` documenting what was done and files changed.

**Collaboration** section: Generate interaction patterns based on the actual team composition. Reference other roles by their IDs. For example, if the team has pm, architect, rd-1, rd-2, and qa:
- pm: "Clarify requirements and document them. Reference other roles' output when relevant. Flag scope or priority issues to the team via @ mentions."
- architect: "Read requirements from @pm if available. Decompose work and document architecture decisions. Reference @rd-1/@rd-2 implementation output when reviewing."
- rd-1: "Read requirements and architecture docs from @pm and @architect if available. Implement assigned work. Reference @rd-2's output for coordination. Flag technical issues via @ mentions."
- qa: "Read requirements, architecture, and implementation output from other roles. Design and execute tests. Report defects via @ mentions to relevant roles."

---

## Dynamic Collaboration Generation Rules

Generate `.ai-team/collaboration.md` dynamically based on the actual team composition.

### Fixed Modules (always included)

Always include these sections in every collaboration.md:

```markdown
# Team Collaboration Guidelines

## Communication Protocol
- All cross-role communication is done through files (issues, role output, notes)
- Use `@{role-ID}` to flag content for another role's attention
- Each role should regularly check for `@` mentions of their ID

## File Naming Conventions
- Issues: `{three-digit-number}-{brief-description}/` (directory with `issue.md` inside)
- Role output: stored in `issues/{issue-dir}/{role-id}/` (each role's own subdirectory)
- Decision records: `ADR-{number}-{topic}.md`

## Read/Write Rules
- **Each role writes only to:** `roles/{id}/` (own knowledge base) and `issues/{issue-dir}/{id}/` (own issue output)
- **Each role reads:** all roles' knowledge bases (`roles/*/`), all issue files, `collaboration.md`, `team.md`, `project/README.md`
- **issue.md is user-maintained** — roles must not modify it
```

### Dynamic Modules (adapt to team composition)

Generate these sections based on the roles actually present:

#### Role Descriptions

For each role on the team, include a brief description of their responsibilities and typical output:
- Example: "**@pm** — Requirement intake, scope definition, acceptance criteria. Output: `requirements.md`"
- Example: "**@architect** — Architecture design, task decomposition, technical decisions. Output: `design.md`, `task-breakdown.md`"
- Example: "**@rd-1** — Code implementation, code review, technical feedback. Output: `worklog.md`"
- Example: "**@qa** — Test design, test execution, defect tracking. Output: `test-report.md`"

#### Code Review Process

- **Multiple development roles** (multiple RD, or FE + BE, etc.): Developers review each other's work. Specify the review pairs based on IDs.
- **Single development role**: Developer performs self-review using a checklist approach.

#### Conflict Resolution

- Technical disagreements are recorded in `project/decisions/` as ADRs.
- If an architect role exists, architect coordinates the discussion and captures the decision.
- If no architect role exists, the team resolves disputes through ADR discussion and user arbitration.

#### Role-Specific `@` Mention Instructions

Include a section listing each role and what types of mentions they should watch for:
- Example: "@pm — requirement clarification, scope questions, priority adjustments"
- Example: "@architect — architecture questions, technical decisions, ADR reviews"
- Example: "@rd-1 — code review requests, implementation questions, bug reports"
- Example: "@qa — test requests, defect confirmations, quality questions"

---

## Guided Flow (init)

When invoked as `/ai-team init`, follow this sequence:

1. **Ask for language preference**: "Generate documents in English or Chinese?" Recommend a default when helpful.
2. **Display preset templates**: Show the Preset Team Templates table and ask the user to select one or enter a custom combination.
3. **User selects**: Accept a template name (e.g., `web-standard`) or a custom combo (e.g., `1pm,1architect,2rd,1qa`).
4. **Ask for project name**: Prompt for a project name. Default: current working directory name.
5. **Ask whether to customize role descriptions**: If yes:
   - Go role by role, showing the default values for each section
   - User can keep defaults, modify parts, or fully customize
   - Overridable sections: core responsibilities, current skills, growth direction, behavioral guidelines
6. **Check for existing `.ai-team/`**: Offer only:
   - `reinitialize and overwrite`
   - `switch to /ai-team update`
   - `cancel`
7. **Show a summary preview**: Display the composition, project name, selected language, and customization choice.
8. **Confirm and generate**: Ask for final confirmation before writing files.

---

## Generation Execution Instructions

Follow these steps to generate the team:

1. **Validate invocation**: If the command is not `/ai-team init`, stop and instruct the user to rerun the correct command.
2. **Resolve language preference**: Ask for language preference first and require an explicit choice.
3. **Parse input**: Determine team composition from the guided preset or custom-combo response. Apply naming rules.
4. **Check for existing `.ai-team/`**: Apply the validation rules from Input Validation above (`reinitialize and overwrite` / `switch to /ai-team update` / `cancel`).
   - If the user chooses `switch to /ai-team update`, stop initialization without writing files.
   - If the user chooses `cancel`, abort without changes.
5. **Create directories** using Bash tool with `mkdir -p`:
   ```
   mkdir -p .ai-team/project/issues .ai-team/project/decisions
   ```
   Then for each role:
   ```
   mkdir -p .ai-team/roles/{id}/templates .ai-team/roles/{id}/notes
   ```
6. **Generate files** using the Write tool for each file. Process roles in order, applying naming rules (single role = no number, multiple = numbered). Generate all files:
   - `.ai-team/team.md`
   - `.ai-team/collaboration.md` (dynamically generated based on team composition)
   - `.ai-team/project/README.md`
   - `.ai-team/project/changelog.md`
   - `.ai-team/project/issues/.gitkeep` (empty file)
   - `.ai-team/project/decisions/.gitkeep` (empty file)
   - `.ai-team/roles/{id}/prompt.md` for each team member
   - `.ai-team/roles/{id}/profile.md` for each team member
   - `.ai-team/roles/{id}/templates/.gitkeep` (empty file) for each team member
   - `.ai-team/roles/{id}/notes/.gitkeep` (empty file) for each team member
7. **Display summary**: After generation, show a summary listing all created files and directories.
8. **Suggest `.gitignore`**: Check if `.gitignore` exists and whether it already contains `.ai-team/`. If not, suggest adding it:
   ```
   echo ".ai-team/" >> .gitignore
   ```
   Let the user decide — do not add automatically.

### Language Handling

When language preference is resolved:
- `zh`: All document **content** is written in Chinese
- `en`: All document **content** is written in English
- Directory names and file names remain in English
- Template structure remains the same, only the text content changes

---
---

# Subcommand: run

## Overview

Launch a single team role on a selected issue. The user picks which issue and which role to run — there is no enforced execution order, no stage gates, and no approval flow.

**Prerequisite:** `.ai-team/` directory must exist (created by `/ai-team init`).

**Interactive-only:** This subcommand accepts only the bare form `/ai-team run`.

If the user supplies any legacy role list, issue number, task text, or language flag inline, stop immediately, explain that the old form is deprecated, and tell them to rerun `/ai-team run`.

---

## Session State

The `run` subcommand maintains **conversation-level session state** so the agent remembers which role and issue are active throughout the conversation.

### State Variables

- `activeRole` — the role ID currently being played (e.g., `rd-1`). Initially unset.
- `activeIssue` — the issue directory currently being worked on (e.g., `001-user-login`). Initially unset.

### Lifecycle

- **Set**: When the Guided Flow (run) completes successfully and Role Execution begins.
- **Replaced**: When the user runs `/ai-team run` again and completes a new selection.
- **Cleared**: When the user runs `/ai-team stop`, `/ai-team init`, or `/ai-team update`.

### In-Role Mode

When both `activeRole` and `activeIssue` are set, the agent is in **role mode**:

- All subsequent user messages (that are NOT `/ai-team` commands) are interpreted as instructions to `activeRole` working on `activeIssue`.
- The agent does NOT re-run the guided flow or re-ask for issue/role selection.
- The agent continues writing output to `issues/{activeIssue}/{activeRole}/`.
- The agent does NOT re-invoke the mandatory Superpowers skill for follow-up messages. Only re-invoke if the user explicitly starts a fundamentally new piece of work within the same issue.

### Status Banner

Whenever session state changes or is queried, display this banner:

```
───────────────────────────────────
Active: {role-id} ({role-name}) on #{issue-number} - {issue-slug}
───────────────────────────────────
```

Display the banner:
- After guided flow completes and role execution begins
- After a role or issue switch
- When the user runs `/ai-team status`

### `/ai-team status` Behavior

- If session state is set: display the Status Banner.
- If no state is set: "No active role or issue. Run `/ai-team run` to start."

### `/ai-team stop` Behavior

- Clear `activeRole` and `activeIssue`.
- Announce: "Exited role mode. No active role or issue."
- The agent returns to normal assistant mode.

---

## Guided Flow (run)

Follow this sequence in order:

1. Validate that `.ai-team/` exists and that `roles/` contains at least one role directory with a `prompt.md`.
2. List existing issues from `.ai-team/project/issues/`, or offer to create a new one.
   - New issue: collect task description, scan for the highest issue number (by directory name), increment, create `issues/{num}-{slug}/issue.md` with only the task description.
   - Existing issue: let the user choose one.
3. List all roles from `.ai-team/roles/`, showing which roles already have output in this issue.
   - If a role's subdirectory exists under the issue directory, mark it as "has output".
4. User picks one role.
5. Load role context and execute in the current conversation.
6. **Set session state**: `activeRole = {selected role ID}`, `activeIssue = {selected issue directory}`. Display the Status Banner. The agent is now in role mode — subsequent messages are handled per the "Subsequent Messages" section below.

---

## Role Selection Display

```
Issue: #001 - user-login
Team roles:
  1. pm (Project Manager)
  2. architect (Architect) — has output
  3. rd-1 (Developer)
  4. rd-2 (Developer) — has output
  5. qa (QA Engineer)

Select a role to run:
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

## Knowledge Base Loading Strategy

When loading `roles/{id}/templates/` and `roles/{id}/notes/`:
- Read the file listing first (not all file contents)
- Select files relevant to the current issue based on file names and the issue description
- If the knowledge base is small (< 10 files), load all
- If large, load only the most relevant files and note which ones were skipped

---

## Role Execution (Initial)

This is the initial execution when a role is first activated via the Guided Flow.

1. Announce: "Now acting as {role-id} ({role-name})." and display the Status Banner.
2. Read issue content and other roles' existing output as input.
3. **Invoke mandatory Superpowers skill** — call the designated skill for this role (see Mandatory Superpowers Mapping). This step is NOT optional. The skill must be invoked before producing any work output.
4. Execute work per own prompt definition, incorporating the skill's output.
5. Write all output to `issues/{num}-{slug}/{id}/`.
6. Update `roles/{id}/profile.md` (learning record).
7. Optionally update `roles/{id}/templates/` or `roles/{id}/notes/` (accumulate reusable assets).
8. Display list of files produced.
9. Session state persists — the agent remains in role mode for subsequent messages.

No stage gate, no approval flow. User decides next step.

---

## Subsequent Messages (In-Role Mode)

When session state is set (`activeRole` and `activeIssue` are both defined) and the user sends a message that is NOT a `/ai-team` command:

1. **Do NOT re-run the guided flow.** Skip issue selection and role selection entirely.
2. **Do NOT re-invoke the mandatory Superpowers skill.** The skill was already invoked during the initial role execution. Only re-invoke if the user explicitly starts a fundamentally new piece of work within the same issue.
3. **Prefix the first subsequent response** with a brief reminder: `Continuing as {role-id} ({role-name}) on #{issue-number}.`  After the first reminder, omit it unless context makes it helpful.
4. **Interpret the message as an instruction to the active role** on the active issue. Execute accordingly.
5. **Continue writing output** to `issues/{activeIssue}/{activeRole}/`.
6. **Re-read other roles' output** if needed for the latest context (e.g., another role may have produced output in parallel).

### When to Re-invoke Superpowers

Do NOT re-invoke for:
- Follow-up questions ("what about edge cases?")
- Refinements ("update the design to use Redis instead")
- Continuation work ("now implement the second endpoint")

DO re-invoke if the user explicitly starts a new, independent task within the same issue that warrants a fresh skill pass (e.g., "now write a completely new module for notifications" — this is new work, not a continuation).

---

## Switching Roles or Issues

Three ways to switch the active context:

### 1. `/ai-team run` (full re-select)

Running `/ai-team run` always triggers the full Guided Flow, regardless of current state. Upon completion, the session state is replaced with the newly selected role and issue. The old state is discarded.

### 2. Natural language switch

If the user says something like "switch to architect" or "switch to issue 002" while in role mode:

1. **Validate** that the requested role or issue exists in `.ai-team/`.
2. **Confirm** the switch: "Switch from {current-role} to {new-role} on #{issue}? (yes/no)" — only ask for confirmation if the intent is ambiguous. If the request is clearly a switch command (e.g., "switch to qa"), proceed directly.
3. **Re-load context** for the new role/issue (Role Context Loading).
4. **Update session state** with new `activeRole` and/or `activeIssue`.
5. **Display the Status Banner** with the new state.
6. **Invoke the mandatory Superpowers skill** for the new role (this is a fresh role activation, not a continuation).

If the request is ambiguous (e.g., "I think the architect should review this"), ask: "Did you want to switch to the architect role, or are you asking me ({activeRole}) to flag this for the architect?"

### 3. `/ai-team stop` (exit role mode)

Clears session state entirely. See `/ai-team stop` Behavior in the Session State section.

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

## Multi-Session Collaboration

Multiple CLI sessions can run different roles on the same issue simultaneously.

### Coordination Model
- Each role writes only to its own directory — no file conflicts between roles
- Roles should read other roles' output at the start of execution for the latest context
- If another role's output directory is empty, it means that role hasn't started or finished yet

### Signaling Completion
After a role finishes, it creates a status marker in its issue output directory:
- File: `issues/{num}-{slug}/{id}/.done`
- Content: completion timestamp and brief summary
Other roles can check for `.done` files to know which roles have finished.

### Safety
- Never run the same role in two sessions simultaneously (profile.md write conflict)
- Different roles can safely run in parallel on the same issue

---

## Error Handling (run)

| Condition | Action |
|---|---|
| Legacy argument-based invocation | Tell the user the form is deprecated and they must rerun `/ai-team run`. |
| `.ai-team/` directory missing | Tell the user: "No `.ai-team/` directory found. Run `/ai-team init` first to set up your AI team." |
| `.ai-team/` exists but `roles/` missing or empty | Report the issue and suggest re-running `/ai-team init`. |
| No issues exist and user wants to continue | Tell the user no issues found and offer to create a new one. |
| Selected role has no `prompt.md` | Report the missing file and suggest checking `roles/{id}/`. |
| Role execution fails | Report the failure and ask the user how to proceed. |
| Session state lost (context truncated) | Re-derive state from conversation history. If unrecoverable, inform the user and offer to re-run `/ai-team run`. |
| Switch to non-existent role | Show available roles from `.ai-team/roles/` and ask the user to pick again. |
| Switch to non-existent issue | Show available issues from `.ai-team/project/issues/` and ask the user to pick again. |
| Ambiguous switch request | Ask the user to clarify: "Did you want to switch roles, or is this an instruction for the current role?" |

---
---

# Subcommand: update

## Overview

Safely update an existing `.ai-team/` without rewriting historical project artifacts. This subcommand is for iterative team maintenance after `/ai-team init` has already created the workspace.

**Prerequisite:** `.ai-team/` directory must exist.

**Interactive-only:** This subcommand accepts only the bare form `/ai-team update`.

If the user supplies any legacy arguments, flags, or inline role changes, stop immediately and tell them the old form is deprecated and they must rerun `/ai-team update`.

---

## Trigger and Invocation Rules (update)

### Deprecated Legacy Forms

Treat any non-bare form that includes inline actions, role edits, or language flags as deprecated.

Do not parse these as partial input or prefilled defaults. Show migration guidance and wait for the user to rerun `/ai-team update`.

### Language Preference

1. Ask for language preference at the start of every invocation.
2. If the current user input suggests a default language, you may recommend it.
3. Recommendation never replaces the explicit question. The user must still choose the language for this run.

---

## Guided Flow (update)

Execute the following sequence in order:

1. Ask for language preference.
2. Confirm `.ai-team/` exists. If it does not, tell the user to run `/ai-team init` first.
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
4. Generate the member's profile and prompt files.

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
4. Show the changelog summary that will be appended on apply.

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

If blocked, instruct the user to reassign or finish that work before retrying `/ai-team update`.

### Historical Preservation

Do not rewrite existing issue history, worklog entries, or archived references. Team updates affect future work only.

### Full Refresh for Active Roles

After any confirmed update, regenerate `profiles/` and `prompts/` for every active role, not just added or directly modified ones.

---

## Apply Phase

After final confirmation, apply the team update in this order:

1. Rewrite `.ai-team/team.md`
2. Rewrite `.ai-team/collaboration.md`
3. Regenerate `.ai-team/roles/{id}/profile.md` for every active role
4. Regenerate `.ai-team/roles/{id}/prompt.md` for every active role
5. Archive removed roles under `.ai-team/archive/roles/{id}/`
6. Append a structured update entry to `.ai-team/project/changelog.md`

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

## Validation Checklist (update)

Before applying changes, confirm all of the following:

- `.ai-team/` exists
- language preference is explicitly chosen for this invocation
- every removal passed the in-progress issues safety check
- every added member received a non-reused stable ID
- the archive preview matches the pending removals
- the user explicitly approved the final preview

If any item fails, stop and resolve it before writing files.
