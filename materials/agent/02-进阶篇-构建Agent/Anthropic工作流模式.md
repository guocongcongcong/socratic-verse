---
title: Anthropic工作流模式
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 4
level: 进阶
status: 未开始
tags: [agent, intermediate, workflow, anthropic, patterns]
confidence: high
teaching_method: V2-费曼
est_hours: 0.75
contested: false
contradictions: []
---

# Anthropic 工作流模式

> 入门篇学了"Agent 是什么"，进阶前几篇学了各种循环模式。但 Anthropic 官方说了一句让很多人意外的话：**大多数问题不需要 Agent，一个简单的工作流就够了。**

---

## 核心区分：Workflow vs Agent

Schluntz 和 Zhang（Anthropic, 2024年12月）画了一条关键的分界线：

| | Workflow（工作流） | Agent（智能体） |
|---|-------------------|----------------|
| **控制权** | 工程师预定义路径 | 模型自主决定路径 |
| **可预测性** | 高——你知道每一步会发生什么 | 低——路径在运行时决定 |
| **调试难度** | 低——每一步是确定性的 | 高——同样的输入可能走不同路径 |
| **成本** | 低——token 消耗可预估 | 高——步数不定，成本不定 |
| **适用场景** | 步骤可枚举的任务 | 开放式的、不可预测的任务 |

**判断法则**：如果你能提前写出这个任务的执行步骤清单，就用 Workflow。如果你写不出来——因为你不知道中间会发生什么——才需要 Agent。

> 🤔 **思考题**："帮我写一篇关于 AI 的博客"——你能提前写出步骤清单吗？"帮我在网上调研 AI Agent 的最新进展，写一份报告"——能吗？为什么后一个需要 Agent？

---

## 五种 Workflow 模式

### 模式一：Prompt Chaining（提示词链）

```
输入 → LLM调用1 → 输出1 → LLM调用2 → 输出2 → ... → 最终输出
```

最简单的模式。上一步的输出是下一步的输入。

**适合**：任务可以干净地分解为顺序步骤。比如：
- 先翻译，再润色
- 先生成大纲，再逐段扩写
- 先提取关键信息，再生成摘要

**不适合**：有分支逻辑的任务（用 Routing），有相互独立子任务的任务（用 Parallelization）。

### 模式二：Routing（路由）

```
输入 → 分类器 → 路径A的LLM → 输出
              → 路径B的LLM → 输出
              → 路径C的LLM → 输出
```

一个分类器（可以是关键词匹配、embedding 相似度、或一个小 LLM）决定输入走哪条路径。

**适合**：不同类型的输入需要截然不同的处理方式。比如：
- 用户问账单问题 → 账单专用 prompt
- 用户问技术问题 → 技术专用 prompt
- 用户投诉 → 升级到人工

**不适合**：所有输入可以用同一个 prompt 处理的情况。

> 🤔 **思考题**：你有一个客服 Agent。用户说"我想退款"和"这个功能怎么用"——用一个 prompt 处理还是分开？为什么？

### 模式三：Parallelization（并行化）

```
输入 → 拆分成N个子任务
     → 子任务1 → 结果1 ↘
     → 子任务2 → 结果2 → 聚合 → 输出
     → 子任务3 → 结果3 ↗
```

两种并行策略：
- **分段（Sectioning）**：不同子任务处理输入的不同部分
- **投票（Voting）**：同一个任务跑多次，取多数或综合

**适合**：
- Sectioning：代码审查（一个看安全，一个看性能，一个看风格）
- Voting：需要高准确率的任务（三个独立评估，取共识）

**不适合**：子任务之间有依赖关系（用 Prompt Chaining）。

### 模式四：Orchestrator-Workers（编排者-执行者）

```
输入 → Orchestrator（动态规划）→ Worker A → 结果A
                               → Worker B → 结果B
                               → Worker C → 结果C
                               → 综合 → 输出
```

和 Parallelization 的关键区别：**Worker 的数量和任务不是预定义的**。Orchestrator LLM 动态决定"要派几个 Worker、每个做什么"。

**适合**：子任务的数量和内容取决于输入内容。比如：
- "帮我分析这个代码库"——根据代码规模动态决定分析维度
- "帮我调研 X 主题"——根据搜索结果决定深入哪些方向

**不适合**：子任务数量和内容完全可预测（用 Parallelization 更便宜）。

### 模式五：Evaluator-Optimizer（评估者-优化者）

```
Proposer（生成）→ Evaluator（评估）→ 不通过 → Proposer（改进）→ Evaluator → 通过 → 输出
```

这就是 Self-Refine / CRITIC 的工程化版本。核心要点：**Evaluator 和 Proposer 的 prompt 必须实质不同**，否则会产生"橡皮图章"——模型自己生成自己评判，永远"看起来不错"。

**适合**：质量要求高、有明确评估标准的任务。比如：
- 翻译（评估者检查准确性和流畅度）
- 代码生成（评估者用测试和 linter 验证）
- 文档写作（评估者检查完整性和清晰度）

**不适合**：没有可靠评估方式的任务（评估者本身不可靠）。

> 🤔 **思考题**：Evaluator 也是 LLM。如果 Evaluator 自己也有偏见怎么办？CRITIC 模式给了什么解决方案？

---

## 什么时候该用 Agent？

Anthropic 的建议：

> 当步骤不可预知、工具调用取决于动态环境、或者任务收益能覆盖 token 成本时——才升级到 Agent。

Agent 是 Workflow 的超集。Agent 循环内部嵌着 Workflow 模式：
- Agent 决定调用哪个工具 = Routing
- Agent 同时调用多个工具 = Parallelization
- Agent 自我纠正 = Evaluator-Optimizer

**升级清单**：
1. ✅ 你能枚举所有步骤吗？→ 能：用 Workflow
2. ✅ 任务需要动态工具选择吗？→ 是：考虑 Agent
3. ✅ 任务长度不确定（可能 10 分钟也可能 1 小时）？→ 是：考虑 Agent
4. ✅ Token 成本能被任务收益覆盖吗？→ 是：用 Agent

---

## 一个真实的决策案例

任务：**"自动回复 GitHub Issues，标记重复问题，给新贡献者指路。"**

**用 Workflow**：
1. 读取 issue 内容
2. 分类器判断类型（bug / feature / question / duplicate）
3. 按类型走不同的回复模板
4. 如果是 duplicate，搜索相似 issue
5. 输出回复草稿
→ 全程确定性，成本可预估，出问题好排查

**用 Agent**：
1. Agent 读取 issue
2. 自己决定去搜文档还是搜历史 issue
3. 自己判断是否需要 @ 某位维护者
4. 可能反复搜索、比对、修改回复
→ 更灵活，但成本不可预估，可能循环 20 步才发现"其实这是个简单问题"

**选哪个**？2026 年的答案：用 Workflow 做 90% 的常规回复，用 Agent 处理那 10% "分类器也不确定"的 issue。

> 🤔 **思考题**：为什么不是 100% Agent？毕竟它更"智能"。Trade-off 在哪里？

---

## 延伸阅读

- [[Agent循环深度解析]] — Evaluator-Optimizer 就是 Self-Refine 的工程化
- [[Agent框架对比]] — 什么时候 Workflow 不够用了，需要框架
- [[Anthropic-构建高效Agent]] — 入门篇：原版 Anthropic 文章

## 相关

- [[理解Agent]] — 入门篇：Agent 基础
- [[驾驭工程]] — 入门篇：Harness 如何管理 Workflow 和 Agent 的边界
