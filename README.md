# Cross Agent Bridge

**[English](#english) | [中文](#中文)**

> 让 10 个不同框架的 AI Agent 一起干活，只需要一个文件夹。

---

<a id="english"></a>

## NO code. NO MCP. NO complexity.

**NO code.** Pure text protocol. No `pip install`, no `npm install`, no SDK, no runtime. Copy the folder structure and go.

**NO MCP.** No servers to start. No brokers, no bridges, no gateways, no middleware. The directory IS the infrastructure.

**NO complex protocols.** No shared memory, no message queues, no event buses, no orchestration layers. Read a file. Write a file. Done.

## Just.

Just a folder. Just files. Just read and write.

Claude Code, Cursor, Gemini CLI, GPT, Ollama, custom Python scripts, Bash one-liners -- throw them all in. Different machines, different OSes, different networks. **If it can read and write files, it collaborates.**

## The pain it solves

You're using Claude Code for planning, Cursor for coding, and a custom Python script for testing. How do they coordinate?

**Right now:** You copy-paste context between windows. You forget what Agent A decided. Agent B does work that conflicts with Agent C. You're the glue, and it sucks.

**With this:** One shared `.agent-bridge/` directory. Planner writes tasks to a JSON file. Executor picks them up, does the work, writes results to per-task files. Human reviews and decides what's next. No server, no API, no nothing.

```
Human ─── tells planner what to do
  │
  ▼
Planner ─── writes tasks.json ──► Executor picks up task
  ▲                                    │
  │                                    ▼
  └── reads results/{id}.json ◄── Executor writes result
  │
  └── Human reviews, approves, moves on
```

## How it works (30 seconds)

```
.agent-bridge/
├── board/
│   ├── tasks.json       ← Active tasks (planner writes)
│   └── archive.json     ← Archived task index
├── results/
│   └── {id}.json        ← One result file per task (executor writes)
├── materials/
│   └── {task_id}/       ← Complex inputs (prompts, data, rubrics)
├── chronicles/          ← Daily records (chronicler writes)
└── archive/             ← Old result bodies
```

**Planner** writes a task → **Executor** picks it up, does the work → writes `results/{id}.json` → **Human** reviews → **Planner** reads result and issues next task.

No status field in tasks.json. A task is pending if `results/{id}.json` doesn't exist yet. Done if it does.

## Human in the loop

This is not an autonomous system. It's a **collaboration tool** designed for creative, pioneering work where human judgment is essential.

- Humans trigger agent actions (via `/check` or equivalent)
- Humans review and approve executor's understanding before execution
- Humans verify results and decide next steps
- Humans resolve blockers and clarify ambiguities

For repetitive, well-defined tasks, one agent is enough. Multi-agent collaboration shines when the work is exploratory and decisions need human oversight.

## Quick Start

### You (the human) do 3 things:

**1. Create the bridge**
```bash
mkdir -p .agent-bridge/{board,results,materials,chronicles,archive}
```

**2. Tell each agent who's who**

Copy role templates into each agent's system:

| Framework | How |
|-----------|-----|
| Claude Code | Copy `agents/planner.md` to `.claude/commands/planner.md`, etc. |
| Cursor / Windsurf | Put template content in `.cursorrules` or project rules |
| Gemini CLI | Add to `GEMINI.md` or system prompt |
| Custom scripts | Read the template file, include in system prompt |
| Any AI with file access | Paste the template and say "follow this" |

**3. Trigger agents to check**

Type `/check` (or your trigger command) in any agent. Agents read the bridge, find their tasks, and work. You review and approve.

### Agent setup

No registration needed. When an agent starts:
1. Read `PROTOCOL.md` for the rules
2. Scan `tasks.json` for tasks with matching `assignee`
3. No matching tasks? Wait. The human or planner will assign work.

No wizard, no configuration, no onboarding flow. Files are the documentation.

### Adding a new agent

1. Tell the agent its identity ("you are executor-2")
2. Done. The planner assigns tasks to that identity by name.

## Real-world scenarios

**Scenario 1: Multi-tool coding workflow**
Claude Code (planner) breaks down a feature → Cursor (executor) writes code → human reviews → Gemini CLI (tester) runs tests → results flow back → new tasks for fixes.

**Scenario 2: Mixed human + AI team**
Human plans with Claude Code → Aider executes → teammate's Cursor reviews → all coordinated through the same `.agent-bridge/` directory.

**Scenario 3: Cross-machine collaboration**
Agents on different machines share `.agent-bridge/` via Dropbox, Syncthing, or a git repo. Each machine runs its own agents. File changes propagate automatically.

## What it's good at / What it's not

**Good at:**
- Coordinating 2-10 agents across different frameworks
- Asynchronous task workflows (not real-time)
- Human-readable audit trail (it's all text files)
- Zero-setup collaboration
- Human-in-the-loop oversight for creative work
- Enforcing discipline through protocol (task description rules, three-layer separation)

**Not good at:**
- Real-time communication (>1 message/second)
- Streaming data between agents
- Systems that cannot read files
- Large-scale orchestration (100+ agents)
- Fully autonomous pipelines (human review is mandatory)

## The secret sauce

### All communication is tasks

No group chat. No private messages. No broadcast channel. Everything is a task:

- **Directionality** → task's `assignee` field
- **Broadcast** → `assignee: "all"`, each agent writes `results/{id}-{agent}.json`
- **Rejection/blocking** → result's `status` + `reason`
- **Protocol upgrade** → edit `PROTOCOL.md`, agents read on next check

### Task Description Discipline (8 rules)

1. Only write requirements and acceptance criteria, not implementation steps
2. Explain or avoid jargon
3. Show concrete examples, not adjectives
4. Delete every "how to" sentence after writing
5. Less is more (150 words > 1500 words)
6. Implementation tasks verify "it works", not "it's good"
7. Complex materials go in folders, not task descriptions
8. Test tasks and implementation tasks are separate

### Executor self-check

Before executing, the executor must verify:
- What files/modules does this task touch?
- Do those files/modules actually exist?
- Any conflicts with existing code?
- If prerequisites are missing → reject immediately

### Alignment report

After self-check passes, the executor reports to the **human**:
- Task understanding in one sentence
- What files/modules will be affected
- Any risks or ambiguities found
- **Wait for human confirmation before executing**

## Comparison

| | Cross Agent Bridge | MCP | LangGraph | Manual |
|---|---|---|---|---|
| Setup | `mkdir` | Install + run server | Install + framework | None |
| Dependencies | None | Python/JS server | Python + LangGraph | None |
| Cross-framework | Any | Same process | Same framework | Copy-paste |
| Human in loop | Built-in | Optional | Custom | Mandatory |
| Audit trail | Built-in (files) | Custom | Checkpointing | Chat history |

## Directory contents

```
cross-agent-bridge/
├── README.md              ← You are here
├── PROTOCOL.md            ← Full protocol specification (v0.9.2)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← Planner role template
│   ├── executor.md        ← Executor role template
│   └── chronicler.md      ← Chronicler role template (optional)
└── example/
    ├── board/
    │   ├── tasks.json     ← Sample active tasks
    │   └── archive.json   ← Sample archive
    ├── results/           ← Per-task result files
    │   ├── T001.json
    │   ├── T002.json
    │   └── T003.json
    └── chronicles/        ← Sample daily records
```

## Protocol versions

- v0.1-v0.6 — Early iterations (chat, inbox, agents.json)
- v0.7 — Housekeeping (archiving, rolling window)
- v0.8 — Results consolidated to single file (reverted)
- v0.9 — Radical simplification: removed chat.md, inbox, agents.json. All communication is tasks.
- v0.9.1 — Added executor self-check: verify prerequisites before executing
- v0.9.2 — Added alignment report: executor reports to human before executing

## License

MIT

---

<a id="中文"></a>

## NO 代码。NO MCP。NO 复杂协议。

**NO 代码。** 纯文本协议。没有 `pip install`，没有 `npm install`，没有 SDK，没有运行时。复制目录结构就能跑。

**NO MCP。** 不用起服务。没有 broker，没有 gateway，没有中间件。目录就是基础设施。

**NO 复杂协议。** 没有共享内存，没有消息队列，没有事件总线，没有编排层。读文件。写文件。完了。

## 就这样。

就一个文件夹。就几个文件。就读和写。

Claude Code、Cursor、Gemini CLI、GPT、Ollama、自定义 Python 脚本、Bash 一行命令——全扔进来。不同机器、不同系统、不同网络。**能读写文件就能协作。**

## 解决什么痛点

你在用 Claude Code 做规划、Cursor 写代码、自定义 Python 脚本做测试。它们之间怎么协调？

**现在的做法：** 你在不同窗口之间复制粘贴上下文。你忘了 Agent A 之前决定的东西。Agent B 做的活和 Agent C 冲突。你就是那个"胶水"，很累。

**用这个之后：** 一个共享的 `.agent-bridge/` 目录。规划者把任务写到 JSON 文件里。执行者领任务、干活、把结果写到独立的结果文件里。人类审查并决定下一步。没有服务器、没有 API、什么都没有。

```
人类 ─── 告诉规划者要做什么
  │
  ▼
规划者 ─── 写 tasks.json ──► 执行者领取任务
  ▲                                    │
  │                                    ▼
  └── 读 results/{id}.json ◄── 执行者写结果
  │
  └── 人类审查、确认、推进
```

## 30 秒看懂

```
.agent-bridge/
├── board/
│   ├── tasks.json       ← 活跃任务（规划者写）
│   └── archive.json     ← 归档任务索引
├── results/
│   └── {id}.json        ← 每任务独立结果文件（执行者写）
├── materials/
│   └── {task_id}/       ← 复杂素材（提示词、数据、评分标准）
├── chronicles/          ← 每日记录（史官写）
└── archive/             ← 旧结果详情
```

**规划者** 写任务 → **执行者** 领任务、干活 → 写 `results/{id}.json` → **人类** 审查 → **规划者** 读结果、颁布下一步任务。

tasks.json 里没有 status 字段。`results/{id}.json` 不存在 = 进行中，存在 = 已处理。就这么简单。

## 人在环中

这不是自动系统。这是一个为**开创性工作**设计的**协作工具**，人在环中是核心设计。

- 人类触发 Agent 行动（通过 `/check` 或等价命令）
- 人类审查并确认执行者的理解后才允许执行
- 人类验证结果并决定下一步
- 人类解决阻塞、澄清歧义

重复性的、明确的工作，一个 Agent 就够了。多 Agent 协作的价值在于探索性工作需要人的判断。

## 快速开始

### 你（人类）只做 3 件事：

**1. 建桥接目录**
```bash
mkdir -p .agent-bridge/{board,results,materials,chronicles,archive}
```

**2. 告诉每个 Agent 它是谁**

把角色模板复制到每个 Agent 的系统里：

| 框架 | 方法 |
|------|------|
| Claude Code | 把 `agents/planner.md` 复制到 `.claude/commands/planner.md` |
| Cursor / Windsurf | 放到 `.cursorrules` 或项目规则文件里 |
| Gemini CLI | 加到 `GEMINI.md` 或系统提示词 |
| 自定义脚本 | 读模板文件，塞进系统提示词 |
| 任何能读文件的 AI | 把模板粘贴到上下文里，说"照着做" |

**3. 叫它们检查**

在任何 Agent 里输入 `/check`。Agent 读桥接目录、找到自己的任务、开始工作。你审查和确认。

### Agent 设置

不需要注册。当一个 Agent 启动时：
1. 读 `PROTOCOL.md` 了解规则
2. 扫 `tasks.json` 找 `assignee` 匹配的任务
3. 没任务？等着。人类或规划者会分配。

没有安装向导，没有配置界面。文件就是文档。

### 添加新 Agent

1. 告诉 Agent 它的身份（"你是 executor-2"）
2. 完成。规划者颁任务时会指定这个身份。

## 真实使用场景

**场景 1：多工具编码工作流**
Claude Code（规划者）拆分功能 → Cursor（执行者）写代码 → 人类审查 → Gemini CLI（测试员）跑测试 → 结果回到规划者 → 新的修复任务发出。

**场景 2：人 + AI 混合团队**
人用 Claude Code 规划 → Aider 执行编码任务 → 队友的 Cursor 审查结果 → 全部通过同一个 `.agent-bridge/` 目录协调。

**场景 3：跨机器协作**
不同机器上的 Agent 通过 Dropbox、Syncthing 或 git 仓库共享 `.agent-bridge/`。每台机器跑自己的 Agent。文件变更自动传播。

## 擅长什么 / 不擅长什么

**擅长：**
- 跨框架协调 2-10 个 Agent
- 异步任务流（非实时）
- 人类可读的审计追踪（全是文本文件）
- 零配置协作
- 人在环中的开创性工作
- 通过协议强制纪律（任务描述规则、三层分离）

**不擅长：**
- 实时通信（>1 条消息/秒）
- Agent 之间的流式数据传输
- 不能读写文件的系统
- 大规模编排（100+ Agent）
- 全自动流水线（人类审查是必须的）

## 秘密武器

### 所有通信都是任务

没有群聊。没有私信。没有广播栏。一切都是任务：

- **指向性** → 任务的 `assignee` 字段
- **广播** → `assignee: "all"`，每个 Agent 写 `results/{id}-{agent}.json`
- **拒绝/阻塞** → result 的 `status` + `reason`
- **协议升级** → 改 `PROTOCOL.md`，Agent 下次检查时自读

### 任务描述纪律（8 条规则）

1. 只写需求和验收标准，不写实现步骤
2. 解释或避免专业术语
3. 给具体例子，不用形容词
4. 写完删掉所有"怎么做"的句子
5. 少即是多（150 字 > 1500 字）
6. 实现任务验证"能跑"，不验证"好不好"
7. 复杂素材放文件夹，不塞任务描述
8. 测试任务和实现任务分开颁布

### 执行者自检（审题）

执行前必须确认：
- 这个任务要动哪些文件/模块？
- 这些文件/模块真的存在吗？
- 和现有代码有冲突吗？
- 前提条件缺失 → 直接拒绝

### 对齐汇报

自检通过后，执行者向**人类**汇报：
- 一句话说清任务理解
- 会影响哪些文件/模块
- 发现的风险和歧义
- **等人类确认后才执行**

## 对比

| | Cross Agent Bridge | MCP | LangGraph | 手动 |
|---|---|---|---|---|
| 安装 | `mkdir` | 装依赖 + 起服务 | 装框架 + 配置 | 无 |
| 依赖 | 无 | Python/JS 服务 | Python + LangGraph | 无 |
| 跨框架 | 任意 | 同进程 | 同框架 | 复制粘贴 |
| 人在环中 | 内置 | 可选 | 自定义 | 必须 |
| 审计追踪 | 内置（文件即日志） | 自定义 | Checkpoint | 聊天记录 |

## 目录内容

```
cross-agent-bridge/
├── README.md              ← 你在这里
├── PROTOCOL.md            ← 完整协议规范 (v0.9.2)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← 规划者角色模板
│   ├── executor.md        ← 执行者角色模板
│   └── chronicler.md      ← 史官角色模板（可选）
└── example/
    ├── board/
    │   ├── tasks.json     ← 示例活跃任务
    │   └── archive.json   ← 示例归档
    ├── results/           ← 每任务独立结果文件
    │   ├── T001.json
    │   ├── T002.json
    │   └── T003.json
    └── chronicles/        ← 示例编年史
```

## 协议版本

- v0.1-v0.6 — 早期迭代（含 chat、inbox、agents.json）
- v0.7 — housekeeping（归档、滚动窗口）
- v0.8 — 结果合并到单文件（后回退）
- v0.9 — 极简化：取消 chat.md / inbox / agents.json，所有通信走任务
- v0.9.1 — 执行者加审题步骤：执行前确认前提
- v0.9.2 — 执行者加对齐汇报：执行前向人类汇报，确认后再执行

## 许可证

MIT
