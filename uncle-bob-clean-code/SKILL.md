---
name: uncle-bob-clean-code
description: Uncle Bob的Clean Code与Clean Architecture蒸馏 - SOLID原则与代码质量
---

# Uncle Bob's Clean Code — Skill Reference

## 📋 概览

### 这是什么？
本 Skill 系统蒸馏了 **Robert C. Martin（Uncle Bob）** 的 Clean Code 与 Clean Architecture 理念。Uncle Bob 是《Clean Code》《Clean Architecture》《The Clean Coder》作者，SOLID 原则创始人，敏捷宣言签署人。SOLID 原则是面向对象设计的基石，至今仍是几乎所有 Java 面试中的必考内容。

### 参考来源
- **书籍**: 《Clean Code: A Handbook of Agile Software Craftsmanship》《Clean Architecture》《The Clean Coder》《Agile Software Development: Principles, Patterns, and Practices》
- **博客**: blog.cleancoder.com - Uncle Bob 的持续写作与思想输出
- **演讲**: 各类技术会议中的 Clean Code / Clean Architecture / TDD 主题演讲
- **社区影响**: SOLID 原则被全球开发者引用，是 OOD 领域引用量最高的原则集之一

### 蒸馏工具
本 Skill 由 **[女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill)** 蒸馏生成。提炼流程：多源信息采集（书籍/博客/演讲） → 思维框架提炼（SOLID/Clean Code规则/Clean Architecture/TDD/Professionalism） → 质量验证 → Skill 装配。

### 能帮你解决什么？
| 场景 | 解决什么问题 |
|------|------------|
| 写类和方法时 | SOLID 原则逐条检查——类够单一吗？接口够小吗？依赖够抽象吗？ |
| 设计函数时 | Clean Code 函数规则——够短吗？只做一件事吗？参数够少吗？ |
| 设计系统架构时 | Clean Architecture 依赖规则——外层依赖内层？接口属于内层？ |
| 写测试时 | TDD 三定律、FIRST 原则、单断言测试——照着写就是好测试 |
| 面试 Java/架构岗时 | "解释 SOLID 原则""怎么做 Clean Architecture"——这类题的标答来源 |

---
## 1. SOLID Principles (In Detail)

### S — Single Responsibility Principle (SRP)
> "A class should have one, and only one, reason to change."

- A **responsibility** is a reason to change. If a class has more than one reason to change, it has more than one responsibility.
- Every module, class, or function should be responsible for a single part of the software's functionality.
- **Signs of SRP violation**: a class that touches persistence, business logic, and presentation; a method that both computes a result AND formats/prints it.
- **Example violation**:
  ```python
  # BAD: Three reasons to change (report format, employee data, hours policy)
  class Employee:
      def calculate_pay(self): ...
      def report_hours(self): ...
      def save(self): ...
  ```
- **Example fix**:
  ```python
  # GOOD: Separate concerns
  class Employee:
      def __init__(self, name, rate): ...
  
  class PayCalculator:
      def calculate_pay(self, employee): ...
  
  class HourReporter:
      def report_hours(self, employee): ...
  
  class EmployeeRepository:
      def save(self, employee): ...
  ```

### O — Open/Closed Principle (OCP)
> "Software entities should be open for extension, but closed for modification."

- You should be able to add new functionality **without changing existing code**.
- Achieve via **abstraction** and **polymorphism** — depend on interfaces/abstract base classes, not concrete implementations.
- The goal: introduce new features by writing NEW code, not editing old, tested code.
- **Example violation**:
  ```python
  # BAD: Adding a new shape requires modifying draw_all()
  def draw_all(shapes):
      for s in shapes:
          if s.type == "circle":
              draw_circle(s)
          elif s.type == "square":
              draw_square(s)
  ```
- **Example fix**:
  ```python
  # GOOD: Each shape knows how to draw itself
  class Shape(ABC):
      @abstractmethod
      def draw(self): ...
  
  class Circle(Shape):
      def draw(self): draw_circle(self)
  
  class Square(Shape):
      def draw(self): draw_square(self)
  
  def draw_all(shapes):
      for s in shapes:
          s.draw()  # No modification needed for new shapes
  ```

### L — Liskov Substitution Principle (LSP)
> "Subtypes must be substitutable for their base types."

- If S is a subtype of T, then objects of type T should be replaceable with objects of type S **without altering the correctness** of the program.
- Derived classes must **preserve the behavior** promised by the base class interface.
- **Classic violation**: The `Rectangle`/`Square` problem — a Square extends Rectangle but violates width/height independence.
  ```python
  # BAD: Square breaks Rectangle behavior
  class Rectangle:
      def set_width(self, w): self.width = w
      def set_height(self, h): self.height = h
      def area(self): return self.width * self.height
  
  class Square(Rectangle):
      def set_width(self, w):
          self.width = w
          self.height = w  # violates LSP — caller expects independent dims
      def set_height(self, h):
          self.height = h
          self.width = h
  ```
- **Better approach**: Use a `Shape` base class with `area()` as abstract; don't force inheritance where behavior differs.
- **Other signs**: Overriding a method to throw `NotImplementedError` or `raise`; checking `isinstance`/`type()` before using a subtype.

### I — Interface Segregation Principle (ISP)
> "Clients should not be forced to depend on interfaces they do not use."

- Large, "fat" interfaces should be split into smaller, more specific ones.
- A class should not be forced to implement methods it doesn't need.
- **Example violation**:
  ```python
  # BAD: Fat interface
  class Worker(ABC):
      @abstractmethod
      def work(self): ...
      @abstractmethod
      def eat(self): ...
  
  class Robot(Worker):
      def work(self): ...
      def eat(self): raise Exception("Robots don't eat")  # forced dependency
  ```
- **Example fix**:
  ```python
  # GOOD: Segregated interfaces
  class Workable(ABC):
      @abstractmethod
      def work(self): ...
  
  class Eatable(ABC):
      @abstractmethod
      def eat(self): ...
  
  class Human(Workable, Eatable):
      def work(self): ...
      def eat(self): ...
  
  class Robot(Workable):
      def work(self): ...
  ```

### D — Dependency Inversion Principle (DIP)
> "Depend on abstractions, not on concretions."

1. High-level modules should not depend on low-level modules. **Both should depend on abstractions.**
2. Abstractions should not depend on details. **Details should depend on abstractions.**
- The goal is to decouple high-level policy from low-level implementation.
- **Example violation**:
  ```python
  # BAD: High-level depends on low-level directly
  class MySQLDatabase:
      def save(self, data): ...
  
  class UserService:
      def __init__(self):
          self.db = MySQLDatabase()  # tight coupling
  ```
- **Example fix**:
  ```python
  # GOOD: Both depend on abstraction
  class Database(ABC):
      @abstractmethod
      def save(self, data): ...
  
  class MySQLDatabase(Database):
      def save(self, data): ...
  
  class UserService:
      def __init__(self, db: Database):  # abstraction injected
          self.db = db
  ```

---

## 2. Clean Code Philosophy

### Naming
> "The name of a variable, function, or class should answer all the big questions: why it exists, what it does, and how it is used."

- **Use intention-revealing names**: `elapsed_time_in_days` not `d`; `customer_list` not `list1`.
- **Avoid disinformation**: Don't use `hp`, `aix`, `sco` (platform-specific terms); avoid `XYZData` and `XYZInfo` — what does "Data" or "Info" actually mean?
- **Make meaningful distinctions**: `money_amount` vs `money` is noise; `customer_info` vs `customer` is noise.
- **Use pronounceable names**: `generation_timestamp` not `genyyyymmddhhmm`.
- **Use searchable names**: Single-letter names only as local loop vars; `MAX_CLASSES_PER_STUDENT` is searchable, `7` is not.
- **Class names**: Nouns or noun phrases (`Customer`, `WikiPage`, `Account`). Avoid `Manager`, `Processor`, `Data`, `Info`.
- **Method names**: Verbs or verb phrases (`delete_page`, `save`, `get_name`). Accessors, mutators, predicates should be prefixed with `get`, `set`, `is`.
- **Pick one word per concept**: Don't mix `fetch`, `retrieve`, and `get`. Pick one and stick with it.
- **Don't pun**: Avoid using the same word for two different purposes. "Add" means "append one to another" — don't use it for "concatenate two lists into a new one."

### Functions
> "The first rule of functions is that they should be small. The second rule of functions is that they should be smaller than that."

- **Functions should do ONE thing**: Do one thing, do it well, do it only.
- **Stay small**: Each function should fit on one screen (~20-30 lines max).
- **Blocks within if/else/while should be one line** — usually a function call.
- **One level of abstraction per function**: Don't mix high-level logic (`save_customer(customer)`) with low-level details (`stmt = conn.prepare("INSERT INTO...")`).
- **Switch statements**: Bury them in an abstract factory and never let them appear twice.
- **Function arguments**: 
  - 0 arguments (niladic) = ideal
  - 1 argument (monadic) = good
  - 2 arguments (dyadic) = okay
  - 3+ arguments (triadic+) = avoid; consider refactoring into an object
- **Flag arguments are terrible**: `render(true)` should be broken into `render_bold()` and `render_normal()`.
- **Do not use output arguments**: `append_footer(s)` mutates `s` by reference. Instead: `s.append_footer()` or `s = s.with_footer()`.
- **DRY — Don't Repeat Yourself**: Extract repeated code into a named function.
- **Side effects**: A function should not do hidden work; no mutation of global state, no writing to a file named "create_order" when the function is called `check_order`.
- **Command-query separation**: A function should either change state (command) or return a value (query), but not both.

### Comments
> "Don't comment bad code — rewrite it."
> "Comments are, at best, a necessary evil."

- Comments **lie** — code changes over time, comments don't always follow.
- The **only truly good comment** is the one you found a way **not to write**.
- **Good uses for comments**:
  - Legal comments (copyright, license)
  - Informative comments that explain intent (not implementation)
  - Clarification of a non-obvious result from a third-party library
  - Warning of consequences (`// Don't run unless you have 8GB+ RAM`)
  - TODO comments (`// TODO: replace with faster sort`)
  - Amplification (`// The trim is especially important because...`)
- **Bad comments (never write)**:
  - Redundant: `i++;  // increment i` — the code already says this
  - Misleading: comments that don't match the code behavior
  - Mandated: "Every function must have a Javadoc"
  - Journal comments: change logs belong in version control
  - Noise comments: `// default constructor` on a default constructor
  - Commented-out code: delete it. Git remembers it.
  - Banners/block markers: `// =========== SECTION ==========`

### Formatting
> "Code formatting is about communication, and communication is the professional developer's first order of business."

- **Vertical formatting**:
  - Files should be ~200 lines; rarely exceed 500
  - Related concepts should be vertically close (high cohesion)
  - Variable declarations at the top of each block
  - Dependent functions should be physically close
  - Blank lines separate concepts (e.g., one blank line between methods)
- **Horizontal formatting**:
  - Lines capped at ~80-120 characters
  - Use spaces around operators and parentheses
  - Indentation: either 2 or 4 spaces — be consistent
  - Break long expressions into intermediate variables
- **Team rules**: A team should agree on one set of formatting rules and apply them universally. Automate with linters/formatters.

### Error Handling
> "Error handling is important, but if it obscures logic, it's wrong."

- Use **exceptions**, not error codes — don't pollute return values with error flags.
- Write the **try-catch-finally** statement first: define the scope and expected errors, then fill in the logic.
- **Provide context** with exceptions: include the operation that failed and the type of failure.
- Define **exception classes in terms of a caller's needs** — classify by source of error (e.g., `StorageException`, `NetworkException`), not by low-level details.
- **Don't return null**: Return empty collections, special-case objects, or use Optionals.
  - `return new ArrayList<>()` instead of `return null`
  - `return Optional.empty()` instead of `return null`
- **Don't pass null**: Avoid passing null into functions; throw `NullPointerException` at the caller or use `Objects.requireNonNull()`.
- **Use the Special Case pattern** (Null Object pattern): create a do-nothing implementation of an interface instead of returning null.

---

## 3. Clean Architecture (by Robert C. Martin)

### The Dependency Rule
> "Source code dependencies must point only inward, toward the highest-level policies."

**The concentric layers** (from outermost to innermost):

```
┌─────────────────────────────┐
│   Frameworks & Drivers      │  (Web, DB, UI, Devices)
│   ┌─────────────────────┐   │
│   │  Interface Adapters  │   │  (Controllers, Gateways, Presenters)
│   │  ┌───────────────┐   │   │
│   │  │  Use Cases     │   │   │  (Application Business Rules)
│   │  │  ┌─────────┐   │   │   │
│   │  │  │ Entities │   │   │   │  (Enterprise Business Rules)
│   │  │  └─────────┘   │   │   │
│   │  └───────────────┘   │   │
│   └─────────────────────┘   │
└─────────────────────────────┘
```

- **Entities**: Enterprise-wide business rules. They are the most stable, highest-level constructs. They should have no dependency on frameworks, databases, or delivery mechanisms.
- **Use Cases**: Application-specific business rules. Orchestrate the flow of data to/from entities. Depend only on entities.
- **Interface Adapters**: Convert data between the form most convenient for use cases/entities and the form most convenient for external agencies (DB, Web, UI). Includes Controllers, Presenters, Gateways.
- **Frameworks & Drivers**: The outermost layer. Concrete infrastructure — databases, web frameworks, device drivers. Keep as thin as possible.

**The Dependency Rule states**:
- Source code dependencies can ONLY point inward.
- Nothing in an inner circle can know anything about an outer circle.
- Outer circles IMPLEMENT interfaces defined by inner circles (DIP in action).

### Boundaries
- **Separate concerns with boundaries**: Place boundaries between different layers of the system.
- **Plugin architecture**: Treat lower-level components as plugins to higher-level policy.
- **Crossing boundaries**: Use interfaces defined in the inner layer, implemented in the outer layer.
- **Example**: The `UserRepository` interface (in the use-cases layer) is defined as an abstraction; the `SQLUserRepository` (in the interface-adapters layer) implements it. The use-case code never imports anything from the persistence layer.

### Use Cases
- A use case is a single, focused interaction between an actor (user or system) and the system.
- **Each use case should produce a single output**: either a success response or an error response.
- Use case objects contain only business logic — they should not know about HTTP, databases, or UI.
- **Data flow**: Input comes in as a simple request object → use case processes it → output comes out as a simple response object.
- A use case can be as simple as `CreateOrder`, `PlaceBid`, `ApproveInvoice`.
- Use cases **own the request/response models**: Don't pass Entity objects to the UI or DB directly. Create dedicated request/response DTOs per use case.

### Screaming Architecture
> "Your architecture should scream: this is a **Rails app** or this is a **banking system**."

- Your project structure should reveal the **intent** of the system, not the frameworks it uses.
- BAD: `controllers/`, `models/`, `views/` (cries "Rails" not "Healthcare System")
- GOOD: `orders/`, `inventory/`, `shipping/`, `billing/` (cries "E-commerce System")
- The architecture should be independent of frameworks, databases, UI, and external agencies.

---

## 4. Testing Philosophy

### TDD — Test-Driven Development
> "You are not allowed to write any production code unless it is to make a failing unit test pass."

**The Three Laws of TDD**:
1. **You may not write production code** until you have written a failing unit test.
2. **You may not write more of a unit test** than is sufficient to fail, and not compiling is failing.
3. **You may not write more production code** than is sufficient to pass the currently failing test.

**The Red-Green-Refactor cycle**:
- **Red**: Write a test that fails (defines the behavior you want)
- **Green**: Write the minimal code to make it pass
- **Refactor**: Clean up the code (both test and production) while keeping tests green

**Benefits**:
- Provides **fearless refactoring** — you can change code with confidence
- Forces **decoupled, testable design** — if it's hard to test, the design is wrong
- Produces a **regression suite** that catches mistakes
- **Reduces debugging time**: if tests pass, you know your code works

### One Assert Per Test (or One Concept Per Test)
> "Each test should test one thing, and one thing only."

- A test should assert a single logical outcome.
- Don't pack multiple assertions into one test — if the first assertion fails, you know nothing about the rest.
- **Limit**: Ideally 1 assert per test. Pragmatically, 2-3 assertions that all test the same **concept** are acceptable.
- **Example approach**:
  ```python
  def test_should_return_0_when_no_items():
      cart = ShoppingCart()
      assert cart.total() == 0
  
  def test_should_return_item_price_with_one_item():
      cart = ShoppingCart()
      cart.add_item(Item(price=10))
      assert cart.total() == 10
  ```

### FIRST Principles of Good Unit Tests

| Letter | Principle | Meaning |
|--------|-----------|---------|
| **F** | **Fast** | Tests should run quickly. Slow tests won't be run often. |
| **I** | **Independent** | Tests should not depend on each other. No shared state. No order dependencies. |
| **R** | **Repeatable** | Tests should produce the same result every time, in any environment. No external dependencies that flake. |
| **S** | **Self-validating** | Tests should have a boolean output — pass or fail. No manual inspection of log files. |
| **T** | **Timely** | Tests should be written **just before** the production code (TDD). If written after, the production code is already hard to test. |

### Testing Patterns Uncle Bob Endorses

- **Unit tests** exercise individual functions/classes in isolation (using mocks/stubs for external dependencies).
- **Integration tests** exercise boundaries (DB, network, file system) but are separate from unit tests.
- **Test one thing**: Name tests with the pattern `test_[scenario]_[expected_behavior]`.
- **Don't depend on test coverage percentages**: 100% coverage is meaningless if the tests don't actually test correctness.
- **Tests are part of the system**: They must be maintained with the same discipline as production code.
- **Never ship with a broken test**: The build should fail if any test fails.

---

## 5. Professionalism in Software Engineering

> "The only way to go fast is to go well."

### Core Professional Responsibilities

1. **Know your craft**
   - Know the tools, languages, frameworks, and paradigms of your field.
   - Read books (Clean Code, The Clean Coder, Clean Architecture, Code Complete, Design Patterns).
   - Study non-software fields (project management, communication, psychology).
   - Practice deliberately — do kata exercises (Bowling Game, Prime Factors, String Calculator).

2. **Say "No" professionally**
   - You cannot deliver the impossible. Saying "yes" when you know it's impossible is unprofessional.
   - Estimate with ranges (e.g., "3-5 days") and refine as you learn.
   - Bad news must be delivered early and often — never let a problem fester.

3. **Saying "Yes" is a commitment**
   - When you say "yes" to a task, you are committing to see it through.
   - If you can't meet the commitment, communicate **immediately** — do not wait until the deadline.

4. **Test-Driven Development is non-negotiable**
   - "TDD gives you the courage to refactor. Without it, you will freeze your codebase."
   - A professional developer writes tests. Period.

5. **Continuous improvement**
   - Every time you check in code, make it **a little bit better** than when you checked it out (the Boy Scout rule).
   - Refactor mercilessly. Leave modules cleaner than you found them.

6. **Don't create chaos**
   - Don't introduce bugs through laziness or haste.
   - "You are a professional — you test your own code before it leaves your hands."

7. **Ethics**
   - Software controls modern life (medical devices, airplanes, banking, elections). Carelessness costs lives.
   - Do no harm to the company, the customer, or the user through sloppy work.

### The Boy Scout Rule
> "Leave the campground cleaner than you found it."

- When you edit a file, leave it a little better — fix a bad variable name, extract a function, improve a comment.
- Over time, this compounds and the codebase improves naturally.

---

## 6. Anti-Patterns & What Uncle Bob Hates

### In Code

| Anti-Pattern | Why He Hates It | Fix |
|---|---|---|
| **Long functions** (>30 lines) | Impossible to understand, impossible to test | Extract tiny, named functions doing one thing |
| **Switch statements** | Violate OCP; grow forever | Replace with polymorphism or abstract factory |
| **Magic numbers** | No intent revealed | Use named constants (`const int MAX_RETRIES = 3`) |
| **Large classes** (>500 lines) | Too many responsibilities (SRP violation) | Split into focused classes |
| **Flag arguments** | Functions doing two things | Split into two functions (`render_bold()` vs `render()`) |
| **Null returns** | Forces callers to do null checks everywhere | Return empty collections or Optionals |
| **Constructor over-injection** (>3-4 deps) | Class does too much | Split the class or group dependencies |
| **Feature envy** | A method uses more features of another class than its own | Move the method to the class it envies |
| **Shotgun surgery** | One change requires touching many files | Good design isolates change to one place |
| **Data clumps** | Same group of fields repeated (address: street, city, zip) | Extract into a new class |

### In Architecture

| Anti-Pattern | Why He Hates It | Fix |
|---|---|---|
| **Big ball of mud** | No separation of concerns | Apply Clean Architecture layering |
| **Framework coupling** | Hard to upgrade, hard to test, hard to replace | Depend on abstractions, not the framework |
| **Anemic domain model** | Classes are just data bags with no behavior | Move business logic into the domain objects |
| **Common closure principle violation** | Things that change together are not kept together | Group code by change axis |
| **Coding by exception** | Using exceptions for control flow | Exceptions are for exceptional conditions only |
| **God class** | One class does everything | SRP: split into focused classes |
| **Circular dependency** | Impossible to build, test, or reason about | Use DIP to invert the dependency |

### On Professionalism

| Anti-Pattern | Why He Hates It | Fix |
|---|---|---|
| **Meritocracy myth** | "Some people are just 10x engineers and don't need process" | Discipline and process prevent the 10x engineer from making 10x the mess |
| **Last-minute heroics** | "Saving the day" by working all night | Prevents root-cause analysis; encourages bad estimates |
| **Cranking out code without tests** | Creates bugs, freezes codebase, destroys courage | TDD: test first, code second |
| **Not saying No** | Leads to burnout, failed projects, low quality | Say "no" early, professionally |
| **WiFi at conferences** | Working during presentations means you miss learning | Be present, dedicated to the craft |
| **"It works on my machine"** | Willful ignorance of the deployment environment | Use CI/CD, containers, proper environment parity |

### On Comments (Specifically His Hatred)

| Bad Comment | Example | Why It's Bad |
|---|---|---|
| Redundant | `i++; // increment i` | Code already communicates intent |
| Journaling | `// 2004-05-13 Bob: Added fix for bug #42` | Git has the history |
| Noise | `// default constructor` on an empty constructor | Adds nothing |
| Position marker | `// ##### END SECTION #####` | Visual noise; refactor into smaller files |
| Commented-out code | `// if (x > 10) { do_thing(); }` | Confusing and never cleaned up |

### Known Uncle Bob Quotes (Sharp Edges)

- "The only way to go fast is to go well."
- "Truth can only be found in one place: the code."
- "A comment is a failure to express yourself in code."
- "Functions should be very small. Really, really small."
- "A long descriptive name is better than a short enigmatic name."
- "It is not enough for code to work."
- "Clean code always looks like it was written by someone who cares."
- "You are a professional. Test your own damn code."
- "The ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code. Making it easy to read makes it easier to write."

---

## 7. Summary Cheat Sheet — Quick Reference

### SOLID

| Principle | Tagline |
|-----------|---------|
| **S**RP | One class, one reason to change |
| **O**CP | Open for extension, closed for modification |
| **L**SP | Subtypes must be substitutable for their base types |
| **I**SP | Don't depend on interfaces you don't use |
| **D**IP | Depend on abstractions, not concretions |

### Function Checklist
- [ ] Is it small? (< 20 lines)
- [ ] Does it do one thing?
- [ ] One level of abstraction?
- [ ] No side effects?
- [ ] No flag arguments?
- [ ] Command or query, not both?
- [ ] Intention-revealing name?

### Class Checklist
- [ ] SRP: one responsibility?
- [ ] High cohesion?
- [ ] Reasonable number of dependencies?
- [ ] Depends on abstractions?
- [ ] Testable without infrastructure?

### Test Checklist
- [ ] Written before production code? (TDD)
- [ ] Tests one concept?
- [ ] Fast?
- [ ] Independent?
- [ ] Repeatable?
- [ ] Self-validating?

---

*This skill distills Robert C. Martin (Uncle Bob)'s teachings from Clean Code (2008), The Clean Coder (2011), and Clean Architecture (2017). For deeper study, read the books and practice the kata exercises.*
