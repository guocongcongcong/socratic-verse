# Socratic Verse 使用指南

> 让 AI 追问你，而不是你问 AI。

---

## 这是什么

Socratic Verse 是一个**开放学习系统**。你把学习资料丢进去（markdown / pdf / txt），AI 不给你讲课——它用一个接一个的问题引导你**自己推理出所有知识**。

2400 年的认知科学证明：**提取一次，抵读三遍。**（Karpicke & Roediger, *Science*, 2008）

---

## 你需要什么

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)（推荐）或任何支持自定义 Prompt 的 AI 助手

---

## 60 秒快速开始

```bash
# 1. 克隆仓库
git clone git@github.com:guocongcongcong/socratic-verse.git
cd socratic-verse

# 2. 把你要学的教材放进 materials/
cp ~/你的教材.md materials/

# 3. 启动 Claude Code
claude
```

然后在 Claude Code 中说：

```
请先读 teacher/system.md，然后用 V7 苏格拉底研讨法教我 materials/里的 <你的教材名>。
```

---

## 支持的学习方法（V1-V8）

| 版本 | 方法 | 一句话 | 适合什么时候 |
|:---:|------|------|------|
| V1 | Cornell 笔记法 | AI 帮你生成 Cornell 格式笔记，你遮住右侧自己复述 | 刚开始建立提取习惯 |
| V2 | 间隔重复（Anki 式） | AI 每天推送推理型卡片 | 长期记忆、碎片时间 |
| V3 | 随机骰子 | 掷骰子抽题，消除"要不要开始"的决策成本 | 不知道今天学什么 |
| V4 | 合书复述 | AI 引导你回忆刚读完的内容 | 刚读完一章 |
| V5 | 费曼学习法 | AI 扮不懂的高中生，逼你用小孩话讲 | 检验理解深度 |
| V6 | 交错魔鬼 | AI 混排不同学科的问题 | 找概念边界 |
| V7 | 苏格拉底研讨法 | AI 全程追问，不给答案 | 深度学习（默认） |
| V8 | 对子互教 | AI 扮同学，互相教学质疑 | 想要社交感的独学 |

在 Claude Code 中告诉 AI 版本号即可：

```
请用 V4 合书复述模式。我刚读完 materials/机器学习.md 第三章。
```

---

## 三位 AI 教师

| 老师 | 风格 | 适合谁 |
|------|------|--------|
| **三月七** | 元气活泼，"对对对！就是这个意思！" | 需要鼓励的人 |
| **甘雨** | 温柔数学少女，步步引导 | 需要耐心引导的人 |
| **刻晴** | 严格高效，直言不讳 | 需要鞭策的人 |

指定老师：

```
请用 V7 苏格拉底模式，老师用刻晴。
```

---

## 高级用法

### 自定义老师

在 `teacher/` 下创建新的 `.md` 文件，仿照已有格式写人设，然后在 Prompt 中指定即可。

### 贡献教材

将 markdown 文件放入 `materials/`，提交 Pull Request。格式不限，推荐带章节标题。

### 一键引文模式

在 Prompt 末尾加上：

```
每轮追问后，自动列出你引用的原文段落和引用理由。
```

AI 会额外输出：
- **引用段落**：你刚才推理中用到的原文
- **引用理由**：为什么这个段落支持你的结论

---

## 常见问题

**Q: 我没有 Claude Code 怎么办？**
A: 把 `teacher/system.md` 和 `prompts/` 里的 Prompt 复制到 ChatGPT / Hermes Agent 里，手动指定教材内容。效果类似。

**Q: 支持什么格式的教材？**
A: 纯文本（.md / .txt）效果最好。PDF 需要先转文字。

**Q: 我能用本地模型吗？**
A: 可以。用 Ollama 跑本地模型，Prompt 系统是模型无关的。推荐用推理能力强的模型。

**Q: 语音模式什么时候有？**
A: Phase 3，预计 2026 Q4。会支持边走边学。

---

MIT License · [GitHub](https://github.com/guocongcongcong/socratic-verse)
