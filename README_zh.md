<p align="center">
  <img src="assets/logo.svg" alt="AI Team Logo" width="200" />
</p>

<h1 align="center">AI Team</h1>

<p align="center">
  <strong>初始化并运行 AI 智能体团队，协同完成 Vibe Coding</strong>
</p>

<p align="center">
  <a href="README.md">English</a>
</p>

---

AI Team 为 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 和 OpenAI Codex CLI 提供三个 skill，帮助你初始化、运行和更新多智能体团队。每个智能体拥有独立的角色、系统提示词和能力档案，并通过共享的项目文档与 Issue 目录协作，像真实开发团队一样完成项目。

生成的提示词和文档是 Claude Code 与 OpenAI Codex CLI 两个受支持工作流共享的资产。这两个平台都支持交互式 `run-team`，可以直接分派当前阶段所需的智能体。

## 可选的外部 Superpowers 支持

AI Team 可以在 Codex 和 Claude Code 两端使用外部 Superpowers skill，但前提是当前平台真的暴露了这些 skill。

- 如果相关外部 skill 可用，对应角色就使用它
- 如果外部 skill 不可用，AI Team 继续按原本流程运行
- AI Team 不会生成或模拟一套本地 Superpowers 替代品

当外部 skill 可用时，默认角色映射如下：

- PM -> `brainstorming`
- Architect -> `writing-plans`
- 开发角色（`rd`、`fe`、`be`）-> `test-driven-development`
- QA -> 测试执行 + `verification-before-completion`
- bug、回归和线上问题任务 -> 开发角色先 `systematic-debugging`，再进入 TDD

## 特性

- **引导式生命周期命令** — 交互式 `init-team`、`run-team`、`update-team`，按阶段逐步推进
- **可复用的提示词与文档** — 生成的 Markdown 资产可在受支持的 Claude Code 与 OpenAI Codex CLI 工作流中共享
- **可选的外部 Superpowers 支持** — 只有在外部 skill 可用时，才使用 brainstorming、planning、TDD、debugging 和 verification
- **角色专属提示词** — 每个智能体清楚自己的身份、职责和协作方式
- **自我学习** — 每个智能体在每轮工作后自动更新能力档案，记录新技能和成长
- **结构化协作** — 基于文件的沟通、Issue 跟踪、决策记录
- **审批门控式任务流** — 按阶段规划、实现、审查与验收
- **安全团队维护** — 归档被移除角色、刷新 active prompts/profiles，并保留历史 Issue
- **多会话支持** — 在一个会话中运行多个智能体，或在多个终端中分别运行

## 安装

### Claude Code

**方式 A：项目级 skills（推荐，便于团队共享）**

将 skills 复制到项目的 `.claude/skills/` 目录：

```bash
# 在你的项目根目录下
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team
mkdir -p .claude/skills
cp -r /tmp/ai-team/skills/claude-code/init-team .claude/skills/init-team
cp -r /tmp/ai-team/skills/claude-code/run-team .claude/skills/run-team
cp -r /tmp/ai-team/skills/claude-code/update-team .claude/skills/update-team
```

Claude Code 会自动发现 `.claude/skills/` 中的 skills，无需额外配置。
项目级 skills 只在**当前项目目录**（或其子目录）中生效。

**方式 B：个人级 skills（所有项目通用）**

```bash
# 复制到个人 skills 目录
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team
mkdir -p ~/.claude/skills
cp -r /tmp/ai-team/skills/claude-code/init-team ~/.claude/skills/init-team
cp -r /tmp/ai-team/skills/claude-code/run-team ~/.claude/skills/run-team
cp -r /tmp/ai-team/skills/claude-code/update-team ~/.claude/skills/update-team
```

**方式 C：软链接（通过 git pull 轻松更新）**

```bash
# 克隆一次
git clone https://github.com/ruanwenjun/ai-team.git ~/ai-team

# 软链接到个人 skills
mkdir -p ~/.claude/skills
ln -s ~/ai-team/skills/claude-code/init-team ~/.claude/skills/init-team
ln -s ~/ai-team/skills/claude-code/run-team ~/.claude/skills/run-team
ln -s ~/ai-team/skills/claude-code/update-team ~/.claude/skills/update-team
```

安装后验证：
1. 如果你是在 Claude Code 会话进行中才安装的，先退出并重新进入 Claude Code
2. 确保你当前就在包含 `.claude/skills/` 的项目根目录中
3. 再执行：

```bash
/init-team
```

如果仍然提示 `Unrecognized command '/init-team'`，通常是因为：
- Claude Code 不是从当前项目根目录启动的
- skills 是在当前会话启动后才复制进去的，尚未重新加载

### OpenAI Codex CLI

将当前仓库中的 skill 安装到 Codex 的个人 skills 目录 `~/.codex/skills/`：

```bash
# 克隆仓库
git clone https://github.com/ruanwenjun/ai-team.git /tmp/ai-team

# 创建 Codex skills 目录
mkdir -p ~/.codex/skills

# 安装这三个 skill
cp -r /tmp/ai-team/skills/codex/init-team ~/.codex/skills/init-team
cp -r /tmp/ai-team/skills/codex/run-team ~/.codex/skills/run-team
cp -r /tmp/ai-team/skills/codex/update-team ~/.codex/skills/update-team
```

**可选：使用软链接，便于后续通过 `git pull` 更新**

```bash
# 克隆一次
git clone https://github.com/ruanwenjun/ai-team.git ~/ai-team

# 软链接到 Codex skills
mkdir -p ~/.codex/skills
ln -s ~/ai-team/skills/codex/init-team ~/.codex/skills/init-team
ln -s ~/ai-team/skills/codex/run-team ~/.codex/skills/run-team
ln -s ~/ai-team/skills/codex/update-team ~/.codex/skills/update-team
```

安装后请重启 Codex，使新 skill 生效。

## 快速开始

所有生命周期命令现在都是交互式的，并按阶段门控推进：

```bash
/init-team
/run-team
/update-team
```

- `/init-team` 会询问语言、模板或自定义组合、项目名，以及是否自定义角色描述。
- `/run-team` 会询问是新任务还是继续已有 Issue，然后只提供当前合法的下一步阶段动作。
- `/update-team` 会通过 add/remove/review 循环维护团队，并在最终确认后才真正落盘。

旧的参数式写法已废弃。如果你之前使用过旧命令形式，请重新运行裸命令，并在引导流程里提供同样的信息。

`/init-team` 会在你的项目中创建 `.ai-team/` 目录：

```
.ai-team/
├── team.md                 # 团队总览
├── collaboration.md        # 协作规范
├── project/
│   ├── README.md           # 项目目标和技术栈
│   ├── issues/             # 任务跟踪
│   ├── requirements/       # 需求文档
│   ├── decisions/          # 架构决策记录 (ADR)
│   └── changelog.md
├── profiles/               # 每个智能体的能力和成长
│   ├── pm.md
│   ├── architect.md
│   ├── rd-1.md
│   └── ...
└── prompts/                # 供受支持平台使用的系统提示词
    ├── pm.md
    ├── architect.md
    ├── rd-1.md
    └── ...
```

`.ai-team/archive/roles/{id}/` 会在后续运行 `/update-team` 且有角色被移除时才创建，不属于 `init-team` 的初始脚手架。

每个 Issue 是 `project/issues/` 下的一个目录，包含 Issue 文件和所有角色的工作日志：

```
project/issues/001-user-login/
├── issue.md
├── pm-stage-1-requirement-intake.md
├── architect-stage-2-planning.md
├── rd-1-stage-3-implementation.md
├── qa-stage-4-review.md
└── architect-stage-5-final-review.md
```

## 演示

### 示例 1：标准 Web 项目

```bash
# 第一步：初始化 Web 团队
/init-team
```

选择 `web-standard`，把项目名设为 `my-web-app`，选择语言并确认预览。这样会创建一个包含 1 个 PM、1 个架构师、2 个开发（rd-1, rd-2）和 1 个 QA 的团队：

```
已生成 .ai-team/，关键文件包括：
  team.md, collaboration.md
  profiles/pm.md, profiles/architect.md, profiles/rd-1.md, profiles/rd-2.md, profiles/qa.md
  prompts/pm.md, prompts/architect.md, prompts/rd-1.md, prompts/rd-2.md, prompts/qa.md
  project/README.md, project/changelog.md
  project/issues/
```

```bash
# 第二步：启动新任务
/run-team
```

在引导流程里选择语言，选择 `start a new task`，并输入 `实现 JWT 用户登录功能`。

Skill 会按门控阶段自动执行：
1. 创建 Issue 目录 `001-implement-user-login/` 及 `issue.md`
2. 启动 PM 接收并澄清需求；如果外部 `brainstorming` 可用，则在这一阶段使用它
3. 在进入架构规划前暂停，等待你的确认
4. 启动架构师拆分工作并分配给 rd-1 和 QA；如果外部 `writing-plans` 可用，则在这一阶段使用它
5. 在进入实现阶段前暂停，等待你的确认

```
## 阶段 1 汇总 - Issue #001
- pm：已澄清登录需求

等待你确认后再进入架构规划。
```

```bash
# 第三步：确认进入架构规划阶段
> proceed
```

你确认后，流程会继续进入架构规划、实现、QA 审查、架构师最终技术审查，以及 PM 最终验收。如果当前阶段对应的外部 Superpowers skill 可用，就在该阶段使用它。

```bash
# 第四步：最终验收
> 可以了
```

Issue 标记为完成。

### 示例 2：更新已有团队

```bash
/update-team
```

通过 action loop 添加成员、移除成员，并在真正应用前查看预览：

- 被移除角色会归档到 `.ai-team/archive/roles/{id}/`
- 确认后会重新生成所有 active 角色的 prompts 和 profiles
- 角色 ID 不复用，例如删掉 `rd-2` 后再次新增开发会得到 `rd-3`
- 如果未完成的 Issue 仍依赖某个角色，移除会被阻止

### 示例 3：多会话模式

终端 1：
```bash
/run-team
```

在这个终端里选择 `continue an existing issue` 时，可以继续该会话里已批准的工作。

终端 2：
```bash
/run-team
```

选择同一个 Issue，并继续当前合法阶段。角色分派来自 Issue 状态，而不是命令行参数。

## 预设模板

| 模板 | 组成 | 适用场景 |
|------|------|----------|
| `web-standard` | 1 PM + 1 架构师 + 2 RD + 1 QA | 标准 Web 项目 |
| `mobile-app` | 1 PM + 1 架构师 + 2 RD + 1 QA + 1 Designer | 移动端应用 |
| `data-pipeline` | 1 PM + 1 架构师 + 2 RD + 1 DE | 数据工程项目 |
| `fullstack` | 1 PM + 1 架构师 + 1 FE + 1 BE + 1 QA | 前后端分离项目 |
| `minimal` | 1 PM + 1 架构师 + 1 RD + 1 QA | 快速验证 |

## 内置角色

| 缩写 | 角色 | 职责方向 |
|------|------|----------|
| `pm` | 项目经理 | 需求接收、业务澄清、最终验收 |
| `architect` | 架构师 | 需求拆解、任务分配、QA 分配、README/变更日志维护、争议审查、最终技术审查 |
| `rd` | 开发工程师 | 代码实现、代码审查 |
| `qa` | 测试工程师 | 测试设计、缺陷跟踪、质量保证 |
| `fe` | 前端工程师 | UI 实现、性能优化、无障碍 |
| `be` | 后端工程师 | API 设计、数据库、服务端逻辑 |
| `de` | 数据工程师 | 数据管道、ETL、数据质量 |
| `designer` | 设计师 | UI/UX 设计、交互规范、视觉稿 |

未在列表中的缩写会被视为自定义角色 — AI 会自动生成合适的内容。

## 命令模型

| 命令 | 引导流程内容 |
|------|--------------|
| `/init-team` | 选择语言、团队构成、项目名和角色自定义设置 |
| `/run-team` | 选择语言、新任务或已有 Issue，再选择当前合法的阶段动作 |
| `/update-team` | 选择语言，通过 add/remove/review 循环调整团队，并最终确认 |

这些命令都是交互式的。旧的参数式写法已废弃，应改为重新运行裸命令。
整个流程按阶段门控推进，因此 `/run-team` 只会根据当前 Issue 状态展示下一步合法动作。

## 工作原理

**init-team** 生成结构化的 Markdown 文件来定义你的团队。每个智能体获得：
- **系统提示词** — 包含身份、职责、行为准则和协作指南
- **能力档案** — 记录技能、成长方向和学习日志
- 对共享 **Issue 目录** 的访问方式 — Issue 目录中包含 issue 文件和所有角色的工作日志

`init-team` 现在总是先询问语言，再进入模板或自定义组合、项目命名、可选角色自定义和最终确认。

在 Claude Code 和 OpenAI Codex CLI 中，**run-team** 会读取这些文件，并根据运行模式在当前会话执行或通过平台的 agent/subagent 机制启动当前阶段：
1. 读取相关智能体的提示词、档案和协作规范
2. 通过引导流程为新任务创建 Issue，或继续已有 Issue
3. 只启动当前阶段所需的角色，并注入完整上下文
4. 当前角色执行工作、在 Issue 目录中记录日志，并**自动更新自己的能力档案**（新技能、成长方向、学习记录）
5. 每个阶段结束后暂停，等待你明确确认后再进入下一阶段
6. 按照 PM 需求接收、架构师规划与分配、实现、QA 审查、架构师最终技术审查、PM 最终验收的顺序继续推进

如果 active team 不包含至少一个 PM、至少一个架构师、至少一个开发角色和至少一个 QA 角色，`run-team` 会直接阻止执行，并提示你先运行 `/update-team`。

**update-team** 用来安全地调整已有团队，而不会重写 Issue 历史：
- 把被移除角色归档到 `.ai-team/archive/roles/{id}/`
- 确认后重新生成所有 active 角色的 prompts 和 profiles
- 在 `.ai-team/project/changelog.md` 里追加 team update 记录
- 因为角色 ID 不复用，历史 Issue 和归档引用可以长期保持清晰

智能体之间通过文件进行沟通 — Issue、工作日志，以及文档中的 `@{角色编号}` 提及。

**自我学习：** 每轮工作结束后，每个智能体会反思自己学到了什么，并更新 `profiles/{id}.md`。随着时间推移，每个智能体的档案会成为丰富的技能和经验记录，让后续的任务分配更加精准。

## 许可证

Apache License 2.0 — 详见 [LICENSE](LICENSE)。
