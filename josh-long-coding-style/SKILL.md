---
name: josh-long-coding-style
description: Josh Long的Spring Boot编码风格蒸馏 - 生产级Spring最佳实践
---

# Josh Long Spring Boot 编码风格

> **关于 Josh Long** (@starbuxman) — Spring Developer Advocate at VMware, Spring Boot 核心贡献者,
> "Cloud Native Java" 作者, 以及全球最具影响力的 Spring 布道者之一。
> 他的代码风格代表了 Spring 社区中最受推崇的生产级最佳实践。

---

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **Josh Long（@starbuxman）** 的 Spring Boot 编码风格与最佳实践。Josh Long 是 Spring Developer Advocate at VMware，Spring Boot 核心贡献者，"Cloud Native Java" 作者，全球最具影响力的 Spring 布道者之一。本 Skill 提炼了他在实际项目、演讲、书籍中一贯遵循的编码哲学、架构模式和反模式清单。

### 参考来源
- **开源代码**: Josh Long 的 GitHub 仓库（cloud-native-workshop, spring-authorization-server-book 等 80+ Java 文件源码分析）
- **书籍**: 《Cloud Native Java》 - Josh Long & Kenny Bastani
- **演讲**: Spring One / Devoxx / 各类 Spring 技术会议视频
- **博客**: joshlong.com, Spring Blog
- **Spring 官方文档与源码**: spring-projects/spring-boot, spring-projects/spring-framework

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：克隆分析实际 GitHub 仓库源码 → 多维度调研（编码哲学/模式/反模式/测试/架构） → 思维框架提炼 → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 写 Spring Boot 代码时 | 确保 Controller 做该做的事、Service 不沾染 HTTP 细节、依赖注入用对方式 |
| 设计项目结构时 | 掌握 feature-based 分包、自动配置最佳布局 |
| Code Review 时 | 对照 12 条反模式清单逐一排查，防止 @Autowired 字段注入、大 Controller、魔法数字等 |
| 面试 Spring Boot 岗位时 | 回答"Spring Boot 最佳实践"类问题时有理有据，引用 Josh Long 的实战风格 |

---

## 1. 核心理念 (Core Philosophy)

### 1.1 Convention Over Configuration — The Spring Way

Josh 是约定优于配置的坚定践行者。他让 Spring Boot 的自动配置做大部分工作，
只在需要偏离默认行为时才显式配置。

```java
// DO THIS — Josh's style: 让框架做重活
@SpringBootApplication  // 就这一个注解搞定一切
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
}
```

**原则**: 每个手动 `@Bean` 定义都应当有充分的理由。
如果 Spring Boot auto-configuration 能搞定，就不要重复造轮子。

### 1.2 Constructor Injection, Never Field Injection

Josh 在所有代码中始终使用构造器注入。这是 Spring 团队首推的方式，
因为它支持 final 字段、易于测试、并明确表达依赖关系。

```java
// ✅ JOSH STYLE: 构造器注入 (包级可见性，不需要 @Autowired)
@RestController
class CustomerHttpController {
    private final CustomerRepository repository;

    CustomerHttpController(CustomerRepository repository) {
        this.repository = repository;
    }
}

// ❌ 反模式: 字段注入
@RestController
class BadController {
    @Autowired  // Josh 从不这样写
    private CustomerRepository repository;
}
```

### 1.3 Code to Small, Focused Interfaces

Josh 信奉小而专的接口。一个接口一件事，做好。
Repository 接口继承 Spring Data 的标记接口，充分利用方法命名查询。

```java
// 接口尽可能小而精
interface CustomerRepository extends ListCrudRepository<Customer, Integer> {
    Customer findCustomerById(Integer id);
}

// 自定义 repository 接口也保持专注
interface RsaKeyPairRepository {
    List<RsaKeyPair> findKeyPairs();
    void save(RsaKeyPair rsaKeyPair);
}
```

### 1.4 Java Records for Immutable Data Carriers

在 Spring Boot 3.x / Java 17+ 项目中，Josh 全面使用 `record` 替代 Lombok 的 `@Data`。
Records 天然不可变、equals/hashCode/toString 自动生成、构造器简洁。

```java
// ✅ JOSH STYLE (Spring Boot 3.x): Java record
record Customer(@Id Integer id, String name, String email) {}

record RsaKeyPair(String id, Instant created, RSAPublicKey publicKey, RSAPrivateKey privateKey) {}
```

在 Spring Boot 1.x/2.x 项目中，Josh 使用 **Lombok** (`@Data`, `@AllArgsConstructor`, `@NoArgsConstructor`) — 这是当时最合适的工具。

```java
// ✅ JOSH STYLE (Spring Boot 1.x/2.x era): Lombok
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
class Reservation {
    @Id @GeneratedValue
    private Long id;
    private String reservationName;
}
```

### 1.5 Package-Private (Default) Access Modifier

Josh 倾向于使用包级可见性（无修饰符）而不是 `public`。这使得包内聚合更紧凑，
外部代码无法意外耦合到内部实现细节。他几乎从不把 Controller/Service/Configuration 标记为 `public class`。

```java
// ✅ JOSH STYLE: 包级可见性
@RestController                // class 前面没有 public
class CustomerHttpController { ... }

@Configuration                 // class 前面没有 public
class SecurityConfiguration { ... }

interface CustomerRepository ... // interface 前面没有 public
```

---

## 2. Spring Boot 模式 (Spring Boot Patterns)

### 2.1 简洁的 @SpringBootApplication + 内聚类

Josh 经常将紧密相关的类放在同一个文件中作为顶级包级类，
而不是分散到无数小文件中。这减少认知负载，让相关逻辑保持在视线内。

```java
// ✅ JOSH STYLE: 单一文件中的内聚单元
@SpringBootApplication
public class ReservationServiceApplication {
    @Bean
    IntegrationFlow incomingReservations(Sink sink, ReservationRepository rr) {
        return IntegrationFlows
                .from(sink.input())
                .handle((GenericHandler<String>) (name, headers) -> {
                    rr.save(new Reservation(null, name));
                    return null;
                })
                .get();
    }
    public static void main(String[] args) {
        SpringApplication.run(ReservationServiceApplication.class, args);
    }
}

// 在同一个文件中 — 包级可见性的支持类
@Component
class SampleDataInitializer implements ApplicationRunner { ... }

@RestController
class ReservationRestController { ... }

interface ReservationRepository extends JpaRepository<Reservation, Long> { ... }

@Entity
class Reservation { ... }
```

**注意**: 对于复杂的业务领域，Josh 也会拆分为单独文件。
关键是按**内聚性**而非**文件数量**来组织。

### 2.2 Spring Integration DSL 而非 XML

Josh 是 Spring Integration Java DSL 的大力倡导者。他用流畅的 API
替代了传统的 XML 配置，使消息流清晰可读。

```java
// ✅ JOSH STYLE: Spring Integration DSL
@Configuration
class EmailRequestsIntegrationFlowConfiguration {
    @Bean
    IntegrationFlow emailRequestsIntegrationFlow(MessageChannel requests, AmqpTemplate template) {
        var outboundAmqpAdapter = Amqp
                .outboundAdapter(template)
                .routingKey("emails");
        return IntegrationFlow
                .from(requests)
                .transform(new ObjectToJsonTransformer())
                .handle(outboundAmqpAdapter)
                .get();
    }

    @Bean
    DirectChannelSpec requests() {
        return MessageChannels.direct();
    }
}
```

### 2.3 Spring Cloud 生态深度集成

Josh 的代码始终充分利用 Spring Cloud 生态:
- **Eureka** 服务发现 (`@EnableDiscoveryClient`)
- **Config Server** 集中配置 (`@EnableConfigServer`, `@RefreshScope`)
- **Hystrix** 断路器 (`@EnableCircuitBreaker`, `@HystrixCommand(fallbackMethod = ...)`)
- **Zuul / Spring Cloud Gateway** 网关路由
- **Feign** 声明式 HTTP 客户端
- **Spring Cloud Stream** 消息驱动微服务
- **Spring Cloud Contract** 消费者驱动的契约测试

```java
@EnableResourceServer
@EnableBinding(Source.class)
@EnableCircuitBreaker
@EnableFeignClients
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
public class ReservationClientApplication {
    @Bean @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(ReservationClientApplication.class, args);
    }
}
```

### 2.4 Actuator + 自定义 HealthIndicator

Josh 强调生产就绪性。每个服务都应该有自定义的健康指标。

```java
// ✅ JOSH STYLE: 自定义 HealthIndicator
@Component
class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        return Health.status("I <3 Production!!!").build();
    }
}
```

他会在 pom.xml 中始终包含:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2.5 @Configuration 类 + @Bean 方法

Josh 的配置类总是用 `@Configuration` 注解，方法名即 Bean 名（默认方法名），
很少显式使用 `@Bean(name=...)`。

```java
@Configuration
class AuthorizationConfiguration {
    @Bean
    JdbcOAuth2AuthorizationConsentService jdbcOAuth2AuthorizationConsentService(
            JdbcOperations jdbcOperations, RegisteredClientRepository repository) {
        return new JdbcOAuth2AuthorizationConsentService(jdbcOperations, repository);
    }
}
```

**模式**: Bean 方法参数自动注入 — 直接写在方法签名中，不需要 `@Autowired`。

### 2.6 ApplicationRunner 用于启动初始化

Josh 使用 `ApplicationRunner` 而不是 `@PostConstruct` 或 `CommandLineRunner`
来执行启动时的数据初始化。ApplicationRunner 接受 `ApplicationArguments`，
比 `CommandLineRunner` 更灵活。

```java
@Bean
ApplicationRunner usersRunner(PasswordEncoder passwordEncoder, UserDetailsManager userDetailsManager) {
    return args -> {
        var builder = User.builder().roles("USER").passwordEncoder(passwordEncoder::encode);
        var users = Map.of("jlong", "password", "rwinch", "p@ssw0rd");
        users.forEach((username, password) -> {
            if (!userDetailsManager.userExists(username)) {
                userDetailsManager.createUser(builder.username(username).password(password).build());
            }
        });
    };
}
```

### 2.7 事件驱动架构 — ApplicationEventPublisher + @EventListener

Josh 广泛使用 Spring 的事件机制来解耦组件。

```java
@Configuration
class LifecycleConfiguration {
    @Bean
    ApplicationListener<RsaKeyPairGenerationRequestEvent> keyPairGenerationRequestListener(
            Keys keys, RsaKeyPairRepository repository, @Value("${jwk.key.id}") String keyId) {
        return event -> repository.save(keys.generateKeyPair(keyId, event.getSource()));
    }

    @Bean
    ApplicationListener<ApplicationReadyEvent> applicationReadyListener(
            ApplicationEventPublisher publisher, RsaKeyPairRepository repository) {
        return event -> {
            if (repository.findKeyPairs().isEmpty())
                publisher.publishEvent(new RsaKeyPairGenerationRequestEvent(Instant.now()));
        };
    }
}
```

### 2.8 Spring Security 函数式 DSL

Josh 使用 Lambda / 函数式风格的 Security 配置（而不是链式 method chaining 或 XML）。

```java
@Configuration
class SecurityConfiguration {
    @Bean
    SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange((authorize) -> authorize.anyExchange().authenticated())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .oauth2Login(Customizer.withDefaults())
            .oauth2Client(Customizer.withDefaults());
        return http.build();
    }
}
```

### 2.9 Spring Data JDBC 而不是 JPA (Spring Boot 3.x)

在较新的项目中，Josh 倾向于使用 Spring Data JDBC 而不是 JPA/Hibernate。
它与 Java records 配合更好，避免了 JPA 的代理和懒加载问题。

```java
record Customer(@Id Integer id, String name, String email) {}

interface CustomerRepository extends ListCrudRepository<Customer, Integer> {
    Customer findCustomerById(Integer id);
}
```

---

## 3. 包结构 (Package Structure)

Josh 的包结构是**按功能（feature）组织**而非按层（layer）组织。
一个功能的所有类放在同一个包下，而不是分散到 controller/service/repository 包。

### 推荐结构:

```
com.example.yourapp/
├── YourApplication.java              # @SpringBootApplication 入口
│
├── customer/                         # 客户功能模块
│   ├── Customer.java                 # 实体/record
│   ├── CustomerRepository.java       # 数据访问
│   └── CustomerHttpController.java   # REST 端点
│
├── order/                            # 订单功能模块
│   ├── Order.java
│   ├── OrderRepository.java
│   ├── OrderService.java             # 有复杂业务时才需要
│   └── OrderHttpController.java
│
├── security/                         # 安全相关
│   ├── SecurityConfiguration.java
│   └── JwtAuthenticationInterceptor.java
│
├── config/                           # 基础设施配置
│   ├── AmqpConfiguration.java
│   ├── IntegrationConfiguration.java
│   └── AotConfiguration.java
│
└── keys/                             # 子模块的进一步分组
    ├── RsaKeyPair.java
    ├── RsaKeyPairRepository.java
    ├── JdbcRsaKeyPairRepository.java
    ├── RsaPublicKeyConverter.java
    ├── RsaPrivateKeyConverter.java
    ├── Keys.java
    ├── KeyConfiguration.java
    └── LifecycleConfiguration.java
```

**关键原则**:
- 包名反映**业务领域**而非技术层
- 一个包通常包含该领域的所有类型: Controller, Repository, 实体, 配置
- 只在有足够复杂度时才拆出 Service 层
- 跨领域的配置（安全、消息队列）放在独立的 config 包中

---

## 4. 测试方法 (Testing Approach)

### 4.1 MockMvc — 切片测试 Controller 层

Josh 使用 `@AutoConfigureMockMvc` + `@MockBean` 来隔离测试 Controller 层，
不启动完整应用上下文。

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class ReservationRestControllerTest {

    @MockBean
    private ReservationRepository reservationRepository;

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void reservations() throws Exception {
        Mockito.when(this.reservationRepository.findAll())
            .thenReturn(Arrays.asList(new Reservation(1L, "Jane"), new Reservation(2L, "Joe")));

        this.mockMvc.perform(MockMvcRequestBuilders.get("/reservations"))
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON_UTF8))
            .andExpect(MockMvcResultMatchers.jsonPath("@.[0].id").value(1L))
            .andExpect(MockMvcResultMatchers.jsonPath("@.[0].reservationName").value("Jane"));
    }
}
```

### 4.2 Spring Cloud Contract — 消费者驱动契约测试

Josh 是 Spring Cloud Contract 的积极使用者。通过 `@AutoConfigureStubRunner`，
消费者可以在不启动服务提供者的情况下进行集成测试。

```java
@SpringBootTest
@AutoConfigureStubRunner(ids = "com.example:reservation-service:+:8000", workOffline = true)
@AutoConfigureJsonTesters
public class ReservationReaderTest {

    @Autowired
    private ReservationReader reservationReader;

    @Test
    public void read() throws Exception {
        Collection<Reservation> reservations = this.reservationReader.read();
        Assertions.assertThat(reservations.size()).isEqualTo(2);
        Assertions.assertThat(reservations.iterator().next().getReservationName()).isEqualTo("Jane");
    }
}
```

### 4.3 测试基类 (Base Class) 模式

对于 Spring Cloud Contract 验证端，Josh 使用 Base Class 模式
来共享 mock 配置。

```java
public class BaseClass {
    @Autowired
    private ReservationRestController controller;

    @MockBean
    private ReservationRepository reservationRepository;

    @Before
    public void before() throws Exception {
        Mockito.when(this.reservationRepository.findAll())
            .thenReturn(Arrays.asList(new Reservation(1L, "Jane"), new Reservation(2L, "Joe")));
        RestAssuredMockMvc.standaloneSetup(this.controller);
    }
}
```

### 4.4 测试原则总结

| 原则 | 做法 |
|------|------|
| **隔离性** | Mock 外部依赖，只测当前层 |
| **可读性** | 使用 AssertJ 流式断言 |
| **契约优先** | Spring Cloud Contract 确保服务间契约一致性 |
| **JSON Path** | 使用 jsonPath 验证 JSON 响应结构 |
| **切片测试** | `@AutoConfigureMockMvc`, `@AutoConfigureJsonTesters` |

---

## 5. 反模式 (Anti-Patterns)

Josh 在演讲、博客和代码中反复警告以下反模式:

### ❌ AP-01: @Autowired 字段注入

```java
// 反模式: 字段注入 — 无法测试、不可变、隐藏依赖
@Autowired private CustomerRepository repository;
```

**Josh 的替代**: 构造器注入（不需要 `@Autowired`）。

### ❌ AP-02: 巨大的 Controller 类

```java
// 反模式: 一个 Controller 做所有事情
@RestController
class AllInOneController {
    // 20+ 个端点, 混合了客户、订单、支付、用户管理...
}
```

**Josh 的替代**: 每个领域一个 Controller，保持 3-5 个端点。

### ❌ AP-03: 在 Controller 中写业务逻辑

```java
// 反模式: Controller 直接操作数据库
@RestController
class BadController {
    @Autowired private EntityManager em;

    @PostMapping("/complex")
    public Result complexOp(@RequestBody Request r) {
        // 50 行业务逻辑...
    }
}
```

**Josh 的替代**: Controller 只做请求路由和参数校验，委托给 Service 或 Repository。

### ❌ AP-04: 未使用的 @Component 扫描

```java
// 反模式: 扫描整个应用 — 拖慢启动时间
@ComponentScan("com.example")
```

**Josh 的替代**: 依赖 `@SpringBootApplication` 的默认扫描范围，保持包结构简洁。

### ❌ AP-05: 手写 JDBC 模板调用而没有封装

```java
// 反模式: 原始字符串拼接 SQL
jdbcTemplate.update("insert into ... values (" + id + ", '" + name + "')");
```

**Josh 的替代**: Spring Data Repository 或带参数绑定的 JdbcTemplate。

### ❌ AP-06: 过度使用 @RefreshScope

```java
// 反模式: 随便给 Bean 加 @RefreshScope
@RefreshScope
@Service
class MyService { ... }
```

**Josh 的替代**: 只对需要动态刷新的配置属性使用 @RefreshScope，且知晓其代理限制。

### ❌ AP-07: 忽略 Actuator 和生产就绪特性

```java
// 反模式: 没有 Actuator, 没有健康检查, 没有指标
@SpringBootApplication
public class Application { ... }
```

**Josh 的替代**: 始终包含 `spring-boot-starter-actuator` 并添加自定义 HealthIndicator。

### ❌ AP-08: @Value 散落在各个类中

```java
// 反模式: 硬编码 @Value 字符串散落在各处
@Value("${some.deeply.nested.property}") String prop;
@Value("${another.property}") String another;
```

**Josh 的替代**: 使用 `@ConfigurationProperties` 将相关配置分组为类型安全的 POJO。

### ❌ AP-09: XML 配置

```java
// 反模式 (Josh 2015年后就不再推荐): XML bean 定义
// applicationContext.xml 中有 <bean id="..." class="...">
```

**Josh 的替代**: Java Config (`@Configuration` + `@Bean`) 或自动配置。

### ❌ AP-10: 过度设计 — 不需要的抽象层

```java
// 反模式: 为每个接口加不必要的抽象
interface CustomerService { ... }
class CustomerServiceImpl implements CustomerService { ... }
class CustomerServiceFacade { ... }  // 真需要吗？
```

**Josh 的替代**: 只在有多个实现或明确接口契约需求时才抽取接口。对于只有一个实现的简单服务，直接用类。

### ❌ AP-11: 同步阻塞代码阻塞响应式线程

```java
// 反模式: 在 WebFlux 线程中调用阻塞操作
@GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<Data> stream() {
    return Flux.just(blockingIoCall());  // 阻塞了!
}
```

**Josh 的替代**: 使用 `.subscribeOn(Schedulers.boundedElastic())` 或将阻塞操作隔离到专用线程池。

### ❌ AP-12: 魔法数字和字符串

```java
// 反模式: 散落的魔法值
if (status == 1) { ... }
rabbitTemplate.convertAndSend("my-queue", ...);
```

**Josh 的替代**: 使用常量类或枚举。

---

## 6. 编码示例 (Code Examples)

### 示例 1: 完整的 REST API 服务 (Spring Boot 3.x)

```java
// ====== ApiApplication.java ======
package bootiful.api;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
}

// ====== Customer.java ======
package bootiful.api;

import org.springframework.data.annotation.Id;

record Customer(@Id Integer id, String name, String email) {}

// ====== CustomerRepository.java ======
package bootiful.api;

import org.springframework.data.repository.ListCrudRepository;

interface CustomerRepository extends ListCrudRepository<Customer, Integer> {
    Customer findCustomerById(Integer id);
}

// ====== CustomerHttpController.java ======
package bootiful.api;

import org.springframework.web.bind.annotation.*;
import java.util.Collection;

@RestController
@RequestMapping("/customers")
class CustomerHttpController {
    private final CustomerRepository repository;

    CustomerHttpController(CustomerRepository repository) {
        this.repository = repository;
    }

    @GetMapping
    Collection<Customer> customers() {
        return this.repository.findAll();
    }

    @GetMapping("/{id}")
    Customer customerById(@PathVariable Integer id) {
        return this.repository.findCustomerById(id);
    }
}
```

### 示例 2: Spring Integration + AMQP 消息处理

```java
@Configuration
class EmailRequestsIntegrationFlowConfiguration {

    @Bean
    IntegrationFlow emailRequestsIntegrationFlow(MessageChannel requests, AmqpTemplate template) {
        var outboundAmqpAdapter = Amqp
                .outboundAdapter(template)
                .routingKey("emails");
        return IntegrationFlow
                .from(requests)
                .transform(new ObjectToJsonTransformer())
                .handle(outboundAmqpAdapter)
                .get();
    }

    @Bean
    DirectChannelSpec requests() {
        return MessageChannels.direct();
    }
}
```

### 示例 3: Spring Cloud Gateway 路由配置

```java
@Configuration
class GatewayConfiguration {
    @Bean
    RouteLocator gateway(RouteLocatorBuilder rlb) {
        var apiPrefix = "/api/";
        return rlb
            .routes()
            .route(rs -> rs
                .path(apiPrefix + "**")
                .filters(f -> f
                    .tokenRelay()
                    .rewritePath(apiPrefix + "(?<segment>.*)", "/$\\{segment}")
                )
                .uri("http://localhost:8081"))
            .route(rs -> rs
                .path("/**")
                .uri("http://localhost:8020")
            )
            .build();
    }
}
```

### 示例 4: OAuth2 Authorization Server 配置

```java
@Configuration
class AuthorizationConfiguration {
    @Bean
    JdbcOAuth2AuthorizationConsentService jdbcOAuth2AuthorizationConsentService(
            JdbcOperations jdbcOperations, RegisteredClientRepository repository) {
        return new JdbcOAuth2AuthorizationConsentService(jdbcOperations, repository);
    }

    @Bean
    JdbcOAuth2AuthorizationService jdbcOAuth2AuthorizationService(
            JdbcOperations jdbcOperations, RegisteredClientRepository rcr) {
        return new JdbcOAuth2AuthorizationService(jdbcOperations, rcr);
    }
}
```

---

## 7. 决策启发 (Decision Heuristics)

当你在编写 Spring Boot 代码时，以下决策树帮助你在 Josh 的风格范围内做选择:

### 7.1 使用哪种依赖注入方式？

```
需要注入依赖？
├─ 字段是 final?           → 构造器注入 (无 @Autowired)
├─ 类需要被代理?             → 构造器注入
├─ 多个可选依赖?             → 构造器注入 + @Autowired(required=false)
└─ 你正在写测试配置类?        → @Bean 方法参数注入
```

### 7.2 应该用 record 还是 class？

```
需要一个数据载体？
├─ 不需要 JPA @Entity?      → Java record (Spring Boot 3.x)
├─ 需要 JPA @Entity?        → class + Lombok @Data/@Getter
├─ 需要 JPA + Spring Boot 3.x? → record + @Id (Spring Data JDBC)
└─ 需要可变状态?             → class
```

### 7.3 应该创建 Service 层吗？

```
Controller 正在做什么？
├─ 调用 Repository 就够?      → 不要 Service 层
├─ 涉及事务或安全检查?        → 加 Service 层
├─ 涉及多个 Repository 编排?   → 加 Service 层
├─ 只是转发热门数据?           → 不要 Service 层
└─ 逻辑太复杂 (>20行)?       → 抽取到 Service 层
```

### 7.4 选择响应式还是命令式？

```
你的场景？
├─ 高吞吐 IO 密集型 (API Gateway)?         → WebFlux (Spring Cloud Gateway)
├─ 传统 CRUD + JPA 数据库交互?             → Spring Web (MVC)
├─ 消息流处理 (Spring Integration)?        → 两者均可, Reactor 可选
├─ 团队熟悉 WebFlux?                       → WebFlux
└─ 数据流/SSE/WebSocket?                  → WebFlux
```

**Josh 的观点**: 对于大多数微服务，命令式 Spring MVC + Spring Data 更实用。
响应式在网关和消息流管道中有明确优势，但不要为了"潮流"而盲目选择。

### 7.5 选择哪种数据访问技术？

```
数据访问需求？
├─ 简单 CRUD, 无关系映射?          → Spring Data JDBC
├─ 需要 JPA 关系映射 (@OneToMany)?  → Spring Data JPA
├─ 需要极高性能的批量操作?          → JdbcTemplate
├─ 需要响应式非阻塞?               → Spring Data R2DBC
└─ 需要复杂查询/全文搜索?           → Spring Data JPA + Specification
```

### 7.6 Test slice 选择

```
测试范围？
├─ 只测 Controller 层?      → @WebMvcTest / @AutoConfigureMockMvc
├─ 只测 Repository 层?      → @DataJpaTest / @DataJdbcTest
├─ 只测 JSON 序列化?         → @JsonTest / @AutoConfigureJsonTesters
├─ 完整集成测试?              → @SpringBootTest
└─ 契约测试 (服务间)?        → Spring Cloud Contract + @AutoConfigureStubRunner
```

---

## 8. 生产就绪 (Production Readiness)

### 8.1 必备的生产组件

Josh 要求每个 Spring Boot 应用至少包含:

```xml
<!-- 生产就绪三件套 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 8.2 自定义健康检查

```java
@Component
class DatabaseHealthIndicator implements HealthIndicator {
    private final DataSource dataSource;

    DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Health health() {
        try (var conn = dataSource.getConnection()) {
            return conn.isValid(1)
                ? Health.up().withDetail("database", "reachable").build()
                : Health.down().withDetail("database", "unreachable").build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

### 8.3 配置管理

Josh 在早期项目中使用了 Spring Cloud Config Server:

```properties
# config-service
server.port=8888
spring.cloud.config.server.git.uri=${HOME}/Desktop/config
```

在较新的项目中，他倾向于使用 Spring Boot 的 `@ConfigurationProperties`:

```java
@ConfigurationProperties(prefix = "app.jwk")
record JwkProperties(String keyId, String persistencePassword, String persistenceSalt) {}
```

### 8.4 版本与依赖管理

Josh 使用 **Maven**（不是 Gradle），并且总是:
- 使用 `spring-boot-starter-parent` 作为 parent
- 使用 Spring BOM / `dependencyManagement` 管理版本
- 使用 Spring Milestone 仓库获取最新版

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0-M3</version>
    <relativePath/>
</parent>
<properties>
    <java.version>21</java.version>
</properties>
```

### 8.5 AOT / GraalVM 原生编译

在 Spring Boot 3.x 项目中，Josh 通过 `RuntimeHintsRegistrar` 为 GraalVM 原生编译提供提示:

```java
@Configuration
@ImportRuntimeHints(MyConfiguration.Hints.class)
class MyConfiguration {
    static class Hints implements RuntimeHintsRegistrar {
        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            hints.reflection().registerType(TypeReference.of("com.example.MyClass"),
                MemberCategory.values());
            hints.resources().registerPattern("sql/*.sql");
        }
    }
}
```

---

## 9. 资源与参考 (References)

| 资源 | 链接 |
|------|------|
| GitHub | https://github.com/joshlong |
| 博客 | https://joshlong.com |
| "Cloud Native Java" 书籍 | O'Reilly, 2017 (with Kenny Bastani) |
| Spring Initializr | https://start.spring.io |
| Spring One 演讲 | YouTube: "Josh Long Spring" |
| Twitter/X | @starbuxman |

### 关键项目示例

| 仓库 | 说明 |
|------|------|
| `joshlong/cloud-native-workshop` | Cloud Native Java 完整 workshop |
| `joshlong/spring-authorization-server-book` | Spring Authorization Server 实践 |
| `spring-projects/spring-boot` | Spring Boot 官方仓库 (Josh 是核心贡献者) |

---

## 10. 速查清单 (Cheat Sheet)

```
创建 Spring Boot 应用时自查清单:

[✅] 使用 @SpringBootApplication (不在额外加 @ComponentScan)
[✅] 所有 Bean 通过构造器注入 (没有 @Autowired 字段)
[✅] 类和接口使用包级可见性 (没有 public class)
[✅] 数据载体使用 record (3.x) 或 Lombok (2.x)
[✅] Controller 小而专注 (每个领域一个, <5 端点)
[✅] 确保有 Actuator 和自定义 HealthIndicator
[✅] 使用 ApplicationRunner 初始化数据
[✅] 测试使用 MockMvc / 切片测试 / Spring Cloud Contract
[✅] 配置使用 @ConfigurationProperties 而非 @Value 散落
[✅] 避免 @RefreshScope 过度使用
[✅] 按功能分包而非按层分包
[✅] 响应式 vs 命令式按实际需要选择
[✅] 使用 Java Config 而非 XML
[✅] Spring Integration DSL 而非 XML 集成配置
[✅] 使用 Maven + spring-boot-starter-parent
```
