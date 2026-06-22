# Java Coding Distill

> 六种编码思维，蒸馏自行业顶级的 Java 工程实践

## 这是什么？

这是一个 **编码风格 Skill 合集**，通过 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 从行业顶级的 Java 开源项目和工程大师的实践中深度蒸馏而成。

每个 Skill 不是简单的规则罗列，而是一套**可运行的认知系统**：包含核心哲学、编码模式、反模式清单、决策启发式和代码示例，可以直接用于日常开发、Code Review 和面试准备。

## 目录

| # | Skill (完整版) | 极简速查版 | 来源 | 专注领域 | 完整版行数 |
|---|---|---|---|---|---|
| 1 | [阿里巴巴Java开发手册](alibaba-java-coding/SKILL.md) | [极简速查](alibaba-java-coding/SKILL_SHORT.md) | 阿里巴巴 · p3c 泰山版 | 命名/OOP/集合/并发/MySQL/异常/安全 | 1,364 |
| 2 | [Google Guava 编码模式](google-guava-patterns/SKILL.md) | [极简速查](google-guava-patterns/SKILL_SHORT.md) | Kevin Bourrillion & Google Java Team | 不可变集合/并发/缓存/IO/哈希 | 1,972 |
| 3 | [Josh Long Spring Boot 风格](josh-long-coding-style/SKILL.md) | [极简速查](josh-long-coding-style/SKILL_SHORT.md) | Josh Long (@starbuxman) | Spring Boot 架构/自动配置/测试/反模式 | 957 |
| 4 | [Martin Fowler 架构与重构](martin-fowler-arch-style/SKILL.md) | [极简速查](martin-fowler-arch-style/SKILL_SHORT.md) | Martin Fowler · ThoughtWorks | 重构模式/企业架构/Code Smells/技术债务 | 358 |
| 5 | [Uncle Bob Clean Code](uncle-bob-clean-code/SKILL.md) | [极简速查](uncle-bob-clean-code/SKILL_SHORT.md) | Robert C. Martin | SOLID/Clean Architecture/TDD/Professionalism | 518 |
| 6 | [Harrison Chase AI Agent 架构](harrison-chase-langchain/SKILL.md) | [极简速查](harrison-chase-langchain/SKILL_SHORT.md) | Harrison Chase · LangChain | Chain/Agent/Tool/Memory/Spring AI 对照 | 710 |

**总计：5,879 行代码规范精华**

## ⚖️ 兼容性与冲突消解

由于本合集包含了不同流派（如阿里开发手册与 Clean Code）的代码规范，当混用或同时加载多个 Skill 时，可能会遇到规范冲突。

我们特地制定了 **[编码思维兼容性与消解指南](COMPATIBILITY.md)**，其中包含：
*   **兼容性矩阵**：哪些风格可以安全地同时加载。
*   **核心冲突与消解规则**：例如方法行数（阿里 80 行 vs Clean Code 5 行）、接口命名（`Impl` 后缀 vs 行为命名）的消解策略。
*   **AI 编程助手提示词模板**：如何让大模型一次性加载多个规范且不产生风格混乱。

## 怎么用

### 作为 Skill 加载（推荐）

如果使用 Hermes Agent 或其他 Agentic IDE（如 Cursor、Cline），可以直接引用对应的 Skill 文件。

> **💡 Token 极简优化建议**：对于日常的代码重构或快速 CR，推荐优先加载 **极简速查版**（如 `alibaba-java-coding-short` 或引用对应的 `SKILL_SHORT.md`），以大幅降低 Token 消耗并缩短 AI 响应延迟。

```text
加载 alibaba-java-coding-short，帮我检查这段代码的命名规范
加载 josh-long-coding-style-short，review 这个 Controller
```

### 作为参考手册

每个 SKILL.md 是自包含的 Markdown 文件，可以直接阅读：

```bash
# 查看某个 Skill 的概览
head -40 alibaba-java-coding/SKILL.md
```

### 面试复习路线

| 面试方向 | 优先看 |
|---------|-------|
| Java 基础面 | alibaba-java-coding（OOP/集合/并发）、uncle-bob-clean-code（SOLID） |
| Spring Boot 面 | josh-long-coding-style（全面覆盖） |
| 架构设计面 | martin-fowler-arch-style（重构/模式）、uncle-bob-clean-code（Clean Architecture） |
| AI Agent 面 | harrison-chase-langchain（LangChain 架构 + Spring AI 对照） |

## 各 Skill 速览

### 阿里巴巴Java开发手册（泰山版）

覆盖全部 14 大节、59 条核心规则，每条含解释 + 正误代码示例。包括：
- 命名风格、常量定义、代码格式
- OOP 规约（equals/hashCode、泛型、接口设计）
- 集合处理（HashMap、ArrayList、Collections）
- 并发处理（线程池、锁、Atomic）
- MySQL 数据库（表设计、索引、SQL）
- 异常处理、日志规约、安全规约

### Google Guava 编码模式

27 个核心编码模式，每个标注状态建议（推荐 / 需迁移）。包括：
- Immutable 集合、Preconditions、Optional
- ListenableFuture、RateLimiter、Striped
- CacheBuilder、Multimap、BiMap、Table
- BloomFilter、Splitter/Joiner、EventBus
- Java 8+ 迁移路径对照

### Josh Long Spring Boot 风格

从实际 GitHub 仓库源码提取的 Spring Boot 编码实践。包括：
- 5 条核心理念（约定优于配置、构造器注入、聚焦接口）
- 9 种 Spring Boot 模式（Actuator、Event、Security DSL）
- 12 条反模式清单（字段注入、大 Controller、魔法数字）
- 6 套决策树（注入方式、响应式 vs 命令式、测试切片选择）
- 生产就绪检查清单（15 项）

### Martin Fowler 架构与重构

- 核心哲学：可读性优先、两顶帽子、演化式设计
- 24 种经典 Code Smells
- 企业架构模式（领域逻辑、数据源、ORM、Web 层、分布式）
- 技术债务四象限模型
- 决策启发式（Rule of Three、何时重构）

### Uncle Bob Clean Code

- SOLID 原则逐条详解 + 违反/修正代码示例
- Clean Code 函数规则（小、单一职责、参数少）
- Clean Architecture 依赖规则 + 分层图解
- TDD 三定律 + FIRST 原则
- 反模式清单（函数级、类级、架构级、职业素养级）

### Harrison Chase AI Agent 架构

- LangChain 5 大核心抽象（Chain/Agent/Tool/Memory/Callback）
- 4 种 Agent 类型（ReAct、Plan-and-Execute、Tool-calling）
- 3 种 Memory 模式 + 选型决策表
- Tool 设计哲学（粒度、描述契约、错误处理）
- Spring AI 完整对照表 + Java 实现示例

## 蒸馏方法

所有 Skill 均通过以下流程生成：

1. **多源信息采集** — 克隆分析实际源码、采集公开著作/演讲/博客
2. **多维度调研** — 哲学/模式/反模式/测试/架构/时间线 六维覆盖
3. **思维框架提炼** — 提取心智模型、决策启发式、表达 DNA
4. **质量验证** — 已知测试（对比真实立场）+ 边缘测试（推断能力）+ 风格测试（辨识度）
5. **Skill 装配** — 按标准化模板组装为可运行的 SKILL.md

蒸馏工具：[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)

## License

MIT License — 详见 [LICENSE](LICENSE)

Copyright © 2026 wjy0705

*Built with [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)*
