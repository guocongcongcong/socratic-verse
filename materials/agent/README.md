# Agent 开发完整课程

> 从零基础到能构建生产级 Agent 系统。三层递进，18 篇教材，预计 12-24 周。

---

## 课程结构

```
materials/agent/
├── README.md                        ← 你在这里
├── _database.md                     ← Obsidian 数据库视图配置
│
├── 01-入门篇-理解Agent/              ← 第 1-4 周 · 8 篇
│   ├── 学习路线总览.md
│   ├── 01-理解Agent.md              ← Agent 是什么、Agent Loop
│   ├── 02-写Skill.md                ← Skill 设计、铁律、拆解
│   ├── 03-工具与MCP.md              ← 工具设计原则、MCP 基础
│   ├── 04-上下文工程.md             ← Context Window 管理
│   ├── 05-驾驭工程.md               ← 六层架构、工程思维
│   ├── Anthropic-构建高效Agent.md    ← 参考文献
│   ├── OpenAI-Agents-SDK指南.md      ← 参考文献
│   └── LilianWeng-Agent综述.md       ← 参考文献
│
├── 02-进阶篇-构建Agent/              ← 第 5-12 周 · 8 篇
│   ├── Agent循环深度解析.md          ← ReAct / ReWOO / Reflexion / ToT
│   ├── 规划与执行模式.md             ← Plan-Execute / Plan-and-Act
│   ├── Agent的记忆系统.md            ← MemGPT / Mem0 / Letta
│   ├── Anthropic工作流模式.md        ← 5 种 Workflow 模式
│   ├── Agent框架对比.md              ← LangGraph / AutoGen / CrewAI / SDKs
│   ├── MCP从原理到生产.md            ← 协议细节 / 安全 / 生产
│   ├── Agent评估体系.md              ← SWE-bench / GAIA / Eval-Driven
│   └── 实战AgentReach拆解.md         ← 真实项目源码分析
│
└── 03-高级篇-Agent系统工程/          ← 第 13-24 周 · 5 篇
    ├── 自主Agent系统设计.md          ← 长时域 / 编码 / 浏览器 Agent
    ├── 多Agent协作与Swarms.md        ← 4 种协作拓扑 / 失败模式
    ├── Agent安全与对齐.md            ← Prompt Injection / RSP
    ├── Agent生产化部署.md            ← 队列 / Checkpoint / 可观测
    └── Agent工作台实战.md            ← 整合全部知识
```

---

## 三层递进

| 层级 | 目标 | 时间 | 教法 |
|------|------|------|------|
| **入门篇** | 理解 Agent 思维方式，能写 Skill | 2-4 周 | V1 苏格拉底追问 |
| **进阶篇** | 能用框架构建 Agent，理解评估体系 | 4-8 周 | V2 费曼 + V5 合书复述 |
| **高级篇** | 能设计和部署生产级 Agent 系统 | 8-12 周 | V7 对子互教 + V8 自适应 |

---

## 如何使用 Socratic Verse 学习

### 1. 启动

```bash
cd /Users/guoliwei/Downloads/workspace/socratic-verse
claude
```

### 2. 选择模式和教材

```
# 入门篇示例
"用 V1 苏格拉底模式教我 materials/agent/01-入门篇/理解Agent.md"

# 进阶篇示例
"用 V2 费曼模式教我 materials/agent/02-进阶篇/Agent的记忆系统.md"

# 高级篇示例
"用 V7 对子互教模式讨论 materials/agent/03-高级篇/多Agent协作与Swarms.md"
```

### 3. 每节课后

AI 会自动更新：
- `teacher/progress.md` — 学习进度
- `teacher/diary.md` — 学习日记（你的视角）
- `teacher/wechat_unread.md` — 三位老师的群聊消息
- `teacher/book_revision_notes.md` — 教材改进建议

---

## 推荐学习节奏

### 入门篇（第 1-4 周）

| 周 | 内容 | 产出 |
|----|------|------|
| 1 | 理解Agent + Anthropic原文 + LilianWeng综述 | 理解 Agent Loop |
| 2 | 写Skill + 拆解一个真实 Skill | 第一个 Skill |
| 3 | 工具与MCP + 上下文工程 | 理解工具设计和 Context |
| 4 | 驾驭工程 | 理解 Harness 架构 |

### 进阶篇（第 5-12 周）

| 周 | 内容 | 产出 |
|----|------|------|
| 5 | Agent循环深度解析 | 能区分四种循环模式 |
| 6 | 规划与执行 + Anthropic工作流 | 能设计 Workflow |
| 7 | Agent的记忆系统 | 理解 Memory 架构 |
| 8 | Agent框架对比 + 选一个框架动手 | 第一个框架 Agent |
| 9 | MCP从原理到生产 | 搭建 MCP Server |
| 10 | Agent评估体系 | eval 套件 |
| 11 | 实战AgentReach拆解 | 读懂真实项目 |
| 12 | 综合实战 | 构建完整 Agent |

### 高级篇（第 13-24 周）

| 周 | 内容 | 产出 |
|----|------|------|
| 13-14 | 自主Agent系统设计 | 长时域 Agent 设计 |
| 15-16 | 多Agent协作 | 多 Agent 系统 |
| 17-18 | Agent安全 | 安全评估报告 |
| 19-20 | 生产化部署 | 生产级 Agent |
| 21-24 | 工作台实战 + 综合项目 | Agent Workbench |

---

## 课程溯源

本课程整合了以下资源：

| 来源 | 贡献 |
|------|------|
| **agent-learning-vault** | 入门篇 5 篇核心概念 + 学习路线框架 |
| **ai-engineering-from-scratch** (Phase 13-16, 19) | 进阶篇和高级篇的完整技术体系 |
| **Agent-Reach** | 实战案例——真实 Agent 工具的架构分析 |
| **Anthropic 官方** | Building Effective Agents 原文 + Workflow 模式 |
| **OpenAI** | Agents SDK 指南 |
| **Lilian Weng** | Agent 综述 |

---

## 相关链接

- [Socratic Verse 主 README](../README.md)
- [9 种学习方法论](../docs/methodology-spectrum.md)
- [教师系统](../teacher/system.md)
