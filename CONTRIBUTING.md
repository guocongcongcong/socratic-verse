# Contributing to Socratic Verse

## 贡献方式

### 贡献教材
将任何学科的 markdown 教材放入 `materials/` 目录。PR 时请在描述中说明教材内容和适用学习阶段。

### 贡献角色
1. 复制 `teacher/march7.md` 作为模板
2. 修改基础信息和性格（保持11个必填字段）
3. 确保角色声音与现有三位角色有明显区分度
4. PR 时附带一次测试对话

### 贡献 Prompt 版本
1. 在 `prompts/` 下新建 `vN-description.md` 
2. 遵循现有 Prompt 结构（角色 + 规则 + 禁止 + 示例）
3. PR 时说明与现有版本的区分度

### 贡献游戏模板
在 `tools/` 下提交 HTML/CSS/JS 小游戏。要求：
- 单文件，零依赖
- 根据输入知识内容动态生成游戏元素
- 有清晰的"学习目标"标注

## 开发
本项目是 markdown 驱动的系统。没有后端，没有数据库，没有构建步骤。
