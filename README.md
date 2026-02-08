# GC Code Review Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE.md)

A collection of Claude Code skills that help developers review code for compliance with Government of Canada (GoC) digital standards. These skills automate compliance checking across accessibility, security, information management, identity and authentication, official languages, and GC branding requirements.

---

## Disclaimer / Avis de non-responsabilité

This project is not affiliated with, endorsed by, or sponsored by the Government of Canada. The information provided is sourced from publicly available Government of Canada policies and standards and is provided for educational and informational purposes only. This tool does not replace formal Security Assessment and Authorization (SA&A) processes.

Ce projet n'est pas affilié, endossé ou parrainé par le gouvernement du Canada. Les informations fournies proviennent de politiques et de normes du gouvernement du Canada accessibles au public et sont fournies à des fins éducatives et informatives uniquement. Cet outil ne remplace pas les processus formels d'évaluation et d'autorisation de sécurité (EAS).

---

## Available Skills

### `gc-review-a11y`
Reviews code for WCAG 2.1 Level AA compliance. Checks semantic HTML, ARIA patterns, focus management, text alternatives, and visual integrity following the Standard on Web Accessibility.

### `gc-review-security`
Reviews code for Protected B security compliance. Evaluates access control, input validation, encryption, secrets management, and logging against ITSG-33 controls.

### `gc-review-im`
Reviews database schemas and data access code for information management compliance. Checks mandatory metadata fields, retention policies, soft deletes, and audit requirements per the Directive on Service and Digital.

### `gc-review-iam`
Reviews authentication implementations for GoC identity standards. Checks OIDC configuration, session security, scope minimization, and RBAC integration against TBS security guidelines.

### `gc-review-branding`
Reviews code for Federal Identity Program compliance. Verifies GC signatures, Canada wordmark usage, typography, color tokens, and mandatory footer elements.

### `gc-review-bilingual`
Reviews code for Official Languages Act compliance. Checks for hardcoded strings, translation file parity, locale-aware routing, and equal prominence of English and French content.

---

## Installation

```bash
npx skills add dougkeefe/gc-code-skills
```

---

## Quick Start

Invoke a skill during a Claude Code session:

```
/gc-review-a11y
/gc-review-security
/gc-review-branding
```

---

## Official Languages / Langues officielles

Contributions in English and French are welcome.

Les contributions en anglais et en français sont les bienvenues.

---

## License

MIT - see [LICENSE.md](LICENSE.md)
