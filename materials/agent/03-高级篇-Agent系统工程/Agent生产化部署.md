---
title: Agent生产化部署
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 4
level: 高级
status: 未开始
tags: [agent, advanced, production, deployment, observability]
confidence: high
teaching_method: V8-自适应
est_hours: 1
contested: false
contradictions: []
---

# Agent 生产化部署

> 在笔记本上跑一个 Agent 是玩具。在服务器上跑 100 个 Agent、24×7、用户数据安全、成本可控——那是工程。

---

## 单机 vs 生产：差距在哪

| 维度 | 笔记本原型 | 生产系统 |
|------|-----------|---------|
| 可靠性 | "挂了就重跑" | 挂了要恢复、不能丢状态、不能重复执行 |
| 并发 | 1 个任务 | 100 个任务并行 |
| 成本 | "token 不贵" | 每个任务有预算，超支要报警 |
| 安全 | "我用我自己的电脑" | 多租户隔离、PII 脱敏、审计日志 |
| 监控 | "看终端输出" | 分布式追踪、指标、告警 |
| 升级 | "改一行重启" | 灰度发布、A/B 测试、回滚 |

> 🤔 **思考题**："挂了就重跑"在生产里有什么问题？假设 Agent 在第 8 步已经给用户发了邮件，第 9 步崩溃了，重跑会发生什么？

---

## 生产基础设施分层

```
┌──────────────────────────────────────┐
│ 接入层：API Gateway + 认证 + 限流    │
├──────────────────────────────────────┤
│ 编排层：队列 + 调度 + 优先级         │
├──────────────────────────────────────┤
│ 执行层：Sandbox + Checkpoint         │
├──────────────────────────────────────┤
│ 模型层：路由 + 缓存 + 降级           │
├──────────────────────────────────────┤
│ 可观测层：Trace + Metric + Log        │
├──────────────────────────────────────┤
│ 安全层：PII 脱敏 + 审计 + 合规       │
└──────────────────────────────────────┘
```

---

## 编排层：三种运行时模式

### 模式一：Queue-Driven（队列驱动）

```
用户请求 → 入队 → Worker Pool → 执行 → 结果
                        ↓ 崩溃
                    Checkpoint → 另一个 Worker 接手
```

**适合**：异步任务（用户提交后等结果）。大多数生产 Agent 是这种模式。

**关键组件**：
- 消息队列（Redis, RabbitMQ, SQS）
- Worker 池（自动扩缩）
- 死信队列（处理不了的请求放这里，后续人工处理）
- 任务去重（幂等性——同一个请求不能执行两次）

### 模式二：Event-Driven（事件驱动）

```
事件 → 触发器 → Agent → 结果 → 下一个事件
 ↑                              ↓
 └──────── 反馈循环 ────────────┘
```

CrewAI 的 Flows 就是这种：`@start` 和 `@listen(topic)` 定义事件驱动的确定性路径。

**适合**：持续运行的 Agent（监听 GitHub webhook、自动回复 issue）。

### 模式三：Cron-Driven（定时触发）

```
每 30 分钟 → 触发 Agent → 检查上一轮的结果 → 决定下一步
```

**适合**：周期性任务（每小时检查一次线上错误、每天生成运营报告）。

> 🤔 **思考题**：一个代码审查 Agent——每次有新的 Pull Request 就自动审查。应该用哪种运行时模式？为什么？

---

## 执行层：Checkpoint 和 Durable Execution

### Checkpoint 的颗粒度

| 颗粒度 | Checkpoint 内容 | 恢复成本 | 存储成本 |
|--------|----------------|---------|---------|
| **每步** | 完整状态快照 | 低（精确恢复） | 高（55 步 = 55 个快照） |
| **每 N 步** | 完整状态快照 | 中（回滚到上一个快照） | 中 |
| **关键点** | 只在有副作用操作前 | 高（可能回到很远） | 低 |

**LangGraph 的 Checkpointer** 就是为"每步"设计的——SQLite/Postgres/Redis 存储。

### Checkpoint 一定要包含的

- ❌ 只存对话历史 → 不够。工具状态（"文件已打开"、"数据库连接已建立"）丢了。
- ❌ 只存状态值 → 不够。非确定性因素（random seed, 当前时间, 外部 API 响应）没存进去。
- ✅ 存完整状态 + 非确定性因素的快照 → 恢复后能精确重现。

### Idempotency（幂等性）

Agent 在步骤 8 给用户发了退款。步骤 9 崩溃了。从步骤 8 的 checkpoint 恢复——又发了一次退款。

**解法**：每个有副作用的操作（退款、发邮件、删文件）带一个 **idempotency key**。同样的 key 执行两次 → 第二次被拒绝。

---

## 模型层：路由、缓存、降级

### Model Routing（模型路由）

```
简单任务 → 小模型（便宜）
中等任务 → 中等模型
复杂任务 → 大模型（贵）
```

一个分类器在任务开始时决定路由。可以省 50-80% 的模型成本。

**关键设计**：路由本身也要消耗 token。100 token 的路由 + 200 token 的便宜模型 vs 300 token 的贵模型——前者不一定更便宜。

### Prompt Caching（提示词缓存）

Agent 的 system prompt 每次调用都一样。Anthropic 的 prompt caching 让你只付一次——后续调用打 90% 折扣。

**适用**：长 system prompt、大工具描述。不适合：每轮都不同的对话历史。

### Graceful Degradation（优雅降级）

主模型挂了 → 自动切到备用模型。MCP 服务器超时了 → 告诉 Agent "该工具不可用"，让 Agent 用替代方案。

---

## 可观测层：OpenTelemetry GenAI

2026 年标准：OTel GenAI Semantic Conventions。

每个 Agent 调用生成一个 trace，包含以下 span：

```
Agent Run（trace）
├── LLM Generation 1（span）
│   ├── Prompt tokens: 2500
│   ├── Completion tokens: 150
│   └── Model: claude-fable-5
├── Tool Call: search_web（span）
│   ├── Input: "AI Agent"
│   ├── Output: [...]
│   └── Duration: 1.2s
├── LLM Generation 2（span）
│   └── ...
├── Handoff → code_reviewer（span）  ← 多 Agent 追踪
│   └── Sub Agent Run（child trace）
└── Guardrail: output_check（span）
    └── Passed: true
```

**核心平台**：Langfuse, Arize Phoenix, Comet Opik。

---

## 成本控制实战

### Action Budget（行动预算）

```
"这个任务最多 40 步"
"这个任务最多花 $2"
"这个任务最多调 3 次外部 API"
```

三个预算同时生效——任何一个触顶就停止。

### 成本报告格式

```
任务: 修复 issue #342
状态: 成功
步数: 23
Token: 输入 124,500 / 输出 8,320
成本: $1.87
模型: Claude-Fable-5 (17步) + Claude-Haiku (6步路由)
时间: 4分32秒
```

> 🤔 **思考题**：你在笔记本上跑 Agent 修一个 bug，花了 $1.87。同一个 bug 你自己修要 1 小时。怎么算这个 Agent 是"值得的"？如果 bug 没修对呢？

---

## 延伸阅读

- [[Agent评估体系]] — 进阶篇：在线评估是生产监控的关键
- [[自主Agent系统设计]] — 高级篇：长时域任务的基础设施
- [[Agent安全与对齐]] — 高级篇：生产安全的完整方案

## 相关

- [[驾驭工程]] — 入门篇：多层架构在设计上就考虑了生产化
- [[理解Agent]] — 入门篇：回到 Agent Loop 最基本的形式
