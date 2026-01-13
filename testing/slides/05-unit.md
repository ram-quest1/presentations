
## Unit Testing a LISP Interpreter

Building confidence in our interpreter, one test at a time.

--

## What We're Building

A basic LISP interpreter in Java that supports:

- ✅ **Arithmetic operations** → `(+ 1 2)`, `(* 3 4)`
- ⏳ If conditions → `(if (> x 0) "positive" "negative")`
- ⏳ Variable definitions → `(define x 10)`

*Today: Testing arithmetic operations*

--

## LISP Syntax Refresher

In LISP, expressions use **prefix notation**:

| LISP Expression | Java Equivalent | Result |
|-----------------|-----------------|--------|
| `(+ 1 2)` | `1 + 2` | `3` |
| `(- 10 4)` | `10 - 4` | `6` |
| `(* 3 5)` | `3 * 5` | `15` |
| `(/ 20 4)` | `20 / 4` | `5` |

--

## Inputs
You come up with different inputs

--

## Nested Expressions

LISP allows nesting:

| LISP Expression | Java Equivalent | Result |
|-----------------|-----------------|--------|
| `(+ 1 (* 2 3))` | `1 + (2 * 3)` | `7` |
| `(* (+ 1 2) (- 5 3))` | `(1 + 2) * (5 - 3)` | `6` |

--

## Our Interpreter Structure

```text
┌─────────────┐     ┌─────────┐     ┌───────────┐
│   Input     │ --> │  Parser │ --> │ Evaluator │ --> Result
│ "(+ 1 2)"   │     │         │     │           │
└─────────────┘     └─────────┘     └───────────┘
```

We'll unit test the **Evaluator** for arithmetic.

--

## The Interpreter Class

```java
public class LispInterpreter {
    
    public Object evaluate(String expression) {
        Object parsed = parse(expression);
        return eval(parsed);
    }
    
    // Parses LISP expression into a List structure
    private Object parse(String expr) { /* ... */ }
    
    // Evaluates the parsed expression
    private Object eval(Object expr) { /* ... */ }
}
```

--

## Why Unit Test Arithmetic First?

- 🧱 Arithmetic is the **foundation** of our interpreter
- 🔢 Easy to verify — clear inputs and outputs
- 🐛 Catches parsing and evaluation bugs early
- 📐 Tests both simple and nested expressions

--

## Setting Up JUnit 5

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

class LispInterpreterTest {
    
    private LispInterpreter interpreter;
    
    @BeforeEach
    void setUp() {
        interpreter = new LispInterpreter();
    }
}
```

--

## Test Strategy

We'll test arithmetic operations in **layers**:

1. Simple two-operand expressions
2. Multiple operands
3. Nested expressions
4. Edge cases and errors

--

## Testing Addition

```java
@Test
@DisplayName("Addition: (+ 1 2) should return 3")
void testSimpleAddition() {
    Object result = interpreter.evaluate("(+ 1 2)");
    assertEquals(3, result);
}

@Test
@DisplayName("Addition: (+ 1 2 3 4) should return 10")
void testMultipleAddition() {
    Object result = interpreter.evaluate("(+ 1 2 3 4)");
    assertEquals(10, result);
}
```

--

## Testing Subtraction

```java
@Test
@DisplayName("Subtraction: (- 10 3) should return 7")
void testSimpleSubtraction() {
    Object result = interpreter.evaluate("(- 10 3)");
    assertEquals(7, result);
}

@Test
@DisplayName("Subtraction: (- 20 5 3 2) should return 10")
void testChainedSubtraction() {
    // 20 - 5 - 3 - 2 = 10
    Object result = interpreter.evaluate("(- 20 5 3 2)");
    assertEquals(10, result);
}
```

--

## Testing Multiplication

```java
@Test
@DisplayName("Multiplication: (* 4 5) should return 20")
void testSimpleMultiplication() {
    Object result = interpreter.evaluate("(* 4 5)");
    assertEquals(20, result);
}

@Test
@DisplayName("Multiplication: (* 2 3 4) should return 24")
void testMultipleMultiplication() {
    Object result = interpreter.evaluate("(* 2 3 4)");
    assertEquals(24, result);
}
```

--

## Testing Division

```java
@Test
@DisplayName("Division: (/ 20 4) should return 5")
void testSimpleDivision() {
    Object result = interpreter.evaluate("(/ 20 4)");
    assertEquals(5, result);
}

@Test
@DisplayName("Division: (/ 100 5 2) should return 10")
void testChainedDivision() {
    // 100 / 5 / 2 = 10
    Object result = interpreter.evaluate("(/ 100 5 2)");
    assertEquals(10, result);
}
```

--

## Testing Nested Expressions

```java
@Test
@DisplayName("Nested: (+ 1 (* 2 3)) should return 7")
void testNestedMultiplicationInAddition() {
    // 1 + (2 * 3) = 1 + 6 = 7
    Object result = interpreter.evaluate("(+ 1 (* 2 3))");
    assertEquals(7, result);
}

@Test
@DisplayName("Nested: (* (+ 1 2) (- 5 3)) should return 6")
void testComplexNesting() {
    // (1 + 2) * (5 - 3) = 3 * 2 = 6
    Object result = interpreter.evaluate("(* (+ 1 2) (- 5 3))");
    assertEquals(6, result);
}
```

--

## Deeply Nested Expressions

```java
@Test
@DisplayName("Deep nesting: (+ (* 2 3) (- 10 (/ 8 2)))")
void testDeeplyNestedExpression() {
    // (+ (* 2 3) (- 10 (/ 8 2)))
    // = (+ 6 (- 10 4))
    // = (+ 6 6)
    // = 12
    Object result = interpreter.evaluate(
        "(+ (* 2 3) (- 10 (/ 8 2)))"
    );
    assertEquals(12, result);
}
```

--

## Testing Edge Cases — Zero

```java
@Test
@DisplayName("Addition with zero: (+ 5 0) should return 5")
void testAdditionWithZero() {
    Object result = interpreter.evaluate("(+ 5 0)");
    assertEquals(5, result);
}

@Test
@DisplayName("Multiplication by zero: (* 100 0) should return 0")
void testMultiplicationByZero() {
    Object result = interpreter.evaluate("(* 100 0)");
    assertEquals(0, result);
}
```

--

## Testing Edge Cases — Negative Numbers

```java
@Test
@DisplayName("Negative result: (- 5 10) should return -5")
void testNegativeResult() {
    Object result = interpreter.evaluate("(- 5 10)");
    assertEquals(-5, result);
}

@Test
@DisplayName("Negative operand: (+ -5 10) should return 5")
void testNegativeOperand() {
    Object result = interpreter.evaluate("(+ -5 10)");
    assertEquals(5, result);
}
```

--

## Testing Error Handling

```java
@Test
@DisplayName("Division by zero should throw exception")
void testDivisionByZero() {
    assertThrows(ArithmeticException.class, () -> {
        interpreter.evaluate("(/ 10 0)");
    });
}

@Test
@DisplayName("Unknown operator should throw exception")
void testUnknownOperator() {
    assertThrows(IllegalArgumentException.class, () -> {
        interpreter.evaluate("(% 10 3)");
    });
}
```

--

## Testing Floating Point

```java
@Test
@DisplayName("Float division: (/ 7 2) should return 3.5")
void testFloatDivision() {
    Object result = interpreter.evaluate("(/ 7.0 2)");
    assertEquals(3.5, (Double) result, 0.001);
}

@Test
@DisplayName("Mixed arithmetic: (+ 1.5 2.5) should return 4.0")
void testFloatAddition() {
    Object result = interpreter.evaluate("(+ 1.5 2.5)");
    assertEquals(4.0, (Double) result, 0.001);
}
```

--

## Parameterized Tests

```java
@ParameterizedTest
@CsvSource({
    "(+ 1 2),       3",
    "(+ 10 20),     30",
    "(+ 0 0),       0",
    "(+ -5 5),      0",
    "(+ 100 -50),   50"
})
@DisplayName("Addition parameterized tests")
void testAdditionParameterized(String expr, int expected) {
    Object result = interpreter.evaluate(expr);
    assertEquals(expected, result);
}
```

--

## Parameterized Tests — All Operations

```java
@ParameterizedTest
@CsvSource({
    "(- 10 3),      7",
    "(* 4 5),       20",
    "(/ 20 4),      5",
    "(+ 1 (* 2 3)), 7"
})
@DisplayName("Mixed arithmetic parameterized tests")
void testArithmeticParameterized(String expr, int expected) {
    Object result = interpreter.evaluate(expr);
    assertEquals(expected, result);
}
```

--

## Complete Test Class Structure

```java
class LispInterpreterTest {
    
    private LispInterpreter interpreter;
    
    @BeforeEach
    void setUp() { /* ... */ }
    
    // Addition tests
    @Nested
    class AdditionTests { /* ... */ }
    
    // Subtraction tests
    @Nested
    class SubtractionTests { /* ... */ }
    
    // Multiplication tests
    @Nested  
    class MultiplicationTests { /* ... */ }
    
    // Division tests
    @Nested
    class DivisionTests { /* ... */ }
    
    // Nested expression tests
    @Nested
    class NestedExpressionTests { /* ... */ }
}
```

--

## Running Tests

```text
├── LispInterpreterTest
│   ├── AdditionTests
│   │   ├── ✓ testSimpleAddition
│   │   ├── ✓ testMultipleAddition
│   │   └── ✓ testAdditionWithZero
│   ├── SubtractionTests
│   │   ├── ✓ testSimpleSubtraction
│   │   └── ✓ testNegativeResult
│   ├── MultiplicationTests
│   │   ├── ✓ testSimpleMultiplication
│   │   └── ✓ testMultiplicationByZero
│   └── DivisionTests
│       ├── ✓ testSimpleDivision
│       └── ✓ testDivisionByZero

Tests: 9 passed, 0 failed
```

--

## Test Coverage Goals

| Component | What to Test | Coverage Target |
|-----------|--------------|-----------------|
| `+` operator | Simple, multiple, nested | 100% |
| `-` operator | Simple, chained, negative | 100% |
| `*` operator | Simple, multiple, by zero | 100% |
| `/` operator | Simple, chained, by zero | 100% |
| Nesting | 2-level, deep nesting | 100% |
| Errors | Invalid input, div by zero | 100% |

--

## Key Takeaways

- 🧪 **Start simple** — test basic operations first
- 📊 **Use parameterized tests** — reduce code duplication
- 🗂️ **Organize with @Nested** — group related tests
- ⚠️ **Test edge cases** — zero, negatives, errors
- 🔄 **Test incrementally** — simple → nested → complex

--

## What's Next?

Now that arithmetic is tested, we can confidently add:

- `if` conditions → `(if (> x 0) "yes" "no")`
- Variable definitions → `(define x 10)`
- Functions → `(define square (lambda (x) (* x x)))`

Each built on our **tested foundation**.
