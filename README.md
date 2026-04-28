# Crafts Hub

Claude Code Skills 仓库。存放可复用的 AI 编码智能体技能（Skills）。

## 技能列表

| 技能 | 说明 |
|------|------|
| [writing-claude-md](skills/writing-claude-md/SKILL.md) | 系统化生成高质量项目 CLAUDE.md，支持 Python/Java/前端/Go/Rust 五种语言 |

## 技能结构约定

```
skills/<skill-name>/
├── SKILL.md                  # 技能入口（frontmatter + 流程定义）
├── <methodology>.md          # 辅助方法论文档
└── <subdirectory>/           # 模板、配置等
```

- `SKILL.md` 前置 frontmatter 的 `name`/`description` 用于技能注册和触发
- 方法论文档供 `SKILL.md` 交叉引用，不独立使用

## 方法论基础

技能设计基于以下研究：

| 来源 | 引用 | 应用于 |
|------|------|--------|
| ETH Zurich arXiv:2602.11988 | LLM 生成文件降低成功率 3%，增加成本 20%+ | 覆盖分析、最小充分性 |
| Stanford arXiv:2307.03172 | U 型注意力曲线 | 结构设计（已有） |
| Anthropic Context Engineering | 最小充分信息集、注意力预算 | 整体方法论 |
| CodeIF-Bench arXiv:2503.22688 | 多轮交互中指令遗忘 | 红线回顾重申 |
| Coding Agents arXiv:2603.20432 | 文件结构即隐式指令 | 引用文件模式 |
| JetBrains NeurIPS 2025 | 简单上下文管理策略 | 迭代验证循环 |
| AGENTS.md Linux Foundation | 跨工具开放标准 | 互操作附录 |
| Cursor Agent Best Practices | 参考文件而非复制、条件加载 | 结构设计 |
| Gemini CLI 文档 | `@file.md` import 机制 | 模块化指导 |
