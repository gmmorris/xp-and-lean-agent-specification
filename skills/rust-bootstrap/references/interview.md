# Architecture Interview

Run these questions in order. Defaults are in parentheses — accept them if the user wants to move fast. Do not proceed to generation until all answers are confirmed.

## 1. Project Purpose

"What will this project do? One or two sentences."

This informs naming, domain module naming, and the README. Record it verbatim.

## 2. Project Shape

"Is this API-only, or will it include a web frontend?"

- **API-only** (default): Rust binary serves JSON over HTTP. No frontend concerns.
- **API + frontend**: Rust binary serves both JSON API and a bundled frontend (React/Vite). Adds frontend scaffold, build orchestration, and E2E browser tests.

## 3. Web Framework

Default: The latest **Axum** with Tokio async runtime. This is the right default — only raise alternatives if the user has a specific reason (e.g. actix-web for actor model, poem for OpenAPI-first workflow).

## 4. Database

"Does the project need a database, or is it stateless/file-based?"

- **None** (default for tools and stateless services)
- **SQLite** via `rusqlite` with bundled feature — zero infrastructure, single-node, good for desktop tools and single-user services
- **PostgreSQL** via `sqlx` with compile-time query checking — for multi-user or production services; requires `DATABASE_URL` at compile time

## 5. Observability

OpenTelemetry is always included. This is not negotiable. `tracing` is the instrumentation API throughout the codebase; OTel is the export pipeline. Every project gets traces and metrics exported via OTLP from day one — retrofitting observability later is painful and the cost upfront is low.

Tell the user this is the default and only ask the two questions below.

### 5a. Log format

"Do you need structured JSON logs (for a log aggregation pipeline like Loki, Datadog, or CloudWatch), or is plain text fine for now?"

- **Plain text** (default): `tracing_subscriber::fmt::layer()` — readable in a terminal; fine for most projects
- **JSON**: `tracing_subscriber::fmt::layer().json()` — required when logs are machine-parsed by a pipeline; the instrumentation code never changes, only the subscriber layer

### 5b. Prometheus scrape endpoint

"Will this service sit behind existing Prometheus infrastructure that scrapes `/metrics` directly?"

- **No** (default): metrics flow out via OTLP only; a collector or backend receives them
- **Yes**: add `GET /metrics` returning Prometheus text format, bridged from the OTel metrics SDK via `opentelemetry-prometheus` — this satisfies scrapers without abandoning OTel as the source of truth

If the user is unsure, default to **No**. The scrape endpoint is additive and can be added later; the OTel foundation cannot be added later without touching every instrumented callsite.

### What is always generated (no questions)

- `tracing` as the instrumentation API — `#[instrument]`, `tracing::info!()`, spans — this never changes regardless of backend
- `tracing-opentelemetry` bridging `tracing` spans to the OTel SDK
- `opentelemetry-otlp` exporting traces and metrics via OTLP gRPC
- `tower-http` `TraceLayer` on every HTTP request — method, path, status, latency
- Jaeger all-in-one in `compose.yml` — local OTLP receiver + trace UI at `http://localhost:16686`
- `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_SERVICE_NAME` in `.env.example`

## 6. CLI Tooling

"Do you want a CLI binary for interacting with the service — admin commands, migrations, seeding data?"

- **No** (default for simple services)
- **Yes**: second binary target using `clap` with derive macros and subcommands; CLI reuses domain logic from `lib.rs`

## 7. Testing Strategy

Three layers, each catching different things. Clarify the distinction before asking — users often conflate them.

**Inline unit tests** (`#[cfg(test)]` in source files): always included. Tests plain functions in isolation. Fast, no I/O.

**Rust integration tests** (`tests/` directory): always included. Tests the HTTP layer via in-process request construction using `axum-test` or similar — no real port, no binary, but exercises routing, middleware, and handler wiring.

**API E2E tests**: spins up the real compiled binary on a real port, hits it with HTTP from outside the process. Catches startup sequencing, env var loading, port binding, actual serialization, and Docker service wiring — things in-process tests cannot see. Relevant for **all** projects, not just those with a frontend.

**Browser E2E tests**: API E2E + a headless browser navigating the UI. Only meaningful with a frontend.

### 7a. API E2E tests

"Do you want API E2E tests that spin up the real binary and hit it over HTTP?" (default: **Yes**)

- **Yes** (default): adds Playwright in API-only mode (`e2e/` directory, no browser); initial smoke test hits `GET /health`; server auto-starts via `playwright.config.ts` webServer config
- **No**: skip; rely on Rust integration tests alone

Playwright is the right default here even for API-only projects — it has a clean request API, good assertion ergonomics, parallel test execution, and zero extra cost when a frontend is added later. It does require Node.js.

### 7b. Browser E2E tests

Only ask this if **both** API + frontend shape was chosen **and** API E2E was chosen.

"Do you want browser E2E tests on top of the API tests?" (default: **Yes** when frontend chosen)

- **Yes**: Playwright runs with a Chromium browser in addition to API tests; tests can drive the UI as a real user would
- **No**: Playwright tests the API only, no browser launched

## 8. Build Tooling

xtask is always used. It is a Rust workspace member that compiles to a build binary — build logic lives in Rust, is refactorable, and can be tested. It is not optional.

The core tasks are always generated based on what the interview selected:

| Task | Always | Condition |
|------|--------|-----------|
| `dev` | yes | hot-reload server (+ frontend watch if frontend chosen) |
| `lint` | yes | clippy + fmt check |
| `check` | yes | delegates to `check.sh` |
| `docker-up` / `docker-down` | if Docker chosen | wraps `docker compose` |
| `frontend-build` | if frontend chosen | runs `npm run build` in `frontend/` |
| `db-migrate` | if DB chosen | runs pending migrations |
| `e2e` | if E2E chosen | runs Playwright |

"Are there any additional domain-specific tasks you want pre-bundled — for example, seeding test data, generating fixtures, resetting the local database, or calling an external API during development?"

Accept a list. Each becomes a named xtask command. If the user isn't sure, skip for now — tasks are easy to add later.

## 9. Docker Services

"Do you want a Docker Compose file for running local infrastructure (database, cache, etc.)?"

- **Yes, always** (default when PostgreSQL is chosen — it needs something to run against): `compose.yml` defining service containers; the Rust binary itself is NOT containerised here, just the services it depends on
- **Yes, optional** (for SQLite or stateless projects): offer it for future-proofing if the user anticipates needing Redis, a message broker, or other services
- **No**: skip; document manual setup in README

When Docker is included, also add:
- `compose.yml` at the project root
- A `compose.override.yml.example` for per-developer overrides
- `.env.example` entries for any ports or credentials exposed by the services

## 10. CI

"Do you want a GitHub Actions workflow?"

- **Yes** (default): `.github/workflows/ci.yml` running `cargo test`, `cargo clippy -- -D warnings`, `cargo fmt --check`
- **No**: skip for now

---

## Confirmation

Before generating any code, present all choices as a table:

| Dimension | Choice |
|-----------|--------|
| Project shape | ... |
| Web framework | ... |
| Database | ... |
| Log format | plain text \| JSON |
| Prometheus scrape endpoint | yes \| no |
| CLI tooling | yes \| no |
| API E2E tests | yes \| no |
| Browser E2E tests | yes \| no \| n/a (API-only) |
| Extra xtask commands | [list or none] |
| Docker services | yes \| no |
| CI | yes \| no |

Ask: "These are your foundation choices. Confirm to begin generation, or adjust anything."

Wait for explicit confirmation.
