# Agent Bridge Protocol v0.6

> File-as-message, write-as-push, poll-as-trigger.
> Any system that can read and write files can participate.

## Directory Structure

```
.agent-bridge/
├── PROTOCOL.md          ← This file
├── chat.md              ← Group chat (any agent may append)
├── board/
│   └── tasks.json       ← Task board (planner writes, others read)
├── results/
│   └── {task_id}.json   ← One result file per task (executor writes)
├── inbox/
│   └── {agent_id}.md    ← Per-agent inbox (append-only)
├── materials/
│   └── {task_id}/       ← Task-related materials (prompts, data, rubrics)
├── chronicles/
│   ├── YYYY-MM-DD.md    ← Daily records (chronicler writes)
│   └── INDEX.md         ← Chronicle index
└── config/
    └── agents.json      ← Agent registry
```

## Core Rules

1. **One file, one writer at a time** -- avoid concurrent writes to the same file
2. **tasks.json is planner-only** -- others read but never write
3. **results/{task_id}.json is executor-only** -- naturally isolated, no conflicts
4. **inbox/ is append-only** -- never modify or delete existing content
5. **chat.md is append-only** -- same rule

## Agent Registry

Agents self-register in `config/agents.json`:

```json
{
  "agents": [
    {
      "id": "planner",
      "name": "Planner",
      "backend": "claude-code",
      "role": "Plans tasks, reviews results",
      "watch": ["results/", "inbox/planner.md"]
    },
    {
      "id": "executor",
      "name": "Executor",
      "backend": "claude-code",
      "role": "Executes tasks, submits results",
      "watch": ["board/tasks.json", "inbox/executor.md"]
    },
    {
      "id": "chronicler",
      "name": "Chronicler",
      "backend": "any",
      "role": "Documents decisions and history",
      "watch": ["results/", "inbox/chronicler.md"]
    }
  ]
}
```

The `backend` field is informational only. The protocol does not care what system an agent runs on.

## Task Format

### tasks.json (planner writes)

```json
{
  "last_updated": "2026-04-17T10:00:00",
  "tasks": [
    {
      "id": "T001",
      "title": "One-line title",
      "description": "What is needed and how to verify",
      "assignee": "executor",
      "created_by": "planner",
      "created_at": "2026-04-17T10:00:00",
      "tags": ["feature"]
    }
  ]
}
```

There is no `status` field. Task state is determined by result files:
- `results/T001.json` does not exist --> pending
- `results/T001.json` exists with `"status": "completed"` --> done
- `results/T001.json` exists with `"status": "failed"` --> error (check `error` field)
- `results/T001.json` exists with `"status": "blocked"` --> blocked (check `reason` field)
- `results/T001.json` exists with `"status": "rejected"` --> refused (check `reason` + `suggestion`)

### results/{task_id}.json (executor writes)

**Completed:**
```json
{
  "task_id": "T001",
  "status": "completed",
  "summary": "What was done",
  "deliverables": ["path/to/file1", "path/to/file2"],
  "issues": [],
  "completed_by": "executor",
  "completed_at": "2026-04-17T10:30:00"
}
```

**Failed:**
```json
{
  "task_id": "T002",
  "status": "failed",
  "error": "What went wrong",
  "completed_by": "executor",
  "completed_at": "2026-04-17T10:35:00"
}
```

**Rejected:**
```json
{
  "task_id": "T003",
  "status": "rejected",
  "reason": "Why the task cannot be completed as specified",
  "suggestion": "What to do instead",
  "completed_by": "executor",
  "completed_at": "2026-04-17T10:40:00"
}
```

When rejecting, the executor MUST also leave a message in the planner's inbox (`inbox/planner.md`) explaining the rejection. The planner can then revise the task (delete the old result, update the task) and re-issue it.

### inbox/{agent_id}.md (append-only)

```markdown
# Agent Inbox

## 2026-04-17 10:00 @planner
Message content here.

## 2026-04-17 10:05 @executor
Reply content here.
```

### chat.md (append-only)

Same format as inbox. Any agent can append.

## Workflows

### Autonomy Principle

**Small things: do them directly. Big things: use the task flow.**

| Type | Example | Action |
|------|---------|--------|
| Trivial | git commit, save file, typo fix | Do it, no task needed |
| Small decision | naming, file location, tool choice | Decide yourself |
| Real task | refactor module, write tests, research | Create formal task |
| External info | docs lookup, web search | Delegate to capable agent |

### Planner

1. Read `results/` to see completed work
2. Read `inbox/planner.md` for requests and issues
3. Write tasks to `board/tasks.json`
4. Review results and issue follow-up tasks

### Executor

1. Read `board/tasks.json`, find tasks assigned to you
2. Skip tasks that already have `results/{task_id}.json`
3. Execute the task
4. Write `results/{task_id}.json`
5. If stuck, write `inbox/planner.md`

### Chronicler

1. Read `results/` to understand latest work
2. Read `inbox/chronicler.md` for recording requests
3. Write daily records to `chronicles/YYYY-MM-DD.md`

### New Agent

1. Read `PROTOCOL.md`
2. Read `config/agents.json`
3. Append your identity to `config/agents.json`
4. Start working in your role

## Task Description Discipline

When writing tasks for executors:

1. **Only write requirements and acceptance criteria, not implementation steps**
   - Specify WHAT is needed and HOW to verify correctness
   - Let the executor decide HOW to achieve it

2. **Explain or avoid jargon**
   - If you must use a technical term, add a plain-language explanation

3. **Show concrete examples for critical constraints, not adjectives**
   - Ambiguous rules lead to misinterpretation
   - Give 1-2 concrete examples instead of describing in abstract terms

4. **After writing, delete every sentence that teaches HOW to do the work**
   - What remains should be: requirements + acceptance criteria

5. **Less is more**
   - If 150 words suffice, don't write 1500
   - Let the executor ask questions rather than pre-emptively addressing every edge case

## Three-Layer Separation

6. **Implementation tasks verify "it works", not "it's good"**
   - Acceptance: flow runs, fields have values, no errors
   - Quality gates like "accuracy >= 85%" belong in separate test tasks

7. **Complex materials go in folders, not task descriptions**
   - Put prompts, examples, data, rubrics in `materials/{task_id}/`
   - Task description references the folder, not the contents
   - Materials can be iterated independently without changing the task

8. **Test tasks and implementation tasks are separate**
   - Implementation: build it (acceptance: "runs without errors")
   - Test: validate quality (acceptance: "meets specific metrics")
   - These are fundamentally different kinds of work

## Trigger Mechanisms

Agents decide how to poll for changes:

- Claude Code: `/loop 5m /check` for periodic checking
- Other systems: file watchers, cron jobs, manual triggers
- No polling also works: user triggers manually when needed

Recommended human commands:

| Command | Action |
|---------|--------|
| `/check` | Read bridge state and report (role-specific) |
| `/report` | Output full task status table |
| `/log` | Append current work summary to chat.md |

## Protocol Updates

When the protocol changes:
1. Update `PROTOCOL.md`
2. Announce changes in `chat.md`
3. Write to each affected agent's inbox

Agents sync on next `/check`.

## Protocol Versions

- v0.1 -- Initial version, single results.json
- v0.2 -- Results split to per-task files, removed status field, generalized identity
- v0.3 -- Added global bridge level, dynamic discovery
- v0.4 -- Added autonomy principle, rejected status, update notification
- v0.5 -- Added task description discipline (5 rules)
- v0.6 -- Added three-layer separation (3 rules)
