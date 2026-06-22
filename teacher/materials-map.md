# 入门篇教材关联图谱

> 生成日期：2026-06-22 | 分析者：Hermes (刻晴) + Claude Code | 交叉验证完成

---

## 一、教材总览

| # | 文件 | order | 类型 | 定位 | 核心贡献 |
|---|------|-------|------|------|---------|
| 0 | 学习路线总览.md | 0 | 导航 | 地图 | 三层递进框架、分层教法、项目驱动 |
| 1 | Anthropic-构建高效Agent.md | 1 | 参考文献 | 权威理念 | Workflow/Agent 二分法、五种工作流、从简原则 |
| 2 | OpenAI-Agents-SDK指南.md | 2 | 参考文献 | 工具参考 | 三原语(Agent/Handoff/Guardrails)、Session、Sandbox |
| 3 | LilianWeng-Agent综述.md | 3 | 参考文献 | 学术全景 | 规划/记忆/工具三组件、ReAct/Reflexion/ToT |
| 4 | **理解Agent.md** | 4 | **核心** | **知识中枢** | Agent Loop、五组件、三分法(Chatbot/Workflow/Agent) |
| 5 | 写Skill.md | 10 | 核心 | 动手第一步 | SOP 概念、五条铁律、交互式 vs 确定性 |
| 6 | 工具与MCP.md | 14 | 核心 | 能力基础 | 四标准、MCP 协议、工具四分类 |
| 7 | 上下文工程.md | 16 | 核心 | 分水岭 | Context 六层模型、三原则、三压缩策略、注入优先级 |
| 8 | 驾驭工程.md | 18 | 核心 | 总收束 | 六层架构、裸 Agent vs Harness、Hermes 全景图 |

---

## 二、依赖关系图

```
                    学习路线总览 (order 0)
                   （导航层，不依赖任何教材）
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    Anthropic         OpenAI SDK      Lilian Weng
   (order 1)          (order 2)       (order 3)
   权威理念            工具参考         学术全景
          │               │               │
          └───────────────┼───────────────┘
                          │
                引用/理论支撑
                          │
                          ▼
                   理解Agent (order 4)
                 （核心锚点，聚合+简化）
                          │
                    "下一步"指向
                          │
                          ▼
                    写Skill (order 10)
                 （动手实践第一步）
                          │
                 "给 Skill 配工具"
                          │
                          ▼
                   工具与MCP (order 14)
                 （能力基础设施）
                          │
               "工具输出占 Context"
                          │
                          ▼
                  上下文工程 (order 16)
                 （视野管理分水岭）
                          │
              "Context 管理的工程化"
                          │
                          ▼
                   驾驭工程 (order 18)
               （知识总装，六层系统视图）
```

### 核心链（强依赖，不可跳读）

```
理解Agent → 写Skill → 工具与MCP → 上下文工程 → 驾驭工程
```

每篇显式引用上一篇为「上一层」、下一篇为「下一层」。

### 参考文献（弱依赖，可后置）

三篇参考文献为理解Agent 提供理论支撑，但不依赖核心链。最佳阅读时机是**写完第一个 Skill 之后反刍**。

---

## 三、概念矛盾（需修正）

### 矛盾 1：Agent vs Workflow 分类不一致

| 教材 | 分类法 |
|------|--------|
| Anthropic (order 1) | **二分法**：Workflow ∪ Agent = Agentic Systems |
| 理解Agent (order 4) | **三分法**：Chatbot ≠ Workflow ≠ Agent |

**严重度**：低 | **建议**：在理解Agent 加注「更严格地说，Workflow 和 Agent 都是 Agentic System 的子类，入门阶段先做简单区分」

### 矛盾 2：ReAct 是通用模式还是特定方法

| 教材 | 处理 |
|------|------|
| 理解Agent (order 4) | Agent Loop = 「所有 Agent 都跑这个循环」 |
| LilianWeng (order 3) | ReAct 是**一种特定 prompt 模板**，与 ToT、Reflexion 并列 |

**严重度**：中 | **建议**：理解Agent Agent Loop 章节加「这是最常见的 ReAct 模式，不是唯一模式——进阶篇会讲更多变体」

### 矛盾 3：记忆分类维度冲突

| 教材 | 分类维度 |
|------|---------|
| LilianWeng | **学术认知**：感觉记忆 / 短期记忆 / 长期记忆 |
| 上下文工程 | **工程实践**：Memory / CLAUDE.md / Skill |
| 理解Agent | **模糊类比**：便利贴 / 视野 |

**严重度**：中 | **建议**：上下文工程加对照表映射学术分类到工程实现

---

## 四、概念重复（需精简）

| # | 主题 | 出现位置 | 严重度 | 建议 |
|---|------|---------|--------|------|
| 1 | Agent Loop 基础定义 | 理解Agent(~80行) + 驾驭工程(~25行) + LilianWeng(~15行) | 低 | 驾驭工程删基础定义，引用理解Agent |
| 2 | 上下文压缩策略 | 上下文工程(详细版) + 驾驭工程(缩略版) | 低 | 驾驭工程第1层直接引用上下文工程 |
| 3 | MCP 概念 | 工具与MCP(协议原理) + OpenAI SDK(SDK集成) | 无 | 两者互补，加交叉引用链接 |
| 4 | 工具设计原则 | 工具与MCP(四标准) + Anthropic(ACI理念) | 无 | 互补，工具与MCP 加一句引用ACI |

---

## 五、建议学习路径

### 路径 A：实践驱动（推荐）
```
理解Agent → 写Skill → 工具与MCP → 上下文工程 → 驾驭工程
→ 然后反刍：Anthropic → LilianWeng → OpenAI SDK
```
适合动手型学习者，2周内建立「能做东西」的正反馈。

### 路径 B：理论驱动
```
理解Agent → Anthropic → 写Skill → LilianWeng → 工具与MCP →
OpenAI SDK → 上下文工程 → 驾驭工程
```
适合需要「确信自己在正确轨道上」的学习者。

### 路径 C：速成型（2.5小时）
```
理解Agent → 写Skill → 上下文工程 → 驾驭工程
```
跳过工具与MCP 和三篇参考文献，建立最小闭环。

### 共同原则
1. **三篇参考文献绝不先读**——最佳时机是写完Skill 之后反刍
2. **写Skill 必须紧接理解Agent**——中间不插入其他材料
3. **驾驭工程必须最后读**——它是「期末考试」，整合所有前置概念

---

## 六、进阶篇教学方法编排（2026-06-22 修正）

| # | 文件 | 教学方法 | 理由 |
|---|------|:---:|------|
| 1 | Agent循环深度解析 | V1-苏格拉底 | 首次接触新循环模式，需追问引导 |
| 2 | 规划与执行模式 | V1-苏格拉底 | 首次接触并行规划算法 |
| 3 | Agent的记忆系统 | V1-苏格拉底 | 大量新术语（MemGPT/virtual memory） |
| 4 | Anthropic工作流模式 | V2-费曼 | 前置知识扎实，适合费曼内化 |
| 5 | Agent框架对比 | V7-对子互教 | 对比类教材最佳方式是互相讲 |
| 6 | MCP从原理到生产 | V1-苏格拉底 | 六原语是新概念，首次接触 |
| 7 | Agent评估体系 | V1-苏格拉底 | SWE-bench/GAIA 新知识体系 |
| 8 | 实战AgentReach拆解 | V5-合书复述 | 案例研究适合合书复述结构 |

**规律**：首次接触新概念→V1，复习深化→V2，对比横评→V7，案例研究→V5。

---

## 七、改动建议优先级

| 优先级 | 改动内容 | 影响 |
|--------|---------|------|
| 🔴 P0 | 将理解Agent 的 order 从 4 改为 1（或其他方案让它在权威原文之前） | 解决当前学习路径反认知规律的问题 |
| 🔴 P0 | 理解Agent Agent Loop 章节加「不是唯一模式」的声明 | 消除对 ReAct 的误解 |
| 🟡 P1 | 上下文工程 加学术记忆分类→工程实现对照表 | 消除记忆分类混淆 |
| 🟡 P1 | 驾驭工程 第1层删压缩描述、引用上下文工程 | 减少 25 行冗余 |
| 🟢 P2 | 理解Agent 加 Agent/Workflow 二分vs三分的注释 | 消除分类混淆 |
| 🟢 P2 | 工具与MCP 加一句引用 Anthropic ACI | 建立跨教材关联 |

---

## 七、关联矩阵（JSON）

```json
{
  "nodes": {
    "学习路线总览": {"order": 0, "type": "导航"},
    "Anthropic-构建高效Agent": {"order": 1, "type": "参考文献"},
    "OpenAI-Agents-SDK指南": {"order": 2, "type": "参考文献"},
    "LilianWeng-Agent综述": {"order": 3, "type": "参考文献"},
    "理解Agent": {"order": 4, "type": "核心教材"},
    "写Skill": {"order": 10, "type": "核心教材"},
    "工具与MCP": {"order": 14, "type": "核心教材"},
    "上下文工程": {"order": 16, "type": "核心教材"},
    "驾驭工程": {"order": 18, "type": "核心教材"}
  },
  "core_chain": ["理解Agent", "写Skill", "工具与MCP", "上下文工程", "驾驭工程"],
  "recommended_order": ["学习路线总览", "理解Agent", "写Skill", "工具与MCP", "上下文工程", "驾驭工程", "Anthropic-构建高效Agent", "LilianWeng-Agent综述", "OpenAI-Agents-SDK指南"]
}
```
