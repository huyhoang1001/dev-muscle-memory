# Architectural Patterns Catalog

## Layered Architecture

**Signs:**
- Folders named `controller/`, `service/`, `repository/`, `model/`
- Strict import direction: outer layers import inner, never reverse
- DTOs or view models at boundaries

**How to identify:**
- Follow the import graph from entry point inward
- Each layer has a single concern (HTTP, business logic, storage)

---

## Plugin / Extension System

**Signs:**
- Trait objects (`dyn Trait`), interfaces, or abstract base classes as primary extension points
- Registry or factory pattern for registering implementations
- Config-driven behavior selection

**How to identify:**
- Look for `register`, `plugin`, `extension`, `handler` naming
- Find the place where implementations are wired together

---

## Event-Driven / Pub-Sub

**Signs:**
- Event bus, message broker, or channel primitives
- `emit`, `publish`, `subscribe`, `on(event)` APIs
- Decoupled producers and consumers

**How to identify:**
- Find event type definitions (enums, union types, message structs)
- Trace from event emission to handler

---

## Actor Model

**Signs:**
- Actors, agents, or workers with isolated state
- Message passing instead of shared memory
- Supervision trees

**How to identify:**
- Look for `spawn`, `send`, `receive`, `mailbox` primitives
- Check if state is ever shared directly

---

## Middleware Pipeline

**Signs:**
- Chain of handlers, each calling `next()`
- Request/response transformation at each step
- Composable via `use()` or similar

**How to identify:**
- Find the pipeline assembly point
- Trace a request through each middleware layer

---

## CQRS / Event Sourcing

**Signs:**
- Separate read and write models
- Command and query objects
- Event store or audit log as source of truth

**How to identify:**
- Look for `Command`, `Query`, `Event` type naming
- Check if state is derived from a log rather than mutated directly

---

## Domain-Driven Design (DDD)

**Signs:**
- Bounded contexts with explicit boundaries
- Aggregates, entities, value objects, domain events
- Repository pattern for persistence abstraction

**How to identify:**
- Rich domain model with behavior, not just data
- Ubiquitous language reflected in naming
