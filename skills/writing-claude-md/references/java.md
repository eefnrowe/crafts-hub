# Java 项目语言参考资料

> 适用于：Java 17+ 项目，覆盖 Spring Boot / Quarkus / Micronaut 及通用 Java 项目

## 命令表

### 构建工具检测

| 文件 | 构建工具 | 编译 | 测试 | 打包 | 运行 |
|------|---------|------|------|------|------|
| pom.xml | Maven | `mvn compile` | `mvn test` | `mvn package` | `mvn spring-boot:run` |
| build.gradle(.kts) | Gradle | `./gradlew build` | `./gradlew test` | `./gradlew jar` | `./gradlew bootRun` |

### 工具链命令

| 操作 | Maven | Gradle |
|------|-------|--------|
| Lint (Checkstyle) | `mvn checkstyle:check` | `./gradlew checkstyleMain` |
| Lint (PMD) | `mvn pmd:check` | `./gradlew pmdMain` |
| 静态分析 (SpotBugs) | `mvn spotbugs:check` | `./gradlew spotbugsMain` |
| 格式化 | `mvn spotless:apply` 或 `./gradlew spotlessApply` | |
| 架构检查 (ArchUnit) | 通过测试执行 | 通过测试执行 |
| 全部检查 | `mvn verify` | `./gradlew check` |

## 工具链覆盖清单

| 工具 | 覆盖的规则 |
|------|-----------|
| **Checkstyle** | 代码风格：缩进、大括号位置、行宽、import 排序、Javadoc |
| **PMD** | 代码质量：空 catch 块、过长方法、God Class、未使用字段 |
| **SpotBugs** | Bug 模式：空指针、资源泄漏、equals/hashCode 不一致 |
| **Error Prone** | 编译期 Bug 检测：常见编程错误 |
| **ArchUnit** | 分层约束、包依赖方向、命名约定 |
| **Spotless** | 代码格式化（google-java-format 或 palantir） |
| **google-java-format** | 缩进 2 空格、大括号风格、行宽 100 |

## 优先级分层

生成 CLAUDE.md 时按以下优先级填充，空间不足时优先保证 Tier 1。

### Tier 1：核心规则（几乎所有项目）
- DI 方式：构造器注入优先，@Autowired 仅用于测试
- DTO 隔离：Controller 接收/返回 DTO，Service 操作 Domain Model
- 全局异常处理：@ControllerAdvice + @ExceptionHandler

### Tier 2：推荐规则（取决于项目风格）
- 分层约束：Controller → Service → Repository → Entity
- 包结构：按功能模块 vs 按层分包，明确选择
- 事务管理：@Transactional 仅放在 Service 层
- 不可变性：参数和变量尽量 final
- Early return 优先

### Tier 3：边缘场景（特定条件才需要）
- 避免 Lombok @Data
- 显式类型（避免 var）
- 注释最小化策略
- MapStruct 映射规范

## Java 特有约束（候选规则池）

### 架构类

- **分层约束**：Controller → Service → Repository → Entity，明确各层职责
- **DTO 隔离**：Controller 接收/返回 DTO，Service 操作 Domain Model，Entity 映射数据库
- **包结构**：按功能模块分包（com.example.feature.*）vs 按层分包（com.example.controller.*），明确选择
- **DI 方式**：构造器注入优先，`@Autowired` 仅用于测试
- **工厂模式 DI**（Tier 2，Q3=手动时激活）
  - ✅ 通过 Factory 类统一创建实例：`ServiceFactory.createUserService()`
  - ✅ Factory 方法返回接口类型，不返回具体实现
  - ❌ 禁止直接 `new ServiceImpl()`（除 Factory 内部）
  - ❌ 禁止在 Factory 外部持有实现类的直接引用

### 代码规范类

- **不可变性**：参数和变量尽量 `final`，优先不可变对象
- **避免 Lombok @Data**：用 `@Getter` / `@Setter` 精细控制，或用 `java.util.Record`
- **显式类型**：避免 `var`，用显式类型声明（提高可读性）
- **Early return**：优先 early return，避免 else 嵌套
- **注释最小化**：仅保留 cron 表达式、Regex、TODO 和测试的 given/when/then

### 异常处理类

- **全局异常处理**：`@ControllerAdvice` + `@ExceptionHandler` 统一拦截
- **Service 层**：抛出业务异常（自定义异常类），不 catch
- **Controller 层**：禁止 try-catch，由全局处理器拦截
- **事务管理**：`@Transactional` 仅放在 Service 层类级别
- **各层独立异常处理**（Tier 2，Q4=各层独立时激活）
  - DAO 层：捕获技术异常，抛出 `RuntimeException`（禁止抛出受检异常）
  - Service 层：捕获 RuntimeException，包装为业务异常（如 `BusinessException`）
  - API/Controller 层：捕获业务异常，映射为 HTTP 状态码
  - ✅ 每层仅处理本层关注点，不吞掉上层异常信息
  - ❌ 禁止 DAO 层直接抛出业务异常（层级跨越）

### 测试类

- **JUnit 5** + Mockito
- **分层测试**：`@WebMvcTest`（Controller）、`@SpringBootTest`（集成）、纯单元测试
- **命名**：`should_预期结果_when_条件` 或 `methodName_场景_预期`
- **结构**：given / when / then

## 框架特定规则

### Spring Boot

| 规则 | 说明 |
|------|------|
| DI 注入 | 构造器注入优先：`@RequiredArgsConstructor` + `final` 字段 |
| 事务 | `@Transactional` 仅放在 Service 层，readOnly 用于查询方法 |
| 配置 | `@ConfigurationProperties` 替代多个 `@Value` |
| 异常 | `@ControllerAdvice` + `@ExceptionHandler` + 自定义 `ErrorResponse` |
| API 校验 | `@Valid` + Bean Validation（`@NotNull`, `@Size`, `@Pattern`） |
| 分页 | Spring Data `Pageable`，禁止手动分页 |
| 映射 | MapStruct `@Mapper(componentModel = "spring")` 或静态 Mapper |

### Quarkus

| 规则 | 说明 |
|------|------|
| DI | CDI（`@Inject` + `@ApplicationScoped`） |
| 配置 | `application.properties` / `application.yml` + `@ConfigProperty` |
| 响应式 | 默认响应式，明确阻塞 vs 非阻塞边界 |

### Micronaut

| 规则 | 说明 |
|------|------|
| DI | 编译时注入（`@Inject` + `@Singleton`） |
| 配置 | `application.yml` + `@Value` / `@ConfigurationProperties` |

## 测试约定

- 框架：JUnit 5 + Mockito + Spring Boot Test
- 命名：`XxxTest.java`，测试目录镜像 `src/main/java` 结构
- 结构：given / when / then
- 集成测试：`@SpringBootTest` + `@Testcontainers`（数据库测试）
- Controller 测试：`@WebMvcTest` + MockMvc
- 避免反射测试（不测私有方法）

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 类名 | PascalCase | `SkillRouterService` |
| 方法名 | camelCase | `matchSkills()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 包名 | 全小写，点分隔 | `com.example.skill.router` |
| 接口 | 无前缀/后缀或 `I` 前缀（统一选择） | `SkillRepository` |
| 实现类 | `*Impl` 或 `Default*` | `InMemorySkillRepository` |
| DTO | `*Request` / `*Response` / `*Dto` | `SkillLoadRequest` |
| 异常 | `*Exception` / `*Error` | `SkillLoadException` |
| 测试 | `*Test` | `SkillRouterServiceTest` |
| 配置类 | `*Config` / `*Configuration` | `SecurityConfig` |
