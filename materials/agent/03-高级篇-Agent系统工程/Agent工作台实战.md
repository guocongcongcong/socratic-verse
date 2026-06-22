---
title: Agent工作台实战
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 5
level: 高级
status: 未开始
tags: [agent, advanced, workbench, capstone, production]
confidence: high
teaching_method: V7-对子互教
est_hours: 1.5
contested: false
contradictions: []
---

# Agent 工作台实战

> 学完整个课程，你有了概念、模式、框架、安全知识。现在把一切整合起来——搭一个 Agent Workbench，让 Agent 在一个受控、可观测、可调试的环境里工作。

---

## 什么是 Agent Workbench

不是"一个 Agent"。是一个 **Agent 的工作环境**——就像开发者的 IDE。

```
Agent Workbench
├── 指令约束 → Agent 能做什么、不能做什么
├── 工具注册表 → 所有可用工具 + schema + 权限
├── 验证门 → 每一步/每 N 步自动验证
├── 反馈环 → Agent 的"反思"和"学习"
├── 可观测性 → 每一步可追踪、可回放
└── 工作台快照 → 整个环境可以存、可以恢复
```

**设计原则**：Workbench 属于**工程师**——你定义规则。Agent 在规则内自主——但它不能修改规则。

---

## 组件一：指令作为可执行约束

**不好的指令**（自然语言提示）：
> "请帮我审查代码，注意安全问题。"

**好的指令**（可执行约束）：
```yaml
agent_config:
  role: code_reviewer
  allowed_tools: [read_file, grep, glob, web_search]
  forbidden_tools: [write_file, shell, delete_file]
  scope:
    paths: ["src/"]
    exclude: ["src/vendor/", "src/generated/"]
  gates:
    - type: file_count
      max_files_per_turn: 5
    - type: security_check
      block_patterns: ["eval(", "exec(", "os.system("]
```

**关键区别**：自然语言可以被忽视。可执行约束无法被忽视——它是代码，不是建议。

> 🤔 **思考题**：入门篇学的"Skill"和可执行约束是什么关系？Skill 是自然语言提示，约束是代码——一个 Agent 应该两者都有吗？

---

## 组件二：工具注册表 + Schema 验证

```python
tool_registry = ToolRegistry()

# 每个工具注册时声明：
tool_registry.register(
    name="read_file",
    schema={"path": "string"},
    permissions={"read": True, "write": False, "network": False},
    annotations={"readOnlyHint": True, "destructiveHint": False},
    sandbox={
        "allowed_paths": ["/home/user/project/"],
        "max_file_size": 10 * 1024 * 1024,  # 10MB
        "timeout_ms": 5000
    }
)
```

调用时，注册表在工具执行前做三道检查：

1. **Schema 验证**：参数类型对吗？
2. **权限检查**：Agent 有权做这个吗？
3. **Sandbox 执行**：在限制范围内跑，超时/越界即停

**来自进阶篇 MCP 安全的教训**：工具描述的 hash 存在注册表里。描述变了 → 注册表拒绝加载。

---

## 组件三：验证门

Agent 在第 37 步做了什么？只能事后看日志。

**验证门**在关键步骤强制执行检查：

| 门类型 | 检查什么 | 失败后果 |
|--------|---------|---------|
| **Schema Gate** | 输出格式对吗？ | 让 Agent 重试 |
| **Test Gate** | 代码通过测试了吗？ | 让 Agent 修复 |
| **Review Gate** | 人类审批通过了吗？ | 暂停等审批 |
| **Diff Gate** | 改了多少文件？超过阈值？ | 拒绝，要求缩小范围 |
| **Security Gate** | 代码里有危险模式吗？ | 拒绝，标记给人类 |

**门的位置**：
- **Pre-tool**：工具调用前检查
- **Post-tool**：工具调用后检查
- **Post-turn**：每轮 Agent 循环后检查
- **Pre-completion**：Agent 说"我完成了"时最后一道检查

---

## 组件四：运行时反馈环

Agent 做了错误的决定。Workbench 捕获并反馈。

### 即时反馈

```
Agent 调用：delete_file("important_config.json")
Security Gate 拦截：该文件在 protected_paths 中
→ 返回给 Agent："操作被拒绝：important_config.json 是受保护文件。"
→ Agent 看到反馈，调整下一步
```

### 延迟反馈（Sleep-Time Compute）

```
Session 结束后：
→ Analyzer Agent 扫描整段轨迹
→ 发现：Agent 在第 12-16 步之间反复搜索同一个问题
→ 生成反思："下次遇到 X 类型的问题，先查 Y 文档避免重复搜索"
→ 存入记忆，下次 session 开始前注入
```

这是 Letta 的"睡眠时计算"模式——Agent 不活跃的时候做改进工作。

### 跨 Session 学习

```
Session 1：Agent 犯了错误 A
→ 反思存入 episodical memory
Session 2：Agent 遇到了类似场景
→ 启动时自动加载相关反思："上次遇到这种情况时你犯了 A，注意避免"
→ Agent 避开了错误
```

---

## 组件五：可观测性——不只是"看日志"

### 轨迹回放

一个生产 Agent 任务失败了。你不需要猜测为什么——Workbench 记录了每一步的完整快照。回放：第 23 步，Agent 读了一个文件，文件内容被截断了（工具返回只给了前 5000 字符），Agent 基于不完整信息做了错误决策。

### 关键指标

| 指标 | 含义 | 告警阈值（参考） |
|------|------|----------------|
| Task Success Rate | 任务成功率 | < 85% |
| Mean Steps Per Task | 平均步数 | 比上周高 30% |
| Tool Call Error Rate | 工具调用失败率 | > 5% |
| Guardrail Trip Rate | 安全检查触发率 | 突然增加 |
| Cost Per Task P95 | 单任务成本 P95 | > $5 |
| Self-Correction Rate | Agent 自己纠正错误的比率 | < 10%（可能反应评估太弱） |

---

## 一个完整的 Agent Workbench 项目结构

```
my-agent-workbench/
├── agents/
│   ├── code_reviewer/
│   │   ├── system.md          ← Agent 指令（自然语言）
│   │   ├── constraints.yaml   ← 可执行约束
│   │   └── eval/              ← 评估用例
│   └── code_fixer/
│       ├── system.md
│       └── constraints.yaml
├── tools/
│   ├── registry.py            ← 工具注册表
│   └── schemas/               ← 工具 schema 定义
├── gates/
│   ├── security_gate.py
│   └── review_gate.py
├── memory/
│   ├── episodic_store.py
│   └── reflections/           ← 反思笔记
├── observability/
│   └── tracer.py              ← OTel 追踪
├── sandbox/
│   └── runner.py              ← 隔离执行
├── eval/
│   ├── benchmarks/            ← 基准测试
│   └── custom/                ← 自定义 eval
└── sessions/
    ├── store.py               ← Session Store
    └── checkpointer.py        ← 持久化
```

**把这个项目结构和你入门篇学的做一个对比**：入门篇的 Agent Loop 是 `while not done: thought → action → observation`。这个 Workbench 在整个循环的外面包了一层——每一步都在约束、验证、记录。

---

## 你学完了全部三层课程

- **入门篇**：理解 Agent Loop、写 Skill、用工具
- **进阶篇**：四种循环、六种 Workflow、五种框架、Memory + MCP + 评估
- **高级篇**：自主系统、多 Agent、安全、生产、Workbench

**现在你可以**：
1. 用入门篇的知识写一个 Agent
2. 用进阶篇的知识判断该用什么模式、什么框架
3. 用高级篇的知识把它部署到生产、确保安全

**下一步**：找一个真的问题。不是"学习 Agent"——是"用 Agent 解决 X"。学到的所有东西，在一个真实的问题上会自己整合起来。

---

## 延伸阅读

- [[aAgent评估体系]] — 进阶篇：Eval-Driven Development 的完整方法论
- [[自主Agent系统设计]] — 高级篇：长期运行的基础设施
- [[Agent生产化部署]] — 高级篇：从 Workbench 到 Production

## 相关

- [[驾驭工程]] — 入门篇：Workbench 是 Harness 思想的最完整实现
- [[Anthropic-构建高效Agent]] — 参考文献：从简单开始，只在必要时加复杂度
