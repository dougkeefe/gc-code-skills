# gc-review-iam Configuration

This document describes the configuration schema for the `gc-review-iam` skill.

## Configuration File Location

**Project config:**
```
.gc-review/config.json
```

If no config exists, default settings are used.

All gc-review-iam options live under the `"iam"` namespace.

---

## Configuration Schema

```json
{
  "version": 1,
  "iam": {
    "additionalIdPs": [
      { "name": "Departmental ADFS", "issuer": "adfs.department.gc.ca" }
    ],
    "assuranceLevel": 2,
    "publicFacing": false,
    "strictMode": false
  }
}
```

---

## Field Definitions

### version
**Type:** `number`
**Required:** Yes
**Description:** Configuration schema version. Currently `1`.

```json
{
  "version": 1
}
```

---

### iam.additionalIdPs
**Type:** `array` of `{ "name": string, "issuer": string }`
**Required:** No
**Default:** `[]`
**Description:** Additional recognized identity providers beyond the GC enterprise services the skill recognizes by default (GCKey `clegc-gckey.gc.ca`, GCCF broker `fjgc-gccf.gc.ca`, CanadaLogin `login.canada.ca` / `app.login-connexion.canada.ca`, and the enterprise directory surface `login.microsoftonline.com`). Use this for legitimate departmental or provincial IdPs (e.g., departmental ADFS, GCpass).

Note: the Government of Canada does not publish a designated allow-list of identity providers. The binding requirement is the *Directive on Identity Management* / *Directive sur la gestion de l'identité*, s. 4.1.9 (effective 2019-07-01) — use mandatory GC enterprise identity, credential, and cyber-authentication services rather than custom credential stores. Entries here declare providers your department has accepted; they do not confer GC-wide approval.

**Example:**
```json
{
  "version": 1,
  "iam": {
    "additionalIdPs": [
      { "name": "Departmental ADFS", "issuer": "adfs.department.gc.ca" },
      { "name": "GCpass", "issuer": "gcpass.ssc-spc.gc.ca" }
    ]
  }
}
```

---

### iam.assuranceLevel
**Type:** `number` (1–4)
**Required:** No
**Default:** undeclared
**Description:** The service's Level of Assurance (LoA) per Appendix A of the Directive on Identity Management (*Standard on Identity and Credential Assurance* / *Norme sur l'assurance de l'identité et des justificatifs*). Drives two checks:

- **Session timeout (Check 5.2):** assertion-lifetime ceilings per ITSP.30.031 v3 — ≤ 12 hours single-domain at LoA 1–2; ≤ 30 minutes single-domain and ≤ 5 minutes cross-domain at LoA 3. When undeclared, the skill uses an 8-hour heuristic default (not a policy value).
- **MFA enforcement (Check 4.1):** MFA is mandatory at LoA 3 and above per ITSP.30.031 v3; missing MFA at LoA 3+ is a ❌ Fail.

```json
{
  "version": 1,
  "iam": {
    "assuranceLevel": 3
  }
}
```

---

### iam.publicFacing
**Type:** `boolean`
**Required:** No
**Default:** undeclared (the skill infers from context)
**Description:** Declares whether the application is a public-facing service. Affects Check 3.1 severity for consumer identity providers:

- `true` (public-facing): consumer IdPs (Google, Facebook, GitHub, Auth0) produce a ⚠️ Warning — verify sign-in is brokered through GCCF/CanadaLogin (GCKey, Interac Sign-In Partners) rather than integrated directly.
- `false` (internal/Protected B): consumer IdPs produce a ❌ Fail.

```json
{
  "version": 1,
  "iam": {
    "publicFacing": true
  }
}
```

---

### iam.strictMode
**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** When enabled, all ⚠️ Warning findings are promoted to ❌ Fail in the report. Equivalent to invoking the skill with `--strict` (either enables strict mode).

```json
{
  "version": 1,
  "iam": {
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
  "iam": {
    "publicFacing": true
  }
}
```

### Internal Protected B Application

```json
{
  "version": 1,
  "iam": {
    "assuranceLevel": 3,
    "publicFacing": false,
    "strictMode": true,
    "additionalIdPs": [
      { "name": "Departmental ADFS", "issuer": "adfs.department.gc.ca" }
    ]
  }
}
```

---

## Validation Rules

### File Existence
1. Check if config file exists
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
| `iam.additionalIdPs` | Must be array of objects with string `name` and `issuer` |
| `iam.assuranceLevel` | Must be number 1–4 |
| `iam.publicFacing` | Must be boolean |
| `iam.strictMode` | Must be boolean |

### Invalid Field Handling

If a field has invalid type:
```
⚠️  Invalid config: {field} must be {expected_type}, got {actual_type}
Using default value for {field}.
```

### Unknown Field Handling

Unknown fields are ignored with a warning:
```
⚠️  Unknown config field: {field_name} (ignored)
```

---

## Config Resolution

```
.gc-review/config.json exists → use project config (iam namespace)
.gc-review/config.json missing → use default configuration
```

---

## Setting Up Configuration

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1,
  "iam": {
    "assuranceLevel": 2,
    "publicFacing": false
  }
}
EOF
```
