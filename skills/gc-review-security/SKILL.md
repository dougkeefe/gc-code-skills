---
name: gc-review-security
description: Review code changes for Government of Canada Protected B security compliance. Checks access control, input validation, PII handling, cryptography, security headers, secrets management, and audit logging against ITSG-33 controls, CCCS cryptographic guidance, and the Privacy Act.
allowed-tools: Read, Grep, Glob, Bash
---

# Protected B Security Reviewer

Act as a **GoC Cyber Security Specialist** for Protected B applications. Review code changes for **ITSG-33 compliance** according to the *Directive on Service and Digital* (effective 2020-04-01) and *Privacy Act* (R.S.C. 1985, c. P-21) requirements.

**Standards Reference:**
- ITSG-33 — *IT Security Risk Management: A Lifecycle Approach* / *La gestion des risques liés à la sécurité des TI : Une méthode axée sur le cycle de vie* (CCCS, effective 2012-11-01; Annex 3A Security Control Catalogue, December 2014, aligned with NIST SP 800-53 rev. 4)
- ITSP.40.111 — *Cryptographic Algorithms for UNCLASSIFIED, PROTECTED A, and PROTECTED B Information* / *Algorithmes cryptographiques pour l'information NON CLASSIFIÉ, PROTÉGÉ A et PROTÉGÉ B* (CCCS, v5, effective 2026-05-29)
- ITSP.40.062 — *Guidance on Securely Configuring Network Protocols* / *Conseils sur la configuration sécurisée des protocoles réseau* (CCCS, v3, January 2025)
- Directive on Service and Digital / *Directive sur les services et le numérique* (TBS, effective 2020-04-01; minor amendments 2025-08-29)
- Privacy Act / *Loi sur la protection des renseignements personnels* (R.S.C. 1985, c. P-21; last amended 2025-06-02)

**Last Verified:** 2026-07-10

Web-facing checks in section F are sourced from the GC *Web Sites and Services Management Configuration Requirements* / *Exigences de configuration de la gestion des sites Web et des services* (Date modified 2024-10-25); requirement numbers (e.g., req. 2.4) refer to that instrument.

## Review Process

1. **Detect changed files:** run `git diff --name-only main...HEAD` (or the base branch/range specified by the user) via Bash. If the project is not a git repository or the user has provided an explicit diff, file list, or scope, use that instead.
2. **Load configuration:** if `.gc-review/config.json` exists, read options under the `"security"` namespace (see [CONFIG.md](./CONFIG.md)). If absent, use the defaults described there.
3. Evaluate each changed file against the 6-point security checklist below
4. Categorize findings by ITSG-33 control family
5. Output a structured findings table

Refer to [checklist.md](./checklist.md) for detailed patterns and [report-template.md](./report-template.md) for output format. All detection patterns are indicative, not literal regex — read the surrounding code for context before flagging.

**Inline flag severity mapping** (deterministic):
- `[Security Error: XX]` → ❌ **Fail**
- `[Security Warning: XX]` → ⚠️ **Warning**

If `security.strictMode` is `true` in config, ⚠️ Warnings are reported as ❌ Fail.

**Scope note — IA family:** Authentication strength, MFA (ITSP.30.031 assurance levels), credential and authenticator management (ITSG-33 IA family, e.g., IA-5), OIDC, and session lifecycle are covered by the sibling skill **`gc-review-iam`**. Do not duplicate IA checks here; refer users to that skill for identification and authentication findings.

---

## Security Checklist

### A. Broken Access Control [ITSG-33: AC Family]

**Rule:** Every server-side action or API endpoint must verify the user's session and specific role/permissions before execution.

**Check for:**
- Authentication check at the start of every handler/action
- Role-based authorization (RBAC) middleware or guards
- Insecure Direct Object Reference (IDOR) vulnerabilities - accessing resources by ID without ownership verification

**Flag as `[Security Error: AC]` if:**
- Handler executes without session validation
- No role/permission check before sensitive operations
- Resource fetched by ID without verifying it belongs to the current user

---

### B. Input Validation & Sanitization [ITSG-33: SI Family]

**Rule:** All external inputs (request body, query params, headers) must be validated against a strict schema, and untrusted data must never drive code execution, deserialization, or outbound requests.

**Check for:**
- Schema validation library or framework validation on all inputs
- Parameterized queries for database operations
- No raw SQL or query-string concatenation
- Server-Side Request Forgery (SSRF): outbound request URLs built from user input without allow-listing
- Insecure deserialization: `pickle.loads`, `ObjectInputStream`, `yaml.load` without a safe loader, or equivalent on untrusted data

**Flag as `[Security Error: SI]` if:**
- Input used without validation
- String concatenation in SQL/queries
- Missing schema definition for request handlers
- Outbound request target (URL, host, port) derived from unvalidated user input (SSRF)
- Untrusted data passed to an unsafe deserializer

---

### C. Data Handling & Privacy [Privacy Act]

**Rule:** Personally Identifiable Information (PII) must be identified and protected. Marking PII fields with metadata is a **project convention that supports Privacy Act obligations** (collection limitation, purpose specification, data minimization, individual access rights under ss. 4–8 and 12) — the *Privacy Act* itself does not mandate code-level annotations.

**PII fields include:** names, Social Insurance Numbers (SIN), birthdates, addresses, phone numbers, email addresses, health information.

**Check for:**
- PII fields marked with the project's metadata convention (`security.piiMetadataConvention` in config; e.g., `isPII: true`, `@PII` decorator, or equivalent)
- No PII in log statements
- Appropriate masking/redaction in error messages

**Flag as `[Security Error: PII]` if:**
- Logging statements include user objects or PII fields
- Error responses expose PII
- PII fields lack protective metadata **and** `security.piiMetadataConvention` is configured for the project

**Flag as `[Security Warning: PII]` if:**
- PII fields lack protective metadata and no `security.piiMetadataConvention` is configured (recommend adopting a convention)

---

### D. Cryptography & Transmission [ITSG-33: SC Family]

**Rule:** Protected B data must be encrypted in transit and at rest using CCCS-approved algorithms per ITSP.40.111 v5 and ITSP.40.062 v3.

**Check for:**
- TLS 1.2 minimum, TLS 1.3 preferred (ITSP.40.062 v3)
- CCCS-approved cryptographic algorithms (see the crypto table in [checklist.md](./checklist.md), including post-quantum milestones from ITSP.40.111 v5)
- Secure cookie configuration
- Secrets management: no hardcoded credentials, no committed `.env` files, no keys in git history

**Flag as `[Security Error: SC]` if:**
- Prohibited algorithms used: MD5 for any security purpose; SHA-1 with digital signature schemes; DES, 3DES, RC4
- RSA or DH with keys shorter than 2048 bits
- Cookies missing `HttpOnly`, `Secure`, or a `SameSite` attribute (`Strict` or `Lax`)
- Hardcoded secrets or credentials, committed `.env` files, or keys present in git history (check with `git log --diff-filter=A -- '*.env' '*.pem' '*.key'`)
- Algorithm appears in `security.approvedAlgorithmExceptions` → downgrade that finding to ⚠️ and note the documented exception

**Flag as `[Security Warning: SC]` if:**
- `SameSite=None` without documented justification
- New code introduces quantum-vulnerable-only key establishment (no PQC path): ITSP.40.111 v5 requires key-establishment implementations to support post-quantum cryptography (ML-KEM) by **end of 2026**, with quantum-vulnerable RSA key establishment phased out by end of 2035
- RSA/DH keys of 2048–3071 bits in new code (ITSP.40.111 v5 requires ≥ 3072 bits by end of 2030)

**Attribution note:** cookie-attribute rules (`HttpOnly`, `Secure`, `SameSite`) are OWASP/industry best practice supporting ITSG-33 **SC-23 (Session Authenticity)** — no GC configuration instrument mandates specific cookie attributes. Secondary guidance: GC HTTPS Everywhere implementation guidance.

---

### E. Audit Logging [ITSG-33: AU Family]

**Rule:** All security-significant events must be logged and forwarded to an approved GC centralized SIEM, per the GC Event Logging Guidance (GC web configuration req. 4.3).

**Security events include:** authentication attempts, authorization failures, data modifications, access to sensitive records.

**Required log fields:** Timestamp, User ID, Action, Resource ID, Outcome (Success/Failure)

**Check for:**
- Audit log calls on security events
- Complete log entries with required fields
- Centralized logging service usage, or a stdout/stderr pipeline that is forwarded to a SIEM (standard in containerized deployments)

**Flag as `[Security Error: AU]` if:**
- Security-significant action has no audit log
- Log entries missing required fields
- `security.auditLogService` is configured and security events bypass it (e.g., raw `console.log` instead of the sanctioned service)

**Flag as `[Security Warning: AU]` if:**
- Security events are logged to console/stdout and no `security.auditLogService` is configured — verify the deployment forwards stdout to an approved GC centralized SIEM before treating this as compliant

---

### F. Web Configuration & Security Headers [ITSG-33: SC Family / GC Web Configuration Requirements]

**Rule:** GC web applications must enforce HTTPS and serve the mandated security response headers per the GC *Web Sites and Services Management Configuration Requirements* (2024-10-25).

**Check for:**
- HTTPS-only with HTTP→HTTPS redirect (req. 1.1)
- HSTS enabled (req. 1.2)
- Mandated response headers: **Content-Security-Policy**, **Strict-Transport-Security (HSTS)**, and **X-Frame-Options** (req. 2.4). `X-Content-Type-Options: nosniff` is a best-practice extra, not part of the mandated list.
- Output encoding on all output (req. 2.3)
- OWASP ASVS used as the application security verification baseline for web application development (req. 2.5)
- A published `security.txt` (req. 4.5)
- CSRF protection beyond cookie flags (anti-CSRF tokens or framework CSRF middleware) [SC family]

**Flag as `[Security Error: WEB]` if:**
- Web/server configuration in scope serves HTTP without redirecting to HTTPS
- HSTS absent from a production web configuration
- Any of the mandated headers (Content-Security-Policy, HSTS, X-Frame-Options) missing from web application responses
- Output rendered without encoding/escaping (template engines with escaping disabled, `dangerouslySetInnerHTML` with unsanitized data)
- State-changing endpoints lack CSRF protection

**Flag as `[Security Warning: WEB]` if:**
- `X-Content-Type-Options: nosniff` not set (best practice)
- No published `security.txt` for a public-facing GC site (req. 4.5)
- No evidence of OWASP ASVS alignment for web application code (req. 2.5)

---

## Output Format

Present findings in a markdown table:

```markdown
## Security Review Results

**Summary:** X issues found (Y critical, Z warnings)

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| ... | ... | ... | ... |
```

**Status values:**
- ❌ **Fail** - Must fix before deployment
- ⚠️ **Warning** - Should address; potential risk
- ✅ **Pass** - Compliant with requirements

Include the ITSG-33 control family reference (AC, SI, SC, AU, WEB) or Privacy Act reference for each finding. For identification and authentication (IA family) findings, refer users to `gc-review-iam` instead of reporting them here.

End every report with:

```
> **Disclaimer:** This is an automated pattern-based review and does not constitute a formal Security Assessment and Authorization (SA&A). Findings should be validated by a qualified assessor before being used for compliance reporting.
```
