# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目身份

Claude Code Skills Hub — 存放 Claude Code 技能（Skills）的仓库。每个技能是一组 Markdown 文档，定义特定任务的方法论、流程和模板。无构建/测试/ lint 命令，纯文档驱动。

## 技能结构

```
skills/<skill-name>/
├── SKILL.md                  # 技能入口：定义流程、步骤、交互方式
├── <methodology>.md          # 辅助方法论文档
└── <subdirectory>/           # 模板、配置文件等
```

- `SKILL.md` 前置 frontmatter 包含 `name` 和 `description`，用于技能注册和触发
- 方法论文档供 `SKILL.md` 交叉引用，不独立使用

## 当前技能

| 技能 | 路径 | 用途 |
|------|------|------|
| writing-claude-md | `skills/writing-claude-md/` | 创建/更新项目 CLAUDE.md，支持 Python/Java/前端/Go/Rust 五种语言 |

## writing-claude-md 架构

7 步流程：模式检测 → 项目探测 → 语言识别 → 工具链覆盖分析 → 交互式规则取舍 → 生成/更新 → 质量验证。

关键设计决策：
- **覆盖分析方法**（`coverage-analyzer.md`）：基于 ETH Zurich 研究，只写 agent 无法自行推断的内容
- **结构设计**：基于 U 形注意力曲线（Stanford "Lost in the Middle"），重要规则放首位和末尾
- **语言模板**（`language-profiles/`）：按语言/框架提供候选规则集，经覆盖分析筛选后写入 CLAUDE.md

## 修改技能时

- 保持 SKILL.md 的 frontmatter `name`/`description` 与目录名一致
- 新增语言模板放入 `language-profiles/`，在 SKILL.md 的语言推断表中补充映射
- 所有方法论变更需在文档中注明来源（学术论文编号或实践经验）
