# gc-review-bilingual

A Claude Code skill for reviewing code against Government of Canada Official Languages Act (OLA) compliance requirements. Ensures applications provide equivalent experiences in both English and French.

## Installation

Install the full skill collection (this skill included):

```bash
npx skills add dougkeefe/gc-code-skills
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
- Placeholder values (TODO, `[TRANSLATE]`, `[French translation needed]`)
- Suspiciously identical long strings

### C. Localized Navigation & Routing
Verifies locale-aware routing and dynamic `<html lang>` attributes.

### D. Locale-Aware Formatting
Checks that dates, numbers, and currencies rendered in the UI use `Intl` formatters with locale parameters.

### E. Accessibility Parity
Ensures `aria-label`, `alt`, and `title` attributes use translation keys.

### F. Active Offer, Language Toggle & Equal Prominence
Verifies a language switch mechanism exists (active offer, OLA s. 28), toggle wording follows the Canada.ca pattern ("Français"/"English" full words; "FR"/"EN" only on small screens with a `title` attribute), and both languages get equal prominence and correct order (OLA s. 29; DOLCS s. 6.1.1).

### G. Website Requirements (DOLCS s. 6.6)
Checks metadata language, UTF-8/diacritics support, hyperlink language notices, and bilingual domain names.

## Supported Frameworks

- next-intl (Next.js)
- react-i18next
- vue-i18n
- @angular/localize
- svelte-i18n

## Configuration

Create `.gc-review/config.json` in your project root, with bilingual options under the `"bilingual"` key:

```json
{
  "version": 1,
  "bilingual": {
    "translationFiles": {
      "english": ["messages/en.json"],
      "french": ["messages/fr.json"]
    },
    "scanPaths": ["src/**/*.tsx", "app/**/*.tsx"],
    "excludePaths": ["**/*.test.*"],
    "framework": "auto",
    "minStringLength": 3,
    "identicalThreshold": 20,
    "ignoreWords": ["GitHub", "OAuth", "API"],
    "strictMode": false
  }
}
```

The legacy `.bilingual-review.json` file is still read as a deprecated fallback for one release; a warning is emitted asking you to migrate.

## Output Format

The skill produces a structured report:

| Status | File | Issue Found | Recommended Action |
|--------|------|-------------|-------------------|
| ❌ Fail | `Header.tsx:24` | Hardcoded string "Sign In" | Use `t('auth.signIn')` |
| ❌ Fail | `fr.json` | Missing key `dashboard.title` | Add French translation |
| ⚠️ Warning | `fr.json` | `nav.help` is "TODO" | Provide translation |
| ✅ Pass | `layout.tsx` | Dynamic `lang` attribute | None |

## Policy Reference

**Skill ID:** GOC-BILINGUAL-001
**Policy Driver:** *Official Languages Act* / *Loi sur les langues officielles* (R.S.C. 1985, c. 31 (4th Supp.)), Part IV; *Official Languages (Communications with and Services to the Public) Regulations*, SOR/92-48; *Policy on Official Languages* (2012-11-19); *Directive on Official Languages for Communications and Services* (2012-11-19); *Directive on the Management of Communications and Federal Identity* (2025-03-27)
**Last Verified:** 2026-07-10

## Résumé en français

<div lang="fr">

Une compétence Claude Code qui vérifie la conformité du code aux exigences de la *Loi sur les langues officielles* (L.R.C. 1985, ch. 31 (4e suppl.)), partie IV, et aux instruments du Conseil du Trésor connexes. Elle veille à ce que les applications offrent une expérience équivalente en français et en anglais.

**Ce qu'elle vérifie :**

- **Extraction des chaînes** — signale le texte codé en dur dans les composants d'interface.
- **Parité des dictionnaires** — compare les fichiers de traduction français et anglais (clés manquantes, valeurs de substitution, chaînes identiques suspectes).
- **Navigation et routage localisés** — vérifie le routage selon la langue et l'attribut `<html lang>` dynamique.
- **Formatage selon la locale** — dates, nombres et devises affichés avec les formateurs `Intl`.
- **Parité de l'accessibilité** — attributs `aria-label`, `alt` et `title` traduits.
- **Offre active, sélecteur de langue et proéminence égale** — mécanisme de changement de langue présent (art. 28 de la LLO), libellé « Français »/« English » conforme au modèle de Canada.ca, proéminence et ordre des langues (art. 29 ; Directive sur les langues officielles pour les communications et services, s. 6.1.1).
- **Exigences Web (s. 6.6 de la Directive)** — langue des métadonnées, prise en charge des signes diacritiques (UTF-8), avis de langue sur les hyperliens, noms de domaine bilingues.

**Configuration :** créez `.gc-review/config.json` avec les options sous la clé `"bilingual"` (le fichier `.bilingual-review.json` est déprécié).

**Avis :** cette revue automatisée ne constitue pas une évaluation formelle de conformité à la *Loi sur les langues officielles*.

</div>

## License

MIT
