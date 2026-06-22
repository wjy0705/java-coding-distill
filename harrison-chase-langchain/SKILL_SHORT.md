---
name: harrison-chase-langchain-short
description: Harrison Chase AI Agent 架构思维极简速查版 - Agent/Tool 设计与 Java/Spring AI 对照
---

# Harrison Chase AI Agent 架构 - 极简速查

## 1. AI Agent 核心抽象组件
*   **Chain (链)**：编排固定的步骤管道（如：提示词 -> 大模型 -> 输出解析器）。
*   **Agent (智能体)**：带有推理核心的调度器。利用 LLM 的决定权，在循环中自主选择下一步的操作（Action）和调用的工具。
*   **Tool (工具)**：向 Agent 暴露的可执行原子能力。在 Java 中通常对应带 `@Tool` 注解的方法。
*   **Memory (记忆)**：在多轮对话（Turns）之间传递并持久化状态（如聊天上下文、总结、向量检索）。
*   **Callback (回调)**：为流式输出、可观测性（Tracing）和监控提供介入钩子。

## 2. Agent 运行模式与工具设计
*   **ReAct 模式 (Reason + Act)**：
    *   **Thought** (推理) -> **Action** (选择工具及参数) -> **Observation** (运行工具并获取结果) -> 重复循环直至输出 **Final Answer**。
*   **Tool (工具) 设计三大黄金法则**：
    1.  **明确而翔实的描述 (Description)**：LLM 仅通过描述决定是否调用工具，描述必须写明：什么时候用、参数代表什么、返回什么。
    2.  **细粒度原则**：一个工具只做好一件事。禁止设计“万能工具”（如 `database_query`），应拆分为 `get_user_by_id`、`update_user_email`。
    3.  **完备的错误处理**：工具执行报错时，不要直接抛出系统崩溃异常，应将异常捕获并格式化为人类可读的字符串返回给 LLM（例如："Error: User ID 123 not found"），引导 LLM 自主尝试修复。

## 3. Java 生态 (Spring AI / LangChain4j) 概念对照

| Python (LangChain) | Java (Spring AI) | Java (LangChain4j) | 核心概念 |
| :--- | :--- | :--- | :--- |
| `ChatModel` | `ChatModel` | `ChatLanguageModel` | LLM 推理底层接口 |
| `PromptTemplate` | `PromptTemplate` | `PromptTemplate` | 结构化 Prompt 管理 |
| `@tool` | `@Tool` / `ToolCallback` | `@Tool` | 工具定义注解 |
| `BaseChatMessageHistory` | `ChatHistory` | `ChatMemory` | 会话记忆保存介质 |
| `LangSmith` | `Spring AI Observability` | `OpenTelemetry` | 可观测性与追踪监测 |
