
## Testing in CI/CD Pipelines

Where, When, and Why Different Tests Run

--

## The Big Picture

```text
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Code   │──>│  Build  │──>│  Test   │──>│ Deploy  │──>│ Release │
│ Commit  │   │         │   │         │   │         │   │         │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
                  │              │             │             │
                  ▼              ▼             ▼             ▼
              Compile       Unit Tests    Integration   Smoke Tests
              Lint          Coverage      API Tests     Canary
              SAST          SAST          E2E Tests     Monitoring
```

--

## CI/CD Pipeline Stages Overview

| Stage | Purpose | Duration | Frequency |
|-------|---------|----------|-----------|
| **Commit** | Validate code quality | Seconds | Every push |
| **Build** | Compile & package | Minutes | Every push |
| **Unit Test** | Component validation | Minutes | Every push |
| **Integration** | Service interaction | 10-30 min | Every PR |
| **Staging** | Production-like tests | 30-60 min | Pre-deploy |
| **Production** | Release validation | Minutes | Each release |

--

## Stage 1: Commit Stage

**Trigger:** Every git push

```text
┌──────────────────────────────────────────┐
│            COMMIT STAGE                  │
│         (Seconds - 2 minutes)            │
├──────────────────────────────────────────┤
│  ✓ Pre-commit hooks                      │
│  ✓ Linting (ESLint, Checkstyle)          │
│  ✓ Code formatting (Prettier, SpotBugs)  │
│  ✓ Static Analysis (SonarQube, SAST)     │
│  ✓ Secrets scanning                      │
│  ✓ Dependency vulnerability check        │
└──────────────────────────────────────────┘
              │
              ▼
         Pass? ──No──> ❌ Block commit
              │
             Yes
              ▼
         Continue to Build
```

--

## Stage 1: Tools & Checks

```yaml
# .github/workflows/commit.yml
commit-checks:
  runs-on: ubuntu-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Lint Code
      run: npm run lint
      
    - name: Check Formatting
      run: npm run format:check
      
    - name: Security Scan
      uses: github/codeql-action/analyze@v2
      
    - name: Secrets Detection
      uses: trufflesecurity/trufflehog@main
```

--

## Stage 2: Build Stage

**Trigger:** After commit checks pass

```text
┌──────────────────────────────────────────┐
│             BUILD STAGE                  │
│           (2 - 10 minutes)               │
├──────────────────────────────────────────┤
│  ✓ Compile source code                   │
│  ✓ Resolve dependencies                  │
│  ✓ Build Docker images                   │
│  ✓ Generate artifacts                    │
│  ✓ SBOM generation (Software Bill of    │
│    Materials)                            │
└──────────────────────────────────────────┘
              │
              ▼
         Pass? ──No──> ❌ Notify & Stop
              │
             Yes
              ▼
         Continue to Unit Tests
```

--

## Stage 3: Unit Testing

**Trigger:** After successful build

```text
┌──────────────────────────────────────────┐
│          UNIT TEST STAGE                 │
│           (5 - 15 minutes)               │
├──────────────────────────────────────────┤
│  ✓ Run all unit tests                    │
│  ✓ Generate code coverage report         │
│  ✓ Mutation testing (optional)           │
│  ✓ Enforce coverage thresholds           │
│                                          │
│  Tools: JUnit, Jest, pytest, Go test     │
│  Coverage: JaCoCo, Istanbul, Coverage.py │
└──────────────────────────────────────────┘
              │
              ▼
      Coverage > 80%? ──No──> ❌ Fail Pipeline
              │
             Yes
              ▼
         Continue to Integration
```

--

## Stage 3: Unit Test Pipeline Config

```yaml
unit-tests:
  runs-on: ubuntu-latest
  needs: build
  steps:
    - name: Run Unit Tests
      run: mvn test
      
    - name: Generate Coverage Report
      run: mvn jacoco:report
      
    - name: Check Coverage Threshold
      run: |
        COVERAGE=$(cat target/site/jacoco/index.html | grep -o 'Total[^%]*%' | grep -o '[0-9]*')
        if [ "$COVERAGE" -lt 80 ]; then
          echo "Coverage $COVERAGE% is below 80% threshold"
          exit 1
        fi
        
    - name: Upload Coverage to Codecov
      uses: codecov/codecov-action@v3
```

--

## Stage 4: Integration Testing

**Trigger:** After unit tests pass (often on PR)

```text
┌──────────────────────────────────────────┐
│       INTEGRATION TEST STAGE             │
│          (10 - 30 minutes)               │
├──────────────────────────────────────────┤
│  ✓ Spin up test containers (Testcontainers)│
│  ✓ Database integration tests            │
│  ✓ Message queue tests                   │
│  ✓ External service integration          │
│  ✓ API contract tests                    │
│                                          │
│  Tools: Testcontainers, WireMock,        │
│         Docker Compose, LocalStack       │
└──────────────────────────────────────────┘
              │
              ▼
         Pass? ──No──> ❌ Block PR Merge
              │
             Yes
              ▼
         Continue to API Tests
```

--

## Stage 4: Integration Test Example

```yaml
integration-tests:
  runs-on: ubuntu-latest
  needs: unit-tests
  services:
    postgres:
      image: postgres:15
      env:
        POSTGRES_PASSWORD: test
      ports:
        - 5432:5432
    redis:
      image: redis:7
      ports:
        - 6379:6379
        
  steps:
    - name: Run Integration Tests
      run: mvn verify -P integration-tests
      env:
        DB_URL: jdbc:postgresql://localhost:5432/test
        REDIS_URL: redis://localhost:6379
```

--

## Stage 5: API / Contract Testing

**Trigger:** After integration tests pass

```text
┌──────────────────────────────────────────┐
│        API / CONTRACT TEST STAGE         │
│          (10 - 20 minutes)               │
├──────────────────────────────────────────┤
│  ✓ REST API endpoint tests               │
│  ✓ GraphQL query tests                   │
│  ✓ Contract verification (Pact, Spring   │
│    Cloud Contract)                       │
│  ✓ OpenAPI schema validation             │
│  ✓ Backward compatibility checks         │
│                                          │
│  Tools: RestAssured, Pact, Karate,       │
│         Postman/Newman, Dredd            │
└──────────────────────────────────────────┘
              │
              ▼
       Contracts Valid? ──No──> ❌ Block Deploy
              │
             Yes
              ▼
         Continue to E2E
```

--

## Stage 5: Contract Testing Flow

```text
┌─────────────┐                    ┌─────────────┐
│  Consumer   │                    │  Provider   │
│  Service    │                    │  Service    │
└──────┬──────┘                    └──────┬──────┘
       │                                  │
       │  1. Generate                     │
       │     Consumer Contract            │
       ▼                                  │
┌─────────────────┐                       │
│   Pact Broker   │ <─────────────────────┤
│  (Contract DB)  │   2. Verify Contract  │
└─────────────────┘                       │
       │                                  │
       │  3. Both sides verified?         │
       ▼                                  │
   ✓ Safe to deploy independently         │
```

--

## Stage 6: E2E / System Testing

**Trigger:** After API tests, before staging deploy

```text
┌──────────────────────────────────────────┐
│         E2E / SYSTEM TEST STAGE          │
│          (30 - 60 minutes)               │
├──────────────────────────────────────────┤
│  ✓ Full user journey tests               │
│  ✓ Cross-service workflows               │
│  ✓ UI automation tests                   │
│  ✓ Browser compatibility                 │
│  ✓ Mobile responsiveness                 │
│                                          │
│  Tools: Cypress, Playwright, Selenium,   │
│         Appium, TestCafe                 │
└──────────────────────────────────────────┘
              │
              ▼
    Critical Paths Pass? ──No──> ❌ Block Release
              │
             Yes
              ▼
         Deploy to Staging
```

--

## Stage 6: E2E Test Pipeline

```yaml
e2e-tests:
  runs-on: ubuntu-latest
  needs: api-tests
  steps:
    - name: Deploy to Test Environment
      run: kubectl apply -f k8s/test/
      
    - name: Wait for Services Ready
      run: ./scripts/wait-for-services.sh
      
    - name: Run Cypress E2E Tests
      uses: cypress-io/github-action@v5
      with:
        browser: chrome
        spec: cypress/e2e/**/*.cy.js
        
    - name: Upload Test Artifacts
      uses: actions/upload-artifact@v3
      with:
        name: cypress-screenshots
        path: cypress/screenshots
```

--

## Stage 7: Staging Environment

**Trigger:** After E2E tests pass

```text
┌──────────────────────────────────────────┐
│         STAGING ENVIRONMENT              │
│          (30 - 60 minutes)               │
├──────────────────────────────────────────┤
│  ✓ Deploy to production-like environment │
│  ✓ Performance / Load testing            │
│  ✓ Security penetration testing          │
│  ✓ Chaos engineering tests               │
│  ✓ Data migration testing                │
│  ✓ UAT (User Acceptance Testing)         │
│                                          │
│  Tools: k6, Gatling, JMeter, Locust,     │
│         OWASP ZAP, Chaos Monkey          │
└──────────────────────────────────────────┘
              │
              ▼
      Performance OK? ──No──> ❌ Investigate
              │
             Yes
              ▼
         Approve for Production
```

--

## Stage 7: Performance Test Gate

```yaml
performance-tests:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - name: Run k6 Load Tests
      run: |
        k6 run --out json=results.json \
          -e TARGET_URL=$STAGING_URL \
          tests/load/scenarios.js
          
    - name: Check Performance Thresholds
      run: |
        # Fail if p95 latency > 500ms
        # Fail if error rate > 1%
        # Fail if throughput < 1000 rps
        python scripts/check_perf_thresholds.py results.json
```

--

## Stage 8: Production Deployment

**Trigger:** Manual approval or auto after staging

```text
┌──────────────────────────────────────────┐
│       PRODUCTION DEPLOYMENT              │
│          (5 - 30 minutes)                │
├──────────────────────────────────────────┤
│  ✓ Blue/Green or Canary deployment       │
│  ✓ Smoke tests on new instances          │
│  ✓ Health check monitoring               │
│  ✓ Feature flag verification             │
│  ✓ Rollback readiness                    │
│                                          │
│  Strategies: Blue/Green, Canary,         │
│              Rolling Update, Feature Flags│
└──────────────────────────────────────────┘
              │
              ▼
      Smoke Tests Pass? ──No──> 🔄 Auto Rollback
              │
             Yes
              ▼
         Route Traffic (Gradual)
```

--

## Stage 8: Canary Deployment Flow

```text
     100% Traffic                     Gradual Shift
    ┌───────────┐                    ┌───────────┐
    │  v1.0.0   │                    │  v1.0.0   │ 90%
    │ (Current) │        ──>         ├───────────┤
    └───────────┘                    │  v1.1.0   │ 10%
                                     │ (Canary)  │
                                     └───────────┘
                                           │
                         Monitor errors, latency, metrics
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    ▼                      ▼                      ▼
              ❌ Errors High         ⚠️ Metrics OK           ✅ All Good
              Auto Rollback         Hold & Monitor         Increase to 50%
                                                           Then 100%
```

--

## Stage 9: Post-Deployment Validation

**Trigger:** After production deployment

```text
┌──────────────────────────────────────────┐
│      POST-DEPLOYMENT VALIDATION          │
│           (Continuous)                   │
├──────────────────────────────────────────┤
│  ✓ Synthetic monitoring (Datadog, NR)    │
│  ✓ Real user monitoring (RUM)            │
│  ✓ Error rate tracking                   │
│  ✓ Performance baseline comparison       │
│  ✓ Business metric validation            │
│  ✓ Alerting on anomalies                 │
│                                          │
│  Tools: Datadog, New Relic, Prometheus,  │
│         Grafana, PagerDuty, Sentry       │
└──────────────────────────────────────────┘
              │
              ▼
    Anomalies Detected? ──Yes──> 🚨 Alert & Investigate
              │
              No
              ▼
         Release Complete ✅
```

--

## Complete Pipeline Visualization

```text
                           DEVELOPER WORKFLOW
┌─────────────────────────────────────────────────────────────────────┐
│  Code ──> Commit ──> Push ──> PR ──> Review ──> Merge ──> Deploy    │
└──────────────────────────────────────────────────────────────────────┘
     │         │         │       │        │         │         │
     ▼         ▼         ▼       ▼        ▼         ▼         ▼
┌────────┐┌────────┐┌────────┐┌────────┐┌────────┐┌────────┐┌────────┐
│Pre-    ││Lint    ││Build   ││Unit    ││Integr- ││E2E     ││Smoke   │
│commit  ││SAST    ││Package ││Tests   ││ation   ││Tests   ││Tests   │
│Hooks   ││Secrets ││Docker  ││Coverage││API     ││Perf    ││Canary  │
└────────┘└────────┘└────────┘└────────┘└────────┘└────────┘└────────┘
   10s       1m        5m       10m       20m       45m       10m
   
├─────────────────────────────────────────────────────────────────────┤
  Commit Stage │    Build    │     Test & Validate      │   Deploy
├─────────────────────────────────────────────────────────────────────┤
                    CONTINUOUS INTEGRATION          │      CD
```

--

## Test Distribution by Stage

| Stage | Test Types | % of Total Tests |
|-------|------------|------------------|
| **Commit** | Linting, SAST | N/A (Quality gates) |
| **Build** | Compilation checks | N/A |
| **Unit** | Unit tests | 70% |
| **Integration** | Service integration | 15% |
| **API** | Contract, endpoint | 8% |
| **E2E** | User journeys | 5% |
| **Production** | Smoke, synthetic | 2% |

--

## The Testing Pyramid in CI/CD

```text
                              Slower, Fewer
                                   /\
                                  /  \         Production
                                 /    \        Smoke & Synthetic
                                /──────\
                               /        \      Staging
                              /   E2E    \     Performance, Security
                             /────────────\
                            /              \   PR Merge Gate
                           /  Integration   \  API, Contract Tests
                          /──────────────────\
                         /                    \ Every Push
                        /     Unit Tests       \ Fast Feedback
                       /────────────────────────\
                                   
                              Faster, More
```

--

## Quality Gates Summary

| Gate | Criteria | Action on Fail |
|------|----------|----------------|
| **Commit** | Lint pass, no secrets | Block push |
| **Build** | Compiles, no vulnerabilities | Stop pipeline |
| **Unit** | 80%+ coverage, all pass | Block merge |
| **Integration** | All services connect | Block merge |
| **Contract** | No breaking changes | Block deploy |
| **E2E** | Critical paths pass | Block release |
| **Performance** | p95 < 500ms, errors < 1% | Block release |
| **Production** | Smoke tests pass | Auto rollback |

--

## Feedback Loop Timing

```text
Test Type        Feedback Time    When Developer Learns
─────────────────────────────────────────────────────────
Unit Tests       ~10 minutes      ☕ Coffee break
                                  
Integration      ~30 minutes      📧 Check email, then results
                                  
E2E Tests        ~1 hour          🍽️ Lunch break
                                  
Performance      ~1 hour          🍽️ Lunch break
                                  
Production       ~15 minutes      😰 Anxiously watching dashboards
                                  
─────────────────────────────────────────────────────────
Goal: Shift Left - Find bugs in cheaper, faster stages
```

--

## Environment Progression

```text
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│    Local    │   │     Dev     │   │   Staging   │   │ Production  │
│ Development │──>│ Environment │──>│ Environment │──>│ Environment │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
   Unit Tests      Integration        E2E + Perf        Smoke +
   Lint + SAST     API Tests          Security          Synthetic
                   Contract           UAT               Monitoring
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
   Developer         Dev Team          QA Team          SRE Team
   Owned             Owned             Owned            Owned
```

--

## Complete GitHub Actions Pipeline

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # Stage 1 & 2: Commit & Build
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint
        run: npm run lint
      - name: Security Scan
        run: npm audit
      - name: Build
        run: npm run build
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/
```

--

## Pipeline Continued (1/2)

```yaml
  # Stage 3: Unit Tests
  unit-tests:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Unit Tests
        run: npm test -- --coverage
      - name: Check Coverage
        run: npx nyc check-coverage --lines 80

  # Stage 4: Integration Tests  
  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
    steps:
      - name: Run Integration Tests
        run: npm run test:integration
```

--

## Pipeline Continued (2/2)

```yaml
  # Stage 5: E2E Tests
  e2e-tests:
    needs: integration-tests
    runs-on: ubuntu-latest
    steps:
      - name: Run E2E Tests
        uses: cypress-io/github-action@v5

  # Stage 6: Deploy to Production
  deploy:
    needs: e2e-tests
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy
        run: ./scripts/deploy.sh
      - name: Smoke Tests
        run: npm run test:smoke
      - name: Notify
        run: ./scripts/notify-slack.sh
```

--

## Key Takeaways

- ⚡ **Fast feedback** — Unit tests on every push (minutes)
- 🔒 **Quality gates** — Block bad code at each stage
- 📊 **Progressive confidence** — More thorough tests as code advances
- 🔄 **Shift left** — Catch bugs in cheaper stages
- 🚀 **Automated deployment** — Reduce human error
- 📉 **Rollback ready** — Always have a safety net

--

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| E2E only | Slow feedback, flaky | Add unit + integration |
| No gates | Bad code reaches prod | Enforce thresholds |
| Manual deploy | Human error, slow | Automate everything |
| No monitoring | Silent failures | Synthetic + RUM |
| Skipping stages | "Works on my machine" | Run full pipeline |
| Long pipelines | Developer frustration | Parallelize tests |

--

## Summary: Test Types by CI/CD Stage

```text
┌────────────┬────────────────────────────────────────────┐
│   STAGE    │              TEST TYPES                    │
├────────────┼────────────────────────────────────────────┤
│ Commit     │ Linting, SAST, Secrets scan                │
├────────────┼────────────────────────────────────────────┤
│ Build      │ Compilation, Dependency check              │
├────────────┼────────────────────────────────────────────┤
│ Unit       │ Unit tests, Coverage, Mutation             │
├────────────┼────────────────────────────────────────────┤
│ Integration│ DB tests, Service tests, API tests         │
├────────────┼────────────────────────────────────────────┤
│ Contract   │ Consumer/Provider contracts, Schema        │
├────────────┼────────────────────────────────────────────┤
│ E2E        │ User journeys, UI tests, Cross-browser     │
├────────────┼────────────────────────────────────────────┤
│ Staging    │ Performance, Load, Security, UAT           │
├────────────┼────────────────────────────────────────────┤
│ Production │ Smoke tests, Canary, Synthetic monitoring  │
└────────────┴────────────────────────────────────────────┘
```