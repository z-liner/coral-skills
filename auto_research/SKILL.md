---
name: auto-research
version: 0.1.0
author: jcoral
description: Run a persistent autonomous research workflow from `/autoresearch <TaskDescription>`. Use when the user asks for autoresearch, auto research, task decomposition, durable research state, long-running investigation, or research-to-implementation planning with checkpoint commits.
license: MIT
tags: [research, autonomous, workflow, planning]
language: en
dependencies: none
---

# Auto Research

Turns an ambiguous research or engineering task into a confirmed plan, maintains durable state in `./auto_research`, advances work through explicit checkpoints, and validates progress against agreed success criteria.

---

## Core Rules

- Treat `/autoresearch <TaskDescription>` as the workflow trigger, not as a shell command.
- Do not rely only on conversation memory for critical facts.
- Persist goals, plans, findings, decisions, validation status, blockers, and handoff notes under `./auto_research`.
- Reread persistent state before every major step.
- Ask the user to confirm unclear goals, scope changes, architectural decisions, destructive actions, and git commits.
- Continue iterating until the Definition of Done is satisfied, the task is blocked, or the user stops the workflow.
- Use git commits only at meaningful checkpoints and only after explicit user authorization unless the session contract pre-approves checkpoint commits.

---

## State Files

Session directory: `./auto_research/sessions/<YYYY-MM-DD>-<task-slug>/`

| File | Purpose |
|------|---------|
| `contract.md` | confirmed goal, Definition of Done, scope, constraints, validation requirements, commit policy |
| `plan.md` | decomposed phases, task breakdown, ordering, dependencies, active workstream |
| `state.md` | current status, next action, blockers, open questions, last checkpoint |
| `findings.md` | evidence from code, docs, commands, web, issues, PRs, logs, or experiments |
| `decisions.md` | user-confirmed decisions, tradeoffs, rejected options, rationale |
| `validation.md` | tests, checks, reproduction steps, acceptance evidence, final verification |
| `handoff.md` | final summary, remaining risks, follow-ups, useful references |
| `artifacts/` | optional supporting outputs too large for main notes |

Also maintain `./auto_research/index.md` listing active and completed sessions.

---

### [Entrypoint: agentlang]

#### def: ParseRequest(task)
1. extract primary objective from task description
2. identify expected deliverable, target repo/system, constraints, deadlines, and non-goals
3. determine whether code changes are expected
4. determine whether commits are allowed during the workflow
5. identify what evidence will prove success
6. if any item is unclear: confirm: clarify ambiguous items before proceeding

#### def: CreateState(task_slug)
1. create session directory `./auto_research/sessions/<YYYY-MM-DD>-<task_slug>/`
2. initialize contract.md, plan.md, state.md, findings.md, decisions.md, validation.md, handoff.md
3. update `./auto_research/index.md` with new session entry
4. return session path

#### def: ConfirmContract
1. write contract.md with goal, Definition of Done, scope, constraints, validation, and git policy
2. confirm: please review and confirm the research contract
3. if user corrects: update contract.md with corrections
4. return confirmed contract

#### def: DecomposeTask
1. read contract.md for goal and scope
2. create plan.md with phased task breakdown:
   - Phase 1: Context Gathering — identify relevant files, docs, APIs, issues
   - Phase 2: Approach Selection — list viable approaches, risks, decision points
   - Phase 3: Execution — perform highest-value research or implementation steps
   - Phase 4: Validation — run agreed checks, record evidence
   - Phase 5: Handoff — summarize outcome, document remaining risks
3. return plan

#### def: ThinkBeforeCoding
1. state assumptions explicitly
2. if multiple interpretations exist: present them to user, don't pick silently
3. if a simpler approach exists: say so and push back when warranted
4. if something is unclear: confirm: name what's confusing and ask
5. comprehensively decompose the problem to resolve the root cause

#### def: Rehydrate
1. read contract.md
2. read plan.md
3. read state.md
4. read decisions.md
5. compare intended next action against the Definition of Done
6. if there is drift: stop and correct the plan before continuing
7. return current context

#### def: ExecuteStep
1. select the next highest-value task from plan.md
2. call ThinkBeforeCoding: ensure clarity before action
3. perform only the next bounded action
4. record evidence or output in the appropriate state file
5. update state.md with status, next action, and blockers
6. update decisions.md only for user-confirmed decisions
7. compare progress against the Definition of Done
8. if path or scope is unclear: confirm: how should we proceed?
9. return progress status

#### def: GitCheckpoint
1. call Rehydrate: ensure current state is fresh
2. inspect git status, staged and unstaged diffs
3. exclude unrelated files, secrets, local config, and noisy artifacts
4. run relevant validation if available
5. draft concise commit message focused on why the change matters
6. confirm: approve this commit?
7. create commit
8. return commit result

#### def: Complete
1. verify Definition of Done is satisfied
2. record evidence in validation.md
3. document important findings and decisions
4. document open risks
5. write handoff.md with final summary, remaining risks, follow-ups
6. update index.md with completion status
7. return final summary in format: Result, Evidence, Decisions, Remaining Risks

#### Main
1. call ParseRequest: parse the `/autoresearch <TaskDescription>` input
2. call CreateState: create persistent state directory for this session
3. call ConfirmContract: write and confirm the research contract with user
4. call DecomposeTask: decompose task into actionable plan
5. repeat until Definition of Done is satisfied or user stops:
   - call Rehydrate: reread persistent state
   - call ExecuteStep: execute next bounded action
   - if same gap remains after 2 attempts: confirm: blocked — how to proceed?
   - if checkpoint reached and commits allowed: call GitCheckpoint
6. call Complete: finalize workflow and deliver summary
7. return final result
