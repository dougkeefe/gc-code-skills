# GC Branding Review Configuration

This document describes the configuration options for the `gc-review-branding` skill.

## Configuration File

Create `.gc-review/config.json` in your project root to customize the review behavior. All branding options live under the `"branding"` namespace, alongside other skills' namespaces in the same shared file.

## Full Schema

```json
{
  "version": 1,
  "branding": {
    "department": "Department of Example",
    "signature": {
      "altText": "Government of Canada / Gouvernement du Canada",
      "componentName": "GCSignature"
    },
    "wordmark": {
      "componentName": "CanadaWordmark"
    },
    "additionalColors": {
      "--dept-accent": "#custom-hex",
      "--dept-secondary": "#another-hex"
    },
    "additionalFonts": ["Montserrat"],
    "exclude": [
      "vendor/*",
      "*.generated.*",
      "node_modules/**"
    ],
    "strictMode": false
  }
}
```

## Field Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | number | `1` | Config schema version (top-level, required) |
| `branding.department` | string | — | Department name for reporting |
| `branding.signature.altText` | string | `"Government of Canada"` | Expected alt text for GC signature |
| `branding.signature.componentName` | string | `"GCSignature"` | Component name to search for |
| `branding.wordmark.componentName` | string | `"CanadaWordmark"` | Wordmark component name |
| `branding.additionalColors` | object | `{}` | Extra approved color tokens (Step 7) |
| `branding.additionalFonts` | array | `[]` | Extra approved font families (Step 6) |
| `branding.exclude` | array | `[]` | Glob patterns to skip (Step 3) |
| `branding.strictMode` | boolean | `false` | Treat warnings as failures (Step 10) |

## Examples

### Basic Department Config

```json
{
  "version": 1,
  "branding": {
    "department": "Employment and Social Development Canada"
  }
}
```

### Custom Component Names

If your project uses non-standard component names:

```json
{
  "version": 1,
  "branding": {
    "signature": {
      "componentName": "GovHeader"
    },
    "wordmark": {
      "componentName": "FederalWordmark"
    }
  }
}
```

### Extended Color Palette

If your department has approved accent colors:

```json
{
  "version": 1,
  "branding": {
    "department": "Parks Canada",
    "additionalColors": {
      "--parks-green": "#2E8B57",
      "--parks-brown": "#8B4513"
    }
  }
}
```

### Additional Approved Fonts

If your application has an approved exception to the Lato / Noto Sans stack:

```json
{
  "version": 1,
  "branding": {
    "additionalFonts": ["Montserrat", "Source Sans Pro"]
  }
}
```

Fonts listed here are treated as approved during the Step 6 typography audit.

### Excluding Generated Files

```json
{
  "version": 1,
  "branding": {
    "exclude": [
      "*.generated.tsx",
      "dist/**",
      "coverage/**",
      "__mocks__/**"
    ]
  }
}
```

### Strict Mode for CI/CD

Enable strict mode to fail builds on any compliance issue:

```json
{
  "version": 1,
  "branding": {
    "strictMode": true
  }
}
```

In strict mode, the Step 10 report elevates warnings to failures, causing the review to report an overall FAIL status when any finding exists.

## Validation

The skill validates your config on load:

1. **JSON syntax** - Must be valid JSON
2. **Version check** - Must contain `"version": 1` (required); configs without it are rejected
3. **Type validation** - Each field must match expected type
4. **Pattern validation** - Exclude patterns must be valid globs

Invalid configs produce a warning and fall back to defaults.

## File Location

The config file must be at:
```
.gc-review/config.json
```

This keeps GC review configuration separate from other project configs and allows easy exclusion from version control if needed.
