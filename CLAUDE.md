# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> 技能仓库（Skills Hub）— 纯文档驱动的 Claude Code 技能集合。每个技能 = 一组 Markdown 文档，定义特定任务的方法论、流程和模板。无构建/测试命令。

## 技能开发流程

**新建或重构技能时，先调用 `writing-skills` 技能处理格式、frontmatter、部署验证等通用工作。** 本文件仅定义项目特有的约定和质量标准。

## 目录结构约定

```
skills/<skill-name>/
├── SKILL.md                  # 必须：技能入口
├── <reference>.md            # 可选：方法论、决策树等辅助文件
└── <subdirectory>/           # 可选：参考资料、模板等
```

- 辅助文件通过 SKILL.md 内相对路径交叉引用，不独立使用
- 子目录按职责命名（如 `references/`、`templates/`）
- 根目录 README.md 是仓库说明，不属于任何技能

## 本项目质量标准

| 检查项 | 要求 | 说明 |
|--------|------|------|
| SKILL.md 行数 | ≤ 500 行 | 超出则将辅助内容移入 `.md` 文件 |
| frontmatter 字段 | 仅 `name` + `description` | 兼容 Claude Code / Cursor / Augment / Codex |
| 内容格式 | 纯 Markdown | 避免工具特有语法（如 YAML 扩展字段） |
| 方法论来源 | 必须注明 | 学术论文编号（arXiv:XXXX）或实践依据 |
| 交叉引用 | 相对路径 | 新增/移动/删除辅助文件后必须同步更新 SKILL.md 引用 |
| 最小充分性 | 每行检验 | "删掉会导致 agent 犯错吗？"不会则删 |

## 辅助文件管理

### 拆分判定

| 信号 | 处理 |
|------|------|
| SKILL.md > 500 行 | 方法论/模板/引用移入 `.md` 文件，SKILL.md 保留流程定义 + 交叉引用 |
| 单个辅助文件 > 300 行 | 按职责拆分（方法论 / 模板 / 框架规则） |
| 技能覆盖多个独立场景 | 拆分为独立技能 |

### 合并判定

两个技能同时满足时考虑合并：触发条件高度重叠 + 流程共享 > 50% + 合并后 description 仍精确。

### 交叉引用维护

- 新增文件 → SKILL.md 交叉引用段添加路径
- 移动/重命名 → 更新所有引用该文件的 SKILL.md
- 删除文件 → 确认无悬空引用后删除

---

**红线：** 先调用 writing-skills · frontmatter 仅 name + description · 方法论变更必须注明来源 · 交叉引用变更后必须同步
