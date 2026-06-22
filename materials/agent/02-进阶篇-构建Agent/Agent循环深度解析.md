---
title: Agent循环深度解析
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 1
level: 进阶
status: 未开始
tags: [agent, intermediate, loop, react, rewoo]
confidence: high
teaching_method: V2-费曼
est_hours: 1
contested: false
contradictions: []
---

# Agent 循环深度解析

> 入门篇学了 Agent Loop 的基本概念——Thought → Action → Observation。但真实世界的 Agent 远不止一种循环方式。这一篇带你深入四种主流循环模式，理解每种模式的适用场景和代价。

---

## 回顾：Agent 的"呼吸"

每个 Agent 都在跑循环。入门篇讲的 ReAct（Reason + Act）是最基础的版本：

```
Thought → Action → Observation → Thought → Action → ... 直到完成
```

这是 Yao et al. 2022 年提出的范式。在 ALFWorld 任务上比传统方法高 34 分，在 WebShop 上高 10 分。

**但 ReAct 有一个致命问题**：每一步都要带着之前所有的上下文。一个 40 步的任务，第 40 步的 prompt 里装着前面 39 步的全部历史。Token 消耗不是线性的——是近似二次增长的。

这就是为什么我们需要更多循环模式。

> 🤔 **思考题**：假设你的 Agent 需要查 10 个独立网站的信息来回答一个问题。用 ReAct 模式，第 10 次查询的 prompt 里装着前 9 次查询的全部结果——这些结果真的都需要吗？

---

## 四种 Agent 循环模式

### 模式一：ReAct（边想边做）

```
适用：环境不确定、步骤不可预测、短链任务
代价：Token 消耗大、长链会"漂移"
```

**核心机制**：每一步都重新审视全部历史，决定下一步做什么。

**优势**：
- 最灵活——每一步都能根据最新结果调整方向
- 实现简单——一个 while 循环加一个 LLM 调用
- 错误恢复自然——失败信息在上下文里，模型自己会调整

**劣势**：
- Token 消耗大。10 步以后，大部分 token 都在"复习历史"
- 长链漂移。超过 20 步后，模型容易忘记最初的目标
- 无法并行。必须等上一步结束才能开始下一步

**生产建议**：预算 40-400 步。始终设置 `max_steps` 和"无工具调用即结束"的停止条件。

> 🤔 **思考题**：你在入门篇学的 Agent Loop 就是 ReAct。想一个场景，ReAct 是明显错误的选择——为什么？

---

### 模式二：ReWOO（先计划，再执行）

```
适用：工具体系明确、子任务可并行、结构化任务
代价：计划错了无法中途调整
```

**核心机制**：ReWOO = **Re**asoning **W**ith**O**ut **O**bservation。把 Agent 拆成三个角色：

1. **Planner（规划者）**：收到问题 → 生成执行计划 DAG（有向无环图）
2. **Workers（执行者）**：按 DAG 并行执行各节点，收集证据
3. **Solver（求解者）**：拿到全部证据 → 生成最终答案

**关键优势**：
- Token 效率提升约 5 倍（HotpotQA 实测）
- 独立子任务可以并行执行
- Planner 可以蒸馏到 7B 小模型（因为它不看 observations）

**关键劣势**：
- 计划写死了，执行中发现信息不够无法回头
- 一个 Worker 失败，Solver 只能看到错误字符串，无法重新规划
- 不适合需要动态探索的任务

**生产建议**：任务结构清晰（"收集 A B C 三个证据然后综合"）用 ReWOO。加一个 replanner 节点变成 Plan-and-Execute 能解决大部分韧性不足的问题。

> 🤔 **思考题**：你让 Agent "比较 React 和 Vue 的生态系统成熟度"。用 ReAct 和 ReWOO 两种方式，整个流程会有什么不同？

---

### 模式三：Reflexion（从失败中学习）

```
适用：可重复的任务类别、有明确成功/失败信号
代价：需要多次试运行、反思可能过时
```

**核心机制**：Reflexion 不改模型权重，而是让 Agent 用自然语言写"反思笔记"：

1. **Actor**：执行一次 ReAct 轨迹
2. **Evaluator**：判断这次执行成功还是失败（三种方式：外部标量 > 预定义启发式 > LLM 自评）
3. **Self-Reflector**：如果失败，写一段反思——"我失败是因为……下次应该……"
4. **Episodic Memory**：反思存入记忆，下一次执行时自动注入 prompt

**为什么厉害**：没有任何梯度更新，纯粹靠自然语言反馈，在 HumanEval 和 MBPP 上达到了当时的 SOTA。

**三种评估器**（从强到弱）：
- **标量评估**：外部测试通过/失败 → 信号最强
- **启发式评估**："连续 5 步都在改同一个文件" → 检测卡死
- **自评**：LLM 自己判断 → 信号最弱，没有标量时用

**生产陷阱——记忆腐烂**：反思越攒越多，旧的反思可能已经过时。解决：设 TTL（过期时间）、定期压缩、用睡眠时间的清理 Agent。

> 🤔 **思考题**：Claude Code 的 CLAUDE.md 和 "save memory" 功能，本质上是不是 Reflexion？它们之间有什么关系？

---

### 模式四：ToT / LATS（搜索式推理）

```
适用：正确性比速度重要、有可靠评估器（如单元测试）
代价：Token 消耗是普通推理的 100-1000 倍
```

**核心机制**：

- **Tree of Thoughts（ToT）**：把推理变成一棵树。每个节点是一个中间想法，LLM 自评每个节点的好坏，用 BFS/DFS/Beam Search 找最佳路径。在 Game of 24 上，CoT 只有 4%，ToT 是 74%。
- **LATS**：ToT + ReAct + Reflexion + MCTS（蒙特卡洛树搜索）。LLM 同时充当策略网络、价值函数和反思者。HumanEval pass@1 达到 92.7%。

**为什么不常用**：100-1000 倍的 token 消耗。大多数生产任务不需要搜索——ReAct + 工具验证（CRITIC 模式）就够了。

**2026 年的使用场景**：
- 编码 Agent（测试就是天然的可靠评估器）
- 深度研究 Agent（需要探索多个方向）
- LangGraph 子图（在特定节点启动搜索）

> 🤔 **思考题**：什么时候你会愿意为一个问题花 100 倍的 token 去搜索答案？想一个具体的场景。

---

## 怎么选？一张决策表

| 任务特征 | 推荐模式 | 为什么 |
|----------|----------|--------|
| 步骤少（<10）、环境不确定 | ReAct | 最简单，够用 |
| 工具体系明确、可并行收集信息 | ReWOO | Token 效率高，延迟低 |
| 可重复的任务、有明确成功标准 | Reflexion | 越跑越好 |
| 正确性 > 速度、有可靠评估器 | ToT/LATS | 搜索保证最优 |
| 长链任务（>20步）、需要重新规划 | Plan-and-Execute | ReWOO + replanner |
| 有外部验证工具（测试、类型检查） | CRITIC（ReAct + 工具验证） | 2026 生产默认 |

---

## 一个贯穿全篇的问题

你现在理解了四种循环模式。回头看入门篇的 Agent Loop——那个 `Thought → Action → Observation` 循环——它在哪个环节可以被替换成 ReWOO？在哪个环节可以用 Reflexion 增强？

试着画一张图：一个同时使用 ReWOO（规划层）+ ReAct（执行层）+ Reflexion（反思层）的三层 Agent 架构。

---

## 延伸阅读

- [[规划与执行模式]] — ReWOO 和 Plan-and-Execute 的深入对比
- [[Agent的记忆系统]] — Reflexion 的 Episodic Memory 怎么存、怎么检索
- [[Anthropic工作流模式]] — Anthropic 官方的 Evaluator-Optimizer 模式就是 Self-Refine 的工程化版本

## 相关

- [[理解Agent]] — 入门篇：Agent Loop 基础
- [[驾驭工程]] — 入门篇：Harness 如何管理这些循环模式
