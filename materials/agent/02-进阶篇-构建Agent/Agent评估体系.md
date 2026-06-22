---
title: Agent评估体系
created: 2026-06-17
updated: 2026-06-17
type: concept
order: 7
level: 进阶
status: 未开始
tags: [agent, intermediate, benchmark, evaluation, swebench, gaia]
confidence: high
teaching_method: V1-苏格拉底
est_hours: 0.75
contested: false
contradictions: []
---

# Agent 评估体系

> 你造了一个 Agent。你说它"很好"。好是主观的——你的"好"和用户的"好"可能完全不同。这一篇教你用 2026 年工业标准的评估体系来量化 Agent 的能力。

---

## 评估的三个层次

Anthropic 2026 年的评估指南：

```
层次 1：静态基准测试 → 跨模型对比、回归检测
层次 2：自定义离线评估 → 你的场景、你的数据、你的标准
层次 3：在线生产监控 → 真实用户、真实数据、真实问题
```

每一层解决不同的问题。仅靠 SWE-bench 分数判断你的 Agent 能不能上线——就像仅靠百米成绩判断一个人能不能当消防员。

> 🤔 **思考题**：为什么一个在 SWE-bench 上得分很高的 Agent，在你的代码库里可能表现很差？

---

## 三大基准测试

### SWE-bench：修 Bug 能力

**测什么**：给 Agent 一个 GitHub 仓库（修复前的版本）和一个自然语言的 issue 描述。Agent 生成 patch。自动跑测试——通过就得分。

**关键版本**：
- **SWE-bench 完整版**：2,294 个真实 GitHub issue，来自 12 个流行 Python 库
- **SWE-bench Verified**（OpenAI, 2024）：人工筛选的 500 题子集，去掉了模糊 issue 和不可靠测试

**你必须知道的——数据污染**：94% 的 SWE-bench issue 在大多数模型训练截止日期之前。SWE-bench+ 审计发现 32.67% 的成功 patch 在 issue 文本中泄露了解决方案，31.08% 因测试覆盖弱而可疑。

**使用建议**：永远报告 SWE-bench Verified 分数，附带污染声明。**不要只报一个 SWE-bench 数字**。

### GAIA：通用工具使用能力

**测什么**：466 个问题（300 保留在私有排行榜），测试推理、多模态、网页搜索、工具链组合。三个难度级别——Level 3 需要跨模态的长工具链。

**核心理念**："对人类来说概念上很简单，对 AI 来说很难"。

**例子**：
- Level 1："2024 年奥斯卡最佳影片的导演出生在哪座城市？"
- Level 3："找到这篇论文的 Figure 3，提取其中的数据，计算平均值，格式化输出"

### AgentBench：多环境推理

**测什么**：8 个环境——代码（Bash, 数据库, 知识图谱）、游戏（ALFWorld）、网页（WebShop, Mind2Web）、开放生成。4K-13K 轮次。

**核心发现**：开源 LLM 在长期推理、决策和指令遵循上仍然落后。

> 🤔 **思考题**：GAIA 说"对人类简单，对 AI 难"。SWE-bench 说"对人类程序员也需要时间，对 AI 也难"。这两个设计哲学有什么本质不同？

---

## 自定义离线评估：你的三件套

基准测试回答"这个模型怎么样"，不能回答"这个 Agent 在我的业务里怎么样"。

### 1. LLM-as-Judge（用 LLM 评判 LLM）

```
输入：用户问题 + Agent 的完整轨迹
Judge LLM 评估：准确性 / 完整性 / 效率 / 安全性
```

**关键**：Judge LLM 也会幻觉。用 CRITIC 模式——Judge 的判断需要外部依据支撑。比如"回答不准确，因为查到的数据是 2023 年的而问题问的是 2024 年"——"查到的数据是 2023 年"这个事实是可验证的。

### 2. Execution-Based（执行验证）

```
Agent 写了一段代码 → 用测试套件跑 → 全过了就通过
Agent 操作了网页 → 检查最终页面状态是否符合预期
```

这就是 SWE-bench 的方法——**用测试作为评估器**。这是最可靠的评估方式，前提是你的测试覆盖了正确的行为。

### 3. Trajectory-Based（轨迹对比）

```
Agent 的实际操作序列 ←→ 黄金操作序列
"第 3 步应该查文档，但 Agent 直接改了代码"
"第 7 步应该确认用户意图，但 Agent 假设了"
```

最细致也最昂贵——适合诊断"Agent 为什么错"而不只是"Agent 对不对"。

---

## Eval-Driven Development：用评估驱动开发

Anthropic 2026 年最核心的工程建议：

> **从简单 prompt 开始，用全面的评估去优化，只在不加 Agent 系统就过不了评估的时候才加。**

这不是一个可选步骤——这是 Agent 开发的**外循环**。

### 五个铁律

1. **Eval 和代码放在一起**。同一个 repo。`eval/` 目录。
2. **CI 里跑 eval**。每个 PR。合并门槛："任何评估不比 main 差 5% 以上"。
3. **每个 Guardrail 都对应一个 eval case**。"不允许给用户退款超过 1000 元"→ 建 5 个测试用例试图突破这个限制。
4. **每个生产事故都变成一个 eval case**。如果你不能在生产事故的 eval 里复现它，你就防止不了它再次发生。
5. **存基线**。没有"上次是多少分"的对比，eval 结果毫无意义。

> 🤔 **思考题**：你的 Agent 上周在线上犯了一个错误——它把用户说的"下周三"理解成了"下周二"。你怎么把它变成一个 eval case？

---

## 基准测试不能告诉你什么

- ❌ 真实运营成本（benchmark 不算 token 账）
- ❌ 对抗环境下的安全行为
- ❌ 你的特定领域的表现
- ❌ 尾部故障（benchmark 看平均，生产怕最差的 1%）

**完整报告格式**（参考 Anthropic）：

```
Model: Claude-Fable-5
SWE-bench Verified: 72.3% (污染声明：Verified 子集低风险)
GAIA Level 3: 68.1% (私有排行榜)
Eval 套件: 89.7% pass (187/209 用例)
成本 P50/P95: $0.12 / $1.87 per task
步数 P50/P95: 5 / 38 steps
已知风险: 高并发下工具超时率 3.2%
```

---

## 延伸阅读

- [[Agent循环深度解析]] — Reflexion 的评估器就是 eval 的一种
- [[Agent安全与对齐]] — 高级篇：安全评估的专项方法
- [[Agent工作台实战]] — 高级篇：构建你自己的 eval harness

## 相关

- [[理解Agent]] — 入门篇：Agent 的"成功"怎么定义
- [[工具与MCP]] — 入门篇：工具调用是评估的关键观测点
