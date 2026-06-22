# Socratic Verse — README

> **让 AI 追问你，而不是你问 AI。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 这是什么

Socratic Verse 是一个**开放的学习系统**。核心就一句话：**AI 不回答你，AI 追问你。**

你把要学的资料给它（markdown/pdf/txt），它不给你讲课——它用一个接一个的问题引导你**自己推理出所有知识**。

2400 年的认知科学证明：**提取一次，抵读三遍。** 苏格拉底在雅典街头追问路人，Socratic Verse 在终端里追问你。不同的是——苏格拉底是稀缺资源，AI 是零边际成本。

---

## 10分钟快速上手

### 前提
- Claude Code 或 Hermes Agent 已安装
- 有一个你想学的东西（一本 markdown 书、一篇论文、一个教程）

### 步骤

```bash
# 1. 克隆
git clone git@github.com:guocongcongcong/socratic-verse.git
cd socratic-verse

# 2. 把教材放进 materials/
cp ~/path/to/your-textbook.md materials/

# 3. 启动
claude  # 或 hermes

# 4. 对 AI 说：
"请先读 teacher/system.md，然后用 V1 标准苏格拉底模式教我 materials/ 里的 <你的教材>。"
```

### 第一节课你会体验什么

```
AI: 这篇讲的是梯度下降。在开始之前——你觉得"找最低点"这件事，为什么不能用解方程的办法？
你: ...因为有些函数太复杂了？
AI: 复杂的函数为什么不能解方程？能给我一个具体的例子吗？
你: ...高维的？多维的没法直接解。
AI: "高维"在这里意味着什么？它让解方程变得不可能的根本原因是什么？
（20分钟后）
AI: 你刚才自己推理出了以下概念：凸优化、局部极小值、学习率、维度灾难。哪一个你最有成就感？
```

---

## 8 种学习方法（V1-V8）+ 自动调度

> 完整方法论体系见 [9种学习方法论](docs/methodology-spectrum.md)。学习方法调度表见 `teacher/learning_schedule.md`。

| 版本 | 方法 | 一句话 | 何时触发 |
|:---:|------|------|------|
| V1 | 苏格拉底 | AI 全程追问，不给答案 | 第一次学新概念（默认） |
| V2 | 费曼 | AI 扮高中生，逼你用大白话讲 | V1 课后 24h 检验 |
| V3 | Anki 推送 | 每天推送推理卡片 | 每天 08:00 自动 |
| V4 | 随机骰子 | 掷骰子抽题 | 每周日 20:00 自动 |
| V5 | 合书复述 | 合上书，复述刚学的内容 | V1 课后 1h 内自动 |
| V6 | 交错魔鬼 | 混排不同学科出题 | 多科全学完后手动触发 |
| V7 | 对子互教 | AI 扮同学互相教学 | V1 课后 72h 自动 |
| V8 | 自适应 | AI 根据状态选方法 | V1-V7 全验证后作为默认 |

**关键原则**：V1 是种子，V5 是收割，V2 是检验，V7 是巩固，V3 是保鲜，V4 是惊喜，V6 是体检，V8 是管家。

---

## 四位 AI 教师

| 老师 | 风格 | 学科偏好 | 一句话 |
|------|------|---------|------|
| **三月七** | 元气活泼 | 编程/工程/实战 | "对对对！就是这个意思！" |
| **甘雨** | 温柔数学 | 概念入门/逻辑/协议 | "你介不介意我换个方式问…" |
| **刻晴** | 严格高效 | 架构/方法论/综述 | "不对，再想想。" |
| **芙宁娜** | 哲学审问 | 元认知/思辨/概念辨析 | "我们为什么会认为这是对的？" |

---

## 内置课程：AI Agent 开发

项目自带一套完整的 AI Agent 学习课程（22 篇教材，三层递进）：

```
入门篇（理解Agent → 写Skill → 工具与MCP → 上下文工程 → 驾驭工程）
进阶篇（Agent循环深度解析 → 规划与执行 → 记忆系统 → 框架对比 → ...）
高级篇（自主Agent系统设计 → 多Agent协作 → 安全与对齐 → 生产化部署）
```

教材关联图谱见 `teacher/materials-map.md`（由 Hermes + Claude Code 交叉验证产出）。

---

## 更多

- 📖 [知乎文章：我发现了人类第一个无痛高强度提取式学习系统](docs/zhihu-article-v2.md)
- 🧠 [9种学习方法论体系](docs/methodology-spectrum.md)
- 🎭 [综合 Prompt 系统](prompts/system-prompts.md)
- 📊 [教材关联图谱](teacher/materials-map.md)
- 📅 [学习方法调度表](teacher/learning_schedule.md)
- 📝 [产品需求文档 (PRD)](specs/prd.md)
- 💰 [商业企划书](docs/business-plan.md)
- 🏗️ [技术架构](CLAUDE.md)

---

## 为什么叫 Socratic Verse

**Socratic** = 苏格拉底教学法。全程提问，不给答案。
**Verse** = 宇宙、诗篇、多元版本。不只一种方法，不只一种模式。

这个名字想表达的是：**在追问构成的宇宙里，每个人都有自己的学法。**

---

## 路线图

- [x] Phase 1: 方法论体系 + 知乎文章 + Prompt 系统 + V1-V8 自动调度
- [ ] Phase 2: Web 文字客户端
- [ ] Phase 3: 语音模式
- [ ] Phase 4: 捏脸 + 知识游戏生成

---

## 贡献

欢迎贡献教材、角色、Prompt 版本、游戏模板。见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

MIT License
