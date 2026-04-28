# CLAUDE.md 重构设计规格

> 日期：2026-04-29
> 状态：已批准

## 目标

重构项目 CLAUDE.md，聚焦技能开发/优化/重构三个核心场景，使 Claude Code 能独立完成技能全生命周期工作。

## 设计决策

- **目标受众**：仅面向 Claude Code 实例
- **方案选择**：项目规范型（方案 A）
- **核心原则**：去除所有非技能开发相关内容

## 内容结构

### 首位区域（注意力最高）

1. **技能目录结构约定** — `skills/<name>/` 的文件组织规范
2. **新建技能规范** — SKILL.md 格式（frontmatter + 内容要求）、质量标准
3. **跨平台兼容性** — frontmatter 仅 `name` + `description`，纯 Markdown

### 中间区域

4. **优化技能** — description 优化方法、SKILL.md 内容精简、触发测试
5. **重构技能** — 拆分/合并判定、辅助文件管理、交叉引用维护

### 近因区域

6. **红线回顾** — 关键约束一句话复述

## 文献支撑

| 来源 | 应用于 |
|------|--------|
| Anthropic Claude Code Skills 文档 | SKILL.md 格式规范 |
| agentskills.io 最佳实践 | 触发精确性、description 优化 |
| AGENTS.md Linux Foundation | 跨平台兼容性 |
| Cursor/Augment 规则规范 | 条件加载模式参考 |

## 不在范围内

- 项目身份/仓库描述（由 README.md 覆盖）
- writing-claude-md 技能内部细节（由 SKILL.md 覆盖）
- 贡献流程（开源后由 CONTRIBUTING.md 覆盖）
