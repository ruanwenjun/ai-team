# Architect Role Defaults Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a built-in `architect` role to every default preset and update the generated/default workflow so architect leads delivery while user approval gates every stage transition.

**Architecture:** The change is documentation- and prompt-driven. We will update the two `init-team` skill definitions, the two `run-team` skill definitions, and the bilingual README files so the role catalog, generated collaboration rules, examples, and workflow descriptions all agree on the new architect-led gated process.

**Tech Stack:** Markdown skills, repository documentation, shell-based verification with `rg`

---

### Task 1: Update `init-team` defaults and built-in role guidance

**Files:**
- Modify: `skills/codex/init-team/SKILL.md`
- Modify: `skills/claude-code/init-team/SKILL.md`
- Test: `rg -n "architect|web-standard|mobile-app|data-pipeline|fullstack|minimal|Task Workflow|Conflict Resolution" skills/codex/init-team/SKILL.md skills/claude-code/init-team/SKILL.md`

- [ ] **Step 1: Write the failing verification target**

Document the expected strings before editing:
- Every preset composition includes `1architect`
- Built-in role table includes `architect`
- PM guidance no longer says PM decomposes or coordinates engineering execution
- Architect guidance covers README/changelog ownership, developer assignment, QA assignment, dispute review, and final technical review
- Multi-developer implementation is described as one gated stage that can include multiple developers in parallel but cannot advance to QA without explicit user approval

- [ ] **Step 2: Run verification to confirm current text is missing**

Run: `rg -n "1architect|\\| \`architect\` \\| Architect \\||README\\.md|changelog\\.md|explicit user approval|all assigned developers" skills/codex/init-team/SKILL.md skills/claude-code/init-team/SKILL.md`
Expected: no matches

- [ ] **Step 3: Write the minimal documentation changes**

Update both `init-team` skills so preset compositions, built-in role tables, behavioral guidance, collaboration examples, dynamic workflow rules, and conflict resolution text all reflect the architect-led default workflow and user approval gates.

- [ ] **Step 4: Run verification to confirm the new text is present**

Run: `rg -n "1architect|Requirement intake and clarification, final acceptance|README\\.md|changelog\\.md|project/decisions/|explicit user approval|all assigned developers" skills/codex/init-team/SKILL.md skills/claude-code/init-team/SKILL.md`
Expected: matches for presets, PM restriction, architect ownership, and gated workflow wording

### Task 2: Update `run-team` staged workflow and acceptance loop

**Files:**
- Modify: `skills/codex/run-team/SKILL.md`
- Modify: `skills/claude-code/run-team/SKILL.md`
- Test: `rg -n "Single-Session Mode|Workflow Execution|Acceptance Flow|user approval|architect|PM acceptance" skills/codex/run-team/SKILL.md skills/claude-code/run-team/SKILL.md`

- [ ] **Step 1: Write the failing verification target**

Define the expected staged flow:
- Single-session mode becomes staged rather than "launch everyone and summarize later"
- The workflow pauses after each role/stage for explicit user approval
- Architect appears as the technical lead before PM acceptance
- If multiple developers are assigned, they are treated as one implementation stage and QA waits for user-approved stage completion

- [ ] **Step 2: Run verification to confirm the current wording is outdated**

Run: `rg -n "After all agents complete|Approval \\(\"approved\"|PM reviews|launch multiple" skills/codex/run-team/SKILL.md skills/claude-code/run-team/SKILL.md`
Expected: matches showing the old all-at-once acceptance model

- [ ] **Step 3: Write the minimal documentation changes**

Revise both `run-team` skills to describe the architect-led stage order, user approval gates between role handoffs, stage-specific relaunch behavior, and architect final review before PM acceptance.

- [ ] **Step 4: Run verification to confirm the new staged wording is present**

Run: `rg -n "explicit user approval|architect final review|PM acceptance|all assigned developers|stage" skills/codex/run-team/SKILL.md skills/claude-code/run-team/SKILL.md`
Expected: matches for gated stage flow, architect review, and multi-developer stage wording

### Task 3: Update bilingual README documentation

**Files:**
- Modify: `README.md`
- Modify: `README_zh.md`
- Test: `rg -n "architect|架构师|profiles/architect\\.md|prompts/architect\\.md|worklog/architect/|web-standard|fullstack|minimal|PM acceptance|用户确认" README.md README_zh.md`

- [ ] **Step 1: Write the failing verification target**

Document the expected doc updates:
- Preset table includes architect in every composition
- Role table includes architect / 架构师
- Examples and generated layouts show architect profile/prompt/worklog artifacts
- Workflow description mentions architect-led delivery and user approval gates
- Any role-specific example that shows the full gated chain includes PM plus architect, not just implementation roles

- [ ] **Step 2: Run verification to confirm the current docs are missing it**

Run: `rg -n "architect|架构师|profiles/architect\\.md|prompts/architect\\.md|worklog/architect/" README.md README_zh.md`
Expected: no architect-specific layout matches

- [ ] **Step 3: Write the minimal documentation changes**

Update the English and Chinese README files so installation/usage examples, preset descriptions, role tables, and workflow narrative all match the new default team shape and gated process.

- [ ] **Step 4: Run verification to confirm the new docs are present**

Run: `rg -n "architect|架构师|profiles/architect\\.md|prompts/architect\\.md|worklog/architect/|user approval|用户确认|PM acceptance|最终验收" README.md README_zh.md`
Expected: matches for architect role, architect artifacts, and gated workflow language

### Task 4: Verify cross-file consistency

**Files:**
- Test: `README.md`
- Test: `README_zh.md`
- Test: `skills/codex/init-team/SKILL.md`
- Test: `skills/claude-code/init-team/SKILL.md`
- Test: `skills/codex/run-team/SKILL.md`
- Test: `skills/claude-code/run-team/SKILL.md`

- [ ] **Step 1: Run repo-wide consistency checks**

Run: `rg -n "1architect|architect final review|PM acceptance|用户确认|user approval|all assigned developers" README.md README_zh.md skills/codex/init-team/SKILL.md skills/claude-code/init-team/SKILL.md skills/codex/run-team/SKILL.md skills/claude-code/run-team/SKILL.md`
Expected: all key files contain the new model

- [ ] **Step 2: Manually review stage ordering**

Confirm the exact ordered sequence appears consistently across the touched files:
`PM requirement intake -> architect planning/assignment -> implementation -> QA review -> architect final review -> PM acceptance`

- [ ] **Step 3: Capture any residual risks**

If any ambiguity remains after the edits, note it separately rather than leaving it unresolved in the plan.
