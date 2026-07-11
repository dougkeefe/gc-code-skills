# gc-review-a11y Configuration

This document describes the configuration schema for the `gc-review-a11y` skill.

## Configuration File Location

**Project config:**
```
.gc-review/config.json
```

Accessibility options live under the `"a11y"` namespace. If no config exists, default settings are used (WCAG 2.2 AA baseline).

**Deprecated legacy path:** `./.a11y/config.json` is still read as a fallback for one release only. When it is used, the skill emits:

```
⚠️  Deprecated config path: ./.a11y/config.json is supported for one release only.
Migrate to ./.gc-review/config.json with {"version": 1, "a11y": {...}}.
```

---

## Configuration Schema

```json
{
  "version": 1,
  "a11y": {
    "wcagLevel": "AA",
    "requireSkipLink": true,
    "minContrastRatio": 4.5,
    "strictMode": false,
    "exclude": ["vendor/*", "*.generated.*"]
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

### a11y.wcagLevel
**Type:** `string`
**Required:** No
**Default:** `"AA"`
**Description:** WCAG 2.2 conformance level to review against. `"AA"` is the skill's baseline (exceeding the WCAG 2.1 AA minimum incorporated by CAN/ASC - EN 301 549:2024). `"AAA"` additionally applies enhanced criteria (e.g., 44×44 CSS px target size per SC 2.5.5) as warnings.

```json
{
  "version": 1,
  "a11y": { "wcagLevel": "AA" }
}
```

---

### a11y.requireSkipLink
**Type:** `boolean`
**Required:** No
**Default:** `true`
**Description:** When `true`, layout files without a "skip to main content" link are flagged as ⚠️ Warning. Set to `false` for projects where skip links are injected by a shared shell or framework.

```json
{
  "version": 1,
  "a11y": { "requireSkipLink": false }
}
```

---

### a11y.minContrastRatio
**Type:** `number`
**Required:** No
**Default:** `4.5`
**Description:** Minimum contrast ratio for normal text (SC 1.4.3). Large text uses 3:1 and non-text elements use 3:1 (SC 1.4.11) regardless of this setting. Raise to `7` to review against the AAA enhanced-contrast value (SC 1.4.6).

```json
{
  "version": 1,
  "a11y": { "minContrastRatio": 4.5 }
}
```

---

### a11y.strictMode
**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** When enabled, all warnings are treated as errors.

```json
{
  "version": 1,
  "a11y": { "strictMode": true }
}
```

**Effect:**
- Small touch targets → ❌ Fail (instead of ⚠️ Warning)
- Missing skip link → ❌ Fail (instead of ⚠️ Warning)
- Contrast findings → ❌ Fail (instead of ⚠️ Warning)

---

### a11y.exclude
**Type:** `string[]`
**Required:** No
**Default:** `[]`
**Description:** Glob patterns for files to exclude from review.

**Common patterns:**
```json
{
  "version": 1,
  "a11y": {
    "exclude": [
      "vendor/*",
      "*.generated.*",
      "**/node_modules/**",
      "**/__tests__/**",
      "*.spec.tsx",
      "*.test.tsx"
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
  "a11y": {
    "exclude": ["vendor/*"]
  }
}
```

### Full Project Config

```json
{
  "version": 1,
  "a11y": {
    "wcagLevel": "AA",
    "requireSkipLink": true,
    "minContrastRatio": 4.5,
    "strictMode": false,
    "exclude": [
      "vendor/*",
      "*.generated.*",
      "**/__tests__/**"
    ]
  }
}
```

### Strict Government Production Config

```json
{
  "version": 1,
  "a11y": {
    "wcagLevel": "AA",
    "requireSkipLink": true,
    "minContrastRatio": 4.5,
    "strictMode": true,
    "exclude": []
  }
}
```

---

## Validation Rules

### File Existence
1. Check if `.gc-review/config.json` exists
2. If missing, check the deprecated `./.a11y/config.json` (emit deprecation warning if used)
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
| `a11y.wcagLevel` | Must be `"AA"` or `"AAA"` |
| `a11y.requireSkipLink` | Must be boolean |
| `a11y.minContrastRatio` | Must be number ≥ 3 |
| `a11y.strictMode` | Must be boolean |
| `a11y.exclude` | Must be array of strings (glob patterns) |

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
.gc-review/config.json exists → use project config ("a11y" namespace)
.gc-review/config.json missing, ./.a11y/config.json exists → use legacy config with deprecation warning
neither exists → use default configuration (WCAG 2.2 AA baseline)
```

---

## Setting Up Configuration

### Create project config

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1,
  "a11y": {
    "wcagLevel": "AA",
    "requireSkipLink": true
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
