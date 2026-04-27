# Refactor Guide (C#)

Run **all module tests** after each move. Stop if any fail.

## Rules

| # | Rule | Violation → Fix |
|---|------|----------------|
| 1 | No `private` method | Too much / too little responsibility → extract class or inline |
| 2 | No `private static` method | Method belongs to another type → move into that type |
| 3 | Method ≤ 5 responsibilities | Count steps, not lines → split method or extract Decorator |
| 4 | Parameters > 2 | Parameter cluster → extract `record` DTO |
| 5 | Complex pre-conditions | Validation / auth / logging → Decorator pattern |
| 6 | `if`/`switch` dispatching implementations | Missing abstraction → Strategy pattern |
| 7 | Service/Domain layer is pure | No framework `using` → hide behind `IRepository` interface |
| 8 | Model conversions on source model | Scattered mapping → move to `source.ToXxx()` instance method |
| 9 | Fluent business logic | Imperative mutation → method chaining |
| 10 | No deferred assignment | Declare then assign → ternary / LINQ / `switch` expression |

## DI Lifetime

| Lifetime | Use when |
|----------|----------|
| Transient | Lightweight, stateless |
| Scoped | Per-request state (default; DbContext) |
| Singleton | App-wide: config, cache, logging |

❌ Never inject Scoped/Transient into Singleton → use `IServiceScopeFactory`  
❌ Never inject Transient into Scoped

## Checklist (in order)

1. Resolve all `private` / `private static` violations
2. Split any method exceeding 5 responsibilities
3. Extract DTO record for parameters > 2
4. Extract Decorator for complex pre-conditions
5. Extract Strategy for conditional branch implementations
6. Remove framework `using` from Service/Domain — add `IRepository`
7. Move model conversions to source model as `To...()`
8. Rewrite imperative sequences as fluent chains
9. Replace deferred assignments with ternary / LINQ / switch expression
10. Verify DI lifetimes — no lifetime pollution

