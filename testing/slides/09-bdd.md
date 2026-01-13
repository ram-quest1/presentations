
## BDD Testing with Cucumber

Writing Tests in Plain English

--

## What is BDD?

**Behavior Driven Development** = Writing tests that describe *behavior* in human-readable language.

```text
Traditional Test:              BDD Test:
─────────────────              ─────────────────
assertEquals(                  Given a customer with credit score 750
  "APPROVED",                  When they apply for a $10,000 loan
  result.getStatus()           Then the loan should be approved
);
```

--

## Why BDD?

| Traditional Tests | BDD Tests |
|-------------------|-----------|
| Written by developers | Written by anyone |
| Technical code | Plain English |
| "What does this test?" | Self-documenting |
| Dev-only readability | Business stakeholder readable |

--

## The BDD Triad

```text
        ┌─────────────────┐
        │   Business      │
        │   Analyst       │
        └────────┬────────┘
                 │
                 │ Writes requirements
                 ▼
        ┌─────────────────┐
        │  Feature File   │  ← Plain English
        │   (Gherkin)     │
        └────────┬────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
┌───────────────┐ ┌───────────────┐
│   Developer   │ │     QA        │
│ (Step Defs)   │ │  (Scenarios)  │
└───────────────┘ └───────────────┘
```

--

## Cucumber Overview

```text
┌──────────────────┐      ┌──────────────────┐
│   Feature File   │      │ Step Definitions │
│    (.feature)    │ ───> │     (Java)       │
│                  │      │                  │
│  Plain English   │      │  Actual Code     │
└──────────────────┘      └──────────────────┘
         │                         │
         └───────────┬─────────────┘
                     ▼
              ┌─────────────┐
              │ Test Runner │
              │  (JUnit)    │
              └─────────────┘
```

--

## Gherkin: The Language of BDD

Gherkin uses simple keywords:

| Keyword | Purpose |
|---------|---------|
| **Feature** | Describes the feature being tested |
| **Scenario** | A specific test case |
| **Given** | Setup / preconditions |
| **When** | Action being tested |
| **Then** | Expected outcome |
| **And** | Additional steps |
| **But** | Negative additional steps |

--

## Feature File Structure

```gherkin
Feature: Short description of the feature
  
  As a [role]
  I want [capability]
  So that [benefit]

  Scenario: Name of test case
    Given some precondition
    And another precondition
    When an action is taken
    Then expected result
    And another expected result
```

--

## Loan Underwriting Feature File

```gherkin
Feature: Loan Application Evaluation
  
  As a loan officer
  I want the system to automatically evaluate loan applications
  So that customers get quick decisions

  Scenario: Approve loan for customer with good credit
    Given a customer named "John"
    And John has a credit score of 750
    And John has a monthly income of $8,000
    And John has no fraud history
    When John applies for a loan of $10,000
    Then the loan should be "APPROVED"
```

--

## More Scenarios: Rejection Cases

```gherkin
  Scenario: Reject loan for low credit score
    Given a customer named "Jane"
    And Jane has a credit score of 450
    And Jane has a monthly income of $5,000
    When Jane applies for a loan of $15,000
    Then the loan should be "REJECTED"
    And the rejection reason should be "LOW_CREDIT_SCORE"

  Scenario: Reject loan when fraud is detected
    Given a customer named "Bob"
    And Bob has a credit score of 800
    And Bob has been flagged for fraud
    When Bob applies for a loan of $5,000
    Then the loan should be "REJECTED"
    And the rejection reason should be "FRAUD_SUSPECTED"
```

--

## Scenario: Insufficient Income

```gherkin
  Scenario: Reject loan when income is insufficient
    Given a customer named "Alice"
    And Alice has a credit score of 700
    And Alice has a monthly income of $2,000
    When Alice applies for a loan of $50,000
    Then the loan should be "REJECTED"
    And the rejection reason should be "INSUFFICIENT_INCOME"
```

--

## Scenario: Edge Cases

```gherkin
  Scenario: Borderline credit score requires manual review
    Given a customer named "Charlie"
    And Charlie has a credit score of 620
    And Charlie has a monthly income of $6,000
    When Charlie applies for a loan of $8,000
    Then the loan should be "PENDING_REVIEW"
    And the review reason should be "BORDERLINE_CREDIT"

  Scenario: Handle credit bureau unavailable
    Given a customer named "Diana"
    And the credit bureau service is unavailable
    When Diana applies for a loan of $5,000
    Then the loan should be "PENDING_REVIEW"
    And the review reason should be "CREDIT_CHECK_FAILED"
```

--

## Scenario Outline: Data-Driven Tests

```gherkin
  Scenario Outline: Loan decisions based on credit score
    Given a customer with credit score of <score>
    And monthly income of $5,000
    When they apply for a loan of $10,000
    Then the loan should be "<decision>"

    Examples:
      | score | decision       |
      | 800   | APPROVED       |
      | 750   | APPROVED       |
      | 650   | APPROVED       |
      | 620   | PENDING_REVIEW |
      | 580   | REJECTED       |
      | 450   | REJECTED       |
```

--

## Scenario Outline: Multiple Factors

```gherkin
  Scenario Outline: Loan evaluation with multiple factors
    Given a customer with credit score of <credit>
    And monthly income of $<income>
    And loan amount of $<amount>
    When the loan application is evaluated
    Then the decision should be "<decision>"

    Examples:
      | credit | income | amount | decision       |
      | 750    | 8000   | 10000  | APPROVED       |
      | 750    | 3000   | 50000  | REJECTED       |
      | 500    | 10000  | 5000   | REJECTED       |
      | 680    | 6000   | 15000  | PENDING_REVIEW |
```

--

## Background: Common Setup

```gherkin
Feature: Loan Application Evaluation

  Background:
    Given the credit bureau service is available
    And the fraud detection service is available
    And the bank verification service is available

  Scenario: Approve loan for good customer
    Given a customer with credit score of 750
    When they apply for a loan of $10,000
    Then the loan should be "APPROVED"

  Scenario: Reject loan for bad credit
    Given a customer with credit score of 450
    When they apply for a loan of $10,000
    Then the loan should be "REJECTED"
```

--

## Tags: Organizing Scenarios

```gherkin
@loan @underwriting
Feature: Loan Application Evaluation

  @smoke @critical
  Scenario: Approve loan for good customer
    Given a customer with credit score of 750
    When they apply for a loan of $10,000
    Then the loan should be "APPROVED"

  @regression
  Scenario: Reject for low credit
    Given a customer with credit score of 450
    When they apply for a loan of $10,000
    Then the loan should be "REJECTED"

  @wip
  Scenario: New fraud detection rules
    # Work in progress...
```

--

## Running Tagged Tests

```bash
# Run only smoke tests
mvn test -Dcucumber.filter.tags="@smoke"

# Run critical but not WIP
mvn test -Dcucumber.filter.tags="@critical and not @wip"

# Run loan OR underwriting tagged
mvn test -Dcucumber.filter.tags="@loan or @underwriting"
```

--

## Doc Strings: Large Text Blocks

```gherkin
  Scenario: Rejection letter contains proper details
    Given a customer named "John" with credit score 450
    When John's loan application is rejected
    Then the rejection letter should contain:
      """
      Dear John,

      We regret to inform you that your loan application
      has been declined due to: LOW_CREDIT_SCORE.

      You may reapply after 6 months.

      Sincerely,
      Loan Department
      """
```

--

## Data Tables: Structured Input

```gherkin
  Scenario: Evaluate customer with complete profile
    Given a customer with the following details:
      | field         | value          |
      | name          | John Smith     |
      | credit_score  | 720            |
      | monthly_income| 7500           |
      | employer      | Acme Corp      |
      | years_employed| 5              |
    When the loan application is submitted for $20,000
    Then the loan should be "APPROVED"
```

--

## Data Tables: Multiple Records

```gherkin
  Scenario: Batch loan processing
    Given the following loan applications:
      | customer | credit | income | amount |
      | John     | 750    | 8000   | 10000  |
      | Jane     | 450    | 5000   | 15000  |
      | Bob      | 680    | 6000   | 8000   |
    When all applications are processed
    Then the results should be:
      | customer | decision       |
      | John     | APPROVED       |
      | Jane     | REJECTED       |
      | Bob      | PENDING_REVIEW |
```

--

## Complete Feature File Example

```gherkin
@loan-underwriting
Feature: Automated Loan Underwriting
  
  As a loan officer
  I want automatic loan evaluation
  So that customers receive quick decisions

  Background:
    Given all external services are available

  @smoke @happy-path
  Scenario: Approve loan for qualified customer
    Given a customer named "John"
    And John has a credit score of 750
    And John has a monthly income of $8,000
    And John has no fraud history
    When John applies for a loan of $10,000
    Then the loan should be "APPROVED"
    And the approval should be instant
```

--

## Step Definitions (Java)

```java
public class LoanUnderwritingSteps {

    private Customer customer;
    private LoanApplication application;
    private LoanDecision decision;
    
    @Autowired
    private UnderwritingService underwritingService;

    @Given("a customer named {string}")
    public void createCustomer(String name) {
        customer = new Customer(name);
    }

    @Given("{word} has a credit score of {int}")
    public void setCreditScore(String name, int score) {
        customer.setCreditScore(score);
    }

    @Given("{word} has a monthly income of ${int}")
    public void setIncome(String name, int income) {
        customer.setMonthlyIncome(income);
    }

    @Given("{word} has no fraud history")
    public void setNoFraud(String name) {
        customer.setFraudFlag(false);
    }

    @When("{word} applies for a loan of ${int}")
    public void applyForLoan(String name, int amount) {
        application = new LoanApplication(customer, amount);
        decision = underwritingService.evaluateLoan(application);
    }

    @Then("the loan should be {string}")
    public void verifyDecision(String expectedStatus) {
        assertEquals(expectedStatus, decision.getStatus());
    }
}
```

--

## Project Structure

```text
src/
├── main/java/
│   └── com/bank/underwriting/
│       ├── UnderwritingService.java
│       ├── LoanApplication.java
│       └── LoanDecision.java
│
└── test/
    ├── java/
    │   └── com/bank/underwriting/
    │       ├── CucumberTestRunner.java
    │       └── steps/
    │           └── LoanUnderwritingSteps.java
    │
    └── resources/
        └── features/
            ├── loan_approval.feature
            ├── loan_rejection.feature
            └── edge_cases.feature
```

--

## Test Runner Configuration

```java
@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(
    key = GLUE_PROPERTY_NAME, 
    value = "com.bank.underwriting.steps"
)
@ConfigurationParameter(
    key = PLUGIN_PROPERTY_NAME, 
    value = "pretty, html:target/cucumber-report.html"
)
public class CucumberTestRunner {
}
```

--

## Gherkin Best Practices

| Do ✅ | Don't ❌ |
|------|---------|
| Write in business language | Use technical jargon |
| Keep scenarios short (5-8 steps) | Write 20+ step scenarios |
| One behavior per scenario | Test multiple things |
| Use Scenario Outline for data | Copy-paste scenarios |
| Declarative ("is approved") | Imperative ("click button") |

--

## Good vs Bad Scenarios

```gherkin
# ❌ BAD - Too technical, too detailed
Scenario: Test loan
  Given I open the database connection
  And I insert customer record with ID 123
  And I call the POST /api/loans endpoint
  And I parse the JSON response
  Then the status field equals "APPROVED"

# ✅ GOOD - Business language, behavior focused
Scenario: Approve qualified customer
  Given a customer with good credit
  When they apply for a loan within their means
  Then the loan should be approved
```

--

## The Cucumber Lifecycle

```text
  Feature File          Step Definitions         System
       │                       │                    │
       │   1. Read Gherkin     │                    │
       ├──────────────────────>│                    │
       │                       │                    │
       │   2. Match Steps      │                    │
       │<──────────────────────┤                    │
       │                       │                    │
       │   3. Execute Code     │                    │
       │                       ├───────────────────>│
       │                       │                    │
       │                       │   4. Return Result │
       │                       │<───────────────────┤
       │                       │                    │
       │   5. Report Pass/Fail │                    │
       │<──────────────────────┤                    │
```

--

## Benefits of BDD with Cucumber

- 📝 **Living Documentation** — Tests describe system behavior
- 🤝 **Collaboration** — Business & tech speak same language
- 🔍 **Clarity** — Requirements become executable specs
- ♻️ **Reusability** — Step definitions reused across scenarios
- 📊 **Readable Reports** — Non-technical stakeholders understand results

--

## Key Takeaways

- 🗣️ **Gherkin is plain English** — Anyone can read and write it
- 📋 **Feature files are specs** — Living documentation
- 🔗 **Steps map to code** — Glue between English and Java
- 📊 **Scenario Outlines** — Data-driven testing made easy
- 🏷️ **Tags organize tests** — Run subsets easily
- 🎯 **Focus on behavior** — Not implementation details