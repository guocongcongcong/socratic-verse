---
title: Agent的记忆系统
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 3
level: 进阶
status: 未开始
tags: [agent, intermediate, memory, memgpt, mem0]
confidence: high
teaching_method: V1-苏格拉底
est_hours: 1
contested: false
contradictions: []
---

# Agent 的记忆系统

> 入门篇讲了上下文工程——怎么在有限的 context window 里塞最有用的信息。这一篇解决一个更深的问题：如果对话持续一周、一个月、一年，Agent 怎么"记住"？

---

## 问题：Context Window 不是记忆

模型有一个致命的局限：**每次对话从零开始**。Context Window 是 Agent 的"工作记忆"——像你的 RAM，断电即失。但真实世界需要持久记忆：

- 用户上周说过"我不喜欢用 Docker"
- 三天前 Agent 尝试过方案 A，失败了
- 六个月里用户偏好从 Vue 转到了 React

这些信息不能全塞在 prompt 里——context window 有上限，而且塞得越多，模型注意力越分散。

> 🤔 **思考题**：你有一个 128K context window 的模型。为什么把六个月的对话历史全塞进去不是一个好的解决方案？

---

## 核心思路：操作系统虚拟内存

MemGPT（Packer et al., 2023）提出了一个影响深远的类比：

| OS 概念 | Agent 中的对应 |
|---------|---------------|
| RAM（内存） | Main Context（主上下文，固定大小，始终可见） |
| Disk（硬盘） | External Context（外部上下文，无界，通过工具搜索） |
| Page Fault（缺页中断） | Memory Tool Call（`memory.search`, `memory.read`） |
| OS Kernel | Agent ReAct Loop |

**关键洞察**：Agent 不需要一开始就知道一切。它只需要知道**去哪找**。就像你的操作系统不会把所有文件加载到内存——它只在需要时读盘。

### MemGPT 的核心工具

```
core_memory_append     → 往"核心记忆"里追加（用户偏好、关键事实）
core_memory_replace    → 替换过时的核心记忆
archival_memory_insert → 往"档案记忆"里存（对话记录、文档摘要）
archival_memory_search → 搜索档案记忆（语义搜索）
conversation_search    → 搜索历史对话
```

Agent 在对话中调用这些工具，就像操作系统在程序需要时从磁盘读数据。

> 🤔 **思考题**：MemGPT 把记忆分成"核心"和"档案"两层。一个用户说"我讨厌 Docker"——这应该进核心还是档案？为什么？

---

## 记忆的三种"作用域"

分清三个概念，否则会出数据泄漏事故：

| 作用域 | 存在时间 | 共享范围 | 存什么 |
|--------|---------|---------|--------|
| **User Memory（用户记忆）** | 跨 session | 同一用户的所有 session | 偏好、习惯、长期事实 |
| **Session Memory（会话记忆）** | 单次 session | 当前 session 内 | 本轮对话上下文 |
| **Agent Memory（Agent 记忆）** | Agent 实例生命周期 | 同一 Agent 实例 | Agent 自身状态、中间结果 |

**关键原则**：不要把用户 A 的信息写到用户 B 能读到的位置。

---

## Mem0：三合一记忆架构

2025 年，Mem0 提出了更精细的方案：**一种存储满足不了所有查询类型**。

Agent 的查询有三种：

1. **语义查询**："和 X 相关的内容" → 需要 **Vector Store**（向量数据库）
2. **精确匹配**："X 的 email 是什么" → 需要 **KV Store**（键值存储）
3. **关系查询**："X 和 Y 什么关系" → 需要 **Graph Store**（图数据库）

Mem0 在每次 `add()` 时同时写入三个 store，在 `search()` 时融合三个 store 的结果：

```
score = w_relevance × 相关性 + w_importance × 重要性 + w_recency × 新鲜度
```

三个权重根据场景调整——聊天 Agent 提高 recency，合规 Agent 提高 importance，检索 Agent 提高 relevance。

**实际效果**：128K context window 的裸 LLM 在长时域基准上被 4K window + Mem0 的 Agent 反超 10+ 分。

> 🤔 **思考题**：一个客服 Agent 和一个代码审查 Agent，它们的 w_relevance / w_importance / w_recency 权重应该怎么设？

---

## 记忆系统的四大陷阱

### 陷阱一：溢出（Overflow）
Context window 装不下了。**解法**：外部存储 + 按需检索，不管 context 多大都只在 prompt 里放最相关的内容。

### 陷阱二：稀释（Dilution）
塞了太多不相关的信息，模型注意力被分散。**解法**：检索时设高相关性阈值，宁可少拿不要滥拿。

### 陷阱三：遗忘（Persistence）
新 session 从零开始，历史全丢。**解法**：每次 session 启动时自动检索核心记忆并注入 prompt。

### 陷阱四：记忆腐烂（Memory Rot）
旧信息过时了但还在检索结果里。**解法**：
- 设 TTL（过期时间）
- 时间感知检索（"用户 3 月的城市是哪里？"——查 3 月有效的事实）
- Mem0 的冲突检测：新事实与旧事实矛盾时，旧事实标记为 invalid 但不删除

> 🤔 **思考题**：你的 Agent 记得用户 6 个月前说"最喜欢的语言是 Python"。但用户最近三个月一直在问 Rust 问题。这个记忆应该怎么处理？

---

## 2026 年选型指南

| 方案 | 适合 | 不适合 |
|------|------|--------|
| **Mem0** (Apache 2.0) | 需要完整三合一记忆的生产系统 | 极简场景（一个 KV 就够了） |
| **Letta** (原 MemGPT) | 需要三层记忆 + 睡眠时计算 | 只需要基础检索的系统 |
| **OpenAI Assistants** | 已经用 OpenAI 全家桶 | 需要自托管 |
| **Claude Session Store** | 用 Claude Agent SDK | 跨模型场景 |
| **自己搭** | 需要精确控制提取器和融合权重 | 快速原型 |

---

## 一个动手练习

读一遍你的 Claude Code 对话历史（上次和 Claude 的对话）。识别出：
1. 哪些信息应该进核心记忆？
2. 哪些该进档案记忆？
3. 哪些该在下次 session 开始时检索出来？

然后问自己：如果 Claude 有这个记忆系统，第二次对话的体验会有什么不同？

---

## 延伸阅读

- [[上下文工程]] — 入门篇：context window 管理的基础
- [[Agent循环深度解析]] — Reflexion 的记忆就是 Episodic Memory
- [[Agent工作台实战]] — 高级篇：生产环境怎么管理记忆

## 相关

- [[理解Agent]] — 入门篇：Agent 基础
- [[Anthropic-构建高效Agent]] — 参考文献：记忆在 Agent 架构中的位置
