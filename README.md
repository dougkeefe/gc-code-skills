# GC Code Review Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE.md)
[![GoC Standards](https://img.shields.io/badge/GoC-Standards%20Compliant-red.svg)]()
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-blueviolet.svg)]()
[![EN/FR](https://img.shields.io/badge/lang-EN%20%7C%20FR-blue.svg)]()

A collection of Claude Code Skills that help developers review code for compliance with Government of Canada (GoC) digital standards. These skills automate compliance checking across accessibility, security, information management, identity and authentication, official languages, and GC branding requirements.

---

## Important Disclaimer / Avis important

> **English**: This project provides automated code review assistance for Government of Canada compliance standards. It does **not** replace the formal Security Assessment and Authorization (SA&A) process. All applications deployed in GoC production environments must complete the appropriate SA&A process as required by departmental policies and Treasury Board directives.

> **Français**: Ce projet fournit une assistance automatisée pour la révision de code selon les normes de conformité du gouvernement du Canada. Il ne remplace **pas** le processus formel d'évaluation et d'autorisation de sécurité (EAS). Toutes les applications déployées dans les environnements de production du GC doivent compléter le processus EAS approprié, conformément aux politiques ministérielles et aux directives du Conseil du Trésor.

---

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed and configured
- Valid Anthropic API key or Claude Pro/Team subscription
- Git (for cloning the repository)

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone https://github.com/dougkeefe/gc-code-skills.git
   cd gc-code-skills
   ```

2. **Add skills to Claude Code:**
   ```bash
   # Add individual skills
   claude skills add ./skills/accessibility-reviewer
   claude skills add ./skills/security-reviewer

   # Or add all skills at once
   claude skills add ./skills/*
   ```

3. **Verify installation:**
   ```bash
   claude skills list
   ```

### Usage

Invoke a skill during a Claude Code session:

```
/accessibility-review src/components/
/security-review --scope=auth
/gc-compliance --full
```

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key | Yes* |
| `GC_SKILLS_LANG` | Preferred output language (`en` or `fr`) | No |
| `GC_SKILLS_STRICT` | Enable strict compliance mode | No |

*Not required if using Claude Pro/Team subscription with authenticated CLI.

---

## Compliance Pillars

This skills collection covers six compliance domains aligned with GoC digital standards:

### 1. Accessibility (`/accessibility-review`)

Reviews code for compliance with:
- [Web Content Accessibility Guidelines (WCAG) 2.1 AA](https://www.w3.org/WAI/WCAG21/quickref/)
- [Standard on Web Accessibility](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=23601)
- [Accessible Canada Act](https://laws-lois.justice.gc.ca/eng/acts/A-0.6/) requirements

**Checks include:** Semantic HTML, ARIA attributes, color contrast, keyboard navigation, screen reader compatibility, focus management.

[View Skill Specification](./skills/accessibility-reviewer/SPEC.md)

---

### 2. Security (`/security-review`)

Reviews code for compliance with:
- [ITSG-33 Security Controls](https://www.cyber.gc.ca/en/guidance/it-security-risk-management-lifecycle-approach-itsg-33)
- [Directive on Security Management](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=32611)
- OWASP Top 10 vulnerabilities

**Checks include:** Input validation, authentication flows, encryption usage, secrets management, dependency vulnerabilities, secure headers.

[View Skill Specification](./skills/security-reviewer/SPEC.md)

---

### 3. Information Management (`/info-management-review`)

Reviews code for compliance with:
- [Directive on Service and Digital](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=32601)
- [Policy on Information Management](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=12742)
- [Library and Archives Canada Act](https://laws-lois.justice.gc.ca/eng/acts/L-7.7/) requirements

**Checks include:** Data retention policies, metadata standards, records management, data classification, PII handling, soft delete implementation.

[View Skill Specification](./skills/info-management-reviewer/SPEC.md)

---

### 4. Identity and Authentication (`/identity-auth-review`)

Reviews code for compliance with:
- [Directive on Identity Management](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=16577)
- [Standard on Identity and Credential Assurance](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=26776)
- GC Sign-In / Credential Broker integration patterns

**Checks include:** OIDC implementation, session management, credential handling, identity federation, RBAC patterns, MFA implementation.

[View Skill Specification](./skills/identity-auth-reviewer/SPEC.md)

---

### 5. GC Branding (`/gc-branding-review`)

Reviews code for compliance with:
- [Federal Identity Program](https://www.canada.ca/en/treasury-board-secretariat/services/government-communications/federal-identity-program.html)
- [Canada.ca Design System](https://design.canada.ca/)
- Official GoC visual identity standards

**Checks include:** Canada wordmark usage, official colors, typography, layout patterns, header/footer requirements, GC Design System components.

[View Skill Specification](./skills/gc-branding-reviewer/SPEC.md)

---

### 6. Official Languages (`/official-languages-review`)

Reviews code for compliance with:
- [Official Languages Act](https://laws-lois.justice.gc.ca/eng/acts/o-3.01/)
- [Directive on Official Languages for Communications and Services](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=26164)

**Checks include:** Bilingual UI strings, language toggle implementation, equal prominence, i18n framework integration, EN/FR parity validation.

[View Skill Specification](./skills/official-languages-reviewer/SPEC.md)

---

## Development

### Repository Structure

```
gc-code-skills/
├── skills/
│   ├── accessibility-reviewer/
│   │   ├── SPEC.md
│   │   ├── skill.json
│   │   └── prompts/
│   ├── security-reviewer/
│   ├── info-management-reviewer/
│   ├── identity-auth-reviewer/
│   ├── gc-branding-reviewer/
│   └── official-languages-reviewer/
├── docs/
├── .devcontainer/
├── LICENSE.md
├── SECURITY.md
├── CONTRIBUTING.md
└── README.md
```

### Containerized Development Environment

This project includes a devcontainer configuration for consistent development:

```bash
# Using VS Code
code --folder-uri vscode-remote://dev-container+$(pwd)

# Using GitHub Codespaces
gh codespace create --repo dougkeefe/gc-code-skills
```

### Pre-commit Hooks

We use pre-commit hooks to ensure quality:

```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

**Configured hooks:**
- Markdown linting
- YAML validation
- Trailing whitespace removal
- End-of-file fixer

---

## Contributing

We welcome contributions from the Government of Canada developer community and beyond.

Please read our [Contributing Guidelines](CONTRIBUTING.md) before submitting pull requests.

### Ways to Contribute

- Report issues or suggest improvements
- Submit pull requests for new compliance checks
- Improve documentation
- Add translations (English/French)
- Share feedback on skill accuracy

**Important:** Any UI-facing changes must include both English and French updates.

---

## Security

For security vulnerabilities, please review our [Security Policy](SECURITY.md).

**Do not** open public issues for security vulnerabilities. Follow the responsible disclosure process outlined in SECURITY.md.

---

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

---

## Official Languages / Langues officielles

This project welcomes contributions in both English and French, Canada's official languages. Issues and Pull Requests are welcome in either official language.

Ce projet accueille les contributions en anglais et en français, les langues officielles du Canada. Les issues et les Pull Requests sont les bienvenues dans l'une ou l'autre des langues officielles.

---

## Contact

- **Issues**: [GitHub Issues](https://github.com/dougkeefe/gc-code-skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/dougkeefe/gc-code-skills/discussions)
