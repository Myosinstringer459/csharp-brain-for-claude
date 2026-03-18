# Architecture Decision Records (ADR)

## Template

```markdown
# ADR-{NNN}: {Title}

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXX
**Date:** YYYY-MM-DD

## Context
What problem are we solving? What constraints exist?

## Decision
What we decided and why.

## Alternatives Considered
| Option | Pros | Cons |
|--------|------|------|
| A      | ...  | ...  |
| B      | ...  | ...  |

## Consequences
- **Positive:** what improves
- **Negative:** what trade-offs we accept
- **Risks:** what could go wrong
```

---

## Accepted Decisions

### ADR-001: Modular Monolith > Microservices

**Status:** Accepted

**Context:** Most projects are single-developer, 1-3 bounded contexts. Microservices add operational overhead without benefit.

**Decision:** Modular Monolith as default. Clean Architecture + Vertical Slices inside. Microservices only when explicitly needed (independent deploy, different SLAs, different teams).

**Consequences:**
- (+) Simple deploy, no network latency between modules
- (+) Single DB, transactions without distributed saga
- (-) Requires discipline on boundaries (NetArchTest helps)

---

### ADR-002: Result\<T\> > Exceptions for flow control

**Status:** Accepted

**Context:** Exceptions are expensive (stack trace), implicit (not visible in signature), break Railway-oriented pipeline.

**Decision:** All expected errors (not found, validation, conflict) via `Result<T>`. Exceptions only for infrastructure failures (DB down, network timeout).

**Consequences:**
- (+) Explicit contract: method returns `Result<T>`, caller must handle
- (+) Pattern matching by error type
- (-) Slightly more boilerplate (but IDE autocomplete helps)

---

### ADR-003: EF Core + Dapper (hybrid)

**Status:** Accepted

**Context:** EF Core is convenient for CRUD and migrations, but generates suboptimal SQL for complex reports.

**Decision:** EF Core for write-side (Commands) and simple Queries. Dapper for complex read-side queries (reports, aggregates, raw SQL). Both use same DbContext / connection.

**Consequences:**
- (+) Better SQL for complex queries
- (+) EF Core for migrations and change tracking
- (-) Two approaches to data access (but separated by CQRS)

---

### ADR-004: Vertical Slices > Layer-per-folder

**Status:** Accepted

**Context:** Layer-per-folder (Controllers/, Services/, Repositories/) creates shotgun surgery — one feature scattered across 5 folders.

**Decision:** Feature folders: `Features/Orders/CreateOrder/` contains Command, Handler, Validator, Endpoint. Shared abstractions in Domain and Application root.

**Consequences:**
- (+) Entire feature in one place
- (+) Easy to delete / move a feature entirely
- (-) Shared logic must be extracted consciously to avoid duplication
