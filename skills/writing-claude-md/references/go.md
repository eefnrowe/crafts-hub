# Go 项目语言参考资料

> 适用于：Go 1.21+ 项目，覆盖 Gin / Echo / Fiber 及标准库项目

## 命令表

### 工具链命令

| 操作 | 命令 |
|------|------|
| 编译 | `go build ./...` |
| 测试 | `go test ./...` |
| 测试（详细） | `go test -v ./...` |
| 测试覆盖率 | `go test -cover ./...` |
| 竞态检测 | `go test -race ./...` |
| Lint | `golangci-lint run` |
| 格式化 | `gofmt -w .` 或 `goimports -w .` |
| 依赖整理 | `go mod tidy` |
| 静态分析 | `go vet ./...` |
| 安全检查 | `gosec ./...` |

## 工具链覆盖清单

| 工具 | 覆盖的规则 |
|------|-----------|
| **gofmt / goimports** | 代码格式：缩进（tab）、大括号、import 排序分组 |
| **go vet** | 常见错误：未使用变量、错误格式化字符串、死锁 |
| **golangci-lint** | 综合 Lint：errcheck、staticcheck、gosimple、ineffassign、govet |
| **staticcheck** | 高级静态分析：废弃 API、性能反模式 |
| **gosec** | 安全扫描：硬编码凭证、SQL 注入、弱加密 |
| **go test -race** | 竞态条件检测 |

## 优先级分层

生成 CLAUDE.md 时按以下优先级填充，空间不足时优先保证 Tier 1。

### Tier 1：核心规则（几乎所有项目）
- 错误处理：始终包装错误上下文 fmt.Errorf("context: %w", err)
- context 传递：所有跨函数调用传 context.Context 作为第一个参数
- 禁止 panic：业务代码禁止 panic

### Tier 2：推荐规则（取决于项目风格）
- 目录结构：cmd/ / internal/ / pkg/ / api/ 约定
- 依赖方向：cmd/ → internal/ → pkg/，禁止反向
- 接口设计：消费者定义接口，小且聚焦
- goroutine 生命周期：每个 goroutine 必须有明确退出机制

### Tier 3：边缘场景（特定条件才需要）
- 通道方向标注
- sync vs channel 选择策略
- table-driven tests 模式
- 竞态检测 -race

## Go 特有约束（候选规则池）

### 架构类

- **目录结构**：`cmd/`（入口）/ `internal/`（私有包）/ `pkg/`（公共库）/ `api/`（协议定义）
- **依赖方向**：`cmd/` → `internal/` → `pkg/`，禁止反向
- **接口定义**：消费者定义接口（return interfaces, accept structs），而非生产者
- **接口隐式实现**：无需 `implements` 声明，接口应小且聚焦（单一方法接口优先）

### 代码规范类

- **错误处理**：`if err != nil { return fmt.Errorf("context: %w", err) }` 始终包装错误上下文
- **各层独立错误处理**（Tier 2，Q4=各层独立时激活）
  - Repository 层：包装底层错误 `fmt.Errorf("repository: %w", err)`，保留错误链
  - Service 层：用 `errors.Is`/`errors.As` 判断错误类型，返回业务级错误
  - Handler 层：将业务错误映射为 HTTP 状态码（如 `ErrNotFound → 404`）
  - ✅ 每层用 `%w` 包装，保持 `errors.Is`/`errors.As` 可追溯
  - ❌ 禁止 Repository 层直接返回 HTTP 状态码
- **禁止 panic**：业务代码禁止 panic，仅允许 init() 和不可恢复场景
- **context 传递**：所有跨函数调用传 `context.Context` 作为第一个参数
- **nil 检查**：函数返回 error 时，调用者必须检查 error（不检查返回值）
- **包循环**：禁止包循环导入（编译器强制，但架构设计时需注意）
- **安全边界校验**（Tier 2，Q8=严格边界校验时激活）
  - 路径校验：使用 `filepath.Rel()` + `strings.HasPrefix` 检查路径遍历，禁止用户输入直接拼入文件路径
  - ID 校验：外部 API 使用 UUID 或 opaque token，禁止暴露自增 ID
  - SQL 注入：强制使用参数化查询（`db.Query("SELECT ... WHERE id = $1", id)`），禁止字符串拼接 SQL
  - API 枚举防护：统一错误响应，不泄露内部状态差异
  - ✅ `net/http` middleware 统一做 auth + input validation
  - ❌ 禁止 `fmt.Sprintf("SELECT * FROM %s", tableName)`

### 并发类

- **goroutine 生命周期**：每个 goroutine 必须有明确的退出机制（context cancel / done channel）
- **通道方向**：声明时标注方向（`chan<-` / `<-chan`）
- **sync vs channel**：状态同步用 `sync.Mutex`，数据传递用 channel
- **禁止共享内存**：优先"不要通过共享内存来通信，而要通过通信来共享内存"

### 测试类

- **table-driven tests**：测试模式统一使用 table-driven
- **测试文件**：`*_test.go` 与被测文件同目录
- **集成测试**：`*_test.go` + `//go:build integration` 标签
- **Benchmark**：`BenchmarkXxx(b *testing.B)` 放在测试文件中
- **TDD 流程**（Tier 2，Q7=TDD优先时激活）
  - 红灯→绿灯→重构：先写失败测试 → 最小实现通过 → 重构
  - 使用 `github.com/cweill/gotests` 生成测试骨架
  - ✅ Table-driven test 作为主要测试模式
  - ❌ 禁止先写实现再补测试
- **测试覆盖要求**（Tier 2，Q7=覆盖要求时激活）
  - 配置 `go test -cover`，设置覆盖阈值
  - 使用 `go test -coverprofile=coverage.out && go tool cover -func=coverage.out` 检查
  - ✅ CI 中 `go test -cover -coverprofile=coverage.out` + 阈值检查作为门禁
  - ❌ 禁止为凑覆盖率写无意义测试

## 框架特定规则

### Gin

| 规则 | 说明 |
|------|------|
| 路由分组 | `router.Group()` 按版本/模块分组，中间件挂载在 group 上 |
| 中间件 | `gin.HandlerFunc`，错误通过 `c.Error()` 传递 |
| 绑定 | `ShouldBindJSON` / `ShouldBindQuery`，禁止手动解析 |
| 响应 | `c.JSON(code, data)`，统一响应结构 |

### Echo

| 规则 | 说明 |
|------|------|
| 路由 | `e.Group()` 分组，`e.Use()` 中间件 |
| 绑定 | `echo.Bind()` 自动绑定 |
| 错误处理 | 自定义 `HTTPErrorHandler` 统一处理 |

### Fiber

| 规则 | 说明 |
|------|------|
| 路由 | `app.Group()` 分组，express 风格 API |
| handler | `fiber.Handler`，返回 error |
| 响应 | `c.JSON()` / `c.Status().JSON()` |

### 标准库 (net/http)

| 规则 | 说明 |
|------|------|
| Handler | `http.Handler` 接口 + `http.HandlerFunc` |
| 中间件 | 函数包装模式：`func(next http.Handler) http.Handler` |
| 路由 | Go 1.22+ `http.NewServeMux()` 支持方法路由 |

## 测试约定

- 框架：`testing` 标准库 + `testify`（断言）
- 模式：table-driven tests
- 命名：`TestXxx(t *testing.T)`，`Xxx` 为被测函数名首字母大写
- HTTP 测试：`httptest.NewRecorder()` + `httptest.NewRequest()`
- Mock：`gomock` 或接口 mock，禁止 mock 具体类型
- 集成测试：`//go:build integration` 标签，`-tags integration` 运行

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 包名 | 全小写，单个词 | `router`, `handler` |
| 导出标识符 | 首字母大写 | `SkillRouter` |
| 未导出标识符 | 首字母小写 | `matchSkills` |
| 接口 | `-er` 后缀（单方法） | `Matcher`, `Reader` |
| 错误变量 | `Err` 前缀 | `ErrNotFound` |
| 常量 | 首字母大写导出 / 小写未导出 | `MaxRetries` |
| 测试 | `Test` + 函数名 | `TestMatchSkills` |
| Benchmark | `Benchmark` + 函数名 | `BenchmarkMatchSkills` |
| 文件名 | snake_case | `skill_router.go` |
