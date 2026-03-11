# GC Code Review Skills / <span lang="fr">Compétences de revue de code GC</span>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE.md)

A collection of Claude Code skills that help developers review code for compliance with Government of Canada (GoC) digital standards. These skills automate compliance checking across accessibility, security, information management, identity and authentication, official languages, and GC branding requirements.

<div lang="fr">

Une collection de compétences Claude Code qui aident les développeurs à vérifier la conformité du code aux normes numériques du gouvernement du Canada (GC). Ces compétences automatisent la vérification de la conformité en matière d'accessibilité, de sécurité, de gestion de l'information, d'identité et d'authentification, de langues officielles et d'image de marque du GC.

</div>

---

## Disclaimer / <span lang="fr">Avis de non-responsabilité</span>

This project is not affiliated with, endorsed by, or sponsored by the Government of Canada. The information provided is sourced from publicly available Government of Canada policies and standards and is provided for educational and informational purposes only. This tool does not replace formal Security Assessment and Authorization (SA&A) processes.

<div lang="fr">
Ce projet n'est pas affilié, endossé ou parrainé par le gouvernement du Canada. Les informations fournies proviennent de politiques et de normes du gouvernement du Canada accessibles au public et sont fournies à des fins éducatives et informatives uniquement. Cet outil ne remplace pas les processus formels d'évaluation et d'autorisation de sécurité (EAS).
</div>

---

## Available Skills / <span lang="fr">Compétences disponibles</span>

### `gc-review-a11y`
Reviews code for WCAG 2.2 Level AA compliance. Checks semantic HTML, ARIA patterns, focus management, text alternatives, and visual integrity following CAN/ASC - EN 301 549:2024.

<div lang="fr">

Vérifie la conformité du code au niveau AA des WCAG 2.2. Contrôle le HTML sémantique, les patrons ARIA, la gestion du focus, les alternatives textuelles et l'intégrité visuelle selon CAN/ASC - EN 301 549:2024.

</div>

### `gc-review-security`
Reviews code for Protected B security compliance. Evaluates access control, input validation, encryption, secrets management, and logging against ITSG-33 controls.

<div lang="fr">

Vérifie la conformité du code aux exigences de sécurité Protégé B. Évalue le contrôle d'accès, la validation des entrées, le chiffrement, la gestion des secrets et la journalisation selon les contrôles ITSG-33.

</div>

### `gc-review-im`
Reviews database schemas and data access code for information management compliance. Checks mandatory metadata fields, retention policies, soft deletes, and audit requirements per the Directive on Service and Digital.

<div lang="fr">

Vérifie les schémas de bases de données et le code d'accès aux données pour la conformité en gestion de l'information. Contrôle les champs de métadonnées obligatoires, les politiques de conservation, la suppression douce et les exigences d'audit selon la Directive sur les services et le numérique.

</div>

### `gc-review-iam`
Reviews authentication implementations for GoC identity standards. Checks OIDC configuration, session security, scope minimization, and RBAC integration against the Directive on Identity Management and Standard on Identity and Credential Assurance.

<div lang="fr">

Vérifie les implémentations d'authentification selon les normes d'identité du GC. Contrôle la configuration OIDC, la sécurité des sessions, la minimisation des portées et l'intégration RBAC selon la Directive sur la gestion de l'identité et la Norme sur l'assurance de l'identité et des justificatifs.

</div>

### `gc-review-branding`
Reviews code for Federal Identity Program compliance. Verifies GC signatures, Canada wordmark usage, typography, color tokens, and mandatory footer elements per the Policy on Communications and Federal Identity.

<div lang="fr">

Vérifie la conformité du code au Programme de coordination de l'image de marque. Contrôle les signatures du GC, l'utilisation du mot-symbole « Canada », la typographie, les jetons de couleur et les éléments obligatoires du pied de page selon la Politique sur les communications et l'image de marque.

</div>

### `gc-review-bilingual`
Reviews code for Official Languages Act compliance. Checks for hardcoded strings, translation file parity, locale-aware routing, and equal prominence of English and French content.

<div lang="fr">

Vérifie la conformité du code à la Loi sur les langues officielles. Contrôle les chaînes codées en dur, la parité des fichiers de traduction, le routage selon la langue et la proéminence égale du contenu en anglais et en français.

</div>

---

## Installation

```bash
npx skills add dougkeefe/gc-code-skills
```

<div lang="fr">

Installez les compétences en exécutant la commande ci-dessus dans votre terminal.

</div>

---

## Quick Start / <span lang="fr">Démarrage rapide</span>

Invoke a skill during a Claude Code session:

<div lang="fr">

Invoquez une compétence pendant une session Claude Code :

</div>

```
/gc-review-a11y
/gc-review-security
/gc-review-branding
```

---

## Configuration

Some skills support project-level configuration via `.gc-review/config.json`. This lets you tailor reviews to your project's specific requirements.

<div lang="fr">

Certaines compétences prennent en charge une configuration au niveau du projet via `.gc-review/config.json`. Cela vous permet d'adapter les revues aux exigences spécifiques de votre projet.

</div>

```bash
mkdir -p .gc-review
cat > .gc-review/config.json << 'EOF'
{
  "version": 1
}
EOF
```

### Supported configuration by skill / <span lang="fr">Configuration prise en charge par compétence</span>

#### `gc-review-iam`

| Field | Type | Description |
|-------|------|-------------|
| `additionalIdPs` | `array` | Additional approved identity providers beyond the defaults (Entra ID, GCKey, Sign-In Canada). Each entry has `name` and `issuer` fields. |

```json
{
  "additionalIdPs": [
    { "name": "Departmental ADFS", "issuer": "adfs.department.gc.ca" }
  ]
}
```

#### `gc-review-im`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `requiredMetadata` | `string[]` | `["record_id", "creator_id", "date_created", "language", "classification"]` | Metadata fields required on business record models |
| `softDeleteFields` | `string[]` | `["deleted_at", "is_deleted", "archived_at"]` | Field names that indicate soft delete support |
| `retentionFields` | `string[]` | `["retention_schedule", "disposition_date", "retention_code", "retention_period"]` | Field names indicating retention/disposition support |
| `exclude` | `string[]` | `[]` | Glob patterns for files to exclude |
| `strictMode` | `boolean` | `false` | When true, warnings are treated as errors |

See [`skills/gc-review-im/CONFIG.md`](skills/gc-review-im/CONFIG.md) for the full schema.

#### `gc-review-branding`

| Field | Type | Description |
|-------|------|-------------|
| `department` | `string` | Department name for reporting |
| `signature.altText` | `string` | Expected alt text for GC signature |
| `signature.componentName` | `string` | Custom component name for GC signature |
| `wordmark.componentName` | `string` | Custom component name for Canada wordmark |
| `additionalColors` | `object` | Extra approved color tokens (map of token name to hex value) |
| `excludePatterns` | `string[]` | File patterns to skip |

---

## Official Languages / <span lang="fr">Langues officielles</span>

Contributions in English and French are welcome.

<span lang="fr">Les contributions en anglais et en français sont les bienvenues.</span>

---

## License / <span lang="fr">Licence</span>

MIT - see [LICENSE.md](LICENSE.md)

<div lang="fr">

MIT - voir [LICENSE.md](LICENSE.md)

</div>
