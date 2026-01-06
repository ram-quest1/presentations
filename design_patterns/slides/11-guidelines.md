## When to Use Design Patterns

> **Design patterns are tools, not rules.** Use them to solve problems, not to show off.

--

### ✅ Use Patterns When

| Situation | Why |
|-----------|-----|
| You recognize a recurring problem | Patterns are proven solutions |
| Code is getting hard to change | Patterns improve flexibility |
| Team needs shared vocabulary | "Use a Factory" is clearer than explaining from scratch |
| Requirements will likely change | Patterns anticipate change |
| You're building frameworks/libraries | Users expect extensibility |

```java
// Good: Database connections are expensive, pool makes sense
ConnectionFactory.getConnection();

// Good: Many listeners need stock updates
stock.addObserver(investor);
```

--

### ❌ Don't Use Patterns When

| Situation | Why |
|-----------|-----|
| Simple code works fine | Don't add complexity for no reason |
| You're forcing it to fit | Wrong pattern is worse than none |
| "Just in case" | YAGNI - You Aren't Gonna Need It |
| Team doesn't understand it | Clever code is hard to maintain |
| Performance is critical | Some patterns add overhead |

```java
// Bad: Singleton for a simple utility
Logger.getInstance().log("Hi");  // Just use static method!

// Bad: Factory for one type that never changes
CarFactory.createCar();  // Just use: new Car()

// Bad: Observer for two objects
subject.notify();  // Just call the method directly!
```

--

### The Golden Rule

```text
Before applying a pattern, ask:

1. What problem am I solving?
2. Does this pattern actually solve it?
3. Is the added complexity worth it?
4. Will my team understand this?

If you can't answer YES to all four → don't use the pattern.
```

--

### Complexity vs Benefit

```text
        │
Benefit │                      ★ Sweet spot
        │                    ╱
        │                  ╱
        │                ╱
        │              ╱
        │       ─────────────── Diminishing returns
        │     ╱
        │   ╱
        │ ╱
        └──────────────────────────
                Complexity

Start simple. Add patterns when pain appears. Rule of 3.
```

--

### Pattern Smells (You're Overdoing It)

- Every class has an interface "just in case"
- Factory that only creates one type
- Singleton for everything
- 5 classes to do what 1 could do
- Pattern names in class names: `UserFactorySingletonProxy`
- You can't explain why you used it

--

### Good Approach

```text
1. Write simple code first
          ↓
2. Feel the pain (duplication, rigidity, bugs)
          ↓
3. Recognize the pattern that fits
          ↓
4. Refactor to use pattern
          ↓
5. Enjoy cleaner code
```

> **"Make it work, make it right, make it fast"** — Kent Beck

--

### Remember

> **Patterns are discovered, not invented.**

They emerged from real problems. If you don't have the problem, you don't need the solution.

The best code is simple code that works. Add patterns only when they make the code **simpler to understand and change** — not more impressive.