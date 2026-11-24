---
name: observability
description: Observability patterns for services. Use when adding tracing spans, structured logs or metrics, when deciding how to record an error, or when a test needs to assert on the spans a request produces.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Observability

STARTER_CHARACTER = 🔭

## Why This Matters

Observability is how production gives us feedback. A test tells us the code does what we expected. A trace tells us what it actually did, for a real user, under real load. When the trace is missing, misleading or unsafe to read, that feedback loop breaks, and we debug by guessing.

Treat telemetry as a feature: it has users (the people on call), it has a contract (span names and fields), and it can be tested.

Examples below use Rust's `tracing` crate with OpenTelemetry. The principles apply in any language.

---

## Spans Record Nothing by Default

Record only the fields you choose. An argument that enters a span by accident can leak personal data, and a large one bloats every trace.

✅ **Opt in to each field:**
```rust
#[tracing::instrument(skip_all, fields(order.id = %order_id))]
fn load_order(&self, order_id: Uuid, request: &Request) -> Result<Order, RepoError> { ... }
```

❌ **Records every argument, including the request:**
```rust
#[tracing::instrument]
fn load_order(&self, order_id: Uuid, request: &Request) -> Result<Order, RepoError> { ... }
```

If the team wraps `instrument` in its own macro, the wrapper should make `skip_all` the default, so the safe choice is the easy one.

### Naming

- Give spans a dotted name that groups them: `db.orders.load`, `http.client.payments.charge`
- Span names within a group are unique, so a name finds one place in the code
- Use [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/) where they exist: `db.system`, `http.request.method`, `http.route`, `http.response.status_code`, `error.type`
- Namespace custom fields by domain: `order.id`, `customer.id`, `caller.kind`
- Use `%value` to record with `Display` and `?value` to record with `Debug`
- Declare a field as `tracing::field::Empty` when its value is known only later, then fill it with `Span::current().record(...)`

---

## Never Record Personal Data

Traces are copied to third-party vendors, kept for weeks and read by many people. Treat every field as public inside the company.

**Never record:** email addresses, names, identity-provider subject ids, session tokens, raw JWTs, or request and response bodies.

**Sanitise database errors before you record them.** Database errors often echo the values passed to the query. For example, a Postgres `DETAIL` line repeats the values that broke a constraint. Formatting the raw error with `to_string()` copies those values into the trace. Write one sanitiser that keeps the safe metadata (error code, constraint name, table) and drops the values, and use it everywhere a database error becomes a string.

---

## Record Errors at the Right Severity

An ERROR-level event usually marks the span as failed. Dashboards and alerts count failed spans, so the level is a statement about the system's health.

| What happened | Level | Why |
|---------------|-------|-----|
| Expected domain outcome returned as `Err` (not found, rejected input) | `info` | The system worked correctly |
| The client sent something invalid | `warn` or lower | The client is at fault, not the service |
| The service or a dependency failed | `error` | Someone may need to act |

Decide severity where it is known. A repository does not know whether `NotFound` is a problem; the request handler does. Record errors from lower layers at `info`, and let the layer that maps them to a response decide whether the request failed.

**Never swallow the source error.** When you map a detailed error to a coarser one, record the source on the current span first. See the `rust` skill for the `map_err` pattern.

---

## Keep Context Across Tasks

Span context follows ordinary `async` calls, but not a new task or thread. Without the parent, child spans start a new trace and the request's trace has holes in it.

```rust
// async task: attach the current span to the future
tokio::spawn(work.instrument(tracing::Span::current()));

// blocking task: enter the span inside the closure
let span = tracing::Span::current();
tokio::task::spawn_blocking(move || {
    let _entered = span.enter();
    heavy_work()
});
```

---

## Test with Spans

Traces are an interface, so tests can assert on them. Export spans from the system under test to an in-process collector, and let tests query it.

### Find your own trace

E2E tests run in parallel, so the collector holds spans from many tests. A test finds the trace of its own request by span name plus an attribute that only it uses, for example the id of the record it created. (See *Own What You Assert On* in the `testing` skill.)

### Export only what tests need

Keep the test server's default log level high (for example `error`), and enable `info` only for the modules whose spans the tests assert on. This keeps the collector small and the tests fast. Record in the test setup which modules are enabled, because a span from any other module is not exported.

---

## Anti-Patterns

- ❌ **Recording all arguments by default** — leaks personal data and bloats traces
- ❌ **`to_string()` on a database error** — copies query values into the trace
- ❌ **Logging expected outcomes or client mistakes at `error`** — marks healthy requests as failed and makes alerts noisy
- ❌ **`.map_err(|_| ...)`** — the reason for the failure is lost
- ❌ **`tokio::spawn` without `.instrument(...)`** — breaks the request's trace in two

---

## Summary Checklist

- [ ] New spans use `skip_all` and opt in to each field
- [ ] Span and field names follow the semantic conventions or a domain namespace
- [ ] No personal data in span fields or log messages
- [ ] Database errors pass through a sanitiser before they become strings
- [ ] Expected `Err` outcomes are recorded at `info`; client mistakes at `warn` or lower
- [ ] Every coarsening `map_err` records the source error first
- [ ] Spawned tasks carry the parent span