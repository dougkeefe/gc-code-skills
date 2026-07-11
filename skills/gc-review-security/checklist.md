# Protected B Security Checklist

Detailed patterns and guidance for ITSG-33 compliance review. This checklist is language-agnostic and applies to any technology stack.

> **Note:** All patterns below are indicative, not literal regex — read the surrounding code for context before flagging.

**Scope note:** Identification and authentication checks (ITSG-33 IA family — MFA, credential management, OIDC, session lifecycle) are covered by the sibling skill `gc-review-iam` and are intentionally not duplicated here.

---

## A. Broken Access Control [AC Family]

### What to Look For

**Authentication Checks:**
- Session validation at handler entry point
- Token verification (JWT, session cookie, etc.)
- Middleware/interceptor that enforces authentication

**Authorization Checks:**
- Role verification before sensitive operations
- Permission checks against user's assigned roles
- Guard clauses or decorators that enforce RBAC

**IDOR Vulnerabilities:**
- Resource ID in URL path or query parameters
- Database lookup by ID without ownership filter
- Missing check: "Does this resource belong to the requesting user?"

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| Handler without auth check at top | Unauthenticated access | Add session/token validation |
| `getById(params.id)` without user filter | IDOR - access other users' data | Add `AND userId = currentUser.id` |
| Admin endpoint without role check | Privilege escalation | Add `requireRole('ADMIN')` guard |
| Resource deletion without ownership check | Unauthorized deletion | Verify resource belongs to user |

### ITSG-33 Controls
- **AC-3:** Access Enforcement
- **AC-6:** Least Privilege
- **AC-17:** Remote Access

---

## B. Input Validation [SI Family]

### What to Look For

**Schema Validation:**
- Request body validated against defined schema
- Query parameters typed and validated
- Path parameters validated (e.g., UUID format)

**Injection Prevention:**
- Parameterized queries or ORM usage
- No string concatenation in queries
- Prepared statements for raw SQL

**Sanitization:**
- HTML encoding for output
- SQL escaping handled by framework
- Command injection prevention

**SSRF Prevention:**
- Outbound request URLs, hosts, or ports built from user input
- Missing allow-list for user-influenced request targets
- Redirect-following on user-supplied URLs

**Deserialization Safety:**
- Untrusted data passed to native deserializers
- YAML/XML loaders without safe mode

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| `query = "SELECT * FROM users WHERE id = " + id` | SQL injection | Use parameterized query |
| Request body used directly without validation | Malformed data, injection | Add schema validation |
| `exec(userInput)` or `eval(userInput)` | Command/code injection | Never execute user input |
| Query param used in file path | Path traversal | Validate and sanitize path |
| `fetch(req.body.url)` or `requests.get(user_url)` | SSRF - internal network access | Allow-list permitted hosts; block private IP ranges |
| `pickle.loads(data)`, `ObjectInputStream`, `yaml.load(input)` without `SafeLoader` | Insecure deserialization - remote code execution | Use safe formats (JSON) or safe loaders |

### ITSG-33 Controls
- **SI-10:** Information Input Validation
- **SI-11:** Error Handling

---

## C. PII Protection [Privacy Act]

### PII Field Types

| Category | Examples |
|----------|----------|
| Direct Identifiers | Full name, SIN, passport number, driver's license |
| Contact Information | Address, phone number, email address |
| Sensitive Personal | Date of birth, health information, financial data |
| Authentication | Passwords, security questions, biometric data |

### What to Look For

**Schema/Model Definitions:**
- PII fields marked with the project's metadata convention (`security.piiMetadataConvention` in `.gc-review/config.json`; e.g., `isPII`, `@Sensitive`)
- Encryption-at-rest configuration for PII columns
- Access controls on PII fields

**Logging:**
- Log statements that output user objects
- Error logs that include request bodies
- Debug logs with PII fields

**Data Exposure:**
- API responses returning full user objects
- Error messages revealing PII
- PII in URLs or query strings

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| `console.log(user)` or `logger.info(userData)` | PII in logs | Log only `userId` or masked data |
| PII field without project-convention metadata | Untracked sensitive data | Add the project's PII annotation (⚠️ Warning if no convention configured; ❌ Fail if `security.piiMetadataConvention` is set) |
| Full user object in API response | Over-exposure of PII | Return only required fields |
| SIN/birthdate in error message | PII leakage | Mask or omit PII in errors |

### Privacy Act Context

The *Privacy Act* (R.S.C. 1985, c. P-21) governs collection limitation, purpose specification, data minimization, and individual access rights (ss. 4–8, 12). It does **not** require code-level PII annotations — the metadata convention above is a project practice that supports those statutory obligations.

---

## D. Cryptography [SC Family]

### Approved vs. Prohibited Algorithms (ITSP.40.111 v5, effective 2026-05-29)

| Purpose | CCCS position | Prohibited / phase-out |
|---------|---------------|------------------------|
| Password storage (KDF) | CCCS-specified: PBKDF2 (NIST SP 800-132). Industry best practice (OWASP): Argon2id, bcrypt, scrypt — acceptable; flag as ⚠️ informational, not ❌ | MD5, SHA-1 |
| Hashing (integrity/signatures) | SHA-2 family (SHA-256/384/512), SHA-3 family | MD5; **SHA-1 must not be used with digital signature schemes** |
| Symmetric encryption | AES (128, 192, or 256-bit keys) | DES, 3DES, RC4 |
| Key establishment | ECDH with curves P-256/P-384/P-521 (P-224 phase-out by 2030); RSA/DH ≥ 2048-bit now, ≥ 3072-bit by end of 2030 | RSA/DH < 2048-bit |
| Post-quantum cryptography (PQC) | ML-KEM (512/768/1024) for key establishment; ML-DSA and SLH-DSA for signatures. Key-establishment implementations must support PQC by **end of 2026**; quantum-vulnerable RSA key establishment phased out by end of 2035 | New code introducing quantum-vulnerable-only key establishment → ⚠️ Warning |

### What to Look For

**TLS Configuration (ITSP.40.062 v3):**
- TLS 1.2 minimum, TLS 1.3 preferred
- Strong cipher suites
- Certificate validation enabled

**Cookie Security (OWASP best practice supporting ITSG-33 SC-23 Session Authenticity — not a GC configuration mandate):**
- `HttpOnly` flag set
- `Secure` flag set
- A `SameSite` attribute present: `Strict` or `Lax` (flag `SameSite=None` as ⚠️ unless justified)

**Secrets Management:**
- No hardcoded credentials
- Environment variables or secrets manager
- No secrets in source control: no committed `.env` files, no keys in git history (`git log --diff-filter=A -- '*.env' '*.pem' '*.key'`)

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| `md5(password)` or `sha1(password)` | Weak password hashing | Use PBKDF2 (CCCS-specified) or Argon2id/bcrypt/scrypt (OWASP) |
| Cookie without `Secure` flag | Transmitted over HTTP | Add `Secure: true` |
| Cookie without any `SameSite` attribute | CSRF exposure | Set `SameSite=Strict` or `SameSite=Lax` |
| `const apiKey = "sk-..."` in code | Hardcoded secret | Use environment variable |
| `.env` file committed or key files in git history | Leaked credentials persist in history | Remove, rotate secrets, purge history |
| TLS 1.0/1.1 configuration | Weak encryption | Require TLS 1.2+ |
| New RSA/ECDH key establishment with no PQC path | Quantum-vulnerable; misses ITSP.40.111 end-of-2026 PQC milestone | Plan ML-KEM support (hybrid acceptable) |

### ITSG-33 Controls
- **SC-8:** Transmission Confidentiality and Integrity
- **SC-12:** Cryptographic Key Establishment and Management
- **SC-13:** Cryptographic Protection
- **SC-23:** Session Authenticity
- **SC-28:** Protection of Information at Rest

---

## E. Audit Logging [AU Family]

### Required Log Fields

| Field | Description | Example |
|-------|-------------|---------|
| Timestamp | ISO 8601 format | `2024-01-15T10:30:00Z` |
| User ID | Authenticated user identifier | `user-abc-123` |
| Action | What operation was performed | `DELETE_RECORD` |
| Resource ID | Affected resource identifier | `record-xyz-789` |
| Outcome | Success or failure | `SUCCESS` / `FAILURE` |
| IP Address | Client IP (optional but recommended) | `192.168.1.100` |

### Security Events to Log

| Event Type | Examples |
|------------|----------|
| Authentication | Login success/failure, logout, session timeout |
| Authorization | Access denied, privilege escalation attempt |
| Data Access | View sensitive record, export data |
| Data Modification | Create, update, delete of sensitive records |
| Admin Actions | User creation, role changes, config changes |

### SIEM Forwarding (GC Event Logging Guidance, web configuration req. 4.3)

Logs must reach an approved GC centralized SIEM. Direct console/stdout logging is **not** automatically a failure: stdout-to-SIEM pipelines are standard in containerized deployments. Flag as ⚠️ Warning and verify forwarding — unless `security.auditLogService` is configured and bypassed, which is ❌ Fail.

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| Delete operation without audit log | Untraceable data changes | Add audit log call |
| `console.log` for security events with no SIEM-forwarded pipeline | Logs not collected by SIEM | Use centralized audit service, or confirm stdout forwarding to the SIEM (⚠️ pending verification) |
| Security events bypassing the configured `security.auditLogService` | Sanctioned audit trail incomplete | Route events through the configured service (❌) |
| Log missing user ID | Cannot attribute action | Include authenticated user ID |
| Failed login not logged | Cannot detect brute force | Log all auth attempts |

### ITSG-33 Controls
- **AU-2:** Audit Events
- **AU-3:** Content of Audit Records
- **AU-6:** Audit Review, Analysis, and Reporting
- **AU-12:** Audit Generation

---

## F. Web Configuration & Security Headers [SC Family / GC Web Configuration Requirements]

Source: GC *Web Sites and Services Management Configuration Requirements* (Date modified 2024-10-25). Requirement numbers below refer to that instrument.

### What to Look For

**Transport (reqs. 1.1–1.2, 1.4–1.6):**
- HTTPS-only; HTTP requests redirect to HTTPS (req. 1.1)
- HSTS enabled (req. 1.2)
- TLS 1.2+ per ITSP.40.062 §3.1 and ITSP.40.111, other algorithms disabled (req. 1.4); SSL v2/v3 and TLS 1.0/1.1 disabled (req. 1.5); RC4 and 3DES disabled (req. 1.6)

**Response Headers (req. 2.4 — mandated list):**
- `Content-Security-Policy`
- `Strict-Transport-Security` (HSTS)
- `X-Frame-Options`
- `X-Content-Type-Options: nosniff` — best-practice extra, not part of the mandated list (⚠️ if absent)

**Application (reqs. 2.2–2.3, 2.5):**
- Input validation/sanitization on all input (req. 2.2 — see section B)
- Output encoding on all output (req. 2.3)
- OWASP ASVS as the web application security verification baseline (req. 2.5)

**CSRF Protection [SC family]:**
- Anti-CSRF tokens or framework CSRF middleware on state-changing endpoints (cookie flags alone are insufficient)

**Disclosure (req. 4.5):**
- Published `security.txt` for public-facing sites

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| Server config listening on HTTP without redirect | Cleartext transmission | Redirect all HTTP to HTTPS (req. 1.1) |
| No `Strict-Transport-Security` header in production config | Downgrade attacks | Enable HSTS (req. 1.2) |
| Missing `Content-Security-Policy` or `X-Frame-Options` | XSS, clickjacking | Add mandated headers (req. 2.4) |
| Template escaping disabled / `dangerouslySetInnerHTML` with unsanitized data | XSS via missing output encoding | Encode all output (req. 2.3) |
| State-changing route without CSRF token/middleware | Cross-site request forgery | Add framework CSRF protection |
| No `security.txt` in a public GC site repo | Vulnerability reporting channel missing | Publish `security.txt` (req. 4.5) — ⚠️ |

### ITSG-33 Controls
- **SC-8:** Transmission Confidentiality and Integrity
- **SC-23:** Session Authenticity
- **AU-2:** Audit Events (SIEM forwarding, req. 4.3 — see section E)
