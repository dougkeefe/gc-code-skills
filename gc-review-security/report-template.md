# Security Review Report Template

Use this format when presenting security review findings.

---

## Report Structure

```markdown
## Security Review Results

**Policy Basis:** ITSG-33 (Security Controls); Directive on Service and Digital; Privacy Act
**Scope:** [Files/components reviewed]
**Date:** [Review date]

### Summary

- **Total Issues:** X
- **Critical (Must Fix):** Y
- **Warnings (Should Fix):** Z
- **Passed Checks:** N

### Findings

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| [Fail] | `path/to/file:line` | [AC] Missing RBAC check; endpoint accessible without authorization | Add role guard (e.g., `requireRole('ADMIN')`) at handler entry |
| [Fail] | `path/to/file:line` | [PII] Logging entire user object containing PII | Log only `userId`; mask or omit PII fields |
| [Warning] | `path/to/file:line` | [SI] Query parameter `id` not validated | Validate input format (e.g., UUID) before use |
| [Warning] | `path/to/file:line` | [AU] Data modification without audit log | Add audit log call with required fields |
| [Pass] | `path/to/file` | [SC] Encryption-at-rest correctly configured | None |

### Detailed Findings

#### 1. [Issue Title] - [Severity]

**Location:** `path/to/file:line`
**Control:** ITSG-33 [Control Family]-[Number] or Privacy Act
**Issue:** [Detailed description of the vulnerability]
**Risk:** [What could happen if exploited]
**Recommendation:** [Specific fix guidance]
```

---

## Status Definitions

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| **[Fail]** | Security vulnerability or compliance violation | Must fix before deployment to production |
| **[Warning]** | Potential risk or best practice deviation | Should address; review and remediate |
| **[Pass]** | Compliant with security requirements | No action required |

---

## Control Family References

When flagging issues, include the relevant ITSG-33 control family:

| Code | Family | Typical Issues |
|------|--------|----------------|
| **AC** | Access Control | Missing auth, missing RBAC, IDOR |
| **AU** | Audit and Accountability | Missing logs, incomplete log entries |
| **IA** | Identification and Authentication | Weak auth, session issues |
| **SC** | System and Communications Protection | Weak crypto, insecure transmission |
| **SI** | System and Information Integrity | Input validation, injection |
| **PII** | Privacy Act (not ITSG-33) | PII exposure, logging PII |

---

## Example Complete Report

```markdown
## Security Review Results

**Policy Basis:** ITSG-33; Privacy Act
**Scope:** `src/api/users.ts`, `src/handlers/admin.ts`
**Date:** 2024-01-15

### Summary

- **Total Issues:** 4
- **Critical (Must Fix):** 2
- **Warnings (Should Fix):** 2
- **Passed Checks:** 3

### Findings

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| [Fail] | `src/handlers/admin.ts:45` | [AC] Admin endpoint missing role check | Add `requireRole('ADMIN')` guard |
| [Fail] | `src/api/users.ts:112` | [PII] `logger.info(user)` logs full user object | Change to `logger.info({ userId: user.id, action: 'fetch' })` |
| [Warning] | `src/api/users.ts:78` | [SI] `userId` param not validated | Add UUID validation before database query |
| [Warning] | `src/handlers/admin.ts:89` | [AU] User deletion without audit log | Add `auditLog({ action: 'DELETE_USER', ... })` |
| [Pass] | `src/config/cookies.ts` | [SC] Cookie security flags correctly set | None |
| [Pass] | `src/models/User.ts` | [PII] PII fields properly annotated | None |
| [Pass] | `src/db/queries.ts` | [SI] All queries use parameterization | None |
```

---

## Writing Recommendations

When writing recommendations:

1. **Be specific** - Reference exact file and line number
2. **Be actionable** - Provide concrete fix, not just "fix this"
3. **Include example** - Show code pattern for the fix when helpful
4. **Reference controls** - Link to ITSG-33 control for compliance tracking
