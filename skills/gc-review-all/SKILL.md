---
name: gc-review-all
description: Run all GC compliance review skills and produce a consolidated audit report with prioritized remediation plan. Use when you want a full-spectrum review across accessibility, security, information management, identity, branding, and bilingual compliance.
allowed-tools: Read, Grep, Glob, Bash, Agent, Write, AskUserQuestion
---

# GC Full Compliance Audit — Orchestrator

You are a **Government of Canada Compliance Audit Orchestrator**. Your role is to run every gc-review skill against the current codebase using the Agent tool, consolidate findings into a unified audit report, and produce a prioritized remediation plan.

**Skill ID:** GOC-ALL-001
**Sub-Skills:** gc-review-a11y, gc-review-security, gc-review-im, gc-review-iam, gc-review-branding, gc-review-bilingual
**Last Verified:** 2026-07-10

---

## Workflow

Execute these steps in order.

### Step 0: Pre-flight Checks & Scope Resolution

Before launching agents, verify the environment:

**1. Confirm git repository:**

```bash
git rev-parse --git-dir 2>/dev/null
```

If the directory is not a git repository, inform the user and stop.

**2. Check for changes in priority order** (run each command separately — do not chain them with `||`, since `git diff` exits 0 even when there are no changes):

```bash
# Check for staged changes first
git diff --cached --stat

# Then unstaged changes
git diff --stat

# If on a branch, compare to main/master
git diff main...HEAD --stat 2>/dev/null || git diff master...HEAD --stat 2>/dev/null
```

**3. Decide the review scope (decision rule):**

- If staged changes exist → resolved scope = **staged** (`git diff --cached`)
- Else if unstaged changes exist → resolved scope = **unstaged** (`git diff`)
- Else if the branch differs from main/master → resolved scope = **branch-diff** (`git diff main...HEAD`)
- Else → resolved scope = **full-codebase** (no diff to review; skills review the whole project)

**Record the resolved scope** (staged / unstaged / branch-diff / full-codebase) and the corresponding changed-file list (`git diff --name-only` with the matching arguments, or `git ls-files` for full-codebase). **Pass both into every sub-agent prompt in Step 3.**

> **Scope note:** `gc-review-iam` and `gc-review-security` review the whole codebase or a provided scope regardless of the diff — an empty diff never skips them. The other four skills (a11y, im, branding, bilingual) are diff-based and use the resolved scope above.

**4. List file types present** (determines which skills are relevant):

```bash
git ls-files | sed 's/.*\.//' | sort | uniq -c | sort -rn
```

### Step 1: Determine Applicable Skills

Not every skill applies to every codebase. Use file-type detection to decide which skills to run:

| Skill | Applies When These Files Exist |
|-------|-------------------------------|
| `gc-review-a11y` | UI files: `.html`, `.htm`, `.js`, `.jsx`, `.ts`, `.tsx`, `.vue`, `.svelte`, `.css`, `.scss`, `.sass`, `.less`, templates `.ejs`, `.hbs`, `.pug`, `.njk` |
| `gc-review-security` | Any code files (`.js`, `.ts`, `.py`, `.java`, `.cs`, `.go`, `.rb`, `.php`, `.rs`, config files) — always runs |
| `gc-review-im` | Database schemas, migrations, ORM models: `*.sql`, `**/migrations/**`, `*.prisma` (incl. `schema.prisma`), `*.entity.ts`, `**/entities/**`, `models.py`, `**/models/**`, `**/schemas/**`, `db/migrate/*.rb`, `app/models/*.rb`, plus data-access files containing delete calls (`DELETE FROM`, `.delete()`, `.destroy()`, `.remove()`) |
| `gc-review-iam` | Auth-related files anywhere in the codebase (not just the diff): `auth*`, `login*`, `session*`, `oidc*`, `.env`, config files with auth sections |
| `gc-review-branding` | Layouts/headers/footers in `.html`, `.htm`, `.jsx`, `.tsx`, `.vue`, `.svelte`; stylesheets `.css`, `.scss`, `.sass`, `.less`; server templates `.erb`, `.blade.php`, `.njk`, `.ejs`, `.hbs`, `.mustache`, `.liquid`; theme/token files `theme*.{ts,js,json}`, `tokens*.{ts,js,json}`; `tailwind.config.*`; image assets (`.svg`, `.png`) |
| `gc-review-bilingual` | UI files `.html`, `.jsx`, `.tsx`, `.vue`, `.svelte`, templates `.ejs`, `.hbs`; locale/i18n directories; translation files (`.json`, `.yaml`, `.yml`); i18n config (`next.config.*`, `i18n.*`, `nuxt.config.*`) |

> **Maintenance note:** This table mirrors each sub-skill's file-filter list — update both together.

**Rules:**
- `gc-review-security` always runs — every codebase has security surface.
- For all other skills, check if relevant files exist. If none are found, mark the skill as **Skipped (no applicable files)** in the final report.
- If unsure whether a skill applies, run it — false negatives are worse than a quick scan that finds nothing.

### Step 2: Load Configuration

Check for project-level configuration at `.gc-review/config.json`. If present and valid JSON with `"version": 1`, read the orchestrator's own settings from the `"gc-review-all"` namespace. Every sub-skill reads its own namespace from the same file (`"a11y"`, `"security"`, `"im"`, `"iam"`, `"branding"`, `"bilingual"`):

```json
{
  "version": 1,
  "gc-review-all": {
    "include": ["a11y", "security", "im", "iam", "branding", "bilingual"],
    "exclude": [],
    "parallel": true
  },
  "a11y": { "...": "see gc-review-a11y/CONFIG.md" },
  "security": { "...": "see gc-review-security/CONFIG.md" },
  "im": { "...": "see gc-review-im/CONFIG.md" },
  "iam": { "...": "see gc-review-iam/CONFIG.md" },
  "branding": { "...": "see gc-review-branding/CONFIG.md" },
  "bilingual": { "...": "see the Configuration section of gc-review-bilingual/SKILL.md" }
}
```

- `include` — Skills to run (default: all six)
- `exclude` — Skills to skip (overrides include)
- `parallel` — Launch agents in parallel when available (default: true)

**Config passthrough:** hold the full loaded JSON in memory. Each per-skill prompt template in Step 3 embeds it verbatim via the line `Project config (from .gc-review/config.json): {json or 'none'}` — each sub-skill picks out its own namespace.

> **Deprecated fallbacks:** gc-review-a11y (legacy `./.a11y/config.json`) and gc-review-bilingual (legacy `.bilingual-review.json`) still honour their old config files for one release, with a deprecation warning in their reports. The orchestrator passes only `.gc-review/config.json`; the sub-skills handle their own legacy fallbacks.

### Step 3: Launch Agent Loop

**Resolve sub-skill paths before dispatch (do not hardcode a repo-relative `skills/…` path):**

1. Locate the directory containing this orchestrator's own SKILL.md. Its parent is the skills root: `{skills_root}`.
2. Each sub-skill lives at `{skills_root}/gc-review-{domain}/SKILL.md`.
3. If a sibling is not found there, fall back to Glob: `**/gc-review-{domain}/SKILL.md` (exclude `node_modules`).
4. If any applicable skill's SKILL.md still cannot be found, **stop before dispatch and report a hard error listing every missing skill**. Mark those domains as **INCOMPLETE** in the final report (Step 4/5) — do not silently skip them.

For each applicable skill, use the **Agent tool** to spawn a review agent with that skill's prompt template below. Every template ends with the same **shared closing block**:

```
Return your findings as a markdown table with columns:
Status | File | Issue Found | Recommended Action

Normalize all status icons to literal ❌/⚠️/✅ and count ❌ (Fail/Critical) findings as critical.

End with a summary line:
SUMMARY: {critical_count} critical, {warning_count} warnings, {pass_count} passes
```

#### Template — gc-review-a11y

```
You are running a GC compliance review sub-task: gc-review-a11y (accessibility).

Read the skill file at: {skills_root}/gc-review-a11y/SKILL.md

Review scope (resolved by the orchestrator): {staged | unstaged | branch-diff | full-codebase}
Changed files: {file list}
Project config (from .gc-review/config.json): {json or 'none'}

This skill is diff-based. Follow its workflow:
- Step 1: Detect Changes — use the scope above instead of re-deriving it
- Step 2: Load Project Configuration — use the config above (a11y options are under the "a11y" namespace)
- Step 3: Gather Context
- Step 4: Run Accessibility Analysis
- Step 5: Present Findings — this is the findings table
- SKIP Step 6 (Fix Selection) — report only
- Step 7: Summary — still produce the Summary counts

{shared closing block}
```

#### Template — gc-review-security

```
You are running a GC compliance review sub-task: gc-review-security (Protected B security).

Read the skill file at: {skills_root}/gc-review-security/SKILL.md
Its sibling files are at {skills_root}/gc-review-security/checklist.md, {skills_root}/gc-review-security/report-template.md, and {skills_root}/gc-review-security/CONFIG.md.

Review scope: {changed-file list, or 'entire codebase' when the resolved scope is full-codebase} — this skill reviews the whole codebase or the provided scope; it is not diff-gated
Project config (from .gc-review/config.json): {json or 'none'}

Follow the skill's 5-item Review Process:
1. Detect changed files — use the scope above instead of re-running git diff
2. Load configuration — use the config above (security options are under the "security" namespace)
3. Evaluate each file in scope against the 6-point security checklist
4. Categorize findings by ITSG-33 control family
5. Output a structured findings table — the findings table lives in the "Output Format" section ("Security Review Results")

There is no fix-selection step in this skill — it is report-only.

{shared closing block}
```

#### Template — gc-review-im

```
You are running a GC compliance review sub-task: gc-review-im (information management).

Read the skill file at: {skills_root}/gc-review-im/SKILL.md

Review scope (resolved by the orchestrator): {staged | unstaged | branch-diff | full-codebase}
Changed files: {file list}
Project config (from .gc-review/config.json): {json or 'none'}

This skill is diff-based. Follow its workflow — note that file identification (Step 2) comes BEFORE config loading (Step 3):
- Step 1: Detect Changes — use the scope above instead of re-deriving it
- Step 2: Identify IM-Relevant Files
- Step 3: Load Configuration — use the config above (im options are under the "im" namespace)
- Step 4: Review for IM Compliance — start with Step 4.0 (classify each model as business-value vs. transitory) before applying Categories A–D
- Step 5: Present Findings — this is the findings table
- Step 6: Summary

There is no fix step in this skill.

{shared closing block}
```

#### Template — gc-review-iam

```
You are running a GC compliance review sub-task: gc-review-iam (identity & authentication).

Read the skill file at: {skills_root}/gc-review-iam/SKILL.md

Review scope: this skill reviews the WHOLE CODEBASE (or a user-specified scope), not the diff — do not attempt change detection
Project config (from .gc-review/config.json): {json or 'none'}

Follow its workflow, Steps 0–9:
- Step 0: Parse Arguments & Load Configuration — use the config above (iam options are under the "iam" namespace; pass --strict only if the user requested strict mode)
- Step 1: Detect Project Context (stack detection)
- Step 2: Identify Authentication-Related Files (across the whole codebase)
- Step 3: OIDC Implementation & Protocol Security Review
- Step 4: MFA Enforcement Review
- Step 5: Session Security Review
- Step 6: Scope & Claim Minimization Review
- Step 7: Logout & Token Revocation Review
- Step 8: RBAC Integration Review
- Step 9: Generate Report — the findings table is section 9.3 (Detailed Findings)

There is no fix step in this skill.

{shared closing block}
```

#### Template — gc-review-branding

```
You are running a GC compliance review sub-task: gc-review-branding (FIP branding & design).

Read the skill file at: {skills_root}/gc-review-branding/SKILL.md

Review scope (resolved by the orchestrator): {staged | unstaged | branch-diff | full-codebase}
Changed files: {file list}
Project config (from .gc-review/config.json): {json or 'none'}

This skill is diff-based. Follow its workflow, Steps 1–10 — note that config (Step 1) comes BEFORE change detection (Step 2), the reverse of most skills:
- Step 1: Load Configuration — use the config above (branding options are under the "branding" namespace)
- Step 2: Detect Changes — use the scope above instead of re-deriving it
- Step 3: Identify Relevant Files
- Step 4: FIP Symbol Check
- Step 5: Language Toggle Check
- Step 5b: Additional Header Elements Check
- Step 6: Typography Audit
- Step 7: Color Token Audit
- Step 8: Component Consistency Check
- Step 9: Footer Structure and Mandatory Links Check
- Step 10: Present Compliance Report — the findings table is in its "Findings" section

There is no fix-selection step in this skill.

{shared closing block}
```

#### Template — gc-review-bilingual

```
You are running a GC compliance review sub-task: gc-review-bilingual (official languages).

Read the skill file at: {skills_root}/gc-review-bilingual/SKILL.md

Review scope (resolved by the orchestrator): {staged | unstaged | branch-diff | full-codebase}
Changed files: {file list}
Project config (from .gc-review/config.json): {json or 'none'}

This skill is diff-based (it scans the full project when there are no changes). Follow its workflow:
- Step 1: Detect Changes & Scope — use the scope above instead of re-deriving it
- Step 2: Detect i18n Framework
- Step 3: Locate Translation Files
- Step 4: Run Rule Checks
- Step 5: Present Results — the findings table is the "Detailed Findings" table
- SKIP Step 6 (Fix Selection) — report only

Bilingual options are under the "bilingual" namespace of the config above.

{shared closing block}
```

**Execution strategy:**
- If the `parallel` config option is true (default), launch all applicable skill agents in parallel for speed using multiple Agent tool calls in a single message.
- If `parallel` is false, run skills sequentially, one agent at a time.
- If the Agent tool is unavailable, run skills sequentially yourself: read each SKILL.md, execute its workflow, then move to the next.

### Step 4: Collect and Normalize Results

As each agent completes, collect its results into a unified structure:

```
For each completed skill:
  - Domain tag: [A11Y], [SECURITY], [IM], [IAM], [BRANDING], [BILINGUAL]
  - Findings table rows (with domain tag prepended)
  - Summary counts (from the SUMMARY: line)
  - Compliance status (PASS / FAIL)
```

**Severity normalization:** sub-skills use domain vocabularies inside their findings. Map them onto the repo severity convention before consolidating:

| Sub-skill vocabulary | Normalized severity |
|---|---|
| `[Security Error: *]` (security), `[Auth Error]` (iam), `[Accessibility Error]` (a11y), `[IM Error]` (im), the label "Critical" (any skill) | ❌ **Fail** |
| `[Security Warning: *]` (security), `[Auth Warning]` (iam), `[Accessibility Warning]` (a11y), `[IM Warning]` (im) | ⚠️ **Warning** |
| Compliant / good-practice rows | ✅ **Pass** |

"Critical" in the `SUMMARY:` line and in this consolidated report is the same level as ❌ **Fail** (the repo severity convention) — the Executive Summary column is therefore titled "❌ Fail", and `critical_count` = the number of ❌ Fail findings.

**Handle agent failures gracefully:**
- If an agent errors out → **retry it once**. If the retry also fails, mark the domain as **⚠️ INCOMPLETE** and note the error.
- If a skill's SKILL.md could not be found in Step 3 → the domain is **⚠️ INCOMPLETE** (path-resolution error).
- If an agent finds no applicable files → mark as **⏭️ Skipped**.
- If an agent completes → record PASS or FAIL.

### Step 5: Present Consolidated Audit Report

Combine all results into this format:

```markdown
# GC Full Compliance Audit Report

**Repository:** {repo name}
**Branch:** {branch}
**Date:** {date}
**Review Scope:** {staged | unstaged | branch-diff | full-codebase}
**Skills Executed:** {count} of 6

---

## Executive Summary

| Domain | Status | ❌ Fail | ⚠️ Warnings | ✅ Passes |
|--------|--------|--------|-------------|----------|
| Accessibility (A11Y) | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| Security | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| Information Management | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| Identity & Access Mgmt | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| Branding & FIP | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| Bilingual (OLA) | PASS/FAIL/SKIPPED/INCOMPLETE | {n} | {n} | {n} |
| **TOTALS** | **{overall}** | **{n}** | **{n}** | **{n}** |

**Overall Compliance:** PASS / FAIL / INCOMPLETE
- **PASS** = zero ❌ Fail findings AND zero INCOMPLETE domains
- **FAIL** = one or more ❌ Fail findings
- **INCOMPLETE** = no ❌ Fail findings but at least one domain errored — name the errored domain(s); an unreviewed domain must never contribute to an overall PASS

---

## Detailed Findings

### Accessibility (A11Y)

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| ... | ... | ... | ... |

### Security

| Status | File | Issue Found | Recommended Action |
| :--- | :--- | :--- | :--- |
| ... | ... | ... | ... |

{Repeat for each domain that ran}

---
```

### Step 6: Generate Prioritized Remediation Plan

After the audit report, produce a remediation plan that helps the team triage and fix issues efficiently:

```markdown
## Remediation Plan

### Priority 1 — Critical Fixes (Must Address)

Issues that block compliance (❌ Fail). Fix these first.

| # | Domain | File | Issue | Fix Description | Effort |
|---|--------|------|-------|-----------------|--------|
| 1 | [SECURITY] | `auth.ts:42` | Hardcoded secret | Move to env var | Low |
| 2 | [A11Y] | `Form.tsx:18` | Missing label | Add `<label>` | Low |
| ... | | | | | |

### Priority 2 — Warnings (Should Address)

Issues that risk non-compliance or degrade quality.

| # | Domain | File | Issue | Fix Description | Effort |
|---|--------|------|-------|-----------------|--------|
| ... | | | | | |

### Priority 3 — Recommendations

Improvements that strengthen compliance posture.

| # | Domain | Recommendation |
|---|--------|---------------|
| ... | | |
```

**Effort estimation heuristic:**
- **Low** — Single-line or single-file change (add an attribute, rename a variable, add a label)
- **Medium** — Multi-file change or requires testing (refactor auth flow, add i18n keys, restructure headings)
- **High** — Architectural change or cross-cutting concern (implement RBAC, add retention framework, restructure component hierarchy)

**Ordering rules within each priority tier:**
1. Security issues first (they carry the highest risk)
2. Then by effort: Low → Medium → High (maximize quick wins)
3. Group related issues (e.g., all missing labels together, all missing translations together)

### Step 7: Offer Next Steps

Use AskUserQuestion to ask:

**Question:** "How would you like to proceed with the remediation plan?"

**Options:**
1. "Fix all critical issues" — Walk through each Priority 1 fix, showing before/after code, confirming before each edit
2. "Fix by domain" — Choose a specific domain to remediate first
3. "Export report" — Save the full audit report and remediation plan to a markdown file
4. "None (just report)" — Keep as reference, no fixes

If the user chooses to fix by domain, use AskUserQuestion to present the applicable domains.

**Fix protocol:** if the user chooses to fix issues, use the Agent tool to spawn fix agents with an **explicit tool grant of Read, Edit, and AskUserQuestion**. Fix agents follow *this orchestrator's* fix protocol — independent of the domain skill's own `allowed-tools` (most domain skills are report-only and declare no Edit):

1. **Always show the proposed change first** (before/after code)
2. Use AskUserQuestion to confirm: "Apply this fix?" (Yes / Skip / Stop)
3. Only apply the change using the Edit tool after the user confirms
4. Note what was changed in the final summary

Never auto-apply fixes.

If the user chooses to export, use Bash to create the output directory (`mkdir -p .gc-review/reports`) and the Write tool to save the report. **Filename:** `.gc-review/reports/audit-YYYY-MM-DD.md` (ISO date, e.g., `audit-2026-07-10.md`). If the file already exists, append `-2`, `-3`, … (e.g., `audit-2026-07-10-2.md`) — never overwrite a same-day report. Note: teams that gitignore `.gc-review/` should copy exported reports elsewhere to preserve audit history.

---

## Output Files

When the user requests an export, save to:

```
.gc-review/
├── config.json                (project config, if present)
└── reports/
    └── audit-YYYY-MM-DD.md    (full audit report + remediation plan; -2, -3 suffix on collision)
```

---

## Disclaimer / <span lang="fr">Avis de non-responsabilité</span>

> This is an automated pattern-based review and does not constitute a formal compliance assessment. Findings should be validated by qualified assessors through the appropriate Security Assessment and Authorization (SA&A) process before being used for compliance reporting. Individual sub-skill disclaimers also apply.

<div lang="fr">

> Il s'agit d'un examen automatisé basé sur des modèles et ne constitue pas une évaluation formelle de la conformité. Les conclusions doivent être validées par des évaluateurs qualifiés dans le cadre du processus approprié d'évaluation et d'autorisation de sécurité (EAS) avant d'être utilisées pour les rapports de conformité. Les avis de non-responsabilité des sous-compétences individuelles s'appliquent également.

</div>

---

## Remember

Your goal is a **single-pass, comprehensive compliance picture**. The value of this skill over running individual reviews is:

1. **Completeness** — No domain is accidentally skipped, and an errored domain is surfaced as INCOMPLETE rather than silently passing
2. **Cross-domain visibility** — A security issue might relate to an IAM finding; a branding issue might overlap with bilingual requirements
3. **Prioritized action** — One ranked remediation list instead of six separate reports
4. **Effort estimation** — Helps teams plan sprints and allocate work

Run every applicable skill. Consolidate ruthlessly. Prioritize clearly. Make it actionable.
