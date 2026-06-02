---
name: auto-research
description: Run a persistent autonomous research workflow from `/AutoResearch <TaskDescription>`. Use when the user asks for AutoResearch, auto research, task decomposition, durable research state, long-running investigation, or research-to-implementation planning with checkpoint commits.
---

# Auto Research

Use this skill when the user invokes:

`/AutoResearch <TaskDescription>`

This skill turns an ambiguous research or engineering task into a confirmed plan, maintains durable state in `./auto_research`, advances work through explicit checkpoints, and validates progress against agreed success criteria.

## Core Rules

- Treat `/AutoResearch <TaskDescription>` as the workflow trigger, not as a shell command.
- Do not rely only on conversation memory for critical facts.
- Persist goals, plans, findings, decisions, validation status, blockers, and handoff notes under `./auto_research`.
- Reread persistent state before every major step.
- Ask the user to confirm unclear goals, scope changes, architectural decisions, destructive actions, and git commits.
- Continue iterating until the Definition of Done is satisfied, the task is blocked, or the user stops the workflow.
- Use git commits only at meaningful checkpoints and only after explicit user authorization unless the session contract pre-approves checkpoint commits.

## Phase 1: Parse The Request

Extract the task from `/AutoResearch <TaskDescription>`.

Identify:

- Primary objective
- Expected deliverable
- Target repo, product area, or system
- Constraints, deadlines, and non-goals
- Whether code changes are expected
- Whether commits are allowed during the workflow
- What evidence will prove success

If any item is unclear, ask the user before broad execution.

## Phase 2: Create Persistent State

Create or reuse a session directory in the active project root:

`./auto_research/sessions/<YYYY-MM-DD>-<task-slug>/`

Maintain these files as durable working memory:

- `contract.md`: confirmed goal, Definition of Done, scope, constraints, validation requirements, commit policy
- `plan.md`: decomposed phases, task breakdown, ordering, dependencies, active workstream
- `state.md`: current status, next action, blockers, open questions, last checkpoint
- `findings.md`: evidence from code, docs, commands, web, issues, PRs, logs, or experiments
- `decisions.md`: user-confirmed decisions, tradeoffs, rejected options, rationale
- `validation.md`: tests, checks, reproduction steps, acceptance evidence, final verification
- `handoff.md`: final summary, remaining risks, follow-ups, useful references
- `artifacts/`: optional supporting outputs that are too large for the main notes

Also maintain:

`./auto_research/index.md`

Use `index.md` to list active and completed sessions with links to their session directories.

## Phase 3: Confirm The Research Contract

Before deep work, write `contract.md` and ask the user to confirm it.

Use this template:

```markdown
# AutoResearch Contract: [Task]

## Goal

[One-sentence target outcome.]

## Definition Of Done

- [Observable outcome 1]
- [Observable outcome 2]
- [Validation or evidence requirement]

## Scope

In scope:
- [...]

Out of scope:
- [...]

## Constraints

- [...]

## Validation

- [...]

## Git Policy

- Checkpoint commits: yes/no
- Commit approval: ask before every commit / pre-approved for confirmed checkpoints
- Files intended for commit: [...]
- Files excluded from commit: secrets, local config, unrelated changes, noisy artifacts
```

Do not begin broad execution until the user confirms or corrects this contract.

## Phase 4: Decompose The Task

Write `plan.md` with a practical task breakdown.

Use this structure:

```markdown
# AutoResearch Plan

## Phase 1: Context Gathering

- [ ] Identify relevant files, docs, services, APIs, issues, PRs, or external sources
- [ ] Capture initial facts in `findings.md`

## Phase 2: Approach Selection

- [ ] List viable approaches
- [ ] Identify risks, decision points, and required user confirmations

## Phase 3: Execution

- [ ] Perform the highest-value research or implementation step
- [ ] Update `state.md` after each meaningful result

## Phase 4: Validation

- [ ] Run agreed checks
- [ ] Record evidence in `validation.md`

## Phase 5: Handoff

- [ ] Summarize outcome
- [ ] Document remaining risks and follow-ups
```

Use TODO tracking for multi-step work when available.


## Phase 5: Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- You MUST be very thorough in your thinking and comprehensively decompose the problem to resolve the root cause.

## Rehydration Protocol

Before each major step, reread persistent state:

1. Read `contract.md`
2. Read `plan.md`
3. Read `state.md`
4. Read `decisions.md`
5. Compare the intended next action against the Definition of Done
6. If there is drift, stop and correct the plan before continuing

Major steps include:

- Starting a new execution loop
- Changing scope
- Making an architecture or implementation decision
- Editing code
- Running final validation
- Creating a git commit
- Resuming after interruption or context loss
- Writing the final response

After each meaningful finding, decision, or validation result, update the relevant state file before continuing.

## Execution Loop

Repeat until the Definition of Done is satisfied:

1. Rehydrate from persistent state.
2. Select the next highest-value task from `plan.md`.
3. Perform only the next bounded action.
4. Record evidence or output in the appropriate state file.
5. Update `state.md` with status, next action, and blockers.
6. Update `decisions.md` only for user-confirmed decisions.
7. Compare progress against the Definition of Done in `contract.md`.
8. Continue if the next step is clear; ask the user if the path or scope is unclear.

Do not let this loop become open-ended trial and error. If the same gap remains after two bounded attempts, stop the loop, record the blocker in `state.md`, and ask the user how to proceed.

Use this progress update format:

```markdown
Progress:
- Completed: [...]
- Learned: [...]
- Changed plan: [...]
- Remaining gap: [...]
- Next step: [...]
```

## User Confirmation Points

Ask the user for confirmation when:

- The goal, deliverable, or Definition of Done is ambiguous.
- Scope expands or changes.
- There are multiple viable directions with meaningful tradeoffs.
- A decision affects architecture, data model, public API, cost, security, or timeline.
- A git commit is about to be created unless checkpoint commits were pre-approved in `contract.md`.
- The workflow reaches a natural milestone and the next phase changes from research to implementation.
- Continuing requires destructive, irreversible, privileged, or externally visible actions.

## Git Checkpoints

When checkpoint commits are allowed, commit only after meaningful validated milestones.

Good checkpoint examples:

- Research contract and plan documented
- Key finding or decision identified with evidence
- Prototype or implementation completed
- Tests or validation added
- Final cleanup completed

Before each commit:

1. Rehydrate from persistent state.
2. Inspect git status.
3. Inspect staged and unstaged diffs.
4. Exclude unrelated files, secrets, local config, and noisy artifacts.
5. Run relevant validation if available.
6. Draft a concise commit message focused on why the change matters.
7. Ask for user authorization unless pre-approved by `contract.md`.

Suggested commit messages:

```text
research: document approach for [topic]

Capture the agreed goal, findings, and validation path so the next phase can proceed from a stable checkpoint.
```

```text
feat: implement [capability]

Add [behavior] based on the confirmed AutoResearch plan and validate it with [test/check].
```

## Completion Criteria

The workflow is complete only when:

- The Definition of Done is satisfied.
- Evidence or validation has been recorded in `validation.md`.
- Important findings and decisions are recorded.
- Open risks are documented.
- The user receives a concise final summary.
- Requested checkpoint commits have been created or intentionally skipped.

Final response format:

```markdown
## Result

[One-paragraph summary of the completed outcome.]

## Evidence

- [Key evidence or validation result]
- [Relevant file, test, doc, source, or command result]

## Decisions

- [Important user-confirmed decision]
- [Important tradeoff or scope choice]

## Remaining Risks

- [Known limitation or follow-up, if any]
```

## Stop Conditions

Stop the loop and ask the user what to do if:

- Required access is missing.
- The task conflicts with safety, policy, or repo constraints.
- Validation repeatedly fails for the same reason.
- The goal changes materially.
- Continuing would require destructive git operations.
- The user has not approved a necessary decision.
- The persistent state contradicts the current instruction and the conflict cannot be resolved locally.
