---
name: init-team
description: Use when starting a new project that needs multiple AI agents working as a team, setting up AI team roles, or initializing a collaborative agent workspace with role-specific prompts and directory structure
---

# Initialize AI Team

## Overview

Quickly scaffold a complete AI agent team with role-specific prompts, capability profiles, collaboration guidelines, and working directories. Platform-agnostic — works with any AI agent tool.

---

## Trigger and Argument Parsing

### Command Mode

- Preset template: `/init-team web-standard`
- Custom combination: `/init-team 1pm,1architect,2rd,1qa`
- With options: `/init-team 1pm,1architect,2rd,1qa --name my-project --lang en`

### Interactive Mode

When invoked with no arguments (`/init-team`), enter guided setup (see Interactive Mode Flow below).

### Argument Format

Custom combinations use `{count}{abbreviation}` comma-separated. Examples:
- `1pm,1architect,2rd,1qa` — PM, architect, 2 developers, 1 QA
- `1architect,1fe,1be,1qa` — architect, 1 frontend, 1 backend, 1 QA

### Options

- `--name {project-name}`: Set project name for generated documents. Defaults to the current working directory name.
- `--lang {zh|en}`: Generate all document content in Chinese or English. If omitted, ask for the language preference before generation. Directory and file names always remain in English.

### Input Validation Rules

Apply these validation rules before generation:

1. **Invalid template name**: Show the available preset templates table and prompt the user to re-select.
2. **Malformed custom combo** (e.g., `2,rd` or `rd2`): Show the correct format `{count}{abbreviation}` with examples, and request re-input.
3. **`.ai-team/` already exists**: Prompt the user with three options:
   - **Overwrite** — delete the existing `.ai-team/` directory and regenerate everything
   - **Merge** — only generate files that do not already exist, leaving existing files untouched
   - **Cancel** — abort without changes
4. **Role count 0 or negative**: Silently ignore that role entry.
5. **Invalid `--lang` value**: Show the accepted values `zh` and `en`, then request a valid selection.

### Language Preference Resolution

- If `--lang` is provided, use it directly and do not ask again.
- If `--lang` is omitted in command mode, detect a recommended default from the current user input, then explicitly ask the user to choose between English and Chinese before generation.
- In interactive mode, always ask for language preference as part of the guided flow.
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
| `pm` | Project Manager | Requirement intake and clarification, final acceptance |
| `architect` | Architect | Task decomposition, developer assignment, QA assignment, project README/changelog maintenance, dispute review in `project/decisions/`, final technical review |
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
├── project/                    # Project documents
│   ├── README.md               # Project overview (goals, tech stack, architecture)
│   ├── requirements/           # Requirements documents
│   │   └── .gitkeep
│   ├── issues/                 # Issue tracking
│   │   └── .gitkeep
│   ├── decisions/              # Architecture Decision Records (ADR)
│   │   └── .gitkeep
│   └── changelog.md            # Changelog
├── profiles/                   # Role capability & responsibility documents
│   ├── {id}.md                 # One per team member
│   └── ...
├── prompts/                    # System prompts for each role
│   ├── {id}.md                 # One per team member
│   └── ...
└── worklog/                    # Work logs
    ├── {id}/                   # One directory per team member
    │   └── .gitkeep
    └── ...
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
- Team Size: {count}

## Team Members

| ID | Role | Status | Profile | Prompt | Worklog |
|----|------|--------|---------|--------|---------|
| pm | Project Manager | active | [profiles/pm.md](profiles/pm.md) | [prompts/pm.md](prompts/pm.md) | [worklog/pm/](worklog/pm/) |
| architect | Architect | active | [profiles/architect.md](profiles/architect.md) | [prompts/architect.md](prompts/architect.md) | [worklog/architect/](worklog/architect/) |
| rd-1 | Developer | active | [profiles/rd-1.md](profiles/rd-1.md) | [prompts/rd-1.md](prompts/rd-1.md) | [worklog/rd-1/](worklog/rd-1/) |
| rd-2 | Developer | active | [profiles/rd-2.md](profiles/rd-2.md) | [prompts/rd-2.md](prompts/rd-2.md) | [worklog/rd-2/](worklog/rd-2/) |
| qa | QA Engineer | active | [profiles/qa.md](profiles/qa.md) | [prompts/qa.md](prompts/qa.md) | [worklog/qa/](worklog/qa/) |

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

### profiles/{id}.md

Generate one profile per team member at `.ai-team/profiles/{id}.md`:

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

**Note:** The Learning Log table and all profile sections are updated by the agent after each work stage (see Self-Learning in the prompt template).

### prompts/{id}.md

Generate one system prompt per team member at `.ai-team/prompts/{id}.md`:

```markdown
# System Prompt - {Role Name} ({ID})

## Identity
You are the {Role Name} (ID: {ID}) on the {Project Name} project team.

## Responsibilities
{Core responsibilities from profile}

## Behavioral Guidelines
- {Generated based on role type}

## Working Directories
- Your capability profile: `.ai-team/profiles/{ID}.md` — update regularly with your skills and growth
- Your work log: `.ai-team/worklog/{ID}/` — log progress after each work session
- Project docs: `.ai-team/project/` — refer to requirements, issues, and decision records
- Collaboration guidelines: `.ai-team/collaboration.md` — follow team collaboration processes

## Collaboration
- {Interaction patterns with other roles on this specific team}
- Regularly check for `@{your-ID}` mentions in team files

## Self-Learning
After completing each stage of work, you MUST update your capability profile at `.ai-team/profiles/{ID}.md`:
- **Strengths**: Add new skills or tools you used successfully in this stage
- **Areas to Develop**: Update based on challenges you encountered
- **Growth Direction**: Adjust based on what you learned
- **Work Preferences**: Record any effective patterns or approaches you discovered

This is not optional. Self-reflection and profile updates are part of your workflow.

## Issue Update
After completing your work in each stage, you MUST update the current issue file in `.ai-team/project/issues/`:
- Append your work summary under the current stage's Progress section
- List all files you created or modified with their paths
- Example:
  ```
  - **{your-ID}** (model: {model}): {brief work summary}
    - Created: `src/auth/login.js`
    - Modified: `src/routes/index.js`
    - Worklog: `.ai-team/worklog/{your-ID}/{entry}.md`
  ```

## Output Standards
- Work log format: `YYYY-MM-DD-{brief-description}.md`
- Issue format: `{three-digit-number}-{brief-description}.md`
```

**Behavioral Guidelines** examples by role type:
- **PM**: Capture and clarify user requirements, document scope decisions, and provide final acceptance only after architect review
- **Architect**: Break down requirements into implementation-ready tasks, assign developers and QA, maintain `project/README.md` and `project/changelog.md`, review disputes in `project/decisions/`, and perform final technical review
- **RD**: Write clear commit messages, document architectural decisions in `project/decisions/`, follow code review process
- **QA**: Document reproduction steps for every defect, write automated test cases, verify fixes before closing issues
- **Designer**: Provide interaction specs with every mockup, document design rationale, iterate based on feedback
- **DE**: Document data lineage, write data quality checks, maintain pipeline documentation
- **FE**: Optimize for performance and accessibility, document component APIs, coordinate with BE on API contracts
- **BE**: Design RESTful APIs with clear documentation, write database migration scripts, maintain API versioning

**Collaboration** section: Generate interaction patterns based on the actual team composition. Reference other roles by their IDs. For example, if the team has pm, architect, rd-1, rd-2, and qa:
- pm's collaboration: "Capture the user requirement, then hand it to @architect for decomposition. Provide final acceptance only after @architect completes technical review."
- architect's collaboration: "Decompose the work, assign @rd-1 and @rd-2, route testing to @qa, and review disputes in `project/decisions/` before PM acceptance."
- rd-1's collaboration: "Implement assigned tasks, submit work for architect review, and notify @qa when features are ready for testing."
- qa's collaboration: "Report defects to @architect, confirm fixes with the assigned developer, and prepare the final QA summary for architect review."

---

## Dynamic Collaboration Generation Rules

Generate `.ai-team/collaboration.md` dynamically based on the actual team composition.

### Fixed Modules (always included)

Always include these sections in every collaboration.md:

```markdown
# Team Collaboration Guidelines

## Communication Protocol
- All cross-role communication is done through files (issues, worklogs, doc comments)
- Use `@{role-ID}` to flag content for another role's attention
- Each role should regularly check for `@` mentions of their ID

## File Naming Conventions
- Issues: `{three-digit-number}-{brief-description}.md` (e.g., `001-user-login.md`)
- Work logs: `YYYY-MM-DD-{brief-description}.md`
- Decision records: `ADR-{number}-{topic}.md`

## Issue Status Flow
open → in-progress → review → testing → done
(Some statuses may be skipped depending on team composition)
```

### Dynamic Modules (adapt to team composition)

Generate these sections based on the roles actually present:

#### Task Workflow

Adapt the workflow based on which roles are on the team:

- **PM + Architect + RD + QA present**: PM captures and clarifies the requirement → user approves → Architect decomposes work and assigns roles → user approves → RD implements → user approves → QA tests → user approves → Architect performs final technical review and resolves `project/decisions/` entries → user approves → PM gives final acceptance
- **Architect + RD + QA present, no PM**: User submits the requirement → user approves → Architect decomposes work and assigns roles → user approves → RD implements → user approves → QA tests → user approves → Architect performs final technical review and resolves `project/decisions/` entries → user approves → user gives final acceptance
- **No Architect**: This is a custom non-default team. Before implementation begins, require the user to appoint a temporary technical lead from the development roles. That lead owns decomposition, QA routing, dispute review, and final technical review while explicit user approval between stages still applies.
- **FE + BE both present**: Include a frontend-backend API contract workflow — FE and BE agree on API specs in `project/decisions/` before implementation begins
- **Designer present**: Designer provides mockups/specs before FE or RD begins UI work
- **Multiple developers assigned in implementation**: Treat them as one gated implementation stage. They may work in parallel, but QA does not start until all assigned developers finish and the user explicitly approves the handoff.

Write the actual workflow section for the specific team — do not list all possibilities.

#### Code Review Process

- **Multiple development roles** (multiple RD, or FE + BE, etc.): Developers review each other's work. Specify the review pairs based on IDs, then route the final technical review through `@architect` when present.
- **Single development role**: Developer performs self-review using a checklist approach, then `@architect` performs the final technical review when present.

#### Conflict Resolution

- **Architect present**: Technical disagreements are recorded in `project/decisions/` as ADRs. Architect coordinates the discussion, captures the decision, and updates project documentation. PM only handles requirement intake and final acceptance when present.
- **No Architect**: Require the user to appoint a temporary technical lead from the development roles to coordinate technical decisions, record ADRs, route QA, and perform final technical review. PM still only handles requirement intake and final acceptance when present.

#### Role-Specific `@` Mention Instructions

Include in the collaboration doc a section listing each role and what types of mentions they should watch for:
- Example: "@pm — requirement intake, scope clarification, final acceptance"
- Example: "@architect — decomposition requests, ADR reviews, technical decisions, README/changelog updates"
- Example: "@rd-1 — code review requests, bug assignments, technical questions"
- Example: "@qa — test requests, defect confirmations, release sign-off"

---

## Interactive Mode Flow

When invoked with no arguments, follow this guided flow:

1. **Display preset templates**: Show the Preset Team Templates table and ask the user to select one or enter a custom combination.
2. **User selects**: Accept a template name (e.g., `web-standard`) or a custom combo (e.g., `1pm,1architect,2rd,1qa`).
3. **Ask for project name**: Prompt for a project name. Default: current working directory name.
4. **Ask for language preference**: "Generate documents in English or Chinese?" Recommend a default based on the current user input.
5. **Ask about customization**: "Would you like to customize role descriptions?" If yes:
   - Go role by role, showing the default values for each section
   - User can keep defaults, modify parts, or fully customize
   - Overridable sections: core responsibilities, current skills, growth direction, behavioral guidelines
6. **Confirm and generate**: Show a summary of what will be generated and ask for confirmation.
7. **Generate**: Execute the Generation Execution Instructions below (which handles `.gitignore` suggestion).

---

## Generation Execution Instructions

Follow these steps to generate the team:

1. **Parse input**: Determine team composition from template or custom combo. Apply naming rules.
2. **Resolve language preference**: If `--lang` is provided, use it. Otherwise ask for language preference before generation, recommending a default based on the current user input.
3. **Check for existing `.ai-team/`**: Apply the validation rules from Input Validation above (overwrite / merge / cancel).
4. **Create directories** using shell commands with `mkdir -p`:
   ```
   mkdir -p .ai-team/project/requirements .ai-team/project/issues .ai-team/project/decisions .ai-team/profiles .ai-team/prompts
   ```
   Then create worklog directories for each role:
   ```
   mkdir -p .ai-team/worklog/{id}
   ```
5. **Generate each file** listed below. Process roles in order, applying naming rules (single role = no number, multiple = numbered). Generate all files:
   - `.ai-team/team.md`
   - `.ai-team/collaboration.md` (dynamically generated based on team composition)
   - `.ai-team/project/README.md`
   - `.ai-team/project/changelog.md`
   - `.ai-team/project/requirements/.gitkeep` (empty file)
   - `.ai-team/project/issues/.gitkeep` (empty file)
   - `.ai-team/project/decisions/.gitkeep` (empty file)
   - `.ai-team/profiles/{id}.md` for each team member
   - `.ai-team/prompts/{id}.md` for each team member
   - `.ai-team/worklog/{id}/.gitkeep` for each team member (empty file)
6. **Display summary**: After generation, show a summary listing all created files and directories.
7. **Suggest `.gitignore`**: Check if `.gitignore` exists and whether it already contains `.ai-team/`. If not, suggest adding it:
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
