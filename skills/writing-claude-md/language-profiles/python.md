# Python 项目语言模板

> 适用于：Python 3.10+ 项目，覆盖 FastAPI / Django / Flask 及通用 Python 项目

## 命令表

### 包管理器检测

| 文件 | 包管理器 | 安装依赖 | 添加依赖 | 运行 |
|------|---------|---------|---------|------|
| pyproject.toml + poetry.lock | Poetry | `poetry install` | `poetry add <pkg>` | `poetry run python <file>` |
| pyproject.toml + uv.lock | uv | `uv sync` | `uv add <pkg>` | `uv run python <file>` |
| requirements.txt | pip | `pip install -r requirements.txt` | 手动编辑 | `python <file>` |
| pyproject.toml + pdm.lock | pdm | `pdm install` | `pdm add <pkg>` | `pdm run python <file>` |

### 工具链命令

| 操作 | 命令 |
|------|------|
| 格式化 | `ruff format src/` |
| Lint | `ruff check src/ --fix` |
| 类型检查 | `mypy src/` |
| 测试 | `pytest tests/ -v` |
| 测试覆盖率 | `pytest tests/ --cov=src --cov-report=term-missing` |
| 全部门禁 | `pre-commit run --all-files` |

## 工具链覆盖清单

以下规则已被工具链强制执行，通常不需要写入 CLAUDE.md：

| 工具 | 覆盖的规则 |
|------|-----------|
| **Ruff format** | 缩进、引号风格、行宽、尾随空白、import 排序 |
| **Ruff lint (E/W/F/I)** | 语法错误、未使用导入、空白规范 |
| **Ruff lint (S)** | 安全反模式（assert in test、hardcoded password 等） |
| **Ruff lint (B)** | bugbear 反模式（mutable default arg 等） |
| **Ruff lint (TID251)** | 禁止特定 API 导入（如禁止 os.environ） |
| **MyPy strict** | 类型注解完整性、类型安全 |
| **import-linter** | 分层依赖方向、模块间导入约束 |
| **ast-grep** | AST 级别的模式禁止（async def、silent exception 等） |

## Python 特有约束（候选规则池）

以下规则是 Python 项目中常见需要写入 CLAUDE.md 的候选：

### 架构类

- **分层约束**：如果项目采用 DDD/Clean Architecture，定义各层允许/禁止的导入
- **组合根**：DI 组合根的跨层导入豁免
- **async/同步架构**：明确项目是全同步还是全异步（反直觉：FastAPI 项目可能全同步）
- **DI 规则**：所有依赖通过 DI 框架（Dishka/python-inject/dependency-injector）注入，禁止手动实例化

### 代码规范类

- **dataclass frozen**：domain 层 dataclass 必须 `frozen=True`
- **全局状态**：模块级变量必须 `Final` 标注，缓存需豁免注释
- **异常三阶段**：domain raise → application 透传 → interface 全局拦截
- **日志**：使用 structlog 而非 logging，except 记录用 `exc_info=True`

### 边界校验类

- **路径校验**：`Path.resolve()` + `is_relative_to()` 防路径遍历
- **ID 校验**：拒绝 `/`、`\`、`..`
- **API 枚举**：`Literal[...]` 或 `Field(pattern=...)`

### 维护义务类

- **Provider 同步**：新增 @provide 方法时同步更新 PROHIBITED_INSTANTIATION 列表
- **import-linter 同步**：新增跨层导入豁免时同步更新 ignore_imports

## 框架特定规则

### FastAPI

| 规则 | 说明 |
|------|------|
| DI 注入 | 通过 Dishka `setup_dishka` 或 FastAPI `Depends()` |
| 异常处理 | `app.add_exception_handler()` 全局拦截，路由函数禁止 try/except |
| 异步架构 | 可全同步（threading 后台任务）或全异步，必须明确 |
| 配置管理 | `pydantic-settings` + `BaseSettings`，环境变量前缀 |
| 请求模型 | Pydantic `BaseModel` + `Field()` 校验，枚举用 `Literal[...]` |

### Django

| 规则 | 说明 |
|------|------|
| 分层 | Views → Services → Models（禁止 View 直接 ORM 查询） |
| ORM | 仅 Service 层操作 Model，View 层通过 Service 中转 |
| URL 路由 | `urls.py` 集中注册，app 级别 include |
| 配置 | `settings.py` + `.env`（django-environ） |
| 测试 | `pytest-django` + fixtures |

### Flask

| 规则 | 说明 |
|------|------|
| 蓝图 | 按功能模块拆分 Blueprint |
| 工厂模式 | `create_app()` 工厂函数 |
| 配置 | `app.config.from_object()` + 环境变量 |
| 测试 | pytest + Flask test client |

## 测试约定

- 框架：pytest（禁止 unittest）
- 目录：`tests/` 镜像 `src/` 结构
- 命名：`test_<模块名>.py`，函数 `test_<场景>_<预期结果>`
- 依赖 mock：通过 DI 框架的 MockProvider（如 Dishka MockProvider），禁止手动 mock
- 禁止 mock 领域层：业务逻辑测试使用真实对象
- fixtures：`conftest.py` 共享 fixtures

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 模块文件 | snake_case | `condition_evaluator.py` |
| 类名 | PascalCase | `SkillRouterService` |
| DI Provider 文件 | `*_provider.py` | `service_provider.py` |
| 配置模型 | `*Settings` / `*Config` | `AppConfig`, `DatabaseSettings` |
| Protocol 接口 | `*Protocol` | `SkillLoaderProtocol` |
| Repository 接口 | `*Repository` | `IntentStackRepository` |
| 常量 | UPPER_SNAKE_CASE + `Final` | `PACKAGE_ROOT: Final[str]` |
