# Rust 项目语言模板

> 适用于：Rust 2021+ 项目，覆盖 Actix-web / Axum / Tokio 及标准库项目

## 命令表

### 工具链命令

| 操作 | 命令 |
|------|------|
| 编译 | `cargo build` |
| 编译（release） | `cargo build --release` |
| 测试 | `cargo test` |
| 测试（详细） | `cargo test -- --nocapture` |
| 测试（单线程） | `cargo test -- --test-threads=1` |
| Lint | `cargo clippy -- -D warnings` |
| 格式化 | `cargo fmt` |
| 格式化检查 | `cargo fmt -- --check` |
| 依赖审计 | `cargo audit` |
| 依赖许可检查 | `cargo deny check` |
| 文档生成 | `cargo doc --open` |
| 全部检查 | `cargo fmt --check && cargo clippy -- -D warnings && cargo test` |

## 工具链覆盖清单

| 工具 | 覆盖的规则 |
|------|-----------|
| **rustfmt** | 代码格式：缩进（4 空格）、大括号、行宽 100、import 分组 |
| **clippy** | 综合 Lint：常见错误、性能反模式、惯用法建议、复杂度警告 |
| **cargo test** | 单元测试、集成测试、文档测试 |
| **cargo audit** | 依赖安全漏洞扫描 |
| **cargo deny** | 依赖许可证合规、重复依赖、废弃 crate |
| **rustc 类型系统** | 所有权/借用规则、生命周期、trait bound（编译器强制） |

## Rust 特有约束（候选规则池）

### 架构类

- **目录结构**：`src/bin/`（入口）/ `src/lib.rs`（库根）/ `src/<module>/`（模块）
- **Error 类型**：定义项目级 `Error` 枚举，实现 `std::error::Error` + `From<具体错误>`
- **Result 传播**：使用 `?` 操作符，禁止 `unwrap()` / `expect()` 在业务代码中（仅测试允许）
- **trait 设计**：小而聚焦，避免大 trait，使用 trait bound 约束泛型

### 代码规范类

- **unwrap 策略**：业务代码禁止 `unwrap()`/`expect()`，使用 `?` 或 `ok_or`/`ok_or_else`
- **clone 策略**：避免不必要的 `clone()`，优先借用和引用。如果必须 clone，说明原因
- **unsafe 策略**：禁止 `unsafe`（除非有明确的 safety 注释说明为何安全），隔离在 `unsafe fn` 中
- **panic 策略**：库代码禁止 panic，仅允许返回 Result。`assert!` 仅在测试中使用
- **字符串**：`&str` 用于借用，`String` 用于拥有所有权。避免不必要的 `to_string()`

### 异步类

- **运行时选择**：明确 Tokio / async-std（通常 Tokio），不混用
- **async 函数**：标注 `#[tokio::main]` 或 `#[tokio::test]`
- **Send 约束**：异步任务需满足 `Send`，注意 `!Send` 类型跨 await point
- **取消安全**：异步操作考虑取消安全性（如数据库事务）

### 错误处理类

- **Error 枚举模式**：`thiserror` crate 定义错误枚举
- **错误链**：`#[source]` 属性保留错误链，`#[from]` 自动转换
- **错误上下文**：`anyhow::Context` 或 `eyre` 添加上下文信息
- **禁止 Box<dyn Error>**：使用具体错误类型或 `anyhow::Error`

## 框架特定规则

### Axum

| 规则 | 说明 |
|------|------|
| Handler | `async fn handler(Extractors) -> impl IntoResponse` |
| State | `State<AppState>` 共享状态，通过 `Router::with_state()` 注入 |
| Extractor | `Json<T>`, `Path<T>`, `Query<T>` 请求解析 |
| 错误处理 | 自定义 `IntoResponse` 实现统一错误响应 |
| 中间件 | Tower middleware，`layer()` 挂载 |
| 路由 | `Router::new().route("/path", post(handler))` |

### Actix-web

| 规则 | 说明 |
|------|------|
| Handler | `async fn handler(HttpRequest, Json<T>) -> HttpResponse` |
| State | `web::Data<AppState>` 共享状态 |
| Extractor | `web::Json<T>`, `web::Path<T>`, `web::Query<T>` |
| 错误处理 | 自定义 `ResponseError` trait 实现 |
| 中间件 | `actix_web::middleware` + 自定义 `Transform` |
| 路由 | `App::new().service(web::resource("/path").route(post(handler)))` |

### Tokio

| 规则 | 说明 |
|------|------|
| 运行时 | `#[tokio::main]` 入口，明确 `flavor = "multi_thread"` 或 `current_thread` |
| 任务 | `tokio::spawn` 要求 `Send + 'static` |
| 通道 | `tokio::sync::mpsc` 有界通道优先 |
| 同步 | `tokio::sync::Mutex`（跨 await）/ `std::sync::Mutex`（不跨 await） |

## 测试约定

- 框架：内置 `#[test]` + `tokio::test`
- 单元测试：`#[cfg(test)] mod tests` 放在同一文件
- 集成测试：`tests/` 目录下独立文件
- 命名：`test_<场景>_<预期>`
- 属性测试：`proptest` crate 用于边界条件覆盖
- Mock：`mockall` crate，仅 mock 外部依赖
- 基准测试：`#[bench]`（需 `#![feature(test)]`）或 criterion.rs

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 类型/结构体 | PascalCase | `SkillRouter` |
| trait | PascalCase | `Matcher`, `SkillLoader` |
| 枚举变体 | PascalCase | `MatchResult::Found` |
| 函数/方法 | snake_case | `match_skills()` |
| 变量 | snake_case | `matched_skills` |
| 常量 | SCREAMING_SNAKE_CASE | `MAX_RETRIES` |
| 模块 | snake_case | `skill_router` |
| 文件 | snake_case | `skill_router.rs` |
| Cargo feature | kebab-case | `semantic-match` |
| 生命周期 | 短小写（`'a`, `'ctx`） | `fn parse<'a>(&'a str)` |
| Error 变体 | PascalCase | `NotFound`, `InvalidInput` |
