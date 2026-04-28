# CLAUDE.md 结构设计指南

> 来源：Stanford "Lost in the Middle"（arXiv:2307.03172）+ Anthropic Context Engineering + CodeIF-Bench（arXiv:2503.22688）+ Cursor Agent Best Practices + AGENTS.md Linux Foundation 标准

## U 形注意力曲线

LLM 对上下文的注意力呈 U 形分布：

```
注意力
  ▲
  │ ████                                    ████
  │ ████                                    ████
  │ ████                                    ████
  │ ████         ████                       ████
  │ ████         ████         ████          ████
  │ ████         ████         ████          ████
  └──────────────────────────────────────────────► 位置
    首位效应         中间低谷         近因效应
   (最高)           (~30%)          (高)
```

**设计含义**：最重要的规则放在文件开头和结尾，次要规则放在中间。

## 推荐结构

```markdown
# 项目名称 规范

> 项目身份（1-2 句话，锚定上下文：这是什么 + 核心技术栈 + 一句话描述核心流程）

## 适用范围（可选）
[仅当规则适用范围有区分时添加]

## 命令
[表格：操作 | 命令 — 最高频引用内容]

## 分层架构 / 核心架构
[分层图 + 层级约束表 — 影响所有代码位置决策]

## 强制规则
[最关键的 2-5 条规则，每条配 ✅/❌ 代码示例]
- 规则 1（最重要的约束）
- 规则 2
- 规则 3
...

## 代码风格
[精简的命名约定和格式规范]

## 测试
[测试框架、模式、命名]

## 质量门禁同步 / 维护义务（如有）
[变更时必须同步的目标表]

## 配置层级（如有）
[配置优先级]

---

**红线：** 规则1 · 规则2 · 规则3 · ...
```

## 行数控制

- **Anthropic 官方建议**：≤ 200 行
- **理想范围**：120-150 行
- **ETH 研究警示**：自动生成文件比没有文件更差——每行都要经得起"删掉它会导致 Claude 犯错吗？"的检验
- **注意力预算**：LLM 注意力随上下文增长而稀释（n² 对关系），每增加一行都消耗这个预算

## 措辞规则

| 场景 | 用词 | 示例 |
|------|------|------|
| 硬约束（安全/架构） | `禁止` / `必须` / `唯一` / `NEVER` / `MUST` | "禁止 `async def`" |
| 软约束（最佳实践） | `推荐` / `建议` / `优先` / `recommended` | "推荐构造器注入" |
| 反直觉约束 | `必须保留并强调` + 解释为什么 | "全同步架构。唯一豁免：app.py lifespan" |
| 代码示例 | `✅` / `❌` 双向约束 | 见下方 |

## 代码示例格式

每个核心强制规则配一个正反例：

````markdown
### 规则名称

[一句话说明规则]

```python
# ✅ 正确做法
@provide
def skill_repository(self) -> SkillRepository:
    return InMemorySkillRepository()

# ❌ 禁止
service = SkillRouterService(...)  # DI-001 违规
```
````

## 引用文件模式

**Cursor 官方建议：** "参考文件而非复制内容"。当一个模式已有代码示例，用引用替代复制：

| 写法 | 效果 |
|------|------|
| `See Button.tsx for canonical component structure` | agent 自行读取文件，不消耗 CLAUDE.md 行数 |
| 复制 200 行 Button.tsx 到 CLAUDE.md | 占用大量行数，且容易过时 |
| `架构分层详见 README.md § Architecture` | 避免与 README 冗余 |

**适用场景：**
- 项目有 canonical example 文件（如 `examples/` 目录）
- 规则在其他文档中已有详细说明
- 代码模式复杂但已有标准实现

## 模块化与 import 机制

当 CLAUDE.md 超过 150 行或项目为 monorepo 时，考虑拆分：

### Anthropic Claude Code：`@path` 语法

```markdown
# CLAUDE.md（主文件）
@./docs/conventions.md
@./docs/testing.md
```

- 支持相对路径和绝对路径
- 最大递归深度 5 层
- 子目录的 CLAUDE.md 在读取该目录文件时 JIT 加载

### Google Gemini CLI：`@file.md` 语法

```markdown
# GEMINI.md（主文件）
@./components/instructions.md
@../shared/style-guide.md
```

### monorepo 嵌套文件策略

| 场景 | 做法 |
|------|------|
| Monorepo（多服务） | 根目录 CLAUDE.md 写共享规则，各服务目录写特有规则 |
| 单体项目（>150 行） | import 拆分：主文件写核心规则，`@./docs/` 引用详细规范 |
| 微前端 | 每个子应用目录独立 CLAUDE.md，根目录写跨应用共享规则 |

## 条件加载模式

部分工具支持按需加载规则，减少每次交互的上下文消耗：

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| `always_apply` | 每次对话都加载 | 核心架构规则、红线规则 |
| `agent_requested` | Agent 根据描述自动判断相关性 | 特定框架规则、特定模块规范 |
| 目录级 JIT | 读取该目录文件时才加载 | 子目录特有规则 |

**Cursor/Augment 支持**：在 frontmatter 中配置 `type` 和 `description`。

**建议**：CLAUDE.md 核心文件始终使用 always_apply。条件加载仅用于拆分后的辅助文件。

## 多工具互操作

主流 AI 编码工具的指令文件正在趋同：

| 工具 | 文件名 | 发现机制 | import 语法 |
|------|--------|---------|------------|
| Claude Code | `CLAUDE.md` | 向上递归 + JIT | `@path` |
| Gemini CLI | `GEMINI.md` | 全局/工作区/JIT | `@file.md` |
| OpenAI Codex | `AGENTS.md` | 根到 cwd 拼接 | 无 |
| Cursor | `.cursor/rules/*.md` | 文件级 + Auto | 无 |
| Augment | `CLAUDE.md` 或 `AGENTS.md` | 向上递归 | 无 |

**跨工具复用建议：**

1. **以 AGENTS.md 为基础文件**：已由 Linux Foundation 标准化，被 OpenAI Codex、Google Jules、Cursor、Augment 支持
2. **根目录 symlink**：`CLAUDE.md → AGENTS.md`，让 Claude Code 和其他工具共用一份
3. **Gemini CLI 配置**：在 `settings.json` 中设置 `"context": { "fileName": ["AGENTS.md", "CLAUDE.md"] }`
4. **内容兼容性**：使用纯 Markdown，避免工具特有语法（如 YAML frontmatter 仅在 Cursor/Augment 中有效）

## 指令遗忘缓解

> 来源：CodeIF-Bench（arXiv:2503.22688）— 对话越长，指令遵循能力越差

**问题**：多轮交互中，agent 可能遗忘之前遵循的规则，尤其是中间区域的规则。

**缓解策略：**

1. **红线回顾**（已有）：文件末尾一句话复述最关键规则
2. **首位-近因双重放置**：对最关键的规则，在文件开头声明一次，在红线回顾中再次出现
3. **控制文件长度**：越短越不容易被遗忘
4. **避免信息堆砌**：ETH 研究证实，不必要的指令让 agent 认为任务更难，运行更多步骤

## 信息密度优化

| 方式 | 替代 | 原因 |
|------|------|------|
| 表格 | 散文段落 | 同信息量 token 更少 |
| 代码示例 | 3 段文字描述 | Anthropic + Cursor 研究证明一个示例 > 三段描述 |
| 项目符号 | 长段落 | 检索更准确 |
| `✅/❌` | 纯文字 | 视觉区分度高 |
| 红线回顾 | 无 | 近因效应强化，一句话复述最关键规则 |
| 引用文件路径 | 复制代码内容 | 节省行数，避免过时 |

## 质量验证清单

生成 CLAUDE.md 后逐项检查：

1. **行数** ≤ 200 行
2. **首位效应**：开头是否有项目身份 + 命令 + 核心架构？
3. **近因效应**：结尾是否有红线回顾？
4. **代码示例**：每个核心强制规则是否有 ✅/❌ 示例？
5. **冗余**：是否有工具链已覆盖但仍写入的规则？
6. **冗余**：是否有 README/docs 已覆盖但仍写入的内容？
7. **最小充分性**：每行通过"删掉会导致 agent 犯错吗？"检验？
8. **互操作性**：如需跨工具使用，是否采用兼容格式？
9. **措辞**：硬约束是否用了绝对语言？
10. **信息密度**：是否有可以表格化的散文段落？
