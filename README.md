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

Policy references used by these skills are catalogued in [REFERENCES.md](REFERENCES.md). / <span lang="fr">Les références de politiques utilisées par ces compétences sont cataloguées dans [REFERENCES.md](REFERENCES.md).</span>

---

## Available Skills / <span lang="fr">Compétences disponibles</span>

### `gc-review-a11y`
Reviews code against WCAG 2.2 Level AA as its baseline, exceeding the WCAG 2.1 AA minimum incorporated by CAN/ASC - EN 301 549:2024. Checks semantic HTML, ARIA patterns, focus management, text alternatives, contrast, visual integrity, and the new WCAG 2.2 criteria.

<div lang="fr">

Vérifie le code selon le niveau AA des WCAG 2.2 comme référence, dépassant le minimum WCAG 2.1 AA intégré par la norme CAN/ASC - EN 301 549:2024. Contrôle le HTML sémantique, les patrons ARIA, la gestion du focus, les alternatives textuelles, le contraste, l'intégrité visuelle et les nouveaux critères des WCAG 2.2.

</div>

### `gc-review-security`
Reviews code for Protected B security compliance. Evaluates access control, input validation, PII handling, cryptography (per CCCS ITSP.40.111 v5, including post-quantum milestones), security headers, secrets management, and audit logging against ITSG-33 controls and GC web configuration requirements.

<div lang="fr">

Vérifie la conformité du code aux exigences de sécurité Protégé B. Évalue le contrôle d'accès, la validation des entrées, le traitement des renseignements personnels, la cryptographie (selon l'ITSP.40.111 v5 du CCC, y compris les jalons post-quantiques), les en-têtes de sécurité, la gestion des secrets et la journalisation d'audit selon les contrôles ITSG-33 et les exigences de configuration Web du GC.

</div>

### `gc-review-im`
Reviews database schemas and data access code for information management compliance. Checks a metadata baseline (per the LAC/TBS digital archival records metadata standards), disposition-aware retention and deletion, searchability, and audit requirements per the Directive on Service and Digital.

<div lang="fr">

Vérifie les schémas de bases de données et le code d'accès aux données pour la conformité en gestion de l'information. Contrôle une base de référence de métadonnées (selon les normes de métadonnées BAC/SCT pour les documents d'archives numériques), la conservation et la suppression tenant compte de la disposition, la repérabilité et les exigences d'audit selon la Directive sur les services et le numérique.

</div>

### `gc-review-iam`
Reviews authentication implementations for GoC identity standards. Checks OIDC configuration, OAuth protocol security (PKCE, state/nonce, redirect URIs), MFA enforcement, session security, token validation, scope minimization, and RBAC integration against the Directive on Identity Management (including Appendix A: Standard on Identity and Credential Assurance), ITSG-33, and ITSP.30.031.

<div lang="fr">

Vérifie les implémentations d'authentification selon les normes d'identité du GC. Contrôle la configuration OIDC, la sécurité du protocole OAuth (PKCE, state/nonce, URI de redirection), l'application de l'authentification multifacteur, la sécurité des sessions, la validation des jetons, la minimisation des portées et l'intégration RBAC selon la Directive sur la gestion de l'identité (y compris l'annexe A : Norme sur l'assurance de l'identité et des justificatifs), l'ITSG-33 et l'ITSP.30.031.

</div>

### `gc-review-branding`
Reviews code for Federal Identity Program compliance. Verifies GC signatures, Canada wordmark usage, typography, color tokens, and mandatory header/footer elements per the Policy on Communications and Federal Identity, and recognizes official GC Design System (GCDS) components as compliant.

<div lang="fr">

Vérifie la conformité du code au Programme de coordination de l'image de marque. Contrôle les signatures du GC, l'utilisation du mot-symbole « Canada », la typographie, les jetons de couleur et les éléments obligatoires de l'en-tête et du pied de page selon la Politique sur les communications et l'image de marque, et reconnaît les composants officiels du Système de design GC (GCDS) comme conformes.

</div>

### `gc-review-bilingual`
Reviews code for Official Languages Act compliance. Checks for hardcoded strings, translation file parity, locale-aware routing and formatting, active offer and language-toggle wording, and equal prominence of English and French content.

<div lang="fr">

Vérifie la conformité du code à la Loi sur les langues officielles. Contrôle les chaînes codées en dur, la parité des fichiers de traduction, le routage et le formatage selon la langue, l'offre active et le libellé du sélecteur de langue, ainsi que la proéminence égale du contenu en anglais et en français.

</div>

### `gc-review-all`
Runs all six GC compliance review skills and produces a consolidated audit report with a prioritized remediation plan covering accessibility, security, information management, identity & authentication, branding, and bilingual compliance.

<div lang="fr">

Lance les six compétences de revue de conformité du GC et produit un rapport d'audit consolidé avec un plan de remédiation priorisé couvrant l'accessibilité, la sécurité, la gestion de l'information, l'identité et l'authentification, l'image de marque et la conformité bilingue.

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
/gc-review-all          # full audit across all domains
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

#### `gc-review-a11y`

Options live under the `"a11y"` namespace in `.gc-review/config.json`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `a11y.wcagLevel` | `string` | `"AA"` | WCAG 2.2 conformance level to review against (`"AA"` or `"AAA"`) |
| `a11y.requireSkipLink` | `boolean` | `true` | Flag layouts missing a "skip to main content" link |
| `a11y.minContrastRatio` | `number` | `4.5` | Minimum contrast ratio for normal text (SC 1.4.3) |
| `a11y.strictMode` | `boolean` | `false` | When true, warnings are treated as errors |
| `a11y.exclude` | `string[]` | `[]` | Glob patterns for files to exclude |

```json
{
  "version": 1,
  "a11y": {
    "wcagLevel": "AA",
    "requireSkipLink": true,
    "minContrastRatio": 4.5
  }
}
```

See [`skills/gc-review-a11y/CONFIG.md`](skills/gc-review-a11y/CONFIG.md) for the full schema.

#### `gc-review-security`

Options are namespaced under a `"security"` key:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `security.piiMetadataConvention` | `string[]` | unset | Annotation names the project uses to mark PII fields (e.g., `isPII`, `@Sensitive`). If unset, missing PII metadata is a warning instead of a failure. |
| `security.auditLogService` | `string` | unset | Identifier of the sanctioned audit logging service. If unset, console logging of security events is a warning, not a failure. |
| `security.approvedAlgorithmExceptions` | `string[]` | `[]` | Algorithms with a documented, assessor-approved exception; findings are downgraded to warnings. |
| `security.strictMode` | `boolean` | `false` | When true, warnings are treated as failures. |

```json
{
  "version": 1,
  "security": {
    "piiMetadataConvention": ["isPII"],
    "auditLogService": "auditLog"
  }
}
```

See [`skills/gc-review-security/CONFIG.md`](skills/gc-review-security/CONFIG.md) for the full schema.

#### `gc-review-iam`

All options live under the `"iam"` namespace.

| Field | Type | Description |
|-------|------|-------------|
| `iam.additionalIdPs` | `array` | Additional recognized identity providers beyond the defaults (GCKey, GCCF federation broker, CanadaLogin, enterprise directory/Entra ID surface). Each entry has `name` and `issuer` fields. |
| `iam.assuranceLevel` | `number` | Level of Assurance (1–4) per Appendix A of the Directive on Identity Management. Drives session-lifetime ceilings (ITSP.30.031 v3) and MFA enforcement checks. |
| `iam.publicFacing` | `boolean` | Declares a public-facing service. Consumer IdPs then produce a ⚠️ Warning (verify GCCF/CanadaLogin brokering) instead of a ❌ Fail. |
| `iam.strictMode` | `boolean` | When true, warnings are treated as failures (equivalent to `--strict`). |

```json
{
  "version": 1,
  "iam": {
    "additionalIdPs": [
      { "name": "Departmental ADFS", "issuer": "adfs.department.gc.ca" }
    ],
    "assuranceLevel": 2,
    "publicFacing": false,
    "strictMode": false
  }
}
```

See [`skills/gc-review-iam/CONFIG.md`](skills/gc-review-iam/CONFIG.md) for the full schema.

<div lang="fr">

Toutes les options se trouvent sous l'espace de noms `"iam"`.

| Champ | Type | Description |
|-------|------|-------------|
| `iam.additionalIdPs` | `array` | Fournisseurs d'identité reconnus supplémentaires au-delà des services par défaut (CléGC, courtier de fédération GCCF, CanadaLogin, annuaire d'entreprise/Entra ID). Chaque entrée comporte les champs `name` et `issuer`. |
| `iam.assuranceLevel` | `number` | Niveau d'assurance (1–4) selon l'annexe A de la Directive sur la gestion de l'identité. Détermine les plafonds de durée de session (ITSP.30.031 v3) et les vérifications d'authentification multifacteur. |
| `iam.publicFacing` | `boolean` | Déclare un service destiné au public. Les fournisseurs d'identité grand public produisent alors un avertissement ⚠️ (vérifier le courtage GCCF/CanadaLogin) au lieu d'un échec ❌. |
| `iam.strictMode` | `boolean` | Si vrai, les avertissements sont traités comme des échecs (équivalent à `--strict`). |

Voir [`skills/gc-review-iam/CONFIG.md`](skills/gc-review-iam/CONFIG.md) pour le schéma complet.

</div>

#### `gc-review-im`

All `gc-review-im` options live under the `"im"` namespace key in `.gc-review/config.json` (alongside the required top-level `"version": 1`).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `im.requiredMetadata` | `string[]` | `["record_id", "creator_id", "date_created", "language", "classification"]` | Metadata baseline fields checked on business-value record models (⚠️ Warning design guidance) |
| `im.softDeleteFields` | `string[]` | `["deleted_at", "is_deleted", "archived_at"]` | Field names that indicate soft delete support |
| `im.retentionFields` | `string[]` | `["retention_schedule", "disposition_date", "retention_code", "retention_period"]` | Field names indicating retention/disposition support |
| `im.exclude` | `string[]` | `[]` | Glob patterns for files to exclude |
| `im.includePatterns` | `string[]` | `[]` | Additional glob patterns for files to include |
| `im.strictMode` | `boolean` | `false` | When true, warnings are treated as errors |
| `im.transitoryPatterns` | `string[]` | `["*_sessions", "*_jobs", "*_cache", "*_tokens", "*_queue", "*_join"]` | Model/table name patterns classified as transitory (exempt from the metadata baseline) |
| `im.businessRecordPatterns` | `string[]` | `[]` | Model/table name patterns always classified as business-value records |

```json
{
  "version": 1,
  "im": {
    "exclude": ["**/test/**"]
  }
}
```

See [`skills/gc-review-im/CONFIG.md`](skills/gc-review-im/CONFIG.md) for the full schema.

#### `gc-review-branding`

All options live under the `"branding"` key. / <span lang="fr">Toutes les options se trouvent sous la clé `"branding"`.</span>

| Field | Type | Description |
|-------|------|-------------|
| `branding.department` | `string` | Department name for reporting |
| `branding.signature.altText` | `string` | Expected alt text for GC signature |
| `branding.signature.componentName` | `string` | Custom component name for GC signature |
| `branding.wordmark.componentName` | `string` | Custom component name for Canada wordmark |
| `branding.additionalColors` | `object` | Extra approved color tokens (map of token name to hex value) |
| `branding.additionalFonts` | `string[]` | Extra approved font families / <span lang="fr">Familles de polices supplémentaires approuvées</span> |
| `branding.exclude` | `string[]` | Glob patterns for files to skip |
| `branding.strictMode` | `boolean` | Treat warnings as failures / <span lang="fr">Traiter les avertissements comme des échecs</span> |

```json
{
  "version": 1,
  "branding": {
    "department": "Parks Canada",
    "additionalFonts": ["Montserrat"],
    "strictMode": false
  }
}
```

See [`skills/gc-review-branding/CONFIG.md`](skills/gc-review-branding/CONFIG.md) for the full schema.

#### `gc-review-bilingual`

Options are nested under the `"bilingual"` key in `.gc-review/config.json`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `translationFiles` | `object` | auto-detected | Glob patterns for translation files, with `english` and `french` arrays |
| `scanPaths` | `string[]` | framework-based | Glob patterns for UI source files to scan |
| `excludePaths` | `string[]` | tests, `node_modules` | Glob patterns for files to exclude |
| `framework` | `string` | `"auto"` | i18n framework (`next-intl`, `react-i18next`, `vue-i18n`, `angular`, `svelte-i18n`, or `auto`) |
| `minStringLength` | `number` | `3` | Minimum string length to flag as hardcoded |
| `identicalThreshold` | `number` | `20` | Minimum length of identical EN/FR strings to flag |
| `ignoreWords` | `string[]` | technical terms | Words never flagged (proper nouns, technical terms) |
| `strictMode` | `boolean` | `false` | When true, warnings are treated as failures |

```json
{
  "version": 1,
  "bilingual": {
    "translationFiles": { "english": ["messages/en.json"], "french": ["messages/fr.json"] },
    "scanPaths": ["src/**/*.tsx"],
    "identicalThreshold": 20
  }
}
```

The legacy `.bilingual-review.json` file remains a deprecated fallback for one release.

<div lang="fr">

Les options sont imbriquées sous la clé `"bilingual"` dans `.gc-review/config.json` :

| Champ | Type | Défaut | Description |
|-------|------|--------|-------------|
| `translationFiles` | `object` | détection automatique | Motifs glob des fichiers de traduction, avec les tableaux `english` et `french` |
| `scanPaths` | `string[]` | selon le cadriciel | Motifs glob des fichiers source d'interface à analyser |
| `excludePaths` | `string[]` | tests, `node_modules` | Motifs glob des fichiers à exclure |
| `framework` | `string` | `"auto"` | Cadriciel d'i18n (`next-intl`, `react-i18next`, `vue-i18n`, `angular`, `svelte-i18n` ou `auto`) |
| `minStringLength` | `number` | `3` | Longueur minimale d'une chaîne signalée comme codée en dur |
| `identicalThreshold` | `number` | `20` | Longueur minimale des chaînes EN/FR identiques à signaler |
| `ignoreWords` | `string[]` | termes techniques | Mots jamais signalés (noms propres, termes techniques) |
| `strictMode` | `boolean` | `false` | Si vrai, les avertissements sont traités comme des échecs |

Le fichier hérité `.bilingual-review.json` demeure une solution de repli dépréciée pour une version.

</div>

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
