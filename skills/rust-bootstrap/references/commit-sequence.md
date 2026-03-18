# Commit Sequence

Each commit leaves the project in a buildable, passing state. Run `cargo build && cargo test` after each commit. Fix any failures before continuing.

Only include commits for dimensions the interview selected. Commits marked optional are skipped when the feature was not chosen.

---

## Commit 1: Workspace scaffold

Always included.

Files:
- `Cargo.toml` — package metadata (edition 2021), workspace members if using xtask
- `src/main.rs` — `fn main() { println!("{}", env!("CARGO_PKG_NAME")); }`
- `src/lib.rs` — empty; single `// domain logic` comment
- `.gitignore` — `target/`, `.env`, `static/` (if frontend chosen)
- `README.md` — project name and one-sentence description from the interview
- `.env.example` — empty for now, populated by subsequent commits

Message: `chore: initialise Rust workspace`

---

## Commit 2: Observability foundation

Always included. Not optional.

### Dependencies

Use `cargo add` for each — never hand-write versions into `Cargo.toml`:

```bash
cargo add tracing
cargo add tracing-subscriber --features env-filter,fmt
cargo add tracing-opentelemetry
cargo add opentelemetry
cargo add opentelemetry_sdk --features rt-tokio
cargo add opentelemetry-otlp --features grpc-tonic
cargo add opentelemetry-semantic-conventions
```

**OTel version coupling warning**: `opentelemetry`, `opentelemetry_sdk`, `opentelemetry-otlp`, `opentelemetry-semantic-conventions`, and `tracing-opentelemetry` must all resolve to compatible versions. Run `cargo add` for all of them in one session so Cargo resolves them together. If you see trait impl conflicts at compile time, check that no two of these crates resolved to different major/minor OTel versions — use `cargo tree -d` to find duplicates.

### New file: `src/telemetry.rs` (in lib crate, no framework imports)

```rust
use opentelemetry::trace::TracerProvider as _;
use opentelemetry_otlp::WithExportConfig;
use opentelemetry_sdk::{runtime, trace as sdktrace};

pub struct TelemetryGuard {
    tracer_provider: sdktrace::TracerProvider,
}

impl Drop for TelemetryGuard {
    fn drop(&mut self) {
        if let Err(e) = self.tracer_provider.shutdown() {
            eprintln!("tracer shutdown error: {e}");
        }
    }
}

pub fn init(otlp_endpoint: &str, service_name: &str) -> (TelemetryGuard, sdktrace::Tracer) {
    let exporter = opentelemetry_otlp::new_exporter()
        .tonic()
        .with_endpoint(otlp_endpoint);

    let provider = opentelemetry_otlp::new_pipeline()
        .tracing()
        .with_exporter(exporter)
        .with_trace_config(
            opentelemetry_sdk::trace::Config::default()
                .with_resource(opentelemetry_sdk::Resource::new(vec![
                    opentelemetry::KeyValue::new(
                        opentelemetry_semantic_conventions::resource::SERVICE_NAME,
                        service_name.to_string(),
                    ),
                ])),
        )
        .install_batch(runtime::Tokio)
        .expect("failed to install OTel tracer");

    let tracer = provider.tracer(service_name.to_string());
    (TelemetryGuard { tracer_provider: provider }, tracer)
}
```

Export from `src/lib.rs`: `pub mod telemetry;`

### Changes to `src/main.rs`

```rust
let otlp_endpoint = std::env::var("OTEL_EXPORTER_OTLP_ENDPOINT")
    .unwrap_or_else(|_| "http://localhost:4317".into());
let service_name = std::env::var("OTEL_SERVICE_NAME")
    .unwrap_or_else(|_| env!("CARGO_PKG_NAME").into());

let (_guard, tracer) = {project_name}::telemetry::init(&otlp_endpoint, &service_name);

tracing_subscriber::registry()
    .with(tracing_subscriber::EnvFilter::new(
        std::env::var("RUST_LOG").unwrap_or_else(|_| "info".into()),
    ))
    .with(tracing_subscriber::fmt::layer())       // stdout — always present
    .with(tracing_opentelemetry::layer().with_tracer(tracer))  // OTel export
    .init();
```

`_guard` is held for the lifetime of `main` — dropping it flushes and shuts down the exporter cleanly.

**For JSON logs** (if chosen): swap `tracing_subscriber::fmt::layer()` for `tracing_subscriber::fmt::layer().json()`. Nothing else changes.

### Docker (`compose.yml`)

Always add Jaeger when Docker is included:

```yaml
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"   # Jaeger UI
      - "4317:4317"     # OTLP gRPC receiver
```

### `.env.example`

```
RUST_LOG=info
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_SERVICE_NAME={project-name}
```

### `README.md`

Add: "View traces: `http://localhost:16686` (start services with `docker compose up -d`)"

Message: `feat: add OpenTelemetry observability foundation`

---

## Commit 2b: Prometheus scrape endpoint (optional)

Only if Prometheus scrape was chosen in the interview. This bridges the OTel metrics SDK to `/metrics` — it does not introduce a separate metrics library.

### Additional dependency

```bash
cargo add opentelemetry-prometheus
```

`prometheus` is a transitive dependency — do not add it explicitly. Let Cargo resolve the version that `opentelemetry-prometheus` requires.

### Changes

New file: `src/metrics.rs` (in lib crate):

```rust
use opentelemetry_prometheus::PrometheusExporter;
use prometheus::Registry;

pub fn init_prometheus() -> (PrometheusExporter, Registry) {
    let registry = Registry::new();
    let exporter = opentelemetry_prometheus::exporter()
        .with_registry(registry.clone())
        .build()
        .expect("failed to build Prometheus exporter");
    (exporter, registry)
}
```

Export from `src/lib.rs`: `pub mod metrics;`

- `src/state.rs` — `AppState` gains `prometheus_registry: Registry`
- `src/router.rs` — add `GET /metrics`:

```rust
async fn metrics_handler(State(state): State<AppState>) -> impl IntoResponse {
    use prometheus::Encoder;
    let encoder = prometheus::TextEncoder::new();
    let mut buffer = Vec::new();
    encoder.encode(&state.prometheus_registry.gather(), &mut buffer)
        .expect("failed to encode metrics");
    (
        [(axum::http::header::CONTENT_TYPE, "text/plain; version=0.0.4")],
        buffer,
    )
}
```

If Docker is chosen, add a Prometheus service to `compose.yml` configured to scrape `host.docker.internal:{PORT}/metrics`.

Message: `feat: add Prometheus scrape endpoint via OTel metrics bridge`

---

## Commit 3: HTTP server scaffold

Always included.

Dependencies: `axum`, `tokio` (features: full), `tower-http` (features: trace), `dotenvy`

New files:
- `src/state.rs` — `AppState` struct (empty), `impl AppState { pub fn new() -> Self { Self {} } }`
- `src/router.rs` — builds `Router` from state; includes `GET /health` returning `StatusCode::OK`

Changes to `src/main.rs`:
- Load `.env` with `dotenvy::dotenv().ok();`
- Bind `PORT` from env (default `3000`)
- Start server:

```rust
let listener = tokio::net::TcpListener::bind(addr).await?;
tracing::info!("listening on {}", listener.local_addr()?);
axum::serve(listener, router).await?;
```

- `.env.example` — add `PORT=3000`

Message: `feat: add Axum HTTP server with health check`

---

## Commit 4: Error handling

Always included.

Dependencies: `thiserror`

New file: `src/errors.rs` in lib crate (no framework imports):

```rust
#[derive(Debug, thiserror::Error, PartialEq)]
pub enum AppError {
    #[error("not found")]
    NotFound,
    #[error("internal error: {0}")]
    Internal(String),
}
```

In the binary layer (`src/handlers/` or similar), implement `IntoResponse` for `AppError` — this is the only place Axum types are permitted to touch the error type.

Export from `src/lib.rs`: `pub mod errors;`

Message: `feat: add error handling foundations`

---

## Commit 5: Database layer (optional — SQLite)

Only if SQLite was chosen.

Dependencies: `rusqlite` (features: bundled)

New file: `src/db.rs` in lib crate:
- `pub fn migrate(conn: &Connection) -> Result<(), rusqlite::Error>` — creates schema
- Document the expected schema in a comment block at the top of the file

Changes to `src/state.rs`:
- `AppState` gains `upload_dir: PathBuf` and `db: Arc<Mutex<rusqlite::Connection>>`
- `AppState::new(upload_dir: PathBuf) -> Result<Self, rusqlite::Error>`

- `.env.example` — add `PERSISTENCE_TARGET=.{project-name}`

Message: `feat: add SQLite database layer`

---

## Commit 5: Database layer (optional — PostgreSQL)

Only if PostgreSQL was chosen.

Dependencies: `sqlx` (features: postgres, runtime-tokio, tls-rustls, macros)

New files:
- `migrations/` directory
- `migrations/20240101000000_initial.sql` — initial schema

New file: `src/db.rs` in lib crate:
- `pub async fn create_pool(url: &str) -> Result<PgPool, sqlx::Error>`

Changes to `src/state.rs`:
- `AppState` gains `db: PgPool`

- `.env.example` — add `DATABASE_URL=postgres://user:password@localhost/dbname`

Note in `README.md`: run `cargo sqlx prepare` after changing queries.

Message: `feat: add PostgreSQL database layer`

---

## Commit 6: xtask build orchestration

Always included.

### Files

`xtask/Cargo.toml` — binary crate, no lib, no dependencies beyond std:

```toml
[package]
name = "xtask"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "xtask"
path = "src/main.rs"
```

`xtask/src/main.rs` — command dispatcher:

```rust
use std::{env, process::Command};

fn main() {
    let task = env::args().nth(1).unwrap_or_else(|| {
        eprintln!("usage: cargo xtask <task>");
        std::process::exit(1);
    });

    let result = match task.as_str() {
        "dev"   => run_dev(),
        "lint"  => run_lint(),
        "check" => run_check(),
        // conditional tasks added here based on interview selections
        _       => {
            eprintln!("unknown task: {task}");
            std::process::exit(1);
        }
    };

    if let Err(e) = result {
        eprintln!("task failed: {e}");
        std::process::exit(1);
    }
}
```

Each task function uses `std::process::Command`. Keep task functions in separate modules under `xtask/src/tasks/` as the list grows.

`.cargo/config.toml` — aliases:

```toml
[alias]
xtask = "run --package xtask --"
```

This lets you run `cargo xtask dev`, `cargo xtask lint`, etc.

### Core tasks (always generated)

**`dev`** — hot-reload development server:
```rust
fn run_dev() -> Result<(), Box<dyn std::error::Error>> {
    cmd("cargo", &["watch", "-x", "run"])
}
```
If frontend was chosen: run Vite watch and cargo watch in parallel via two child processes.

**`lint`** — code quality gates:
```rust
fn run_lint() -> Result<(), Box<dyn std::error::Error>> {
    cmd("cargo", &["fmt", "--check"])?;
    cmd("cargo", &["clippy", "--", "-D", "warnings"])
}
```

**`check`** — delegates to `check.sh` (runs full test suite):
```rust
fn run_check() -> Result<(), Box<dyn std::error::Error>> {
    cmd("bash", &["check.sh"])
}
```

### Conditional tasks (add only if selected in interview)

**`docker-up` / `docker-down`** (if Docker chosen):
```rust
fn run_docker_up() -> Result<(), Box<dyn std::error::Error>> {
    cmd("docker", &["compose", "up", "-d"])
}
```

**`db-migrate`** (if DB chosen):
- SQLite: run the `migrate()` function via a small migration binary or inline in xtask
- PostgreSQL: `sqlx migrate run`

**`frontend-build`** (if frontend chosen):
```rust
fn run_frontend_build() -> Result<(), Box<dyn std::error::Error>> {
    cmd_in("npm", &["run", "build"], "frontend")
}
```

**`e2e`** (if E2E chosen):
```rust
fn run_e2e() -> Result<(), Box<dyn std::error::Error>> {
    cmd_in("npx", &["playwright", "test"], "e2e")
}
```

### Extra tasks from the interview

For each domain-specific task the user named in the interview, add a corresponding function stub with a descriptive comment and `todo!()` body. The user fills in the implementation. Name them after what they do: `seed-db`, `generate-fixtures`, `reset-db`, etc.

### Update `Cargo.toml`

Add xtask as a workspace member:
```toml
[workspace]
members = [".", "xtask"]
```

### `README.md`

Add a "Development tasks" section listing every generated `cargo xtask <name>` command with a one-line description.

Message: `chore: add xtask build orchestration`

---

## Commit 7: Linting configuration

Always included.

New or updated files:
- `.cargo/config.toml` — if not already created; add `[build]` and `[alias]` sections
- `rust-toolchain.toml` — pin `channel = "stable"` (or the specific version desired)

Optional (only if cargo-deny is available in the environment):
- `deny.toml`:

```toml
[advisories]
ignore = []

[licenses]
allow = ["MIT", "Apache-2.0", "ISC", "Unicode-DFS-2016"]

[bans]
multiple-versions = "warn"
```

Message: `chore: add linting and toolchain configuration`

---

## Commit 8: CLI binary (optional)

Only if CLI tooling was chosen.

Dependencies: `clap` (features: derive)

New files:
- `src/bin/cli.rs` — entry point; parses args with clap, calls into `lib.rs` functions
- `src/cli/` directory — one module per subcommand

Updated `Cargo.toml`:
```toml
[[bin]]
name = "{project-name}-cli"
path = "src/bin/cli.rs"
```

The CLI never contains business logic — it calls into `lib.rs`. Handler pattern applies: thin CLI command, plain function in lib.

Message: `feat: add CLI binary with clap subcommands`

---

## Commit 9: Frontend scaffold (optional)

Only if API + frontend was chosen.

Run outside of Rust:
```bash
npm create vite@latest frontend -- --template react-ts
```

New files:
- `frontend/` — Vite + React TypeScript project
- `templates/shell.html` — HTML shell served by the Rust binary for all non-API routes
- `static/` — gitignored build output directory

Changes to router: add fallback route serving `templates/shell.html`. Add static file serving from `static/assets/`.

Update build tooling (justfile or xtask) with:
- `frontend-build` — runs `npm run build` in `frontend/`
- `dev` — parallel: Vite watch + `cargo watch -x run`

Message: `feat: add React/Vite frontend scaffold`

---

## Commit 10: Docker services (optional)

Only if Docker was chosen in the interview.

The Rust binary is **not** containerised here. Docker is for ancillary services only — database, cache, message broker, etc.

New files:
- `compose.yml` — defines service containers required by the application

PostgreSQL example:
```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

SQLite projects: if Docker was chosen for future services, generate a minimal `compose.yml` with a comment block explaining how to add services.

- `compose.override.yml.example` — per-developer port/credential overrides; copy to `compose.override.yml` (gitignored):

```yaml
# Copy to compose.override.yml and customise
services:
  db:
    ports:
      - "5433:5432"   # use a different host port if 5432 is taken
```

- `.gitignore` — add `compose.override.yml`
- `.env.example` — add service connection strings (e.g. `DATABASE_URL=postgres://app:app@localhost:5432/app`)
- `README.md` — add "Start services: `docker compose up -d`"

Message: `chore: add Docker Compose for local services`

---

## Commit 11: API E2E tests (optional — default yes)

Only if API E2E was chosen. Not conditional on frontend — applies to all project shapes.

Run from project root:
```bash
npm init playwright@latest e2e -- --quiet
```

Select: TypeScript, no browser download yet (browsers added in commit 11b if needed).

### Files

`e2e/playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'http://localhost:3001',
  },
  webServer: {
    command: 'cargo run',
    url: 'http://localhost:3001',
    reuseExistingServer: !process.env.CI,
    env: {
      PORT: '3001',
      RUST_LOG: 'error',   // quiet during tests
    },
  },
});
```

Use port 3001 for tests to avoid colliding with the dev server on 3000.

`e2e/tests/api/health.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test('health check returns 200', async ({ request }) => {
  const response = await request.get('/health');
  expect(response.ok()).toBeTruthy();
});
```

Note: this uses Playwright's `request` fixture — no browser, pure HTTP. All API tests follow this pattern.

**Also generate infrastructure verification tests** — these verify the observability stack is wired correctly, not just that the business logic works. These are the first tests to write; if they fail, nothing else is worth checking.

`e2e/tests/api/observability.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

// Verify Prometheus metrics endpoint (only if metrics were chosen)
test('metrics endpoint is reachable and returns valid Prometheus format', async ({ request }) => {
  const response = await request.get('/metrics');
  expect(response.ok()).toBeTruthy();
  const contentType = response.headers()['content-type'];
  expect(contentType).toContain('text/plain');
  const body = await response.text();
  // Prometheus text format always starts with # HELP or # TYPE lines
  expect(body).toMatch(/^#\s+(HELP|TYPE)/m);
});

// Verify traces are being emitted to Jaeger (requires Docker services running)
// Jaeger's HTTP API is available at :16686
test('a request to /health produces a trace in Jaeger', async ({ request }) => {
  // Make a traceable request
  await request.get('/health');

  // Give the batch exporter a moment to flush
  await new Promise(resolve => setTimeout(resolve, 500));

  // Query Jaeger for services — our service name should appear
  const jaeger = await request.get('http://localhost:16686/api/services');
  expect(jaeger.ok()).toBeTruthy();
  const body = await jaeger.json();
  const serviceName = process.env.OTEL_SERVICE_NAME ?? 'unknown';
  expect(body.data).toContain(serviceName);
});
```

Omit the metrics test if Prometheus scrape endpoint was not chosen. Omit the Jaeger test if Docker was not chosen. Use comments in the file to note which test corresponds to which interview choice, so future contributors understand why each test exists.

If Docker services are present, add a note in `e2e/README.md`: "Run `docker compose up -d` before `cargo xtask e2e` or `check.sh`. The Jaeger trace verification test requires the Jaeger container to be running."

Update xtask `e2e` task to run `npx playwright test` from the `e2e/` directory.

Message: `test: add Playwright API E2E test scaffold`

---

## Commit 11b: Browser E2E tests (optional)

Only if browser E2E was chosen. Requires both frontend scaffold and API E2E. Adds on top of commit 11.

Install browsers:
```bash
cd e2e && npx playwright install chromium
```

Update `e2e/playwright.config.ts` to add a browser project alongside the API tests:

```typescript
projects: [
  {
    name: 'api',
    testMatch: 'tests/api/**/*.spec.ts',
    use: { /* no browser */ },
  },
  {
    name: 'chromium',
    testMatch: 'tests/ui/**/*.spec.ts',
    use: { ...devices['Desktop Chrome'] },
  },
],
```

New directory: `e2e/tests/ui/` — browser tests live here, separate from `e2e/tests/api/`.

Initial test `e2e/tests/ui/smoke.spec.ts`:
```typescript
import { test, expect } from '@playwright/test';

test('app loads', async ({ page }) => {
  await page.goto('/');
  await expect(page).not.toHaveTitle('');
});
```

Message: `test: add Playwright browser E2E tests`

---

## Commit 12: check.sh

Always included.

New file: `check.sh` — executable script at the project root that runs the full local test suite in the correct order:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Start services if Docker is configured
if [ -f compose.yml ]; then
  echo "Starting services..."
  docker compose up -d
fi

echo "Running unit and integration tests..."
cargo test

# Run E2E tests if present
if [ -d e2e ]; then
  echo "Running E2E tests..."
  npx --prefix e2e playwright test
fi

echo "All checks passed."
```

Make executable:
```bash
chmod +x check.sh
```

Notes:
- Omit the Docker block if Docker was not chosen
- Omit the E2E block if E2E tests were not chosen
- Do not add lint steps here — `check.sh` is for test execution, not code quality gates (those live in the justfile/xtask `check` recipe and CI)

Update build tooling to call `check.sh` from the `test-all` or `check` recipe rather than duplicating the logic.

Message: `chore: add check.sh for full local test suite`

---

## Commit 13: CI (optional)

Only if GitHub Actions was chosen.

New file: `.github/workflows/ci.yml`

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: Swatinem/rust-cache@v2
      - run: cargo fmt --check
      - run: cargo clippy -- -D warnings
      - run: ./check.sh
```

If Docker services are present, add a service container block to the CI job matching `compose.yml`. Example for PostgreSQL:

```yaml
    services:
      db:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: app
          POSTGRES_PASSWORD: app
          POSTGRES_DB: app
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
```

Message: `ci: add GitHub Actions workflow`

---

## Commit 14: Developer and agent guidance

Always last.

New file: `AGENTS.md` — written for both human developers and AI agents working on this project. Use the template below, filling in project-specific values from the interview.

```markdown
# Agent Configuration

## Engineering Rules

This project follows XP and Lean development practices. These are not optional.

**Test-Driven Development is non-negotiable.** Every production code change begins
with a failing test. No exceptions. The cycle is RED → GREEN → REFACTOR.

Load these skills when working on this project:
- `tdd` — for all code changes
- `rust` — for Rust-specific patterns and boundaries
- `testing` — for test structure and factory patterns
- `refactoring` — after tests pass, before committing

When starting significant new work, also load:
- `planning` — to break work into testable increments
- `code-review` — before presenting work as complete

## Project

**Purpose**: [one sentence from the interview]

**Shape**: [API-only | API + frontend]

## Architecture

```
src/
  lib.rs           domain logic — no framework imports
  main.rs          binary entry point
  router.rs        HTTP routes
  state.rs         application state
  errors.rs        domain error types
  telemetry.rs     OTel initialisation
  [db.rs]          database layer (if present)
  [metrics.rs]     Prometheus bridge (if present)
  [cli/]           CLI subcommands (if present)
```

**Rules that must not be broken:**
- `lib.rs` has zero Axum, SQLx, or other framework imports
- Business logic lives in plain synchronous functions, not async handlers
- Errors are plain enums deriving `Debug` and `PartialEq`
- No `util`, `helpers`, or `common` modules — organise by domain

## Development Workflow

```bash
# Start infrastructure
docker compose up -d          # starts DB, Jaeger, and any other services

# Run the server
cargo xtask dev               # hot-reload on file changes

# Run all checks (unit + integration + E2E)
./check.sh

# Individual steps
cargo test                    # unit and integration tests
cargo xtask e2e               # API E2E tests (requires services running)
cargo xtask lint              # clippy + fmt check
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | HTTP listen port |
| `RUST_LOG` | `info` | Log level filter (e.g. `debug`, `tower_http=trace`) |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | `http://localhost:4317` | OTel collector endpoint |
| `OTEL_SERVICE_NAME` | [project name] | Service name in traces |
[add DB vars, other vars from interview here]

Copy `.env.example` to `.env` for local development.

## Observability

- **Traces**: Jaeger UI at `http://localhost:16686` (start with `docker compose up -d`)
- **Logs**: stdout; controlled by `RUST_LOG`
[- **Metrics**: `http://localhost:3000/metrics` (Prometheus text format) — if chosen]

## Test Strategy

| Layer | Tool | What it covers |
|-------|------|----------------|
| Unit | `cargo test` (inline `#[cfg(test)]`) | Domain logic, plain functions |
| Integration | `cargo test` (`tests/` directory) | HTTP layer, routing, middleware |
| API E2E | Playwright (`e2e/tests/api/`) | Real binary over real socket, observability stack |
[| Browser E2E | Playwright + Chromium (`e2e/tests/ui/`) | UI flows — if frontend chosen |]

Run `./check.sh` to execute all layers in order.
```

Populate every section from the interview answers. Omit bracketed optional sections if not applicable. Do not leave placeholder text — every line should be specific to this project.

Message: `docs: add developer and agent guidance`

---

## Completion Announcement

After the final commit, output:

```
Project ready. Structure:

src/
  lib.rs         domain logic
  main.rs        server entry point
  router.rs      HTTP routes
  state.rs       application state
  errors.rs      domain error types
  [db.rs]        database layer (if chosen)

Development:
  [docker compose up -d]              start services (if Docker chosen)
  [just dev | cargo watch -x run]     start server with hot reload
  ./check.sh                          run full test suite
```

List the actual files generated. Do not list files that were not created.
