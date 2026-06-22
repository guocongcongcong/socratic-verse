---
title: 多Agent协作与Swarms
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 2
level: 高级
status: 未开始
tags: [agent, advanced, multi-agent, swarm, collaboration]
confidence: high
teaching_method: V7-对子互教
est_hours: 1
contested: false
contradictions: []
---

# 多 Agent 协作与 Swarms

> 一个 Agent 是工具，多个 Agent 是组织。进阶篇学了怎么构建一个 Agent，框架对比里提了多 Agent 模式。这一篇深入：多个 Agent 怎么协作、怎么避免集体翻车。

---

## 为什么需要多 Agent

**不是因为"多就是好"**。多 Agent 解决的是单 Agent 的结构性问题：

| 单 Agent 的问题 | 多 Agent 的解法 |
|----------------|----------------|
| 一个 context window 装不下所有信息 | 每个 Agent 有自己的 context |
| 一个思考角度容易有盲区 | 多个角度互相纠正 |
| 长任务注意力漂移 | 每个 Agent 聚焦一个小任务 |
| 单点故障 | 一个 Agent 挂了，其他可以接管 |

**但多 Agent 引入新问题**：通信开销、协调成本、集体盲区、无限对话循环。

> 🤔 **思考题**：为什么不是"Agent 越多越好"？两个 Agent 来回对话 20 轮还没达成共识——问题在哪？

---

## 四种协作拓扑

### 拓扑一：Supervisor / Orchestrator-Worker

```
           Supervisor（中央调度者）
          /        |        \
    Worker A    Worker B    Worker C
```

**Supervisor** 收到任务 → 决定派谁 → Worker 执行 → Supervisor 检查结果 → 综合最终输出。

**适合**：任务结构清晰，中央调度者能把任务合理分解。

**实现方式**（2026 推荐）：
- ✅ Supervisor 用 tool call 路由到 Worker："调用 `transfer_to_code_reviewer` 工具"
- ❌ 用 LangGraph 内置的 `create_supervisor()`（上下文控制不如 tool-call 路由）

**注意**：Supervisor 是单点。Supervisor 的模型能力和 context window 成了全系统的瓶颈。

### 拓扑二：Peer-to-Peer / Swarm

```
    Agent A ←→ Agent B
       ↖       ↗
         Agent C
```

没有中央调度。Agent 之间通过 **Handoff**（任务移交）和共享消息通道协作。

**AutoGen v0.4 的 Actor 模型就是这种**：每个 Agent 有收件箱。消息来了处理，处理完了发消息给下一个。没有 Supervisor。

**适合**：任务边界模糊、需要动态协作。

**核心挑战**：
- 谁先说？→ Speaker Selection 机制
- 怎么知道什么时候结束？→ 收敛条件
- 消息顺序乱了怎么办？→ 消息队列 + 去重

### 拓扑三：Hierarchical（层级式）

```
        Strategic Supervisor
         /              \
    Tactical A      Tactical B
      /    \          /    \
   Op1   Op2       Op3   Op4
```

Supervisor 的 Supervisor。每一层有不同的抽象层次：
- **战略层**：决定"做什么"、"为什么"
- **战术层**：决定"怎么做"、"用什么工具"
- **操作层**：实际执行

**适合**：复杂项目，需要不同粒度的决策。

**核心挑战**：**分解漂移（Decomposition Drift）**——战略 Supervisor 把任务分给战术层，战术层执行时发现战略层的分解不合理，但反馈回不来。

### 拓扑四：Group Chat — 群聊模式

```
    [共享消息频道]
    Agent A: "我觉得应该先分析数据"
    Agent B: "同意，但我建议用 pandas 而不是纯 Python"
    Agent C: "我来写分析脚本"
    Agent A: "写完给我审查"
```

所有 Agent 在一个共享频道里。看到全部消息。Speaker Selection 机制决定"谁下一个说"。

**CrewAI 的 Sequential Process 就是最简单的群聊**：按固定顺序发言。

**核心挑战**：
- 上下文爆炸：群聊消息增长极快
- 旁听者困境：Agent C 被迫看 Agent A 和 B 的所有对话
- 搭便车：某个 Agent 一直不发言

> 🤔 **思考题**：你和两个同事在一个群里讨论问题。什么情况下"所有人看所有消息"是好的？什么时候应该"拆分频道"？

---

## 多 Agent 协作的关键机制

### Handoff：任务移交

```
Agent A: "这个问题需要数据库专家" → transfer_to_db_expert()
Agent B (db_expert): 接手，在自己的 context 里工作 → 返回结果
Agent A: 收到结果，继续
```

**关键设计**：Handoff 时要不要携带历史？
- ✅ **携带摘要**：省 token，给接手方足够的上下文
- ❌ **携带全部历史**：接手方的 context 被污染
- ✅ **Collapse（折叠）**：Handoff 工具调用自动把历史压缩成摘要

### Shared Memory / Blackboard

```
         [共享黑板]
    Agent A 写了：候选方案 = [方案1, 方案2]
    Agent B 读了黑板 → 补充：方案1 的缺点 → 写回黑板
    Agent C 读了黑板 → 投票方案2 → 写回黑板
```

黑板模式来自 1970 年代的 AI 研究。每个 Agent 读写同一块"黑板"，不需要知道其他 Agent 的存在。

**和"群聊"的区别**：黑板是结构化数据，群聊是非结构化对话。

### Debate（辩论）

```
Agent A（正方）: "这个重构应该做，因为..."
Agent B（反方）: "不应该做，因为..."
Agent A: "反驳：虽然...但..."
Judge（裁判）: 听完辩论 → 裁决
```

**Debate 的力量**：两个模型在诚实度上倾向于相互校准。一个说谎的模型更容易被另一个诚实的模型揭穿。

**最新研究**（Multi-Agent Debate, 2024-2025）：
- 奇数个 Agent 辩论比偶数个更容易收敛
- 3-5 个 Agent 是最佳范围
- 超过 7 个 Agent 后边际收益趋近于零

---

## 多 Agent 的失败模式

| 失败模式 | 症状 | 解法 |
|----------|------|------|
| **Groupthink（集体思维）** | 所有 Agent 观点趋同，没人质疑 | 故意安排"魔鬼代言人"角色 |
| **Monoculture（单一文化）** | 所有 Agent 用同一个模型，盲区相同 | 用不同模型、不同温度、不同 prompt 风格 |
| **Cascading（级联失败）** | Agent A 出错 → B 基于 A 的错误继续 → C 基于 B 的错误继续 | 每一步加验证门 |
| **Deadlock（死锁）** | A 等 B，B 等 A | 超时 + 默认决策 |
| **Infinite Chat（无限对话）** | 两个 Agent 一直说"你说得对但是…" | 最大轮次 + 收敛检测 |

---

## 选型指南

| 任务类型 | 推荐拓扑 | 推荐轮次上限 |
|----------|----------|-------------|
| 代码审查（多维度） | Parallel + Vote | 1（并行跑，取共识） |
| 研究报告 | Supervisor-Worker | 10-15 |
| 头脑风暴 | Group Chat | 5-8 |
| 复杂软件工程 | Hierarchical | 20-30 |
| 探索性问题 | Debate (3 Agents) | 8-12 |

---

## 延伸阅读

- [[Agent框架对比]] — 进阶篇：五大框架的多 Agent 实现
- [[自主Agent系统设计]] — 高级篇：多 Agent 系统的基础设施
- [[Anthropic工作流模式]] — 进阶篇：Orchestrator-Workers 模式

## 相关

- [[理解Agent]] — 入门篇：单 Agent 的基础
- [[驾驭工程]] — 入门篇：多层架构哲学
