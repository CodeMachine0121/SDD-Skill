# Deep Module

> From John Ousterhout, *A Philosophy of Software Design*.

A **deep module** hides a large amount of complexity behind a simple interface.  
A **shallow module** has a complex interface relative to the functionality it provides — it leaks its internals to callers.

## Visual

```
Deep (good)                Shallow (bad)
┌─────────┐                ┌──────────────────────────────┐
│  Place  │                │ ValidateStock                │
│  Order  │                │ ReserveInventory             │
└────┬────┘                │ CalculatePricing             │
     │                     │ CreateOrderRecord            │
  complex                  │ PublishOrderCreatedEvent     │
  internals                └──────────────────────────────┘
  hidden                   (caller sequences the steps)
```

## Principles

| Principle | Meaning | Code signal |
|-----------|---------|-------------|
| **Simple interface** | One method does one *business* thing | Verb-phrase name, ≤ 2 params |
| **Hide decisions** | Algorithms, sequences, fallbacks are internal | No leaking `private` methods — extract a collaborator instead |
| **Absorb complexity** | The module bears the pain so callers don't have to | Caller has zero conditional logic about *how* the work is done |
| **Stable abstraction** | Interface changes rarely; implementation changes freely | `IXxxService` contract stays constant across sprints |

## Diagnostic Questions

Ask these when reviewing a service boundary:

1. Does the caller have to call multiple methods in a specific order to complete one business action? → **shallow** — merge into one method.
2. Does the caller need to know which sub-step failed to recover correctly? → the module is leaking internal structure.
3. Does adding a new implementation variant require changing callers? → missing abstraction (Strategy / polymorphism).
4. Is the public interface larger than the implementation? → the module has negative depth; inline it.

## Shallow vs Deep Signs

| Shallow sign | Deep sign |
|-------------|-----------|
| Caller sequences 3+ method calls | Caller makes 1 call |
| Method name contains "And" or "Then" | Method name is a single business verb |
| Parameter count grows each sprint | Interface stable; DTO absorbs new fields |
| Callers duplicate validation / null-check logic | Module validates once, internally |
