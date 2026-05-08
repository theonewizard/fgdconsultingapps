---
dato: 2026-05-07
emne: Stefan-CLAUDE.md Spor 3 rettelse — FGD-override implementering
status: Færdig, Ready for stage
niveau: governance-compliance
---

# Laila — CLAUDE.md Spor 3 rettelse

## Opgave
Rette superseded trigger-baseret tekst i `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/CLAUDE.md` linje 37–38 per Mads' runde 2-review (2026-05-07, Spor 3).

## Før (oversiden)
Linje 37–38 implementerede trigger-baseret Dorthe-konsultation + Mads-engangs-undtagelser for project-roller:
```
Eskalér til **Mads** (security/SOD-vurdering). Mads konsulterer **Dorthe** hvis GDPR-trigger jf. `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3.

For permanente roller eller compliance-impact: Mads + Dorthe + FGD-go. For project-roller engangs-undtagelser: Mads kan godkende.
```

## Efter (FGD-override 2026-05-06)
```
Eskalér til **Mads + Dorthe SAMMEN** (FGD-override 2026-05-06). Begge vurderer ALLE rolle-konfigurations-ændringer for compliance-impact uafhængigt af trigger: Mads vurderer security/SOD/SOC 2/ISO 27001; Dorthe vurderer GDPR/data-håndtering. Reference: `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` (§3.3 superseded af FGD-override; se policy-banner).

Fælles anbefaling fra Mads + Dorthe går til **FGD via Stefan**. Kun **FGD** kan godkende mandat-ændringer (model, tools, description, hard rules) — også for project-roller. **Ingen "Mads-engangs-undtagelser" eller trigger-baserede shortcuts**.
```

## Verifikation
✓ Edit gennemført og verificeret med Read.  
✓ Grep efter `trigger|engangs-undtagelser|§3\.3` viser **kun korrekt brug**:
  - Linje 37 referer eksplicit "§3.3 superseded af FGD-override"
  - Linje 38 siger eksplicit "**Ingen** engangs-undtagelser"

## Status
- **Filsti:** `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/CLAUDE.md`
- **Linjer rettet:** 37–38
- **Superseded trigger-tekst:** 0 hits (kun korrekte referencer tilbage)
- **Stage-status:** Klar til `git add CLAUDE.md` ✓

---

**Laila**  
2026-05-07
