---
title: Agent框架对比
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 5
level: 进阶
status: 未开始
tags: [agent, intermediate, framework, langgraph, autogen, crewai, sdk]
confidence: high
teaching_method: V7-对子互教
est_hours: 1
contested: false
contradictions: []
---

# Agent 框架对比

> 入门篇教你从零写 Agent Loop。但真实项目里没人从零写——框架帮你处理状态管理、错误恢复、并行调度。问题是：2026 年有五个主流框架，该选哪个？

---

## 先问一个更根本的问题

你真的需要框架吗？

Anthropic 的建议是：**先用 stdlib 实现**。一个 Agent Loop 不过 100-200 行代码。当你明确感到痛——状态丢了没法恢复、并发控制写崩了、角色管理混乱——那时候再引入框架。

框架帮你解决的问题：
- ✅ 状态持久化和恢复
- ✅ 多 Agent 之间的消息路由
- ✅ 可观测性（追踪、日志、指标）
- ✅ 并发和容错

框架带来的代价：
- ❌ 学习曲线和依赖
- ❌ 抽象泄漏（出了问题你要懂底层才能修）
- ❌ 升级和维护负担

> 🤔 **思考题**：200 行的 Agent Loop 和 2000 行的框架集成，哪一个更容易出 bug？哪一个出了问题更容易修？

---

## 五大框架速览

### 1. LangGraph — 状态机模型

**一句话**：把 Agent 建模为有向图——节点是函数，边是状态转移，每一步自动 checkpoint。

**核心概念**：
- State：类型化的状态对象（TypedDict 或 Pydantic）
- Node：纯函数 `(state) → state_update`
- Edge：条件边（LLM 决定走哪条）或直接边（固定路由）
- Checkpointer：每一步后序列化状态（支持 SQLite/Postgres/Redis）

**杀手特性**：**持久化执行**。Agent 在步骤 38 崩溃了？`resume(session_id)` 从步骤 38 继续，状态完全恢复。Klarna、Uber、J.P. Morgan 在用。

**什么时候选**：
- ✅ 你的 Agent 有明显的图结构（分支、循环、条件路由）
- ✅ 需要崩溃恢复（长期运行的任务）
- ✅ 需要人工审批节点（HITL）
- ❌ 简单的线性链——杀鸡用牛刀

**注意**：2026 年 LangChain 团队推荐用 tool-call-based supervisor 而不是内置的 `create_supervisor()`——工具调用路由有更好的上下文控制。

### 2. AutoGen v0.4 — Actor 模型

**一句话**：每个 Agent 是一个 Actor——有私有状态、收件箱、`receive(message)` 处理器。消息是唯一的通信方式。

**核心概念**：
- Actor：私有状态 + 收件箱 + 消息处理器
- 两个 Actor 不能共享内存——消息是唯一通道
- 一个 Actor 崩溃不影响其他 Actor

**杀手特性**：**故障隔离**。一个 Agent 挂了不会拖垮整个系统。AutoGen v0.4（2025年1月重写）把整个架构从同步调用改成了 Actor 模型。

**什么时候选**：
- ✅ 需要 Agent 之间的故障隔离
- ✅ 异步协作场景
- ✅ 研究和原型验证
- ❌ 生产新项目——微软 Agent Framework（2025年10月公测）是后继者

**注意**：v0.7.x 是稳定版，但处于维护模式。新项目考虑微软 Agent Framework 或 LangGraph。

### 3. CrewAI — 角色扮演模型

**一句话**：给每个 Agent 分配角色（Role）、目标（Goal）、背景故事（Backstory），像一个团队一样协作。

**核心概念**：
- Agent：role + goal + backstory + tools
- Task：description + expected_output + agent
- Crew：agents + tasks + process（Sequential / Hierarchical）
- Flow：事件驱动的确定性执行路径

**杀手特性**：**Crews + Flows 双模式**。Crews 用于 LLM 驱动的自主协作（研究、头脑风暴），Flows 用于生产级的确定性路径。2026 官方建议：**生产应用从 Flow 开始**，在需要自主性的环节嵌入 Crew。

**什么时候选**：
- ✅ 需要快速原型多角色协作
- ✅ 研究和头脑风暴类任务
- ✅ 团队结构天然适合角色建模
- ❌ 需要精确控制执行路径（用 Flow 覆盖）
- ❌ 支付/删除等有副作用操作（放 Flow 里，别放 Crew 里）

**三个常见坑**：
1. Backstory 太长（>200 字/Agent）——烧 context budget
2. Hierarchical 模式每次路由多一次 LLM 调用——token 成本可能翻三倍
3. 任务之间用自由文本传数据——用 `output_pydantic` 强制类型

### 4. OpenAI Agents SDK — Handoff 模型

**一句话**：Agent 之间的委托建模为工具调用——Agent A 调用 `transfer_to_agent_b` 工具来移交任务。

**核心概念**：
- Agent：LLM + instructions + tools + handoffs
- Handoff：建模为名叫 `transfer_to_<agent_name>` 的工具
- Guardrail：输入/输出/工具调用的安全检查
- Session：自动管理对话历史（SQLite/Redis/自定义）
- Tracing：默认开启，每个 LLM 调用/工具调用/handoff 都有 span

**杀手特性**：**Handoff 作为一等公民**。多 Agent 协作不需要额外框架——就是工具调用的自然扩展。

**什么时候选**：
- ✅ 已经在 OpenAI 生态里
- ✅ 需要 Guardrail 做安全检查
- ✅ 多 Agent 通过 Handoff 协作
- ❌ 需要非 OpenAI 模型（架构深度绑定）
- ❌ 需要复杂的图结构控制（用 LangGraph）

### 5. Claude Agent SDK — Harness 模型

**一句话**：Claude Code 的库形式。内置文件/Shell/Grep/Glob/Web 工具，Subagent 并行化，生命周期 Hook，Session Store。

**核心概念**：
- Subagent：两种用途——并行化（20 个独立任务同时跑）和上下文隔离（子 Agent 有自己的 context window）
- Hook：PreToolUse, PostToolUse, SessionStart, SessionEnd 等生命周期钩子
- Session Store：append/load/list/delete，支持 `--session-mirror` 实时转录
- W3C Trace Context：跨进程追踪传播

**杀手特性**：**Subagent 上下文隔离**。Orchestrator 的 context budget 不受子 Agent 影响——子 Agent 在自己的 context 里工作，只返回结果。

**什么时候选**：
- ✅ 用 Claude Code 作为开发环境
- ✅ 需要 Subagent 并行化 + 上下文隔离
- ✅ 需要生命周期 Hook（速率限制、审计日志）
- ❌ 需要多模型支持
- ❌ 需要复杂的图结构（不是设计目标）

> 🤔 **思考题**：OpenAI Agents SDK 的 Handoff 和 Claude Agent SDK 的 Subagent，表面很像。本质区别是什么？（提示：一个在同一个 context 里路由，一个创建独立 context）

---

## 选型决策表

| 需求 | 推荐 | 不推荐 |
|------|------|--------|
| "我想从零学 Agent" | **不用框架，写 200 行 Loop** | 任何框架 |
| "我有一个有分支和循环的复杂流程" | **LangGraph** | Claude Agent SDK |
| "我需要多个 Agent 角色协作" | **CrewAI Flows** | AutoGen |
| "我在 OpenAI 生态里，需要 Guardrail" | **OpenAI Agents SDK** | LangGraph |
| "我用 Claude Code，需要并行 Subagent" | **Claude Agent SDK** | OpenAI Agents SDK |
| "我需要崩溃恢复和持久化状态" | **LangGraph** | CrewAI |
| "我的 Agents 需要故障隔离" | **AutoGen / Claude Subagent** | CrewAI |
| "快速原型，不写太多代码" | **CrewAI** | LangGraph |

---

## 核心教训

1. **从简单开始**。Anthropic 的官方立场：先用直接 API 调用，不够再加 Workflow，还不够才上 Agent 框架。
2. **框架选择是锁定决策**。换框架的成本很高——迁移状态管理、重写工具集成、重新培训团队。
3. **所有框架底下都是同一个 Agent Loop**。你入门篇学的那个 `Thought → Action → Observation` 循环，在 LangGraph 里叫 Node，在 AutoGen 里叫 Message Handler，在 CrewAI 里叫 Task.execute()，在 OpenAI SDK 里叫 Runner.run()——本质不变。

---

## 延伸阅读

- [[Anthropic工作流模式]] — 大多数时候你不需要框架，Workflow 就够了
- [[Agent循环深度解析]] — 所有框架底层的循环模式
- [[Agent生产化部署]] — 高级篇：框架选好之后怎么上线

## 相关

- [[理解Agent]] — 入门篇：Agent 基础
- [[OpenAI-Agents-SDK指南]] — 参考文献：OpenAI SDK 完整指南
