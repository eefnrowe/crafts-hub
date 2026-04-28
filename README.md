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

- ETH Zurich ICML 2026（arXiv:2602.11988）— 上下文文件有效性评估
- Stanford "Lost in the Middle"（arXiv:2307.03172）— LLM 注意力 U 型曲线
- Anthropic Context Engineering — 最小充分信息集
- CodeIF-Bench（arXiv:2503.22688）— 多轮交互指令遵循
- AGENTS.md Linux Foundation 标准 — 跨工具互操作
