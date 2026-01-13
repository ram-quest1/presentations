
## Integration & System Testing

Testing our LISP Interpreter REST API

--

## Architecture Overview

```text
┌─────────────┐      HTTP       ┌─────────────────────────┐
│   Client    │ ──────────────> │    LISP REST API        │
│  (Tests)    │ <────────────── │                         │
└─────────────┘    JSON         │  ┌─────────────────┐    │
                                │  │ LispInterpreter │    │
                                │  └─────────────────┘    │
                                │  ┌─────────────────┐    │
                                │  │   Sessions      │    │
                                │  │  (Environment)  │    │
                                │  └─────────────────┘    │
                                └─────────────────────────┘
```

--

## API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/api/sessions` | Create new session |
| `POST` | `/api/sessions/{id}/evaluate` | Evaluate expression |
| `GET` | `/api/sessions/{id}/variables` | List all variables |
| `GET` | `/api/sessions/{id}/variables/{name}` | Get variable value |
| `DELETE` | `/api/sessions/{id}` | End session |

--

## Request/Response Format

**Request:**
```json
{
    "expression": "(+ 1 2)"
}
```

**Response:**
```json
{
    "success": true,
    "result": 3,
    "type": "INTEGER",
    "executionTimeMs": 5
}
```

--

## Why REST API Changes Testing

| Aspect | Direct Call | REST API |
|--------|-------------|----------|
| Invocation | Method call | HTTP request |
| State | Object instance | Session ID |
| Errors | Exceptions | HTTP status codes |
| Data | Java objects | JSON |
| Latency | Negligible | Network overhead |

--

## Test Setup with RestAssured

```java
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.*;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

class LispApiIntegrationTest {
    
    private String sessionId;
    
    @BeforeAll
    static void setup() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = 8080;
        RestAssured.basePath = "/api";
    }
}
```

--

## Session Management in Tests

```java
@BeforeEach
void createSession() {
    sessionId = given()
        .contentType(ContentType.JSON)
    .when()
        .post("/sessions")
    .then()
        .statusCode(201)
        .extract()
        .path("sessionId");
}

@AfterEach
void cleanupSession() {
    if (sessionId != null) {
        delete("/sessions/" + sessionId);
    }
}
```

--

## Helper Method for Evaluation

```java
private ValidatableResponse evaluate(String expression) {
    return given()
        .contentType(ContentType.JSON)
        .body(Map.of("expression", expression))
    .when()
        .post("/sessions/" + sessionId + "/evaluate")
    .then();
}
```

--

## Testing: Simple Arithmetic

```java
@Test
@DisplayName("POST /evaluate - Addition returns correct result")
void testAddition() {
    evaluate("(+ 1 2)")
        .statusCode(200)
        .body("success", equalTo(true))
        .body("result", equalTo(3))
        .body("type", equalTo("INTEGER"));
}
```

--

## Testing: All Arithmetic Operations

```java
@Test
@DisplayName("POST /evaluate - Subtraction")
void testSubtraction() {
    evaluate("(- 10 3)")
        .statusCode(200)
        .body("result", equalTo(7));
}

@Test
@DisplayName("POST /evaluate - Multiplication")
void testMultiplication() {
    evaluate("(* 4 5)")
        .statusCode(200)
        .body("result", equalTo(20));
}

@Test
@DisplayName("POST /evaluate - Division")
void testDivision() {
    evaluate("(/ 20 4)")
        .statusCode(200)
        .body("result", equalTo(5));
}
```

--

## Testing: Define Variable via API

```java
@Test
@DisplayName("Define variable and retrieve via API")
void testDefineVariable() {
    // Define variable
    evaluate("(define x 42)")
        .statusCode(200)
        .body("success", equalTo(true));
    
    // Retrieve via dedicated endpoint
    given()
    .when()
        .get("/sessions/" + sessionId + "/variables/x")
    .then()
        .statusCode(200)
        .body("name", equalTo("x"))
        .body("value", equalTo(42));
}
```

--

## Testing: Variable in Arithmetic

```java
@Test
@DisplayName("Use defined variable in arithmetic expression")
void testVariableInArithmetic() {
    // Setup
    evaluate("(define x 10)");
    evaluate("(define y 5)");
    
    // Test
    evaluate("(+ x y)")
        .statusCode(200)
        .body("result", equalTo(15));
}
```

--

## Testing: Session State Persistence

```java
@Test
@DisplayName("Variables persist across multiple API calls")
void testStatePersistence() {
    evaluate("(define counter 0)");
    
    evaluate("(define counter (+ counter 1))");
    evaluate("(define counter (+ counter 1))");
    evaluate("(define counter (+ counter 1))");
    
    evaluate("counter")
        .statusCode(200)
        .body("result", equalTo(3));
}
```

--

## Testing: List All Variables

```java
@Test
@DisplayName("GET /variables returns all defined variables")
void testListVariables() {
    evaluate("(define a 10)");
    evaluate("(define b 20)");
    evaluate("(define c 30)");
    
    given()
    .when()
        .get("/sessions/" + sessionId + "/variables")
    .then()
        .statusCode(200)
        .body("variables.size()", equalTo(3))
        .body("variables.a", equalTo(10))
        .body("variables.b", equalTo(20))
        .body("variables.c", equalTo(30));
}
```

--

## Testing: If Conditions

```java
@Test
@DisplayName("If condition with true predicate")
void testIfConditionTrue() {
    evaluate("(if (> 10 5) \"yes\" \"no\")")
        .statusCode(200)
        .body("result", equalTo("yes"))
        .body("type", equalTo("STRING"));
}

@Test
@DisplayName("If condition with false predicate")
void testIfConditionFalse() {
    evaluate("(if (< 10 5) \"yes\" \"no\")")
        .statusCode(200)
        .body("result", equalTo("no"));
}
```

--

## Testing: If with Variables

```java
@Test
@DisplayName("Conditional logic using defined variables")
void testIfWithVariables() {
    evaluate("(define age 25)");
    evaluate("(define votingAge 18)");
    
    evaluate("(if (>= age votingAge) \"can-vote\" \"cannot-vote\")")
        .statusCode(200)
        .body("result", equalTo("can-vote"));
}
```

--

## Testing: Complex Business Logic

```java
@Test
@DisplayName("Order discount calculation via API")
void testOrderDiscountCalculation() {
    evaluate("(define quantity 10)");
    evaluate("(define unitPrice 25)");
    evaluate("(define subtotal (* quantity unitPrice))");
    
    // Apply 15% discount if subtotal > 200
    evaluate(
        "(define finalPrice " +
        "  (if (> subtotal 200) " +
        "    (* subtotal 0.85) " +
        "    subtotal))"
    );
    
    evaluate("finalPrice")
        .statusCode(200)
        .body("result", closeTo(212.5, 0.01));
}
```

--

## Testing: Session Isolation

```java
@Test
@DisplayName("Different sessions have isolated environments")
void testSessionIsolation() {
    // Session 1: Define x = 100
    evaluate("(define x 100)");
    
    // Create Session 2
    String session2Id = given()
        .post("/sessions")
    .then()
        .extract().path("sessionId");
    
    // Session 2: x should not exist
    given()
        .body(Map.of("expression", "x"))
    .when()
        .post("/sessions/" + session2Id + "/evaluate")
    .then()
        .statusCode(400)
        .body("error", containsString("undefined"));
    
    // Cleanup
    delete("/sessions/" + session2Id);
}
```

--

## Error Handling: Invalid Expression

```java
@Test
@DisplayName("400 Bad Request for syntax errors")
void testSyntaxError() {
    evaluate("(+ 1 2")  // Missing closing paren
        .statusCode(400)
        .body("success", equalTo(false))
        .body("error", containsString("parse"))
        .body("errorType", equalTo("SYNTAX_ERROR"));
}
```

--

## Error Handling: Undefined Variable

```java
@Test
@DisplayName("400 Bad Request for undefined variable")
void testUndefinedVariable() {
    evaluate("(+ x 10)")
        .statusCode(400)
        .body("success", equalTo(false))
        .body("error", containsString("undefined"))
        .body("errorType", equalTo("UNDEFINED_VARIABLE"));
}
```

--

## Error Handling: Division by Zero

```java
@Test
@DisplayName("400 Bad Request for division by zero")
void testDivisionByZero() {
    evaluate("(/ 10 0)")
        .statusCode(400)
        .body("success", equalTo(false))
        .body("error", containsString("division by zero"))
        .body("errorType", equalTo("ARITHMETIC_ERROR"));
}
```

--

## Error Handling: Invalid Session

```java
@Test
@DisplayName("404 Not Found for invalid session ID")
void testInvalidSession() {
    given()
        .contentType(ContentType.JSON)
        .body(Map.of("expression", "(+ 1 2)"))
    .when()
        .post("/sessions/invalid-session-id/evaluate")
    .then()
        .statusCode(404)
        .body("error", containsString("session not found"));
}
```

--

## Error Handling: Empty Request

```java
@Test
@DisplayName("400 Bad Request for empty expression")
void testEmptyExpression() {
    given()
        .contentType(ContentType.JSON)
        .body(Map.of("expression", ""))
    .when()
        .post("/sessions/" + sessionId + "/evaluate")
    .then()
        .statusCode(400)
        .body("error", containsString("empty"));
}

@Test
@DisplayName("400 Bad Request for missing expression field")
void testMissingExpression() {
    given()
        .contentType(ContentType.JSON)
        .body("{}")
    .when()
        .post("/sessions/" + sessionId + "/evaluate")
    .then()
        .statusCode(400);
}
```

--

## Testing: Response Headers

```java
@Test
@DisplayName("Response includes proper headers")
void testResponseHeaders() {
    evaluate("(+ 1 2)")
        .statusCode(200)
        .contentType(ContentType.JSON)
        .header("X-Execution-Time", notNullValue())
        .header("X-Session-Id", equalTo(sessionId));
}
```

--

## Testing: Execution Time Tracking

```java
@Test
@DisplayName("Response includes execution time")
void testExecutionTimeTracking() {
    evaluate("(+ 1 2)")
        .statusCode(200)
        .body("executionTimeMs", greaterThanOrEqualTo(0))
        .body("executionTimeMs", lessThan(1000));
}
```

--

## System Test: Multi-Step Workflow

```java
@Test
@DisplayName("E2E: Complete calculation workflow")
void testCompleteWorkflow() {
    // Step 1: Create session (already done in @BeforeEach)
    
    // Step 2: Define base values
    evaluate("(define principal 10000)");
    evaluate("(define rate 0.05)");
    evaluate("(define years 3)");
    
    // Step 3: Calculate compound interest
    // Simple interest: principal * rate * years
    evaluate("(define interest (* (* principal rate) years))");
    
    // Step 4: Calculate final amount
    evaluate("(define total (+ principal interest))");
    
    // Step 5: Verify result
    evaluate("total")
        .statusCode(200)
        .body("result", closeTo(11500.0, 0.01));
    
    // Step 6: Verify all variables via GET
    given()
        .get("/sessions/" + sessionId + "/variables")
    .then()
        .body("variables.size()", equalTo(5));
}
```

--

## System Test: Banking Operations

```java
@Test
@DisplayName("E2E: Banking transaction simulation")
void testBankingOperations() {
    // Initialize account
    evaluate("(define balance 1000)");
    
    // Deposit
    evaluate("(define balance (+ balance 500))");
    evaluate("balance")
        .body("result", equalTo(1500));
    
    // Withdrawal with validation
    evaluate("(define amount 200)");
    evaluate(
        "(if (>= balance amount) " +
        "  (define balance (- balance amount)) " +
        "  \"insufficient-funds\")"
    );
    
    evaluate("balance")
        .body("result", equalTo(1300));
    
    // Failed withdrawal
    evaluate("(define amount 2000)");
    evaluate(
        "(if (>= balance amount) " +
        "  (- balance amount) " +
        "  \"insufficient-funds\")"
    )
        .body("result", equalTo("insufficient-funds"));
}
```

--

## System Test: Grading System

```java
@Test
@DisplayName("E2E: Student grading calculation")
void testGradingSystem() {
    // Input scores
    evaluate("(define midterm 78)");
    evaluate("(define final 85)");
    evaluate("(define assignment 90)");
    
    // Calculate weighted average (30%, 40%, 30%)
    evaluate(
        "(define average (+ (+ (* midterm 0.3) " +
        "                       (* final 0.4)) " +
        "                    (* assignment 0.3)))"
    );
    
    // Determine grade
    evaluate(
        "(define grade " +
        "  (if (>= average 90) \"A\" " +
        "    (if (>= average 80) \"B\" " +
        "      (if (>= average 70) \"C\" \"F\"))))"
    );
    
    evaluate("grade")
        .body("result", equalTo("B"));
}
```

--

## Load Testing Setup

```java
@Test
@DisplayName("API handles concurrent requests")
void testConcurrentRequests() throws Exception {
    int numThreads = 10;
    ExecutorService executor = Executors.newFixedThreadPool(numThreads);
    CountDownLatch latch = new CountDownLatch(numThreads);
    AtomicInteger successCount = new AtomicInteger(0);
    
    for (int i = 0; i < numThreads; i++) {
        final int value = i;
        executor.submit(() -> {
            try {
                evaluate("(+ " + value + " 1)")
                    .statusCode(200);
                successCount.incrementAndGet();
            } finally {
                latch.countDown();
            }
        });
    }
    
    latch.await(30, TimeUnit.SECONDS);
    assertEquals(numThreads, successCount.get());
}
```

--

## Performance Assertions

```java
@Test
@DisplayName("Simple expression evaluates within 100ms")
void testPerformanceSLA() {
    long startTime = System.currentTimeMillis();
    
    evaluate("(+ 1 2)")
        .statusCode(200);
    
    long duration = System.currentTimeMillis() - startTime;
    assertTrue(duration < 100, 
        "Expected < 100ms but took " + duration + "ms");
}

@Test
@DisplayName("Complex expression evaluates within 500ms")
void testComplexPerformance() {
    long startTime = System.currentTimeMillis();
    
    evaluate("(* (+ 1 2) (- 10 (/ 20 4)))")
        .statusCode(200);
    
    long duration = System.currentTimeMillis() - startTime;
    assertTrue(duration < 500);
}
```

--

## Contract Testing

```java
@Test
@DisplayName("Success response follows contract")
void testSuccessResponseContract() {
    evaluate("(+ 1 2)")
        .statusCode(200)
        .body("$", hasKey("success"))
        .body("$", hasKey("result"))
        .body("$", hasKey("type"))
        .body("$", hasKey("executionTimeMs"))
        .body("success", instanceOf(Boolean.class))
        .body("executionTimeMs", instanceOf(Integer.class));
}

@Test
@DisplayName("Error response follows contract")
void testErrorResponseContract() {
    evaluate("(+ x 1)")
        .statusCode(400)
        .body("$", hasKey("success"))
        .body("$", hasKey("error"))
        .body("$", hasKey("errorType"))
        .body("success", equalTo(false));
}
```

--

## Test Organization

```java
class LispApiIntegrationTest {
    
    @Nested
    @DisplayName("Session Management")
    class SessionTests { /* ... */ }
    
    @Nested
    @DisplayName("Arithmetic Operations")
    class ArithmeticTests { /* ... */ }
    
    @Nested
    @DisplayName("Variable Integration")
    class VariableTests { /* ... */ }
    
    @Nested
    @DisplayName("Conditional Logic")
    class ConditionalTests { /* ... */ }
    
    @Nested
    @DisplayName("Error Handling")
    class ErrorTests { /* ... */ }
    
    @Nested
    @DisplayName("End-to-End Scenarios")
    class E2ETests { /* ... */ }
    
    @Nested
    @DisplayName("Performance & Load")
    class PerformanceTests { /* ... */ }
}
```

--

## HTTP Status Code Summary

| Status | Meaning | When Used |
|--------|---------|-----------|
| `200` | OK | Successful evaluation |
| `201` | Created | New session created |
| `400` | Bad Request | Syntax error, undefined var |
| `404` | Not Found | Invalid session ID |
| `500` | Server Error | Unexpected failures |

--

## Test Execution Results

```text
├── LispApiIntegrationTest
│   ├── SessionTests
│   │   ├── ✓ testCreateSession
│   │   ├── ✓ testSessionIsolation
│   │   └── ✓ testDeleteSession
│   ├── ArithmeticTests
│   │   ├── ✓ testAddition
│   │   ├── ✓ testSubtraction
│   │   └── ✓ testNestedArithmetic
│   ├── VariableTests
│   │   ├── ✓ testDefineVariable
│   │   └── ✓ testStatePersistence
│   ├── ErrorTests
│   │   ├── ✓ testSyntaxError
│   │   └── ✓ testUndefinedVariable
│   └── E2ETests
│       ├── ✓ testBankingOperations
│       └── ✓ testGradingSystem

Tests: 18 passed, 0 failed
```

--

## API Testing Pyramid

```text
              /\
             /  \      Contract Tests
            /    \     Schema validation
           /------\
          /        \   E2E Scenarios
         / Banking  \  Multi-step workflows
        /  Grading   \
       /--------------\
      /                \  Integration Tests
     / Session + Vars   \ Variable persistence
    /  Arithmetic + If   \ Component interaction
   /----------------------\
  /                        \ Endpoint Tests
 /  POST /evaluate          \ Individual API calls
/  GET /variables            \ Request/Response
/----------------------------\
```

--

## Key Differences: Direct vs API Testing

| Aspect | Direct Testing | API Testing |
|--------|---------------|-------------|
| **Setup** | `new Interpreter()` | HTTP session creation |
| **Invocation** | `evaluate()` | `POST /evaluate` |
| **State** | Object field | Session ID in URL |
| **Assertions** | `assertEquals()` | `body("result", equalTo())` |
| **Errors** | `assertThrows()` | `statusCode(400)` |
| **Cleanup** | Garbage collection | `DELETE /session` |

--

## Key Takeaways

- 🌐 **REST APIs require session management** for stateful operations
- 📋 **HTTP status codes** replace exceptions for error handling
- 🔒 **Session isolation** is critical for multi-tenant safety
- ⏱️ **Performance testing** matters more with network overhead
- 📜 **Contract testing** ensures API consistency

--

## What's Next?

Advanced API testing topics:

- **Authentication** → API keys, JWT tokens
- **Rate limiting** → Throttling tests
- **Caching** → Response caching validation
- **API versioning** → `/v1/` vs `/v2/` compatibility
- **Documentation** → OpenAPI/Swagger integration


--

## Use of containers like Docker in integration testing 

- Essential to spin up services and dependencies 
- Worth learning just for this even if not used in production
- Used extensively, wrapper by frameworks like [Testcontainers](https://testcontainers.com/)