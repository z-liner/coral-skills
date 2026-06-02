---
name: AgentLang
version: 0.1.0
author: jcoral
description: A lightweight instruction language for agents, written directly in markdown.
license: MIT
tags: [agent, language, markdown, skill]
language: en
dependencies: none
---

# AgentLang

Uses markdown heading hierarchy as structural skeleton and numbered lists as execution steps. Friendly to non-programmers, unambiguous to agents.

---

## Keywords

8 keywords covering all control flow:

| Keyword | Purpose | Example |
|---------|---------|---------|
| `def:` | Define an action | `#### def: FetchData(url)` |
| `call` | Invoke an action (local or other agent) | `call FetchData: get user list` |
| `if` | Conditional | `if fails: return` |
| `else` | Alternative branch | `else: retry` |
| `repeat` | Loop | `repeat until done` |
| `parallel` | Parallel execution | `parallel:` + sub-step list |
| `confirm` | Pause for user confirmation or input | `confirm: continue?` |
| `return` | Terminate current action and output result | `return data` |

---

## Writing Rules

1. `### [Entrypoint: agentlang]` — H3 + brackets = application entry, identifies this as an AgentLang program
2. `#### def: Name` — H4 + `def:` = action definition
3. `#### Main` — H4 + `Main` = execution entry
4. Numbered lists = sequential steps
5. `parallel:` followed by `-` unordered list = parallel steps
6. Description after colon = optional intent context for the agent

---

## Parsing Guide

Parsing rules (execute in order):

1. Scan all `#### def:` to build an action index
2. Start sequential execution from `#### Main`
3. On `call X` → resolve as "execute X", no distinction between local def and external agent (unified model)
4. On `call X: description` or `call X("arg"): description` → description serves as execution context
5. On `parallel:` → execute child `-` list items concurrently
6. On `confirm` → pause execution, wait for user response; if user declines, terminate current flow
7. Natural language in steps → agent interprets and executes autonomously
8. Parameters: inferred from context by default; explicit passing uses `Name(arg0, arg1)` form

### Structure Recognition

| Markdown Element | Semantics |
|-----------------|-----------|
| `### [Entrypoint: agentlang]` | Application entry, identifies an AgentLang program |
| `#### def: Name` | Action definition start |
| `#### def: Name(args)` | Action definition with parameters |
| `#### Main` | Execution entry |
| `1. 2. 3.` numbered list | Sequential steps |
| `- ` unordered list (under `parallel:`) | Parallel steps |

### Execution Model

- Steps execute in numbered order unless `parallel:` is encountered
- `call` is the sole invocation mechanism, handling both local actions and external agents uniformly
- `confirm` pauses execution flow, waits for user response; user decline terminates current flow
- `if` / `else` provide branching, agent decides based on runtime state
- `repeat` creates loops, requires termination condition (`until`, attempt limit)
- `return` terminates current action, outputs result to caller
- Each step's return value is implicitly passed to the next step (unless explicitly specified)

---

## Example

### [Entrypoint: agentlang]

#### def: ParseSkill(source)
1. scan source for `### [Entrypoint: agentlang]` to identify skill boundary
2. scan all `#### def:` headings to build action index
3. locate `#### Main` as execution entry
4. return parsed skill structure

#### def: ExecuteStep(step)
1. if step starts with `call`: resolve target action and execute it
2. if step starts with `confirm`: pause and wait for user response; if declined, terminate flow
3. if step starts with `parallel:`: execute child list items concurrently
4. if step starts with `if`: evaluate condition and branch
5. if step starts with `repeat`: loop until termination condition met
6. else: interpret natural language and execute

#### Main
1. call ParseSkill: parse the given AgentLang source document
2. read the first step from Main block
3. repeat until all steps are processed:
   - call ExecuteStep: execute current step
   - if step returns error: return error
   - advance to next step
4. return final result
