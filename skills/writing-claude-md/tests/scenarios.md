# 测试场景

修改技能后逐场景跑通，验证渐进式披露和问答流无回归。

## 场景 A：简单 FastAPI 项目（最小输出）

**输入项目：**
```
pyproject.toml (fastapi, uvicorn, pydantic; 无 DI 框架)
ruff 配置（E/W/F/I/S/B）
pytest 配置
README.md: "A FastAPI project"
无分层目录结构、无质量门禁
```

**问答模拟：**
| 步骤 | 问题 | 回答 |
|------|------|------|
| 步骤 0 | CLAUDE.md 存在？ | 不存在 → 创建模式 |
| 步骤 2 Q1 | Python/FastAPI 确认？ | 确认 |

**步骤 3 渐进式披露预期：**

Tier 1 前置检查：
- DI 注入规则 → 无 dishka/di 依赖 → 不适用 ✅
- 全局状态 Final → 适用 → 必须保留 ✅

Tier 2：
- 分层约束 → 无 domain 层 → 不适用 ✅
- async/同步 → 适用 → 等待 Q5
- 异常处理 → 适用 → 等待 Q4
- 日志规范 → 无 structlog → 不适用 ✅
- 测试框架 → 适用 → 等待 Q6
- 安全边界 → 适用 → 等待 Q7

Tier 3：
- dataclass frozen → 无 domain 层 → 不适用 ✅
- 维护同步义务 → 无质量门禁 → 不适用 ✅
- 路径/ID 校验 → 适用但默认不纳入
- API 枚举 → 适用但默认不纳入

**步骤 4 问答模拟：**
| 问题 | 回答 | Tier 2 激活 |
|------|------|------------|
| Q2 分层架构 | 无特定架构 | 分层: 不激活 |
| Q3 DI 方式 | 无DI | DI: 不激活 |
| Q4 异常处理 | 简单try-catch | 异常: 不激活 |
| Q5 同步/异步 | 全同步 | async: 激活 |
| Q6 测试策略 | 最小测试 | 测试: 不激活 |
| Q7 安全校验 | 基本校验 | 安全: 不激活 |
| Q8 特有规则 | 跳过 | Tier 3: 全不纳入 |

**步骤 6a 生成预期：**
- Tier 1: Final + 命令表 + 项目身份 → ~35 行
- Tier 2 激活: async/同步（反直觉）→ ~15 行
- Tier 2 通用池: 无（无映射且无依赖）
- Tier 3: 无
- 红线: ~3 行
- **总计: ~53 行**（简单项目，短是正确的）

**通过判定：**
- [ ] 生成的 CLAUDE.md 不包含 DI 相关内容
- [ ] 包含 async/同步规则（反直觉：FastAPI 全同步）
- [ ] 不包含分层架构规则
- [ ] 不包含 Tier 3 内容
- [ ] ≤ 200 行
- [ ] 红线回顾在末尾

---

## 场景 B：复杂 DDD + DI 项目（接近上限）

**输入项目：**
```
pyproject.toml (fastapi, dishka, pydantic, structlog, uvicorn)
ruff 配置 + ast-grep + import-linter
质量门禁: .quality_gate/ 目录
目录结构: src/domain/, src/application/, src/interface/, src/infrastructure/
README.md: "DDD-based FastAPI service"
```

**步骤 3 渐进式披露预期：**

Tier 1 前置检查：
- DI 注入规则 → 有 dishka → 适用 → 必须保留 ✅
- 全局状态 Final → 适用 → 必须保留 ✅

Tier 2：
- 分层约束 → 有 domain/application 层 → 适用
- async/同步 → 适用
- 异常处理 → 适用
- 日志规范 → 有 structlog → 适用 → 通用池
- 测试框架 → 适用
- 安全边界 → 适用

Tier 3：
- dataclass frozen → 有 domain 层 → 适用
- 维护同步义务 → 有质量门禁 → 适用
- 路径/ID 校验 → 适用
- API 枚举 → 适用

**步骤 4 问答模拟：**
| 问题 | 回答 | Tier 2 激活 |
|------|------|------------|
| Q2 分层架构 | DDD/Clean Architecture | 分层: 激活 |
| Q3 DI 方式 | 自动(框架管理) | DI: 激活 |
| Q4 异常处理 | 全局拦截 | 异常: 激活 |
| Q5 同步/异步 | 全同步 | async: 激活（反直觉） |
| Q6 测试策略 | 测试覆盖要求 | 测试: 激活 |
| Q7 安全校验 | 严格边界校验 | 安全: 激活 |
| Q8 特有规则 | "Provider 同步更新 PROHIBITED_INSTANTIATION" | Tier 3 维护同步: 激活 |

**步骤 6a 生成预期：**
- Tier 1: DI 注入 + Final + 命令表 + 项目身份 → ~50 行
- Tier 2 激活: 分层 + DI + 异常 + async + 测试 + 安全 → ~75 行
- Tier 2 通用池: 日志规范（行数余量判断）→ ~10 行
- Tier 3 (Q8): 维护同步义务 → ~15 行
- 红线: ~3 行
- **总计: ~143-153 行**

**通过判定：**
- [ ] 包含 DI 注入规则 + ✅/❌ 示例
- [ ] 包含分层架构规则
- [ ] 包含 async/同步规则（标注反直觉）
- [ ] 包含维护同步义务（Q8 激活）
- [ ] 不包含 dataclass frozen（Tier 3 未被 Q8 提及）
- [ ] 日志规范是否纳入取决于行数余量
- [ ] ≤ 200 行
- [ ] 红线回顾在末尾

---

## 场景 C：增量更新

**输入项目：**
```
已有 CLAUDE.md（旧版，不含渐进式披露产出的结构）
场景 B 的项目结构
```

**步骤 0：** 检测到已有 CLAUDE.md → AskUserQuestion Q0 → 用户答"增量更新"

**步骤 6b 预期行为：**
- [ ] 输出差异建议表（旧版内容 vs 分析结果）
- [ ] 标记：保留 / 删除 / 新增 / 合并
- [ ] 标注文档冗余（旧版中与 README 重复的内容）
- [ ] 用户确认后合并

---

## 场景 D：触发精确性测试

**应触发的查询（正例）：**
1. "帮我创建这个项目的 CLAUDE.md"
2. "项目需要一份 AI 编码规范文件"
3. "生成 CLAUDE.md"
4. "更新项目指令文件"
5. "这个项目需要 CLAUDE.md 吗"
6. "帮我写 AGENTS.md"
7. "优化现有的 CLAUDE.md"
8. "重构项目级 AI 指令"
9. "给项目加上编码规范文档给 AI 用"
10. "初始化项目规范文件"

**不应触发的近似查询（反例）：**
1. "帮我写 README"（README ≠ CLAUDE.md）
2. "添加代码注释"（注释 ≠ 指令文件）
3. "创建 .editorconfig"（editorconfig ≠ AI 指令）
4. "写 API 文档"（API 文档 ≠ 编码规范）
5. "配置 ESLint"（linter 配置 ≠ AI 指令）
6. "帮我写 CONTRIBUTING.md"（贡献指南 ≠ AI 指令文件）
7. "给函数加 type hints"（类型注解 ≠ 项目规范）
8. "整理项目目录结构"（目录整理 ≠ 规范文件）

**通过判定：**
- [ ] 正例触发率 ≥ 90%（10 条中 ≥ 9 条触发）
- [ ] 反例误触发率 ≤ 10%（8 条中 ≤ 1 条误触发）
