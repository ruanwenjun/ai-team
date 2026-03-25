# Architect Role Default Workflow Design

**Date:** 2026-03-25

## Goal
Add `architect` as a first-class built-in AI Team role, include it in every preset template by default, and shift the default team workflow so PM handles requirement intake while architect leads execution planning, review, and technical decision-making.

## Approved Scope
- Add `architect` to all preset templates in both `skills/claude-code/init-team/SKILL.md` and `skills/codex/init-team/SKILL.md`.
- Add `architect` to the built-in role tables and role guidance in both `init-team` skills.
- Redefine default responsibilities:
  - `pm`: requirement intake, business clarification, final acceptance.
  - `architect`: requirement decomposition, task assignment, QA assignment, `project/README.md` maintenance, `project/changelog.md` maintenance, technical dispute review in `project/decisions/`, final technical review before PM acceptance.
- Update generated collaboration rules so the default staged flow is:
  1. PM submits or clarifies the requirement.
  2. Architect breaks the requirement into subtasks and assigns developers plus QA ownership.
  3. Assigned development roles implement.
  4. Assigned QA role reviews/tests.
  5. Architect performs final technical review.
  6. PM performs final acceptance.
- Add explicit user approval gates between every stage. The next role or stage does not start automatically after the current role finishes.
- Update both `run-team` skills so the documented workflow matches this gated stage model.
- Update `README.md` and `README_zh.md` to document the new default presets, architect role, example outputs, and gated workflow.

## Design Decisions

### Architect as a built-in role
`architect` should no longer rely on the custom-role fallback. Making it built-in ensures consistent prompts, collaboration rules, preset membership, and documentation across platforms.

### Architect-led execution workflow
The default workflow changes from PM-led task decomposition to architect-led delivery orchestration. This aligns technical planning, QA routing, dispute handling, and final engineering review under one role.

### User-gated progression
Single-session team execution should stop after each completed stage and wait for explicit user confirmation before proceeding. This applies to the handoffs from PM to architect, architect to implementation, implementation to QA, QA to architect review, and architect review to PM acceptance.

### Multi-developer teams
When multiple developer roles are present in the implementation stage, architect may assign work to one or more developers. Each participating role still reports completion separately, and the workflow does not advance to QA until the user approves the next step.

## Files To Change
- `skills/claude-code/init-team/SKILL.md`
- `skills/codex/init-team/SKILL.md`
- `skills/claude-code/run-team/SKILL.md`
- `skills/codex/run-team/SKILL.md`
- `README.md`
- `README_zh.md`

## Verification Strategy
- Confirm every preset composition now includes `1architect`.
- Confirm both built-in role tables include `architect`.
- Confirm PM text is limited to requirement intake, business clarification, and final acceptance.
- Confirm architect text explicitly owns task decomposition, developer assignment, QA assignment, `project/README.md`, `project/changelog.md`, and dispute review in `project/decisions/`.
- Confirm the stage order appears exactly as PM requirement intake -> architect planning/assignment -> implementation -> QA review -> architect final review -> PM acceptance.
- Confirm collaboration and `run-team` sections say the next stage does not start until the user explicitly approves it.
- Confirm multi-developer implementation stages are described as a single gated stage: architect may assign one or more developer roles in parallel, but QA does not start until all assigned developers finish and the user approves advancing.
- Confirm README examples include architect artifacts in the generated layout and examples.
