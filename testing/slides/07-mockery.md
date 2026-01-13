
## Mocking in Unit & Integration Tests

Using Mockito with Real-World Examples

--

## What is Mocking (Unit Test)?

**Mocking** = Creating fake versions of dependencies so you can test your code in isolation.

```text
Without Mocking:
┌──────────┐      ┌──────────────┐      ┌──────────────┐
│ Your Code │ ──> │ Real Database │ ──> │ Real 3rd Party│
└──────────┘      └──────────────┘      └──────────────┘
                        ❌ Slow, Unreliable, Costly

With Mocking:
┌──────────┐      ┌──────────────┐
│ Your Code │ ──> │  Fake (Mock)  │
└──────────┘      └──────────────┘
                        ✅ Fast, Reliable, Free
```

--

## What is Mocking (E2E Test)?

**Mocking** = Creating fake versions of dependencies so you can test your code in isolation.

```text
Without Mocking:
┌──────────┐      ┌──────────────┐
│ Your Code │ ──> │Real 3rd Party│
└──────────┘      └──────────────┘
                        ❌ Slow, Unreliable, Costly

With Mocking:
┌──────────┐      ┌──────────────┐
│ Your Code │ ──> │  Fake (Mock) │
└──────────┘      └──────────────┘
                        ✅ Fast, Reliable, Free
```

--

## Why Mock?

| Problem | Solution with Mocking |
|---------|----------------------|
| 3rd party API costs money | Mock returns free fake data |
| Database is slow | Mock returns instant data |
| External service is down | Mock is always available |
| Can't simulate errors easily | Mock can return any error |
| Tests affect real data | Mock affects nothing |
| CI/CD pipelines | Self contained environment |

--

## Our Example: Loan Underwriting System

```text
┌─────────────────────────────────────────────────────────┐
│              LOAN UNDERWRITING SERVICE                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   Customer applies for loan                             │
│                    │                                    │
│                    ▼                                    │
│   ┌────────────────────────────────┐                   │
│   │    UnderwritingService         │                   │
│   │    - evaluateLoan()            │                   │
│   └───────────┬────────────────────┘                   │
│               │                                         │
│       ┌───────┼───────┬───────────┐                    │
│       ▼       ▼       ▼           ▼                    │
│   ┌───────┐┌───────┐┌───────┐┌────────┐               │
│   │Credit ││ Bank  ││Salary ││ Fraud  │               │
│   │Bureau ││  API  ││  API  ││  API   │               │
│   └───────┘└───────┘└───────┘└────────┘               │
│      $$$      $$$      $$$       $$$    ← Cost Money!  │
└─────────────────────────────────────────────────────────┘
```

--

## The Interfaces We'll Mock

```java
// Credit Bureau - Returns credit score
public interface CreditBureauClient {
    CreditReport getCreditReport(String oderId, String oderId);
}

// Bank - Returns account balance & history
public interface BankClient {
    BankStatement getBankStatement(String oderId, String oderId);
}

// Salary - Returns income verification
public interface SalaryVerificationClient {
    SalaryDetails verifySalary(String oderId, String oderId);
}

// Fraud - Returns fraud risk score
public interface FraudDetectionClient {
    FraudReport checkFraud(String oderId);
}
```

--

## The Service Under Test

```java
public class UnderwritingService {
    
    private final CreditBureauClient creditClient;
    private final BankClient bankClient;
    private final SalaryVerificationClient salaryClient;
    private final FraudDetectionClient fraudClient;
    
    // Constructor injection (important for testing!)
    public UnderwritingService(
            CreditBureauClient creditClient,
            BankClient bankClient,
            SalaryVerificationClient salaryClient,
            FraudDetectionClient fraudClient) {
        this.creditClient = creditClient;
        this.bankClient = bankClient;
        this.salaryClient = salaryClient;
        this.fraudClient = fraudClient;
    }
    
    public LoanDecision evaluateLoan(LoanApplication app) {
        // Business logic that uses all these clients
    }
}
```

--

## Mockito Setup

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class UnderwritingServiceTest {

    @Mock
    private CreditBureauClient creditClient;
    @Mock
    private BankClient bankClient;
    @Mock
    private SalaryVerificationClient salaryClient;
    @Mock
    private FraudDetectionClient fraudClient;
    
    private UnderwritingService underwritingService;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        underwritingService = new UnderwritingService(
            creditClient, bankClient, salaryClient, fraudClient
        );
    }
}
```

--

## Basic Mocking: Happy Path

```java
@Test
void approveLoan_whenAllChecksPass() {
    // Arrange - Setup mock responses
    when(creditClient.getCreditReport("SSN123", "John"))
        .thenReturn(new CreditReport(750, "GOOD"));
    
    when(bankClient.getBankStatement("ACC456", "John"))
        .thenReturn(new BankStatement(50000, 12));
    
    when(salaryClient.verifySalary("EMP789", "John"))
        .thenReturn(new SalaryDetails(80000, true));
    
    when(fraudClient.checkFraud("John"))
        .thenReturn(new FraudReport(false, 0.1));
    
    // Act
    LoanApplication app = new LoanApplication("John", 10000);
    LoanDecision decision = underwritingService.evaluateLoan(app);
    
    // Assert
    assertEquals("APPROVED", decision.getStatus());
}
```

--

## Mocking: Low Credit Score

```java
@Test
void rejectLoan_whenCreditScoreTooLow() {
    // Mock a poor credit score
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenReturn(new CreditReport(450, "POOR"));
    
    // Other mocks return good values
    when(bankClient.getBankStatement(anyString(), anyString()))
        .thenReturn(new BankStatement(100000, 24));
    
    when(salaryClient.verifySalary(anyString(), anyString()))
        .thenReturn(new SalaryDetails(120000, true));
    
    when(fraudClient.checkFraud(anyString()))
        .thenReturn(new FraudReport(false, 0.05));
    
    // Act
    LoanDecision decision = underwritingService.evaluateLoan(
        new LoanApplication("John", 10000)
    );
    
    // Assert
    assertEquals("REJECTED", decision.getStatus());
    assertEquals("LOW_CREDIT_SCORE", decision.getReason());
}
```

--

## Mocking: API Failure

```java
@Test
void handleCreditBureauTimeout() {
    // Mock an API timeout/failure
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenThrow(new ServiceUnavailableException("Credit Bureau down"));
    
    // Act
    LoanDecision decision = underwritingService.evaluateLoan(
        new LoanApplication("John", 10000)
    );
    
    // Assert - System should handle gracefully
    assertEquals("PENDING_REVIEW", decision.getStatus());
    assertEquals("CREDIT_CHECK_FAILED", decision.getReason());
}
```

--

## Mocking: Fraud Detected

```java
@Test
void rejectLoan_whenFraudDetected() {
    // Good credit and income
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenReturn(new CreditReport(800, "EXCELLENT"));
    
    when(salaryClient.verifySalary(anyString(), anyString()))
        .thenReturn(new SalaryDetails(200000, true));
    
    // But fraud detected!
    when(fraudClient.checkFraud(anyString()))
        .thenReturn(new FraudReport(true, 0.95));
    
    // Act
    LoanDecision decision = underwritingService.evaluateLoan(
        new LoanApplication("John", 10000)
    );
    
    // Assert
    assertEquals("REJECTED", decision.getStatus());
    assertEquals("FRAUD_SUSPECTED", decision.getReason());
}
```

--

## Verify: Was the Mock Called?

```java
@Test
void shouldCallAllServicesForEvaluation() {
    // Setup mocks
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenReturn(new CreditReport(700, "GOOD"));
    when(bankClient.getBankStatement(anyString(), anyString()))
        .thenReturn(new BankStatement(30000, 6));
    when(salaryClient.verifySalary(anyString(), anyString()))
        .thenReturn(new SalaryDetails(60000, true));
    when(fraudClient.checkFraud(anyString()))
        .thenReturn(new FraudReport(false, 0.1));
    
    // Act
    underwritingService.evaluateLoan(new LoanApplication("John", 5000));
    
    // Verify all services were called
    verify(creditClient).getCreditReport(anyString(), anyString());
    verify(bankClient).getBankStatement(anyString(), anyString());
    verify(salaryClient).verifySalary(anyString(), anyString());
    verify(fraudClient).checkFraud("John");
}
```

--

## Verify: Call Count

```java
@Test
void shouldNotCallBankIfCreditFails() {
    // Credit check fails
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenReturn(new CreditReport(300, "VERY_POOR"));
    
    // Act
    underwritingService.evaluateLoan(new LoanApplication("John", 5000));
    
    // Verify credit was called
    verify(creditClient, times(1)).getCreditReport(anyString(), anyString());
    
    // Verify bank was NEVER called (short-circuit on bad credit)
    verify(bankClient, never()).getBankStatement(anyString(), anyString());
}
```

--

## Argument Matchers

```java
@Test
void testWithArgumentMatchers() {
    // any() - matches any value
    when(creditClient.getCreditReport(any(), any()))
        .thenReturn(new CreditReport(700, "GOOD"));
    
    // anyString() - matches any string
    when(bankClient.getBankStatement(anyString(), anyString()))
        .thenReturn(new BankStatement(50000, 12));
    
    // eq() - matches exact value
    when(salaryClient.verifySalary(eq("EMP123"), anyString()))
        .thenReturn(new SalaryDetails(75000, true));
    
    // Custom matcher with argThat()
    when(fraudClient.checkFraud(argThat(name -> name.length() > 2)))
        .thenReturn(new FraudReport(false, 0.1));
}
```

--

## Argument Captor

```java
@Test
void captureArgumentsPassedToMock() {
    // Setup
    ArgumentCaptor<String> ssnCaptor = ArgumentCaptor.forClass(String.class);
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenReturn(new CreditReport(700, "GOOD"));
    
    // Act
    LoanApplication app = new LoanApplication("John", 5000);
    app.setSsn("123-45-6789");
    underwritingService.evaluateLoan(app);
    
    // Capture and verify the SSN passed to credit bureau
    verify(creditClient).getCreditReport(ssnCaptor.capture(), anyString());
    assertEquals("123-45-6789", ssnCaptor.getValue());
}
```

--

## Multiple Return Values

```java
@Test
void handleRetryLogic() {
    // First call fails, second succeeds
    when(creditClient.getCreditReport(anyString(), anyString()))
        .thenThrow(new ServiceUnavailableException("Timeout"))
        .thenReturn(new CreditReport(720, "GOOD"));
    
    // Act - service should retry
    LoanDecision decision = underwritingService.evaluateLoan(
        new LoanApplication("John", 5000)
    );
    
    // Verify it was called twice (retry worked)
    verify(creditClient, times(2)).getCreditReport(anyString(), anyString());
    assertEquals("APPROVED", decision.getStatus());
}
```

--

## Mocking in Integration Tests

```java
@SpringBootTest
class UnderwritingIntegrationTest {

    @Autowired
    private UnderwritingService underwritingService;
    
    @MockBean  // Replaces real bean with mock
    private CreditBureauClient creditClient;
    
    @MockBean
    private FraudDetectionClient fraudClient;
    
    // Real beans - test actual integration
    @Autowired
    private BankClient bankClient;  // Uses WireMock server
    
    @Test
    void integrationTest_withPartialMocking() {
        // Mock external paid services
        when(creditClient.getCreditReport(anyString(), anyString()))
            .thenReturn(new CreditReport(700, "GOOD"));
        
        // Let bank client hit WireMock for realistic testing
        // ... test with mix of mocks and real calls
    }
}
```

--

## WireMock for HTTP Mocking

```java
@SpringBootTest
@AutoConfigureWireMock(port = 8089)
class UnderwritingWireMockTest {

    @Test
    void mockExternalHttpCall() {
        // Setup WireMock stub for Credit Bureau API
        stubFor(get(urlPathEqualTo("/api/credit/report"))
            .withQueryParam("ssn", equalTo("123-45-6789"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {
                        "score": 750,
                        "rating": "GOOD"
                    }
                    """)));
        
        // Now your real HTTP client will hit WireMock
        // instead of the actual Credit Bureau
    }
}
```

--

## When TO Mock ✅

| Scenario | Why Mock? |
|----------|-----------|
| **External APIs** | Cost money, rate limited, slow |
| **Databases** | Slow, need setup, state issues |
| **Email/SMS services** | Don't want real messages sent |
| **Payment gateways** | Never test with real money! |
| **Time-dependent code** | Mock `Clock` to control time |
| **Random generators** | Make tests deterministic |
| **File system** | Avoid actual file I/O |
| **Network calls** | Unreliable, slow |

--

## When NOT to Mock ❌

| Scenario | Why Not Mock? |
|----------|---------------|
| **The class under test** | You're testing it, not faking it! |
| **Simple value objects** | Just use real ones (DTOs, POJOs) |
| **Collections** | Use real `List`, `Map`, etc. |
| **Your own utility classes** | Test them separately |
| **Everything** | Over-mocking makes tests brittle |

--

## The Mockist vs Classicist Debate

```text
OVER-MOCKING (Bad)                 BALANCED (Good)
─────────────────────              ─────────────────────
Mock EVERYTHING                    Mock BOUNDARIES only

┌─────────┐                        ┌─────────┐
│ Service │                        │ Service │
└────┬────┘                        └────┬────┘
     │ mock                             │ real
┌────▼────┐                        ┌────▼────┐
│ Helper  │                        │ Helper  │
└────┬────┘                        └────┬────┘
     │ mock                             │ real
┌────▼────┐                        ┌────▼────────┐
│  Utils  │                        │ External API │ ← mock only this!
└─────────┘                        └──────────────┘
```

--

## Mock Only at Boundaries

```text
┌──────────────────────────────────────────────────────────┐
│                    YOUR APPLICATION                      │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                  │    │
│  │   Controller → Service → Repository → Helper    │    │
│  │                                                  │    │
│  │        ↑ Test all of this with REAL objects     │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│                          │ MOCK HERE (boundaries)        │
│                          ▼                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Database │  │ HTTP API │  │  Email   │  │  Queue   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
└──────────────────────────────────────────────────────────┘
```

--

## Don't Mock What You Don't Own?

```java
// ❌ BAD - Mocking 3rd party library internals
@Mock
private ObjectMapper objectMapper;  // Jackson's class

@Test
void badTest() {
    when(objectMapper.readValue(anyString(), any()))
        .thenReturn(new MyObject());  // Fragile!
}

// ✅ GOOD - Wrap 3rd party, mock your wrapper
public interface JsonParser {
    <T> T parse(String json, Class<T> type);
}

@Mock
private JsonParser jsonParser;  // Your interface

@Test
void goodTest() {
    when(jsonParser.parse(anyString(), any()))
        .thenReturn(new MyObject());  // Stable!
}
```

--

## Quick Reference: Mockito Methods

| Method | Purpose |
|--------|---------|
| `mock(Class)` | Create a mock |
| `@Mock` | Annotation to create mock |
| `when().thenReturn()` | Define return value |
| `when().thenThrow()` | Define exception |
| `verify()` | Check mock was called |
| `verify(times(n))` | Check call count |
| `verify(never())` | Check NOT called |
| `any()`, `anyString()` | Match any argument |
| `eq()` | Match exact value |
| `ArgumentCaptor` | Capture passed arguments |

--

## Quick Reference: Annotations

```java
// JUnit 5 + Mockito
@ExtendWith(MockitoExtension.class)
class MyTest {

    @Mock           // Creates a mock
    private MyDependency dep;
    
    @Spy            // Partial mock (real methods by default)
    private MyHelper helper;
    
    @InjectMocks    // Injects mocks into this object
    private MyService service;
    
    @Captor         // Creates ArgumentCaptor
    private ArgumentCaptor<String> captor;
}

// Spring Boot
@MockBean          // Mock in Spring context
@SpyBean           // Spy in Spring context
```

--

## Common Mistakes

```java
// ❌ MISTAKE 1: Mocking the class under test
@Mock
private UnderwritingService service;  // NO! Test this, don't mock it

// ❌ MISTAKE 2: Not using lenient() for unused stubs
when(mock.method()).thenReturn(value);  // Fails if not called
lenient().when(mock.method()).thenReturn(value);  // OK if not called

// ❌ MISTAKE 3: Mixing matchers with real values
when(mock.method(anyString(), "literal"))  // WRONG!
when(mock.method(anyString(), eq("literal")))  // CORRECT!

// ❌ MISTAKE 4: Forgetting to initialize mocks
// Always use @ExtendWith(MockitoExtension.class) 
// or MockitoAnnotations.openMocks(this)
```

--

## Summary: Mocking Decision Tree

```text
                    Should I Mock This?
                           │
                           ▼
                 Is it an external service?
                  (API, DB, Email, Queue)
                    /              \
                  YES               NO
                   │                 │
                   ▼                 ▼
              ✅ MOCK IT       Is it your own code?
                                /          \
                              YES           NO
                               │             │
                               ▼             ▼
                     Is it simple?     ✅ MOCK IT
                     (DTO, POJO)       (3rd party lib)
                      /       \
                    YES        NO
                     │          │
                     ▼          ▼
              ❌ USE REAL   Consider mocking
                           if complex setup
```

--

## Key Takeaways

- 🎯 **Mock external dependencies** — APIs, databases, services
- 🚫 **Don't mock everything** — Over-mocking = brittle tests
- 🔌 **Mock at boundaries** — Where your code meets the outside world
- ✅ **Use real objects** — For DTOs, collections, utilities
- 🧪 **Verify behavior** — Ensure mocks are called correctly
- ⚡ **Fast & reliable** — Mocks make tests predictable

--

## Final Thought

> "Mock things that are **slow**, **expensive**, **unreliable**, or **have side effects**. Use real objects for everything else."

--

## What Design Pattern is used for mocking?

# **Proxy** <!-- .element: class="fragment fade-up" -->