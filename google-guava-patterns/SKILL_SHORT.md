---
name: google-guava-patterns-short
description: Google Guava 最佳实践极简速查版 - 不可变集合、缓存与高并发组件
---

# Google Guava 最佳实践 - 极简速查

## 1. 基础工具与校验 (Preconditions)
*   **参数校验**：使用 `Preconditions` 进行防御式编程，提供更具表达力的错误抛出：
    ```java
    Preconditions.checkArgument(age > 0, "Age must be positive: %s", age);
    Preconditions.checkState(isInitialized, "Service is not initialized");
    Objects.requireNonNull(user, "User cannot be null"); // JDK 9+ 原生
    ```

## 2. 集合与扩展数据结构
*   **不可变集合**：优先使用 JDK 9+ `List.of()`、`Map.of()`。如需复杂构建，使用 Guava 专属的 Builder：
    ```java
    ImmutableList<String> list = ImmutableList.<String>builder()
        .add("a").add("b").build();
    ```
*   **Multimap (一对多映射)**：避免手动维护 `Map<K, List<V>>`，直接使用 `ArrayListMultimap` 或 `HashMultimap`：
    ```java
    ListMultimap<String, Integer> scores = ArrayListMultimap.create();
    scores.put("Alice", 90);
    scores.put("Alice", 95); // 自动合并
    ```
*   **BiMap (双向映射)**：键值对必须双向唯一，支持通过 `inverse()` 反向键值对查询。
*   **Table (行列映射)**：行、列、值的三维关系映射，替代多重嵌套的 Map。

## 3. 并发、限流与本地缓存
*   **LoadingCache (本地缓存)**：集成数据自动加载与过期机制，防止缓存击穿：
    ```java
    LoadingCache<String, User> userCache = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build(new CacheLoader<String, User>() {
            public User load(String key) { return fetchUserFromDb(key); }
        });
    ```
*   **RateLimiter (限流器)**：基于令牌桶算法，支持平滑消费限流：
    ```java
    RateLimiter limiter = RateLimiter.create(5.0); // 每秒产生 5 个令牌
    if (limiter.tryAcquire()) {
        processRequest();
    }
    ```
*   **迁移路线**：JDK 8+ 之后，对于 `Optional`、`Splitter`/`Joiner`、`ListenableFuture`，优先使用原生 JDK API（如 Java 8 Stream, java.util.Optional, CompletableFuture）。
