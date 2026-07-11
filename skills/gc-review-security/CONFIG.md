# gc-review-security Configuration

This document describes the configuration schema for the `gc-review-security` skill.

## Configuration File Location

**Project config:**
```
.gc-review/config.json
```

If no config exists, default settings are used. All `gc-review-security` options live under the `"security"` namespace so the file can be shared with other skills.

---

## Configuration Schema

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII", "@PII", "@Sensitive"],
    "auditLogService": "auditLog",
    "approvedAlgorithmExceptions": [],
    "strictMode": false
  }
}
```

---

## Field Definitions

### version
**Type:** `number`
**Required:** Yes
**Description:** Configuration schema version. Currently `1`. Top-level field shared by all gc-review skills.

```json
{
  "version": 1
}
```

---

### security.piiMetadataConvention
**Type:** `string[]`
**Required:** No
**Default:** unset
**Description:** Annotation/metadata names the project uses to mark PII fields (e.g., `isPII`, `@PII`, `@Sensitive`). Marking PII with metadata is a project convention that supports Privacy Act obligations — the *Privacy Act* does not mandate code-level annotations.

**Severity effect (rule C):**
- **Set:** PII fields missing one of these annotations → ❌ **Fail**
- **Unset:** PII fields without protective metadata → ⚠️ **Warning** (recommend adopting a convention)

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII", "@Sensitive"]
  }
}
```

---

### security.auditLogService
**Type:** `string`
**Required:** No
**Default:** unset
**Description:** Identifier (function, module, or service name) of the project's sanctioned audit logging service.

**Severity effect (rule E):**
- **Set:** security events that bypass this service (e.g., raw `console.log`) → ❌ **Fail**
- **Unset:** console/stdout logging of security events → ⚠️ **Warning** only (stdout-to-SIEM pipelines are standard in containerized deployments; verify forwarding to an approved GC centralized SIEM)

```json
{
  "version": 1,
  "security": {
    "auditLogService": "auditLog"
  }
}
```

---

### security.approvedAlgorithmExceptions
**Type:** `string[]`
**Required:** No
**Default:** `[]`
**Description:** Algorithm identifiers with a documented, assessor-approved exception (e.g., a legacy interoperability requirement). Findings involving these algorithms are downgraded from ❌ **Fail** to ⚠️ **Warning**, with a note referencing the exception. This does not exempt the project from ITSP.40.111 phase-out timelines.

```json
{
  "version": 1,
  "security": {
    "approvedAlgorithmExceptions": ["3DES"]
  }
}
```

---

### security.strictMode
**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** When enabled, all ⚠️ **Warning** findings are reported as ❌ **Fail**.

```json
{
  "version": 1,
  "security": {
    "strictMode": true
  }
}
```

---

## Complete Example Configurations

### Minimal Project Config

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII"]
  }
}
```

### Full Project Config

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII", "@PII", "@Sensitive"],
    "auditLogService": "SecurityAuditLogger",
    "approvedAlgorithmExceptions": [],
    "strictMode": false
  }
}
```

### Strict Government Production Config

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII"],
    "auditLogService": "auditLog",
    "strictMode": true
  }
}
```

---

## Validation Rules

### File Existence
1. Check if `.gc-review/config.json` exists
2. If missing, use defaults

### JSON Validation
```bash
# Validate JSON syntax
jq empty .gc-review/config.json 2>&1
```

If invalid JSON:
```
⚠️  Invalid JSON in .gc-review/config.json: {error}
Using default configuration.
```

### Field Validation

| Field | Validation |
|-------|------------|
| `version` | Must be number, currently `1` |
| `security.piiMetadataConvention` | Must be array of strings |
| `security.auditLogService` | Must be string |
| `security.approvedAlgorithmExceptions` | Must be array of strings |
| `security.strictMode` | Must be boolean |

### Invalid Field Handling

If a field has invalid type:
```
⚠️  Invalid config: {field} must be {expected_type}, got {actual_type}
Using default value for {field}.
```

### Unknown Field Handling

Unknown fields under `security` are ignored with a warning:
```
⚠️  Unknown config field: security.{field_name} (ignored)
```

---

## Config Resolution

```
.gc-review/config.json exists → use "security" namespace from project config
.gc-review/config.json missing → use default configuration
```

---

## Setting Up Configuration

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII"]
  }
}
EOF
```
