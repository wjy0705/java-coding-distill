---
name: josh-long-coding-style-short
description: Josh Long Spring Boot 编码风格极简速查版 - 构造器注入与测试切片最佳实践
---

# Josh Long Spring Boot 风格 - 极简速查

## 1. 核心设计理念
*   **构造器注入（Constructor Injection）**：绝对禁止使用 `@Autowired` 进行字段注入。所有依赖声明为 `final`，通过构造器注入（免去 `@Autowired` 显式注解，利于单元测试与不变性）。
    ```java
    @RestController
    class CustomerHttpController {
        private final CustomerRepository repository;
        CustomerHttpController(CustomerRepository repository) {
            this.repository = repository;
        }
    }
    ```
*   **包私有（Package-Private）可见性**：避免盲目将类和构造器声明为 `public`。如果接口或类只在当前包内使用，不加 `public` 关键字（包级别封装，限定边界）。
*   **小而专注的接口**：Repository 接口继承 `ListCrudRepository`，充分利用方法命名解析。

## 2. 核心模式与反模式
*   **事件驱动解耦**：使用 `ApplicationEventPublisher` 替代直接的方法耦合调用，保持 Service 单一职责。
*   **异常处理**：使用 `ProblemDetail` (Spring 6+) 或 `@RestControllerAdvice` 统一处理异常响应，禁止直接在 Controller 中捕获并处理异常。
*   **反模式清单**：
    *   **字段注入**：`@Autowired private XxxService xxx;`（❌ 严重反模式）。
    *   **上帝 Controller**：将数据库访问或复杂计算逻辑直接写在 HTTP 接口层。
    *   **过度 Bean 声明**：无意义地手动创建配置类和 `@Bean`，应当首选 Spring Boot 的自动配置。

## 3. 测试与生产准备
*   **测试切片（Test Slices）**：禁止滥用 `@SpringBootTest`（加载完整容器太慢）。根据需要选择精准切片：
    *   `@WebMvcTest`：仅测试 Web 表现层。
    *   `@DataJpaTest`：仅测试 JPA 持久化层。
    *   `@JsonTest`：仅测试 JSON 序列化与反序列化。
*   **生产就绪**：必须引入 `spring-boot-starter-actuator`，配置健康检查（`/actuator/health`）及优雅停机（`server.shutdown=graceful`）。
