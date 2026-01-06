## Visitor Design Pattern

**Goal:** Add new operations to a class hierarchy without modifying the classes.

> **When to use:** AST traversal, compilers, document processing, or when you need many unrelated operations on a structure.

--

### The Problem

```java
// Arithmetic expression AST: (3 + 5) * 2
//
//        Multiply
//        /      \
//      Add       2
//     /   \
//    3     5

// We need multiple operations on this tree:
// - Evaluate: compute the result → 16
// - Print: show as string → "((3 + 5) * 2)"
// - Compile: generate bytecode
// - Optimize: simplify expressions

// Without Visitor - add method to EVERY node class for EACH operation
class AddNode {
    int evaluate() { ... }
    String print() { ... }
    byte[] compile() { ... }    // Every new operation = edit ALL classes!
    Node optimize() { ... }
}
```

**Problem:** Adding new operations means modifying every node class.

--

### Solution: Visitor Pattern

```text
Instead of:  Node.evaluate(), Node.print(), Node.compile()
We do:       Evaluator.visit(Node), Printer.visit(Node), Compiler.visit(Node)

┌─────────────────┐         ┌─────────────────┐
│   Expression    │         │     Visitor     │
│   (AST Nodes)   │ ◄─────► │  (Operations)   │
└─────────────────┘         └─────────────────┘
│ NumberExpr      │         │ visit(Number)   │
│ AddExpr         │         │ visit(Add)      │
│ MultiplyExpr    │         │ visit(Multiply) │
└─────────────────┘         └─────────────────┘

Node says: "I accept visitors"
Visitor says: "I know what to do with each node type"
```

> New operation = new Visitor class. No changes to AST nodes!

--

### Step 1: The Expression Interface

```java
// All AST nodes implement this
public interface Expression {
    
    // Accept a visitor - let it "visit" this node
    <T> T accept(ExpressionVisitor<T> visitor);
}
```

--

### Step 2: AST Node Classes

```java
public class NumberExpr implements Expression {
    
    public final int value;
    
    public NumberExpr(int value) {
        this.value = value;
    }
    
    public <T> T accept(ExpressionVisitor<T> visitor) {
        return visitor.visitNumber(this);
    }
}

public class AddExpr implements Expression {
    
    public final Expression left;
    public final Expression right;
    
    public AddExpr(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    public <T> T accept(ExpressionVisitor<T> visitor) {
        return visitor.visitAdd(this);
    }
}

public class MultiplyExpr implements Expression {
    
    public final Expression left;
    public final Expression right;
    
    public MultiplyExpr(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    public <T> T accept(ExpressionVisitor<T> visitor) {
        return visitor.visitMultiply(this);
    }
}
```

--

### Step 3: The Visitor Interface

```java
// One method per node type
public interface ExpressionVisitor<T> {
    
    T visitNumber(NumberExpr expr);
    
    T visitAdd(AddExpr expr);
    
    T visitMultiply(MultiplyExpr expr);
}
```

--

### Step 4: Evaluator Visitor

```java
// Computes the result of the expression
public class Evaluator implements ExpressionVisitor<Integer> {
    
    public Integer visitNumber(NumberExpr expr) {
        return expr.value;
    }
    
    public Integer visitAdd(AddExpr expr) {
        int left = expr.left.accept(this);   // Evaluate left subtree
        int right = expr.right.accept(this); // Evaluate right subtree
        return left + right;
    }
    
    public Integer visitMultiply(MultiplyExpr expr) {
        int left = expr.left.accept(this);
        int right = expr.right.accept(this);
        return left * right;
    }
}
```

--

### Step 5: Printer Visitor

```java
// Converts expression to readable string
public class Printer implements ExpressionVisitor<String> {
    
    public String visitNumber(NumberExpr expr) {
        return String.valueOf(expr.value);
    }
    
    public String visitAdd(AddExpr expr) {
        String left = expr.left.accept(this);
        String right = expr.right.accept(this);
        return "(" + left + " + " + right + ")";
    }
    
    public String visitMultiply(MultiplyExpr expr) {
        String left = expr.left.accept(this);
        String right = expr.right.accept(this);
        return "(" + left + " * " + right + ")";
    }
}
```

--

### Using the Visitors

```java
public class Application {
    
    public static void main(String[] args) {
        
        // Build AST for: (3 + 5) * 2
        Expression expr = new MultiplyExpr(
            new AddExpr(
                new NumberExpr(3),
                new NumberExpr(5)
            ),
            new NumberExpr(2)
        );
        
        // Use different visitors on the SAME tree
        Evaluator evaluator = new Evaluator();
        Printer printer = new Printer();
        
        int result = expr.accept(evaluator);
        String text = expr.accept(printer);
        
        System.out.println("Expression: " + text);
        System.out.println("Result: " + result);
    }
}
```

--

### Output

```text
Expression: ((3 + 5) * 2)
Result: 16
```

--

### How the Visitor Traverses

```text
expr.accept(evaluator)
        │
        ▼
   MultiplyExpr
   visitor.visitMultiply(this)
        │
   ┌────┴────┐
   ▼         ▼
left.accept  right.accept
   │              │
   ▼              ▼
 AddExpr     NumberExpr(2)
   │              │
   │         returns 2
   ▼
┌──┴──┐
▼     ▼
3     5
│     │
└──┬──┘
   8
   │
   └────► 8 * 2 = 16
```

> Each node calls the right `visit` method. Visitor handles the logic.

--

### Adding New Operations

```java
// Need to count operations? Just add a new visitor!
public class OperationCounter implements ExpressionVisitor<Integer> {
    
    public Integer visitNumber(NumberExpr expr) {
        return 0;  // Numbers aren't operations
    }
    
    public Integer visitAdd(AddExpr expr) {
        int left = expr.left.accept(this);
        int right = expr.right.accept(this);
        return 1 + left + right;  // Count this + children
    }
    
    public Integer visitMultiply(MultiplyExpr expr) {
        int left = expr.left.accept(this);
        int right = expr.right.accept(this);
        return 1 + left + right;
    }
}

// Usage
int ops = expr.accept(new OperationCounter());
System.out.println("Operations: " + ops);  // Output: 2
```

> Added new functionality WITHOUT touching AST classes!

--

### Visitor Pattern Structure

```text
┌────────────────────┐       ┌────────────────────────┐
│    Expression      │       │   ExpressionVisitor<T> │
│   <<interface>>    │       │     <<interface>>      │
├────────────────────┤       ├────────────────────────┤
│ accept(visitor): T │       │ visitNumber(): T       │
└─────────┬──────────┘       │ visitAdd(): T          │
          │                  │ visitMultiply(): T     │
    ┌─────┼─────┐            └───────────┬────────────┘
    ▼     ▼     ▼                  ┌─────┴─────┐
 Number  Add  Multiply             ▼           ▼
                              Evaluator    Printer
```

--

### Real-World Examples

```java
// Java Compiler - AST processing
TreeVisitor<Void> typeChecker = new TypeChecker();
TreeVisitor<String> codeGenerator = new CodeGenerator();

// ANTLR Parser Generator
MyLanguageBaseVisitor<Integer> eval = new EvalVisitor();
tree.accept(eval);

// DOM/XML Processing
document.accept(new XMLValidator());
document.accept(new XMLTransformer());
```

> Compilers, interpreters, and parsers heavily use the Visitor pattern.

--

### When to Use Visitor

| Use Visitor When | Avoid Visitor When |
|------------------|-------------------|
| Many operations on stable structure | Structure changes frequently |
| Operations are unrelated | Operations are closely related |
| Adding operations often | Adding new node types often |
| Need double dispatch | Simple single dispatch works |

> **Trade-off:** Easy to add operations, hard to add new node types.

--

### Visitor Pattern Checklist

| Component | Purpose |
|-----------|---------|
| Element interface | Defines `accept(visitor)` method |
| Concrete elements | AST nodes that accept visitors |
| Visitor interface | One `visit` method per element type |
| Concrete visitors | Implement operations (eval, print, etc.) |

**Remember:**

1. **Double dispatch** - right method called based on both node AND visitor type
2. **Open/Closed** - open for new operations, closed for modification
3. **accept() calls visit()** - node passes itself to visitor
4. **Traverse in visitor** - visitor decides how to walk children