---
name: security-and-hardening
description: Security-first development practices. Use when handling user input, authentication, data storage, or external integrations. Guides the mindset and process — load the code-review skill for the detailed security checklist.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Edit
  - Write
  - AskUserQuestion
---

# Security and Hardening

STARTER_CHARACTER = 🔒

## Why This Matters

Security isn't a phase — it's a constraint on every line of code that touches user data, authentication, or external systems. The Lean principle of *building quality in* applies directly: security retrofitting is 10x harder than building it in from the start. Every "we'll add security later" is technical debt with compounding interest.

## When to Use

- Building anything that accepts user input
- Implementing authentication or authorization
- Storing or transmitting sensitive data
- Integrating with external APIs or services
- Adding file uploads, webhooks, or callbacks
- Handling payment or PII data

## The Three-Tier Boundary System

### Always Do (No Exceptions)

- **Validate all external input** at the system boundary (see also `api-and-interface-design`)
- **Parameterize all database queries** — never concatenate user input into SQL/queries
- **Encode output** to prevent XSS (use framework auto-escaping, don't bypass it)
- **Use HTTPS** for all external communication
- **Hash passwords** with bcrypt/scrypt/argon2 (never store plaintext)
- **Set security headers** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **Use httpOnly, secure, sameSite cookies** for sessions
- **Audit dependencies** for known vulnerabilities before every release

### Ask First (Requires Human Approval)

- Adding new authentication flows or changing auth logic
- Storing new categories of sensitive data (PII, payment info)
- Adding new external service integrations
- Changing CORS configuration
- Adding file upload handlers
- Modifying rate limiting or throttling
- Granting elevated permissions or roles

### Never Do

- **Never commit secrets** to version control (API keys, passwords, tokens)
- **Never log sensitive data** (passwords, tokens, full credit card numbers)
- **Never trust client-side validation** as a security boundary
- **Never disable security headers** for convenience
- **Never use `eval()` or equivalent** with user-provided data
- **Never store sessions in client-accessible storage** (e.g., localStorage for auth tokens)
- **Never expose stack traces** or internal error details to users

## OWASP Top 10 Prevention

These are the most common web application vulnerabilities. The principles apply beyond web:

| # | Vulnerability | Prevention |
|---|---|---|
| 1 | **Injection** (SQL, NoSQL, OS Command) | Parameterize all queries. Never concatenate user input into commands. |
| 2 | **Broken Authentication** | Hash passwords (salt rounds >= 12). Session tokens: httpOnly, secure, sameSite. Rate limit auth endpoints. |
| 3 | **Cross-Site Scripting (XSS)** | Use framework auto-escaping. Never render raw user input as HTML. Sanitize if you must. |
| 4 | **Broken Access Control** | Check authorization on every endpoint, not just authentication. Users can only access their own resources. |
| 5 | **Security Misconfiguration** | Set security headers. Restrict CORS to known origins. Don't expose debug info in production. |
| 6 | **Sensitive Data Exposure** | Never return sensitive fields in API responses. Use environment variables for secrets. Encrypt PII at rest. |
| 7 | **Missing Rate Limiting** | Rate limit authentication endpoints (strict) and general API access (standard). |
| 8 | **Insecure Dependencies** | Audit regularly. Patch critical/high vulnerabilities promptly. See triage tree below. |
| 9 | **Insufficient Logging** | Log security events (failed auth, access denied). Don't log sensitive data. |
| 10 | **SSRF / Request Forgery** | Validate and restrict URLs in server-side requests. Don't let user input control fetch targets without allowlisting. |

## Triaging Dependency Audit Results

Not all audit findings require immediate action:

```
Dependency audit reports a vulnerability
├── Severity: critical or high
│   ├── Is the vulnerable code reachable in your app?
│   │   ├── YES → Fix immediately (update, patch, or replace)
│   │   └── NO (dev-only dep, unused code path) → Fix soon, not a blocker
│   └── Is a fix available?
│       ├── YES → Update to the patched version
│       └── NO → Workaround, replace, or allowlist with review date
├── Severity: moderate
│   ├── Reachable in production? → Fix in the next release cycle
│   └── Dev-only? → Fix when convenient, track in backlog
└── Severity: low
    └── Track and fix during regular dependency updates
```

**Key questions:**
- Is the vulnerable function actually called in your code path?
- Is the dependency a runtime dependency or dev-only?
- Is the vulnerability exploitable given your deployment context?

When you defer a fix, document the reason and set a review date.

## Secrets Management

```
.env files:
  ├── .env.example  → Committed (template with placeholder values)
  ├── .env          → NOT committed (contains real secrets)
  └── .env.local    → NOT committed (local overrides)

.gitignore must include:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**Always check before committing** — look for accidentally staged passwords, API keys, or tokens in the diff.

## File Upload Safety

When accepting file uploads:
- Restrict allowed MIME types explicitly
- Enforce maximum file size
- Don't trust the file extension — check magic bytes for critical applications
- Store uploads outside the web root
- Generate new filenames (don't use user-provided filenames in paths)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is an internal tool, security doesn't matter" | Internal tools get compromised. Attackers target the weakest link. |
| "We'll add security later" | Security retrofitting is 10x harder than building it in. Add it now. |
| "No one would try to exploit this" | Automated scanners will find it. Security by obscurity is not security. |
| "The framework handles security" | Frameworks provide tools, not guarantees. You still need to use them correctly. |
| "It's just a prototype" | Prototypes become production. Security habits from day one. |

## Red Flags

- User input passed directly to database queries, shell commands, or HTML rendering
- Secrets in source code or commit history
- API endpoints without authentication or authorization checks
- Missing CORS configuration or wildcard (`*`) origins
- No rate limiting on authentication endpoints
- Stack traces or internal errors exposed to users
- Dependencies with known critical vulnerabilities

## See Also

For detailed pre-commit security checks and review checklists, see the `code-review` skill's [security checklist](../code-review/references/security-checklist.md).

## Verification

After implementing security-relevant code:

- [ ] No secrets in source code or git history
- [ ] All user input validated at system boundaries
- [ ] Authentication and authorization checked on every protected endpoint
- [ ] Security headers present in responses
- [ ] Error responses don't expose internal details
- [ ] Rate limiting active on auth endpoints
- [ ] Dependency audit shows no unaddressed critical or high vulnerabilities
- [ ] File uploads restricted by type and size (if applicable)
