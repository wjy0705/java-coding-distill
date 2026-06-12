---
name: google-guava-patterns
description: Google Guava团队Java编码模式蒸馏 - 生产级Java集合、并发、工具链最佳实践
---

# Google Guava 编码模式

> Google Guava 是 Google 核心 Java 库的开源版本（Apache 2.0）。主要贡献者：Kevin Bourrillion, Jared Levy, Louis Wasserman, 以及 Google Java 团队。Guava 代表着 Google 内部 Java 编码实践的精华。

---

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **Google Guava 团队** 的 Java 编码模式与工具链最佳实践。Guava 是 Google 核心 Java 库的开源版本（Apache 2.0），由 Kevin Bourrillion、Jared Levy、Louis Wasserman 等 Google Java 团队核心成员主导开发。Guava 代表着 Google 内部 Java 编码实践的精华，包含了 Google 工程师多年积累的集合、并发、缓存、IO、哈希等领域的工程智慧。

### 参考来源
- **开源代码**: Google/guava - GitHub 核心仓库源码分析（Apache 2.0 协议）
- **Google 内部实践**: Guava 代码库本身就是 Google 内部 Java 编码标准的外部体现
- **文档**: Guava Wiki、Google GitHub、Official Release Notes
- **社区验证**: 大量生产项目对 Guava 模式的使用验证（如 Spring、Android 等）

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：分析 Guava 核心源码 → 提炼 27 个核心编码模式 → 标注每个模式的状态建议（✅推荐/⚠️需迁移） → 提供代码示例对比 → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 写集合操作时 | ImmutableList 还是 ArrayList？Multimap 还是 Map<k, List>？照着模式选 |
| 需要缓存时 | CacheBuilder + LoadingCache 一行搞定本地缓存，不用手写 ConcurrentHashMap 逻辑 |
| 需要限流/分段锁时 | RateLimiter 令牌桶限流、Striped 分段锁，生产级实现直接用 |
| 处理并发 Future 时 | ListenableFuture + Futures.transform 链式回调，比 CompletableFuture 更可控 |
| 写工具类代码时 | Joiner/Splitter/CharMatcher 一行替代 10 行手写循环 |
| 面试时 | "用过 Guava 的哪些特性？"——从 Immutable 到 BloomFilter 逐条讲清楚 |

---

## 目录

1. [不可变集合](#1-不可变集合-immutable-collections)
2. [Optional（空值处理）](#2-optional空值处理)
3. [前置条件检查（Preconditions）](#3-前置条件检查-preconditions)
4. [MoreObjects 与 ToStringHelper](#4-moreobjects-与-tostringhelper)
5. [排序器（Ordering）](#5-排序器-ordering)
6. [函数式模式（FluentIterable）](#6-函数式模式-fluentiterable)
7. [ListenableFuture 与并发](#7-listenablefuture-与并发)
8. [RateLimiter（限流器）](#8-ratelimiter限流器)
9. [Striped 锁](#9-striped-锁)
10. [Service 框架](#10-service-框架)
11. [缓存（CacheBuilder）](#11-缓存-cachebuilder)
12. [Multiset（多集合）](#12-multiset多集合)
13. [Multimap（多值映射）](#13-multimap多值映射)
14. [BiMap（双向映射）](#14-bimap双向映射)
15. [Table（表格结构）](#15-table表格结构)
16. [RangeSet 与 RangeMap](#16-rangeset-与-rangemap)
17. [Joiner（连接器）](#17-joiner连接器)
18. [Splitter（分割器）](#18-splitter分割器)
19. [CharMatcher（字符匹配器）](#19-charmatcher字符匹配器)
20. [Strings 工具类](#20-strings-工具类)
21. [I/O：ByteSource / CharSource](#21-io-bytesource--charsource)
22. [Files 工具](#22-files-工具)
23. [Hashing（哈希）与 BloomFilter](#23-hashing哈希与-bloomfilter)
24. [原始类型工具（Ints / Longs / Doubles）](#24-原始类型工具ints--longs--doubles)
25. [EventBus（事件总线）](#25-eventbus事件总线)
26. [须避免的模式](#26-须避免的模式)
27. [Maven / Gradle 依赖](#27-maven--gradle-依赖)

---

## 1. 不可变集合（Immutable Collections）

### 模式名称
不可变集合工厂方法

### 何时使用
- 任何不需要（且不应）被修改的集合
- 作为 public API 的返回值，保证调用方不会修改内部数据
- 常量、配置、查找表
- 函数式 pipeline 的最终结果

### 代码示例

```java
// ✅ GOOD — Guava 不可变集合
import com.google.common.collect.ImmutableList;
import com.google.common.collect.ImmutableMap;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.ImmutableBiMap;

// 创建方式1：of() 工厂（0~多个元素）
ImmutableList<String> names = ImmutableList.of("Alice", "Bob", "Charlie");
ImmutableSet<Integer> ids = ImmutableSet.of(1, 2, 3);
ImmutableMap<String, Integer> ageMap =
    ImmutableMap.of("Alice", 30, "Bob", 25);

// 创建方式2：builder()（超过5个元素或不确定数量）
ImmutableList<String> list = ImmutableList.<String>builder()
    .add("first")
    .addAll(otherCollection)
    .add("last")
    .build();

// 创建方式3：copyOf()（从可变集合复制防御性拷贝）
List<String> mutable = new ArrayList<>();
mutable.add("a");
ImmutableList<String> copy = ImmutableList.copyOf(mutable);

// ❌ BAD — JDK不可变包装（运行时抛异常，无编译时保证）
List<String> bad = Collections.unmodifiableList(mutable);
// 如果 mutable 被其他地方修改，bad 的内容也跟着变！

// ✅ 不可变Map的构建
ImmutableMap<String, Config> configs = ImmutableMap.<String, Config>builder()
    .put("database", dbConfig)
    .put("cache", cacheConfig)
    .buildKeepingLast()  // Guava 31+，遇到重复key保留最后一个而非抛异常
    .build();
```

### 为什么更好
| 特性 | Guava Immutable* | Collections.unmodifiable* |
|------|------------------|---------------------------|
| 编译时类型安全 | ✅ 返回不可变类型 | ❌ 返回普通 List/Map |
| 防御性拷贝 | ✅ copyOf 是快照 | ❌ 只是视图，源变它也变 |
| 拒绝 null | ✅ | ✅ |
| 内存效率 | ✅ 专用实现，内存更小 | ❌ 包装模式，内存浪费 |
| CPU 效率 | ✅ 不可变=线程安全无需同步 | ❌ |
| 迭代顺序 | ✅ 插入顺序 | ❌ 取决于源集合 |

**原则：永远倾向于 ImmutableList/ImmutableSet/ImmutableMap 作为 API 返回类型和字段类型。**

---

## 2. Optional（空值处理）

### 模式名称
Guava Optional（与 JDK Optional 的选择）

### 何时使用
- 方法返回值可能为空，需要调用方显式处理
- 链式调用中避免 NullPointerException

### 代码示例

```java
import com.google.common.base.Optional;  // Guava Optional

// ✅ GOOD — 用 Optional 代替 null 返回值
public Optional<Account> findAccount(String id) {
    Account result = db.lookup(id);
    return Optional.fromNullable(result);  // result == null → absent
}

// 调用方
Optional<Account> maybe = service.findAccount("123");
if (maybe.isPresent()) {
    Account a = maybe.get();
}

// 默认值
Account a = maybe.or(defaultAccount);
Account b = maybe.orNull();

// 转换（Guava Optional 不支持 map，但可以）
if (maybe.isPresent()) {
    String name = maybe.get().getName();
}
```

### ⚠️ Guava Optional vs JDK Optional

| 维度 | Guava Optional | JDK Optional（Java 8+） |
|------|---------------|------------------------|
| Serializable | ✅ 是 | ✅ 是 |
| map/flatMap | ❌ 不支持 | ✅ 支持函数式链 |
| 推荐度（Java 8+） | ❌ 用 JDK 的 | ✅ **首选** |
| 推荐度（Java 7） | ✅ 首选 | ❌ 不存在 |

```java
// ✅ Java 8+ 建议用 JDK Optional + Stream
import java.util.Optional;

Optional<User> user = findUser(id);
String name = user.map(User::getName)
    .orElse("Unknown");

// Guava Optional 在 Java 8+ 的唯一合理用法：
// 如果你的代码库还在用 Guava Optional，不要混用两种 Optional
// 应统一迁移到 JDK Optional
```

### 新模式（Guava 32+）
```java
// ✅ 如果必须用 Guava Optional，用 transform 模拟 map
Optional<String> upper = optional.transform(String::toUpperCase);
Optional<Integer> len = optional.transform(String::length);
```

---

## 3. 前置条件检查（Preconditions）

### 模式名称
方法参数校验

### 何时使用
- 任何 public/protected 方法的参数合法性校验
- 构造函数参数校验
- 状态检查

### 代码示例

```java
import static com.google.common.base.Preconditions.*;

public class AccountService {
    
    public void transfer(Account from, Account to, BigDecimal amount) {
        // ✅ GOOD — 清晰的参数校验
        checkNotNull(from, "source account must not be null");
        checkNotNull(to, "target account must not be null");
        checkNotNull(amount, "amount must not be null");
        checkArgument(amount.compareTo(BigDecimal.ZERO) > 0,
            "amount must be positive, got: %s", amount);
        
        // ❌ BAD — 手写 if-throw 啰嗦且不统一
        // if (from == null) throw new NullPointerException("from is null");
        // if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) ...
    }

    public void setState(String state) {
        // 状态检查
        checkState(initialized, "service not initialized yet");
    }
    
    // 索引检查
    public User getUser(int index) {
        checkElementIndex(index, users.size(),
            "user index out of bounds");
        return users.get(index);
    }
    
    public void setPosition(int pos, int total) {
        checkPositionIndex(pos, total,
            "position must be between 0 and %s (inclusive)", total);
    }
}
```

### Preconditions 方法速查

| 方法 | 抛异常 | 当条件 |
|------|--------|--------|
| `checkArgument(expr)` | `IllegalArgumentException` | expr == false |
| `checkNotNull(ref)` | `NullPointerException` | ref == null |
| `checkState(expr)` | `IllegalStateException` | expr == false |
| `checkElementIndex(i, size)` | `IndexOutOfBoundsException` | i < 0 \|\| i >= size |
| `checkPositionIndex(i, size)` | `IndexOutOfBoundsException` | i < 0 \|\| i > size |
| `checkPositionIndexes(start, end, size)` | `IndexOutOfBoundsException` | 区间非法 |

**所有方法都支持 `%s` 格式化占位符。**

---

## 4. MoreObjects 与 ToStringHelper

### 模式名称
优雅的 toString()

### 何时使用
- 任何需要 toString() 的 POJO 或 DTO
- 替代手写字符串拼接或 Apache Commons ToStringBuilder

### 代码示例

```java
import com.google.common.base.MoreObjects;

public class User {
    private String name;
    private int age;
    private String email;
    
    @Override
    public String toString() {
        // ✅ GOOD — 简洁、可读、自动处理 null
        return MoreObjects.toStringHelper(this)
            .add("name", name)
            .add("age", age)
            .add("email", email)
            .omitNullValues()  // 跳过 null 字段
            .toString();
    }
    
    // 输出: "User{name=Alice, age=30, email=alice@example.com}"
    
    // ❌ BAD — 手写
    // return "User{name=" + name + ", age=" + age + ", email=" + email + "}";
}
```

### MoreObjects.firstNonNull
```java
// 从两个候选值中选第一个非 null 的
String result = MoreObjects.firstNonNull(nullable, fallback);
// ⚠️ 两个都为 null 则抛 NullPointerException
```

---

## 5. 排序器（Ordering）

### 模式名称
Guava Ordering（已标记为 @Beta，建议迁移）

### 何时使用
- Java 7 及以下需要复杂排序逻辑
- 链式组合多个排序条件

### ⚠️ 注意
Guava 的 `Ordering` 类已标记 `@Beta` 多年，在 Java 8+ 中基本被 `Comparator` 接口的默认方法取代。**新代码建议直接用 Java 8 Comparator。**

```java
// ✅ Java 8+ — 用 Comparator
Comparator<User> byName = Comparator.comparing(User::getName);
Comparator<User> byAge = Comparator.comparingInt(User::getAge);
Comparator<User> compound = byName.thenComparing(byAge);
users.sort(compound);

// ❌ 旧代码（Guava）— 仅 Java 7 迁移场景保留
Ordering<User> ordering = Ordering.natural()
    .nullsFirst()
    .onResultOf(User::getName);
Collections.sort(users, ordering);
```

### 仍有用的 Ordering 模式
```java
// 求最大值/最小值（简洁）
List<Integer> scores = Arrays.asList(85, 92, 78, null, 95);
int max = Ordering.natural().nullsLast().max(scores);  // 95

// 判断是否有序
boolean sorted = Ordering.natural().isOrdered(list);
```

---

## 6. 函数式模式（FluentIterable）

### 模式名称
流畅链式集合操作

### 何时使用
- Java 7 或以下需要对集合做 chain-filter-map 操作
- **Java 8+ 建议用 Stream API** 代替

### 代码示例

```java
import com.google.common.collect.FluentIterable;
import com.google.common.base.Function;
import com.google.common.base.Predicate;

// ❌ 仅用于 Java 7 遗留代码的迁移
// Java 8 请用 stream()

// Java 7 风格
FluentIterable<User> adults = FluentIterable.from(users)
    .filter(new Predicate<User>() {
        @Override public boolean apply(User u) {
            return u.getAge() >= 18;
        }
    })
    .transform(new Function<User, String>() {
        @Override public String apply(User u) {
            return u.getName();
        }
    });

// ✅ Java 8+ 模式
List<String> adultNames = users.stream()
    .filter(u -> u.getAge() >= 18)
    .map(User::getName)
    .collect(Collectors.toList());
```

### Functions / Predicates / Suppliers

```java
import com.google.common.base.Functions;
import com.google.common.base.Predicates;
import com.google.common.base.Suppliers;

// Functions — 函数组合
Function<String, Integer> lengthFunc = String::length;
Function<Integer, String> toStringFunc = String::valueOf;
Function<String, String> composed = Functions.compose(toStringFunc, lengthFunc);
// composed.apply("hello") → "5"

// Predicates — 谓词组合
Predicate<String> notEmpty = s -> s != null && !s.isEmpty();
Predicate<String> lengthGt3 = s -> s.length() > 3;
Predicate<String> both = Predicates.and(notEmpty, lengthGt3);

// Suppliers — 提供者模式（惰性求值）
Supplier<ExpensiveObject> lazy = Suppliers.memoize(
    () -> new ExpensiveObject()  // 只在第一次 get() 时创建
);
// 后续 get() 返回缓存实例

// Suppliers.memoizeWithExpiration(..., Duration, TimeUnit) — 带过期
Supplier<Config> config = Suppliers.memoizeWithExpiration(
    () -> loadConfigFromDB(),
    5, TimeUnit.MINUTES
);
```

---

## 7. ListenableFuture 与并发

### 模式名称
可监听 Future

### 何时使用
- 任何异步计算，特别是需要回调/组合的
- 替代 `Future.get()` 的阻塞模式
- 多个异步操作的编排（并行、串行、聚合）

### 代码示例

```java
import com.google.common.util.concurrent.*;
import java.util.concurrent.*;

// 创建 ListeningExecutorService
ListeningExecutorService service = MoreExecutors
    .listeningDecorator(Executors.newFixedThreadPool(10));

// ✅ GOOD — 异步执行 + 回调
ListenableFuture<Image> future = service.submit(() -> fetchImage(url));

// 回调（Callback 在完成时触发）
Futures.addCallback(future, new FutureCallback<Image>() {
    @Override public void onSuccess(Image result) {
        display(result);
    }
    @Override public void onFailure(Throwable t) {
        logger.error("Failed to fetch image", t);
    }
}, MoreExecutors.directExecutor());  // 在完成线程执行回调

// ✅ 组合（transform）
ListenableFuture<String> nameFuture = service.submit(() -> getUser(id));
ListenableFuture<Integer> ageFuture = Futures.transform(
    nameFuture,
    name -> getAgeFromName(name),  // Function<String, Integer>
    service  // 转换函数执行的 executor
);

// ✅ 异步转换（transformAsync）
ListenableFuture<Profile> profileFuture = Futures.transformAsync(
    nameFuture,
    name -> service.submit(() -> loadProfile(name)),  // 返回 ListenableFuture
    service
);

// ✅ 聚合
ListenableFuture<List<Object>> allFutures = Futures.allAsList(f1, f2, f3);
ListenableFuture<Object> successful = Futures.successfulAsList(f1, f2, f3);

// ✅ 并行处理 + 等待全部完成
List<ListenableFuture<Result>> futures = tasks.stream()
    .map(task -> service.submit(() -> process(task)))
    .collect(Collectors.toList());

ListenableFuture<List<Result>> all = Futures.allAsList(futures);
Futures.addCallback(all, ... callback ..., executor);
```

### SettableFuture

```java
// 手动控制 Future 的完成
SettableFuture<String> future = SettableFuture.create();

// 在其他线程或回调中
future.set("result");        // 正常完成
// 或
future.setException(ex);     // 异常完成
// 或
future.cancel(true);         // 取消

// 这是实现异步方法返回值的好工具
public ListenableFuture<String> asyncOperation() {
    SettableFuture<String> result = SettableFuture.create();
    startAsyncProcess(result::set, result::setException);
    return result;
}
```

### Futures 方法速查

| 方法 | 说明 |
|------|------|
| `transform(f, fn, exec)` | 同步转换输出 |
| `transformAsync(f, fn, exec)` | 异步转换输出 |
| `allAsList(f1, f2...)` | 所有 Future 完成，返回 List |
| `successfulAsList(f1, f2...)` | 所有完成（无论成功与否） |
| `addCallback(f, cb, exec)` | 注册成功/失败回调 |
| `withTimeout(f, timeout, exec)` | 超时包装 |
| `immediateFuture(v)` | 创建已完成的 Future |
| `immediateFailedFuture(t)` | 创建已失败的 Future |
| `immediateCancelledFuture()` | 创建已取消的 Future |

---

## 8. RateLimiter（限流器）

### 模式名称
令牌桶限流

### 何时使用
- 保护后端服务不被突发流量打垮
- 控制 API 调用频率
- 数据库连接限制
- 节流外部服务调用

### 代码示例

```java
import com.google.common.util.concurrent.RateLimiter;

// ✅ GOOD — 每秒最多 100 个请求
RateLimiter limiter = RateLimiter.create(100.0);

// 方式1：阻塞式（简单）
public void handleRequest(Request req) {
    limiter.acquire();  // 等待直到获得许可，可能阻塞
    process(req);
}

// 方式2：非阻塞式（更优雅）
public boolean tryHandle(Request req) {
    if (limiter.tryAcquire(100, TimeUnit.MILLISECONDS)) {
        process(req);
        return true;
    }
    return false;  // 限流拒绝
}

// 方式3：批量
limiter.acquire(10);  // 批量获取10个许可

// ✅ 突发流量控制 — 预热期（避免冷启动时打满）
RateLimiter warmup = RateLimiter.create(1000, 10, TimeUnit.SECONDS);
// 前10秒逐渐从 1/3 速率上升到全速

// ❌ BAD — 用简单的 Thread.sleep 节流
// Thread.sleep(10);  // 不均匀、不准确
```

### 实际应用模式

```java
// 限流 + 降级
public class ThrottledClient {
    private final RateLimiter limiter = RateLimiter.create(50);
    private final HttpClient client;
    
    public Response call(String url) {
        if (!limiter.tryAcquire()) {
            // 限流降级：返回缓存、空结果或抛出友好异常
            return cachedResponse(url);
        }
        return client.get(url);
    }
}
```

---

## 9. Striped 锁

### 模式名称
分段锁 / 分片锁

### 何时使用
- 需要按 key 加锁（user_id, order_id 等）
- 避免对整个数据结构加全局锁
- 无法使用 ConcurrentHashMap 的场景（如需要原子地操作多个数据结构）
- 比 ConcurrentHashMap 的 computeIfAbsent 更精细的锁控制

### 代码示例

```java
import com.google.common.util.concurrent.Striped;
import java.util.concurrent.locks.Lock;

// ✅ GOOD — 按用户 ID 分片锁
public class UserAccountService {
    private final Striped<Lock> locks = Striped.lock(1024);
    // 1024 个锁，取模分配。内存开销远比 ConcurrentHashMap 小
    
    public void updateBalance(String userId, BigDecimal delta) {
        Lock lock = locks.get(userId);  // 同一 userId 获取同一把锁
        lock.lock();
        try {
            Account acc = findAccount(userId);
            acc.setBalance(acc.getBalance().add(delta));
            save(acc);
        } finally {
            lock.unlock();
        }
    }
}

// ✅ 读写锁模式
private final Striped<ReadWriteLock> rwLocks = Striped.readWriteLock(256);

public void read(String key) {
    Lock rLock = rwLocks.get(key).readLock();
    rLock.lock();
    try { /* read */ } finally { rLock.unlock(); }
}

public void write(String key) {
    Lock wLock = rwLocks.get(key).writeLock();
    wLock.lock();
    try { /* write */ } finally { wLock.unlock(); }
}

// ❌ BAD — 对整个方法加锁
// public synchronized void updateBalance(...)
// → 所有用户互相阻塞！
```

### Striped 方法速查

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `Striped.lock(n)` | `Striped<Lock>` | n 个普通锁 |
| `Striped.readWriteLock(n)` | `Striped<ReadWriteLock>` | n 个读写锁 |
| `Striped.semaphore(n, permits)` | `Striped<Semaphore>` | n 个信号量 |
| `lazyWeak()` | 弱引用 | 无引用时自动回收，适合大量短命 key |

---

## 10. Service 框架

### 模式名称
服务生命周期管理

### 何时使用
- 组件有明确的生命周期（启动→运行→停止）
- 需要优雅启动/停止、健康检查
- 服务之间有依赖关系（先启动 A 再启动 B）

### 代码示例

```java
import com.google.common.util.concurrent.*;
import com.google.common.util.concurrent.Service.State;

// ✅ GOOD — 实现一个 Service
public class DatabaseService extends AbstractExecutionThreadService {
    
    private volatile boolean running = true;
    
    @Override
    protected void run() throws Exception {
        while (running) {
            // 执行主要工作
            processEvents();
        }
    }
    
    @Override
    protected void startUp() {
        // 初始化连接池
        log.info("DatabaseService started");
    }
    
    @Override
    protected void shutDown() {
        // 释放资源
        log.info("DatabaseService stopped");
    }
    
    @Override
    protected void triggerShutdown() {
        running = false;  // 让 run() 循环优雅退出
    }
}

// 服务类型选择
// ┌──────────────────────┬────────────────────────────────────┐
// │ 基类                  │ 适用场景                           │
// ├──────────────────────┼────────────────────────────────────┤
// │ AbstractService       │ 任何需要自定义启动/停止逻辑的服务 │
// │ AbstractIdleService   │ 只有 startUp/shutDown，没有持续线程 │
// │ AbstractExecution…    │ 有持续运行的 run() 循环            │
// │ AbstractScheduled…    │ 定时任务（已废弃，用 ScheduledExecutor）│
// └──────────────────────┴────────────────────────────────────┘

// ✅ 服务管理器
public class Application {
    public static void main(String[] args) {
        ServiceManager manager = new ServiceManager(
            ImmutableList.of(
                new DatabaseService(),
                new CacheService(),
                new HttpServer()
            )
        );
        
        manager.addListener(new ServiceManager.Listener() {
            @Override
            public void healthy() {
                log.info("All services are healthy");
            }
            @Override
            public void failure(Service service) {
                log.error("Service {} failed", service);
            }
        }, MoreExecutors.directExecutor());
        
        // 按依赖顺序启动，反向顺序停止
        manager.startAsync().awaitHealthy();
        
        // ... 运行中 ...
        
        manager.stopAsync().awaitStopped();
    }
}
```

### 服务状态图

```
  [NEW] ──start──> [STARTING] ──> [RUNNING]
                             └──> [STOPPING] ──> [TERMINATED]
  [NEW] ──stop──> [STOPPING] ──────────────────> [TERMINATED]
                     任何状态失败 ──> [FAILED]
```

---

## 11. 缓存（CacheBuilder）

### 模式名称
本地缓存构建

### 何时使用
- 需要进程内缓存而不是 Redis/Memcached
- 数据读取开销大（DB 查询、远程调用）
- 数据具有一定时效性
- 需要自动回收、统计、并发控制

### 代码示例

```java
import com.google.common.cache.*;

// ✅ GOOD — LoadingCache（自动加载）
public class UserCache {
    private final LoadingCache<String, User> cache = CacheBuilder.newBuilder()
        .maximumSize(10_000)                     // 最大条目数
        .expireAfterWrite(10, TimeUnit.MINUTES)  // 写入后过期
        .expireAfterAccess(1, TimeUnit.HOURS)    // 访问后过期（可选）
        .refreshAfterWrite(5, TimeUnit.MINUTES)  // 过期前刷新（异步加载）
        .recordStats()                           // 开启缓存统计
        .removalListener(new RemovalListener<String, User>() {
            @Override
            public void onRemoval(RemovalNotification<String, User> n) {
                // 条目被移除时回调（可以用来释放资源）
                log.info("Evicted: {} cause: {}", n.getKey(), n.getCause());
            }
        })
        .build(new CacheLoader<String, User>() {
            @Override
            public User load(String key) throws Exception {
                return loadFromDatabase(key);  // 缓存未命中时自动加载
            }
        });
    
    public User get(String id) throws ExecutionException {
        return cache.get(id);  // 自动调用 CacheLoader.load()
    }
    
    public User getUnchecked(String id) {
        return cache.getUnchecked(id);  // 不抛 checked exception
    }
    
    public void invalidate(String id) {
        cache.invalidate(id);  // 手动失效
    }
    
    public void put(String id, User user) {
        cache.put(id, user);   // 手动写缓存
    }
}

// ✅ Callable 缓存（无需预定义 CacheLoader）
Cache<String, User> cache = CacheBuilder.newBuilder()
    .maximumSize(10_000)
    .build();

User user = cache.get(key, () -> loadFromDatabase(key));

// ✅ 缓存统计
CacheStats stats = cache.stats();
log.info("hit rate: {}, miss count: {}, avg load: {} ns",
    stats.hitRate(),
    stats.missCount(),
    stats.averageLoadPenalty());

// ✅ 异步刷新
// refreshAfterWrite 不会阻塞请求，而是触发异步刷新
// 旧值在刷新完成前仍然可用
```

### CacheBuilder 参数速查

| 参数 | 说明 |
|------|------|
| `maximumSize(n)` | 最大条目数，LRU 淘汰 |
| `maximumWeight(n)` | 按权重淘汰 |
| `expireAfterWrite(d, u)` | 写入后固定时间过期 |
| `expireAfterAccess(d, u)` | 最后一次访问后过期 |
| `refreshAfterWrite(d, u)` | 写入后预刷新（避免瞬间大量加载） |
| `weakKeys()` / `weakValues()` | 弱引用，GC 可回收 |
| `softValues()` | 软引用，GC 按需回收 |
| `recordStats()` | 开启统计 |
| `removalListener(l)` | 移除监听器 |
| `concurrencyLevel(n)` | 并发级别（默认 4） |

---

## 12. Multiset（多集合）

### 模式名称
可计数集合

### 何时使用
- 需要统计元素出现次数
- 取代 `Map<E, Integer>` 的手动计数模式（更简洁）
- 词频统计、日志分析、数据聚合

### 代码示例

```java
import com.google.common.collect.HashMultiset;
import com.google.common.collect.Multiset;

// ✅ GOOD — 用 Multiset 计数
Multiset<String> wordCounts = HashMultiset.create();

// 添加元素
wordCounts.add("apple");
wordCounts.add("banana");
wordCounts.add("apple");
wordCounts.add("apple", 3);  // 再加 3 次

// 查询
int count = wordCounts.count("apple");    // 4
int total = wordCounts.size();            // 6 (总条数)

// 遍历（不重复元素）
for (Multiset.Entry<String> entry : wordCounts.entrySet()) {
    log.info("{}: {}", entry.getElement(), entry.getCount());
}

// 添加一个集合中的所有元素
wordCounts.addAll(Arrays.asList("apple", "orange", "banana"));

// ❌ BAD — 手动维护 Map 计数
// Map<String, Integer> counts = new HashMap<>();
// counts.merge(key, 1, Integer::sum);  // 啰嗦，易错

// ✅ 实现类型选择
// HashMultiset      — 无序，高性能（最常用）
// TreeMultiset      — 有序（按自然顺序或 Comparator）
// LinkedHashMultiset — 插入顺序
// ConcurrentHashMultiset — 线程安全

// ✅ 不可变版本
ImmutableMultiset<String> immutable = Multisets.unmodifiableMultiset(set);
```

---

## 13. Multimap（多值映射）

### 模式名称
一对多映射

### 何时使用
- 一个 key 关联多个值
- 取代 `Map<K, List<V>>` 或 `Map<K, Set<V>>` 的啰嗦模式
- 分组操作（按部门分组、按标签分组）

### 代码示例

```java
import com.google.common.collect.*;

// ✅ GOOD — ArrayListMultimap
Multimap<String, String> tags = ArrayListMultimap.create();

tags.put("java", "spring");
tags.put("java", "hibernate");
tags.put("java", "guava");
tags.put("python", "flask");

// 查询
Collection<String> javaTags = tags.get("java");     // [spring, hibernate, guava]
int size = tags.size();                             // 4（总键值对数）

// 移除
tags.remove("java", "guava");                       // 移除一个
tags.removeAll("python");                           // 移除 key 所有值

// 替换
Iterable<String> newValues = Arrays.asList("spring-boot", "micronaut");
tags.replaceValues("java", newValues);

// ✅ 实现类型选择
// ┌────────────────────┬──────────┬──────────────┬────────────┐
// │ 实现                │ 值集合    │ key 有序     │ 线程安全    │
// ├────────────────────┼──────────┼──────────────┼────────────┤
// │ ArrayListMultimap   │ List     │ 否           │ 否         │
// │ HashMultimap        │ Set      │ 否           │ 否         │
// │ LinkedHashMultimap  │ Set      │ 插入顺序     │ 否         │
// │ TreeMultimap        │ Set      │ 自然排序     │ 否         │
// │ ImmutableMultimap   │ List     │ 是(不可变)   │ 是         │
// └────────────────────┴──────────┴──────────────┴────────────┘

// ❌ BAD — 手动 List 映射
// Map<String, List<String>> bad = new HashMap<>();
// bad.computeIfAbsent(key, k -> new ArrayList<>()).add(value);

// ✅ 按部门分组人员
Multimap<String, Employee> byDept = ArrayListMultimap.create();
for (Employee e : employees) {
    byDept.put(e.getDepartment(), e);
}
```

---

## 14. BiMap（双向映射）

### 模式名称
双射映射（key ↔ value 唯一）

### 何时使用
- 需要同时按 key 和按 value 查询
- value 也必须唯一（不像普通 Map 允许 value 重复）
- 枚举双向查找、ID ↔ Name 互转、编码表

### 代码示例

```java
import com.google.common.collect.*;

// ✅ GOOD — BiMap 保证 value 唯一
BiMap<String, Integer> nameToId = HashBiMap.create();

nameToId.put("Alice", 1001);
nameToId.put("Bob", 1002);
// nameToId.put("Charlie", 1001);  // ⚠️ 抛 IllegalArgumentException！
                                    // value 1001 已存在

// 强制覆盖
nameToId.forcePut("Charlie", 1001);  // 删除 Alice 的映射

// 双向查询
Integer id = nameToId.get("Alice");      // key → value: 1001
String name = nameToId.inverse().get(1001);  // value → key: "Alice"

// 反向视图（与原始 BiMap 联动）
BiMap<Integer, String> idToName = nameToId.inverse();
idToName.put(1003, "David");            // 同步到 nameToId!

// ❌ BAD — 维护两个 Map 手动同步
// Map<String, Integer> nameToId = new HashMap<>();
// Map<Integer, String> idToName = new HashMap<>();
// // 手动保持同步，很容易不一致

// ✅ 实现
// HashBiMap       — 哈希实现（最常用）
// ImmutableBiMap  — 不可变版本

// ✅ 枚举查找
public enum Status {
    ACTIVE("A"), INACTIVE("I"), PENDING("P");
    
    private static final BiMap<String, Status> CODE_MAP =
        ImmutableBiMap.copyOf(
            Arrays.stream(values())
                .collect(Collectors.toMap(s -> s.code, s -> s))
        );
    
    private final String code;
    
    public static Status fromCode(String code) {
        return CODE_MAP.get(code);
    }
    public String toCode() { return code; }
}
```

---

## 15. Table（表格结构）

### 模式名称
二维表格数据结构（行 × 列）

### 何时使用
- 数据有行和列两个维度
- 取代 `Map<R, Map<C, V>>` 的嵌套映射（N 倍简洁）
- 矩阵、Excel 风格数据、二维索引

### 代码示例

```java
import com.google.common.collect.*;

// ✅ GOOD — Table 取代嵌套 Map
Table<String, String, BigDecimal> salaryTable = HashBasedTable.create();

// 填充数据：行=员工姓名, 列=月份, 值=工资
salaryTable.put("Alice", "2024-01", new BigDecimal("10000"));
salaryTable.put("Alice", "2024-02", new BigDecimal("11000"));
salaryTable.put("Bob", "2024-01", new BigDecimal("9000"));
salaryTable.put("Bob", "2024-02", new BigDecimal("9500"));

// 查询
BigDecimal aliceJan = salaryTable.get("Alice", "2024-01");  // 10000

// 按行/列获取
Map<String, BigDecimal> aliceRow = salaryTable.row("Alice");
Map<String, BigDecimal> janColumn = salaryTable.column("2024-01");

// 所有行/列/值
Set<String> employees = salaryTable.rowKeySet();    // [Alice, Bob]
Set<String> months = salaryTable.columnKeySet();    // [2024-01, 2024-02]

// 遍历
for (Table.Cell<String, String, BigDecimal> cell : salaryTable.cellSet()) {
    log.info("{} earned {} in {}", 
        cell.getRowKey(), cell.getValue(), cell.getColumnKey());
}

// ✅ 实现选择
// ┌─────────────────┬─────────────────────────────────┐
// │ 实现             │ 特点                            │
// ├─────────────────┼─────────────────────────────────┤
// │ HashBasedTable   │ 基于 HashMap, 无序（最常用）    │
// │ TreeBasedTable   │ 行/列有序                       │
// │ ArrayTable       │ 固定行列集合，内存最优          │
// │ ImmutableTable   │ 不可变                          │
// └─────────────────┴─────────────────────────────────┘

// ❌ BAD — 嵌套 Map
// Map<String, Map<String, BigDecimal>> nested = new HashMap<>();
// nested.computeIfAbsent("Alice", k -> new HashMap<>()).put("2024-01", value);
// // 查询啰嗦，检查 null，容易 NPE
```

---

## 16. RangeSet 与 RangeMap

### 模式名称
区间集合与区间映射

### 何时使用
- 处理连续区间（IP 范围、时间范围、数值范围、版本号范围）
- 区间合并、交集、包含性检查
- 区间到值的映射

### 代码示例

```java
import com.google.common.collect.*;

// ======================== RangeSet ========================

// ✅ GOOD — RangeSet 管理不重叠区间
RangeSet<Integer> rangeSet = TreeRangeSet.create();

rangeSet.add(Range.closed(1, 10));    // [1, 10]
rangeSet.add(Range.closed(11, 20));   // [1, 20] ← 自动合并相邻区间
rangeSet.add(Range.closed(15, 25));   // [1, 25] ← 与 [11, 20] 重叠合并

// 查询
rangeSet.contains(12);   // true
rangeSet.contains(30);   // false

// 区间操作
RangeSet<Integer> complement = rangeSet.complement();  // (-∞, 1) ∪ (25, +∞)
RangeSet<Integer> intersect = rangeSet.intersection(
    Range.closed(5, 15));  // [5, 15]

// 遍历
for (Range<Integer> r : rangeSet.asRanges()) {
    log.info("Range: {}", r);  // [1, 25]
}

// ======================== RangeMap ========================

// ✅ GOOD — 区间到值的映射
RangeMap<Integer, String> versionMap = TreeRangeMap.create();

versionMap.put(Range.closedOpen(1, 3), "v1");
versionMap.put(Range.closedOpen(3, 5), "v2");
versionMap.put(Range.closedOpen(5, 10), "v3");

// 查询
versionMap.get(2);  // "v1"
versionMap.get(4);  // "v2"

// 区间会分裂（如果新区间覆盖部分旧区间）
versionMap.put(Range.closedOpen(4, 6), "v2.5");
// [1, 3) → "v1"
// [3, 4) → "v2"
// [4, 6) → "v2.5"
// [6, 10) → "v3"

// ❌ BAD — 手写 if-else 链
// if (version >= 1 && version < 3) return "v1";
// else if (version >= 3 && version < 5) return "v2";
// // 不好维护、不易扩展

// ✅ 典型用途：IP 地址范围（用 Long 表示）
RangeMap<Long, String> ipRangeMap = TreeRangeMap.create();
ipRangeMap.put(Range.closedOpen(ipToLong("192.168.0.0"), 
                                 ipToLong("192.168.255.255")), "internal");
```

### Range 端点表示

| 端点 | 符号 | 含义 | Range 示例 |
|------|------|------|-----------|
| `(a..b)` | open | 两端不包含 | `Range.open(1, 10)` → 2..9 |
| `[a..b]` | closed | 两端包含 | `Range.closed(1, 10)` → 1..10 |
| `[a..b)` | closedOpen | 左闭右开 | `Range.closedOpen(1, 10)` → 1..9 |
| `(a..b]` | openClosed | 左开右闭 | `Range.openClosed(1, 10)` → 2..10 |
| `(a..+∞)` | greaterThan | 下界无上界 | `Range.greaterThan(0)` |
| `(-∞..b)` | lessThan | 有上界无下界 | `Range.lessThan(100)` |
| `(-∞..+∞)` | all | 所有值 | `Range.all()` |

---

## 17. Joiner（连接器）

### 模式名称
优雅的字符串连接

### 何时使用
- 将集合/数组连接为分隔字符串
- 跳过或替换 null 值
- 追加到已有 StringBuilder

### 代码示例

```java
import com.google.common.base.Joiner;

// ✅ GOOD — 比手写循环更安全、更简洁
Joiner joiner = Joiner.on(", ").skipNulls();

List<String> parts = Arrays.asList("a", "b", null, "c");
String result = joiner.join(parts);  // "a, b, c"

// 替换 null 而不是跳过
Joiner joiner2 = Joiner.on(", ").useForNull("(null)");
result = joiner2.join(parts);  // "a, b, (null), c"

// 连接 Map
Joiner.MapJoiner mapJoiner = Joiner.on("&")
    .withKeyValueSeparator("=");
String params = mapJoiner.join(ImmutableMap.of(
    "name", "Alice", "age", "30"
));
// "name=Alice&age=30"

// 追加到 StringBuilder
StringBuilder sb = new StringBuilder("Prefix: ");
joiner.appendTo(sb, parts);

// ❌ BAD — 手写循环，末尾多余分隔符
// StringBuilder sb = new StringBuilder();
// for (String s : list) {
//     if (sb.length() > 0) sb.append(", ");
//     sb.append(s);
// }

// ⚠️ Joiner 是 immutable 的！链式调用不修改原对象
Joiner base = Joiner.on(",");
Joiner skipped = base.skipNulls();  // base 仍是未跳过 null 的 Joiner！
```

---

## 18. Splitter（分割器）

### 模式名称
健壮的分割操作

### 何时使用
- 字符串分割为集合
- 比 `String.split()` 更可控、更可预测
- 需要精确控制空字符串、修剪空白

### 代码示例

```java
import com.google.common.base.Splitter;

// ✅ GOOD — Splitter 击败 String.split()
String input = " foo , bar, , baz ";

// String.split() 的问题：
// String[] parts = input.split(",");  // [" foo ", " bar", " ", " baz "]
// 不修剪、不跳过空值、正则转义坑多

// Splitter 解决方案：
Iterable<String> parts = Splitter.on(",")
    .trimResults()           // 修剪空白
    .omitEmptyStrings()      // 跳过空字符串
    .split(input);
// → ["foo", "bar", "baz"]

// 固定长度分割
Iterable<String> fixed = Splitter.fixedLength(3)
    .split("abcdefgh");
// → ["abc", "def", "gh"]

// 正则分割
Iterable<String> regex = Splitter.onPattern("\\s+")
    .split("hello   world  foo");
// → ["hello", "world", "foo"]

// 限制结果数量
Iterable<String> limited = Splitter.on(",")
    .limit(3)
    .split("a,b,c,d,e");
// → ["a", "b", "c,d,e"]

// ❌ BAD — String.split()
// String[] arr = input.split(",");  // 正则编译开销，行为微妙

// ⚠️ Splitter 也是 immutable 的
Splitter base = Splitter.on(",");
Splitter trimmed = base.trimResults();  // base 不变
```

### String.split() vs Splitter

| 特性 | String.split() | Splitter |
|------|---------------|----------|
| 正则编译 | 每次调用都编译 | 一次构建，永久复用 |
| 空字符串处理 | 末尾空串自动丢弃 | 明确控制/跳过/保留 |
| 修剪 | ❌ 不支持 | ✅ trimResults() |
| 固定长度 | ❌ 不支持 | ✅ fixedLength() |
| 限制数量 | ❌ 不支持 | ✅ limit() |
| Map/List 结果 | ✅ 返回数组 | ✅ splitToList() 返回 List |

---

## 19. CharMatcher（字符匹配器）

### 模式名称
高级字符处理

### 何时使用
- 字符分类、匹配、替换
- 移除/保留/修剪特定类型的字符
- 替代手写 Character.isLetter() 等

### 代码示例

```java
import com.google.common.base.CharMatcher;

// ✅ GOOD — CharMatcher 替代手写字符过滤

// 预定义匹配器
CharMatcher.digit()          // 数字
CharMatcher.javaLetter()     // Java 字母
CharMatcher.whitespace()     // 空白字符
CharMatcher.is('x')          // 特定字符
CharMatcher.noneOf("abc")    // 不是 a/b/c
CharMatcher.any()           // 所有字符
CharMatcher.none()          // 没有字符

// 修饰方法
CharMatcher.digit().or(CharMatcher.is('-'))    // 数字或负号
CharMatcher.javaLetter().and(CharMatcher.is('x'))  // 既是字母又是 x
CharMatcher.digit().negate()                    // 非数字

// 实际应用
// 1. 移除所有非数字字符
String phone = "Tel: (010) 1234-5678";
String digits = CharMatcher.digit()
    .retainFrom(phone);  // "01012345678"

// 2. 修剪空白并合并连续空白
String messy = "  hello   world  ";
String clean = CharMatcher.whitespace()
    .trimAndCollapseFrom(messy, ' ');  // "hello world"

// 3. 替换
String masked = CharMatcher.javaDigit()
    .replaceFrom("My SSN: 123-45-6789", '*');
// "My SSN: ***-**-****"

// 4. 移除控制字符
String safe = CharMatcher.javaIsoControl()
    .removeFrom(rawInput);

// 5. 统计
int letterCount = CharMatcher.javaLetter().countIn(text);

// ❌ BAD — 逐个字符手写循环
// StringBuilder sb = new StringBuilder();
// for (char c : str.toCharArray()) {
//     if (Character.isLetter(c)) sb.append(c);
// }

// ✅ 性能：CharMatcher 是编译优化 + 位图实现
// 比手写循环快很多
```

---

## 20. Strings 工具类

### 模式名称
通用字符串辅助

### 何时使用
- 常见字符串操作但 Java 标准库没有提供
- 空/空串检查、补零、重复

### 代码示例

```java
import com.google.common.base.Strings;

// null/空串安全
Strings.isNullOrEmpty(str);       // true if str == null || str.equals("")
Strings.nullToEmpty(str);         // null → ""，其他保持
Strings.emptyToNull(str);         // "" → null，其他保持

// 补零/填充
String padded = Strings.padStart("123", 6, '0');   // "000123"
String padded2 = Strings.padEnd("123", 6, '0');    // "123000"

// 重复
String repeated = Strings.repeat("ab", 3);  // "ababab"

// 公共前缀/后缀
String a = "prefix_abc";
String b = "prefix_xyz";
String common = Strings.commonPrefix(a, b);  // "prefix_"
Strings.commonSuffix("abc.zip", "xyz.zip");  // ".zip"

// ❌ BAD — JDK 没有对应方法
// if (str == null || str.isEmpty()) { ... }
```

---

## 21. I/O：ByteSource / CharSource

### 模式名称
流的抽象工厂模式

### 何时使用
- 读写文件、网络流、内存数据
- 将 I/O 操作统一为 Source/Sink 抽象
- 需要复制、哈希、包装流

### 代码示例

```java
import com.google.common.io.*;

// ✅ GOOD — Source/Sink 抽象

// ===== CharSource（字符数据）=====
File file = new File("data.txt");
CharSource source = Files.asCharSource(file, Charsets.UTF_8);

// 读取全部
String content = source.read();

// 按行读取
List<String> lines = source.readLines();

// 复制到输出
CharSink sink = Files.asCharSink(new File("copy.txt"), Charsets.UTF_8);
source.copyTo(sink);

// 链式操作
long lineCount = source.readLines().size();

// ===== ByteSource（二进制数据）=====
ByteSource byteSource = Files.asByteSource(file);
byte[] raw = byteSource.read();

// 流式操作
ByteSink byteSink = Files.asByteSink(new File("output.bin"));
byteSource.copyTo(byteSink);

// 哈希
HashCode hash = byteSource.hash(Hashing.sha256());

// 大小
long size = byteSource.size();

// ⚠️ 注意：CharSource.read() 将全部内容读入内存
// 大文件不要用！用 readLines() 或 Closer 手动流处理

// ✅ 创建 Source 的各种方式
ByteSource.wrap(new byte[]{...});         // 内存字节数组
ByteSource.concat(source1, source2);      // 拼接多个源
CharSource.concat(source1, source2);      // 拼接多个字符源
Files.asCharSource(file, charset);        // 文件字符源
Files.asByteSource(file);                 // 文件字节源
Resources.asByteSource(url);              // 资源 URL
```

---

## 22. Files 工具

### 模式名称
文件操作工具

### 何时使用
- 快速的文件读写、遍历、临时文件
- 替代 Apache Commons IO 的 `FileUtils`

### 代码示例

```java
import com.google.common.io.Files;
import java.io.File;

// ✅ GOOD — 常用的文件操作

// 文件复制
File original = new File("source.txt");
File copy = new File("target.txt");
Files.copy(original, copy);

// 文件移动
Files.move(copy, new File("moved.txt"));

// 删除目录递归
Files.deleteRecursively(new File("tempDir"));
// ⚠️ 谨慎使用，会删除全部内容

// 创建临时目录
File tempDir = Files.createTempDir();
// JVM 退出时不会自动删除！需要自己维护

// 读取文件为字节
byte[] bytes = Files.toByteArray(file);

// 按行读取
List<String> lines = Files.readLines(file, Charsets.UTF_8);

// 写入
Files.write("content", file, Charsets.UTF_8);
Files.append("more content", file, Charsets.UTF_8);

// 文件扩展名/无扩展名
String ext = Files.getFileExtension("example.txt");  // "txt"
String name = Files.getNameWithoutExtension("example.txt");  // "example"

// 等价文件的规范路径
String canonical = Files.simplifyPath("./a/../b/c.txt");  // "b/c.txt"

// MIME 类型（基于文件扩展名）
String mime = Files.getFileExtension("photo.jpg")
    .map(ext -> MimeTypes.getExtension(ext))
    .orElse("application/octet-stream");

// ✅ 文件遍历
File dir = new File("mydir");
File[] children = dir.listFiles();

// ❌ BAD — 使用 JDK 的 File(File, String) 构造函数时注意
// new File(dir, "../etc/passwd")  // 路径穿越漏洞！
// Files.simplifyPath() 可帮助规范化
```

### ⚠️ Files 工具注意事项

- `Files.toByteArray()` 会将整个文件读入内存，大文件勿用
- `Files.createTempDir()` 创建的文件在 JVM 退出时**不会自动清理**
- Guava Files 工具在 Guava 31+ 中部分方法已标记 `@Beta`
- 新代码可考虑 Java 7/8 的 `Files.readAllBytes()`、`Files.readAllLines()` 等

---

## 23. Hashing（哈希）与 BloomFilter

### 模式名称
哈希函数抽象与布隆过滤器

### 何时使用
- 需要一致、高质量的哈希算法
- 布隆过滤器用于集合成员资格检查（存在性判断，可容忍假阳性）
- 去重、缓存穿透防御、拼写检查

### 代码示例

```java
import com.google.common.hash.*;

// ======================== Hashing ========================

// ✅ GOOD — 统一的哈希接口

// 各种哈希函数
HashFunction sha256 = Hashing.sha256();
HashFunction md5 = Hashing.md5();
HashFunction murmur3_32 = Hashing.murmur3_32();  // 32位，高性能
HashFunction murmur3_128 = Hashing.murmur3_128(); // 128位
HashFunction crc32 = Hashing.crc32();
HashFunction goodFastHash = Hashing.goodFastHash(128); // 平台自适应

// 哈希输入
HashCode hashCode = sha256
    .hashString("hello world", java.nio.charset.StandardCharsets.UTF_8);
String hex = hashCode.toString();       // 十六进制字符串
long longVal = hashCode.asLong();        // as long
int intVal = hashCode.asInt();           // as int（仅对32位哈希）

// 一致性哈希（用于分布式缓存分片）
// 自行实现 consistentHash

// 文件哈希
File file = new File("data.bin");
HashCode fileHash = Files.asByteSource(file).hash(Hashing.sha256());

// ======================== BloomFilter ========================

// ✅ GOOD — 布隆过滤器

// 预计插入 10000 个元素，期望假阳性率 1%
int expectedInsertions = 10_000;
double fpp = 0.01;  // 1% 假阳性率

BloomFilter<String> bloomFilter = BloomFilter.create(
    Funnels.stringFunnel(Charsets.UTF_8),
    expectedInsertions,
    fpp
);

// 添加元素
bloomFilter.put("alice@example.com");
bloomFilter.put("bob@example.com");

// 查询（可能有假阳性，无假阴性）
bloomFilter.mightContain("alice@example.com");   // true
bloomFilter.mightContain("eve@example.com");     // very likely false

// ❌ BAD — 用 Set 做很大的去重检查
// Set<String> allUsers = new HashSet<>(10_000_000);
// // 内存爆炸！

// ✅ 典型应用：缓存穿透防护
public class CacheProtector {
    private final BloomFilter<String> knownKeys;
    
    public CacheProtector(Set<String> allValidKeys) {
        this.knownKeys = BloomFilter.create(
            Funnels.stringFunnel(Charsets.UTF_8),
            allValidKeys.size(),
            0.001  // 0.1% 假阳性
        );
        allValidKeys.forEach(knownKeys::put);
    }
    
    public boolean shouldQueryDatabase(String key) {
        if (!knownKeys.mightContain(key)) {
            return false;  // 肯定不存在，拒绝查询
        }
        return true;  // 可能存在，查询 DB（少量假阳性请求）
    }
}
```

### BloomFilter 参数

| 参数 | 含义 |
|------|------|
| `expectedInsertions` | 预计插入元素数 |
| `fpp` | 期望的假阳性率（默认 3%） |
| `Funnel` | 对象到字节流的序列化方式 |

**物理意义**：1000 万元素，1% 假阳性率 → 大约 120 MB 内存。同样元素用 HashSet → 数百 MB + 大量 GC 开销。

---

## 24. 原始类型工具（Ints / Longs / Doubles）

### 模式名称
原始类型集合与转换

### 何时使用
- `int[]` / `long[]` 与 `List<Integer>` / `List<Long>` 之间的转换
- 解析、比较、连接原始类型数组
- 处理含 `byte` 的原始类型常量

### 代码示例

```java
import com.google.common.primitives.*;

// ===== Ints =====
int[] array = Ints.toArray(Arrays.asList(1, 2, 3, 4, 5));
// List<Integer> → int[]

List<Integer> list = Ints.asList(1, 2, 3);
// int[] → List<Integer>（视图，修改同步）

// 连接
int[] combined = Ints.concat(
    new int[]{1, 2, 3},
    new int[]{4, 5, 6}
);  // [1, 2, 3, 4, 5, 6]

// 最大值/最小值
int max = Ints.max(3, 7, 2, 9, 1);  // 9
int min = Ints.min(3, 7, 2, 9, 1);  // 1

// 包含
boolean hasSeven = Ints.contains(new int[]{1, 2, 3, 7}, 7);  // true

// 索引
int index = Ints.indexOf(new int[]{1, 2, 3, 7, 3}, 3);  // 1（第一个匹配）

// 解析（安全，不抛异常）
Integer parsed = Ints.tryParse("123");   // 123
Integer bad = Ints.tryParse("abc");     // null（不抛 NumberFormatException）

// 字节转换
byte[] bytes = Ints.toByteArray(0x12345678);  // 4字节大端
int value = Ints.fromByteArray(bytes);         // 0x12345678

// 约束范围
int clamped = Ints.constrainToRange(value, 0, 100);
// 相当于 Math.max(0, Math.min(100, value))

// ===== Longs / Doubles / Floats =====
// 相同的 API

long[] longs = Longs.toArray(longList);
Double d = Doubles.tryParse("3.14");  // 3.14
boolean isFinite = Doubles.isFinite(d);  // true（非 NaN/Infinity）

// ===== Bytes / Shorts / Chars =====
byte[] byteArray = Bytes.toArray(byteList);
short[] shortArray = Shorts.toArray(shortList);

// ❌ BAD — 手动装箱拆箱循环
// int[] result = new int[list.size()];
// for (int i = 0; i < list.size(); i++) {
//     result[i] = list.get(i);
// }
```

---

## 25. EventBus（事件总线）

### 模式名称
发布-订阅事件机制

### 何时使用
- 同一 JVM 内的组件间松耦合通信
- 替代观察者模式的硬编码注册
- 不需要跨进程的消息传递（那是消息队列的事）

### 代码示例

```java
import com.google.common.eventbus.*;

// ===== 事件类（POJO）=====
public class UserCreatedEvent {
    private final String userId;
    private final String email;
    // 构造器、getters...
}

// ===== 订阅者 =====
public class EmailNotificationListener {
    
    @Subscribe  // 标记为事件处理器
    public void onUserCreated(UserCreatedEvent event) {
        sendWelcomeEmail(event.getEmail());
    }
    
    @Subscribe
    public void onAnyEvent(Object event) {
        log.info("Event received: {}", event);
    }
}

// ===== 发布者 =====
public class UserService {
    private final EventBus eventBus;
    
    public UserService(EventBus eventBus) {
        this.eventBus = eventBus;
    }
    
    public void createUser(String email) {
        User user = saveToDatabase(email);
        // 发布事件，不关心谁在处理
        eventBus.post(new UserCreatedEvent(user.getId(), email));
    }
}

// ===== 使用 =====
EventBus eventBus = new EventBus("default");     // 同步 EventBus
AsyncEventBus asyncBus = new AsyncEventBus(      // 异步 EventBus
    "async", Executors.newFixedThreadPool(4));

// 注册订阅者
eventBus.register(new EmailNotificationListener());
eventBus.register(new AuditLogListener());
eventBus.register(new IndexingListener());

// 触发事件（由 UserService 触发）
userService.createUser("alice@example.com");

// ✅ 事件继承：订阅父类事件也会收到子类事件
// 如 subscribe(Object.class) 会收到所有事件
```

### EventBus 使用原则

| 原则 | 说明 |
|------|------|
| 事件是 POJO | 不需要继承或实现任何接口 |
| @Subscribe | 方法访问级别任意，但必须只有一个参数 |
| 默认同步 | 在 post 线程执行。异步用 AsyncEventBus |
| 异常处理 | 默认抛异常。用 SubscriberExceptionHandler 自定义 |
| 不序列化 | 不跨 JVM，不序列化事件对象 |
| 死事件 | 无订阅者的事件会被包装为 DeadEvent 再发一次 |

### 异常处理

```java
// 自定义异常处理
EventBus eventBus = new EventBus((exception, context) -> {
    log.error("Event {} failed at subscriber {}: {}",
        context.getEvent(),
        context.getSubscriberMethod(),
        exception.getMessage());
});
```

### ⚠️ EventBus 注意事项
- 订阅者抛出异常**不会影响**其他订阅者（但默认会打印）
- 订阅者按注册顺序执行（但不要依赖顺序）
- 事件没有超时机制
- 不适合高吞吐量场景（可用 Disruptor / LMAX）

---

## 26. 须避免的模式

### 1. Guava Optional 在 Java 8+

```java
// ❌ BAD — 继续使用 Guava Optional
import com.google.common.base.Optional;
Optional<String> opt = Optional.fromNullable(value);

// ✅ GOOD — 迁移到 JDK Optional
import java.util.Optional;
Optional<String> opt = Optional.ofNullable(value);
```

### 2. Ordering（Java 8+）

```java
// ❌ BAD — 用 Guava Ordering
Ordering<User> byName = Ordering.natural().onResultOf(User::getName);

// ✅ GOOD — 用 JDK Comparator
Comparator<User> byName = Comparator.comparing(User::getName);
```

### 3. 滥用 Functions / Predicates（Java 8+）

```java
// ❌ BAD — 用 Guava Predicate
Predicate<String> p = input -> input != null;
FluentIterable.from(list).filter(p).toList();

// ✅ GOOD — 用 Stream API
list.stream().filter(x -> x != null).collect(Collectors.toList());
```

### 4. 过度使用 Collections2 / Iterables

```java
// ❌ BAD — Guava 的 Collections2/Iterables
Iterables.filter(list, predicate);
Iterables.transform(list, function);

// ✅ GOOD — Stream API
list.stream().filter(predicate).map(function);
```

### 5. 在 Java 8+ 使用 Objects.firstNonNull

```java
// ❌ BAD
MoreObjects.firstNonNull(a, b);

// ✅ GOOD
Objects.requireNonNullElse(a, b);
```

### 6. 用 Multimap 时误用 get() 而不是 asMap()

```java
// ❌ BAD — get(key) 返回空集合不抛异常
multimap.get("nonexistent");  // 返回空 List，不是 null！

// ✅ GOOD — 检查是否存在
if (multimap.containsKey("key")) {
    Collection<String> values = multimap.get("key");
}
```

### 7. CacheBuilder 的 weakKeys() 陷阱

```java
// ⚠️ 使用 weakKeys() 时，key 用 == 而不是 equals() 比较！
CacheBuilder.newBuilder()
    .weakKeys()     // 🚨 使用引用相等性
    .build(loader);

// 这几乎总不是你想要的。避免用 weakKeys()。
// 除非你确定理解引用相等性的含义。
```

### 8. 将 ListenableFuture 用于纯 CPU 密集计算

```java
// ❌ BAD — CPU 密集任务用额外线程没意义
// CPU 密集任务应该用 ForkJoinPool，而不是 listeningDecorator
```

### 9. 过度使用 EventBus（超出松耦合的合理范围）

```java
// ❌ BAD — 一个方法调用远比其他方式重
eventBus.post(new SaveEvent(user));
// 如果是紧耦合组件，直接调用更快、更可追踪

// ✅ GOOD — EventBus 适合的边界
// 1. 不同模块之间（如 UserService → EmailService）
// 2. 插件式架构
// 3. 第三方扩展点
```

### 10. Guava 故意避免的功能

| Guava 不做 | 替代方案 |
|-----------|---------|
| JSON 解析 | Jackson / Gson |
| HTTP 客户端 | OkHttp / HttpClient |
| ORM / 数据库 | MyBatis / Hibernate / jOOQ |
| 模板引擎 | Mustache / Thymeleaf |
| 日志框架 | SLF4J + Logback |
| 依赖注入 | Dagger / Guice / Spring |
| 字节码操作 | ASM / ByteBuddy |
| RPC 框架 | gRPC / Dubbo |
| 测试框架 | JUnit / TestNG / Mockito |
| Web 框架 | Spring MVC / JAX-RS |

Guava 的定位始终是 **核心 Java 库的增强**，不涉足应用框架层面。

---

## 27. Maven / Gradle 依赖

### Maven

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.1.0-jre</version>  <!-- 或 33.1.0-android 用于 Android -->
</dependency>
```

### Gradle

```groovy
implementation 'com.google.guava:guava:33.1.0-jre'
```

### 版本说明

| 版本后缀 | 目标平台 | 说明 |
|---------|---------|------|
| `-jre` | Java 8+ | 主推版本，使用 java.time, java.util.function 等 |
| `-android` | Android + Java 7 | 不依赖 Java 8 API |
| 无后缀 | 旧版 | 推荐使用带后缀的明确版本 |

### 依赖冲突

使用 `com.google.guava:guava` 而不是 `com.google.guava:guava-jdk5` 或散装分模块。

```xml
<!-- ✅ 正确：单一 guava 坐标 -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
</dependency>

<!-- ❌ 避免：散装模块（Guava 已合并为一个 jar） -->
<!-- guava-collections, guava-concurrent, guava-cache 等已废弃 -->
```

---

## 总结速查

### 什么时候该用 Guava？

```
┌─────────────────────────────────────────────────────────┐
│ ✅ 强烈推荐                                             │
│   ImmutableList / ImmutableMap / ImmutableSet            │
│   Preconditions                                         │
│   MoreObjects.toStringHelper                            │
│   CacheBuilder (LoadingCache)                           │
│   Splitter / Joiner / CharMatcher                       │
│   Multiset / Multimap / BiMap / Table / RangeSet        │
│   ListenableFuture + Futures                            │
│   RateLimiter                                           │
│   Striped                                               │
│   BloomFilter + Hashing                                 │
│   Ints / Longs / Doubles（原始类型工具）                 │
├─────────────────────────────────────────────────────────┤
│ ⚠️ 有条件推荐（Java 7 或已有大量 Guava 代码库）          │
│   FluentIterable / Functions / Predicates                │
│   Optional（Java 8+ 迁移前临时使用）                     │
│   Ordering（尽快迁移到 Comparator.comparing）             │
│   EventBus（同 JVM 松耦合，但注意不要过度使用）          │
├─────────────────────────────────────────────────────────┤
│ ❌ Java 8+ 新代码应避免                                  │
│   FluentIterable（→ Stream API）                         │
│   Ordering（→ Comparator.comparing）                     │
│   Guava Optional（→ JDK Optional）                       │
│   Collections2 / Iterables（→ Stream API）               │
│   Suppliers.memoize（→ ConcurrentHashMap 缓存）          │
└─────────────────────────────────────────────────────────┘
```

### 关键迁移路径

```
┌──────────────────┬──────────────────────┬──────────────┐
│ 旧风格             │ 新风格（Java 8+）    │ 建议          │
├──────────────────┼──────────────────────┼──────────────┤
│ FluentIterable    │ Stream<E>            │ 立即迁移      │
│ Ordering          │ Comparator           │ 立即迁移      │
│ Guava Optional    │ java.util.Optional   │ 立即迁移      │
│ Functions/Preds   │ java.util.function   │ 立即迁移      │
│ ListenableFuture  │ CompletableFuture    │ 逐步评估      │
│ Preconditions     │ 保留                 │ 长期使用      │
│ Immutable*        │ 保留                 │ 长期使用      │
│ CacheBuilder      │ Caffeine（可替代）    │ 可选升级      │
│ Splitter/Joiner   │ 保留                 │ 长期使用      │
│ RateLimiter       │ 保留                 │ 长期使用      │
└──────────────────┴──────────────────────┴──────────────┘
```
