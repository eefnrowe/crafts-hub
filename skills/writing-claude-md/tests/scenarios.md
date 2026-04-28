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
- 通用类库 → 适用 → 等待 Q5（选项含"无，跳过"和"有，请自行探测"，也可通过 Other 提供路径）
- async/同步 → 适用 → 等待 Q6
- 异常处理 → 适用 → 等待 Q4
- 日志规范 → 无 structlog → 不适用 ✅
- 测试框架 → 适用 → 等待 Q7
- 安全边界 → 适用 → 等待 Q8

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
| Q5 通用类库 | 无，跳过 | 通用类库: 不激活 |
| Q6 同步/异步 | [检测结果] 全同步（确认现状） | async: 激活 |
| Q7 测试策略 | 最小测试 | 测试: 不激活 |
| Q8 安全校验 | 基本校验 | 安全: 不激活 |
| Q9 特有规则 | 跳过 | Tier 3: 全不纳入 |

**步骤 6a 生成预期：**
- 标题 + 项目身份: ~3 行
- 适用范围: ~3 行
- 命令表: ~5 行
- 核心架构规则: Final + async/同步（反直觉）→ ~25 行
- 代码风格（扩展至达标）: ~15 行
- 测试规范（扩展至达标）: ~12 行
- 红线: ~3 行
- **总计: ~66 行核心 + 扩展至 ≥150 行**
- 扩展来源：核心规则增加 ✅/❌ 示例 → 测试模式说明 → 代码风格细化

**通过判定：**
- [ ] 生成的 CLAUDE.md 不包含 DI 相关内容
- [ ] 包含 async/同步规则（反直觉：FastAPI 全同步）
- [ ] 不包含分层架构规则
- [ ] 不包含 Tier 3 内容
- [ ] ≥ 150 行（通过扩展代码示例和测试规范达标）
- [ ] 红线回顾在末尾

---

## 场景 B：复杂 DDD + DI 项目（接近上限）

**输入项目：**
```
pyproject.toml (fastapi, dishka, pydantic, structlog, uvicorn, company-common-utils)
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
- 通用类库 → 有 company-common-utils → 等待 Q5（选"有，请自行探测"触发自动识别，或通过 Other 提供路径触发针对性分析）
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
| Q5 通用类库 | Other: "libs/common-utils（统一异常+日志+认证中间件）" | 通用类库: 激活（触发 Explore Agent 针对性分析） |
| Q6 同步/异步 | [检测结果] 全同步（确认现状） | async: 激活（反直觉） |
| Q7 测试策略 | 测试覆盖要求 | 测试: 激活 |
| Q8 安全校验 | 严格边界校验 | 安全: 激活 |
| Q9 特有规则 | "Provider 同步更新 PROHIBITED_INSTANTIATION" | Tier 3 维护同步: 激活 |

**步骤 6a 生成预期：**
- 标题 + 项目身份: ~3 行
- 适用范围: ~3 行
- 命令表: ~8 行
- 核心架构规则: DI 注入 + Final + 分层 + 异常 + async + 安全 + 通用类库 → ~60 行
- 通用类库依赖: ~8 行
- 代码风格: ~15 行
- 测试规范: ~15 行
- 维护义务 (Q9): ~10 行
- Tier 2 通用池: 日志规范（行数余量判断）→ ~10 行
- 红线: ~3 行
- **总计: ~135 行核心 + 扩展至 ≥150 行**
- 扩展来源：核心规则增加边界情况示例 → 测试模式说明 → 代码风格细化

**通过判定：**
- [ ] 包含 DI 注入规则 + ✅/❌ 示例
- [ ] 包含分层架构规则
- [ ] 包含通用类库依赖表格（由 Q5 用户输入触发 Explore 分析生成）
- [ ] 包含 async/同步规则（标注反直觉）
- [ ] 包含维护同步义务（Q9 激活）
- [ ] 不包含 dataclass frozen（Tier 3 未被 Q9 提及）
- [ ] 日志规范是否纳入取决于行数余量
- [ ] ≥ 150 行
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

---

## 分支覆盖矩阵

所有问答选项与对应测试场景的映射。每个选项至少被一个场景覆盖。

| 问题 | 选项 | 覆盖场景 |
|------|------|---------|
| Q0 | 不存在 → 创建模式 | A, E |
| Q0 | 全量重写 | G |
| Q0 | 增量更新 | C |
| Q1 | 确认 | A, B |
| Q1 | 不对，是其他类型 | F |
| Q2 | [检测结果] | E |
| Q2 | DDD/Clean Architecture | B |
| Q2 | MVC | E |
| Q2 | 简单分层(无严格约束) | G |
| Q2 | 无特定架构 | A |
| Q3 | [检测结果] | E |
| Q3 | 自动(框架管理) | B |
| Q3 | 手动(工厂模式) | G |
| Q3 | 无DI | A |
| Q4 | [检测结果] | E |
| Q4 | 全局拦截(统一错误响应) | B |
| Q4 | 各层独立处理 | G |
| Q4 | 简单try-catch | A |
| Q5 | 无，跳过 | A |
| Q5 | 有，请自行探测 | E |
| Q5 | Other（路径/标识） | B |
| Q6 | [检测结果] | A, B, E |
| Q6 | 全同步 | G |
| Q6 | 混合(无约束) | G |
| Q7 | [检测结果] | E |
| Q7 | TDD优先 | G |
| Q7 | 测试覆盖要求 | B |
| Q7 | 最小测试 | A |
| Q7 | 无特殊要求 | F |
| Q8 | 严格边界校验 | B |
| Q8 | 基本校验 | A, G |
| Q8 | 无特殊要求 | E, F |
| Q9 | 有输入 | B, G |
| Q9 | 跳过 | A, E, F |

---

## 场景 E：[检测结果] 全覆盖验证

**目的：** 验证 Q2/Q3/Q4/Q7 新增的 [检测结果] 选项和 Q5"有，请自行探测"分支。

**输入项目：**
```
pyproject.toml (flask, sqlalchemy, pytest, pytest-cov; 无 DI 框架)
目录结构: app/controllers/, app/models/, app/templates/, app/services/
ruff 配置
.company_utils/ (内部类库: logging_middleware.py, auth_decorator.py)
无全局异常处理
pytest 配置 + conftest.py (仅基础 fixture)
pyproject.toml [tool.coverage.run] branch = true
pyproject.toml [tool.coverage.report] fail_under = 80
```

**预分析预期：**
- Q2 检测：controllers/ + models/ + templates/ → MVC
- Q3 检测：无 dishka/python-inject 等 DI 框架依赖 → 无DI
- Q4 检测：无 @app.errorhandler / 无 BaseExceptionHandler → 简单try-catch
- Q6 检测：无 async/await → 全同步
- Q7 检测：pytest-cov 配置 + fail_under=80 → 覆盖要求

**步骤 4 问答模拟：**
| 问题 | 回答 | Tier 2 激活 |
|------|------|------------|
| Q2 分层架构 | [检测结果] MVC | 分层: 激活（MVC 约束） |
| Q3 DI 方式 | [检测结果] 无DI | DI: 不激活 |
| Q4 异常处理 | [检测结果] 简单try-catch | 异常: 不激活 |
| Q5 通用类库 | 有，请自行探测 | 通用类库: 激活（Explore Agent 自动识别） |
| Q6 同步/异步 | [检测结果] 全同步 | async: 不激活 |
| Q7 测试策略 | [检测结果] 覆盖要求: pytest-cov ≥ 80% | 测试: 激活 |
| Q8 安全校验 | 无特殊要求 | 安全: 不激活 |
| Q9 特有规则 | 跳过 | Tier 3: 不纳入 |

**步骤 6a 生成预期：**
- MVC 分层约束（controllers 路由转发 / models 数据映射 / services 业务逻辑）
- 通用类库依赖表（Explore Agent 自动探测 .company_utils/ → logging_middleware + auth_decorator）
- 测试覆盖规则（pytest-cov ≥ 80%）
- 不含 DI 规则、全局异常规则、async 规则、安全边界规则

**通过判定：**
- [ ] [检测结果] 选项出现在 Q2/Q3/Q4/Q7 的第一个位置
- [ ] [检测结果] 描述与项目实际状态一致（MVC / 无DI / 简单try-catch / 覆盖要求）
- [ ] 选 [检测结果] 后，以检测到的状态为基准写入对应规则
- [ ] MVC 分层约束规则写入（✅ controller 仅路由 / ❌ controller 含业务逻辑）
- [ ] 通用类库依赖来自 Explore Agent 自动探测（非用户手动输入路径）
- [ ] 测试覆盖规则包含 fail_under 阈值
- [ ] 不含 DI、全局异常、async、安全边界规则
- [ ] ≥ 150 行

---

## 场景 F：最小配置 + 语言纠正

**目的：** 覆盖 Q1 语言纠正、Q7/Q8 无特殊要求。

**输入项目：**
```
build.gradle.kts (Kotlin Ktor 项目；检测为 Java)
src/main/kotlin/com/example/
src/test/kotlin/com/example/
application.conf (Ktor 配置)
无分层目录、无 DI 框架、无质量门禁
已有 CLAUDE.md（旧版，约 80 行）
```

**步骤 0：** 检测到已有 CLAUDE.md → Q0 → 用户答"全量重写"

**步骤 2 预期：**
- 初始检测 build.gradle.kts → 推断 Java
- Q1 用户纠正 → Kotlin
- 无 references/kotlin.md → 按 JVM 语言通用规则 + Kotlin 特性处理

**步骤 4 问答模拟：**
| 问题 | 回答 | Tier 2 激活 |
|------|------|------------|
| Q2 分层架构 | 无特定架构 | 分层: 不激活 |
| Q3 DI 方式 | 无DI | DI: 不激活 |
| Q4 异常处理 | 简单try-catch | 异常: 不激活 |
| Q5 通用类库 | 无，跳过 | 通用类库: 不激活 |
| Q6 同步/异步 | 全同步（显式选择） | async: 不激活 |
| Q7 测试策略 | 无特殊要求 | 测试: 不激活 |
| Q8 安全校验 | 无特殊要求 | 安全: 不激活 |
| Q9 特有规则 | 跳过 | Tier 3: 不纳入 |

**步骤 6a 生成预期：**
- 仅 Tier 1 必选规则 + 命令表 + 红线
- Kotlin 特有规则从代码结构推断（data class、suspend 函数等）
- 大量扩展 ✅/❌ 代码示例至 ≥ 150 行

**通过判定：**
- [ ] Q1 语言纠正后加载正确的语言特性（Kotlin 而非 Java）
- [ ] 无 references/kotlin.md 时不报错，改为从代码推断
- [ ] Q6 显式选"全同步"不生成 async 规则（行为与 [检测结果] 全同步一致）
- [ ] Q7"无特殊要求"不生成测试策略规则
- [ ] Q8"无特殊要求"不生成安全边界规则
- [ ] 通过代码示例扩展达到 ≥ 150 行
- [ ] 红线回顾在末尾

---

## 场景 G：手动选项未覆盖分支

**目的：** 覆盖 MVC、简单分层、手动DI、各层独立异常、混合异步、TDD、Q0 全量重写。

**输入项目：**
```
pom.xml (无 Spring，纯 Servlet + JDBI)
目录结构: src/main/java/com/example/api/, service/, dao/, model/
checkstyle.xml
JUnit 5 + 测试文件结构完整（test/ 目录结构与 src/ 对称，TDD 特征）
Factory 类: com.example.service.ServiceFactory
各层独立 try-catch（DAO → RuntimeException，Service → BusinessException）
Servlet AsyncContext 异步 + 同步 doGet/doPost 混合
```

**步骤 4 问答模拟：**
| 问题 | 回答 | Tier 2 激活 |
|------|------|------------|
| Q2 分层架构 | 简单分层(无严格约束) | 分层: 激活（轻量分层） |
| Q3 DI 方式 | 手动(工厂模式) | DI: 激活（工厂模式约束） |
| Q4 异常处理 | 各层独立处理 | 异常: 激活（各层异常边界） |
| Q5 通用类库 | 无，跳过 | 通用类库: 不激活 |
| Q6 同步/异步 | 混合(无约束) | async: 不激活 |
| Q7 测试策略 | TDD优先 | 测试: 激活（TDD 流程） |
| Q8 安全校验 | 基本校验 | 安全: 不激活 |
| Q9 特有规则 | "DAO 层禁止抛出受检异常" | Tier 3: 部分纳入 |

**步骤 6a 生成预期：**
- 轻量分层约束（api/service/dao 职责边界，无 DDD 严格规则）
- 工厂模式 DI 规则（ServiceFactory 职责、禁止直接 new Service）
- 各层独立异常处理（DAO → RuntimeException / Service → BusinessException / API → HTTP 映射）
- TDD 流程规范（红-绿-重构、测试先行）
- 维护义务：DAO 异常规则

**通过判定：**
- [ ] 包含工厂模式 DI 规则（非 Spring 自动注入）
- [ ] 包含各层独立异常处理规则（三层异常边界不同）
- [ ] 包含 TDD 流程规范
- [ ] 包含"DAO 层禁止抛出受检异常"（Q9 激活）
- [ ] 不包含全局异常拦截规则
- [ ] 不包含 async/同步规则（混合模式下不约束）
- [ ] 不包含通用类库规则
- [ ] ≥ 150 行
