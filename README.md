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

**With this:** One shared `.agent-bridge/` directory. Planner writes tasks to a JSON file. Executor picks them up, does the work, writes results to individual JSON files. Everyone reads the group chat file to stay in sync. No server, no API, no nothing.

```
Claude Code (planner)          Cursor (executor)          Python script (tester)
       |                              |                            |
       +--- writes tasks.json --------+                            |
       |                              +--- writes results/T001.json|
       |                              |                            +--- reads results, writes T002.json
       |<--- reads results/ ----------+<---------------------------+
       |
     .agent-bridge/    <-- the only thing they share
```

## How it works (30 seconds)

```
.agent-bridge/
├── board/tasks.json       ← Planner writes tasks here
├── results/T001.json      ← Executor writes results (one file per task)
├── inbox/planner.md       ← Direct messages (append-only)
├── inbox/executor.md
├── chat.md                ← Group chat (append-only)
└── config/agents.json     ← Who's who
```

**Planner** writes a task → **Executor** does the work → writes a result file → **Planner** reads it. That's the whole loop.

No status field in tasks.json. A task is pending if its result file doesn't exist. Done if it does.

## Quick Start

```bash
# 1. Create the bridge
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config,materials}

# 2. Copy the example setup
# (or see example/ in this repo for a ready-to-use template)
```

Then tell your agents where `.agent-bridge/` is. That's it.

For a complete working example, see the [`example/`](example/) directory.

## For AI Agents: Installation Guide

This protocol is designed for AI agents to self-install. Here's how to set it up in different systems:

### Claude Code

Create agent skills (commands) so Claude can act as planner or executor:

```bash
# Create the commands directory if it doesn't exist
mkdir -p .claude/commands

# Copy the agent template as a slash command
cp agents/planner.md .claude/commands/planner.md
cp agents/executor.md .claude/commands/executor.md
```

Now type `/planner` or `/executor` in Claude Code to activate the role.

### Any AI with file access

Give your AI agent these instructions (paste into its system prompt or context):

1. Read `.agent-bridge/PROTOCOL.md` for the full protocol
2. Read `.agent-bridge/config/agents.json` to see who's registered
3. Read your assigned role template from `agents/{role}.md`
4. Start working: read tasks, execute, write results

That's all the setup needed. No installation, no configuration wizard.

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
├── PROTOCOL.md            ← Full protocol specification (v0.6)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← Planner role template
│   ├── executor.md        ← Executor role template
│   └── chronicler.md      ← Chronicler role template (optional)
└── example/
    ├── board/tasks.json   ← Sample tasks
    ├── chat.md            ← Sample group chat
    ├── config/agents.json ← Sample agent registration
    ├── inbox/             ← Sample inboxes
    ├── results/           ← Sample result file
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

**用这个之后：** 一个共享的 `.agent-bridge/` 目录。规划者把任务写到 JSON 文件里。执行者领任务、干活、把结果写到独立 JSON 文件里。大家通过群聊文件保持同步。没有服务器、没有 API、什么都没有。

```
Claude Code (规划者)          Cursor (执行者)          Python 脚本 (测试员)
       |                              |                            |
       +--- 写 tasks.json -----------+                            |
       |                              +--- 写 results/T001.json   |
       |                              |                            +--- 读结果，写 T002.json
       |<--- 读 results/ -------------+<---------------------------+
       |
     .agent-bridge/    ← 它们唯一共享的东西
```

## 30 秒看懂

```
.agent-bridge/
├── board/tasks.json       ← 规划者在这里写任务
├── results/T001.json      ← 执行者写结果（每个任务一个文件）
├── inbox/planner.md       ← 私信（只追加）
├── inbox/executor.md
├── chat.md                ← 群聊（只追加）
└── config/agents.json     ← 谁是谁
```

**规划者** 写任务 → **执行者** 干活 → 写结果文件 → **规划者** 读取。这就是整个循环。

tasks.json 里没有 status 字段。结果文件不存在 = 进行中，存在 = 已完成。就这么简单。

## 快速开始

```bash
# 1. 创建桥接目录
mkdir -p .agent-bridge/{board,results,inbox,chronicles,config,materials}

# 2. 复制示例配置
# （或者直接看本仓库的 example/ 目录）
```

然后告诉你的 Agent `.agent-bridge/` 在哪。完了。

完整示例见 [`example/`](example/) 目录。

## 给 AI Agent 的安装指南

这个协议设计给 AI Agent 自行安装。不同系统的设置方式：

### Claude Code

创建 slash command，让 Claude 扮演规划者或执行者：

```bash
mkdir -p .claude/commands
cp agents/planner.md .claude/commands/planner.md
cp agents/executor.md .claude/commands/executor.md
```

然后在 Claude Code 里输入 `/planner` 或 `/executor` 即可激活角色。

### 任何有文件访问能力的 AI

把以下指令粘贴到 AI 的系统提示词或上下文里：

1. 读 `.agent-bridge/PROTOCOL.md` 了解完整协议
2. 读 `.agent-bridge/config/agents.json` 看谁已注册
3. 读对应角色模板 `agents/{角色}.md`
4. 开始工作：读任务、执行、写结果

这就是全部设置。没有安装向导，没有配置界面。

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
├── PROTOCOL.md            ← 完整协议规范 (v0.6)
├── LICENSE                ← MIT
├── agents/
│   ├── planner.md         ← 规划者角色模板
│   ├── executor.md        ← 执行者角色模板
│   └── chronicler.md      ← 史官角色模板（可选）
└── example/
    ├── board/tasks.json   ← 示例任务
    ├── chat.md            ← 示例群聊
    ├── config/agents.json ← 示例注册
    ├── inbox/             ← 示例收件箱
    ├── results/           ← 示例结果
    └── chronicles/        ← 示例编年史
```

## 许可证

MIT
