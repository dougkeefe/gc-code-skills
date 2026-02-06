# gc-review-bilingual

A Claude Code skill for reviewing code against Government of Canada Official Languages Act (OLA) compliance requirements. Ensures applications provide equivalent experiences in both English and French.

## Installation

```bash
claude skills add /path/to/gc-review-bilingual
```

Or add to your project's `.claude/settings.json`:

```json
{
  "skills": [
    "/path/to/gc-review-bilingual"
  ]
}
```

## Usage

Invoke the skill in Claude Code:

```
/gc-review-bilingual
```

Or ask Claude to review for bilingualism:

> "Review this code for Official Languages Act compliance"
> "Check if all strings are translated to French"
> "Verify bilingual support in my changes"

## What It Checks

### A. String Extraction (Anti-Hardcoding)
Flags hardcoded text in UI components that should use translation functions.

### B. Dictionary Parity
Compares English and French translation files for:
- Missing keys in either language
- Placeholder values (TODO, TRANSLATE ME)
- Suspiciously identical long strings

### C. Localized Navigation & Routing
Verifies locale-aware routing and dynamic `<html lang>` attributes.

### D. Locale-Aware Formatting
Checks that dates, numbers, and currencies use `Intl` formatters with locale parameters.

### E. Accessibility Parity
Ensures `aria-label`, `alt`, and `title` attributes use translation keys.

## Supported Frameworks

- next-intl (Next.js)
- react-i18next
- vue-i18n
- @angular/localize
- svelte-i18n

## Configuration

Create `.bilingual-review.json` in your project root:

```json
{
  "translationFiles": {
    "english": ["messages/en.json"],
    "french": ["messages/fr.json"]
  },
  "scanPaths": ["src/**/*.tsx", "app/**/*.tsx"],
  "excludePaths": ["**/*.test.*"],
  "minStringLength": 3,
  "identicalThreshold": 20,
  "ignoreWords": ["GitHub", "OAuth", "API"]
}
```

## Output Format

The skill produces a structured report:

| Status | File | Issue Found | Recommended Action |
|--------|------|-------------|-------------------|
| :x: Fail | `Header.tsx:24` | Hardcoded string "Sign In" | Use `t('auth.signIn')` |
| :x: Fail | `fr.json` | Missing key `dashboard.title` | Add French translation |
| :warning: Warning | `fr.json` | `nav.help` is "TODO" | Provide translation |
| :white_check_mark: Pass | `layout.tsx` | Dynamic `lang` attribute | None |

## Policy Reference

**Skill ID:** GOC-BILINGUAL-001
**Policy Driver:** Official Languages Act; Directive on the Management of Communications

## License

MIT
