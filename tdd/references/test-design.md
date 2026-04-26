# C# Test Design

## Stack
**Default:** TUnit · NSubstitute · FluentAssertions `[7.2.2]`  
**If the project already has a test framework in use → follow it instead.**

## Naming
`MethodName_Scenario_ExpectedBehavior`

## Structure (AAA)
- `[Before(Test)]` — declare SUT + substituted interfaces as fields, initialize here
- **Arrange** — setup inputs, call `Given...` helpers
- **Act** — single call to SUT
- **Assert** — one `Should()` chain

## Helper Methods
- `Given...` — extract mock setup out of Arrange block
- `Create...` — extract `new` expressions for inputs / test data

## Assertions
One `Should()` chain per test. Multiple assertions allowed only when verifying the same observable outcome.

## What to Mock

| ✅ Mock | ❌ Don't mock |
|--------|--------------|
| Repository / external service interfaces | Domain models, value objects |
| HTTP clients, message bus, email sender | Pure computation services |
| File system, clock, randomness | In-memory collections |

## Behavior Rules
- Test through **public API only** — never via reflection or internal members
- Assert on: return values, exceptions, boundary calls (`Received()`)
- Never assert on private/internal state

