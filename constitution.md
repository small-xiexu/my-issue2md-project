# Project Constitution (Java 8 LTS Edition)
# Version: 1.0
# Ratified: 2025-12-19

本文件定义了本项目（基于 Java 8 环境）不可动摇的核心开发原则。
所有 AI Agent 及人类开发者在进行技术规划和代码实现时，必须无条件遵循。

---

## 第一条：简单性与克制 (Simplicity & Restraint)
**核心：** 在 Java 8 相对冗余的语法限制下，保持代码精简，对抗“样板代码”膨胀。

- **1.1 (YAGNI - You Ain't Gonna Need It):**
    - **严禁**过度设计。只实现 `spec.md` 或需求文档中明确要求的功能。
    - **严禁**引入未被使用的依赖库。

- **1.2 (库的务实选择):**
    - **HTTP 客户端**：鉴于 Java 8 原生 `HttpURLConnection` 的局限性，**必须**使用 OkHttp 或 Apache HttpClient (4.5+)。
    - **JSON 处理**：**统一**使用 Jackson。严禁引入 Fastjson（因安全隐患及 API 设计问题）。
    - **工具库**：允许适度使用 Guava 或 Apache Commons Lang3 补充标准库缺失（如判空、集合操作），但这不应成为引入巨大依赖树的理由。

- **1.3 (不可变性与数据载体):**
    - **严禁**使用 Lombok 的 `@Data`（因其自动生成 Setter 破坏不可变性，且生成的 equals/hashCode 在继承场景下有隐患）。
    - **DTO/VO (API层)**：**必须**保证不可变性。推荐使用 Lombok 的 `@Value` 或 `@Builder`。
    - **Entity (持久层)**：允许适度可变（使用 `@Getter`, `@Setter`），但**严禁**直接暴露给 API 层。

- **1.4 (数值精度铁律):**
    - **严禁**使用 `float` 或 `double` 类型进行任何涉及金额、数量或需要精确计算的业务逻辑处理。
    - **必须**统一使用 `java.math.BigDecimal`，且在构造时必须使用 String 参数构造器 `new BigDecimal("0.1")` 或 `BigDecimal.valueOf(0.1)`。

---

## 第二条：测试先行铁律 (Test-First Imperative)
**核心：** 测试是遗留系统和新功能的唯一安全网。不可协商。

- **2.1 (TDD 循环):**
    - 严格遵循 **Red-Green-Refactor** 流程。
    - 在编写任何业务实现代码之前，**必须**先编写一个编译通过但断言失败的测试用例。

- **2.2 (测试框架规范):**
    - **强烈建议**引入 JUnit 5（即使在 Java 8 项目中也是完全支持的）。
    - 若受限于历史原因必须使用 JUnit 4，则**严禁**在一个 `@Test` 方法中堆砌大量断言。每个测试用例必须只验证一个独立的逻辑分支。

- **2.3 (集成优先与真实环境):**
    - **严禁**在 DAO/Repository 层使用 Mockito 进行“自欺欺人”的测试。
    - **必须**使用 Testcontainers (支持 Java 8) 启动真实的数据库容器。
    - **降级方案**：若无法使用 Docker，允许使用 H2 内存数据库，但必须配置 **MySQL 兼容模式** (`MODE=MySQL`) 以验证 SQL 语法。

---

## 第三条：明确性与防守型编程 (Clarity & Defensive Coding)
**核心：** 利用严格的代码习惯弥补 Java 8 编译器检查能力的不足。

- **3.1 (Optional 的正确用法):**
    - **严禁**将 `Optional` 用作方法参数（Param）。
    - `Optional` **仅限**用于方法的返回值（Return Type）。
    - 消费 `Optional` 时，**优先**使用 `.orElse()` / `.orElseGet()` / `.map()`。
    - **严禁**在未调用 `.isPresent()` 的情况下直接调用 `.get()`。

- **3.2 (Stream API 的节制):**
    - **可读性 > 炫技**。如果 Stream 调用链超过 **3 行**，或包含复杂的 Lambda 逻辑，**必须**拆分为传统的 `for-loop` 或提取为独立方法。
    - **严禁**在 Stream 操作中修改外部变量（避免副作用）。

- **3.3 (时间处理):**
    - **严禁**使用 `java.util.Date`、`java.sql.Date` 和 `SimpleDateFormat`。
    - **必须**全量使用 Java 8 引入的 `java.time` 包（`LocalDateTime`, `ZonedDateTime`, `DateTimeFormatter`）。

---

## 第四条：单一职责与隔离 (Single Responsibility & Isolation)
**核心：** 模块解耦，防止“上帝类”的诞生。

- **4.1 (可见性控制):**
    - **默认**类修饰符应为 **Package-Private**（不加 `public`）。
    - 只有明确需要被其他模块引用的 DTO、Service 接口或工具类，才允许声明为 `public`。

- **4.2 (接口与抽象):**
    - **慎用** Java 8 接口的 `default` 方法。接口应定义契约，而非承载复杂的业务逻辑。
    - **严禁**为了“复用代码”而建立深层次的继承关系（超过 2 层）。优先使用**组合（Composition）**。

- **4.3 (异常规约):**
    - **严禁**吞掉异常（`catch (Exception e) {}`）。
    - 所有的 Checked Exception（受检异常）必须在模块边界处转化为 Unchecked Exception (`RuntimeException`) 抛出，或者在原地进行有意义的处理。
    - **日志规范**：日志记录时必须使用占位符 `log.error("Error: {}", e.getMessage(), e)`，严禁使用字符串拼接。

---

## 治理 (Governance)

本宪法具有最高优先级。
在与 AI Agent 协作时，任何生成的计划（Plan）或代码（Code）若违反上述条款，必须被**直接驳回**并要求重写。