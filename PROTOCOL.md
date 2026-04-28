# 跨 Agent 协作协议 v0.9.2

> 文件即通信，任务即指令。没有闲聊，没有私信，没有广播栏。

## 设计哲学

所有通信都是任务。Agent 之间只通过"派活 / 交活"协作。

- 指向性通过任务的 `assignee` 字段表达
- 广播通过 `assignee: "all"` 表达
- 拒绝 / 阻塞通过 result 的 `status` 字段表达
- 协议升级通过直接改 `PROTOCOL.md` 实现，其他 agent 下次 `/check` 自然读到

## 目录结构

```
.agent-bridge/
├── PROTOCOL.md          ← 本文件
├── board/
│   ├── tasks.json       ← 活跃任务（规划者写）
│   └── archive.json     ← 归档任务索引（规划者维护）
├── results/
│   └── {id}.json        ← 任务结果（执行者写）
│   └── {id}-{agent}.json ← 广播任务的多人回应
├── materials/
│   └── {task_id}/       ← 任务素材（复杂输入放这里）
├── chronicles/          ← 史官写的历史（非通信）
│   ├── YYYY-MM-DD.md
│   ├── INDEX.md
│   └── legacy/          ← v0.9 之前的历史归档
└── archive/             ← 超阈值后归档的完整 result 内容
```

## 核心规则

1. **tasks.json 只由规划者写**，其他人只读
2. **results/{id}.json 由执行者写**，每任务一文件，天然隔离
3. **结果文件不修改**——只新增或归档，不覆盖
4. **没有 chat.md、没有 inbox、没有 agents.json**——通信全走任务

## 任务格式

### tasks.json（规划者写）

```json
{
  "last_updated": "2026-04-17T23:40:00",
  "tasks": [
    {
      "id": "T026",
      "title": "一句话标题",
      "description": "需求 + 验收标准",
      "assignee": "executor",
      "created_by": "planner",
      "created_at": "2026-04-17T23:40:00",
      "tags": []
    }
  ]
}
```

**assignee 可选值**：
- 具体 agent 名（`"executor"` / `"executor-1"` / `"chronicler"` / `"planner"`）
- `"all"` — 广播，所有 agent 都要处理
- 多身份 executor 由人类指派（"你是 executor-2"），协议不自动分配

**没有 status 字段**。任务状态靠 result 文件存在与否隐式表达。

### results/{id}.json（执行者写）

```json
{
  "task_id": "T026",
  "status": "completed",
  "summary": "做了什么",
  "deliverables": ["文件路径"],
  "issues": [],
  "completed_by": "executor",
  "completed_at": "2026-04-17T23:50:00"
}
```

**status 取值**：

| status | 含义 | 必须带的字段 |
|---|---|---|
| `completed` | 完成 | `summary` / `deliverables` |
| `rejected` | 拒绝（任务不合理/需要澄清） | `reason` + `suggestion` |
| `blocked` | 阻塞（做到一半缺条件） | `reason` |
| `failed` | 执行失败（出错） | `error` |

**拒绝或阻塞时不需要另外发消息。** 规划者 `/check` 时扫 results/，看到 rejected/blocked 的 result 会读 reason 处理。

### 广播任务的结果

任务 `assignee: "all"` 时，每个 agent 各自写 `results/{id}-{agent}.json`：
```
results/T027-planner.json
results/T027-executor.json
results/T027-chronicler.json
```

规划者通过扫 results/ 目录判断所有人是否都回应了。

## 任务状态判断

| results/{id}.json | 含义 |
|---|---|
| 不存在 | pending（还没做） |
| status=completed | 完成 |
| status=rejected | 拒绝，看 reason/suggestion 决定改任务或撤销 |
| status=blocked | 阻塞，看 reason 决定补素材或发澄清任务 |
| status=failed | 执行失败，看 error |

## 归档机制

**归档是规划者的活**（规划者有全局视野）。

**触发**：规划者 `/check` 时发现 `tasks.json` + `results/` 合计活跃条目 > 15，顺手归档。

**归档动作**：
1. 挑完成/拒绝/失败/阻塞已处理的条目
2. 完整 result body 追加到 `archive/results-YYYY-MM.json`
3. 标题索引追加到 `board/archive.json`
4. 从 `tasks.json` 和 `results/` 中删除这些条目

**board/archive.json 格式**：
```json
{
  "last_updated": "...",
  "archived_tasks": [
    {"id": "T001", "title": "...", "result_status": "completed", "archived_at": "..."}
  ]
}
```

## 新 Agent 加入

**不需要自注册**。流程：
1. 读 `PROTOCOL.md` 了解规则
2. 人类指派身份（"你是 executor-2"）
3. 扫 `tasks.json` 过滤 `assignee == 你的名字` 或 `assignee == "all"`
4. 没活干就闲着，规划者分派任务时再行动

**多个 executor 并行**：规划者颁任务时 `assignee` 明确指定 `executor-1` / `executor-2` 等。身份由人类口头指派。

## 自治原则

| 类型 | 做法 |
|---|---|
| 顺手能做的（git commit、改 typo） | 直接做，不颁任务 |
| 小决策（命名、文件位置） | 自己定 |
| 长耗时任务（重构、调研） | 颁任务走正式流程 |
| 需要外部信息 | 交给有搜索能力的 agent |

判断标准：**token 消耗和上下文占用**。几行能搞定的不走任务流程。

## 任务描述纪律

给执行者写任务时，**只写需求和验收标准，不教如何实现**。

1. **只写"要什么"+"如何验收"，不写"如何做"**
2. **每个专业词补白话解释**，别假设执行者懂术语
3. **关键约束举具体例子**，不用形容词
4. **写完自审**：哪些句子是在"教他怎么干"？全删
5. **少即是多**：150 字说清的不写 1500 字

## 三层分离

6. **实现任务验收只验"跑通"**，不验质量指标
7. **复杂素材放 `materials/{task_id}/`**，不塞任务描述
8. **测试任务和实现任务分开颁**：实现任务验"跑通"，测试任务验"达标"

## 工作流

### 规划者
1. 扫 `results/` 看完成情况
2. 颁布新任务到 `tasks.json`
3. 处理 rejected/blocked 的 result
4. 定期归档（超 15 条顺手做）

### 执行者
1. 扫 `tasks.json` 过滤 `assignee == 我 || "all"`
2. 跳过已有 `results/{id}.json` 的任务
3. **审题**（执行前必须过）：
   - 理解任务要改什么文件/模块
   - 快速确认这些文件/模块的当前状态（列是否已加、函数是否已有等）
   - 如果任务要加的东西已存在、描述有歧义、或与项目方向矛盾 → 直接 `status: "rejected"` + reason
   - 通过 → 进入对齐
4. **对齐汇报**（审题通过后、执行前必须做）：
   - 直接向**用户**口头汇报，包含：
     - 任务理解：一句话说明你认为要做什么
     - 影响范围：会动哪些文件/模块，对现有功能的潜在影响
     - 具体问题：发现的歧义、风险、与现有代码的冲突
   - 等用户确认后才动手执行
   - 用户说"做"→ 继续；用户调整方向→按新方向来；用户说"不做"→ reject
5. 执行任务
6. 写 `results/{id}.json`（或广播时 `results/{id}-{agent}.json`）
7. 有疑问 → `status: "rejected"` + reason/suggestion
8. 卡住 → `status: "blocked"` + reason

### 史官
1. 扫 `results/` 了解产出
2. 写 `chronicles/YYYY-MM-DD.md` 记录历史
3. 不参与通信主流程

## 人工命令

| 命令 | 含义 |
|---|---|
| `/check` | 按身份执行对应检查流程 |
| `/report` | 输出当前任务状态总表 |

## 协议版本

- v0.1-v0.6 — 见 `chronicles/legacy/`
- v0.7 — housekeeping（归档/滚动窗口/收件箱裁剪）
- v0.8 — results 合并到单文件（后回退）
- v0.9 — 极简化：取消 chat.md / inbox / agents.json，所有通信走任务
- v0.9.1 — 执行者加审题步骤：执行前必须确认任务前提是否存在，已存在则 reject
- v0.9.2 — 执行者加对齐汇报：审题通过后向规划者口头汇报理解和影响，确认后再执行
