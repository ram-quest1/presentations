## Guidelines 
As a developer you will commonly write 
- Unit Tests
- E2E/Integration Tests 
- Smoke Tests
- Regression Tests 

### You may also write 
- Load Tests (Performance Engineer)
- Acceptance Tests (QA, SDET)

--

## How NOT to test?

- CRUD Apps for example

```text
Controller --> Service --> Repository --> DB
```

- Mock service and test controller
- Mock Repository and test service 
- Mock DB and test repository

**Making a mockery of ourselves 🤡**

--

## Some rules 
- Except for unit tests never mock DB, we have docker and containers
- Mock 3rd party services in integration tests which run in CI/CD pipelines mostly 
- Not everything needs to be tested
- Don't obssess over the coverage % (your manager will anyways 🤣)
    - Testing getter/setters 🤡
    - Getter/Setters should be tested indirectly as part of application flow
- Prefer BDD based testing for complex applications

--

## Some rules 
- Talk with your QA and team leads on the test strategy and work accordingly
- Same person who writes code **should** write unit test since context is important
   - Example from past, new team came in earlier coverage was 35%, they increased to 90%
   - Still most issues remained 🤡