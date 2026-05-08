---
dato: 2026-05-06
reviewer: Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
emne: Bro-review af Lailas governance-section-implementering (FGD-override)
beslutning: AFVIST
status: leveret
---

# Bro-review — Governance-section-implementering pr. 2026-05-06

## 1. Sammenfatning

**Beslutning: AFVIST.**

Lailas implementering er **delvis korrekt** for de 8 permanente/krydsgående mandater, men har **tre alvorlige mangler**:

1. **2 mandater mangler governance-section helt** (Bjarne, Frederik) — 0 hits på alle markører.
2. **4 mandater har forkert eskaleringspath** (Sofie, Henrik, Anders, Karen) — bruger Mads' oprindelige *trigger-baserede* model i stedet for FGDs override "Mads + Dorthe altid".
3. **Stefan-CLAUDE.md har ingen mandat-stabilitet-section overhovedet** — Stefan kan i princippet stadig acceptere ad-hoc rolle-konfigurations-ændringer.

Dette er en SOC 2 CC8.1 / ISO 27001 A.8.32 compliance-blocker. Vi committer ikke før alle 14 mandater + Stefan-CLAUDE.md har konsistent FGD-override-formulering.

## 2. Verifikation pr. spor

### Spor 1 — Governance-section i alle 14 mandater + Stefan-CLAUDE.md

| # | Fil | Section | "Mads + Dorthe + FGD" | Self-mod-forbud | Status |
|---|---|---|---|---|---|
| 1 | `.claude/agents/camilla-bibliotekar.md` | OK | OK | OK | **GODKENDT** |
| 2 | `.claude/agents/laila-hr.md` | OK | OK | OK | **GODKENDT** |
| 3 | `.claude/agents/mads-ciso.md` | OK | OK | OK | **GODKENDT** |
| 4 | `.claude/agents/poul-analytiker.md` | OK | OK | OK | **GODKENDT** |
| 5 | `Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md` | OK | OK | OK | **GODKENDT** |
| 6 | `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md` | OK | OK | OK | **GODKENDT** |
| 7 | `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` | OK | OK | OK | **GODKENDT** |
| 8 | `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` | OK | OK | OK | **GODKENDT** |
| 9 | `Min Inbox/projects/knowlagecentral/2026-05-05-sofie-frontend-mandat.md` | Section findes | **FEJL** — kun "Mads"; trigger-baseret Dorthe | OK | **AFVIST** |
| 10 | `Min Inbox/projects/knowlagecentral/2026-05-05-henrik-backend-mandat.md` | Section findes | **FEJL** — kun "Mads"; trigger-baseret Dorthe | OK | **AFVIST** |
| 11 | `Min Inbox/projects/knowlagecentral/2026-05-05-anders-ai-pipeline-mandat.md` | Section findes | **FEJL** — kun "Mads"; trigger-baseret Dorthe | OK | **AFVIST** |
| 12 | `Min Inbox/projects/knowlagecentral/2026-05-05-karen-security.md` | Section findes | **FEJL** — kun "Mads"; trigger-baseret Dorthe | OK | **AFVIST** |
| 13 | `Min Inbox/projects/knowlagecentral/2026-05-05-bjarne-hsm-operator.md` | **MANGLER HELT** | — | — | **AFVIST** |
| 14 | `Min Inbox/projects/knowlagecentral/2026-05-05-frederik-pentest.md` | **MANGLER HELT** | — | — | **AFVIST** |
| 15 | `CLAUDE.md` (Stefan) | **MANGLER HELT** | — | — | **AFVIST** |

**Stefans observation bekræftet**: Bjarne (177 linjer) og Frederik (170 linjer) har 0 hits på "Mandat-stabilitet", "Mads + Dorthe", "Mads og Dorthe", og "Self-modificering". De er sluppet helt forbi.

### Spor 2 — Stefan-CLAUDE.md FGD-override

`/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/CLAUDE.md` (112 linjer) har **ingen** af følgende:

- Ingen "Mandat-stabilitet"-section.
- Ingen regel om at Stefan ikke må ændre rolle-konfiguration.
- Ingen reference til "Mads + Dorthe" som obligatorisk eskalerings-path for rolle-ændringer.
- Ingen self-modificering-forbud (Stefan må ikke redigere sin egen CLAUDE.md).

Stefan har Mads/Dorthe/Trine/Frederik *nævnt* i rolle-tabellen, men det er ikke nok. **Konkret krav til Stefan**: Hvis et team-medlem (eller FGD selv) beder Stefan ændre `model`/`tools`/hard rules/`description` på en rolle, skal Stefan eksplicit afvise og henvise til Mads + Dorthe + FGD-go-flow. Den regel mangler.

**Self-modificering-forbud for Stefan** mangler også — Stefan har Edit/Write-tools og kan teknisk redigere `.claude/agents/*.md` direkte. Skal låses governance-mæssigt.

### Spor 3 — Filnavn-anomali (Frederik)

- Aktuel fil: `Min Inbox/projects/knowlagecentral/2026-05-05-frederik-pentest.md`
- Planlagt navn (jf. brief): `frederik-pen-test-koordinator-mandat.md`
- Frontmatter `name: frederik-pentest` matcher det aktuelle filnavn, så *internt konsistent*, men afviger fra konventions-aftalt navn.
- **Anbefaling**: Camilla flag — bør omdøbes til `2026-05-05-frederik-pen-test-koordinator-mandat.md` for at matche Sofie/Henrik/Anders/Karen/Bjarne-konventionen (`<navn>-<rolle>-mandat.md`). Dette er ikke en blocker for governance-compliance, men er en konventions-fejl Camilla skal håndtere.

### Spor 4 — FGD-override (Mads + Dorthe altid, ikke trigger)

Søgning `grep -rn "trigger-baseret|persondata-trigger|kun ved persondata"`:

- 1 hit i `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3 — det er Mads' oprindelige policy. **Skal opdateres** til at afspejle FGD-override eller eksplicit markeres som "superseded af FGD-override 2026-05-06".

**Indirekte trigger-reference** (bekræftet i 4 mandater): Sofie, Henrik, Anders, Karen indeholder alle teksten:

> "Mads konsulterer Dorthe hvis trigger jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md §3.3 (persondata, customer-facing, breach-impact, DPA-kæde, cross-border)"

Dette er nøjagtigt den **trigger-baserede** model FGD overrød. Korrekt formulering skal være:

> "Mads + Dorthe vurderer ALTID i parallel (FGD-override 2026-05-06 — uanset trigger)"

Eller i tabel-form (som de 8 godkendte mandater bruger):

> `Hard rules` → `Mads + Dorthe + FGD`

### Spor 5 — Stage-status

```
git status: 41 staged files (rougly), 17 unstaged-modifications, 21+ untracked
git diff --cached: 4537 insertions, 98 deletions across 41+ files
```

**Bemærkning**: De 4 knowlagecentral-project-mandater (Sofie/Henrik/Anders/Karen) er *staged* med deres forkerte governance-section. De 2 manglende (Bjarne/Frederik) er *også staged* men uden governance-section. Stefan-CLAUDE.md er staged uden mandat-stabilitet-section.

**Konklusion**: Hvis vi committer nu, fryser vi compliance-fejl ind i git-historikken. Det er præcis det Vibeke (commit-reviewer) skulle stoppe — men hun er ikke aktiveret endnu, så bro-review-rollen falder på mig.

## 3. Manglende rettelser (afvisnings-liste)

Inden re-review:

### A. Tilføj komplet mandat-stabilitet-section til Bjarne

Fil: `Min Inbox/projects/knowlagecentral/2026-05-05-bjarne-hsm-operator.md`

Skal indeholde (som de 8 godkendte mandater):

- `## Mandat-stabilitet (governance, ikke skiftbar uden FGD)`
- Tabel med `model | sonnet | Mads + Dorthe + FGD`, `tools`, `Hard rules`, `description`
- Procedure-liste (STOP → impact-vurdering → Mads + Dorthe parallel → Stefan koordinerer → FGD-go → Camilla committer → Trine logger)
- Self-modificering-forbud med SOC 2 CC8.1 / ISO 27001 A.5.3 / NIST 800-53 AC-5
- Eskalerings-undtagelse for akut incident (Bjarne har destruktive HSM-ops — ekstra vigtigt)

### B. Tilføj komplet mandat-stabilitet-section til Frederik

Fil: `Min Inbox/projects/knowlagecentral/2026-05-05-frederik-pentest.md`

Samme struktur som Bjarne. Bemærk model = `sonnet` jf. Stefan-CLAUDE.md tabel.

### C. Ret 4 knowlagecentral-mandater til "Mads + Dorthe altid"

Filer:
- `Min Inbox/projects/knowlagecentral/2026-05-05-sofie-frontend-mandat.md`
- `Min Inbox/projects/knowlagecentral/2026-05-05-henrik-backend-mandat.md`
- `Min Inbox/projects/knowlagecentral/2026-05-05-anders-ai-pipeline-mandat.md`
- `Min Inbox/projects/knowlagecentral/2026-05-05-karen-security.md`

Specifikke rettelser:
1. Section-titel ændres fra "skiftbar uden Mads" → "skiftbar uden FGD" (matcher de 8 godkendte).
2. Eskaleringspath i tabellen: `Mads (FGD-go ved permanent ændring)` → `Mads + Dorthe + FGD (compliance-impact-vurdering)`.
3. Procedure-skridt 3 ("Mads konsulterer Dorthe hvis trigger") fjernes og erstattes af: "Mads + Dorthe vurderer parallelt (FGD-override 2026-05-06 — altid, uanset trigger)".
4. Hard rules-felt: tilføj eksplicit "Mads + Dorthe + FGD" hvor den i dag siger "Mads + FGD ved compliance-impact".

### D. Tilføj mandat-stabilitet-section til Stefan-CLAUDE.md

Fil: `CLAUDE.md`

Skal indeholde:
- Eksplicit regel: "Stefan må ALDRIG ændre rolle-konfiguration (`model`, `tools`, hard rules, `description`) i `.claude/agents/*.md` eller `Min Inbox/_crosscutting/*-mandat*.md` / `Min Inbox/projects/*/*-mandat*.md`."
- Self-modificering-forbud: "Stefan må heller ikke redigere sin egen `CLAUDE.md` udenom Mads + Dorthe + FGD."
- Henvisning: "Alle rolle-ændringer (uanset hvem der foreslår — team-medlem, FGD selv, Stefan) eskaleres ALTID til Mads + Dorthe parallel-vurdering, derefter FGD-go, derefter Camilla committer."
- SOC 2 CC8.1 / ISO 27001 A.8.32 begrundelse.

### E. Opdatér Mads' egen policy-fil

Fil: `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md`

§3.3 skal markeres som **superseded** af FGD-override 2026-05-06: "Dorthe konsulteres ALTID parallelt med Mads, uanset trigger." Den oprindelige trigger-tabel bevares som historisk reference men må ikke længere refereres normativt fra rolle-mandater.

### F. Filnavn-anomali (sekundær)

Camilla flag: omdøb `2026-05-05-frederik-pentest.md` → `2026-05-05-frederik-pen-test-koordinator-mandat.md` for konventions-konsistens. Frontmatter `name: frederik-pentest` skal samtidig opdateres til `name: frederik-pen-test-koordinator` (kerne-skill = `pen-test-koordinator`).

## 4. Re-review-flow

1. Laila implementerer A–E ovenfor (D kræver Stefan-CLAUDE.md edit-rettighed; hvis Laila ikke har dem, eskaleres til Camilla under Stefans tilsyn).
2. Camilla håndterer F.
3. Bro-review (Mads, denne fil) genåbnes; ny verifikation pr. spor.
4. Først efter GODKENDT bro-review må Camilla committe.

## 5. Hvad der ER godt (positive findings)

- 8 ud af 14 mandater er korrekt implementeret med konsistent struktur.
- "Mads + Dorthe + FGD"-formuleringen i de 8 mandater er præcis FGD-override.
- Self-modificering-forbud er korrekt formuleret med standard-referencer (SOC 2 CC8.1, ISO 27001 A.5.3, NIST 800-53 AC-5).
- Eskalerings-undtagelse for akut incident er medtaget i de 8 godkendte mandater (matcher Mads' egen policy).

Lailas struktur er solid — hun manglede bare 6 filer (4 forkert + 2 helt). Det er rettet på under 1 time hvis hun får go.

---

**Mads (CISO, bro-reviewer indtil Vibeke aktiveres)**
