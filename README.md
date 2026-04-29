# Crafts Hub

Claude Code Skills 仓库。存放可复用的 AI 编码智能体技能（Skills）。

## 技能列表

| 技能                                                     | 说明                                                                                                                                                          |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [writing-claude-md](skills/writing-claude-md/SKILL.md) | 通过系统化方法论为项目生成高质量的 CLAUDE.md，确保 LLM 能最高效理解和遵守项目规范。<br/>核心原则： 只写 agent 无法自行推断的内容。工具链已强制执行的不写，能从代码推断的不写，已有文档覆盖的不重复，反直觉约束必须强调。<br/>支持语言： Python/Java/前端/Go/Rust |

## 技能结构约定

```
skills/<skill-name>/
├── SKILL.md                  # 技能入口（frontmatter + 流程定义）
├── <methodology>.md          # 辅助方法论文档
└── <subdirectory>/           # 模板、配置等
```

- `SKILL.md` 前置 frontmatter 的 `name`/`description` 用于技能注册和触发
- 方法论文档供 `SKILL.md` 交叉引用，不独立使用

## 快速开始

```bash
# 克隆仓库
git clone https://github.com/eefnrowe/crafts-hub.git

# 装配 skills 到全局
cp -r crafts-hub/skills ~/.claude/skills

# 复制 skills 到你的项目
cp -r crafts-hub/skills /your/project/.claude/skills


# 使用方式
cd /your/project

claude

/writing-claude-md

```

## 方法论基础

技能设计基于以下研究：

| 来源                                                                                                                 | 核心发现                                 | 应用于             |
|--------------------------------------------------------------------------------------------------------------------|--------------------------------------|-----------------|
| [ETH Zurich arXiv:2602.11988](https://arxiv.org/abs/2602.11988)                                                    | 上下文文件降低成功率、增加成本 20%+；冗余是主因；应只写最小必要要求 | 覆盖分析、最小充分性      |
| [Stanford arXiv:2307.03172](https://arxiv.org/abs/2307.03172)                                                      | LLM 注意力呈 U 型曲线（首位效应 + 近因效应）          | 结构设计：核心规则放首尾    |
| [Anthropic Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 最小充分信息集、从失败模式迭代添加规则                  | 整体方法论、步骤 8 迭代   |
| [CodeIF-Bench arXiv:2503.22688](https://arxiv.org/abs/2503.22688)                                                  | 多轮交互中指令遵循能力评测基准                      | 红线回顾、文件长度控制     |
| [Coding Agents arXiv:2603.20432](https://arxiv.org/abs/2603.20432)                                                 | Coding agent 具备文件结构隐式知识，可作为长上下文处理器   | 引用文件模式、渐进式披露    |
| [AGENTS.md Linux Foundation](https://agents.md/)                                                                   | 跨工具开放标准（60k+ 开源项目采用）                 | 互操作附录、多工具复用     |
| [Cursor Agent Best Practices](https://cursor.com/blog/agent-best-practices)                                        | 参考文件而非复制内容、条件加载                      | 引用文件模式、结构设计     |
| [Gemini CLI 文档](https://geminicli.com/docs/cli/gemini-md/)                                                         | `@file.md` import 机制、层级加载            | 模块化指导、import 语法 |
