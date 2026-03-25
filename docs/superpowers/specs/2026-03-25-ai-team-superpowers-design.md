# AI Team Superpowers Integration Design

## Summary

This design adds a lightweight Superpowers-compatible workflow layer to AI Team for both Codex and Claude Code.

The integration follows a hybrid model:

- Prefer externally installed Superpowers skills when they are available
- Fall back to AI Team's built-in compatible workflow rules when they are not
- Apply the same behavior model to both platform variants

The goal is not to mirror the full Superpowers ecosystem. The goal is to make AI Team teams work with a small, opinionated set of high-value workflows:

- `brainstorming`
- `writing-plans`
- `systematic-debugging`
- `test-driven-development`
- `verification-before-completion`

## Goals

- Make PM work start with brainstorming
- Make architect work start with plan writing
- Make developer work default to TDD
- Make QA work centered on test design, execution, and verification
- Support both Codex and Claude Code
- Prefer external Superpowers skills but keep AI Team usable without them
- Keep the change set small and focused on team workflow quality

## Non-Goals

- Vendoring the full Superpowers repository into AI Team
- Reproducing every Superpowers skill or reference file
- Changing the overall six-stage AI Team workflow
- Introducing platform-specific behavior differences beyond tool syntax

## Role Workflow Contract

AI Team will define the following role-to-workflow mapping as a hard behavioral contract:

- `pm` -> `brainstorming`
- `architect` -> `writing-plans`
- `rd` / `fe` / `be` -> `test-driven-development`
- `qa` -> testing workflow plus `verification-before-completion`

For bug fixes, regressions, and production issue investigation:

- `rd` / `fe` / `be` must run `systematic-debugging` before entering TDD

## Stage Mapping

The existing AI Team issue flow remains intact, but each stage gains an explicit workflow method:

1. PM requirement intake -> brainstorming
2. Architect planning -> writing plans
3. Implementation -> TDD
4. QA review -> testing and verification
5. Architect final review -> technical review and completion verification
6. PM acceptance -> acceptance based on completed prior stages

## External Skill Preference and Fallback

AI Team should use a hybrid execution policy:

1. Detect whether the relevant external Superpowers skill is available in the current platform
2. If available, instruct the role to use that external skill directly
3. If unavailable, require the role to follow the AI Team built-in compatible workflow described in generated prompts and team docs

This fallback must be explicit and must never block the workflow.

### Availability Detection Policy

Availability detection should stay documentation-driven rather than implementation-heavy.

- If the current platform exposes a matching skill by the expected name, treat it as available
- If the current platform does not expose that skill, use the AI Team fallback workflow
- AI Team should not assume a specific filesystem layout for third-party skill installations
- AI Team should not fail the stage simply because an external skill cannot be confirmed

## Built-In Compatible Workflow Definitions

AI Team will define minimal built-in versions of the required workflows.

### PM Brainstorming Fallback

The PM must:

- clarify goals, constraints, and success criteria
- propose 2-3 approaches with trade-offs
- confirm the preferred direction before handing work to the architect
- record the agreed requirement framing in the issue

### Architect Planning Fallback

The architect must:

- produce a concrete implementation plan
- break work into stages and assignments
- define acceptance criteria and risks
- record developer and QA ownership in the issue

### Developer TDD Fallback

The developer must:

- write or update a failing test first
- implement the smallest change that makes the test pass
- run targeted verification before claiming completion
- document files changed and verification performed

### Developer Debugging Fallback

For bug-oriented tasks, the developer must:

- reproduce the issue
- identify likely root cause
- capture debugging evidence
- only then move into the TDD loop

### QA Testing Fallback

The QA role must:

- design test coverage for the assigned change
- execute relevant automated and manual checks
- record evidence, defects, and pass/fail results
- use verification-before-completion before sign-off

QA testing is primarily AI Team-native behavior. When available, `verification-before-completion` strengthens the sign-off step, but QA should not depend on an external skill literally named `testing`.

## Generated Team Artifact

`init-team` and `update-team` will generate:

- `.ai-team/superpowers.md`

This file acts as the team-level workflow contract. It will include:

- the role mapping
- the stage mapping
- the external-skill-preferred policy
- the fallback rules
- bug-task handling rules
- completion verification expectations

## Prompt Template Changes

Generated prompts in `.ai-team/prompts/{id}.md` will include role-specific Superpowers instructions.

Examples:

- PM prompts must direct the role to start requirement work with brainstorming
- Architect prompts must direct the role to start planning work with writing plans
- Developer prompts must direct the role to use TDD and to prepend debugging for bug work
- QA prompts must direct the role to design and execute tests and verify before sign-off

These prompt changes should work even if external Superpowers skills are unavailable.

## Run-Team Context Injection Changes

`run-team` will inject `.ai-team/superpowers.md` into every launched role context alongside:

- role prompt
- role profile
- issue content
- collaboration guidelines

This ensures the team-wide workflow contract is visible during execution rather than only at initialization time.

## Update-Team Changes

`update-team` must regenerate `.ai-team/superpowers.md` whenever the active team changes.

It must also refresh prompts and profiles so that role workflow expectations stay aligned with the current team composition.

## File-Level Implementation Scope

The implementation should update these files:

- `skills/codex/init-team/SKILL.md`
- `skills/codex/run-team/SKILL.md`
- `skills/codex/update-team/SKILL.md`
- `skills/claude-code/init-team/SKILL.md`
- `skills/claude-code/run-team/SKILL.md`
- `skills/claude-code/update-team/SKILL.md`
- `README.md`
- `README_zh.md`

The generated `.ai-team/` structure should also be updated to include `.ai-team/superpowers.md`.

`collaboration.md` and issue-stage descriptions should also be refreshed so the visible team workflow language matches the new role contract.

## Documentation Changes

The READMEs should document:

- the new Superpowers-compatible workflow behavior
- the preferred external-skill usage model
- the fallback behavior when external skills are absent
- the role mapping for PM, architect, developer, and QA
- the fact that both Codex and Claude Code variants support the same method

## Validation Plan

Validation should cover four levels.

### 1. Skill Text Validation

Confirm the Codex and Claude Code skills both explicitly define:

- PM -> brainstorming
- Architect -> writing-plans
- Developer -> TDD
- QA -> testing and verification
- bug work -> debugging before TDD

### 2. Generated Artifact Validation

Confirm `init-team` output includes:

- `.ai-team/superpowers.md`
- prompts that contain role-specific workflow instructions

### 3. Workflow Validation

Confirm `run-team` stage descriptions and execution rules align with:

- Stage 1 PM brainstorming
- Stage 2 architect planning
- Stage 3 developer TDD
- Stage 4 QA testing
- Stage 5 architect review
- Stage 6 PM acceptance

### 4. Fallback Validation

Confirm the docs and skill text clearly state:

- use external Superpowers when available
- otherwise use AI Team's built-in compatible workflow
- missing external skills do not stop execution

## Risks

- If the prompt wording is too soft, roles may ignore the intended workflow
- If the fallback definitions are too vague, behavior will drift across platforms
- If role coverage is limited to `rd` only, split roles like `fe` and `be` may miss TDD/debugging rules

## Decisions

- Use a hybrid integration model
- Keep the supported workflow set intentionally small
- Enforce the contract through both team-level docs and role-level prompts
- Preserve the existing AI Team stage model rather than redesigning it
