---
name: rust
description: Rust patterns for web services and systems code. Use when writing any Rust code — covers type-driven correctness, testability boundaries, project structure, error handling, and module organisation.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Rust

STARTER_CHARACTER = 🦀

## Core Principle: Functional Core, Imperative Shell

Every Rust service separates into two layers:

- **Functional core** — plain functions that take ordinary types, return ordinary types, carry no async or framework dependencies, and are fully unit-testable.
- **Imperative shell** — thin adapters that hold I/O, async runtimes, and framework extractors. They extract inputs, call the core, and map results to responses.

This separation is the prerequisite for TDD in Rust, not an optional style preference.

**Related skills:**
- **For TDD workflow**, load the `tdd` skill
- **For testing patterns**, load the `testing` skill

---

## Type-Driven Correctness

Rust's type system can enforce correctness at compile time that other languages defer to runtime or tests. Use it. When you reach for a hard-coded value, ask: "Is this a property of the data or a fixed protocol constant?" If it describes variable data, derive it from the data.

### Derive discriminators from data

When a function must tell an external API "this data is type X," compute X from the data via exhaustive `match`. Never hard-code a discriminator that describes variable input.

❌ **WRONG — hard-coded discriminator:**
```rust
fn write_geojson_fgb(shapes: &[Shape], writer: &mut FgbWriter) -> Result<()> {
    // BUG: assumes all shapes are Polygon, but data may contain MultiPolygon
    let mut fgb = FgbWriter::create("shapes", GeometryType::Polygon)?;
    for shape in shapes {
        fgb.add_feature(shape.to_geojson())?;
    }
    fgb.write(writer)
}
```

✅ **CORRECT — discriminator derived from data:**
```rust
fn geometry_type_for(shapes: &[Shape]) -> Result<GeometryType, ConversionError> {
    let types: HashSet<_> = shapes.iter().map(|s| match s {
        Shape::Polygon(_) => GeometryType::Polygon,
        Shape::MultiPolygon(_) => GeometryType::MultiPolygon,
        Shape::Point(_) => GeometryType::Point,
        // exhaustive — compiler forces update when Shape gains a variant
    }).collect();

    match types.len() {
        0 => Err(ConversionError::NoShapes),
        1 => Ok(types.into_iter().next().unwrap()),
        _ => Err(ConversionError::MixedGeometryTypes),
    }
}

fn write_geojson_fgb(shapes: &[Shape], writer: &mut FgbWriter) -> Result<()> {
    let geo_type = geometry_type_for(shapes)?;
    let mut fgb = FgbWriter::create("shapes", geo_type)?;
    // ...
}
```

The exhaustive `match` means the compiler will force you to handle new `Shape` variants — a hard-coded value silently becomes wrong.

### Newtypes as proof carriers

When a value must come from a specific derivation step, wrap it in a newtype so call sites cannot fabricate it. Use this when the same broad enum appears at multiple call sites and a mismatch has already caused bugs.

```rust
/// Can only be constructed via `from_shapes`, guaranteeing the geometry type
/// was derived from actual shape data.
pub struct DerivedGeometryType(GeometryType);

impl DerivedGeometryType {
    pub fn from_shapes(shapes: &[Shape]) -> Result<Self, ConversionError> {
        let geo_type = geometry_type_for(shapes)?;
        Ok(DerivedGeometryType(geo_type))
    }

    pub fn get(&self) -> GeometryType {
        self.0
    }
}

// Now the FGB writer requires proof that the type was derived:
fn write_fgb(shapes: &[Shape], geo_type: DerivedGeometryType, writer: &mut FgbWriter) -> Result<()> {
    let mut fgb = FgbWriter::create("shapes", geo_type.get())?;
    // ...
}
```

The newtype makes it impossible to pass an arbitrary `GeometryType` — callers must go through `from_shapes`, which derives the value from real data.

### Make invalid states unrepresentable

Prefer types that cannot express wrong combinations over runtime validation. This applies to domain types where invariants matter.

❌ **WRONG — callers must remember to check all fields:**
```rust
struct FileSet {
    shp: Option<Vec<u8>>,
    shx: Option<Vec<u8>>,
    dbf: Option<Vec<u8>>,
}
// Every function using FileSet must check all three Options
```

✅ **CORRECT — the type guarantees the invariant:**
```rust
struct ValidatedFileSet {
    shp: Vec<u8>,
    shx: Vec<u8>,
    dbf: Vec<u8>,
}

impl ValidatedFileSet {
    pub fn new(shp: Vec<u8>, shx: Vec<u8>, dbf: Vec<u8>) -> Result<Self, ValidationError> {
        if shp.is_empty() || shx.is_empty() || dbf.is_empty() {
            return Err(ValidationError::MissingFiles);
        }
        Ok(Self { shp, shx, dbf })
    }
}
// Functions taking ValidatedFileSet know all files are present and non-empty
```

### When hard-coding IS acceptable

Not every constant needs derivation. Hard-code when:

- **Test helpers** where you control the data and the value is part of the test fixture
- **Fixed protocol constants** that are requirements of the protocol, not properties of variable input (e.g., a magic number, a fixed header version)

The distinction: "this is always X because the spec says so" → hard-code. "This is X because the data I'm looking at happens to be X" → derive.

---

## Framework Boundaries

Framework types — Axum extractors (`Multipart`, `Path`, `Query`, `Json`), SQLx executors, tower middleware, `clap::Args` — cannot be constructed in unit tests. Any function that takes one as a parameter is an adapter, not logic.

**Rule**: Before implementing a handler or any framework-bound function, extract its logic into a plain function first. Write tests for that function. The handler is written last and contains no logic of its own.

❌ **WRONG — logic embedded in the handler:**
```rust
async fn upload(
    State(state): State<AppState>,
    mut multipart: Multipart,
) -> Html<String> {
    let mut files = HashMap::new();
    while let Ok(Some(field)) = multipart.next_field().await {
        // extract fields...
    }
    // validation, session storage, response generation all in here
    // none of this can be unit tested
}
```

✅ **CORRECT — thin handler delegates to a plain function:**
```rust
// Plain function: takes ordinary types, fully unit-testable
fn process_upload(files: HashMap<String, (String, Vec<u8>)>, state: &AppState) -> Html<String> {
    match validate_filenames(&filenames) {
        Ok(base_name) => { /* store session, return success */ }
        Err(e) => { /* return error response */ }
    }
}

// Thin handler: only extracts inputs from the request, then delegates
async fn upload(State(state): State<AppState>, mut multipart: Multipart) -> Html<String> {
    let mut files = HashMap::new();
    while let Ok(Some(field)) = multipart.next_field().await {
        // extraction only — no branching, no business logic
    }
    process_upload(files, &state)
}
```

This pattern applies to every framework boundary:

| Framework type | Thin adapter | Plain function receives |
|----------------|-------------|------------------------|
| Axum `Multipart` | handler | `HashMap<String, (String, Vec<u8>)>` |
| Axum `Json<T>` | handler | `T` directly |
| SQLx `PgPool` | repository fn | domain types |
| `clap::Args` | CLI entrypoint | config struct |
| Message queue message | consumer fn | message payload |

**TDD order when writing a handler:**
1. Write tests for the plain function — **Red**
2. Implement the plain function — **Green**
3. Write the thin handler that calls it (the adapter needs no unit tests)

---

## Project Structure

A mixed crate (`src/lib.rs` + `src/main.rs`) is the standard shape for a Rust service. The library crate holds domain logic; the binary crate holds the web or CLI layer.

```
src/
  lib.rs            domain: pub modules, no framework dependencies
  <domain>.rs       domain logic with inline unit tests

  main.rs           entry point: declares binary-only modules
  state.rs          application state (Arc<Mutex<...>>)
  router.rs         builds the Router from state
  handlers/
    mod.rs          declares modules, re-exports handler fns
    <resource>.rs   thin adapter + plain process_* fn + tests
```

**Rules:**
- `lib.rs` has no dependency on Axum, SQLx, or any I/O framework
- Binary modules import from `lib.rs` via the crate name (`use my_crate::domain::...`)
- Domain types live in `lib.rs`; HTTP request/response shapes live in handlers
- No `util`, `common`, or `helpers` modules — organise by domain

---

## Error Handling

Use different strategies at different layers:

| Layer | Strategy |
|-------|----------|
| Domain (`lib.rs`) | Plain enums with `#[derive(Debug, PartialEq)]` |
| Handlers | Convert domain errors to HTTP responses explicitly |
| Tests | `unwrap()` / `expect()` is fine |

Domain errors as plain enums lets tests assert on them directly without pattern-matching on strings:

```rust
#[derive(Debug, PartialEq)]
pub enum ValidationError {
    MissingFiles(Vec<String>),
    MismatchedBaseNames,
}

// In tests:
assert_eq!(validate(&input), Err(ValidationError::MismatchedBaseNames));
```

Handlers map domain errors to responses — they do not bubble `?` to a generic error handler.

---

## Testing

### Inline test modules

Unit tests live in the same file as the code, inside `#[cfg(test)]`. This is idiomatic Rust:

```rust
pub fn validate(input: &[&str]) -> Result<String, ValidationError> {
    // implementation
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn rejects_empty_input() {
        let result = validate(&[]);
        assert!(matches!(result, Err(ValidationError::MissingFiles(_))));
    }
}
```

Do not create separate `tests/` files for unit tests — those are for integration tests against the public API of a library crate.

### Test helpers

Avoid duplicating setup. Define helpers inside the test module:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    fn make_files(entries: &[(&str, &str)]) -> HashMap<String, (String, Vec<u8>)> {
        entries.iter()
            .map(|(field, name)| (field.to_string(), (name.to_string(), vec![])))
            .collect()
    }
}
```

### What belongs at each test level

| Behaviour | Test level |
|-----------|------------|
| Domain validation logic | Unit — inline in `lib.rs` module |
| Handler logic (via plain function) | Unit — inline in handler file |
| Framework wiring (routes respond correctly) | E2E |
| HTML content users see | E2E |
| Multipart parsing, HTTP headers | E2E |

---

## External Crates

Prefer external crates over reimplementing established domain knowledge — binary format parsers, protocol encoders, cryptographic primitives, compression, and similar well-specified concerns are better served by a maintained library than by a hand-rolled implementation.

**Reach for a crate when:**
- The domain is well-specified (binary formats, protocols, standards)
- The crate handles edge cases, spec compliance, and error conditions that would take significant effort to replicate correctly
- The feature is not a core product differentiator

**Implement yourself instead when:**
- The feature *is* a core product differentiator — the thing that makes this product unique
- The crate's abstraction leaks heavily into your domain model and a small local implementation is easier to reason about and maintain

**Before adding a crate, verify:**
- It is actively maintained (recent releases, issues being responded to)
- It has meaningful usage (download count, number of dependents)
- Its API fits your use case (e.g., accepts `Read + Seek` for in-memory buffers, not file paths only)

**When a crate looks useful but is unmaintained or has very low usage, do not add it silently — ask the user.** State what you found (last release date, approximate download count, notable open issues) and offer the trade-off between using it and writing a minimal local implementation instead.

---

## Anti-Patterns

- ❌ **Business logic in async handler functions** — untestable; extract to plain functions first
- ❌ **Domain types importing Axum or SQLx** — `lib.rs` must have no framework dependencies
- ❌ **Writing the handler before the plain function exists** — the plain function must be tested first
- ❌ **Relying solely on E2E tests for business logic coverage** — slow feedback, hides bugs
- ❌ **`unwrap()` in production code** — use `?` or explicit error handling
- ❌ **`util` or `helpers` modules** — organise by domain, not by technical role
- ❌ **Hard-coding an enum variant that describes variable data** — derive discriminators from the data via exhaustive match
- ❌ **Passing a broad enum directly when the value must come from a derivation step** — consider a newtype to carry proof of derivation

---

## Summary Checklist

When generating Rust code, verify:

- [ ] Business logic is in plain synchronous functions, not in async handlers
- [ ] Plain functions have unit tests written before implementation
- [ ] `lib.rs` has no Axum, SQLx, or other framework imports
- [ ] Handler functions contain only: input extraction + call to plain function
- [ ] Domain error types derive `Debug` and `PartialEq`
- [ ] Module structure follows domain boundaries, not technical layers
- [ ] No `unwrap()` in production code paths
- [ ] `#[cfg(test)]` modules are inline, not in separate test files
- [ ] Enum variants passed to external APIs are derived from data, not hard-coded
- [ ] Types that carry proof of derivation (newtypes) are used where a mismatch has caused bugs
