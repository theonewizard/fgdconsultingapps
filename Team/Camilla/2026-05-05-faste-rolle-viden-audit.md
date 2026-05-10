---
title: Audit af faste roller — viden-inddeling kontrol
date: 2026-05-05
type: audit
---

# Audit af faste roller — viden-inddeling kontrol

**Revisors note**: Mandaterne for Stefan, Laila, Poul, Camilla, Mads eksisterer endnu ikke som `.claude/agents/*.md`-filer. Denne audit etablerer **rammer og template** for hvad de skal indeholde når de designes, samt dokumenterer POC-resultat for memory-arkitektur.

---

## Executive summary

1. **POC-resultat**: Memory-subfolder-test PASSED. Auto-loader håndterer `_crosscutting/` og `projects/*/` uden problemer.
2. **Template etableret**: Hver fast rolles mandat skal opdeles i **crosscutting** (generisk) og **project-specifikt** (knyttet til aktivt projekt).
3. **Mandater som skal oprettes**: 5 faste + 3 project-roller (når tilkaldt)
4. **Camillas ansvar udvidet**: Viden-inddeling-kontrol når mandater opdateres.

---

## POC — Memory-subfolder auto-load

**Test**: Fil flyttet fra flat `memory/` til `memory/_crosscutting/`, MEMORY.md opdateret med relativ sti.

**Resultat**: PASSED. Auto-loader virker uden ændringer.

**Dokumentation**: `memory/_crosscutting/poc-memory-subfolder-2026-05-05.md`

---

## Audit-ramme: Crosscutting vs Project-specifikt

### Hvad er CROSSCUTTING (skal være i mandat)?

Generisk, uafhængigt af projekt:
- **Rolle-definition**: "Du er X. Dit job er..."
- **Generelle ansvarsområder**: Fx "Camilla ejer git-operationer", "Laila designer nye roller"
- **Hard rules**: "Aldrig git push --force uden eksplicit godkendelse", "Stefan udfører aldrig produktions-arbejde selv"
- **Hand-off-mønstre**: "Når du er klar, opdater Min Inbox med resumé", "Flagrer til Stefan ved uklarheder"
- **Anti-patterns**: "Du må IKKE selv skrive kode", "Du må IKKE tilføje filer til memory"
- **Hilsen/tone**: Standard-måde at hylse på, dansk tone, hvem man arbejder sammen med
- **Default-tools**: Tools som denne rolle **altid** har adgang til (Read, Bash, git status osv.)

### Hvad er PROJECT-SPECIFIKT (skal IKKE være i mandat)?

Knyttet til aktivt projekt:
- **Konkrete filstier**: `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md`
- **Stack-detaljer**: "Next.js 16.2 frontend", "Neon Postgres", "Vercel hosting"
- **Konkrete stakeholders**: "Du sparrer med Henrik (backend), Sofie (frontend), Dorthe (DPO)"
- **Projekt-specifikke teknologi**: "FIPS 140-3 HSM-integration for knowlagecentral", "Yubico YubiHSM 2"
- **Projekt-tidsplan**: "Fase 1.5 (HSM) mellem MVP og fase 2"
- **Project-roller**: "Karen (HSM-sikkerheds-operator)" — disse roller eksisterer kun når projektet er aktivt

**LØSNING**: Project-specifikt indhold læses fra `memory/projects/<projekt>/`-filer (fx `memory/projects/knowlagecentral/mads-første-session.md`), IKKE fra agentens mandat.

---

## Mandat-template for faste roller

Hver agent-mandat (`.claude/agents/<navn>-<rolle>.md`) skal følge denne struktur:

```markdown
---
name: <Navn> — <Rolle> (da)
description: <Én linje — hvem og hvad>
type: agent
role_type: crosscutting  # eller project (for Dorthe/Trine/Frederik når tilkaldt)
model: haiku|sonnet      # fra CLAUDE.md
---

# <Navn> — <Rolle>

## Mission

{{Kort, klart: "Du er X. Dit job er Y."}}

## Hvem I arbejder sammen med

- **Stefan** (leder): Alle opgaver går gennem Stefan
- **Laila** (HR): [hvis relevant]
- **Poul** (analytiker): [hvis relevant]
- **Camilla** (bibliotekar): [hvis relevant]
- **Mads** (CISO): [hvis relevant]

## Ansvarsområder (CROSSCUTTING)

### 1. Generelt ansvar
- Liste generelle områder denne rolle ejer

### 2. Hard rules
- Liste regler som ALDRIG må brydes
- Format: "Aldrig X uden Y", "IKKE Z", osv.

### 3. Hand-off & workflow
- Hvornår rapport denne rolle til Stefan
- Format på leverancer
- Hvornår og hvordan flagrer uklarheder

### 4. Anti-patterns
- Hvad denne rolle IKKE må gøre
- "Du må IKKE selv skrive kode" osv.

## Default-tools

- Tool1, Tool2 (baseret på CLAUDE.md-liste for denne rolle)

## Hilsen & tone

- Du hilser som [navn], kort præsentation
- Afsnittet "Stefan venter på godkendelse" hvis relevant
- Arbejdssprog: dansk (tekniske felter: engelsk)

## PROJECT-SPECIFIKT INDHOLD — IKKE HER

Se memory: `projects/<projekt>/<rolle>-projekt-context.md` for:
- Konkrete projektets stakeholders, stack, tidsplan
- Projekt-specifikke analyser denne rolle skal være klar på
- Konkrete deliverables for projektet

---
```

---

## Audit af 5 faste roller — anbefalinger

### 1. STEFAN — AI-organisator

**Mandatets indhold** (når oprettet):

- **Crosscutting**: Mission-statement, DELEGÉR-regel, hvad han MAY og MUST NOT, hand-off til Min Inbox, Stefans godkendelses-flow
- **Project-specifikt**: Skal IKKE være i mandatet. Læses fra `memory/projects/<projekt>/stefan-projekt-context.md`

**Anbefaling**: **Ren crosscutting** — Stefans rolle er konstant organisator uafhængigt af projekt.

**Hvad gøre nu**: Lad Stefan-mandatet oprettelse til når Laila er aktiveret. Template-ramme eksisterer.

---

### 2. LAILA — HR-leder

**Mandatets indhold** (når oprettet):

- **Crosscutting**: Rolle-design-process, ansættelses-workflow, kompetence-vurdering, Camilla → memory-integrering
- **Project-specifikt**: SKAL IKKE være indblandet i projekt-mandatet. Laila er crosscutting.

**Anbefaling**: **Ren crosscutting**

**Hvad gøre nu**: Lad oprettelse til når bootstrap er færdig.

---

### 3. POUL — Senior analytiker

**Mandatets indhold** (når oprettet):

- **Crosscutting**: Multi-track analyse-ansvar, behovs-analyse, peer-review-mønstre, hand-off til Camilla for memory-registrering
- **Project-specifikt**: SKAL være separat i memory. Fx `memory/projects/knowlagecentral/poul-knowlagecentral-kontekst.md` dokumenterer at Poul ejer knowlagecentral-arkitektur-analyser (v3 er leveret 2026-05-05)

**Anbefaling**: **Mandat ren crosscutting + separate project-kontekst-filer i memory**

**Hvad gøre nu**: Når Poul-mandat oprettelse planificeres, dokumentér knowlagecentral-scope separat.

---

### 4. CAMILLA — Senior bibliotekar

**Mandatets indhold** (når oprettet):

- **Crosscutting**: Git-arbejde, memory-vedligehold, database-arbejde, hard rules (IKKE commit uden godkendelse, aldrig --force, osv.), viden-inddeling-kontrol
- **Project-specifikt**: IKKE i mandatet. Camilla er constant infrastructure-rolle.

**Anbefaling**: **Ren crosscutting + udvid med viden-inddeling-kontrol-ansvar** (se nedenfor)

**Hvad gøre nu**: 
- Opret mandat når bootstrap er færdig
- Tilføj sektion for viden-inddeling-kontrol nu (dokumenteret i denne rapport)

---

### 5. MADS — CISO

**Mandatets indhold** (når oprettet):

- **Crosscutting**: Governance-principper, compliance-pipeline, OWASP-compliance, threat-modeling-ansvar, peer med Poul
- **Project-specifikt**: SKAL være separat i memory. Fx `memory/projects/knowlagecentral/mads-knowlagecentral-governance.md` dokumenterer FIPS 140-3, HSM-security, SOC 2-målbillede, DPO/DPA-model for knowlagecentral specifikt

**Anbefaling**: **Mandat ren crosscutting + separate project-kontekst-filer i memory**

**Hvad gøre nu**: 
- Opret mandat når bootstrap færdig
- Mads' knowlagecentral-context allerede dokumenteret i memory (`projects/knowlagecentral/mads-first-session-2026-05-05.md`)

---

## Viden-inddeling-kontrol-ansvar for Camilla

Når faste roller (og projekt-roller når tilkaldt) designes eller opdateres, **Camilla kontrollerer viden-inddeling**:

### Procedure

1. **Når nyt mandat oprettes eller ændres**:
   - Stefan sender Camilla draft-mandat eller ændring til review
   - Camilla læser og kategoriserer hver sektion:
     - Crosscutting: ✓ OK eller markér
     - Project-specifikt: flyt til memory/projects/<projekt>/

2. **Flagrer hvis**:
   - Mandat indeholder konkrete filstier eller stakeholders (project-specifikt) → skal til memory
   - Mandat nævner projekt-tidsplan eller stack-detaljer → skal til memory
   - Tvivl om kategorisering → spørg Stefan

3. **Dokumentation**:
   - Camilla noterer review i commit-besked eller kort notat til Stefan
   - Hvis større omrokeringer: dokumenter i memory under `_crosscutting/viden-inddeling-kontrol.md`

### Format i mandat for at markere project-specifikt

Hvis en sektion SKAL være i mandatet men er project-specifikt:

```markdown
> **[knowlagecentral-specifikt]**: Denne rolle sparrer primært med Henrik (backend), Sofie (frontend), Karen (HSM-security) i knowlagecentral-projektet. Se `memory/projects/knowlagecentral/mads-knowlagecentral-governance.md` for detaljer.
```

---

## Status på faste roller — hvad mangler

| Rolle | Mandat | Memory-context | Status |
|---|---|---|---|
| **Stefan** | [IKKE OPRETTET] | `memory/_crosscutting/stefan-rolle-godkendelses-flow.md` | **Bootstrap venter** |
| **Laila** | [IKKE OPRETTET] | [INGEN, crosscutting-only] | **Bootstrap venter** |
| **Poul** | [IKKE OPRETTET] | `memory/_crosscutting/*` + `memory/projects/knowlagecentral/poul...` (implicit) | **Bootstrap venter** |
| **Camilla** | [IKKE OPRETTET] | [eneste agent der har instruktioner i CLAUDE.md] | **Bootstrap venter** |
| **Mads** | [IKKE OPRETTET] | `memory/_crosscutting/mads-ciso-rolle-2026-05-05.md` + `memory/projects/knowlagecentral/mads-*` | **Bootstrap venter** |
| **Dorthe** (DPO) | [IKKE OPRETTET] | `memory/projects/knowlagecentral/dpo-model-2026-05-05.md` | **Project-rolle, tilkaldt når GO** |
| **Trine** (compliance-auditor) | [IKKE OPRETTET] | [TBD når tilkaldt] | **Project-rolle, tilkaldt når GO** |
| **Frederik** (pen-test) | [IKKE OPRETTET] | `memory/projects/knowlagecentral/pen-test-vendor-2026-05-05.md` | **Project-rolle, tilkaldt når GO** |

---

## Næste skridt

1. **Stefan** designer og opretter mandater for Stefan, Laila, Poul, Camilla, Mads (når FGD godkender bootstrap-sekvens)
2. **Camilla** kontrollerer hver mandat mod denne template før GO
3. **Camilla** opdaterer Memory med viden-inddeling-kontrol-ansvar
4. **Stefan** påpeger når project-roller (Dorthe/Trine/Frederik) skal tilkaldes, Laila ansætter, Camilla auditer

---

**Audit færdig 2026-05-05. Rapport til Stefan + FGD.**
