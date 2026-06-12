---
name: harrison-chase-langchain
description: Harrison Chase的LangChain架构思维蒸馏 - AI Agent框架设计与编码模式
---

# Harrison Chase's LangChain Architecture Patterns

> Distilled from the LangChain codebase, talks, and design philosophy — applicable to any language including Java/Spring AI.

---

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **Harrison Chase** 的 LangChain 架构思维与 AI Agent 框架设计模式。Harrison Chase 是 LangChain 的创始人和核心架构师，LangChain 是全球最流行的 AI Agent 框架。虽然 LangChain 主要用 Python 实现，但其核心抽象（Chain/Agent/Tool/Memory/Callback）已经被 Spring AI、LangChain4j 等 Java 框架完全采纳，是 AI Agent 开发的架构蓝本。

### 参考来源
- **开源代码**: LangChain 核心库源码（langchain-ai/langchain, MIT 协议）
- **架构文档**: LangChain 官方文档的架构说明与设计原则
- **Harrison Chase 的访谈/演讲**: 他在各类 AI 会议和技术播客中的架构思路分享
- **Spring AI / LangChain4j**: Java 生态对 LangChain 模式的对应实现

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：多源信息采集（源码/文档/演讲） → 架构框架提炼（5 大核心抽象/4 种 Agent 类型/3 种 Memory 模式/Tool 设计哲学） → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 设计 AI Agent 时 | 选 Chain 还是 Agent？Agent 用 ReAct 还是 Tool-calling？架构决策有依据 |
| 设计 Tool 时 | Tool 的粒度怎么控制？描述怎么写？输入输出怎么设计？ |
| 选 Memory 方案时 | ConversationBuffer 还是 VectorStore？Summary 还是 Combined？各有什么代价？ |
| 面试 AI Agent 岗时 | "LangChain 的核心抽象是什么""Agent 和 Chain 的区别"——答案直接从这里出 |
| 在 Java 中实现 AI Agent 时 | Spring AI 的对应实现对照表，Python 概念 → Java 代码实现无痛过渡 |

---

## 1. Core Architecture: The Component Model

LangChain is not a monolithic framework — it is a **component library** built around composable primitives.

### The Five Core Abstractions

```
┌─────────────────────────────────────────────────────┐
│                    Application                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐ │
│  │  Chain   │─▶│  Agent   │─▶│  Executor / Loop  │ │
│  └──────────┘  └──────────┘  └───────────────────┘ │
│       │              │                               │
│       ▼              ▼                               │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐ │
│  │  Tool    │  │  Memory  │  │     Callbacks     │ │
│  └──────────┘  └──────────┘  └───────────────────┘ │
└─────────────────────────────────────────────────────┘
```

| Component | Role | Java Analogy |
|-----------|------|-------------|
| **Chain** | Composable pipeline of processing steps | `Chain<T, R>` or `Function<T, R>` composition |
| **Agent** | Decision-making loop — chooses which action to take next | A scheduler with a reasoning core |
| **Tool** | Single capability exposed to the agent (read-only, no side-effect tracking) | `@Tool` bean in Spring AI |
| **Memory** | State persistence across turns | Conversation history store |
| **Callback** | Hook system for observability, streaming, logging | Spring AOP / Listener pattern |

### Design Principle

> "Every component should be usable independently, and combinable arbitrarily."
> — Harrison Chase

This means: a `Tool` should work without an `Agent`, a `Chain` without `Memory`, etc.

---

## 2. Agent Patterns

### 2.1 ReAct Agent (Reason + Act)

The foundational pattern — cited in the ReAct paper (Yao et al., 2023).

**Loop:**
1. **Thought** — LLM reasons about the current state and decides what to do
2. **Action** — LLM emits a structured action (tool call)
3. **Observation** — Tool result is fed back
4. Repeat until a final answer is produced

```
Thought: I need to find the weather in Tokyo.
Action: weather_api(city="Tokyo")
Observation: {"temp": 22, "condition": "cloudy"}
Thought: Got it. I can answer now.
Final Answer: The weather in Tokyo is 22°C and cloudy.
```

**Python (LangChain):**
```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain.tools import tool

@tool
def weather_api(city: str) -> str:
    """Get the current weather for a city."""
    return f"Fetching weather for {city}..."

agent = create_react_agent(llm, tools=[weather_api], prompt=react_prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
executor.invoke({"input": "What's the weather in Tokyo?"})
```

**Java (Spring AI):**
```java
@Tool(name = "weather_api", description = "Get the current weather for a city")
public String getWeather(@ToolParam("city") String city) {
    return weatherService.fetch(city);
}

// Spring AI's ChatClient with tool binding
String response = chatClient.prompt()
    .user("What's the weather in Tokyo?")
    .tools(weatherTool)
    .call()
    .content();
```

### 2.2 Plan-and-Execute Agent

> "First plan, then execute step by step, re-planning only when necessary."
> — Harrison Chase

Decomposes a complex goal into a step-by-step plan, then executes each step. More efficient than ReAct for multi-step tasks because it avoids re-planning at every turn.

```
PLAN:
1. Search for recent LangChain releases
2. Extract the latest version number
3. Write a summary

STEP 1: search_web("LangChain recent releases") → [results]
STEP 2: parse_version([results]) → "0.3.0"
STEP 3: write_summary("0.3.0") → done
```

**Key insight:** Plan-and-Execute reduces LLM calls by batching the planning overhead into one call instead of one per tool invocation.

### 2.3 OpenAI Function Calling / Tool-Calling Agent

Leverages models fine-tuned to output structured tool-call arguments directly — no "Thought/Action/Observation" text format needed.

```python
# LangChain expression (tool-calling agent)
from langchain.agents import create_tool_calling_agent

agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools)
```

**Java (Spring AI):**
```java
// Spring AI implicitly uses function-calling format for supported models
ChatResponse response = chatClient.prompt()
    .user("Book a flight to London")
    .tools(bookFlightTool, searchFlightsTool)
    .call();
```

### When to Use Which Agent

| Pattern | Use Case | Cost |
|---------|----------|------|
| **ReAct** | Open-ended reasoning, unknown number of steps | High (LLM call per step) |
| **Plan-and-Execute** | Multi-step tasks with clear sub-goals | Medium (one plan call + exec calls) |
| **Tool-Calling** | Model supports it natively (GPT-4, Claude 3, Gemini) | Lowest (structured output) |

---

## 3. Chain-of-Thought Patterns

### 3.1 LLMChain

The simplest chain — prompt template + LLM.

```python
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(
    input_variables=["product"],
    template="Write a catchy tagline for {product}."
)
chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(product="a new AI-powered toaster")
```

**Java:**
```java
// Spring AI PromptTemplate
PromptTemplate template = PromptTemplate.from(
    "Write a catchy tagline for {product}."
);
Prompt prompt = template.create(Map.of("product", "a new AI-powered toaster"));
String result = chatClient.prompt(prompt).call().content();
```

### 3.2 SequentialChain

Compose chains in sequence — output of one becomes input of the next.

```python
from langchain.chains import SequentialChain

chain_1 = LLMChain(llm=llm, prompt=analyze_prompt, output_key="analysis")
chain_2 = LLMChain(llm=llm, prompt=summarize_prompt, output_key="summary")

pipeline = SequentialChain(
    chains=[chain_1, chain_2],
    input_variables=["document"],
    output_variables=["summary"]
)
```

**Key principle:** Each chain has explicit input/output keys. This gives you a typed contract between stages — the same philosophy as Unix pipes or Java `Function` composition.

### 3.3 RouterChain

Route input to one of many specialized chains based on classification.

```python
from langchain.chains.router import RouterChain, MultiRouteChain

router = RouterChain.from_llm(
    llm=llm,
    destination_chains={
        "technical": tech_chain,
        "creative": creative_chain,
        "default": general_chain
    }
)
```

**Java analogy:** A `Router` class that uses the LLM to classify intent, then dispatches to the appropriate `Chain` bean.

### Chain Design Heuristics (from Chase)

> "If your chain has more than 3 steps, consider making it an agent instead."
> "A chain is for known-route pipelines. An agent is for unknown-route exploration."

| Indicator | Use Chain | Use Agent |
|-----------|-----------|-----------|
| Steps are fixed and known | ✅ | ❌ |
| Branching depends on intermediate results | ❌ | ✅ |
| Need for human-in-the-loop | ❌ | ✅ |
| Pure data transformation | ✅ | ❌ |
| External tool usage | ❌ | ✅ |

---

## 4. Tool Design Philosophy

### The Interface

LangChain's `Tool` is dead simple:

```python
class Tool(BaseModel):
    name: str
    description: str           # LLM-facing — critical for correct routing
    func: Callable              # The actual implementation
    args_schema: Type[BaseModel]  # Typed inputs (Pydantic model)
```

### Principles from the LangChain Codebase

1. **One tool = one capability.** A tool should do exactly one thing and do it well. Not "process_data" but "calculate_statistics". Not "file_ops" but "read_file" and "write_file" as separate tools.

2. **Description is the contract.** The LLM reads the `description` to decide which tool to call. Harrison Chase emphasizes:
   > "The description is more important than the implementation. If the LLM can't understand when to use it, the tool is useless."

3. **Typed inputs.** Every parameter should have a clear type and description. This lets the model generate correct arguments.

4. **Fail gracefully.** Tools should catch their own errors and return a string describing what went wrong — never throw an unhandled exception to the agent loop.

```python
@tool
def search_database(query: str, limit: int = 5) -> str:
    """Search the product database by keyword. Returns up to `limit` results."""
    try:
        results = db.search(query, limit=limit)
        return format_results(results)
    except Exception as e:
        return f"Search failed: {str(e)}"  # ← graceful error, not crash
```

### Java Equivalent

```java
@Tool(
    name = "search_database",
    description = "Search the product database by keyword. Returns up to limit results."
)
public String searchDatabase(
    @ToolParam("query") String query,
    @ToolParam(value = "limit", defaultValue = "5") int limit
) {
    try {
        List<Product> results = productRepository.search(query, limit);
        return formatResults(results);
    } catch (Exception e) {
        return "Search failed: " + e.getMessage();
    }
}
```

### Tool Granularity Decision Guide

| If you're tempted to... | Instead... |
|------------------------|-----------|
| `create_update_delete_anything()` | One tool per CRUD operation |
| `search_web_and_analyze()` | `search_web()` + `analyze_text()` |
| `file_manager()` | `read_file()`, `write_file()`, `list_directory()` |
| `do_everything()` | `execute_command()` with explicit security constraints |

---

## 5. Memory Patterns

### 5.1 ConversationBufferMemory

Stores the raw conversation history as a list of messages.

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.chat_memory.add_user_message("Hi, I'm Alice.")
memory.chat_memory.add_ai_message("Hello Alice! How can I help?")
```

**Problem:** Grows unbounded. Every turn adds tokens.

### 5.2 ConversationSummaryMemory

Summarizes older conversation turns — retains key information without token bloat.

```python
from langchain.memory import ConversationSummaryMemory

memory = ConversationSummaryMemory(llm=llm)
# Automatically summarizes history when token limit is reached
```

**When to use:** Long-running conversations (>10 turns) where the full history would exceed context window.

**Key insight (Chase):** SummaryMemory trades fidelity for token economy. Use it when precision of recall matters less than context budget.

### 5.3 VectorStoreMemory

Stores conversation fragments as embeddings, retrieves semantically relevant chunks.

```python
from langchain.memory import VectorStoreRetrieverMemory

memory = VectorStoreRetrieverMemory(
    retriever=vector_store.as_retriever(),
    memory_key="relevant_history"
)
```

**When to use:** Conversations that span hundreds of turns where only contextually relevant history matters.

### 5.4 Combined Memory

Use multiple memory types together:

```python
# Keep last 2 turns in full, summarize older, vector-search for relevant context
memory = CombinedMemory(
    short_term=ConversationBufferMemory(k=2),
    long_term=ConversationSummaryMemory(llm=llm),
    semantic=VectorStoreRetrieverMemory(retriever=retriever)
)
```

### Memory Decision Heuristics

| Scenario | Memory Type |
|----------|-------------|
| < 5 turns, simple chat | BufferMemory |
| > 10 turns, same topic | SummaryMemory |
| > 50 turns, multiple topics | VectorStoreMemory |
| Chatbot that persists across sessions | VectorStoreMemory + persistence |
| Need perfect recall of user preferences | SummaryMemory + BufferMemory(k=2) |

---

## 6. Best Practices from the LangChain Codebase

### 6.1 Clean Abstractions

The LangChain codebase is organized by **responsibility**, not by feature.

```
langchain/
  chains/        # Pipeline abstractions
  agents/        # Decision-making loops
  tools/         # Capability wrappers
  memory/        # State management
  callbacks/     # Observability hooks
  llms/          # Model integrations
```

**Apply this in Java:**
```
com.example.ai/
  chain/          # Chain implementations
  agent/          # Agent executors
  tool/           # Tool definitions
  memory/         # Conversation memory
  callback/       # Logging, metrics, tracing
  model/          # LLM client wrappers
```

### 6.2 Separation of Concerns

Each component owns one dimension of the system:

| Component | Owns |
|-----------|------|
| **Chain** | Orchestration order |
| **Agent** | Decision logic |
| **Tool** | Capability execution |
| **Memory** | State serialization |
| **Callback** | Side effects (logging, metrics, streaming) |

> "A tool should not know about memory. A chain should not know about callbacks."
> — implied by LangChain's composition design

### 6.3 Callback System

LangChain's callback system is a **fire-and-forget observer pattern** — components emit events, callbacks consume them. This is the key to adding observability without coupling.

```python
class CallbackManager:
    def on_llm_start(self, serialized, prompts, **kwargs): ...
    def on_llm_end(self, response, **kwargs): ...
    def on_tool_start(self, serialized, input_str, **kwargs): ...
    def on_tool_end(self, output, **kwargs): ...
    def on_chain_start(self, serialized, inputs, **kwargs): ...
    def on_agent_action(self, action, **kwargs): ...
```

**Java pattern:**
```java
public interface AiEventListener {
    default void onLlmStart(String prompt) {}
    default void onLlmEnd(String response) {}
    default void onToolStart(String toolName, String input) {}
    default void onToolEnd(String toolName, String output) {}
    default void onChainStart(String chainName) {}
    default void onChainEnd(String chainName, String result) {}
}
```

Use cases for callbacks:
- **Logging** every LLM call (debugging, auditing)
- **Streaming** tokens to the UI in real-time
- **Metrics** (token count, latency, tool usage frequency)
- **Security** (audit every tool invocation)
- **Tracing** (LangSmith, OpenTelemetry)

### 6.4 Error Handling Philosophy

From Harrison Chase's talks:

1. **Tools catch and return errors as strings** — never let exceptions bubble up to the agent loop
2. **Agents retry on tool failure** — the agent loop handles retry logic, not the tool
3. **Chains fail fast** — if a chain step fails, the whole chain fails. Chains are for deterministic pipelines
4. **Callbacks never throw** — a callback failure should never break the main flow

### 6.5 Testability Pattern

LangChain components are designed so that each piece can be tested in isolation:

```python
# Test a tool independently of any agent
def test_weather_tool():
    result = weather_tool.func("Tokyo")
    assert "Tokyo" in result
    assert "temperature" in result or "error" in result.lower()

# Test an agent's decision logic with a mock LLM
def test_agent_reasoning():
    mock_llm = FakeLLM(responses=["action: search_web\naction_input: query"])
    agent = create_react_agent(mock_llm, tools, prompt)
    # ... assert correct tool was called
```

---

## 7. Anti-Patterns

### 7.1 The God Chain ❌

```python
# BAD: One chain that does everything
chain = LLMChain(llm=llm, prompt=prompt)
chain.run("analyze this, write a report, send an email, and update the database")
```

**Problem:** The LLM can't use tools, can't loop, can't handle errors per step.

**Fix:** Use an agent with specialized tools.

### 7.2 Unhandled Tool Errors ❌

```python
@tool
def delete_file(path: str) -> str:
    os.remove(path)  # ← crashes the agent if file doesn't exist
    return "Deleted"
```

**Fix:** Always wrap tool implementations in try/except and return descriptive error strings.

### 7.3 Tight Coupling ❌

```python
# BAD: Chain directly instantiates a specific LLM
class MyChain:
    def __init__(self):
        self.llm = OpenAI(model="gpt-4")  # ← hardcoded

# GOOD: Accept dependencies
class MyChain:
    def __init__(self, llm: BaseLLM):
        self.llm = llm
```

### 7.4 Ignoring Memory Overhead ❌

```python
# BAD: Buffer grows unbounded
memory = ConversationBufferMemory()
for turn in range(100):
    chain.run(input=user_messages[turn], memory=memory)
```

**Fix:** Set `k` (keep last N turns) or switch to SummaryMemory for long conversations.

### 7.5 Over-Agenting ❌

```python
# BAD: Using an agent when a chain would do
agent = create_react_agent(llm, tools=[], prompt=prompt)
executor = AgentExecutor(agent=agent, tools=[])
result = executor.invoke({"input": "Translate 'hello' to French"})
```

**Fix:** Use `LLMChain` for simple transforms, `SequentialChain` for known pipelines, agents only when tool selection or branching is needed.

### 7.6 Vague Tool Descriptions ❌

```python
@tool
def process(x: str) -> str:
    """Do stuff."""  # ← LLM has no idea when to call this
```

**Fix:** Be explicit: *when* to use, *what* it does, *what* it returns.

### 7.7 Hardcoding Secrets in Tools ❌

```python
@tool
def call_api(query: str) -> str:
    api_key = "sk-..."  # ← Never!
```

**Fix:** Use environment variables / Spring's `@Value` / a secrets manager.

---

## 8. Spring AI Equivalents (Java/Spring Boot)

Spring AI is the closest Java ecosystem equivalent to LangChain.

| LangChain | Spring AI | Notes |
|-----------|-----------|-------|
| `LLMChain` | `ChatClient.prompt().call()` | Spring AI is less chain-oriented, more fluent API |
| `Tool` | `@Tool` annotation on `@Service` beans | Very similar in concept |
| `AgentExecutor` | `ChatClient` with tools + `@Service` orchestration | No built-in ReAct loop; implement yourself |
| `ConversationBufferMemory` | `ChatMemory` interface | `InMemoryChatMemory`, `CassandraChatMemory` |
| `ConversationSummaryMemory` | Manual impl via summarization chain | Not built-in; easy to add |
| `VectorStoreRetrieverMemory` | `VectorStore` + custom memory | Combine `PineconeVectorStore` with `ChatMemory` |
| `CallbackHandler` | Various event listeners | Spring AI has fewer hooks; use AOP |
| `SequentialChain` | Manual composition | Spring AI doesn't have chain abstraction |
| `RouterChain` | `@ConditionalOnProperty` / routing logic | No LLM-based routing built-in |

### Implementing a ReAct Agent in Spring Boot

```java
@Service
public class ReActAgentService {

    private final ChatClient chatClient;
    private final List<ToolCallback> tools;
    private final ChatMemory chatMemory;

    public ReActAgentService(ChatClient.Builder builder,
                             List<ToolCallback> tools,
                             ChatMemory chatMemory) {
        this.tools = tools;
        this.chatMemory = chatMemory;
        this.chatClient = builder
            .defaultSystem("""
                You are a helpful assistant. You can use tools.
                Always respond in this format:
                Thought: <reasoning>
                Action: <tool_name>
                Action Input: <input>
                Observation: <result>
                ... (repeat if needed)
                Final Answer: <answer>
                """)
            .build();
    }

    public String execute(String userInput) {
        chatMemory.add(new UserMessage(userInput));
        int maxIterations = 10;
        String response = "";

        for (int i = 0; i < maxIterations; i++) {
            response = chatClient.prompt()
                .messages(chatMemory.get())
                .tools(tools.toArray(new ToolCallback[0]))
                .call()
                .content();

            chatMemory.add(new AssistantMessage(response));

            if (response.contains("Final Answer:")) {
                return response.split("Final Answer:")[1].trim();
            }
        }
        return "Max iterations reached. Last response: " + response;
    }
}
```

---

## 9. Decision Heuristics

### Agent vs. Chain

```
Input → Is the sequence of steps known at design time?
  ├─ YES, fixed steps → Use a Chain (LLMChain, SequentialChain)
  └─ NO, depends on intermediate output → Use an Agent
       └─ Is the task decomposable into clear sub-goals?
            ├─ YES → Plan-and-Execute Agent
            └─ NO → ReAct or Tool-Calling Agent
```

### When to Add Memory

```
Does the application need multi-turn context?
  ├─ NO (one-shot Q&A) → No memory needed
  └─ YES
       ├─ < 10 turns, same topic → ConversationBufferMemory
       ├─ > 10 turns, same topic → ConversationSummaryMemory
       └─ > 50 turns, multi-topic → VectorStoreRetrieverMemory
```

### Tool Granularity

```
Does the tool do more than one "thing"?
  ├─ YES → Split it. One tool = one capability.
  └─ NO
       └─ Can the description fit in 1-2 sentences?
            ├─ YES → Good granularity
            └─ NO → Too complex, consider splitting
```

### Error Handling Strategy

```
Is the error...
  ├─ Recoverable (network timeout, rate limit)?
  │   → Return error string to agent, let it retry
  ├─ Permanent (invalid input, missing resource)?
  │   → Return descriptive error string, agent can try other tools
  └─ Security violation (unauthorized access)?
       → Log and return generic error; never expose internals
```

### Callback Usage

```
Do you need to...
  ├─ Log every LLM call? → on_llm_start/on_llm_end
  ├─ Stream tokens to UI? → on_llm_new_token
  ├─ Track tool usage? → on_tool_start/on_tool_end
  ├─ Trace execution for debugging? → All events
  └─ Alert on errors? → on_tool_error (if available)
```

---

## 10. Key Takeaways from Harrison Chase

1. **Composition over configuration.** LangChain's power comes from being able to mix and match components, not from a massive config file.

2. **Let the LLM decide, but constrain the space.** The agent picks which tool to use and how — but the developer defines the available tools and their boundaries.

3. **Observability is not optional.** Without callbacks/tracing, debugging an agent is nearly impossible. Always instrument from day one.

4. **Fail gracefully.** Every component should degrade gracefully — tools return error strings, agents retry, callbacks never crash the main flow.

5. **Test at every level.** Test tools in isolation, test agents with mock LLMs, test chains with dummy data.

6. **The simplest thing that works.** Don't use an agent when a chain would do. Don't add vector memory for a 3-turn conversation. Start simple, add complexity only when needed.

7. **Memory is a design decision, not an afterthought.** How you handle state determines whether your application works at turn 100 or breaks because the context window overflowed.

---

## References

- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)
- [ReAct Paper (Yao et al., 2023)](https://arxiv.org/abs/2210.03629)
- [Plan-and-Solve Prompting (Wang et al., 2023)](https://arxiv.org/abs/2305.04091)
- [Spring AI Reference](https://docs.spring.io/spring-ai/reference/)
- Harrison Chase talks: "LangChain: The Future of LLM Applications" (various conferences)
