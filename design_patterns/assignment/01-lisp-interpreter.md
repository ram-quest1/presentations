## Assignment: Building a LISP Interpreter in Java

### Objective

Your goal is to implement a basic **LISP Interpreter** using Java. This project will test your understanding of tree-based data structures and, more importantly, your ability to apply core **Gang of Four (GoF)** design patterns: **Visitor, Factory, and Singleton.**

--

### Project Scope

You will build a system that can parse and evaluate simple Lisp-style S-expressions. The interpreter should support:

* **Arithmetic operations:** `(+ 1 2)`, `(* 3 4)`
* **Variable definitions:** `(define x 10)`
* **Basic Logic:** `(if (> x 5) 1 0)`

--

### Phase 1: The AST & Visitor Pattern

LISP is naturally represented as an Abstract Syntax Tree (AST). Instead of putting evaluation logic inside your node classes (which violates the Single Responsibility Principle), you will use the **Visitor Pattern**.

* **The Nodes:** Create an interface `Node` with an `accept(Visitor v)` method. Implement this for `NumberNode`, `SymbolNode`, and `ListNode`.
* **The Visitor:** Create a `Visitor` interface with `visit` methods for each node type.
* **The Evaluator:** Implement an `EvaluationVisitor` that traverses the tree and computes the results.

--

### Phase 2: Object Creation & The Factory Pattern

To decouple the parser from the specific implementations of your AST nodes, you must use a **Factory Pattern**.

* **The Task:** Create a `NodeFactory` class.
* **The Implementation:** The parser should never call `new NumberNode()`. Instead, it should call `nodeFactory.createNumber(value)`. This allows us to change node implementations (e.g., switching from `Integer` to `BigDecimal`) without touching the parser logic.

--

### Phase 3: Global State & The Singleton Pattern

Every interpreter needs an **Environment** (a symbol table) to store variables defined during execution (like `x` in `(define x 10)`).

* **The Task:** Implement the `GlobalEnvironment` using the **Singleton Pattern**.
* **The Requirement:** Ensure that only one instance of the global scope exists throughout the application’s lifecycle, providing a global point of access for your `EvaluationVisitor`.

--

### Technical Requirements

| Component | Design Pattern | Purpose |
| --- | --- | --- |
| **AST Traversal** | **Visitor** | Separates the operation (evaluation) from the object structure (nodes). |
| **Node Instantiation** | **Factory** | Centralizes logic for creating different types of AST nodes. |
| **Symbol Table** | **Singleton** | Ensures a single, consistent global scope for variable storage. |

--

### Deliverables

1. **Source Code:** A clean, documented Java project.
2. **Test Suite:** Demonstrate your interpreter handling nested expressions like `(+ 1 (* 2 3))`.
3. **Pattern Justification:** A brief README explaining why the Visitor pattern is preferable here over simple recursion inside the classes.

--

## [Helpful reading](https://gjdanis.github.io/2015/09/14/writing-a-simple-interpreter-cs/)