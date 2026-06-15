# Socratic Verse — README

> **让 AI 追问你，而不是你问 AI。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 这是什么

Socratic Verse 是一个**开放的学习系统**。核心就是一句话：**AI 不回答你，AI 追问你。**

你把要学的资料给它（markdown/pdf/txt），它不给你讲课——它用一个接一个的问题引导你**自己推理出所有知识**。

2400年的认知科学证明：**提取一次，抵读三遍。** 苏格拉底在雅典街头追问路人，Socratic Verse 在终端里追问你。不同的是——苏格拉底是稀缺资源，AI 是零边际成本。

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
AI: 这篇讲的是梯度下降。在开始之前——你觉得"找最低点"这件事，为什么不能直接用解方程的办法？
你: ...因为有些函数太复杂了？
AI: 复杂的函数为什么不能解方程？能给我一个具体的例子吗？
你: ...高维的？多维的没法直接解。
AI: "高维"在这里意味着什么？它让解方程变得不可能的根本原因是什么？
（20分钟后）
AI: 你刚才自己推理出了以下概念：凸优化、局部极小值、学习率、维度灾难。哪一个你最有成就感？
```

---

## 支持的 8 种主动学习方法（V1-V8）

> 完整的方法论体系包含 9 种方法（含"重读"作为 baseline），详见 [9种学习方法论体系](docs/methodology-spectrum.md)。Prompt 系统提供 V1-V8 共 8 种**主动**学习模式。

| 版本 | 方法 | 一句话 | 适合场景 |
|:---:|------|------|------|
| V1 | 苏格拉底研讨法 | AI 全程追问，不给答案 | 深度学习（默认） |
| V2 | 费曼学习法 | AI 扮不懂的高中生，逼你用小孩话讲 | 检验理解深度 |
| V3 | Anki 推送 | AI 每天推送推理型卡片 | 碎片时间巩固 |
| V4 | 随机骰子 | 掷骰子抽题，消除开始门槛 | 不知道学啥 |
| V5 | 合书复述 | AI 引导你回忆刚读的内容 | 刚读完一本书 |
| V6 | 交错魔鬼 | AI 混排不同学科的问题 | 找概念边界 |
| V7 | 对子互教 | AI 扮你的同学，互相教学质疑 | 社交学习 |
| V8 | 自适应 | AI 根据你的状态自动选方法 | 不想思考"怎么学" |

在启动时告诉 AI 版本号即可：`请用 V4 随机骰子模式。`

---

## 三位 AI 教师

| 老师 | 风格 | 一句话 |
|------|------|------|
| **三月七** | 元气活泼 | "对对对！就是这个意思！那如果…" |
| **甘雨** | 温柔数学 | "你介不介意我换个方式问…" |
| **刻晴** | 严格高效 | "不对，再想想。这个你已经会了。" |

---

## 更多

- 📖 [知乎文章：我发现了人类第一个无痛高强度提取式学习系统](docs/zhihu-article-v2.md)
- 🧠 [9种学习方法论体系](docs/methodology-spectrum.md)
- 📝 [产品需求文档 (PRD)](specs/prd.md)
- 💰 [商业企划书](docs/business-plan.md)
- 📢 [推广方案](docs/marketing-plan.md)
- 🎭 [综合 Prompt 系统](prompts/system-prompts.md)
- 🏗️ [技术架构](CLAUDE.md)

---

## 为什么叫 Socratic Verse

**Socratic** = 苏格拉底教学法。全程提问，不给答案。
**Verse** = 宇宙、诗篇、多元版本。不只一种方法，不只一种模式。

这个名字想表达的是：**在追问构成的宇宙里，每个人都有自己的学法。**

---

## 路线图

- [x] Phase 1: 方法论体系 + 知乎文章 + Prompt 系统
- [ ] Phase 2: Web 文字客户端
- [ ] Phase 3: 语音模式
- [ ] Phase 4: 捏脸 + 知识游戏生成

---

## 贡献

欢迎贡献教材、角色、Prompt 版本、游戏模板。见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

MIT License
