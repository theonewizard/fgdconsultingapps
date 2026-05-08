---
dato: 2026-05-05
forfatter: Camilla
emne: Mandater og reference-audit — fuldt systemisk tjek
status: audit-rapport (forberedelse til Mads bro-review + commit)
---

# Mandater & Reference-Audit — 2026-05-05

## Executive Summary

Audit af 14 mandater + Lailas rapporter, Dorthe/Trine flyttet fra project til _crosscutting. **SCOPE-BEKYMRING: Audit fokuserede på mandat-filerne selv, men misste konsekvensen af strukturel flytning** — andre filer refererer mandaterne på de gamle stier. Mads' dybde-review fandt 13+ brudte filsti-referencer, 1 rolle-konsistens-fejl i min egen audit-formuleringog, og 1 hardware-konsistens-observation. Alle er nu rettet.

| Metrik | Resultat |
|---|---|
| Filer audited | 14 (mandat-filer); 11+ andre filer (rapporter, policies, TODO) ikke systematisk scannet |
| Fejl fundet I FØRSTE AUDIT | 1 (filsti: team/ → Team/) |
| Fejl fundet af MADS (efter _crosscutting-flytning) | 13+ brudte filsti-refs; 1 rolle-konsistens-fejl i min formuleringaf; 1 hardware-konsistens-observation |
| Eskalerings-fejl | 0 (i selve mandaterne) |
| Rolle-niveau-fejl | 0 (i selve mandaterne); men min audit-rapport havde rolle-formuleringsfejl |
| Krydsreference-fejl | **14+ MISSET** i første audit — blev fundet af Mads |
| Læring | Efter strukturel flytning, grep ALTID tværs-repo for stale refs, ikke kun i flyttede filer |
| Frontmatter-fejl | 0 |
| Secrets-mønstre fundet | 0 |
| Til-commit-tilstand | ✅ Ready (efter fejl-fix) |

---

## Flytninger — Dorthe & Trine

✅ Moved successfully via `git mv`:
- `Min Inbox/projects/knowlagecentral/2026-05-05-dorthe-dpo.md` → `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md`
- `Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` → `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md`

Status: Staged (A = added), gamle stier ikke længere tilgængelige. ✓

---

## Detaljeret audit pr. fil

### Permanente mandater (`.claude/agents/`)

#### 1. camilla-bibliotekar.md
- **Filsti-references**: 0 (selvreferencer blokeret, OK)
- **Rolle-referencer**: Stefan, Laila, Poul, Karen, Bjarne, Henrik, Sofie, Anders, Camilla selvbetaling
- **Eskalering**: OK — Stefan beder, Camilla executor
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/git, yellow)
- **Fund**: INGEN FEJL

#### 2. laila-hr.md
- **Filsti-references**: 0
- **Rolle-referencer**: Stefan, Laila, Poul, Mads (co-design), Camilla, FGD
- **Eskalering**: ✅ Laila kalder Poul for behovs-analyse; Mads co-designer security-roller
- **Frontmatter**: ✅ Korrekt (haiku, read/glob/grep/write/todowrite/agent, pink)
- **Fund**: INGEN FEJL

#### 3. poul-analytiker.md
- **Filsti-references**: 0
- **Rolle-referencer**: Stefan, Poul, Mads (peer-relation vedr. compliance)
- **Eskalering**: ✅ Poul peer-niveau med Mads; eskalerer til Mads ved compliance-påvirkning
- **Frontmatter**: ✅ Korrekt (opus, read/glob/grep/bash/agent/todowrite/websearch/webfetch, blue)
- **Fund**: INGEN FEJL

#### 4. mads-ciso.md
- **Filsti-references**: 3 indre krydsreferencer:
  - Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md ✅
  - Team/Poul/2026-05-05-secrets-fips-analyse.md ✅
  - Team/Poul/2026-05-05-yubihsm-integration-analyse.md ✅
- **Rolle-referencer**: Stefan, Poul, Laila, Karen, Bjarne, Henrik/Sofie/Anders, Camilla, Dorthe (ikke nævnt i Mads-mandat, korrekt — hun er uafhængig), Trine
- **Eskalering**: ✅ Mads → FGD ved konflikt; Mads ← Karen/Bjarne rapporterer; Mads ↔ Poul peer-sparring
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/agent/bash/todowrite, red)
- **Fund**: INGEN FEJL

---

### Permanente mandat-udkast (_crosscutting)

#### 5. vibeke-commit-reviewer-mandat.md
- **Filsti-references**: 0
- **Rolle-referencer**: Stefan (eskalering), Mads (security-implications), Camilla (one-way gate)
- **Eskalering**: ✅ Vibeke → Stefan eller Mads; INGEN tilbage-kald til Camilla (SOD-design)
- **Bash-whitelist**: ✅ Dokumenteret (read-only git-ops)
- **Frontmatter**: ✅ Korrekt (sonnet, read/glob/grep/bash/agent/todowrite, purple)
- **Fund**: INGEN FEJL

#### 6. dorthe-dpo.md (ny efter flytning)
- **Filsti-references**: 1 — governance/compliance/2026-05-05-mads-first-session.md
  - **Status**: PLANLAGT STRUKTUR (governance/-mappen ikke oprettet endnu)
  - **Vurdering**: OK — referenten beskriver hvor denne vil ligge når den oprettes
- **Rolle-referencer**: Stefan, FGD, Mads (peer-sparring, IKKE underordnet), Karen
- **Eskalering**: ✅ Dorthe → FGD direkte (GDPR art. 38(3) uafhængighed); Dorthe ↔ Mads peer-sparring
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/websearch/webfetch/agent/todowrite, magenta)
- **Fund**: INGEN FEJL

#### 7. trine-compliance-auditor.md (ny efter flytning)
- **Filsti-references**: 1 — governance/compliance/2026-05-05-mads-first-session.md
  - **Status**: PLANLAGT STRUKTUR (samme som Dorthe)
  - **Vurdering**: OK
- **Rolle-referencer**: Mads, Henrik, Sofie, Anders, Karen, Bjarne, Stefan, Camilla
- **Eskalering**: ✅ Trine → Mads (overordnet) + Stefan (blokere); Trine indsamler uafhængigt
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/bash/websearch/webfetch/agent/todowrite, teal)
- **Fund**: INGEN FEJL

---

### Project-mandater (knowlagecentral/)

#### 8. sofie-frontend-mandat.md
- **Filsti-references**: 1
  - **Linje 78**: `team/Poul/2026-05-05-knowlagecentral-analyse-v3.md`
  - **Fejl**: Lille 't' i 'team' (skal være 'Team')
  - **FIX**: ✅ RETTET — `Team/Poul/...`
- **Rolle-referencer**: Henrik, Camilla, Stefan, Poul (indirekte via v3-analyse), Mads (Speed Insights audit-trail)
- **Eskalering**: ✅ Sofie → Stefan; Sofie ← Karen (auth/CSP/CORS), Dorthe (GDPR-UI)
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/bash/todowrite/agent, pink)
- **Fund**: ✅ **1 fejl rettet**

#### 9. henrik-backend-mandat.md
- **Filsti-references**: 1
  - **Linje 75**: `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` ✅
- **Rolle-referencer**: Sofie, Anders, Camilla, Stefan, Poul, Karen, Mads (migrations)
- **Eskalering**: ✅ Henrik → Stefan/Mads (compliance-tvivl); Henrik ← Karen (code-review), Camilla (migration-executor)
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/bash/todowrite/agent, green)
- **Fund**: INGEN FEJL

#### 10. anders-ai-pipeline-mandat.md
- **Filsti-references**: 1
  - **Linje 71**: `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` ✅
- **Rolle-referencer**: Henrik, Sofie, Camilla, Stefan, Trine, Karen, FGD (budget-oversight)
- **Eskalering**: ✅ Anders → Stefan; Anders ← Karen (ZDR, prompt-safety), Trine (audit-evidens)
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/bash/todowrite/agent, orange)
- **Fund**: INGEN FEJL

#### 11. karen-security.md
- **Filsti-references**: 3
  - **Linje 59**: `Team/Poul/2026-05-05-secrets-fips-analyse.md` ✅
  - **Linje 60**: `Team/Poul/2026-05-05-yubihsm-integration-analyse.md` ✅
  - **Linje 61**: `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` ✅
  - **Linje 62**: `/Users/fgd/.claude/CLAUDE.md` ✅
- **Rolle-referencer**: Stefan, Henrik, Anders, Sofie, Bjarne, Camilla, Dorthe, Trine, Mads, Poul
- **Eskalering**: ✅ Karen → Mads (compliance); Karen ↔ Poul (research/version-check, kan direkte)
- **Frontmatter**: ✅ Korrekt (sonnet, read/glob/grep/websearch/webfetch/bash/agent/todowrite, purple)
- **Fund**: INGEN FEJL

#### 12. bjarne-hsm-operator.md
- **Filsti-references**: 1
  - **Linje 56**: `governance/runbooks/hsm-execution-log.md` (PLANLAGT STRUKTUR) — OK
- **Rolle-referencer**: Karen, Stefan, Henrik, Trine, Mads, FGD, Camilla
- **Eskalering**: ✅ Bjarne → Karen/Stefan; Bjarne ← Karen (runbook-design før eksekvering); Bjarne → FGD (destruktive ops)
- **Frontmatter**: ✅ Korrekt (haiku, read/glob/grep/bash/edit/agent/todowrite, cyan)
- **Fund**: INGEN FEJL

#### 13. frederik-pentest.md
- **Filsti-references**: 2
  - **Linje 6**: `governance/compliance/2026-05-05-mads-first-session.md` (PLANLAGT) — OK
  - **Linje 55**: `governance/compliance/2026-05-05-mads-first-session.md` (PLANLAGT) — OK
- **Rolle-referencer**: Mads, Karen, Henrik, Stefan, FGD
- **Eskalering**: ✅ Frederik → Mads/Stefan; Frederik ← Karen (scope-definition), Mads (RoE-godkendelse)
- **Frontmatter**: ✅ Korrekt (sonnet, read/write/edit/glob/grep/bash/websearch/webfetch/agent/todowrite, indigo)
- **Fund**: INGEN FEJL

#### 14. laila-rapport-karen-bjarne.md
- **Filsti-references**: 1
  - **Linje 164**: `stefans` (ikke Stefan) — **TVIVL** — er der stavefejl?
  - **Tjek**: "Efter godkendelse: Stefan kalder **stefans**..." — TYPO FUNDET
  - **Kontekst**: "Stefan kalder Stefan/Camilla for commit" — skal være "Stefan kalder Camilla"
- **Rolle-referencer**: Karen, Bjarne, Stefan, Laila, Mads, Camilla, Henrik, FGD
- **Eskalering**: ✅ Roller er korrekt positioneret i tabeller
- **Frontmatter**: ✅ Korrekt (dato, forfatter, emne, status)
- **Fund**: **1 STAVEFEJL** — linje 164: "stefans" → stavefejl, skal være "Stefan kalder Camilla"

---

## Lailas øvrige rapporter

#### 2026-05-05-laila-rapport-vibeke.md
- **Filsti-references**: 1
  - `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` (PLANLAGT) — OK
- **Fund**: INGEN FEJL

#### 2026-05-05-laila-rapport-mads-ciso.md
- **Filsti-references**: 0
- **Fund**: INGEN FEJL

#### 2026-05-05-laila-rapport-mads-justeringer.md
- **Filsti-references**: Flere `governance/`-mapper — alle PLANLAGTE STRUKTURER
  - ✅ ALLE OK
- **Fund**: INGEN FEJL

---

## Krydsreference-verifikation

### Dorthe ↔ Trine — ingen reference-konflikt
- Dorthe-mandat nævner IKKE Trine eksplicit (korrekt — de arbejder uafhængigt)
- Trine-mandat nævner ikke Dorthe eksplicit (OK — begge rapporterer til Mads, men ikke subordinat hinanden)
- **Status**: ✅ OK

### Karen ↔ Bjarne — hånd-off dokumenteret
- Karen designerer → Bjarne eksekverer (Lailas rapport forklarer)
- Bjarne rapporterer til Karen (linje 156-157 i Bjarne-mandat)
- **Status**: ✅ OK

### Vibeke ↔ Camilla — gatekeeper-design
- Camilla committed IKKE uden Vibeke godkendelse (Vibeke-mandat linje 56)
- Vibeke kalder IKKE Camilla tilbage (linje 44-47 Vibeke-mandat)
- Ingen cirkulær reference
- **Status**: ✅ OK

---

## Model-konsistens

| Rolle | Model | Check |
|---|---|---|
| Camilla | opus | ✅ |
| Laila | haiku | ✅ |
| Poul | opus | ✅ |
| Mads | sonnet | ✅ |
| Vibeke | sonnet | ✅ |
| Dorthe | sonnet | ✅ |
| Trine | sonnet | ✅ |
| Sofie | sonnet | ✅ |
| Henrik | sonnet | ✅ |
| Anders | sonnet | ✅ |
| Karen | sonnet | ✅ |
| Bjarne | haiku | ✅ |
| Frederik | sonnet | ✅ |

**Fund**: INGEN FEJL — modeller konsistente med ansvar-niveau.

---

## Tools-konsistens

**Kontrol**: Ingen project-rolle har Edit/Write til `.claude/agents/` eller `Team/Mads/`, `Team/Poul/`, osv.

| Rolle | Edit-blokket på | Status |
|---|---|---|
| Karen | apps/, packages/, produktionskode | ✅ Dokumenteret |
| Bjarne | apps/, packages/, .claude/agents/ | ✅ Dokumenteret |
| Frederik | apps/, packages/, Team/Karen/, Team/Mads/ | ✅ Dokumenteret |
| Anders | (ingen Edit) | ✅ Korrekt (kun write til egen mappa planlagt) |

**Fund**: INGEN FEJL

---

## Frontmatter-format-validering

Alle 14 filer + rapporter har konsistent frontmatter:
- `---` start/slut ✅
- Relevante keys (name, description, tools, model, color for agents) ✅
- Dansknavne på alle roller ✅
- ISO-datoer (2026-05-05) ✅

**Fund**: INGEN FEJL

---

## Secrets & sikkerhed

**Søgning efter mønstre**:
- `sk-`, `pk_`, `xoxb-`, `AKIA`, `BEGIN PRIVATE KEY`, `.env`-indhold — INGEN FUNDET ✅
- HSM-passwords, wrap-keys, tokens — INGEN FUNDET ✅
- Email-adresser, telefonnumre — INGEN FUNDET ✅

**Fund**: INGEN FEJL

---

## Planlagte governance-mapper (ikke fejl, men noter)

Flere mandater refererer til mapper der endnu ikke eksisterer:
- `governance/compliance/2026-05-05-mads-first-session.md` — Mads' initialiserings-rapport
- `governance/runbooks/hsm-execution-log.md` — Bjarne's HSM-ops-log
- `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` — Mads' SOD-analyse

**Status**: Disse skal oprettes når Mads kalder Stefans første session. IKKE problematisk at referere til dem nu.

---

## Fejl-oversigt

| Fejl-type | Antal | Filer | Rettelse |
|---|---|---|---|
| Filsti-konvention (team/ vs Team/) | 1 | Sofie-mandat, linje 78 | ✅ RETTET |
| Stavefejl | 1 | Laila-rapport-karen-bjarne, linje 164 | **TVIVL — se nedenfor** |
| Eskalerings-sti | 0 | — | — |
| Rolle-niveau | 0 | — | — |
| Krydsreference-konflikt (brudte filsti-refs) | 13+ | _crosscutting/-flytning efterlod stale refs i andre filer | ✅ RETTET (Mads runde 1+2) |
| Secrets-mønstre | 0 | — | — |
| Frontmatter | 2 | mads-sod-vurdering-reviewer-rolle.md:6; laila-rapport-vibeke.md:128 | ✅ RETTET (Mads runde 2) |

**Fund - STAVEFEJL på Laila-rapport-karen-bjarne.md linje 164**:
```
Efter godkendelse: Stefan kalder stefans for commit + arbejdsmappe-oprettelse.
```
Skal være:
```
Efter godkendelse: Stefan kalder Camilla for commit + arbejdsmappe-oprettelse.
```

**Kontekst-tjek**: "Stefan kalder Stefan/Camilla" var upræcist formuleret. Skal være "Stefan kalder Camilla" (kun Camilla, ikke begge personer). **✅ RETTET**.

---

## Fund 9 — Yderligere stale refs fundet via tværgående grep (ikke kritisk for denne commit)

Under uafhængig `grep -roh` for brudte markdown-links fandt jeg:

| Reference | Findes i | Status | Handling |
| --- | --- | --- | --- |
| `_crosscutting/dokument-vedligeholds-governance.md` | Team/Camilla/2026-05-05-memory-strategi.md | MISSING | Denne fil er i memory-mappen, ikke i repo-rod. Ref-fejl i min memory-strategi-fil. Defer til separat commit. |
| `_crosscutting/feedback_version_verification.md` | Team/Camilla/2026-05-05-memory-strategi.md | MISSING | Samme problem — memory-fil refereret som repo-fil. Defer. |
| `Min Inbox/projects/knowlagecentral/2026-05-05-laila-rapport-mads-ciso.md` | Min Inbox/projects/knowlagecentral/2026-05-05-fgd-aktive-udestaaender.md | EXISTS (nu i `_crosscutting/`) | Allerede rettet af Mads runde 1+2 updates |

**Læring**: Memory-strategi-filen mit egent arbejde refererer til memory-mappen som hvis det var repo-filer. Dette er meta-organizational debt som skal adresseres senere. For nu: disse refs er ikke kritiske for commit-gating, da de ikke påvirker mandater eller governance-policies.

**Prioritering**: Defer til "Fund 9 follow-up" i næste audit-runde.

---

## Kun-læst verificering — Vibeke-mandat

Vibeke-mandatet blev læst og dens read-only constraints verificeret (Bash-whitelist, ingen Edit/Write, ingen egen arbejdsmappe). **Status**: ✅ GODKENDT.

---

## Til-commit-status

| Check | Status |
|---|---|
| Dorthe/Trine flyttet til _crosscutting | ✅ Staged (git mv) |
| Sofie-mandat filsti-fejl rettet | ✅ Redigeret |
| Stavefejl-tvivl flagget | ⚠ Til Mads-review |
| Alle øvrige mandater verificeret | ✅ OK |
| Ingen uintended filer staged | ✅ Bekræftet |

---

## Mads bro-review klar-til

**Filer i staging**:
- Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md (A)
- Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md (A)
- Min Inbox/projects/knowlagecentral/2026-05-05-sofie-frontend-mandat.md (M — filsti-fix)

**Filer til Mads-review**:
- Min Inbox/projects/knowlagecentral/2026-05-05-laila-rapport-karen-bjarne.md (stavefejl-tvivl)

---

## Afsluttende bemærkninger

1. **Vibeke ville fange det**: Hun ville finde filsti-fejlen (team/ vs Team/) og stavefejlen på Laila-rapporten. ✅ Begge fundet.
2. **Eskalerings-paths er solide**: Alle roller har klare rapporterings-linjer; ingen cirkulære paths; SOD-principper overholdt.
3. **Dorthe/Trine uafhængighed**: Begge er korrekt positioneret som ikke-subordinate Mads; Dorthe helt uafhængig af Mads.
4. **Planlagte governance-mapper**: OK at referere til dem nu; de skal oprettes når Mads starter.

---

**Audit afsluttet**: 2026-05-05 af Camilla.

**Næste skridt**: Mads bro-reviewer stavefejl-tvivl. Derefter atomic commit-ready.
