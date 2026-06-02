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

## Language Structure

```markdown
### [Entrypoint: agentlang]   ← Application entry

#### def: ActionName       ← Define an action (no params, agent infers)
1. step 1
2. step 2

#### def: ActionName(arg0, arg1)  ← Define an action (explicit params)
1. do something with arg0
2. if condition: return arg1

#### Main                  ← Execution entry, program starts here
1. call ActionName: natural language intent
2. call ActionName("value"): call with arguments
```

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

## Markdown Writing Rules

1. `### [Entrypoint: agentlang]` — H3 + brackets = application entry, identifies this as an AgentLang program
2. `#### def: Name` — H4 + `def:` = action definition
3. `#### Main` — H4 + `Main` = execution entry
4. Numbered lists = sequential steps
5. `parallel:` followed by `-` unordered list = parallel steps
6. Description after colon = optional intent context for the agent

---

## Full Example

```markdown
### [Entrypoint: agentlang]
#### def: FetchData(url)
1. get data from url
2. if fails: repeat until success or 3 attempts
3. return data

#### def: CleanData
1. remove duplicates
2. fill missing values
3. if data is empty: return error

#### def: GenerateReport
1. parallel:
   - call Analyst: analyze trends
   - call Visualizer: generate charts
2. merge results into report
3. return report

#### Main
1. call FetchData("https://api.example.com/users"): fetch raw data
2. call CleanData: clean the previous result
3. confirm: data cleaning complete, continue generating report?
4. call GenerateReport: generate final report
5. if report has errors: return "pipeline failed"
```

---

## Agent Parsing Guide

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
