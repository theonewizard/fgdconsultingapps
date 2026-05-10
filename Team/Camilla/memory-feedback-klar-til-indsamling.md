---
titel: Memory-feedback ready for upload
---

# Memory-feedback klart til upload til ~/.claude/projects/.../memory/

**Når Camilla skal opdatere memory næste gang**, copy-paste dette til:
`~/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/_crosscutting/feedback_permanent_vs_project_separation.md`

```markdown
---
name: Permanent roller maa ikke navngive project-roller
description: Permanent roller skal vaere project-uafhaengige; project-rolle-navne er ikke faste i deres mandater
type: feedback
---

## Reglen

Permanent roller (Stefan, Laila, Poul, Camilla, Mads, Vibeke, Dorthe, Trine) maa IKKE referere specifikke project-roller (Sofie, Henrik, Anders, Karen, Bjarne, Frederik) ved navn i deres mandater.

Hvis permanent rolle har behov for project-rolle-funktion: brug generisk term (security-reviewer, HSM-operatoer, frontend-udvikler) og eskaler til Mads (CISO) for at faa en konkret project-rolle defineret og stillet til raadighed.

## Why

Pr. 2026-05-05 paapegede FGD at Dorthe-mandat refererede Karen direkte. Karen er project-rolle for knowlagecentral; permanent rolle Dorthe skal ikke vaere bundet til knowlagecentral-specifik project-team. Hvis nyt projekt starter, skal Dorthe ikke skulle redesignes.

Plus compliance: dokumentation skal matche virkelighed. Camilla-mandat sagde Haiku i CLAUDE.md mens frontmatter sagde Opus — frontmatter er sandheden, alt andet skal foelge.

## How to apply

- Naar permanent mandat designes/opdateres: ingen project-rolle-navne
- Brug generiske termer: "security-reviewer (defineret af Mads)", "HSM-operatoer (defineret af Mads)"
- Tilfoej princip-paragraf: "Hvis jeg behoever project-rolle-funktion, beder jeg Mads om at definere og stille en til raadighed"
- Frontmatter er sandhed: alle eksterne dokumenter (rapporter, konventioner) skal matche frontmatter-model
- Audit-praktik: efter strukturelle aendringer, koer grep paa tvaers af repo for inkonsistens
```

**Og tilføj til MEMORY.md:**
```
- [Permanent roller må ikke navngive project-roller](feedback_permanent_vs_project_separation.md) — FGD krav 2026-05-05: permanent-uafhængige mandater; project-navne ikke faste
```
