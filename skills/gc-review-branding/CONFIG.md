# GC Branding Review Configuration

This document describes the configuration options for the `gc-review-branding` skill.

## Configuration File

Create `.gc-branding/config.json` in your project root to customize the review behavior.

## Full Schema

```json
{
  "version": 1,
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
  "excludePatterns": [
    "vendor/*",
    "*.generated.*",
    "node_modules/**"
  ],
  "strictMode": false
}
```

## Field Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | number | `1` | Config schema version |
| `department` | string | — | Department name for reporting |
| `signature.altText` | string | `"Government of Canada"` | Expected alt text for GC signature |
| `signature.componentName` | string | `"GCSignature"` | Component name to search for |
| `wordmark.componentName` | string | `"CanadaWordmark"` | Wordmark component name |
| `additionalColors` | object | `{}` | Extra approved color tokens |
| `additionalFonts` | array | `[]` | Extra approved font families |
| `excludePatterns` | array | `[]` | Glob patterns to skip |
| `strictMode` | boolean | `false` | Treat warnings as failures |

## Examples

### Basic Department Config

```json
{
  "version": 1,
  "department": "Employment and Social Development Canada"
}
```

### Custom Component Names

If your project uses non-standard component names:

```json
{
  "version": 1,
  "signature": {
    "componentName": "GovHeader"
  },
  "wordmark": {
    "componentName": "FederalWordmark"
  }
}
```

### Extended Color Palette

If your department has approved accent colors:

```json
{
  "version": 1,
  "department": "Parks Canada",
  "additionalColors": {
    "--parks-green": "#2E8B57",
    "--parks-brown": "#8B4513"
  }
}
```

### Excluding Generated Files

```json
{
  "version": 1,
  "excludePatterns": [
    "*.generated.tsx",
    "dist/**",
    "coverage/**",
    "__mocks__/**"
  ]
}
```

### Strict Mode for CI/CD

Enable strict mode to fail builds on any compliance issue:

```json
{
  "version": 1,
  "strictMode": true
}
```

In strict mode, warnings are elevated to failures, causing the review to report an overall FAIL status.

## Validation

The skill validates your config on load:

1. **JSON syntax** - Must be valid JSON
2. **Version check** - Must be version `1`
3. **Type validation** - Each field must match expected type
4. **Pattern validation** - Exclude patterns must be valid globs

Invalid configs produce a warning and fall back to defaults.

## File Location

The config file must be at:
```
.gc-branding/config.json
```

This keeps branding configuration separate from other project configs and allows easy exclusion from version control if needed.
