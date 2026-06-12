---
name: alibaba-java-coding
description: 阿里巴巴Java开发手册蒸馏 - 泰山版Java编码规范精华
---

# 阿里巴巴Java开发手册蒸馏 (泰山版)

The de facto Java coding standard in Chinese tech companies. This skill covers the 14 sections of the **泰山版 (Taishan Edition)** of the Alibaba Java Coding Guidelines, with key rules, explanations, and code examples for each.

---

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **《阿里巴巴Java开发手册》泰山版** 的全部 14 大节规范。该手册由阿里巴巴集团技术团队制定，是**中国互联网公司 Java 编码的事实标准**，覆盖命名风格、OOP规约、集合处理、并发处理、MySQL数据库、异常处理、安全规约等核心领域。大量 Java 面试中的编码规范题直接出自此手册。

### 参考来源
- **官方规范**: GitHub alibaba/p3c - 《阿里巴巴Java开发手册》泰山版 (MIT 协议开源)
- **配套工具**: Alibaba Java Coding Guidelines 插件（IDEA/Eclipse）
- **官方解读**: 阿里技术团队公开的技术分享与规范解读文章
- **实际案例**: 阿里巴巴集团内部 Java 项目的代码规范要求

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：采集阿里官方开源规范 → 按 14 大节逐节提炼关键规则 → 辅以代码示例（正确/错误对比） → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 写 Java 代码时 | 命名规范、常量定义、代码格式一条条对着检查，避免低级规范错误 |
| 处理集合/并发时 | 线程池怎么建、HashMap 怎么用、ArrayList 扩容怎么避免——每一条都有阿里规范依据 |
| 设计数据库时 | 表名命名规则、索引最佳实践、SQL 写法规范，直接照着写 |
| Code Review 时 | 团队不够规范？拿这个 Skill 做检查清单，每人过一遍 |
| 面试 Java 岗位时 | "阿里的命名规范是什么""线程池不能用 Executors"——面试官最爱问的规范类问题，这里全有 |

---

## 1. 命名风格 (Naming Style)

### Rule 1.1 — Class/Interface Naming

**Rule:** Class names use UpperCamelCase. Interface names use UpperCamelCase; prefer using `-able`/`-ible` suffix for capability interfaces.

**Explanation:** PascalCase for all class/interface names. Service/component interfaces conventionally use the `-Service` suffix; capability interfaces (whether a class can do something) use `-able`.

```java
// Correct
public class UserService { ... }
public class OrderQueryProcessor { ... }
public interface Serializable { ... }
public interface Runnable { ... }

// Incorrect
public class userService { ... }
public class orderquery { ... }
```

### Rule 1.2 — Method/Variable Naming

**Rule:** Methods and variables use lowerCamelCase. Constants use UPPER_SNAKE_CASE.

**Explanation:** Method names should be verbs or verb phrases; variable names should be nouns.

```java
// Correct
private String userName;
private int orderCount;
public void getUserInfo() { ... }
public void deleteOrderById(Long orderId) { ... }

// Incorrect
private String UserName;
public void GetUserInfo() { ... }
```

### Rule 1.3 — Package Naming

**Rule:** Package names are all lowercase, single-word segments separated by dots. Use reversed company domain as prefix.

**Explanation:** `com.alibaba.xxx`, `com.taobao.xxx`. Package names must be meaningful and reflect the module or layer.

```java
// Correct
package com.alibaba.user.service;
package com.taobao.order.dao;

// Incorrect
package com.alibaba.UserService;
package com.taobao.order_utils;
```

### Rule 1.4 — Boolean Variable Naming

**Rule:** Boolean variables should not be prefixed with `is` to avoid serialization issues in frameworks (e.g., Spring, MyBatis).

**Explanation:** Frameworks like Spring's `BeanUtils` and some JSON libraries treat `isXxx()` as the getter for field `xxx`, causing field name mismatch. Use positive, meaningful names.

```java
// Correct
private boolean deleted;
private boolean success;
private boolean running;

// Incorrect
private boolean isDeleted;   // gets serialized as "deleted" by some tools
private boolean isSuccess;   // isSuccess() getter → field "success"
```

### Rule 1.5 — Abstract Class / Test Naming

**Rule:** Abstract classes must be prefixed with `Abstract` or suffixed with `Base`. Test classes must be suffixed with `Test`.

**Explanation:** Clear naming convention makes code structure obvious at the file level.

```java
// Correct
public abstract class AbstractBaseService { ... }
public class BaseController { ... }
public class UserServiceTest { ... }

// Incorrect
public abstract class Base { ... }
public class TestUserService { ... }   // ambiguous
```

---

## 2. 常量定义 (Constant Definition)

### Rule 2.1 — No Magic Numbers

**Rule:** Magic numbers (bare literals with no explanation) are forbidden. Define constants with meaningful names.

**Explanation:** Any literal value that appears in business logic must be assigned to a named constant, unless the meaning is obvious from context (e.g., `0` for string compare, `-1` for not found index).

```java
// Correct
private static final int MAX_RETRY_COUNT = 3;
private static final long CACHE_EXPIRE_SECONDS = 3600L;
if (retryCount > MAX_RETRY_COUNT) { ... }

// Incorrect
if (retryCount > 3) { ... }    // what does 3 mean?
if (responseCode == 200) { ... } // REST 200?
```

### Rule 2.2 — Constant Class vs Enum

**Rule:** Long or variable-length constant collections should use `enum` instead of a `Constants` class. Short, fixed-domain constants may use a final utility class.

**Explanation:** Enums provide type safety and are self-documenting. Avoid the "Constant Interface Antipattern".

```java
// Correct — use enum for groups of related constants
public enum OrderStatus {
    PENDING(0, "待支付"),
    PAID(1, "已支付"),
    SHIPPED(2, "已发货"),
    COMPLETED(3, "已完成");

    private final int code;
    private final String desc;
    // constructor + getters
}

// Use final class for unrelated system-wide constants
public final class CommonConstants {
    public static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";
    public static final int PAGE_SIZE_DEFAULT = 20;
    private CommonConstants() {} // prevent instantiation
}
```

### Rule 2.3 — Constant Naming and Scope

**Rule:** Constants must be `static final`. Prefer `private` or `package-private` visibility; only expose in `public static final` when truly shared across modules.

**Explanation:** `public static final` constants become part of the API contract. If the value changes, all consumers must recompile. Use interface constants (`interface XxxConstants`) only if the constant set is stable.

```java
// Correct — scoped to class
public class OrderService {
    private static final int MAX_BATCH_SIZE = 1000;
    private static final String DEFAULT_SORT = "create_time";
}

// Incorrect — global constant antipattern
public interface GlobalConstants {
    int MAX_RETRY = 3;   // implicitly public static final
}
```

---

## 3. 代码格式 (Code Formatting)

### Rule 3.1 — Braces and Indentation

**Rule:** Use 4 spaces for indentation (no tabs). Opening brace `{` appears at the end of the same line (K&R style). `}` alone on its own line.

**Explanation:** Consistent brace placement improves readability. No tabs to avoid render differences across editors.

```java
// Correct
public void process() {
    if (condition) {
        doSomething();
    } else {
        doOther();
    }
}

// Incorrect
public void process()
{
    if(condition)
        {
        doSomething();
        }
}
```

### Rule 3.2 — Line Length and Wrapping

**Rule:** Maximum line length is 120 characters. When wrapping, break before operators and align with the previous line's argument start.

**Explanation:** Long lines force horizontal scrolling. Wrapping before the operator (`&&`, `+`, `.`) makes it clear the expression continues.

```java
// Correct
String result = someMethodWithLongName(param1, param2, param3,
                                        param4, param5);
boolean valid = condition1 && condition2
                && condition3;

// Incorrect — exceeds 120 chars
String result = someMethodWithLongName(param1, param2, param3, param4, param5, param6);
```

### Rule 3.3 — Blank Lines

**Rule:** One blank line between method definitions. Blank lines between logical sections within a method improve readability.

**Explanation:** Visual separation reduces cognitive load. Use sparingly — 1-2 blank lines max in a row.

```java
public void methodOne() {
    // setup
    doSetup();

    // main logic
    executeMain();
}

public void methodTwo() {    // blank line above
    ...
}
```

### Rule 3.4 — Whitespace Around Keywords and Operators

**Rule:** Add a space after `if`, `for`, `while`, `synchronized`, `catch`, and before `{`. Add spaces around binary operators (`+`, `-`, `*`, `/`, `&&`, `||`, `==`, `!=`). No space before `;` or between method name and `(`.

**Explanation:** Follows standard Java conventions and improves readability.

```java
// Correct
if (value > threshold) {
    for (int i = 0; i < size; i++) {
        result = a + b;
    }
}

// Incorrect
if(value>threshold){
    for(int i=0;i<size;i++){
        result=a+b;
    }
}
```

---

## 4. OOP规约 (OOP Rules)

### Rule 4.1 — equals() and hashCode()

**Rule:** Every class that overrides `equals()` **must** override `hashCode()`. Use `Objects.equals()` and `Objects.hash()` for safety.

**Explanation:** Breaking the `equals`/`hashCode` contract causes collections (`HashMap`, `HashSet`) to malfunction. Objects with the same equality may hash to different buckets.

```java
// Correct
public class User {
    private Long id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(id, user.id)
            && Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

### Rule 4.2 — toString() Override

**Rule:** Every POJO/DTO/VO **must** override `toString()`. Use an auto-generated method or `ToStringBuilder`.

**Explanation:** Meaningful `toString()` is essential for logging and debugging. Without it, logs show `com.example.User@1a2b3c` — useless.

```java
// Correct
@Override
public String toString() {
    return "User{id=" + id + ", name='" + name + "'}";
}

// Or use Lombok
// @ToString
// public class User { ... }
```

### Rule 4.3 — String Comparison

**Rule:** Use `Objects.equals()` over `.equals()` when either value could be null. For string literals, call `.equals()` on the **literal** to avoid NPE.

**Explanation:** Known best practice to avoid `NullPointerException`.

```java
// Correct — literal first
if ("SUCCESS".equals(status)) { ... }

// Correct — objects.equals when both are variables
if (Objects.equals(status, expectedStatus)) { ... }

// Incorrect — NPE risk
if (status.equals("SUCCESS")) { ... }   // NPE if status is null
```

### Rule 4.4 — Deprecation

**Rule:** Deprecated interfaces/methods must be annotated with `@Deprecated` and have a Javadoc `@deprecated` tag explaining the replacement.

**Explanation:** `@Deprecated` suppresses compiler warnings for callers within the same module. The Javadoc tag tells consumers what to use instead.

```java
/**
 * 获取用户信息
 * @deprecated 请使用 {@link #getUserDetail(Long)} 代替
 */
@Deprecated
public User getUser(Long userId) { ... }
```

### Rule 4.5 — Interface vs Abstract Class

**Rule:** Prefer interfaces for defining contracts (behavior). Use abstract classes only when sharing state or implementation among closely related classes.

**Explanation:** Interfaces allow multiple inheritance of type and are easier to mock/test. Abstract classes couple the subtype to the supertype's internal implementation.

```java
// Interface — preferred for contracts
public interface PaymentService {
    PaymentResult pay(Order order, PaymentMethod method);
}

// Abstract class — use when sharing implementation state
public abstract class AbstractPaymentService implements PaymentService {
    protected PaymentRepository paymentRepository;
    protected abstract void validate(Order order);
}
```

---

## 5. 集合处理 (Collection Processing)

### Rule 5.1 — toArray() Type Safety

**Rule:** Use `collection.toArray(new T[0])` — pass an **empty** typed array. Do NOT pre-size.

**Explanation:** JVM intrinsics optimized in modern JDK (especially JDK 11+) make `new T[0]` faster and safer than pre-sizing. Pre-sized arrays risk reflection-based allocation slowdowns.

```java
// Correct — modern JDK optimized
String[] arr = list.toArray(new String[0]);

// Incorrect — can be slower, uses reflection to allocate
String[] arr = list.toArray(new String[list.size()]);
```

### Rule 5.2 — ArrayList vs LinkedList

**Rule:** Default to `ArrayList` unless you need frequent insertion/deletion at the **middle** of the list. LinkedList has high memory overhead per element.

**Explanation:** ArrayList uses a compact array (O(1) random access, O(n) tail-insert). LinkedList is a doubly-linked list (O(n) random access, O(1) mid-insert but with pointer overhead). In most business CRUD scenarios, ArrayList is superior.

```java
// Correct — most business logic
List<User> userList = new ArrayList<>();

// Consider LinkedList only for:
// 1. Frequent mid-list insertion/deletion
// 2. Queue/Deque operations (but ArrayDeque is usually better)
```

### Rule 5.3 — HashMap Initial Capacity

**Rule:** When you know the expected size, set `HashMap`'s initial capacity using the formula `(expectedSize / 0.75f) + 1`.

**Explanation:** Default capacity is 16 (load factor 0.75). If you store 1000 entries in a default HashMap, it resizes ~6 times. Pre-sizing avoids costly `resize()` and rehashing.

```java
// Correct — avoids resize
int expected = 1000;
Map<String, User> userMap = new HashMap<>((int)(expected / 0.75f) + 1);

// Incorrect — will resize ~6 times to reach capacity 2048
Map<String, User> userMap = new HashMap<>(1000);
```

### Rule 5.4 — Unmodifiable Collections

**Rule:** Return `Collections.unmodifiableList()` / `List.copyOf()` (JDK 10+) for read-only views. Never return a direct reference to an internal mutable collection.

**Explanation:** Returning an internal list lets callers modify your object's state, violating encapsulation and causing hard-to-find bugs.

```java
// Correct
public List<User> getUsers() {
    return Collections.unmodifiableList(users);
}

// Or JDK 10+ immutable copy
public List<User> getUsersSafe() {
    return List.copyOf(users);
}

// Incorrect — callers can modify internal state
public List<User> getUsers() {
    return users;
}
```

### Rule 5.5 — asList Pitfall

**Rule:** `Arrays.asList()` returns a **fixed-size list** backed by the original array. Do NOT call `add()` or `remove()` on it. To get a mutable list, wrap in `new ArrayList<>(Arrays.asList(...))` or use `List.of()` (JDK 9+).

**Explanation:** `Arrays.asList()` returns an inner class of `Arrays` that delegates `set()` to the array but throws `UnsupportedOperationException` on structural modification.

```java
// Correct — fully mutable
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));

// Or immutable list (preferred when no modification needed)
List<String> immutable = List.of("a", "b", "c");

// Incorrect — UnsupportedOperationException on add/remove
List<String> list = Arrays.asList("a", "b", "c");
list.add("d");   // throws!
```

---

## 6. 并发处理 (Concurrency)

### Rule 6.1 — Use Thread Pool, Not `new Thread()`

**Rule:** Always use `ThreadPoolExecutor` for async tasks. Never create threads manually via `new Thread()`.

**Explanation:** Manual thread creation is unmanaged — threads are not reused, uncontrolled creation can OOM the JVM, and there is no queue/boundary mechanism. `ThreadPoolExecutor` provides bounded queues, rejection policies, and thread reuse.

```java
// Correct
private static final ThreadPoolExecutor EXECUTOR = new ThreadPoolExecutor(
    CORE_POOL_SIZE, MAX_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(QUEUE_SIZE),
    new ThreadFactoryBuilder().setNameFormat("biz-pool-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
EXECUTOR.execute(() -> processTask(orderId));

// Incorrect — unmanaged thread creation
new Thread(() -> processTask(orderId)).start();
```

### Rule 6.2 — Lock Selection Strategy

**Rule:** Prefer `synchronized` for small critical sections. Use `ReentrantLock` for fairness, timed wait, or multiple condition variables. Always unlock in `finally`.

**Explanation:** `synchronized` is simpler and JIT-optimized for low-contention cases. `ReentrantLock` provides richer control but requires manual unlock.

```java
// Synchronized — simple, prefer when sufficient
public synchronized void updateStock(Long productId, int delta) {
    int current = stockMap.get(productId);
    stockMap.put(productId, current + delta);
}

// ReentrantLock — richer control
private final ReentrantLock lock = new ReentrantLock();
public void batchUpdate(List<Order> orders) {
    lock.lock();
    try {
        // critical section
    } finally {
        lock.unlock();  // MUST release in finally
    }
}
```

### Rule 6.3 — volatile Semantics

**Rule:** `volatile` guarantees visibility but **not** atomicity. Use `AtomicInteger`, `AtomicLong`, or `synchronized` for read-modify-write operations.

**Explanation:** `volatile` ensures reads always see the latest write (happens-before), but `count++` is `read → increment → write` — three operations. Without atomicity, two threads can interleave and lose an update.

```java
// Correct — use Atomic class
private final AtomicInteger count = new AtomicInteger(0);
public void increment() {
    count.incrementAndGet();
}

// Incorrect — volatile does NOT make ++ atomic
private volatile int count;
public void increment() {
    count++;   // read-modify-write, NOT thread-safe
}
```

### Rule 6.4 — SimpleDateFormat Thread Safety

**Rule:** `SimpleDateFormat` is not thread-safe. Do NOT use as a `static` field. Use `DateTimeFormatter` (JDK 8+, immutable and thread-safe) or `ThreadLocal<SimpleDateFormat>`.

**Explanation:** `SimpleDateFormat` maintains internal mutable state during parsing/formatting. Shared across threads, it produces wrong results or throws exceptions.

```java
// Correct — JDK 8+ immutable formatter
private static final DateTimeFormatter FORMATTER =
    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

// Correct — ThreadLocal wrapper for legacy code
private static final ThreadLocal<SimpleDateFormat> DATE_FORMAT =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

// Incorrect — shared mutable formatter
private static final SimpleDateFormat FORMAT = new SimpleDateFormat("yyyy-MM-dd");
```

---

## 7. 控制语句 (Control Statements)

### Rule 7.1 — Complex Condition Extraction

**Rule:** Extract complex boolean expressions into named variables or methods. Nested ternary operators are forbidden (max 1 level).

**Explanation:** Inline complex conditions are hard to read and impossible to debug. A named variable serves as inline documentation.

```java
// Correct
boolean isEligibleForDiscount = user.getLevel() >= VIP_LEVEL
    && order.getAmount() > DISCOUNT_THRESHOLD
    && !order.hasUsedCoupon();
if (isEligibleForDiscount) { ... }

// Incorrect — what does this condition mean?
if (a > 10 && b.getFlag() && !c.isExpired() && d >= 5) { ... }

// Ternary — 1 level max
String label = score >= 60 ? "pass" : "fail";

// Forbidden — nested ternary
String result = a > b ? (c > d ? "first" : "second") : "third";
```

### Rule 7.2 — Return Early / Guard Clauses

**Rule:** Use guard clauses to handle edge cases first and reduce nesting. Avoid deeply nested `if-else`.

**Explanation:** Deep nesting (4+ levels) is unreadable. Returning early flattens the structure and makes the happy path clear.

```java
// Correct — guard clauses
public Result processOrder(Order order) {
    if (order == null) {
        return Result.failure("order is null");
    }
    if (order.isCancelled()) {
        return Result.failure("order already cancelled");
    }
    if (order.getAmount() <= 0) {
        return Result.failure("invalid amount");
    }
    // happy path — minimal nesting
    return doProcess(order);
}

// Incorrect — deep nesting
public Result processOrder(Order order) {
    if (order != null) {
        if (!order.isCancelled()) {
            if (order.getAmount() > 0) {
                return doProcess(order);
            }
        }
    }
    return Result.failure("invalid order");
}
```

### Rule 7.3 — Switch Default Branch

**Rule:** Every `switch` statement **must** include a `default` branch (even if it just logs or throws). For `switch` on `enum`, `default` is still required for defensive coding against new enum values.

**Explanation:** When an enum gains a new constant, a switch without `default` silently does nothing — a likely bug source.

```java
// Correct — default handles unexpected values
switch (status) {
    case PENDING:
        handlePending(); break;
    case PAID:
        handlePaid(); break;
    default:
        throw new IllegalArgumentException("Unknown status: " + status);
}
```

### Rule 7.4 — try-catch Scope Minimization

**Rule:** Keep the `try` block as narrow as possible. One `try-catch` per independent operation, not one giant try block around everything.

**Explanation:** A huge try block masks which operation failed. Narrow try blocks make error recovery and logging specific.

```java
// Correct — narrow scope
try {
    processPayment(order);
} catch (PaymentException e) {
    log.error("Payment failed for order {}", order.getId(), e);
    return Result.failure("PAYMENT_FAILED");
}
try {
    sendNotification(order);
} catch (NotificationException e) {
    log.error("Notification failed for order {}", order.getId(), e);
    // Non-critical — log and continue
}

// Incorrect — monolithic try block
try {
    processPayment(order);
    sendNotification(order);
    updateInventory(order);
} catch (Exception e) {
    // Which operation failed? What to do?
    log.error("Failed", e);
}
```

---

## 8. 注释规约 (Comments)

### Rule 8.1 — All Public APIs Must Have Javadoc

**Rule:** All `public` methods, classes, interfaces, and constants must have full Javadoc comments. Javadoc must include `@param`, `@return`, and `@throws` as applicable.

**Explanation:** Javadoc is the API contract for consumers. Missing Javadoc on public APIs makes them impossible to use without reading source code.

```java
/**
 * 根据用户ID查询用户详细信息
 *
 * @param userId 用户ID，不能为空
 * @return 用户详细信息，不存在时返回 null
 * @throws IllegalArgumentException 如果 userId 为空或小于等于0
 */
public UserDetail getUserDetail(Long userId) {
    // ...
}
```

### Rule 8.2 — Comments Must Explain "Why", Not "What"

**Rule:** Comments should explain **why** code exists, not **what** it does (the code itself should make "what" obvious). Delete commented-out code instead of leaving it.

**Explanation:** "What" comments quickly become stale and are noise. The code is the single source of truth for "what". "Why" comments preserve design decisions, workarounds, and business rationale.

```java
// Correct — explains "why"
// Retry up to 3 times because the remote service has eventual consistency
// and may return 404 for recently created resources.
for (int i = 0; i < 3; i++) { ... }

// Incorrect — explains "what" (code should be self-documenting)
// Increment counter by 1
counter++;

// Never leave commented-out code — delete it.
// if (oldLogic()) { doSomething(); }
```

### Rule 8.3 — TODO / FIXME / HACK Annotations

**Rule:** Use `// TODO`, `// FIXME`, and `// HACK` consistently with your name and date. Each annotation should be actionable.

**Explanation:** These annotations are tracked by IDEs in a "TODO" panel. Without a name and description, they turn into technical debt with no owner.

```java
// TODO: (zhangsan, 2024-03-15) 接入分布式缓存后替换此本地缓存
private final Map<Long, User> localCache = new ConcurrentHashMap<>();

// FIXME: (lisi, 2024-03-14) 浮点数精度问题，需要改用 BigDecimal
double taxRate = 0.1;

// HACK: (wangwu, 2024-03-10) 第三方SDK的bug，参数必须传null而非空串
thirdPartyApi.call(null);
```

### Rule 8.4 — Method Internal Comments

**Rule:** Methods longer than ~50 lines should have section comments to delineate logical blocks: `// === 校验参数 ===`, `// === 业务处理 ===`, `// === 结果组装 ===`.

**Explanation:** Long methods benefit from visual section markers, but the ideal fix is extracting sections into named private methods.

```java
public Result processRefund(RefundRequest request) {
    // === 参数校验 ===
    if (request == null || request.getOrderId() == null) {
        return Result.failure("INVALID_PARAM");
    }

    // === 业务处理 ===
    RefundResult refundResult = doRefund(request);

    // === 结果发送 ===
    sendRefundNotification(request, refundResult);

    return Result.success(refundResult);
}
```

---

## 9. 前后端规约 (Front-end/Backend)

### Rule 9.1 — RESTful URL Design

**Rule:** Use nouns (not verbs) for RESTful URLs. Use HTTP methods to represent actions. Use plural nouns for collections.

**Explanation:** RESTful conventions make APIs predictable and self-documenting. Verbs in URLs indicate RPC-style design.

```
Correct:
GET    /api/v1/users          — list users
POST   /api/v1/users          — create user
GET    /api/v1/users/{id}     — get user by ID
PUT    /api/v1/users/{id}     — full update user
DELETE /api/v1/users/{id}     — delete user
PATCH  /api/v1/users/{id}     — partial update

Incorrect:
POST   /api/v1/getUser
POST   /api/v1/createUser
POST   /api/v1/deleteUser
GET    /api/v1/deleteUser?id=123
```

### Rule 9.2 — Unified Response Body

**Rule:** All API responses must follow a unified response structure (e.g., `ApiResponse<T>`) with `code`, `message`, and `data` fields.

**Explanation:** A standard response body lets frontend write a single response handler. Without it, every API may return a different shape.

```java
// Unified response
public class ApiResponse<T> {
    private int code;         // 0 = success, non-zero = error code
    private String message;   // human-readable message
    private T data;           // payload

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(0, "success", data);
    }

    public static <T> ApiResponse<T> error(int code, String msg) {
        return new ApiResponse<>(code, msg, null);
    }
}

// Usage in controller
@GetMapping("/users/{id}")
public ApiResponse<UserVO> getUser(@PathVariable Long id) {
    return ApiResponse.success(userService.getUser(id));
}
```

### Rule 9.3 — Pagination Convention

**Rule:** Pagination requests use `page` (1-indexed) and `pageSize`. Pagination responses return `total`, `pages`, `currentPage`, and `items`.

**Explanation:** Consistent pagination prevents frontend index confusion. 1-indexed is more intuitive for business users.

```java
// Request
@GetMapping("/users")
public ApiResponse<PageResult<UserVO>> listUsers(
    @RequestParam(defaultValue = "1") int page,
    @RequestParam(defaultValue = "20") int pageSize
) { ... }

// Response
public class PageResult<T> {
    private long total;        // total records
    private int pages;         // total pages
    private int currentPage;   // current page number (1-indexed)
    private List<T> items;     // current page data
}
```

---

## 10. 异常处理 (Exception Handling)

### Rule 10.1 — Do NOT Suppress Exceptions

**Rule:** Never write an empty `catch` block. At minimum, log the exception. Never catch generic `Exception` or `Throwable` in application code.

**Explanation:** Suppressing exceptions hides failure signals. Catching `Throwable` catches `OutOfMemoryError` and other JVM errors that should propagate.

```java
// Correct — log and wrap
try {
    processOrder(order);
} catch (OrderException e) {
    log.error("Order processing failed: {}", order.getId(), e);
    throw new BizException("处理订单失败", e);
}

// Incorrect — silent suppression
try {
    processOrder(order);
} catch (Exception e) {
    // nothing — bug hiding!
}

// Incorrect — catches everything including JVM errors
try {
    ...
} catch (Throwable t) { ... }
```

### Rule 10.2 — Custom Exception Hierarchy

**Rule:** Use custom business exceptions. Define at least a base `BizException` and optionally system-level and application-level subclasses.

**Explanation:** Built-in exceptions (`RuntimeException`) don't convey business meaning. Custom exceptions carry error codes and are catchable at the right granularity.

```java
// Base exception
public class BizException extends RuntimeException {
    private final String errorCode;
    public BizException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    // getter ...
}

// Specific exceptions
public class OrderNotFoundException extends BizException {
    public OrderNotFoundException(Long orderId) {
        super("ORDER_NOT_FOUND", "订单不存在: " + orderId);
    }
}

public class InsufficientStockException extends BizException {
    public InsufficientStockException(Long productId) {
        super("STOCK_INSUFFICIENT", "库存不足: " + productId);
    }
}
```

### Rule 10.3 — finally or try-with-resources

**Rule:** Always release resources in `finally` blocks or use `try-with-resources` (JDK 7+). Never close resources in the `try` block directly.

**Explanation:** If an exception occurs after resource acquisition but before the close statement, the resource leaks. `try-with-resources` guarantees `AutoCloseable.close()` is called in all paths.

```java
// Correct — try-with-resources (JDK 7+)
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {
    while (rs.next()) { ... }
} catch (SQLException e) {
    log.error("DB error", e);
}

// Correct — finally block (pre-JDK 7)
InputStream is = null;
try {
    is = new FileInputStream(path);
    // ...
} finally {
    if (is != null) {
        try { is.close(); } catch (IOException ignored) { }
    }
}
```

### Rule 10.4 — Precise Exception Types

**Rule:** Catch the most specific exception type possible. Multiple catches are preferred over a single `catch (Exception e)` with `instanceof` checks.

**Explanation:** Specific exceptions make error handling precise and self-documenting. A single generic catch masks subtle differences between failure modes.

```java
// Correct — specific catches
try {
    process();
} catch (FileNotFoundException e) {
    log.error("File not found", e);
} catch (IOException e) {
    log.error("IO error", e);
} catch (BizException e) {
    log.error("Business error: {}", e.getErrorCode(), e);
}

// Incorrect — generic catch
try {
    process();
} catch (Exception e) {
    if (e instanceof FileNotFoundException) { ... }
    else if (e instanceof BizException) { ... }
}
```

---

## 11. 日志规约 (Logging)

### Rule 11.1 — Log Level Discipline

**Rule:** Follow strict log level conventions:
- **ERROR**: Runtime errors or unexpected conditions needing immediate attention.
- **WARN**: Suspicious but non-critical (e.g., deprecated API usage, retry attempts).
- **INFO**: Business-level milestones (startup, order placed, user registered).
- **DEBUG**: Development debugging; disabled in production by default.

**Explanation:** Wrong log levels cause noise blindness (too many INFOs) or missed alerts (ERROR used for recoverable retries).

```java
// ERROR — needs attention
log.error("Payment failed for order {}, retries exhausted", orderId, exception);

// WARN — suspicious but recoverable
log.warn("Retry attempt {} for payment, orderId={}", attempt, orderId);

// INFO — business milestone
log.info("Order {} created successfully, amount={}", orderId, amount);

// DEBUG — diagnostic only
if (log.isDebugEnabled()) {
    log.debug("Order detail: {}", orderDetail);
}
```

### Rule 11.2 — Use Parameterized Logging

**Rule:** Always use parameterized logging (`{}` placeholders). Never use string concatenation in log statements.

**Explanation:** Parameterized logging skips string creation when the log level is not enabled. Concatenation always evaluates, even when the log line is suppressed.

```java
// Correct — parameterized, skips when DEBUG disabled
log.debug("Processing order {} for user {}", orderId, userId);

// Incorrect — always evaluates, even when DEBUG disabled
log.debug("Processing order " + orderId + " for user " + userId);
```

### Rule 11.3 — Log Framework Unification

**Rule:** Use SLF4J as the logging facade. Use `logback` as the implementation. Avoid mixing `Log4j`, `java.util.logging`, and `commons-logging` via dependency exclusions.

**Explanation:** Multiple logging frameworks in one application produce inconsistent output and duplicate configuration. SLF4J bridges unify all frameworks behind one API.

```java
// SLF4J + Logback — the standard
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(MyClass.class);
```

### Rule 11.4 — Log Contextual Data

**Rule:** Include enough context in log messages to trace a request without needing to correlate multiple log lines manually. Use MDC (Mapped Diagnostic Context) for request-scoped fields (traceId, userId).

**Explanation:** Without context (order ID, user ID), ERROR logs are untraceable. MDC automatically attaches context to every log line in the same thread.

```java
// Set MDC context in filter/interceptor
MDC.put("traceId", UUID.randomUUID().toString());
MDC.put("userId", String.valueOf(currentUserId));

// logback pattern includes MDC: %X{traceId} %X{userId}
// Output: 2024-03-15 10:30:00.123 [http-nio-8080] INFO  [abc123, user42] c.a.OrderService - ...

// Clean up in finally
try {
    // process
} finally {
    MDC.clear();
}
```

---

## 12. MySQL数据库 (MySQL Database)

### Rule 12.1 — Table and Column Naming

**Rule:** Table names use lowercase `snake_case`. Use plural nouns for table names. Column names use lowercase `snake_case`. Every table **must** have an `id` (bigint auto-increment PK), `create_time`, and `update_time`.

**Explanation:** Consistent naming avoids quoting issues and makes SQL readable. The three mandatory columns enable standard tooling.

```sql
-- Correct
CREATE TABLE `order_items` (
    `id`          BIGINT       NOT NULL AUTO_INCREMENT,
    `order_id`    BIGINT       NOT NULL COMMENT '订单ID',
    `product_id`  BIGINT       NOT NULL COMMENT '商品ID',
    `quantity`    INT          NOT NULL DEFAULT 1 COMMENT '数量',
    `price`       DECIMAL(10,2) NOT NULL COMMENT '单价',
    `create_time` DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单明细表';

-- Incorrect — inconsistent naming
CREATE TABLE OrderItems (
    OrderId    BIGINT NOT NULL,
    Product_id BIGINT NOT NULL,
    ...
);
```

### Rule 12.2 — Index Rules

**Rule:** Indexes must follow specific naming: `idx_<table>_<column>` for non-unique, `uk_<table>_<column>` for unique. Do NOT over-index (max 5 indexes per table). For composite indexes, place high-selectivity columns first.

**Explanation:** Proper naming helps with query analysis. Too many indexes slow down writes. The leftmost prefix rule means column order matters.

```sql
-- Correct — naming convention
CREATE INDEX `idx_order_items_order_id` ON `order_items`(`order_id`);
CREATE UNIQUE INDEX `uk_users_mobile` ON `users`(`mobile`);

-- Composite index: put high-selectivity column first
-- (user_id has more unique values than status)
CREATE INDEX `idx_orders_user_status` ON `orders`(`user_id`, `status`);
-- This covers: WHERE user_id = ?, WHERE user_id = ? AND status = ?
-- Does NOT cover: WHERE status = ?
```

### Rule 12.3 — SQL Statements — No SELECT *

**Rule:** Never use `SELECT *` in production code. Explicitly list the columns needed.

**Explanation:** `SELECT *` retrieves all columns — wastes I/O, memory, and network bandwidth. It also breaks silently when columns are added/removed, potentially exposing sensitive data.

```sql
-- Correct
SELECT id, order_id, product_id, quantity, price FROM order_items WHERE order_id = ?;

-- Incorrect
SELECT * FROM order_items WHERE order_id = ?;
```

### Rule 12.4 — Pagination with LIMIT

**Rule:** Use `LIMIT offset, size` for pagination, but avoid large offsets (e.g., `LIMIT 100000, 20`). For deep pagination, use cursor-based (WHERE id > ? ORDER BY id LIMIT ?) or subquery pagination.

**Explanation:** Large offsets force MySQL to scan and discard rows. Cursor-based pagination is O(1) regardless of page depth.

```sql
-- Correct — cursor-based pagination (preferred for deep pages)
SELECT id, order_id, amount
FROM orders
WHERE id > ?   -- last id from previous page
ORDER BY id
LIMIT 20;

-- Correct — subquery pagination for large offsets
SELECT t.*
FROM orders t
INNER JOIN (
    SELECT id FROM orders ORDER BY id LIMIT 100000, 20
) tmp ON t.id = tmp.id;

-- Incorrect — huge offset, database scans 100K rows
SELECT * FROM orders ORDER BY id LIMIT 100000, 20;
```

### Rule 12.5 — Count Usage

**Rule:** Use `COUNT(*)` instead of `COUNT(1)` or `COUNT(column)`. `COUNT(*)` is optimized by InnoDB. `COUNT(column)` excludes NULLs, which is usually not what you want.

**Explanation:** InnoDB can optimize `COUNT(*)` without checking column values. `COUNT(col)` skips NULLs — use `SUM(col IS NOT NULL)` if you intentionally want that.

```sql
-- Correct — fastest, counts all rows
SELECT COUNT(*) FROM orders WHERE status = 'PAID';

-- Incorrect — slightly slower, also may skip NULLs unintentionally
SELECT COUNT(status) FROM orders;

-- Only use COUNT(column) when you explicitly want to exclude NULLs
SELECT COUNT(remark) FROM orders;   -- counts only rows where remark IS NOT NULL
```

---

## 13. 工程结构 (Project Structure)

### Rule 13.1 — Layered Architecture

**Rule:** Standard layered architecture: **Controller → Service → Manager → DAO**. Cross-layer calls only go downward. DO NOT skip layers (e.g., Controller calling DAO directly).

**Explanation:** Each layer has a responsibility: Controller (request/response), Service (business logic orchestration), Manager (common business logic / cross-DAO aggregation), DAO (database access). Skipping layers couples business logic to transport.

```
Standard package structure:
com.alibaba.xxx
    ├── common        — shared utilities, constants, enums
    ├── config        — Spring configuration classes
    ├── controller    — REST API endpoints
    ├── service       — business logic interface
    │   └── impl      — implementation
    ├── manager       — common business logic (transactional, cross-DAO)
    ├── dao           — MyBatis/JPA data access
    ├── model
    │   ├── dto       — data transfer objects
    │   ├── vo        — view objects (response to frontend)
    │   ├── entity    — database entity objects
    │   └── query     — query parameter objects
    └── exception     — custom exceptions
```

### Rule 13.2 — Dependency Rule

**Rule:** Dependencies between modules must form a **directed acyclic graph** (DAG). No circular dependencies between modules.

**Explanation:** Circular dependencies break the build (Maven/Gradle cannot resolve the compile order) and couple modules that should be independent. Use dependency inversion or introduce a shared module to break cycles.

```
Correct:  web → service → dao
                              ↑
                          common (shared types)
                          
Incorrect (circular):
  service ↔ dao    — service depends on dao, dao depends on service
```

### Rule 13.3 — Object Disambiguation

**Rule:** Domain objects must be clearly categorized: `DO` (Data Object, DB entity), `DTO` (Data Transfer Object, service layer), `VO` (View Object, response to frontend), `Query` (query parameter). Never use the same class across layers.

**Explanation:** Each layer has different concerns. Using a DO in the controller layer exposes DB internals. Copying fields between objects is overhead but enforces encapsulation.

```java
// DO — database entity
public class UserDO {
    private Long id;
    private String userName;
    private String passwordHash;    // sensitive, don't expose
    private LocalDateTime createTime;
}

// DTO — service layer transfer
public class UserDTO {
    private Long id;
    private String userName;
}

// VO — frontend response
public class UserVO {
    private Long id;
    private String userName;
    private String level;
}
```

### Rule 13.4 — Package-by-Layer vs Package-by-Feature

**Rule:** For small-to-medium projects, package-by-layer (controller/service/dao) is standard. For large projects, consider package-by-feature (order/user/product) with internal layers (controller/service/dao).

**Explanation:** Package-by-feature improves cohesion within a business domain and is easier to split into microservices later.

```java
// Package-by-feature (large projects)
com.alibaba.xxx
    ├── order
    │   ├── controller
    │   ├── service
    │   └── dao
    ├── user
    │   ├── controller
    │   ├── service
    │   └── dao
    └── product
        ├── controller
        ├── service
        └── dao
```

---

## 14. 安全规约 (Security)

### Rule 14.1 — SQL Injection Prevention

**Rule:** Always use parameterized queries (`PreparedStatement`, MyBatis `#{}`). Never concatenate user input into SQL strings. Do NOT use MyBatis `${}` for user input.

**Explanation:** SQL injection is the most critical web vulnerability. Parameterized queries ensure user input is treated as data, not executable SQL.

```java
// Correct — MyBatis #{} (parameterized)
@Select("SELECT * FROM users WHERE user_name = #{userName} AND status = 1")
UserDO findByUserName(@Param("userName") String userName);

// Incorrect — MyBatis ${} (string substitution, injectable)
@Select("SELECT * FROM users WHERE user_name = '${userName}'")
UserDO findByUserName(@Param("userName") String userName);

// Correct — JDBC PreparedStatement
String sql = "SELECT * FROM users WHERE user_name = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, userName);

// Incorrect — string concatenation
String sql = "SELECT * FROM users WHERE user_name = '" + userName + "'";
```

### Rule 14.2 — XSS Prevention

**Rule:** All user-generated content displayed in web pages must be HTML-escaped. Use template engine auto-escaping (Thymeleaf, FreeMarker with `?html`, or output encoding filters).

**Explanation:** Without escaping, user input containing `<script>alert('xss')</script>` renders as executable JavaScript. Escape based on context (HTML, JavaScript, CSS, URL).

```java
// Thymeleaf (auto-escaped by default — safe)
<p th:text="${user.remark}">user remark here</p>

// FreeMarker — use ?html for HTML escaping
<p>${user.remark?html}</p>

// Never output raw user content without escaping
// @ResponseBody returning a string with user content = safe (no HTML rendering)
// But rendering in a JSP without escape:
// <p><%= request.getParameter("input") %></p>   // XSS!
```

### Rule 14.3 — Sensitive Data Handling

**Rule:** Personally Identifiable Information (PII) — phone numbers, ID cards, bank accounts — must be masked in logs and API responses. Sensitive data must be encrypted at rest.

**Explanation:** Exposing full PII in logs violates compliance (中国的《个人信息保护法》, GDPR). Masking provides a safety net even if logs leak.

```java
// Mask phone number: 138****1234
public static String maskPhone(String phone) {
    if (phone == null || phone.length() < 7) return phone;
    return phone.substring(0, 3) + "****" + phone.substring(7);
}

// Mask ID card: 110101****1234
public static String maskIdCard(String idCard) {
    if (idCard == null || idCard.length() < 10) return idCard;
    return idCard.substring(0, 6) + "****" + idCard.substring(idCard.length() - 4);
}

// In log statements — NEVER log raw sensitive data
log.info("User registered: phone={}", maskPhone(user.getPhone()));

// Incorrect — leaks PII
log.info("User registered: phone={}", user.getPhone());
```

### Rule 14.4 — Authentication and Authorization Checks

**Rule:** Every API that accesses user data **must** verify that the requesting user is authorized. Never trust user-supplied identifiers (e.g., `orderId` in URL) without verifying ownership.

**Explanation:** Without ownership checks, user A can access user B's data by changing a URL parameter (IDOR — Insecure Direct Object Reference).

```java
// Correct — verify ownership
@GetMapping("/orders/{orderId}")
public ApiResponse<OrderVO> getOrder(@PathVariable Long orderId) {
    Long currentUserId = SecurityContextHolder.getCurrentUserId();
    Order order = orderService.findById(orderId);

    if (!order.getUserId().equals(currentUserId)) {
        // 403 — not authorized
        return ApiResponse.error(403, "无权访问该订单");
    }
    return ApiResponse.success(orderService.toVO(order));
}

// Incorrect — no ownership check
@GetMapping("/orders/{orderId}")
public ApiResponse<OrderVO> getOrder(@PathVariable Long orderId) {
    // Anyone can access any order by changing orderId!
    return ApiResponse.success(orderService.toVO(orderService.findById(orderId)));
}
```

### Rule 14.5 — Input Validation

**Rule:** All user input must be validated on the server side (not just frontend). Validate length, format, range, and allowed values.

**Explanation:** Frontend validation is for UX only — it can be bypassed trivially (curl/Postman). Server-side validation is the security boundary.

```java
// Correct — server-side validation
public void createUser(UserCreateRequest request) {
    // Bean Validation annotations
    if (request.getUserName() == null || request.getUserName().length() > 50) {
        throw new BizException("PARAM_ERROR", "用户名长度不能超过50个字符");
    }
    if (request.getAge() != null && (request.getAge() < 0 || request.getAge() > 150)) {
        throw new BizException("PARAM_ERROR", "年龄不合法");
    }
    if (!Pattern.matches("^1[3-9]\\d{9}$", request.getPhone())) {
        throw new BizException("PARAM_ERROR", "手机号格式不正确");
    }
    userService.create(request);
}

// Also use @Valid / @Validated with JSR-380 annotations
public ApiResponse<Void> createUser(@Valid @RequestBody UserCreateRequest request) { ... }
```

---

## Appendix: Quick Reference

| Category | Rule | Priority |
|----------|------|----------|
| Naming | UpperCamelCase for classes, lowerCamelCase for methods/vars | **Mandatory** |
| Naming | Package names all lowercase, single words | **Mandatory** |
| Constants | No magic numbers | **Mandatory** |
| Formatting | 4-space indent, K&R braces, 120 char line limit | **Recommended** |
| OOP | Override hashCode() when overriding equals() | **Mandatory** |
| OOP | POJO must override toString() | **Recommended** |
| Collections | Use `collection.toArray(new T[0])` | **Recommended** |
| Collections | Set initial capacity for HashMap | **Recommended** |
| Concurrency | Use ThreadPoolExecutor, not new Thread() | **Mandatory** |
| Concurrency | SimpleDateFormat is not thread-safe | **Mandatory** |
| Control | Extract complex conditions to named variables | **Recommended** |
| Control | Guard clauses to reduce nesting | **Recommended** |
| Comments | Public APIs must have Javadoc | **Mandatory** |
| Comments | Comments explain "why", not "what" | **Recommended** |
| RESTful | Use nouns, HTTP methods as actions | **Recommended** |
| Exceptions | Don't suppress exceptions | **Mandatory** |
| Exceptions | Try-with-resources for Closeable resources | **Mandatory** |
| Logging | SLF4J facade, parameterized logging | **Mandatory** |
| MySQL | Table = snake_case, include id/create_time/update_time | **Mandatory** |
| MySQL | No SELECT *, parameterized queries | **Mandatory** |
| Security | SQL injection: use PreparedStatement / MyBatis #{} | **Mandatory** |
| Security | Mask PII in logs and responses | **Mandatory** |
| Security | Validate input server-side | **Mandatory** |

---

*Reference: 阿里巴巴Java开发手册 (泰山版) — published by Alibaba Group. This SKILL.md is a condensed reference with distilled rules and examples for daily use.*
