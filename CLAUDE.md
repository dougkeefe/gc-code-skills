# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

gc-code-skills is a collection of Claude Code skills that review code for compliance with Government of Canada (GoC) digital standards. Each skill is a standalone markdown file (SKILL.md) with YAML frontmatter — there is no build system, test runner, or compiled code.

Installation: `npx skills add dougkeefe/gc-code-skills`

## Repository Structure

- `skills/gc-review-*/SKILL.md` — Skill definitions (the core of the project)
- `skills/gc-review-*/CONFIG.md` — Configuration schema docs (gc-review-im, gc-review-branding)
- `.claude-plugin/` — Plugin metadata for Claude Code marketplace

Seven skills exist: `gc-review-a11y`, `gc-review-security`, `gc-review-im`, `gc-review-iam`, `gc-review-branding`, `gc-review-bilingual`, `gc-review-all`.

## Skill File Format

Each SKILL.md follows this structure:

```yaml
---
name: gc-review-[domain]
description: [What the skill reviews]
allowed-tools: [Comma-separated list: Read, Grep, Glob, Bash, Edit, Write, Agent, AskUserQuestion]
---
```

Followed by markdown containing:
1. Role context and policy driver references (with statute citations and effective dates)
2. Step-by-step workflow (detect changes → load config → gather context → analyze → report)
3. Detailed rules with detection patterns, pass/fail criteria, and severity levels
4. Output format specification with markdown tables
5. Disclaimer stating the review is automated and not a formal compliance assessment

## Key Conventions

- **Policy references must include citations**: statute numbers (e.g., `R.S.C. 1985, c. P-21`), effective dates, and a `Last Verified` date field.
- **Detection patterns are guidance, not literal regex**: SKILL.md patterns are labeled "indicative" — the reviewing agent should read actual code for context rather than blindly matching.
- **Fix workflows require user confirmation**: Skills that apply fixes (currently gc-review-a11y) must show proposed changes and use AskUserQuestion before editing. Never auto-apply fixes.
- **Configuration path**: Skills that support project-level config read from `.gc-review/config.json`. The config uses `"version": 1` as a required field.
- **allowed-tools must list every tool the skill uses**, including AskUserQuestion if the workflow calls it.
- **Severity levels**: ❌ Fail (must fix), ⚠️ Warning (should address), ✅ Pass (compliant).
- **Agent tool for orchestration**: `gc-review-all` uses the `Agent` tool to spawn sub-agents for each domain skill. Individual domain skills do not use `Agent`.
- **Bilingual**: README sections include French translations. Contributions in both English and French are welcome.

## No Build/Test/Lint

This is a documentation-only repository. There are no commands to build, test, or lint. Changes are reviewed via pull requests.
