---
dato: 2026-05-05
forfatter: Laila
emne: Implementering af Mads' role-justeringer — 3 nye roller + 9 mandater opdateret
status: udkast til FGD-godkendelse
---

# Implementering af Mads' Role-Justeringer

## Executive Summary

Jeg har implementeret alle Mads' (CISO) justeringer fra hans first-session-leverance baseret på hans governance-krav for de 5 compliance-teams:

1. **3 nye rolle-udkast designet og oprettet** i `Min Inbox/`:
   - Dorthe (DPO, permanent, sonnet)
   - Trine (Compliance Auditor, permanent, sonnet)
   - Frederik (Pen-test Koordinator, project, sonnet)

2. **9 eksisterende mandater opdateret** med Mads' konkrete justeringer:
   - 5 project-mandater (Sofie, Henrik, Anders, Karen, Bjarne) i `Min Inbox/`
   - 4 permanente mandater (Laila, Poul, Camilla, Mads) i `.claude/agents/`

3. **1 projekt-niveau justering flagget** til Camilla (Stefan-CLAUDE.md).

**Status**: Alle 3 nye role-udkast og 9 mandater-opdateringer er klare til FGD-godkendelse. Ingen konflikter eller afvigelser fra Mads' anbefalinger.

---

## 1. Nye Rolle-Udkast (3 stk.)

### Dorthe — Data Protection Officer (DPO)

**Fil**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md`

**Design**:
- **Model**: sonnet (compliance-ræsonnement over lovtekster kræver dybde)
- **Tools**: Read, Write, Edit, Glob, Grep, WebSearch, WebFetch, Agent, TodoWrite
- **Permanent rolle** (GDPR-overhold er ikke projekt-bundet)
- **Uafhængig af Mads** (GDPR art. 38(3) kræver DPO-uafhængighed)
- **Rapporterer direkte til FGD** (ikke gennem Mads-hierarkiet)
- **Ansvar**: ROPA, DPIA, breach-respons (72-timer), data-subject-rettigheder, DPA-vedligehold

**Baseret på Mads' §5.1** — alle elementerne fra hans rolle-profil inkluderet.

### Trine — Compliance Auditor

**Fil**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md`

**Design**:
- **Model**: sonnet (audit-ræsonnement + control-design-vurdering)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Agent, TodoWrite
- **Permanent rolle** (ISO 27001 + SOC 2 er løbende cykler, ikke projekt)
- **Revisor-uafhængighed**: Trine auditer uafhængigt; Mads definerer rammer
- **Rapporterer til Mads** (overordnet) + Stefan (når blocker)
- **Ansvar**: SoA, control-mapping, evidens-indsamling, månedsrapporter, audit-forberedelse

**Baseret på Mads' §5.2** — alle elementerne inkluderet, plus eksplicit SOD-klargjøring (Trine er ikke under udvikler-teamet).

### Frederik — Pen-test Koordinator

**Fil**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/projects/vidensapp/2026-05-05-frederik-pentest.md`

**Design**:
- **Model**: sonnet (juridisk + teknisk ræsonnement for RoE-skrivning)
- **Tools**: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Agent, TodoWrite
- **Project-rolle** (aktiv før hver pen-test-cyklus, dvale imellem, ~0,2-0,3 FTE)
- **Rapporterer til Mads** (RoE-godkendelse) + Stefan (tidslinje)
- **Ansvar**: Vendor-shortlist, RoE-drafting, scope-definition, pre-pen-test-scanning, remediation-tracking

**Baseret på Mads' §5.3 + §7** — vendor-shortliste fra hans §7 Pen-test-vendor-analyse integreret.

---

## 2. Mandater-Opdateringer (9 stk.)

### Project-Mandater (5 stk. i `Min Inbox/`)

#### Sofie (Frontend) — 3 nye hard rules

**Justeringer fra Mads §6.6**:
1. ✅ Tilføjet: "Hver PR til `apps/web/` kræver mindst én reviewer der ikke er author. For PRs der ændrer auth-flow/CSP/CORS/cookie-håndtering: Karen er obligatorisk reviewer."
2. ✅ Tilføjet: "e2e-tests (`e2e-testing` skill) skal eksistere for nye user-flows før merge."
3. ✅ Tilføjet: "Speed Insights må ikke disables uden Mads-godkendelse (audit-trail-krav, SOC 2 CC7.2)."
4. ✅ Tilføjet: "GDPR-impacted UI (cookie-banner, consent-flow, data-export-UI) reviewes af Dorthe før merge."

#### Henrik (Backend) — 4 nye hard rules

**Justeringer fra Mads §6.7**:
1. ✅ Tilføjet: "Migrations + RLS-policies merges aldrig uden 2 reviewers: Karen (security) + Mads-godkendelse. Camilla executor kun efter Mads-godkendelse (compliance-blocker)."
2. ✅ Tilføjet: "OpenAPI-ændringer af auth-scheme/error-format/PII-felter reviewes af Dorthe + Karen."
3. ✅ Tilføjet: "Audit-log-struktur designes i samarbejde med Trine; immutability, hash-chain, retention er governance-krav."
4. ✅ Tilføjet: "Idempotency-cache, rate-limiting, RFC 9457 errors obligatoriske ved hver write-endpoint; `security-review` skill kører i CI som gate."

#### Anders (AI-pipeline) — 4 nye hard rules

**Justeringer fra Mads §6.8**:
1. ✅ Tilføjet: "Prompt-bibliotek-ændringer reviewes af Karen + Trine før merge."
2. ✅ Tilføjet: "Audit-event-emission: Anders ejer emission, Henrik ejer tabel-skema, Trine ejer review. Adskillelse obligatorisk (revisor-uafhængighed)."
3. ✅ Tilføjet: "ZDR-verifikation før første kunde-data (ikke sidste-minut-tjek)."
4. ✅ Tilføjet: "Spend-cap-ændringer kræver Mads-godkendelse (SOC 2 A.1.1 capacity-management)."

#### Karen (Security) — Scope-reduktion + co-reviewer med Trine

**Justeringer fra Mads §6.9**:
1. ✅ Fjernet: "DPA-review" fra direkte ansvar (Dorthe overtager)
2. ✅ Fjernet: "DPA-tracker vedligehold" (Dorthe overtager)
3. ✅ Tilføjet: "Co-reviewer med Trine på SOC 2 CC6-CC8 (Logical/Physical Access Controls)"
4. ✅ Tilføjet: "Rapporterer til Mads (gøres eksplicit)"
5. ✅ Tilføjet: "Samarbejde med Dorthe på features med GDPR-påvirkning" (både-og, ikke enten-eller)

#### Bjarne (HSM-operatør) — Logging + Trine-involvement

**Justeringer fra Mads §6.10**:
1. ✅ Tilføjet: "Bjarne logger hver HSM-ops til `governance/runbooks/hsm-execution-log.md` (immutable append-only) — Trine reviewer månedligt for SOC 2-evidens."
2. ✅ Tilføjet: "Quarterly DR-test rapporteres til Trine + Mads (ikke blot Karen)."

### Permanente Mandater (4 stk. i `.claude/agents/`)

#### Laila (HR) — Co-design for compliance-roller

**Justering fra Mads §6.2**:
- ✅ Udvidet co-design-aftale: nu gælder også for **compliance-roller** (Dorthe, Trine, Frederik), ikke kun security-roller.

#### Poul (Analytiker) — Design-docs i governance/

**Justering fra Mads §6.3**:
- ✅ Tilføjet: "Når Poul leverer arkitektur-analyser, lægger han kopi i `governance/design-docs/` så Trine/Dorthe kan auditere baggrund for kontrol-design (compliance-transparens)."

#### Camilla (Bibliotekar) — SOD-håndhævelse + migration-gates + memory-governance

**Justeringer fra Mads §6.4** (kritisk):
1. ✅ Tilføjet hard rule: "Camilla må aldrig commit uden **referenceret godkendelse** — enten `Min Inbox/<dato>-<emne>.md` med FGD-signoff eller Stefan-besked. Hver commit-besked skal pege til godkendelses-kilde."
2. ✅ Tilføjet hard rule: "Migrations til prod kræver **Henrik (designer) + Camilla (executor) + Mads-godkendelse hvis crypto/RLS/audit-log påvirkes**."
3. ✅ Tilføjet hard rule: "Memory-opdateringer der vedrører compliance/security skal cc:'es til Mads."

#### Mads (CISO) — Audit-uafhængighed eksplicitgjort

**Justering fra Mads §6.5**:
- ✅ Tilføjet hard rule: "Mads ejer ikke audit-evidens-indsamling — det er Trines opgave. Mads godkender SoA; Trine indsamler evidens uafhængigt."

---

## 3. Projekt-Niveau Justering — Stefan-CLAUDE.md

**Flag til Camilla**:

Mads' justering fra §6.1 skal implementeres i projektets `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/CLAUDE.md`:

**Kommende tilføjelse til Stefan's Mission-sektion**:
```
## Feature-godkendelse governance-flow (efter Mads-justering)

**StandardFlow**: Stefan sikrer at:
1. Feature-dokumentation inkluderer compliance-scorecard (fra skabelon i `governance/compliance/feature-scorecard-template.md`)
2. Mads + Trine har reviewet scorecard inden Stefan kalder FGD for final go.
3. Hvis compliance-impact er høj: Dorthe (DPO) har også reviewet (særlig for PII/data-flow-ændringer).
```

**Denne ændring er ikke kritisk-path for denne runde** (kan vente til Camilla-koordination med Stefan), men skal noteres.

---

## 4. Dvivlsspørgsmål til FGD

Ingen åbne spørgsmål fra min side. Alle Mads' specifikationer var klare, og jeg har fulgt dem direkte.

**Eneste bemærkning**: I Mads' rapport §8 stillede han 7 åbne spørgsmål til FGD (antal roller, audit-log-retention, pen-test-cyklus, `skill-comply`-test, DPO-publicering, vendor-rotation, team-start-timing). Disse vil påvirke næste skridt, men de ligger uden for Laila's scope — de skal besvares af FGD.

---

## 5. Filer Oprettet + Opdateret

### Oprettede filer (3 nye rolle-udkast):
- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md`
- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md`
- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/projects/vidensapp/2026-05-05-frederik-pentest.md`

### Opdaterede filer (9 mandater):

**Project-mandater** (i `Min Inbox/`):
- `2026-05-05-sofie-frontend-mandat.md` — +4 hard rules (reviewer, Karen-obligatory, e2e, Speed Insights, GDPR-UI)
- `2026-05-05-henrik-backend-mandat.md` — +4 hard rules (SOD, OpenAPI-review, audit-log-design, RFC 9457)
- `2026-05-05-anders-ai-pipeline-mandat.md` — +4 hard rules (prompt-review, audit-SOD, ZDR, spend-cap)
- `2026-05-05-karen-security.md` — scope-reduktion (DPA → Dorthe) + co-reviewer (Trine) + rapportering-eksplicit
- `2026-05-05-bjarne-hsm-operator.md` — logging-krav + Trine-involvement

**Permanente mandater** (i `.claude/agents/`):
- `laila-hr.md` — co-design også for compliance-roller
- `poul-analytiker.md` — design-docs i governance/
- `camilla-bibliotekar.md` — commit-godkendelse-tracea + migration-gates + memory-governance
- `mads-ciso.md` — audit-uafhængighed eksplicitgjort

---

## 6. Validering Mod Mads' Rapport

✅ Alle 3 nye roller designet per Mads §5.1-5.3
✅ Alle 9 mandater-justeringer implementeret per Mads §6.1-6.10
✅ Pen-test-vendor-shortliste fra Mads §7 integreret i Frederik's rolle
✅ Governance-principper fra Mads §2 refereret (men ikke genimplementeret — de er Mads' område)
✅ Skill-gap-analyse fra Mads §4 noteret (men ikke implementeret — det er Camilla/Poul's omrade)

---

## Næste Skridt

1. **FGD godkender de 3 nye role-udkast** (eller beder om justeringer)
2. **FGD godkender de 9 mandater-opdateringer** (eller beder om justeringer)
3. **Stefan svar på Mads' 7 åbne spørgsmål** (§8 i Mads' rapport) — dette låser dybder i team-struktur og compliance-prioritering
4. **Camilla committer**:
   - De 3 nye rolle-filer fra `Min Inbox/` til `.claude/agents/`
   - Opretter arbejdsmapper: `Team/Dorthe/`, `Team/Trine/`, `Team/Frederik/`
   - Opdaterer `Team/_konventioner.md` med nye navne
   - (Later) Opdaterer Stefan-CLAUDE.md per §3-flaget
5. **Stefan aktiverer de 3 nye roller** i næste session baseret på FGD's svar på Mads' open questions

---

**Underskrift**: Laila
**Dato**: 2026-05-05
