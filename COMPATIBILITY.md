# Coding Style Compatibility & Alignment Guide
# 编码思维兼容性与消解指南

> **💡 说明**：本合集蒸馏自 6 种行业顶级的 Java/AI 编码思维。由于作者背景、设计流派和技术时代的不同，各规范在实际混用时可能产生冲突。本指南为人类开发者与 AI 编程助手提供一套 **兼容性矩阵** 与 **冲突消解启发式规则**，确保它们能够和谐共处。

---

## 1. 兼容性矩阵 (Compatibility Matrix)

| Skill 1 (行) \ Skill 2 (列) | alibaba-java | google-guava | josh-long | martin-fowler | uncle-bob | harrison-chase |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **alibaba-java** | — | ⚠️ *部分冲突* | 🟢 *高度兼容* | 🟢 *高度兼容* | ⚠️ *部分冲突* | 🟢 *高度兼容* |
| **google-guava** | ⚠️ *部分冲突* | — | ⚠️ *部分冲突* | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* |
| **josh-long** | 🟢 *高度兼容* | ⚠️ *部分冲突* | — | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* |
| **martin-fowler** | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* | — | ⚠️ *架构冲突* | 🟢 *高度兼容* |
| **uncle-bob** | ⚠️ *部分冲突* | 🟢 *高度兼容* | 🟢 *高度兼容* | ⚠️ *架构冲突* | — | 🟢 *高度兼容* |
| **harrison-chase** | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* | 🟢 *高度兼容* | — |

*   🟢 **高度兼容**：理念无缝集成，可以直接同时加载到 AI 上下文中。
*   ⚠️ **部分冲突**：细节命名、函数长度或 API 选型存在微观冲突，需按规则消解。
*   ⚠️ **架构冲突**：演进式设计 (Fowler) 与预置式整洁架构 (Uncle Bob) 存在方法论分歧。

---

## 2. 核心冲突与消解规则 (Conflict Resolution Heuristics)

### 2.1 阿里规约 (alibaba-java) vs. 干净代码 (uncle-bob)

#### 冲突 1：函数/方法长度限制
*   **阿里规约**：建议单个方法总行数不超过 **80 行**（不含注释和空行）。
*   **Clean Code**：主张函数应当“**极其短小**”（通常为 5-10 行，甚至更短），每个函数只做一件事。
*   **💡 消解规则**：
    1.  **首选 Clean Code 的“单一抽象层级”原则**：在编写复杂的业务逻辑时，尽可能提取出高可读性的小子函数。
    2.  **以阿里规约作为硬性上限**：任何重构后的方法都**绝对不能超过 80 行**。如果一个方法达到 30 行以上且包含多个逻辑块，AI 必须主动执行 `Extract Method`（提取方法）重构。

#### 冲突 2：接口实现类的命名规约
*   **阿里规约**：对于 Service 和 DAO 类，接口必须是 `XxxService`，其实现类必须以 `Impl` 结尾（如 `UserServiceImpl`）。
*   **Clean Code**：认为 `Impl` 是无用的噪声（Hungarian Notation 变体）。主张实现类应该根据其特征命名（如 `SqlUserStore` 或 `InMemoryUserStore`）。
*   **💡 消解规则**：
    1.  **业务服务层（Service/DAO）**：严格遵循 **阿里规约**，使用 `UserServiceImpl`，以符合国内 Spring 团队的普遍开发习惯。
    2.  **核心领域模型/设计模式层**：当存在多种并行策略时（如策略模式、工厂模式），遵循 **Clean Code**，使用具体的行为特征命名实现类（如 `AlipayPaymentStrategy` 而不是 `PaymentStrategyImpl`）。

---

### 2.2 谷歌 Guava (google-guava) vs. 现代 Java / Spring Boot (josh-long)

#### 冲突：不可变集合与 Optional 的 API 选型
*   **Google Guava**：大量使用 `ImmutableList.of()`、`Optional.of()`（Guava 自带的）和 `Preconditions.checkNotNull()`。
*   **现代 Java (JDK 9+) / Josh Long**：提倡使用 JDK 原生的 `List.of()`、`java.util.Optional` 以及 `Objects.requireNonNull()`。
*   **💡 消解规则**：
    1.  **JDK 版本隔离原则**：
        *   在 **Java 8+** 项目中，**禁止**引入 Guava 的 `Optional` 和基本集合工厂，必须全面使用 JDK 原生的 `Optional` 和 Stream API。
        *   使用 JDK 原生的 `List.of()`、`Set.of()`、`Map.of()`（JDK 9+）替代 Guava 的 `ImmutableList.of()` 等。
    2.  **保留 Guava 专有能力**：仅在 JDK 无法提供原生替代方案时使用 Guava 扩展，如 `Multimap`、`BiMap`、`Table` 以及 `BloomFilter`。

---

### 2.3 演进式架构 (martin-fowler) vs. 整洁架构 (uncle-bob)

#### 冲突：前期设计粒度 (Premature Architecture)
*   **Uncle Bob (Clean Architecture)**：主张从一开始就严格分层（Entities -> Use Cases -> Interface Adapters -> Frameworks），以保证绝对的测试隔离和框架无关性。
*   **Martin Fowler (Refactoring)**：主张**演进式设计**与 **YAGNI（You Aren't Gonna Need It）**，建议前期采用最简单的单体或 Transaction Script（事务脚本）架构，待复杂度提升后再通过重构演进为复杂的领域模型。
*   **💡 消解规则**：
    *   **微服务/小型项目**：采用 **Martin Fowler 路线**。使用扁平的包结构，避免过度设计。仅在代码散发出 Smell（如重复逻辑、上帝类）时才开始提炼领域层。
    *   **大型单体/核心业务域**：采用 **Uncle Bob 路线**。在项目初期即建立清晰的六边形或整洁架构边界，防止业务代码向外部框架（如 MyBatis/DB）严重泄露。

---

### 2.4 响应式编程与命令式编程的注入习惯 (josh-long)

#### 冲突：Josh Long 响应式流 vs 阿里巴巴传统阻塞编程
*   **Josh Long**：推崇响应式编程（Spring WebFlux、Project Reactor），广泛使用 `Flux`、`Mono`。
*   **阿里巴巴**：基于稳定性和排查难度，在传统的持久化层与事务层推荐使用传统的线程阻塞模型（Thread-per-request）。
*   **💡 消解规则**：
    1.  在网关（API Gateway）和高并发无状态微服务中，优先加载 `josh-long-coding-style` 以获取最优的响应式代码布局。
    2.  在涉及复杂关系型数据库事务（Spring Declarative Transactions）的传统业务中，以 `alibaba-java-coding` 为主导，避免引入响应式链条，防止由于响应式事务上下文丢失导致的悲观锁或死锁问题。

---

## 3. 给 AI 编程助手的提示词模板 (Prompt Template for LLM Agents)

当你同时向 AI 加载多个 Skill 时，你可以直接复制以下 System Instruction 以消解风格冲突：

```markdown
You are a senior Java developer. You are loaded with the following styles:
1. Alibaba Java Coding Guidelines (alibaba-java-coding)
2. Uncle Bob Clean Code (uncle-bob-clean-code)
3. Josh Long Spring Boot Style (josh-long-coding-style)

When generating or reviewing code, apply these priorities:
- Use Constructor Injection exclusively (Josh Long style).
- Keep methods highly cohesive and short (Clean Code style), but do not exceed 80 lines (Alibaba).
- For service implementation class naming, use the 'Impl' suffix (Alibaba). For design patterns, use behavioral names (Clean Code).
- Always prefer standard JDK (17+) functional APIs and Collections.of() over legacy Google Guava dependencies.
- Resolve any architectural trade-offs using the COMPATIBILITY.md guidelines.
```
