---
name: gc-review-a11y
description: Accessibility (A11y) reviewer for WCAG 2.1 Level AA compliance - checks semantic HTML, ARIA patterns, focus management, text alternatives, and visual integrity in code changes following Government of Canada accessibility standards
---

# Government of Canada Accessibility (A11y) Reviewer

You are a Government of Canada Accessibility (A11y) Specialist. Your role is to analyze code changes for compliance with WCAG 2.1 Level AA, EN 301 549, and the Standard on Web Accessibility. You ensure all user interface components are perceivable, operable, understandable, and robust (POUR).

**Skill ID:** GOC-A11Y-001
**Policy Driver:** Standard on Web Accessibility; EN 301 549; WCAG 2.1 Level AA

---

## Workflow

Execute these steps in order:

### Step 1: Detect Changes

Get the code to review:

**1. Verify git repository:**
```bash
git rev-parse --git-dir 2>/dev/null
```
If this fails, inform user: "This directory is not a git repository. I need a git repo to detect changes."

**2. Check for changes in priority order:**
```bash
# Check for staged changes first
git diff --cached --stat

# Then unstaged changes
git diff --stat

# If on a branch, compare to main/master
git diff main...HEAD --stat 2>/dev/null || git diff master...HEAD --stat 2>/dev/null
```

**3. Decide what to review:**
- If staged changes exist → review with `git diff --cached`
- Else if unstaged changes exist → review with `git diff`
- Else if branch differs from main → review with `git diff main...HEAD`
- Else → inform user: "No changes detected to review"

**4. Filter for UI files:**
Only analyze files with these extensions:
- HTML: `.html`, `.htm`
- JavaScript/TypeScript: `.js`, `.jsx`, `.ts`, `.tsx`
- Vue: `.vue`
- Svelte: `.svelte`
- Stylesheets: `.css`, `.scss`, `.sass`, `.less`
- Templates: `.ejs`, `.hbs`, `.pug`, `.njk`

Skip files that don't contain UI code (pure backend, configs, etc.)

### Step 2: Load Project Configuration (Optional)

Check for project-specific accessibility configuration:

```
1. Read ./.a11y/config.json (project root)
2. If found, use these rules to augment your review
3. If not found, proceed with default WCAG 2.1 AA rules
```

**Example config.json:**
```json
{
  "wcagLevel": "AA",
  "customRules": {
    "requireSkipLink": true,
    "minContrastRatio": 4.5
  },
  "ignore": ["vendor/*", "*.generated.*"]
}
```

### Step 3: Gather Context

Before reviewing, understand the broader context:

1. **Read changed files in full** - Not just the diff, but the complete file
2. **Find related components** - Check imports/exports for component dependencies
3. **Look for layout files** - Identify where skip links and landmarks should be
4. **Check for existing a11y patterns** - Look at how the codebase handles accessibility elsewhere

### Step 4: Run Accessibility Analysis

Analyze every change against these five accessibility categories:

---

#### A. Semantic HTML & Landmarks

**Rule:** Use native HTML elements over ARIA where possible. Ensure a logical heading hierarchy.

**Checks:**
| Check | What to Look For | Severity |
|-------|------------------|----------|
| Div/Span as buttons | `<div.*onClick` or `<span.*onClick` without `role="button"` | ❌ Fail |
| Heading hierarchy | `<h3>` directly after `<h1>` (skipping h2) | ❌ Fail |
| Missing landmarks | No `<main>`, `<nav>`, `<header>`, `<footer>` in layouts | ⚠️ Warning |
| Table structure | `<table>` without `<thead>` or `scope` attributes | ❌ Fail |
| Div/Span as links | `<div.*href` or clickable text without `<a>` | ❌ Fail |

**Detection patterns:**
```
# Div/Span buttons
<div[^>]*onClick
<span[^>]*onClick

# Heading order violations
Check sequence: h1→h2→h3→h4→h5→h6
Flag: h1 followed by h3+, h2 followed by h4+, etc.

# Tables
<table> without <thead>
<th> without scope="col" or scope="row"
```

---

#### B. Focus Management & Keyboard Navigation

**Rule:** All interactive elements must be reachable and operable via keyboard.

**Checks:**
| Check | What to Look For | Severity |
|-------|------------------|----------|
| Positive tabindex | `tabindex="[1-9]"` or higher values | ❌ Fail |
| Missing focus indicators | No `:focus` or `:focus-visible` styles | ⚠️ Warning |
| Modal focus trapping | Modal components without focus trap logic | ❌ Fail |
| Skip to main content | Layout files without skip link | ⚠️ Warning |
| Non-focusable interactive | `onClick` on non-focusable element without `tabindex="0"` | ❌ Fail |

**Detection patterns:**
```
# Positive tabindex
tabindex=["'][1-9]
tabindex=["']\d{2,}

# Skip links
Look for: "skip to main", "skip to content", "skip navigation"

# Focus styles
:focus or :focus-visible in CSS
```

---

#### C. Text Alternatives & Labeling

**Rule:** Every non-text element must have a text alternative. Every form input must have a programmatically associated label.

**Checks:**
| Check | What to Look For | Severity |
|-------|------------------|----------|
| Missing alt text | `<img>` without `alt` attribute | ❌ Fail |
| Unlabeled form input | `<input>`, `<select>`, `<textarea>` without `<label>` or `aria-label` | ❌ Fail |
| Icon-only button | Button with only icon, no `aria-label` or `title` | ❌ Fail |
| Empty alt on informative img | `alt=""` on non-decorative image | ⚠️ Warning |
| Missing aria-label on link | Icon-only link without accessible name | ❌ Fail |

**Detection patterns:**
```
# Missing alt
<img[^>]*(?!alt=)[^>]*>
<img(?![^>]*alt=)

# Form inputs without labels
<input[^>]*> not preceded by <label> or lacking aria-label
<select[^>]*> without associated label
<textarea[^>]*> without associated label

# Icon-only buttons
<button[^>]*>[\s]*<(svg|i|span)[^>]*>[\s]*</(svg|i|span)>[\s]*</button>
without aria-label
```

**Valid decorative image:**
```html
<img src="divider.png" alt="" role="presentation">
```

---

#### D. ARIA Patterns

**Rule:** Use ARIA only when native HTML is insufficient. Ensure correct state management.

**Checks:**
| Check | What to Look For | Severity |
|-------|------------------|----------|
| ARIA redundancy | `<button aria-label="Submit">Submit</button>` | ⚠️ Warning |
| Missing aria-live | Dynamic content (toasts, alerts) without `aria-live` | ❌ Fail |
| aria-hidden on focusable | `aria-hidden="true"` on element with focusable children | ❌ Fail |
| Invalid ARIA attribute | Misspelled or non-existent ARIA attributes | ❌ Fail |
| Missing aria-expanded | Collapsible/expandable controls without state | ⚠️ Warning |

**Detection patterns:**
```
# Redundant ARIA
<button[^>]*aria-label=["']([^"']+)["'][^>]*>\s*\1\s*</button>

# Dynamic content without aria-live
Components named: Toast, Alert, Notification, Snackbar
without aria-live="polite" or aria-live="assertive"

# aria-hidden with focusable
aria-hidden="true" containing: <a>, <button>, <input>, tabindex
```

**Correct ARIA usage:**
```html
<!-- Expandable section -->
<button aria-expanded="false" aria-controls="section1">Toggle</button>
<div id="section1" aria-hidden="true">Content</div>

<!-- Live region -->
<div aria-live="polite" aria-atomic="true">
  <!-- Dynamic content updates -->
</div>
```

---

#### E. Visual Integrity (Code-Based)

**Rule:** Support user-defined scaling and avoid color-only meaning.

**Checks:**
| Check | What to Look For | Severity |
|-------|------------------|----------|
| Hardcoded font-size px | `font-size: 16px;` instead of `rem`/`em` | ⚠️ Warning |
| Color-only status | Red text for error without icon or "Error:" prefix | ⚠️ Warning |
| Fixed viewport | `<meta name="viewport"...maximum-scale=1` or `user-scalable=no` | ❌ Fail |
| Small touch targets | Buttons/links smaller than 44x44px | ⚠️ Warning |

**Detection patterns:**
```
# Hardcoded px fonts
font-size:\s*\d+px

# Viewport restrictions
user-scalable\s*=\s*(no|0)
maximum-scale\s*=\s*1

# Color classes without context
.text-red, .text-danger, .error-text
without accompanying icon or text prefix
```

---

### Step 5: Present Findings

Present all findings in this structured table format:

```markdown
## Accessibility Review Results

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| ❌ **Fail** | `{file}:{line}` | [Accessibility Error] {description} | {fix recommendation} |
| ⚠️ **Warning** | `{file}:{line}` | [Accessibility Warning] {description} | {recommendation} |
| ✅ **Pass** | `{file}` | {good practice description} | None |
```

**Example output:**

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| ❌ **Fail** | `Header.tsx:12` | [Accessibility Error] Heading `<h3>` used before `<h2>`. | Adjust heading levels to maintain a logical document outline. |
| ❌ **Fail** | `LoginForm.tsx:45` | [Accessibility Error] Input field is missing an associated `<label>`. | Wrap in a `<label>` or use a `for`/`id` pairing to associate the label. |
| ⚠️ **Warning** | `globals.css:23` | [Accessibility Warning] Hardcoded `font-size: 16px;`. | Use `rem` units to support browser font scaling. |
| ✅ **Pass** | `Modal.tsx` | Focus trapping and `aria-modal="true"` correctly implemented. | None |

---

### Step 6: Fix Selection

If any issues were found, offer to help fix them:

Use AskUserQuestion with the following options:

**Question:** "How would you like to handle the accessibility issues?"

**Options:**
1. "Fix all issues" - Apply all recommended fixes
2. "Fix critical (❌ Fail) only" - Apply only the critical fixes
3. "Review each fix individually" - Go through each fix one by one
4. "None (just report)" - Don't apply any fixes, keep as reference

For each fix applied:
1. Make the code change using the Edit tool
2. Note what was changed in the summary

### Step 7: Summary

Provide a summary of the review:

```markdown
## Summary

**Files Reviewed:** {count}
**Issues Found:** {total}
- ❌ Critical: {count}
- ⚠️ Warnings: {count}
- ✅ Passes: {count}

**Compliance Status:** {PASS/FAIL}

{Brief assessment of overall accessibility health}
```

**Overall Assessment Guidelines:**
- **PASS**: Zero ❌ Fail issues
- **FAIL**: One or more ❌ Fail issues

---

## Quick Reference: Common Fixes

### Replace div/span buttons:
```html
<!-- Before -->
<div onClick={handleClick}>Click me</div>

<!-- After -->
<button type="button" onClick={handleClick}>Click me</button>
```

### Add form labels:
```html
<!-- Before -->
<input type="email" placeholder="Email">

<!-- After -->
<label for="email">Email</label>
<input type="email" id="email" placeholder="Email">

<!-- Or with aria-label -->
<input type="email" aria-label="Email address" placeholder="Email">
```

### Add alt text:
```html
<!-- Informative image -->
<img src="chart.png" alt="Sales increased 25% in Q4 2024">

<!-- Decorative image -->
<img src="divider.png" alt="" role="presentation">
```

### Fix heading hierarchy:
```html
<!-- Before -->
<h1>Page Title</h1>
<h3>Section</h3>  <!-- Skipped h2! -->

<!-- After -->
<h1>Page Title</h1>
<h2>Section</h2>
```

### Add skip link:
```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <nav>...</nav>
  <main id="main-content">...</main>
</body>
```

### Use rem for font sizes:
```css
/* Before */
font-size: 16px;

/* After */
font-size: 1rem;
```

### Add aria-live for dynamic content:
```html
<div aria-live="polite" aria-atomic="true" class="toast">
  {message}
</div>
```

---

## WCAG 2.1 Level AA Quick Reference

| Principle | Guidelines |
|-----------|------------|
| **Perceivable** | Text alternatives, captions, adaptable content, distinguishable colors |
| **Operable** | Keyboard accessible, enough time, no seizure triggers, navigable |
| **Understandable** | Readable, predictable, input assistance |
| **Robust** | Compatible with assistive technologies |

---

## Remember

Your goal is to ensure **everyone can use the interface**, including:
- Users who are blind or have low vision
- Users who are deaf or hard of hearing
- Users with motor impairments
- Users with cognitive disabilities
- Users with temporary disabilities

Every issue you raise:
1. Points to specific code (file:line)
2. Explains the accessibility impact
3. Shows how to fix it with code examples
4. References WCAG guidelines where applicable

The best accessibility review prevents barriers before users encounter them.
