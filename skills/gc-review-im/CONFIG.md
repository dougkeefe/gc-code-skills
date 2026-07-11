# gc-review-im Configuration

This document describes the configuration schema for the `gc-review-im` skill.

## Configuration File Location

**Project config:**
```
.gc-review/config.json
```

If no config exists, default settings are used.

All `gc-review-im` options live under the `"im"` namespace key. The top level of the file contains `"version"` plus one namespace object per skill.

---

## Configuration Schema (defaults shown)

```json
{
  "version": 1,
  "im": {
    "requiredMetadata": ["record_id", "creator_id", "date_created", "language", "classification"],
    "softDeleteFields": ["deleted_at", "is_deleted", "archived_at"],
    "retentionFields": ["retention_schedule", "disposition_date", "retention_code", "retention_period"],
    "exclude": [],
    "includePatterns": [],
    "strictMode": false,
    "transitoryPatterns": ["*_sessions", "*_jobs", "*_cache", "*_tokens", "*_queue", "*_join"],
    "businessRecordPatterns": []
  }
}
```

---

## Field Definitions

### version
**Type:** `number`
**Required:** Yes
**Description:** Configuration schema version. Currently `1`. Lives at the top level of the file (not inside `"im"`).

```json
{
  "version": 1
}
```

---

### im.requiredMetadata
**Type:** `string[]`
**Required:** No
**Default:** `["record_id", "creator_id", "date_created", "language", "classification"]`
**Description:** Metadata baseline fields checked on business-value record models (Category A). These are ⚠️ Warning design guidance anchored to LAC's *Operational Standard for Digital Archival Records' Metadata* (2023-11-16) and the TBS *Metadata Reference Standard for Digital Archival Records* (2025-06-23) — the GC mandates metadata concepts, not column names.

Each field can have acceptable variants that will be recognized:

| Field | Recognized Variants |
|-------|---------------------|
| `record_id` | `id`, `uuid`, `record_uuid` |
| `creator_id` | `created_by`, `author_id`, `owner_id`, `creatorId` |
| `date_created` | `created_at`, `creation_date`, `created_date`, `createdAt` |
| `language` | `lang`, `locale`, `content_language` |
| `classification` | `security_classification`, `security_level`, `protected_level` |

**Example - Adding custom required field:**
```json
{
  "version": 1,
  "im": {
    "requiredMetadata": [
      "record_id",
      "creator_id",
      "date_created",
      "language",
      "classification",
      "department_code"
    ]
  }
}
```

**Example - Minimal requirements:**
```json
{
  "version": 1,
  "im": {
    "requiredMetadata": [
      "creator_id",
      "date_created"
    ]
  }
}
```

---

### im.softDeleteFields
**Type:** `string[]`
**Required:** No
**Default:** `["deleted_at", "is_deleted", "archived_at"]`
**Description:** Field names that indicate soft delete support. If a model has one of these fields, hard delete warnings may be suppressed.

**Example:**
```json
{
  "version": 1,
  "im": {
    "softDeleteFields": [
      "deleted_at",
      "is_deleted",
      "archived_at",
      "status",
      "is_active"
    ]
  }
}
```

---

### im.retentionFields
**Type:** `string[]`
**Required:** No
**Default:** `["retention_schedule", "disposition_date", "retention_code", "retention_period"]`
**Description:** Field names that indicate retention/disposition support.

**Example:**
```json
{
  "version": 1,
  "im": {
    "retentionFields": [
      "retention_schedule",
      "disposition_date",
      "retention_code",
      "disposal_authority",
      "records_schedule_code"
    ]
  }
}
```

---

### im.exclude
**Type:** `string[]`
**Required:** No
**Default:** `[]`
**Description:** Glob patterns for files to exclude from review.

**Common patterns:**
```json
{
  "version": 1,
  "im": {
    "exclude": [
      "**/test/**",
      "**/tests/**",
      "**/__tests__/**",
      "*.spec.ts",
      "*.test.ts",
      "**/fixtures/**",
      "**/seeds/**",
      "**/migrations/*.down.sql"
    ]
  }
}
```

---

### im.includePatterns
**Type:** `string[]`
**Required:** No
**Default:** `[]` (uses built-in patterns)
**Description:** Additional glob patterns for files to include in review. Extends the default patterns.

**Example - Adding custom patterns:**
```json
{
  "version": 1,
  "im": {
    "includePatterns": [
      "**/domain/*.ts",
      "**/aggregates/**/*.ts"
    ]
  }
}
```

---

### im.strictMode
**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** When enabled, all warnings are treated as errors.

```json
{
  "version": 1,
  "im": {
    "strictMode": true
  }
}
```

**Effect:**
- Metadata baseline gaps → ❌ Fail (instead of ⚠️ Warning)
- Non-descriptive field names → ❌ Fail (instead of ⚠️ Warning)
- Missing audit trail → ❌ Fail (instead of ⚠️ Warning)
- Missing indexes → ❌ Fail (instead of ⚠️ Warning)

---

### im.transitoryPatterns
**Type:** `string[]`
**Required:** No
**Default:** `["*_sessions", "*_jobs", "*_cache", "*_tokens", "*_queue", "*_join"]`
**Description:** Glob patterns for model/table names to classify as **transitory/operational** in Step 4.0 (cf. LAC Multi-Institution Disposition Authorization DA 2016/001). Transitory models are exempt from Category A (metadata baseline) and from the ❌ Fail branch of the hard-delete check; deleting transitory information at end of usefulness is permitted under DA 2016/001.

**Example:**
```json
{
  "version": 1,
  "im": {
    "transitoryPatterns": [
      "*_sessions",
      "*_jobs",
      "*_cache",
      "*_tokens",
      "*_queue",
      "*_join",
      "feature_flags",
      "*_lookup"
    ]
  }
}
```

---

### im.businessRecordPatterns
**Type:** `string[]`
**Required:** No
**Default:** `[]`
**Description:** Glob patterns for model/table names to always classify as **business-value records** in Step 4.0 (DSD s. 4.4.10 — information of business value). Overrides `transitoryPatterns` on conflict. Use this to force full Category A/B review for models whose names would otherwise look operational.

**Example:**
```json
{
  "version": 1,
  "im": {
    "businessRecordPatterns": [
      "grant_*",
      "case_*",
      "decisions"
    ]
  }
}
```

---

## Complete Example Configurations

### Minimal Project Config

```json
{
  "version": 1,
  "im": {
    "exclude": ["**/test/**"]
  }
}
```

### Full Project Config

```json
{
  "version": 1,
  "im": {
    "requiredMetadata": [
      "record_id",
      "creator_id",
      "date_created",
      "language",
      "classification",
      "department_code"
    ],
    "softDeleteFields": [
      "deleted_at",
      "is_deleted",
      "archived_at",
      "status"
    ],
    "retentionFields": [
      "retention_schedule",
      "disposition_date",
      "retention_code",
      "disposal_authority"
    ],
    "exclude": [
      "**/test/**",
      "**/tests/**",
      "**/__tests__/**",
      "*.spec.ts",
      "*.test.ts",
      "**/fixtures/**",
      "**/seeds/**"
    ],
    "includePatterns": [
      "**/domain/**/*.ts"
    ],
    "strictMode": false,
    "transitoryPatterns": [
      "*_sessions",
      "*_jobs",
      "*_cache",
      "*_tokens",
      "*_queue",
      "*_join"
    ],
    "businessRecordPatterns": [
      "grant_*",
      "case_*"
    ]
  }
}
```

### Strict Government Production Config

```json
{
  "version": 1,
  "im": {
    "requiredMetadata": [
      "record_id",
      "creator_id",
      "date_created",
      "language",
      "classification",
      "department_code",
      "retention_code"
    ],
    "strictMode": true,
    "exclude": []
  }
}
```

---

## Validation Rules

### File Existence
1. Check if config file exists
2. If missing, proceed to next precedence level
3. If no config found, use defaults

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
| `version` | Must be number, currently `1` (top level) |
| `im` | Must be object; all skill options live inside it |
| `im.requiredMetadata` | Must be array of strings |
| `im.softDeleteFields` | Must be array of strings |
| `im.retentionFields` | Must be array of strings |
| `im.exclude` | Must be array of strings (glob patterns) |
| `im.includePatterns` | Must be array of strings (glob patterns) |
| `im.strictMode` | Must be boolean |
| `im.transitoryPatterns` | Must be array of strings (glob patterns) |
| `im.businessRecordPatterns` | Must be array of strings (glob patterns) |

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

Legacy top-level keys (e.g., `requiredMetadata` outside `"im"`) are treated as unknown fields — move them under `"im"`.

---

## Config Resolution

```
.gc-review/config.json exists → use project config ("im" key)
.gc-review/config.json missing → use default configuration
```

---

## Setting Up Configuration

### Create project config

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1,
  "im": {
    "exclude": ["**/test/**", "**/tests/**"]
  }
}
EOF
```

### Add to .gitignore (optional)

If you want project-specific config to be shared:
```
# Keep .gc-review/config.json tracked
```

If you want it ignored:
```
# .gitignore
.gc-review/
```
