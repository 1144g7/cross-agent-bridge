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

**With this:** One shared `.agent-bridge/` directory. Planner writes tasks to a JSON file. Executor picks them up, does the work, appends results to `results.json`. Everyone reads the group chat file to stay in sync. No server, no API, no nothing.

```
Claude Code (planner)          Cursor (executor)          Python script (tester)
       |                              |                            |
       +--- writes tasks.json --------+                            |
       |                              +--- appends to results.json |
       |                              |                            +--- reads results.json, appends own
       |<--- reads results.json ------+<---------------------------+
       |
     .agent-bridge/    <-- the only thing they share
```

## How it works (30 seconds)

```
.agent-bridge/
├── board/tasks.json       ← Active tasks only (planner writes)
├── board/archive.json     ← Completed tasks (executor archives)
├── results.json           ← All active results in one file
├── inbox/planner.md       ← Direct messages (append-only)
├── inbox/executor.md
├── chat.md                ← Group chat (rolling window, last 30)
└── config/agents.json     ← Who's who
```

**Planner** writes a task → **Executor** does the work → appends to `results.json` → **Planner** reads it. That's the whole loop.

No status field in tasks.json. A task is pending if it's not in `results.json` yet. Done if it is.

## Quick Start

### You (the human) do 3 things:

**1. Create the bridge**
```bash
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config,materials}
```

**2. Tell each agent who's who**

Copy role templates into each agent's system. How depends on the framework:

| Framework | How |
|-----------|-----|
| Claude Code | Copy `agents/planner.md` to `.claude/commands/planner.md`, etc. |
| Cursor / Windsurf | Put the template content in `.cursorrules` or your project rules file |
| Gemini CLI | Add to your `GEMINI.md` or system prompt |
| Custom scripts | Just read the template file and include it in the system prompt |
| Any AI with file access | Paste the template into its context and say "follow this" |

Each agent template (`agents/*.md`) includes everything the AI needs to know.

**3. Tell them to check**

Type `/check` (or your trigger command) in any agent to start. From there, agents self-coordinate through files.

That's it. You don't write tasks, you don't review results, you don't run anything. The agents do all of that through the bridge.

### Agent self-setup

Agents read the protocol and templates themselves. When an agent starts:
1. It reads `.agent-bridge/PROTOCOL.md` for the rules
2. It reads `.agent-bridge/config/agents.json` to see who's registered
3. It reads its role template (`agents/{role}.md`) for instructions
4. It starts working

No wizard, no configuration, no onboarding flow. Files are the documentation.

### Adding a new agent type

1. Write a role template in `agents/your-agent.md`
2. Register it in `config/agents.json`
3. Done. Other agents will discover it on their next check.

## Real-world scenarios

**Scenario 1: Multi-tool coding workflow**
Claude Code (planner) breaks down a feature → Cursor (executor) writes code → Gemini CLI (tester) runs tests → results flow back to planner → new tasks issued for fixes.

**Scenario 2: Mixed human + AI team**
Human uses Claude Code to plan → Aider executes coding tasks → a teammate's Cursor instance reviews results → all coordinated through the same `.agent-bridge/` directory.

**Scenario 3: Cross-machine collaboration**
Agents on different machines share `.agent-bridge/` via Dropbox, Syncthing, or a git repo. Each machine runs its own agents. File changes propagate automatically.

## What it's good at / What it's not

**Good at:**
- Coordinating 2-10 agents across different frameworks
- Asynchronous task workflows (not real-time)
- Human-readable audit trail (it's all text files)
- Zero-setup collaboration
- Enforcing discipline through protocol (task description rules, three-layer separation)

**Not good at:**
- Real-time communication (>1 message/second)
- Streaming data between agents
- Systems that cannot read files
- Large-scale orchestration (100+ agents)

## The secret sauce: Task Description Discipline

This protocol includes 8 battle-tested rules for writing tasks that actually work. Learned the hard way from real multi-agent projects.

**Core insight:** Write what you want, not how to do it.

1. Only write requirements and acceptance criteria
2. Explain or avoid jargon
3. Show concrete examples, not adjectives
4. Delete every "how to" sentence after writing
5. Less is more (150 words > 1500 words)

Plus the **Three-Layer Separation**:
6. Implementation tasks verify "it works", not "it's good"
7. Complex materials go in folders, not task descriptions
8. Test tasks and implementation tasks are separate

Full details in [PROTOCOL.md](PROTOCOL.md).

## Comparison

| | Cross Agent Bridge | MCP | LangGraph | Manual |
|---|---|---|---|---|
| Setup | `mkdir` | Install + run server | Install + framework | None |
| Dependencies | None | Python/JS server | Python + LangGraph | None |
| Cross-framework | Any | Same process | Same framework | Copy-paste |
| AI tool A + tool B | Works | Need shared server | Need shared code | Pain |
| Audit trail | Built-in (files) | Custom | Checkpointing | Chat history |

## Directory contents

```
cross-agent-bridge/
├── README.md              ← You are here
├── PROTOCOL.md            ← Full protocol specification (v0.8)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← Planner role template
│   ├── executor.md        ← Executor role template
│   └── chronicler.md      ← Chronicler role template (optional)
└── example/
    ├── board/tasks.json   ← Sample active tasks
    ├── board/archive.json ← Sample archive
    ├── chat.md            ← Sample group chat
    ├── config/agents.json ← Sample agent registration
    ├── inbox/             ← Sample inboxes
    ├── results.json       ← Sample results
    └── chronicles/        ← Sample daily records
```

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

**用这个之后：** 一个共享的 `.agent-bridge/` 目录。规划者把任务写到 JSON 文件里。执行者领任务、干活、把结果追加到 `results.json`。大家通过群聊文件保持同步。没有服务器、没有 API、什么都没有。

```
Claude Code (规划者)          Cursor (执行者)          Python 脚本 (测试员)
       |                              |                            |
       +--- 写 tasks.json -----------+                            |
       |                              +--- 追加到 results.json    |
       |                              |                            +--- 读 results.json，追加自己的
       |<--- 读 results.json ---------+<---------------------------+
       |
     .agent-bridge/    ← 它们唯一共享的东西
```

## 30 秒看懂

```
.agent-bridge/
├── board/tasks.json       ← 只有活跃任务（规划者写）
├── board/archive.json     ← 已完成任务（执行者归档）
├── results.json           ← 所有活跃结果在一个文件里
├── inbox/planner.md       ← 私信（只追加）
├── inbox/executor.md
├── chat.md                ← 群聊（滚动窗口，最近30条）
└── config/agents.json     ← 谁是谁
```

**规划者** 写任务 → **执行者** 干活 → 追加到 `results.json` → **规划者** 读取。这就是整个循环。

tasks.json 里没有 status 字段。results.json 里没有这个任务 = 进行中，有了 = 已完成。就这么简单。

## 快速开始

### 你（人类）只做 3 件事：

**1. 建桥接目录**
```bash
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config,materials}
```

**2. 告诉每个 Agent 它是谁**

把角色模板复制到每个 Agent 的系统里。方法取决于框架：

| 框架 | 方法 |
|------|------|
| Claude Code | 把 `agents/planner.md` 复制到 `.claude/commands/planner.md`，以此类推 |
| Cursor / Windsurf | 放到 `.cursorrules` 或项目规则文件里 |
| Gemini CLI | 加到 `GEMINI.md` 或系统提示词 |
| 自定义脚本 | 读模板文件，塞进系统提示词 |
| 任何能读文件的 AI | 把模板粘贴到它的上下文里，说"照着做" |

每个角色模板（`agents/*.md`）已经包含了 AI 需要知道的一切。

**3. 叫它们检查**

在任何 Agent 里输入 `/check`（或你的触发命令）。之后 Agent 们通过文件自行协调。

就这样。你不用写任务、不用审结果、不用跑任何东西。Agent 们通过桥接目录自己搞定。

### Agent 自行设置

Agent 自己读协议和模板。当一个 Agent 启动时：
1. 读 `.agent-bridge/PROTOCOL.md` 了解规则
2. 读 `.agent-bridge/config/agents.json` 看谁已注册
3. 读角色模板（`agents/{角色}.md`）了解自己的职责
4. 开始工作

没有安装向导，没有配置界面。文件就是文档。

### 添加新的 Agent 类型

1. 在 `agents/` 里写一个角色模板
2. 在 `config/agents.json` 里注册
3. 完成。其他 Agent 下次检查时自动发现。

## 真实使用场景

**场景 1：多工具编码工作流**
Claude Code（规划者）拆分功能 → Cursor（执行者）写代码 → Gemini CLI（测试员）跑测试 → 结果回到规划者 → 新的修复任务发出。

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
- 通过协议强制纪律（任务描述规则、三层分离）

**不擅长：**
- 实时通信（>1 条消息/秒）
- Agent 之间的流式数据传输
- 不能读写文件的系统
- 大规模编排（100+ Agent）

## 秘密武器：任务描述纪律

本协议包含 8 条实战验证的任务写作规则。来自真实多 Agent 项目的血泪教训。

**核心洞察：** 写你要什么，不要写怎么做。

1. 只写需求和验收标准
2. 解释或避免专业术语
3. 给具体例子，不用形容词
4. 写完删掉所有"怎么做"的句子
5. 少即是多（150 字 > 1500 字）

加上 **三层分离：**
6. 实现任务验证"能跑"，不验证"好不好"
7. 复杂素材放文件夹，不塞任务描述
8. 测试任务和实现任务分开颁布

详见 [PROTOCOL.md](PROTOCOL.md)。

## 对比

| | Cross Agent Bridge | MCP | LangGraph | 手动 |
|---|---|---|---|---|
| 安装 | `mkdir` | 装依赖 + 起服务 | 装框架 + 配置 | 无 |
| 依赖 | 无 | Python/JS 服务 | Python + LangGraph | 无 |
| 跨框架 | 任意 | 同进程 | 同框架 | 复制粘贴 |
| 工具 A + 工具 B | 直接协作 | 需要共享服务器 | 需要共享代码 | 痛苦 |
| 审计追踪 | 内置（文件即日志） | 自定义 | Checkpoint | 聊天记录 |

## 目录内容

```
cross-agent-bridge/
├── README.md              ← 你在这里
├── PROTOCOL.md            ← 完整协议规范 (v0.8)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← 规划者角色模板
│   ├── executor.md        ← 执行者角色模板
│   └── chronicler.md      ← 史官角色模板（可选）
└── example/
    ├── board/tasks.json   ← 示例活跃任务
    ├── board/archive.json ← 示例归档
    ├── chat.md            ← 示例群聊
    ├── config/agents.json ← 示例注册
    ├── inbox/             ← 示例收件箱
    ├── results.json       ← 示例结果
    └── chronicles/        ← 示例编年史
```

## 许可证

MIT
