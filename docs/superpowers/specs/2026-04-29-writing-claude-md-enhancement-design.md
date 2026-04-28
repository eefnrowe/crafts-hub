# writing-claude-md 技能增强设计规格

> 日期：2026-04-29
> 状态：已批准

## 背景

基于对 Anthropic、Google、OpenAI、Cursor、Augment 的官方文档和 6 篇学术论文的研究，评估现有 `writing-claude-md` 技能的质量并补全缺口。

## 设计决策

### 决策 1：文件结构重组（方案 A：拆分职责）

- 根目录 `README.md` 改写为仓库说明
- 结构设计指南移入 `skills/writing-claude-md/structure-guide.md`
- SKILL.md 交叉引用更新

### 决策 2：改动范围（补全缺口+流程优化）

保持 7 步流程基本结构，增强各步骤内容，新增步骤 8（迭代验证）。

## 变更清单

### 1. 文件结构重组

| 操作 | 文件 |
|------|------|
| 移动+增强 | 根目录 `README.md` → `skills/writing-claude-md/structure-guide.md` |
| 新建 | 根目录 `README.md`（仓库说明） |

### 2. SKILL.md 改进

| 改进项 | 位置 | 内容 |
|--------|------|------|
| 现有文档预检 | 步骤 0→1 之间 | 检查 README/docs 内容，标记冗余源 |
| 探测范围扩展 | 步骤 1 | 增加 README/docs、AGENTS.md/GEMINI.md 收集 |
| 冗余维度 | 步骤 3 | 新增"冗余标记"判定：已有文档覆盖的规则不写入 |
| 模块化指导 | 步骤 5 | import 机制、monorepo 嵌套、引用文件模式 |
| 质量验证增强 | 步骤 7 | 最小充分性测试、互操作性检查 |
| 迭代验证 | 新增步骤 8 | 测试→根据失败模式添加→最小化迭代 |
| 互操作附录 | 文末 | AGENTS.md/GEMINI.md 跨工具复用指导 |
| 引用修正 | 交叉引用 | `structure-guide.md` 路径更新 |

### 3. coverage-analyzer.md 增强

- 文档冗余判定维度
- ETH 量化证据引用
- 最小充分性检验分支

### 4. structure-guide.md 增强（从 README.md 移入后）

- 引用文件模式
- import 机制指导
- monorepo 嵌套文件策略
- 条件加载模式说明
- 多工具互操作建议
- 指令遗忘缓解

### 5. 语言模板渐进式披露

为 5 个语言模板增加 Tier 分层：
- Tier 1：核心规则（几乎所有项目）
- Tier 2：推荐规则（取决于项目风格）
- Tier 3：边缘场景（特定条件）

## 文献支撑

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

## 不在范围内

- 语言模板内容大幅修改（覆盖面足够）
- 新增语言支持
- 自动化测试
