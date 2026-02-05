# GC Code Review Skills

Claude Code skills for Government of Canada web application development.

## Available Skills

### gc-review-branding

Review code for Government of Canada branding compliance.

**Invoke with:** `/gc-review-branding`

**What it checks:**
- Federal Identity Program (FIP) symbols (GC Signature, Canada Wordmark)
- Language toggle (EN/FR) implementation
- Typography compliance (Noto Sans)
- GC Design System color tokens
- Standard GC components vs custom implementations
- Mandatory footer links (Privacy, Terms, Contact)
- Date Modified format (YYYY-MM-DD)

**Configuration:** See `gc-review-branding/CONFIG.md`

## Installation

To install these skills for use with Claude Code:

```bash
# Clone the repository
git clone <repo-url> ~/.agents/skills/gc-code-skills

# Or symlink individual skills
ln -s /path/to/gc-review-branding ~/.claude/skills/gc-review-branding
```

## References

- [Federal Identity Program Manual](https://www.canada.ca/en/treasury-board-secretariat/services/government-communications/federal-identity-program/manual.html)
- [GC Design System](https://design.canada.ca/)
- [Policy on Communications and Federal Identity](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=30683)
