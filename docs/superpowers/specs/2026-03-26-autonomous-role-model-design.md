# Autonomous Role Model Design

## Date: 2026-03-26

## Summary

Redesign the ai-team workflow from a stage-gated state machine to a user-driven autonomous role model. Each role is a self-contained unit that owns its prompt, profile, and knowledge base. There is no enforced execution order — the user decides which role to run on which issue at any time.

---

## Core Principles

1. **No state machine** — Fully user-driven, no ordering constraints, no stage gates, no approval flow.
2. **Two-layer knowledge base** — Global knowledge base (`roles/{id}/`) for cross-issue experience + issue-level workspace (`issues/{num}-{slug}/{id}/`) for per-issue output.
3. **Read all, write own** — Directory convention + prompt constraint. All files globally readable, each role writes only to its own directories.
4. **Fully autonomous roles** — Prompt, profile, and knowledge base all owned by the role. Roles can evolve their own behavior definitions over time.
5. **Minimal interaction** — Select issue → select role → execute. No language preference prompt, no team composition validation, no stage gates.

---

## Directory Structure

### Global Role Directory

```
.ai-team/
├── team.md                          # Team overview (links to roles/{id}/)
├── collaboration.md                 # Collaboration rules (read/write rules, naming conventions)
├── roles/                           # Each role owns everything about itself
│   ├── pm/
│   │   ├── prompt.md                # Behavior definition (identity, responsibilities, work style)
│   │   ├── profile.md              # Self-learning record (abilities, preferences, growth)
│   │   ├── templates/              # Reusable assets (requirement templates, etc.)
│   │   └── notes/                  # Experience notes (free-form, self-organized)
│   ├── architect/
│   │   ├── prompt.md
│   │   ├── profile.md
│   │   ├── templates/
│   │   └── notes/
│   ├── rd-1/
│   │   ├── prompt.md
│   │   ├── profile.md
│   │   ├── templates/
│   │   └── notes/
│   ├── rd-2/
│   │   └── ...
│   └── qa/
│       └── ...
├── project/
│   ├── README.md                    # Project overview
│   ├── changelog.md                 # Version history
│   └── issues/
│       └── {num}-{slug}/
│           ├── issue.md             # User-maintained task description
│           ├── pm/                  # PM output
│           ├── architect/           # Architect output
│           ├── rd-1/                # Developer output
│           ├── rd-2/                # Developer output
│           └── qa/                  # QA output
```

### Removed Directories

- ~~`prompts/`~~ → merged into `roles/{id}/prompt.md`
- ~~`profiles/`~~ → merged into `roles/{id}/profile.md`
- ~~`worklog/`~~ → merged into issue-level `{id}/` subdirectories

### Unchanged

- `team.md` — Team overview (update role links to point to `roles/{id}/`)
- `collaboration.md` — Rewrite content to match new model
- `project/README.md`, `project/changelog.md`

---

## Read/Write Rules

### Write Rules

Each role can **only write** to:

| Location | Description |
|----------|-------------|
| `roles/{id}/prompt.md` | Own behavior definition |
| `roles/{id}/profile.md` | Own learning record |
| `roles/{id}/templates/*` | Own reusable assets |
| `roles/{id}/notes/*` | Own experience notes |
| `project/issues/{num}-{slug}/{id}/*` | Own output in current issue |

### Read Rules

Each role can **read**:

| Location | Description |
|----------|-------------|
| `roles/*/` | All roles' global knowledge bases (prompt, profile, templates, notes) |
| `project/issues/{num}-{slug}/*` | All files in current issue (issue.md + all roles' output) |
| `collaboration.md` | Collaboration rules |
| `team.md` | Team overview |
| `project/README.md` | Project overview |

**One-line summary: globally readable, write only to own directories.**

### Enforcement

Enforced via directory convention + prompt constraint. Each role's `prompt.md` baseline includes:

> You can only create and modify files in `roles/{id}/` and `issues/{current-issue}/{id}/`. You can read all files in the issue directory and other roles' knowledge bases for reference, but must not modify others' files.

---

## Role Default Output

Defined as suggestions in each role's `prompt.md` baseline. Not enforced — roles can create additional files as needed.

### PM

```
issues/{num}-{slug}/pm/
├── requirements.md          # Requirements (features, acceptance criteria)
└── ...
```

### Architect

```
issues/{num}-{slug}/architect/
├── design.md                # Architecture design (components, data flow, tech choices)
├── task-breakdown.md        # Task decomposition (assignments)
└── ...
```

### Developer

```
issues/{num}-{slug}/{rd-id}/
├── worklog.md               # Implementation record (what was done, files changed)
└── ...
```

### QA

```
issues/{num}-{slug}/qa/
├── test-report.md           # Test report (test cases, results, defects)
└── ...
```

---

## `/run-team` Flow

### Interaction

```
User runs /run-team
    ↓
1. Validate .ai-team/ environment exists
    ↓
2. List issues (or create new)
   - New: user provides description → create issues/{num}-{slug}/issue.md
   - Existing: user picks one
    ↓
3. List team roles, user picks one
   - Mark roles with existing output in this issue
    ↓
4. Load role context, execute in current session
    ↓
5. Role completes, updates files
```

### Environment Validation

- Confirm `.ai-team/` exists
- Confirm `roles/` has at least one role directory with `prompt.md`
- No team composition requirements (no mandatory PM + architect + dev + QA)

### Issue Selection Display

```
现有 Issue：
  1. #001 - user-login
  2. #003 - optimize-readme
  3. 新建 Issue

请选择：
```

### Role Selection Display

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

### Context Loading (in order)

1. `roles/{id}/prompt.md` — behavior definition
2. `roles/{id}/profile.md` — capabilities and experience
3. `roles/{id}/templates/` and `roles/{id}/notes/` — knowledge base
4. `issues/{num}-{slug}/issue.md` — task description
5. `issues/{num}-{slug}/*/` — other roles' existing output (read-only reference)
6. `collaboration.md` — collaboration rules

### Execution

1. Announce "Now acting as {role-id} ({role-name})"
2. Read issue and other roles' output as input
3. Execute work per own prompt definition
4. Write output to `issues/{num}-{slug}/{id}/`
5. Update `roles/{id}/profile.md` (learning record)
6. Optionally update `roles/{id}/templates/` or `roles/{id}/notes/` (accumulate reusable assets)
7. Display list of files produced

No stage gate, no approval flow. User decides next step.

### New Issue Creation

1. User provides task description
2. System creates `issues/{num}-{slug}/issue.md` with only the task description
3. No role assignment, no status, no stages

---

## Superpowers Skill Usage

- **No mandatory mapping** between roles and skills
- Role's `prompt.md` baseline includes a note about available platform skills (e.g., TDD, brainstorming, debugging)
- Agent autonomously decides whether to use any skill during execution
- Roles can learn and record skill preferences in their knowledge base over time

---

## Impact on `init-team`

### Directory Generation

Changes from generating `prompts/`, `profiles/`, `worklog/` to generating `roles/{id}/` with `prompt.md`, `profile.md`, `templates/`, `notes/`.

### `collaboration.md` Template

Remove:
- 8-stage workflow
- Stage gate and approval flow
- Issue status flow (open → in-progress → review → testing → done)

Add/Keep:
- Read/write rules (globally readable, write only to own directories)
- File naming conventions
- @ mention protocol
- Conflict resolution

### `prompt.md` Baseline

Each role's prompt includes:
1. Identity and responsibilities
2. Working directories (`roles/{id}/` and `issues/{num}-{slug}/{id}/`)
3. Read/write rules
4. Default output types (suggestions, not enforced)
5. Self-learning requirement (update profile.md after each work session)
6. Available platform skills (note, not mandate)

Removed:
- Stage-related behavior rules
- Issue status update requirements (issue.md is user-maintained)

### Team Validation

No longer requires "at least 1 PM + 1 architect + 1 dev + 1 QA". Only requires at least one role in `roles/`.

---

## Impact on `run-team`

### Massive Simplification

Current: ~470 lines. New: ~150-200 lines.

**Removed:**
- 6-stage workflow, stage gates, approval flow
- Role-level stage state (pending/in-progress/done)
- Multi-session coordination (file locks, race conditions)
- Subagent context injection
- Superpowers skill mapping table
- Stage summary / role-progress report
- Worklog entry template
- File attribution rules
- Model reporting
- Language preference prompt
- Team composition validation

**Retained:**
- Environment validation
- Issue selection/creation
- Role selection
- Context loading
- Role execution in current session
- Post-execution updates (profile, knowledge base)
- Legacy command handling
- Basic error handling

**Added:**
- "Has output" annotation on role selection
- Knowledge base loading logic (`templates/` and `notes/`)
