<p align="center">
  <img src="assets/logo.svg" alt="AI Team Logo" width="200" />
</p>

<h1 align="center">AI Team</h1>

<p align="center">
  <strong>Initialize and run AI agent teams for collaborative vibe coding</strong>
</p>

<p align="center">
  <a href="README_zh.md">中文文档</a>
</p>

---

AI Team provides two skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that help you quickly set up and manage multi-agent teams. Each agent gets its own role, system prompt, capability profile, and working directory — so they can collaborate on your project like a real development team.

**Platform-agnostic** — the generated prompts and docs work with any AI agent tool.

## Features

- **One-command team setup** — preset templates or custom role combinations
- **Role-specific system prompts** — each agent knows its identity, responsibilities, and how to collaborate
- **Self-learning agents** — each agent updates its capability profile after every work stage, tracking new skills and growth
- **Structured collaboration** — file-based communication, issue tracking, decision records
- **Approval-gated task workflow** — plan, implement, review, and accept one stage at a time
- **Multi-session support** — run agents in one session or across multiple terminals

## Installation

### Claude Code

**Option A: Project-level skills (recommended for team sharing)**

Copy the skills into your project's `.claude/skills/` directory:

```bash
# In your project root
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team
mkdir -p .claude/skills
cp -r /tmp/ai-team/skills/claude-code/init-team .claude/skills/init-team
cp -r /tmp/ai-team/skills/claude-code/run-team .claude/skills/run-team
```

Claude Code auto-discovers skills in `.claude/skills/` — no configuration needed.
Project-level skills only apply when Claude Code is started in this project directory (or one of its subdirectories).

**Option B: Personal skills (available across all projects)**

```bash
# Copy to your personal skills directory
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team
mkdir -p ~/.claude/skills
cp -r /tmp/ai-team/skills/claude-code/init-team ~/.claude/skills/init-team
cp -r /tmp/ai-team/skills/claude-code/run-team ~/.claude/skills/run-team
```

**Option C: Symlink (easy updates via git pull)**

```bash
# Clone once
git clone https://github.com/ruanwenjun/ai-team.git ~/ai-team

# Symlink to personal skills
mkdir -p ~/.claude/skills
ln -s ~/ai-team/skills/claude-code/init-team ~/.claude/skills/init-team
ln -s ~/ai-team/skills/claude-code/run-team ~/.claude/skills/run-team
```

After installation, verify skills are available:
1. If you installed the skills while Claude Code was already running, exit and start a new Claude Code session
2. Make sure you are in the project root that contains `.claude/skills/`
3. Then run:

```bash
/init-team web-standard
```

If you still see `Unrecognized command '/init-team'`, it usually means:
- Claude Code was not started from this project root
- The skills were copied in after the current session started and have not been reloaded yet

### OpenAI Codex CLI

Install the skills into Codex's personal skills directory at `~/.codex/skills/`:

```bash
# Clone the repository
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team

# Create the Codex skills directory
mkdir -p ~/.codex/skills

# Install both skills
cp -r /tmp/ai-team/skills/codex/init-team ~/.codex/skills/init-team
cp -r /tmp/ai-team/skills/codex/run-team ~/.codex/skills/run-team
```

**Optional: use symlinks for easier updates via `git pull`**

```bash
# Clone once
git clone https://github.com/ruanwenjun/ai-team.git ~/ai-team

# Symlink into Codex skills
mkdir -p ~/.codex/skills
ln -s ~/ai-team/skills/codex/init-team ~/.codex/skills/init-team
ln -s ~/ai-team/skills/codex/run-team ~/.codex/skills/run-team
```

Restart Codex after installation so it can pick up the new skills.

### Other AI Agent Tools

AI Team generates **platform-agnostic markdown files**. You can use the generated prompts with any AI agent tool:

1. Run `/init-team` in Claude Code (or manually create the `.ai-team/` directory structure)
2. Copy the system prompt from `.ai-team/prompts/{role-id}.md`
3. Paste it into your preferred tool's system prompt field:

| Tool | Where to paste the prompt |
|------|--------------------------|
| **Cursor** | Rules for AI / `.cursorrules` file, or paste into chat |
| **OpenAI Codex CLI** | `instructions.md` in your project root, or `~/.codex/instructions.md` |
| **GitHub Copilot** | `.github/copilot-instructions.md` |
| **Windsurf** | `.windsurfrules` file |
| **Other tools** | System prompt / custom instructions field |

The `run-team` skill requires Claude Code's Agent tool for subagent dispatch. For other platforms, use the generated prompts directly and manage task coordination manually through the `.ai-team/project/issues/` files.

---

## Quick Start

### Initialize a Team

```bash
# Use a preset template
/init-team web-standard

# Custom combination
/init-team 1pm,1architect,2rd,1qa --name my-project

# Interactive mode (guided setup)
/init-team
```

This creates a `.ai-team/` directory in your project:

```
.ai-team/
├── team.md                 # Team overview
├── collaboration.md        # How agents work together
├── project/
│   ├── README.md           # Project goals & tech stack
│   ├── issues/             # Task tracking
│   ├── requirements/       # Requirements docs
│   ├── decisions/          # Architecture Decision Records
│   └── changelog.md
├── profiles/               # Each agent's skills & growth
│   ├── pm.md
│   ├── architect.md
│   ├── rd-1.md
│   └── ...
├── prompts/                # System prompts (copy to any platform)
│   ├── pm.md
│   ├── architect.md
│   ├── rd-1.md
│   └── ...
└── worklog/                # Work logs per agent
    ├── pm/
    ├── architect/
    ├── rd-1/
    └── ...
```

### Run Your Team

```bash
# Assign a task to the architect-led delivery chain
/run-team pm,architect --task "implement user authentication"

# Run all team members
/run-team all --task "set up the project structure"

# Continue an existing issue
/run-team rd-1 --issue 001
```

## Demo

### Example 1: Standard Web Project

```bash
# Step 1: Initialize a web team
/init-team web-standard --name my-web-app
```

This creates a team with 1 PM, 1 Architect, 2 Developers (rd-1, rd-2), and 1 QA Engineer:

```
Generated .ai-team/ with 22 files:
  team.md, collaboration.md
  profiles/pm.md, profiles/architect.md, profiles/rd-1.md, profiles/rd-2.md, profiles/qa.md
  prompts/pm.md, prompts/architect.md, prompts/rd-1.md, prompts/rd-2.md, prompts/qa.md
  project/README.md, project/changelog.md
  worklog/pm/, worklog/architect/, worklog/rd-1/, worklog/rd-2/, worklog/qa/
```

```bash
# Step 2: Assign a task
/run-team all --task "implement user login with JWT authentication"
```

The skill now runs as a gated sequence:
1. Creates issue `001-implement-user-login.md`
2. Launches PM to capture and clarify the requirement
3. Pauses for your confirmation before architect planning starts
4. Launches architect to decompose the work and assign rd-1 plus QA
5. Pauses for your confirmation before implementation starts

```
## Stage 1 Summary - Issue #001
- pm: Clarified the login requirement

Awaiting your confirmation to start architect planning.
```

```bash
# Step 3: Approve architect planning
> proceed
```

After you confirm, the workflow continues with architect planning, implementation, QA review, final technical review from architect, and PM acceptance, pausing between each stage.

```bash
# Step 4: Final acceptance
> looks good
```

Issue marked as done.

### Example 2: Full-Stack with Custom Roles

```bash
/init-team 1pm,1architect,1fe,1be,1qa,1devops --name saas-platform
```

Creates 6 agents. The `devops` role isn't built-in, so AI Team intelligently generates DevOps-appropriate content (CI/CD, infrastructure, monitoring responsibilities).

### Example 3: Multi-Session Mode

In Terminal 1:
```bash
/run-team pm --task "implement payment service"
# Creates issue #002 and records the requirement
```

In Terminal 2:
```bash
/run-team architect --issue 002
# Architect picks up the approved issue, decomposes it, and assigns implementation/QA
```

After user approval, later terminals continue with `/run-team rd-1 --issue 002`, `/run-team qa --issue 002`, and the final architect/PM review stages through the same shared `.ai-team/` directory.

## Preset Templates

| Template | Composition | Use Case |
|----------|------------|----------|
| `web-standard` | 1 PM + 1 Architect + 2 RD + 1 QA | Standard web project |
| `mobile-app` | 1 PM + 1 Architect + 2 RD + 1 QA + 1 Designer | Mobile application |
| `data-pipeline` | 1 PM + 1 Architect + 2 RD + 1 DE | Data engineering |
| `fullstack` | 1 PM + 1 Architect + 1 FE + 1 BE + 1 QA | Frontend/backend split |
| `minimal` | 1 PM + 1 Architect + 1 RD + 1 QA | Quick validation |

## Built-in Roles

| Abbr | Role | Focus |
|------|------|-------|
| `pm` | Project Manager | Requirement intake, business clarification, final acceptance |
| `architect` | Architect | Requirement decomposition, task assignment, QA routing, README/changelog maintenance, dispute review, final technical review |
| `rd` | Developer | Implementation, code review |
| `qa` | QA Engineer | Testing, defect tracking, quality assurance |
| `fe` | Frontend Engineer | UI, performance, accessibility |
| `be` | Backend Engineer | APIs, databases, server-side logic |
| `de` | Data Engineer | Pipelines, ETL, data quality |
| `designer` | Designer | UI/UX, interaction design, visual specs |

Any abbreviation not listed above is treated as a custom role — AI generates appropriate content automatically.

## Options

| Flag | Description | Default |
|------|------------|---------|
| `--name` | Project name in generated docs | Current directory name |
| `--lang {zh|en}` | Language for the current `init-team` or `run-team` invocation; if omitted, the skill asks and recommends a default from the current input language | Ask each run |
| `--task` | Task description for `run-team` | (required) |
| `--issue` | Resume an existing issue number | (creates new) |

## How It Works

**init-team** generates structured markdown files that define your team. Each agent gets:
- A **system prompt** with its identity, responsibilities, behavioral guidelines, and collaboration instructions
- A **capability profile** tracking skills and growth, with a learning log
- A **worklog directory** for recording progress

Before generation starts, `init-team` resolves a language preference. If `--lang` is missing, it asks each time and recommends English or Chinese based on the current input language.

**run-team** reads these files and launches agents:
1. Reads the relevant prompts, profiles, and collaboration guidelines
2. Creates an issue to track the task
3. Dispatches the current stage with full context
4. The active role works, writes its worklog, and **updates its own capability profile** (new skills, growth areas, learning log)
5. Pauses after each stage and waits for your explicit confirmation before moving on
6. Continues through PM requirement intake, architect planning and assignment, implementation, QA review, architect final technical review, and PM final acceptance

Before the first stage starts, `run-team` also resolves a language preference. If `--lang` is missing, it asks each time and recommends English or Chinese based on the current input language. For `--issue`, the selected language affects only newly appended content in that invocation.

Agents communicate through files — issues, worklogs, and `@{role-id}` mentions in documents.

**Self-learning:** After each work stage, every agent reflects on what it learned and updates its `profiles/{id}.md`. Over time, each agent's profile becomes a rich record of accumulated skills and experience — making future task assignments more informed.

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
