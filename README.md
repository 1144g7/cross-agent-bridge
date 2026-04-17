# Agent Bridge

File-based multi-agent collaboration protocol. No server, no API, no runtime dependency.

## Why

Coordinating multiple AI agents is painful. MCP requires a running server. LangGraph locks you into a framework. Copy-pasting context between chat windows is error-prone and doesn't scale.

Agent Bridge uses the simplest possible coordination mechanism: **files on disk**. Any system that can read and write files can participate. No setup, no dependencies, no network.

**Typical scenario**: You have Claude Code for coding, a separate AI tool for documentation, and maybe a custom script for testing. Instead of manually copying context between them, each agent reads and writes to a shared directory. Tasks flow in one direction (planner writes tasks), results flow back (executor writes results), and everyone stays in sync through a group chat file.

## Quick Start

### 1. Create the bridge directory

```bash
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config}
```

### 2. Register your agents

Create `.agent-bridge/config/agents.json`:

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
    }
  ]
}
```

`backend` is just a note for humans. The protocol doesn't care what system an agent runs on.

### 3. Start collaborating

**Planner** writes a task:

```json
// .agent-bridge/board/tasks.json
{
  "last_updated": "2026-04-17T10:00:00",
  "tasks": [
    {
      "id": "T001",
      "title": "Add user authentication",
      "description": "Add login/logout endpoints. Acceptance: POST /login returns a token, GET /me returns user info.",
      "assignee": "executor",
      "created_by": "planner",
      "created_at": "2026-04-17T10:00:00",
      "tags": ["feature"]
    }
  ]
}
```

**Executor** does the work, then writes a result:

```json
// .agent-bridge/results/T001.json
{
  "task_id": "T001",
  "status": "completed",
  "summary": "Added JWT auth with login/logout/me endpoints",
  "deliverables": ["api/auth.py", "tests/test_auth.py"],
  "issues": [],
  "completed_by": "executor",
  "completed_at": "2026-04-17T10:30:00"
}
```

That's it. No server to start, no webhook to configure.

## Core Concepts

### Fragments

The bridge directory contains four types of fragments:

| Fragment | Who writes | Purpose |
|----------|-----------|---------|
| `board/tasks.json` | Planner | Task queue |
| `results/{id}.json` | Executor (one file per task) | Task results |
| `inbox/{agent}.md` | Anyone (append-only) | Direct messages |
| `chat.md` | Anyone (append-only) | Group chat |

### Task Lifecycle

```
tasks.json (planner writes)
    |
    v
results/T001.json exists?  --No-->  pending
    |
    Yes
    |
    +-- status: "completed" --> done
    +-- status: "failed"    --> error (check error field)
    +-- status: "blocked"   --> waiting (check reason field)
    +-- status: "rejected"  --> refused (check reason + suggestion)
```

There is no `status` field in `tasks.json`. Task state is implicit: it depends on whether the result file exists and what it says.

### Autonomy Principle

**Small things: just do them. Big things: formalize.**

| Type | Example | Action |
|------|---------|--------|
| Trivial | git commit, file save, typo fix | Do directly, no task needed |
| Small decision | variable naming, file location | Decide yourself, report after |
| Real task | refactor module, write tests, research | Create a formal task |
| External info | docs lookup, web search | Delegate to an agent with search capability |

The threshold is: if it takes a few lines, don't go through the task flow.

### Task Description Discipline

This is the most important lesson from real usage. **Write what you want, not how to do it.**

1. **Only write requirements and acceptance criteria, not implementation steps**
   - Good: "Benchmark model X on Y scenarios, report Z accuracy"
   - Bad: "Load model, warm up with 1 call, run 5 iterations, take median, iterate over..."

2. **Explain or avoid jargon**
   - "Run once to load the model into memory" is clearer than "warm-up"

3. **Show concrete examples for critical constraints, not adjectives**
   - Bad: "pair adjacent frames" (ambiguous: sliding window or non-overlapping?)
   - Good: "frame 1+2 as pair, 3+4 as pair, 5+6 as pair, discard unpaired"

4. **After writing, re-read and delete every sentence that teaches HOW**
   - What remains should be: "what I want" + "how to verify it's correct"

5. **Less is more**
   - If 150 words suffice, don't write 1500
   - Let the executor ask questions instead of pre-emptively blocking every edge case

**Test**: after writing a task, ask yourself -- would an experienced engineer feel instructed or informed? If the former, revise.

### Three-Layer Separation

6. **Implementation tasks verify "it works", not "it's good"**
   - The goal is: flow runs, fields have values, no errors
   - Don't put quality gates like "accuracy >= 85%" in implementation tasks
   - Quality validation belongs in separate test tasks

7. **Complex materials go in folders, not task descriptions**
   - Prompt drafts, reference examples, input data, scoring rubrics -- put in `.agent-bridge/materials/{task_id}/`
   - Task description just says "materials are in materials/T019/"
   - This keeps descriptions short and materials independently iterable

8. **Test tasks and implementation tasks are separate**
   - Implementation task: build the feature (acceptance: "it runs")
   - Test task: run experiments to validate quality (acceptance: "meets X metric")
   - Testing is a data experiment, different work from building

## Minimal Working Example

This is a complete end-to-end flow from zero:

```bash
# 1. Create bridge
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config,materials}

# 2. Copy example config
cp -r example/* .agent-bridge/

# 3. Planner creates a task (edit board/tasks.json)
# 4. Executor picks it up, does the work, writes results/T001.json
# 5. Planner checks results/
# 6. Repeat
```

For a concrete walkthrough, see the `example/` directory in this repository.

## Comparison

| Aspect | Agent Bridge | MCP | LangGraph | Manual |
|--------|-------------|-----|-----------|--------|
| Setup | mkdir | Install + run server | Install + configure | None |
| Dependencies | None | Python/JS server | Python + framework | None |
| Cross-system | Any (file-based) | Same process/network | Same framework | Copy-paste |
| Concurrency | File-per-task isolation | Connection management | Graph state | Error-prone |
| Persistence | Files (permanent) | In-memory or custom | Checkpointing | Chat history |
| Complexity | Minimal | Medium | High | Low but fragile |

Agent Bridge works best when:
- You're coordinating 2-5 agents across different systems
- Tasks are asynchronous (not real-time)
- You want a human-readable audit trail
- You don't want to maintain infrastructure

Agent Bridge is NOT for:
- High-frequency agent communication (>1 msg/sec)
- Streaming data between agents
- Systems that can't read files
- Production multi-agent orchestration at scale

## Full Protocol Specification

See [PROTOCOL.md](PROTOCOL.md) for the complete protocol specification including file formats, status codes, rejection handling, and versioning.

## Agent Templates

See [agents/](agents/) for ready-to-use agent role templates:
- `planner.md` -- Task planning and result review
- `executor.md` -- Task execution and result submission
- `chronicler.md` -- Documentation and history tracking

## License

MIT License. See [LICENSE](LICENSE).
