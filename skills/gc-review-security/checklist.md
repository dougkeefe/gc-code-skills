# Protected B Security Checklist

Detailed patterns and guidance for ITSG-33 compliance review. This checklist is language-agnostic and applies to any technology stack.

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

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| `query = "SELECT * FROM users WHERE id = " + id` | SQL injection | Use parameterized query |
| Request body used directly without validation | Malformed data, injection | Add schema validation |
| `exec(userInput)` or `eval(userInput)` | Command/code injection | Never execute user input |
| Query param used in file path | Path traversal | Validate and sanitize path |

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
- PII fields marked with metadata (`isPII`, `@Sensitive`, etc.)
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
| PII field without `isPII` metadata | Untracked sensitive data | Add `isPII: true` annotation |
| Full user object in API response | Over-exposure of PII | Return only required fields |
| SIN/birthdate in error message | PII leakage | Mask or omit PII in errors |

### Privacy Act Requirements
- Collection limitation
- Purpose specification
- Data minimization
- Individual access rights

---

## D. Cryptography [SC Family]

### Approved vs. Prohibited Algorithms

| Purpose | Approved | Prohibited |
|---------|----------|------------|
| Hashing (passwords) | bcrypt, scrypt, Argon2 | MD5, SHA-1 |
| Hashing (integrity) | SHA-256, SHA-384, SHA-512 | MD5, SHA-1 |
| Symmetric encryption | AES-256 | DES, 3DES, RC4 |
| Key exchange | ECDH, DH (2048+ bit) | RSA < 2048 bit |

### What to Look For

**TLS Configuration:**
- TLS 1.2 minimum, TLS 1.3 preferred
- Strong cipher suites
- Certificate validation enabled

**Cookie Security:**
- `HttpOnly` flag set
- `Secure` flag set
- `SameSite: Strict` or `SameSite: Lax`

**Secrets Management:**
- No hardcoded credentials
- Environment variables or secrets manager
- No secrets in source control

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| `md5(password)` or `sha1(password)` | Weak password hashing | Use bcrypt/Argon2 |
| Cookie without `Secure` flag | Transmitted over HTTP | Add `Secure: true` |
| `const apiKey = "sk-..."` in code | Hardcoded secret | Use environment variable |
| TLS 1.0/1.1 configuration | Weak encryption | Require TLS 1.2+ |

### ITSG-33 Controls
- **SC-8:** Transmission Confidentiality
- **SC-12:** Cryptographic Key Management
- **SC-13:** Cryptographic Protection
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

### Common Patterns to Flag

| Pattern | Risk | Fix |
|---------|------|-----|
| Delete operation without audit log | Untraceable data changes | Add audit log call |
| `console.log` for security events | Logs not collected by SIEM | Use centralized audit service |
| Log missing user ID | Cannot attribute action | Include authenticated user ID |
| Failed login not logged | Cannot detect brute force | Log all auth attempts |

### ITSG-33 Controls
- **AU-2:** Audit Events
- **AU-3:** Content of Audit Records
- **AU-6:** Audit Review, Analysis, and Reporting
- **AU-12:** Audit Generation
