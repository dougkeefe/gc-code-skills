# gc-review-im Configuration

This document describes the configuration schema for the `gc-review-im` skill.

## Configuration File Locations

**Project config (highest priority):**
```
.gc-review/config.json
```

**User config (fallback):**
```
~/.gc-review/config.json
```

If neither exists, default settings are used.

---

## Configuration Schema

```json
{
  "version": 1,
  "requiredMetadata": ["record_id", "creator_id", "date_created", "language", "classification"],
  "softDeleteFields": ["deleted_at", "is_deleted", "archived_at"],
  "retentionFields": ["retention_schedule", "disposition_date", "retention_code", "retention_period"],
  "exclude": ["**/test/**", "**/tests/**", "*.spec.ts", "*.test.ts"],
  "includePatterns": [],
  "strictMode": false
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

### requiredMetadata
**Type:** `string[]`
**Required:** No
**Default:** `["record_id", "creator_id", "date_created", "language", "classification"]`
**Description:** List of metadata fields required on business record models.

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
  "requiredMetadata": [
    "record_id",
    "creator_id",
    "date_created",
    "language",
    "classification",
    "department_code"
  ]
}
```

**Example - Minimal requirements:**
```json
{
  "requiredMetadata": [
    "creator_id",
    "date_created"
  ]
}
```

---

### softDeleteFields
**Type:** `string[]`
**Required:** No
**Default:** `["deleted_at", "is_deleted", "archived_at"]`
**Description:** Field names that indicate soft delete support. If a model has one of these fields, hard delete warnings may be suppressed.

**Example:**
```json
{
  "softDeleteFields": [
    "deleted_at",
    "is_deleted",
    "archived_at",
    "status",
    "is_active"
  ]
}
```

---

### retentionFields
**Type:** `string[]`
**Required:** No
**Default:** `["retention_schedule", "disposition_date", "retention_code", "retention_period"]`
**Description:** Field names that indicate retention/disposition support.

**Example:**
```json
{
  "retentionFields": [
    "retention_schedule",
    "disposition_date",
    "retention_code",
    "disposal_authority",
    "records_schedule_code"
  ]
}
```

---

### exclude
**Type:** `string[]`
**Required:** No
**Default:** `[]`
**Description:** Glob patterns for files to exclude from review.

**Common patterns:**
```json
{
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
```

---

### includePatterns
**Type:** `string[]`
**Required:** No
**Default:** `[]` (uses built-in patterns)
**Description:** Additional glob patterns for files to include in review. Extends the default patterns.

**Example - Adding custom patterns:**
```json
{
  "includePatterns": [
    "**/domain/*.ts",
    "**/aggregates/**/*.ts"
  ]
}
```

---

### strictMode
**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** When enabled, all warnings are treated as errors.

```json
{
  "strictMode": true
}
```

**Effect:**
- Non-descriptive field names → ❌ Fail (instead of ⚠️ Warning)
- Missing audit trail → ❌ Fail (instead of ⚠️ Warning)
- Missing indexes → ❌ Fail (instead of ⚠️ Warning)

---

## Complete Example Configurations

### Minimal Project Config

```json
{
  "version": 1,
  "exclude": ["**/test/**"]
}
```

### Full Project Config

```json
{
  "version": 1,
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
  "strictMode": false
}
```

### Strict Government Production Config

```json
{
  "version": 1,
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
| `version` | Must be number, currently `1` |
| `requiredMetadata` | Must be array of strings |
| `softDeleteFields` | Must be array of strings |
| `retentionFields` | Must be array of strings |
| `exclude` | Must be array of strings (glob patterns) |
| `includePatterns` | Must be array of strings (glob patterns) |
| `strictMode` | Must be boolean |

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

## Config Precedence Examples

### Scenario 1: Project config only

```
.gc-review/config.json exists → use project config
```

### Scenario 2: User config only

```
.gc-review/config.json missing
~/.gc-review/config.json exists → use user config
```

### Scenario 3: Both exist

```
.gc-review/config.json exists → use project config
~/.gc-review/config.json ignored
```

### Scenario 4: Neither exists

```
.gc-review/config.json missing
~/.gc-review/config.json missing
→ use default configuration
```

---

## Setting Up Configuration

### Create project config

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1,
  "exclude": ["**/test/**", "**/tests/**"]
}
EOF
```

### Create user config

```bash
mkdir -p ~/.gc-review
cat > ~/.gc-review/config.json << 'EOF'
{
  "version": 1,
  "strictMode": false
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
