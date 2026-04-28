# writing-claude-md 问答流程缺陷修复计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 修复测试发现的 10 项问答流程缺陷（P1-P10），覆盖步骤 6a 跳过逻辑、步骤 2 语言 fallback、java.md 规则池缺口、测试文件问题。

**架构：** 分 4 组任务按依赖顺序执行。先修复 SKILL.md 核心逻辑（步骤 6a + 步骤 2 + 步骤 4），再修复语言参考资料和测试文件，最后全量回归测试。

**技术栈：** 纯 Markdown 技能文档，无构建/测试命令。验证方式为按场景模拟执行。

---

## 文件结构

| 文件 | 变更类型 | 职责 |
|------|---------|------|
| `skills/writing-claude-md/SKILL.md` | 修改 | 步骤 6a 激活/跳过表、步骤 2 fallback、步骤 4 多语言检测 |
| `skills/writing-claude-md/references/java.md` | 修改 | 补充工厂模式 DI 规则 + 各层独立异常规则 |
| `skills/writing-claude-md/tests/scenarios.md` | 修改 | 删重复行 + 更新通过判定 |

---

## 任务 1：修复 SKILL.md 步骤 6a — 统一激活/跳过表（P2, P3, P4, P5, P10）

**文件：** `skills/writing-claude-md/SKILL.md`（L260-275）

**问题：** 步骤 6a 当前用散列行描述 Tier 2 条件纳入逻辑，存在 5 个缺陷：
- P2: Q7"无特殊要求"/"最小测试"缺少跳过行
- P3: Q4 [检测结果]="简单try-catch" 时应跳过而非写入
- P4: Q4"简单try-catch"缺少显式跳过行
- P5: 非跳过选项的激活逻辑均为隐含
- P10: 激活行与跳过行不对称

**方案：** 将散列行替换为紧凑的激活/跳过映射表，同时补全缺失逻辑。[检测结果] 的行为统一为"与对应显式选项一致"——检测到"跳过级"值（如无DI、简单try-catch）时跳过，检测到"激活级"值（如DDD、全局拦截）时写入。

- [ ] **步骤 1：替换步骤 6a 第二层 Tier 2 规则段**

将 L260-275 替换为：

```markdown
第二层：Tier 2 规则（条件纳入）
  → 仅纳入步骤 4 用户回答激活的 Tier 2 规则

  Q&A → Tier 2 激活映射表：
  | 问题 | 激活（写入规则） | 跳过 |
  |------|-----------------|------|
  | Q2 分层 | [检测结果] DDD/MVC/简单分层 → 写入对应分层约束规则 | 无特定架构 |
  | Q3 DI | [检测结果] 自动/手动 → 写入对应 DI 注入规则 | [检测结果] 无DI / 无DI |
  | Q4 异常 | [检测结果] 全局拦截/各层独立 → 写入对应异常处理规则 | [检测结果] 简单try-catch / 简单try-catch |
  | Q5 通用类库 | 有，请自行探测 / Other | 无，跳过 |
  | Q6 异步 | [检测结果] 全异步等 → 写入 async/同步规则 | [检测结果] 全同步 / 全同步 / 混合(无约束) |
  | Q7 测试 | [检测结果] TDD/覆盖要求 → 写入对应测试框架规则 | [检测结果] 最小测试 / 最小测试 / 无特殊要求 |
  | Q8 安全 | 严格边界校验 | 基本校验 / 无特殊要求 |

  Q5 细化：
  → Q5 选"有，请自行探测" → 纳入 Explore Agent 自动识别的类库规范摘要，放入首位效应区域
  → Q5 选 Other（提供了路径或标识）→ 纳入针对性分析的类库规范摘要，放入首位效应区域

  Q6 反直觉特例：[检测结果] 全同步时，若项目使用天然异步框架（FastAPI/Node/Actix），仍写入 async 规则并标注反直觉。

  无 Q&A 映射的 Tier 2 规则（通用池）：行数 < 120 行时纳入，否则跳过
  → 放在中间区域
```

- [ ] **步骤 2：同步更新步骤 6a 中的旧引用**

步骤 6a 后文 L276-295 中如有引用旧散列行的内容（如"用户答'无DI' → 跳过"等），需确认与映射表一致。逐一检查并删除冗余行。

- [ ] **步骤 3：验证映射表覆盖所有 Q2-Q8 选项**

逐行对照 SKILL.md 步骤 4 中 Q2-Q8 的完整选项列表，确认映射表每个选项都出现在"激活"或"跳过"列中。输出覆盖检查清单。

---

## 任务 2：修复 SKILL.md 步骤 2 — 非预设语言 fallback（P1）

**文件：** `skills/writing-claude-md/SKILL.md`（L113）

**问题：** 用户纠正为非 5 种预设语言（如 Kotlin/Scala）时，`references/<lang>.md` 不存在，SKILL.md 无 fallback 说明。

- [ ] **步骤 1：在步骤 2 末尾添加 fallback 说明**

在 L113 `确认后加载对应的 \`references/<lang>.md\` 语言参考资料。` 之后添加：

```markdown

若 `references/<lang>.md` 不存在（如 Kotlin/Scala 等非预设语言）：
- 不报错，改为从步骤 1 的探测结果推断语言特性
- 跳过步骤 3 中对语言参考文件的 Tier 判定，仅基于工具链覆盖分析生成规则
- 在步骤 6a 生成时，从项目代码结构中提取语言特有模式（如 Kotlin data class、coroutines 等）
```

---

## 任务 3：修复 SKILL.md 步骤 4 — 多语言检测示例（P6）

**文件：** `skills/writing-claude-md/SKILL.md`（L161-166, L195-199）

**问题：** 第一轮和第三轮预分析示例全是 Java/Python 特征，缺少前端/Go/Rust 的检测说明。

- [ ] **步骤 1：扩展第一轮预分析为多语言覆盖**

将 L161-164 替换为：

```markdown
基于步骤 1 的探测结果，预分析架构现状：
- 分层模式（Python: domain/application → DDD，Java: controllers/models → MVC，前端: pages/components + services → 简单分层，Go/Rust: cmd/ + internal/ → 简单分层，无分层目录 → 无架构）
- DI 方式（Java: Spring @Autowired / CDI，Python: dishka / FastAPI Depends，前端: React Context / Vue provide-inject，Go: wire / dig，Rust: trait + Box<dyn>，工厂类 → 手动，无 DI 框架）
- 异常策略（Java: @ControllerAdvice / ResponseEntityExceptionHandler，Python: @app.exception_handler / FastAPI HTTPException，前端: ErrorBoundary / 全局 catch，Go: panic/recover / error return，Rust: Result<T,E> / thiserror，零统一处理）
```

- [ ] **步骤 2：扩展第三轮预分析为多语言覆盖**

将 L195-198 替换为：

```markdown
基于步骤 1 的探测结果，预分析代码规范现状：
- 同步/异步现状（Python: async/await，Java: CompletableFuture/AsyncContext，前端: Promise/async-await，Go: goroutine/channel，Rust: tokio::spawn/async fn）
- 测试现状（Python: pytest-cov + fixture 密度，Java: JUnit5 + 测试结构对称性，前端: vitest/jest + testing-library，Go: go test + table-driven，Rust: cargo test + proptest）
```

---

## 任务 4：修复 references/java.md — 补充规则（P7, P8）

**文件：** `skills/writing-claude-md/references/java.md`

**问题：** DI 规则仅覆盖 Spring（@Autowired），异常规则仅覆盖 @ControllerAdvice 全局拦截。缺少工厂模式 DI 和各层独立异常处理的规则内容。

- [ ] **步骤 1：读取 java.md 找到 DI 规则段和异常规则段**

读取文件，定位 DI 相关 Tier 2 规则和异常相关 Tier 2 规则的具体行号。

- [ ] **步骤 2：在 DI 规则段补充工厂模式**

在 DI 规则段末尾追加工厂模式规则（Tier 2，条件：Q3=手动 或 [检测结果]=手动）：

```markdown
- **工厂模式 DI**（Tier 2，Q3=手动时激活）
  - ✅ 通过 Factory 类统一创建实例：`ServiceFactory.createUserService()`
  - ✅ Factory 方法返回接口类型，不返回具体实现
  - ❌ 禁止直接 `new ServiceImpl()`（除 Factory 内部）
  - ❌ 禁止在 Factory 外部持有实现类的直接引用
```

- [ ] **步骤 3：在异常规则段补充各层独立处理模式**

在异常规则段末尾追加各层独立异常规则（Tier 2，条件：Q4=各层独立 或 [检测结果]=各层独立）：

```markdown
- **各层独立异常处理**（Tier 2，Q4=各层独立时激活）
  - DAO 层：捕获技术异常，抛出 `RuntimeException`（禁止抛出受检异常）
  - Service 层：捕获 RuntimeException，包装为业务异常（如 `BusinessException`）
  - API/Controller 层：捕获业务异常，映射为 HTTP 状态码
  - ✅ 每层仅处理本层关注点，不吞掉上层异常信息
  - ❌ 禁止 DAO 层直接抛出业务异常（层级跨越）
```

---

## 任务 5：修复 tests/scenarios.md（P9）+ 更新测试预期

**文件：** `skills/writing-claude-md/tests/scenarios.md`

- [ ] **步骤 1：删除场景 A 重复行**

删除 L73 `- [ ] 红线回顾在末尾`（重复行，L72 已有）。

- [ ] **步骤 2：更新场景 A 的 Q6 测试预期**

场景 A Q6 使用 [检测结果] 全同步。根据任务 1 的映射表更新，Q6 [检测结果] 全同步在 FastAPI 项目中走"反直觉特例"（仍写入 async 规则）。确认场景 A 预期不变（仍包含 async/同步规则）。

- [ ] **步骤 3：更新场景 G 的通过判定**

场景 G 的 Q3=手动、Q4=各层独立，在任务 4 补充 java.md 规则后，应从 FAIL 升级为 PASS。更新通过判定说明，引用 java.md 新增规则。

---

## 任务 6：全量回归测试

**文件：** 无（验证步骤）

- [ ] **步骤 1：重新执行所有 6 个场景的预分析检测**

对照修改后的 SKILL.md，对每个场景重新验证：
- 步骤 4 预分析检测结果是否与项目实际一致
- 步骤 6a 映射表中每个选项是否正确路由到激活/跳过
- 步骤 2 fallback 是否生效（场景 F）

- [ ] **步骤 2：验证修复项对应的测试通过**

| 问题编号 | 对应场景 | 验证点 |
|---------|---------|--------|
| P1 | F | 步骤 2 有 fallback 说明 → PASS |
| P2 | A, F | Q7 无特殊要求/最小测试 → 跳过测试规则 |
| P3 | E | Q4 [检测结果]=简单try-catch → 跳过异常规则 |
| P4 | A | Q4 简单try-catch → 跳过异常规则 |
| P5 | B, G | 映射表有显式激活列 |
| P6 | 跨场景 | 预分析包含前端/Go/Rust 检测说明 |
| P7 | G | java.md 有工厂模式 DI 规则 |
| P8 | G | java.md 有各层独立异常规则 |
| P9 | A | 通过判定无重复行 |
| P10 | 跨场景 | 映射表激活/跳过对称 |

- [ ] **步骤 3：输出最终测试报告**

格式同之前的测试报告，含总通过率。

---

## 自检

**规格覆盖度：** P1-P10 每个问题都能对应到上述任务。无遗漏。

**占位符扫描：** 所有步骤都包含具体的文件路径、行号引用和替换内容。无"待定"/"TODO"/"后续实现"。

**类型一致性：** 映射表中的选项名称（如"简单try-catch"、"手动(工厂模式)"）与步骤 4 问答定义中的选项名称一致。后续任务引用的前面任务产出物（如映射表、java.md 新增规则）定义明确。

**依赖关系：** 任务 1 → 任务 5（测试预期依赖映射表）→ 任务 6（回归测试）。任务 2/3/4 互相独立，可与任务 1 并行。任务 6 依赖所有前置任务。
