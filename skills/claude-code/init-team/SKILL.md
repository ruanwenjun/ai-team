---
name: init-team
description: Use when starting a new project that needs multiple AI agents working as a team, setting up AI team roles, or initializing a collaborative agent workspace with role-specific prompts and directory structure
---

# Initialize AI Team

## Overview

Quickly scaffold a complete AI agent team with role-specific prompts, capability profiles, collaboration guidelines, and working directories. Platform-agnostic — works with any AI agent tool.

---

## Trigger and Invocation Rules

### Interactive-only Command Model

`init-team` is interactive-only and accepts only bare command invocation:

- `/init-team`

Legacy forms that append templates, custom compositions, or inline flags to the command are deprecated.

If the user provides a legacy form:

1. Stop immediately.
2. Do not treat the extra input as partial state or defaults.
3. Tell the user the old form is deprecated.
4. Instruct them to rerun the bare command.

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
   - **Switch to `update-team`** — stop initialization and direct the user to the dedicated team-maintenance flow
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

## Available Platform Skills
The platform may expose skills (e.g., brainstorming, TDD, systematic-debugging, verification). Use them at your discretion when they help your work. You can record useful skill experiences in your knowledge base for future reference.
- {Generated role-specific platform skill guidance based on role type}

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

**Available Platform Skills** examples by role type:
- **PM**: The platform may provide brainstorming, planning, or research skills. Use them when they help clarify requirements or explore solutions.
- **Architect**: The platform may provide planning, design, or analysis skills. Use them when they help with decomposition or architecture decisions.
- **RD**: The platform may provide TDD, debugging, code review, or other development skills. Use them when they improve implementation quality.
- **QA**: The platform may provide verification, testing, or analysis skills. Use them when they improve test coverage or defect detection.
- **Designer**: The platform may provide design review, prototyping, or user research skills. Use them when they improve design quality or user experience.
- **DE**: The platform may provide data analysis, pipeline testing, or quality validation skills. Use them when they improve data reliability or pipeline robustness.
- **FE**: The platform may provide TDD, debugging, code review, or other development skills. Use them when they improve implementation quality.
- **BE**: The platform may provide TDD, debugging, code review, or other development skills. Use them when they improve implementation quality.

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

## Guided Flow

When invoked as `/init-team`, follow this sequence:

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
   - `switch to update-team`
   - `cancel`
7. **Show a summary preview**: Display the composition, project name, selected language, and customization choice.
8. **Confirm and generate**: Ask for final confirmation before writing files.

---

## Generation Execution Instructions

Follow these steps to generate the team:

1. **Validate invocation**: If the command is not the bare `/init-team`, stop and instruct the user to rerun the bare command.
2. **Resolve language preference**: Ask for language preference first and require an explicit choice.
3. **Parse input**: Determine team composition from the guided preset or custom-combo response. Apply naming rules.
4. **Check for existing `.ai-team/`**: Apply the validation rules from Input Validation above (`reinitialize and overwrite` / `switch to update-team` / `cancel`).
   - If the user chooses `switch to update-team`, stop initialization without writing files.
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
