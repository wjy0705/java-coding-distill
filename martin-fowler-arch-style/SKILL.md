---
name: martin-fowler-arch-style
description: Martin Fowler的架构与重构思维蒸馏 - 企业应用架构与代码质量
---

# Martin Fowler — Architecture & Refactoring Style Guide

> **Martin Fowler** (Chief Scientist, ThoughtWorks) — Author of *Refactoring*, *Patterns of Enterprise Application Architecture*, *UML Distilled*, *Refactoring: Improving the Design of Existing Code* (2nd Ed). Co-author of *Continuous Delivery* and the *Agile Manifesto*.

---

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **Martin Fowler** 的架构设计、重构思维与代码质量理念。Martin Fowler 是 ThoughtWorks 首席科学家，《重构》《企业应用架构模式》《UML Distilled》作者，敏捷宣言签署人之一。他的重构目录（Refactoring Catalog）和 Code Smells 分类是软件工程的经典标准，影响力横跨 20 年。

### 参考来源
- **书籍**: 《Refactoring: Improving the Design of Existing Code》(第2版)、《Patterns of Enterprise Application Architecture》、《UML Distilled》、《NoSQL Distilled》
- **博客**: martinfowler.com - 持续 20+ 年的技术博客，数百篇架构/重构/敏捷文章
- **开源**: ThoughtWorks 开源项目中的编码实践
- **技术雷达**: ThoughtWorks TechRadar 的架构选型理念

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：多源信息采集（书籍/博客/技术雷达） → 思维框架提炼（核心哲学/重构手法/架构模式/技术债务分类） → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 接手烂代码时 | 24 种 Code Smells 逐一排查，找到最该重构的地方 |
| 设计系统架构时 | 企业架构模式（领域逻辑/数据源/ORM/Web 层/分布式）做参考 |
| 写代码时 | "两个帽子"思维——时刻分清楚是在加功能还是在重构 |
| 讨论技术债务时 | Fowler 的四象限模型（ reckless/prudent × deliberate/inadvertent ）跟团队对齐 |
| 面试架构题时 | 重构手法、设计模式、架构模式——回答有权威出处 |

---

## 1. Core Philosophy

### The Readability Imperative
> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand."

- **Code is communication** — written primarily for human readers, not machines.
- **Simplicity over cleverness** — the best code is boring, obvious, and easy to reason about.
- **Names matter** — method and variable names should reveal intent. If you need a comment, extract into a method named after the intent.
- **Three Virtues of a Programmer**: Laziness (write once, automate), Impatience (don't wait on computers), Hubris (write code worth being proud of).

### Two Hats Theory
When coding, you are always wearing exactly one of two hats:
1. **Adding Function** — writing new behavior (tests may fail because features are missing)
2. **Refactoring** — restructuring existing code without changing observable behavior (no test failures should occur)

Never wear both hats at once. A single commit/change should be either function or refactoring, never both.

### Evolutionary Design
> "Design is not something you do at the beginning, it's something you do throughout the development process."

- Design **emerges** through disciplined refactoring, not from a Big Design Up Front (BDUF).
- The best architecture comes from teams who can **safely and continuously reshape** their code.
- Architecture decisions should be **reversible** — prefer decisions that are easy to change over ones that are "optimal" but rigid.

---

## 2. Refactoring Approach

### Definition
**Refactoring**: A disciplined technique for restructuring an existing body of code, altering its internal structure without changing its external behavior, using a catalog of behavior-preserving transformations.

### The Golden Rules
1. **Each refactoring is a small, behavior-preserving transformation.**
2. **Test before and after** — if tests aren't green before starting, refactor is dangerous.
3. **Make one change at a time** — tiny steps, commit frequently.
4. **Never refactor and add features in the same change.**
5. **Leave the code cleaner than you found it** (Boy Scout Rule).

### The Refactoring Process
```
1. Identify code smell
2. Write/ensure passing tests
3. Apply one refactoring from catalog
4. Run tests (should pass)
5. Commit
6. Repeat from step 2 or 3
```

### The Refactoring Catalog (Key Examples)

| Refactoring | Intent |
|---|---|
| **Extract Method** | Turn a code fragment into a method named for its intent |
| **Inline Method** | Replace indirect method call with its body |
| **Extract Variable** | Give a name to a complex expression |
| **Inline Temp** | Replace a temp used only once with the expression |
| **Replace Temp with Query** | Calculate derived values via method calls, not local vars |
| **Split Loop** | Separate a loop doing two things into two loops |
| **Slide Statements** | Move related code together |
| **Extract Function** | (JS/Python) Same as Extract Method |
| **Rename Variable / Function** | Improve clarity by renaming |
| **Introduce Parameter Object** | Group related parameters into an object |
| **Combine Functions into Class** | Group related functions with shared data into a class |
| **Combine Functions into Transform** | Derive new data from immutable input |
| **Encapsulate Record / Collection** | Hide raw data structures behind accessors |
| **Replace Primitive with Object** | Promote a small concept (email, money, phone) to its own class |
| **Replace Conditional with Polymorphism** | Use subtype dispatch instead of switch/if-else |
| **Replace Nested Conditional with Guard Clauses** | Flatten deep nesting via early returns |
| **Decompose Conditional** | Extract complex if/else branches into functions |
| **Move Function / Field / Statement** | Place code where it belongs |
| **Pull Up / Push Down Method** | Move methods between super- and subclasses |
| **Replace Inline Code with Function Call** | Use library/utility instead of hand-rolled |
| **Separate Query from Modifier** | A method should either return data OR change state, not both |
| **Parameterize Function** | Unify similar functions by adding a parameter |
| **Remove Dead Code** | Delete unused code immediately |
| **Replace Inheritance with Delegation** | Prefer composition over inheritance |
| **Replace Subclass with Delegate / Replace Superclass with Delegate** | Strategy/State patterns via delegation |

---

## 3. Code Smells (Indicators for Refactoring)

> "If it stinks, change it." — Kent Beck, applied by Martin Fowler

### Smell Catalog

| Smell | Description |
|---|---|
| **Mysterious Name** | The most common smell — names that don't reveal intent |
| **Duplicated Code** | Same structure in more than one place |
| **Long Method** | Everything should be under ~10 lines; extract, extract, extract |
| **Long Parameter List** | Hard to understand and change; introduce parameter objects |
| **Global Data** | Mutable global state is the root of many bugs |
| **Mutable Data** | Favor immutability; encapsulate mutations |
| **Divergent Change** | One module changes for many different reasons |
| **Shotgun Surgery** | One change requires editing many files |
| **Feature Envy** | A method cares more about another class than its own |
| **Data Clumps** | Items that always appear together belong in their own object |
| **Primitive Obsession** | Using primitives for domain concepts (money, email, phone) |
| **Repeated Switches** | The same switch/then-if chain appears in multiple places |
| **Loops** | Replace with higher-order functions (map, filter, reduce) |
| **Lazy Element** | A class/method that doesn't justify its existence |
| **Speculative Generality** | Code for "future needs" that never arrive |
| **Temporary Field** | An attribute only used in some circumstances |
| **Message Chains** | A long chain of calls: `a.b().c().d()` |
| **Middle Man** | A class that mostly delegates to another |
| **Insider Trading** | Excessive coupling between modules |
| **Large Class** | Too many responsibilities; extract class/module |
| **Alternative Classes with Different Interfaces** | Similar concepts but different method signatures |
| **Data Class** | Class with only fields and getters/setters — no behavior |
| **Refused Bequest** | A subclass that doesn't use inherited methods |
| **Comments** | Often used to mask bad code (but sometimes necessary) |

---

## 4. Enterprise Architecture Patterns

From *Patterns of Enterprise Application Architecture* (PoEAA).

### Domain Logic Patterns

| Pattern | Use Case |
|---|---|
| **Transaction Script** | Simple business logic — one procedure per request (like CRUD) |
| **Domain Model** | Complex business logic — object model with behavior and rules |
| **Table Module** | Single class per DB table, works with RecordSet (common in .NET) |
| **Service Layer** | Defines application boundary and transaction control |

### Data Source Architectural Patterns

| Pattern | Use Case |
|---|---|
| **Table Data Gateway** | Gateway to a single table — CRUD operations only |
| **Row Data Gateway** | One object per row in the result set |
| **Active Record** | Row data + domain logic in same object (data-carrying + behavior) |
| **Data Mapper** | Separates in-memory objects from DB (ORM layer) |
| **Repository** | Mediates between domain and data mapping — feels like a collection |

### Object-Relational Behavioral Patterns

| Pattern | Purpose |
|---|---|
| **Unit of Work** | Tracks changes during a business transaction, flushes at commit |
| **Identity Map** | Ensures each object is loaded only once per transaction |
| **Lazy Load** | Defers loading until the object is actually accessed |

### Object-Relational Structural Patterns

| Pattern | Purpose |
|---|---|
| **Single Table Inheritance** | One DB table for entire class hierarchy (simple, denormalized) |
| **Class Table Inheritance** | One table per class in hierarchy (normalized, complex joins) |
| **Concrete Table Inheritance** | One table per concrete class (complete, but duplicate columns) |
| **Inheritance Mappers** | Map inheritance hierarchies to DB tables |

### Web Presentation Patterns

| Pattern | Purpose |
|---|---|
| **Model-View-Controller (MVC)** | Separates data, presentation, and input handling |
| **Page Controller** | One page/action handler per URL |
| **Front Controller** | Single handler for all requests (like Router) |
| **Template View** | Render HTML by embedding code in template |
| **Transform View** | Transform domain data to HTML via XSLT/transform |
| **Two Step View** | Two-stage rendering (content then layout) |
| **Application Controller** | Centralized screen flow/navigation logic |

### Distribution & Concurrency Patterns

| Pattern | Purpose |
|---|---|
| **Remote Facade** | Coarse-grained facade over fine-grained objects for remote access |
| **Data Transfer Object (DTO)** | Plain data container for serialization across network |
| **Optimistic Offline Lock** | Read without lock, check version on write |
| **Pessimistic Offline Lock** | Lock resource at read time, release after write |
| **Coarse-Grained Lock** | Lock a group of related objects as one unit |
| **Implicit Lock** | Framework auto-acquires/auto-releases locks |

---

## 5. Code Organization Principles

### Modularity & Structure
- **Separate the domain from the infrastructure** — domain logic should be pure and framework-independent.
- **Hexagonal Architecture (Ports & Adapters)** — Fowler was an early proponent: business logic at the core, adapters around it for UI/DB/external.
- **Package by feature, not by layer** — group by business concept, not technical concern.
- **Screaming Architecture** — the package structure should scream the business domain, not the framework.

### Method Size
- A method should rarely exceed **5–10 lines**.
- If you need to scroll, extract.
- If you feel the urge to write a comment inside a method, extract that section as a method named after the comment.

### Composition over Inheritance
- "Inheritance is good, but delegation is often better."
- Prefer **Strategy**, **State**, and **Template Method** patterns over deep inheritance hierarchies.

---

## 6. Testing Philosophy

### Self-Testing Code
- A system should be able to **test itself** with a single command.
- Without tests, refactoring is impossible — you can't verify behavior preservation.
- Tests provide the **safety net** that makes continuous refactoring viable.

### Test Characteristics
- **Fast** — tests should run quickly so developers run them often
- **Deterministic** — same test, same result, every time
- **Isolated** — no test depends on another
- **Expressive** — test names describe behavior, not implementation
- **Independent of refactoring** — tests should only break when behavior changes, not when structure changes

### Test-Driven Development (TDD)
- Fowler is not dogmatic about TDD but recognizes its value:
  - Red → Green → Refactor
  - Test-first provides design feedback (testability = good design)
  - Tests document the intended behavior

### Types of Tests
- **Unit Tests** — single class/module in isolation, fast (< 1ms each)
- **Integration Tests** — boundary crossing (DB, network, filesystem)
- **Contract Tests** — verify API/service contracts
- **End-to-End Tests** — full system, small number, slow

> "Good tests act as documentation for what the code does."

---

## 7. Technical Debt

### Definition
> "Technical debt is a metaphor, coined by Ward Cunningham, that equates bad design and implementation choices to financial debt — you get short-term speed at the cost of long-term interest payments."

### Fowler's Taxonomy
- **Reckless vs. Prudent**
  - *Reckless*: "We don't have time for design" (intentional bad code)
  - *Prudent*: "We'll ship now and refactor later" (conscious trade-off)
- **Deliberate vs. Inadvertent**
  - *Deliberate*: Known and chosen
  - *Inadvertent*: Introduced by ignorance of better design

| | Deliberate | Inadvertent |
|---|---|---|
| **Reckless** | "No time, ship it" | "What's a design pattern?" |
| **Prudent** | "Ship now, refactor next sprint" | "We didn't know what we didn't know" |

### Managing Debt
- **Pay interest regularly** — small, continuous refactoring is cheaper than major rewrites.
- **Refactor before adding to smelly code** — reduce friction before extending.
- **The Big Rewrite is almost always a mistake** — incremental improvement is safer.
- **Track debt visually** — hot spots in the codebase that cost the team time.

---

## 8. Agile & Continuous Delivery

### Agile Manifesto (Co-author, 2001)
> **Individuals and interactions** over processes and tools
> **Working software** over comprehensive documentation
> **Customer collaboration** over contract negotiation
> **Responding to change** over following a plan

### Continuous Delivery (with Jez Humble)
- **Deployment Pipeline** — every commit triggers build → test → stage → release.
- **Everything as Code** — infrastructure, configuration, database migrations.
- **Automated everything** — no manual steps from commit to production.
- **Low-risk releases** — small, frequent, reversible deployments.
- **Feature Toggles** — deploy dark, release when ready.
- **Blue/Green Deployments** — minimize downtime and risk.

### Microservices
- Fowler was an early and prominent voice on microservices.
- **Microservices** = independently deployable services modeled around business domains.
- **Smart endpoints, dumb pipes** — business logic in services, not in the messaging layer.
- **Decentralized governance** — teams choose their own tech stacks.
- **Decentralized data management** — each service owns its database.
- **Evolutionary design** — microservices allow independent evolution of subsystems.
- **Warning**: Don't start with microservices — evolve from a monolith as boundaries emerge.

---

## 9. Anti-Patterns & Traps

| Anti-Pattern | Fowler's View |
|---|---|
| **Big Design Up Front** | Resists change; produces wrong architecture when reality hits |
| **Premature Optimization** | "The root of all evil" (quoting Knuth, applying to architecture) |
| **Golden Hammer** | Using a familiar pattern/solution for every problem |
| **Big Rewrite** | High risk, low chance of success vs. incremental refactoring |
| **Architecture Astronaut** | Designing abstractions without building real code |
| **Analysis Paralysis** | Over-planning instead of building and adapting |
| **Not Invented Here** | Rejecting good libraries/frameworks to build custom |
| **Vendor Lock-in** | Using proprietary tech that makes switching difficult |
| **Framework Obsession** | Letting the framework dictate your architecture |
| **Lava Layer** | The last generation's architecture becomes today's foundation, even if outdated |

---

## 10. Decision Heuristics & Guidelines

### When to Refactor
1. **Rule of Three** — when you are about to add the third instance of similar code, refactor first.
2. **Difficulty adding features** — if adding a new feature is painful, the design needs improvement.
3. **Bug-prone areas** — if bugs cluster in certain modules, refactor those modules.
4. **Code review feedback** — if a pattern is questioned, consider refactoring.
5. **Before extending smelly code** — always clean up before adding to confusing code.

### Design Trade-off Heuristics
- **Duplication is cheaper than the wrong abstraction** — don't abstract prematurely.
- **Prefer value objects (immutable) over entity objects (mutable) where possible**.
- **Infrastructure decisions are postponable** — you can always add a DB, a queue, or a cache later.
- **A monolith you can refactor is better than a microservices architecture you can't change.**
- **When in doubt, leave the code in a better state than you found it.**

### Pattern Selection
- Start with **Transaction Script**; evolve to **Domain Model** as logic grows.
- Start with **Active Record**; evolve to **Data Mapper** when mapping grows complex.
- Use **Repository** when you need collection-like access to aggregates.
- Use **Service Layer** when you need transaction control and authorization boundaries.

> "I'm not a big fan of trying to figure out all your patterns in advance. The patterns you need will emerge as you build the system."

---

## 11. Key References

| Work | Relevance |
|---|---|
| *Refactoring: Improving the Design of Existing Code* (2nd Ed, 2018) | Definitive refactoring catalog |
| *Patterns of Enterprise Application Architecture* (2002) | Enterprise patterns reference |
| *UML Distilled* (3rd Ed, 2003) | Concise UML guide |
| *Continuous Delivery* (with Jez Humble, 2010) | CI/CD foundational text |
| *Refactoring: Ruby Edition* (with Jay Fields, 2007) | Ruby-focused refactoring |
| martinfowler.com | Extensive articles on all topics above |

---

## Summary: Fowler's DNA

```
CORE: Readability > Cleverness | Small Steps > Big Bang | Tests > Trust
PROCESS: Smell → Refactor → Test → Commit → Repeat
ARCHITECTURE: Domain Core → Ports & Adapters → Evolve as Needed
DELIVERY: Continuous Integration → Continuous Delivery → Feedback Loop
MINDSET: "It's harder to read code than to write it. Make reading easy."
```
