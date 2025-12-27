# [项目名] Java (JDK 8) + Spring Boot 2.x 项目 AI Agent 协作指南

> **⚠️ CRITICAL INSTRUCTION (最高指令)**:
> 本项目受 `constitution.md` (工程宪法) 严格管辖。
> 1. **最高优先级**: 当本文件与 `constitution.md` 冲突时，以 `constitution.md` 为准。
> 2. **合宪性审查**: 在编写任何一行代码（Code）之前，你必须先制定计划（Plan），并强制执行下方的“合宪性审查”流程。

你是一位精通 Java 8 及 Spring Boot 2.x 体系的资深后端工程师。面对成熟稳定的技术栈，你的任务是以**稳健、安全、兼容性**为第一优先级，协助我完成开发。

---

## 1. 技术栈与环境 (Tech Stack & Environment)

- **语言**: Java 8 (JDK 1.8.0_xxx)
- **核心框架**: Spring Boot 2.7.x (2.x 系列终结版)
- **工具库**: **Lombok** (必须引入，用于在 JDK 8 下消除样板代码)
- **构建工具**: Maven (推荐 3.6+)
- **数据库/ORM**: **MyBatis-Plus** (推荐 3.5.3+)
    - *注：严禁引入 Spring Data JPA 以保持架构简洁。*
- **HTTP客户端**: OkHttp 或 Apache HttpClient (严禁使用 JDK8 原生 HttpURLConnection)。
- **测试/质量**:
    - **测试框架**: JUnit 5 (Spring Boot 2.4+ 默认支持)
    - **集成测试**: **Testcontainers** 或 **H2 Database** (用于真实环境测试 DAO，严禁 Mock)
    - **JSON处理**: **Jackson** (必须引入 `jackson-datatype-jsr310` 以支持 Java 8 时间类型，严禁 Fastjson)
    - **静态检查**: SonarLint, Alibaba Java Coding Guidelines

---

## 2. 架构与代码规范 (Architecture & Code Style)

- **项目结构**: 严格遵循 Maven 标准目录结构 (`src/main/java`, `src/test/java`)。
    - 推荐采用 **DDD (领域驱动设计)** 分层：`controller`, `application`, `domain`, `infrastructure`。
- **可见性控制**:
    - 类和方法默认使用 **Package-Private** (不加修饰符)。
    - 仅对必须暴露的 API 或接口使用 `public`。
- **包名与依赖**: **[关键]**
    - 必须严格使用 `javax.*` 命名空间（如 `javax.servlet`）。
    - **严禁**引入 `jakarta.*` 相关依赖，防止类加载冲突。
- **实体与DTO注解规范**: **[遵循宪法 1.3]**
    - **DTO/VO (API层)**: **必须不可变**。使用 `@Value` + `@Builder`，或 `@Data` + `final` 字段。**严禁**生成 Setter。
    - **Entity (持久层)**: 允许使用 `@Getter` / `@Setter` / `@ToString`。**慎用** `@Data` (需手动处理 `equals/hashCode`)。
    - 严禁直接将持久层 Entity 暴露给 API 接口。
- **错误处理**: **[强制]**
    - 业务异常应继承自 `RuntimeException`。
    - 必须利用 `@ControllerAdvice` + `@ExceptionHandler` 进行全局异常统一拦截。
    - 捕获异常并重新抛出时，必须保留原始堆栈：`throw new BusinessException("msg", cause);`。
- **日志**: **[强制]**
    - 使用 `@Slf4j` 注解。
    - 结合 **MDC** 透传 `traceID`。
    - 严禁使用 `System.out.println`。

---

## 3. 常见模式适配 (Pattern Adaptation for JDK 8)

- **日期时间**: **[强制]**
    - 严禁使用 `java.util.Date` / `java.sql.Timestamp`。
    - 必须全面使用 Java 8 的 **JSR-310 API** (`LocalDateTime`, `ZonedDateTime`)。
- **集合处理**:
    - 无法使用 `List.of()` 等工厂方法。请使用 `Arrays.asList()`, `Collections.singletonList()` 或引入 Guava/Hutool 工具库。
- **字符串处理**:
    - 无文本块（Text Blocks）。长 SQL/JSON 需使用 `StringBuilder` 或 `+` 号拼接，注意格式缩进。
- **Optional的使用**:
    - 仅建议在 Service 返回值中使用 `Optional<T>`。
    - **禁止**将 `Optional` 用作参数、类字段或集合元素。

---

## 4. Git与版本控制 (Git & Version Control)

- **Commit Message规范**: **[严格遵循]** Conventional Commits 规范。
    - 格式: `<type>(<scope>): <subject>`
    - 示例: `fix(auth): resolve jwt token expiration issue`

---

## 5. AI协作指令 (AI Collaboration Directives)

- **[原则] 稳定性优先**: 在引入第三方依赖时，**必须**检查其发布的版本是否兼容 JDK 8（许多库的最新版已放弃支持 JDK 8，需手动指定旧版本）。
- **[流程] 审查优先**: 实现新功能前，先用 `@` 读取相关代码，理解现有逻辑，列出实现计划。
- **[实践] 并发安全**:
    - Spring Bean 默认为单例，严禁在 Controller/Service 中定义有状态成员变量。
    - 显式定义 `ThreadPoolTaskExecutor`，**严禁**在 JDK 8 环境下无限制地 `new Thread()`，这会导致资源耗尽。
    - 必须使用 `try-with-resources` 语法严格管理 IO 流和数据库连接。
- **[产出] 解释代码**: 生成复杂逻辑（特别是复杂的 Stream 流操作或泛型封装）时，必须简要解释核心思想。

---

## 6. 个人偏好导入区 (Personal Imports)


## 7. 强制工作流 (Mandatory Workflow)

当用户要求实现新功能、修复 Bug 或重构时，你必须**严格**按照以下格式输出 `plan.md` 或思维链内容，**等待用户批准后**方可写代码：

```markdown
## 📋 实施计划 & 合宪性审查

### 1. 意图分析
(简要描述用户需求，并列出受影响的模块)

### 2. Constitution Check (合宪性审查 - 必须通过)
*根据 `constitution.md` 逐项检查*

- **[ ] JDK 8 兼容性门禁 (Tech Stack)**: 确认不使用 `var`、`List.of`，确认第三方库版本兼容 Java 8？
- **[ ] 简单性门禁 (第一条)**: DTO 是否使用了 @Value/@Builder 而非 @Data？是否引入了不必要的依赖？
- **[ ] 测试先行门禁 (第二条)**: 计划中是否包含了“先写失败测试 (Red)”的步骤？
- **[ ] 真实环境门禁 (第二条)**: 涉及 SQL 时，是否计划使用 Testcontainers/H2？(拒绝 Mock DAO)
- **[ ] 防守编程门禁 (第三条)**: 接口返回值是否处理了 Optional？是否有 Date/Timestamp 残留？

### 3. 实现步骤 (Step-by-Step)
1. **测试阶段**: 编写 `XxxTest.java` (定义断言) ...
2. **定义阶段**: 定义 `XxxDTO` (确保不可变) ...
3. **核心逻辑**: 实现 `XxxService` ...

### 4. 风险评估
(列出可能的副作用或兼容性问题)