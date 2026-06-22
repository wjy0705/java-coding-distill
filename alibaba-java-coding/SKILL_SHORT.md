---
name: alibaba-java-coding-short
description: 阿里巴巴Java开发手册极简速查版 - 规避高频踩坑点与命名规约
---

# 阿里巴巴Java开发手册 (泰山版) - 极简速查

## 1. 命名与OOP规约
*   **类与接口**：大驼峰（UpperCamelCase）。接口实现类必须以 `Impl` 结尾。
*   **变量与方法**：小驼峰（lowerCamelCase）。**POJO类中的布尔变量绝对不要加 `is` 前缀**（防止RPC序列化时字段丢失）。
*   **常量**：全部大写以底划线分隔（UPPER_SNAKE_CASE），禁止出现魔法值。
*   **数值处理**：禁止使用 `float`/`double` 进行货币计算，必须使用 `BigDecimal`（且必须通过 `BigDecimal.valueOf(double)` 或 `new BigDecimal(String)` 初始化）。
*   **对象比较**：包装类比较（如 `Integer`）必须使用 `equals()`，禁止使用 `==`（防缓存失效）。

## 2. 集合与并发处理
*   **集合初始化**：创建 `HashMap` 时需指定初始化容量：`initialCapacity = (需要存储的元素个数 / 负载因子) + 1`。
*   **集合遍历**：禁止在 `foreach` 循环里进行元素的 `remove`/`add` 操作，必须使用 `Iterator` 或 JDK 8+ 的 `removeIf()`。
*   **线程池创建**：**绝对禁止使用 `Executors` 创建线程池**，必须通过 `ThreadPoolExecutor` 手动指定核心参数，避免 OOM。
*   **锁的释放**：使用 `Lock` 时，加锁动作必须在 `try` 块之外，且首行紧跟 `try` 块，并在 `finally` 中释放锁。
    ```java
    lock.lock();
    try {
        // 业务逻辑
    } finally {
        lock.unlock();
    }
    ```

## 3. 异常与数据库规约
*   **异常捕获**：禁止捕获 `Throwable`；禁止捕获异常后吞掉，必须打印堆栈或重新抛出。
*   **建表规范**：表名、字段名必须小写加下划线（`lower_snake_case`），禁止使用保留字。
*   **索引命名**：主键 `pk_字段名`，唯一索引 `uk_字段名`，普通索引 `idx_字段名`。
*   **SQL查询**：禁止使用 `select *`，必须写明具体字段；禁止在代码中拼接 SQL，防注入。
