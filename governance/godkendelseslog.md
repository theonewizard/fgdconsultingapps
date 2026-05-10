# Godkendelseslog

Løbende log over alle store beslutninger truffet under 4-eyes princippet.
Vedligeholdes af Camilla. Stefan er ansvarlig for at bede om opdatering efter hvert FGD-GO.

Format pr. entry:
- **Dato**: YYYY-MM-DD
- **Beslutning**: hvad blev godkendt
- **Reviewer 1**: navn + rolle
- **Reviewer 2**: navn + rolle
- **FGD GO**: bekræftelse
- **Logget af**: Camilla

---

## 2026-05-09 — Aktivering af FGDCorePKI Application Owner + Application Provider projekt-roller

- **Beslutning**: Aktivering af to nye projekt-roller for FGDCorePKI PKI-infrastruktur:
  - `.claude/agents/fgdcorepki-application-owner.md` (sonnet, blue)
  - `.claude/agents/fgdcorepki-application-provider.md` (opus, cyan)
- **Reviewer 1**: Mads (CISO) — 4-eyes: CONDITIONAL GO (5 blokerende); final-check: CONDITIONAL GO (FC2+FC4); re-check: GO
- **Reviewer 2**: Dorthe (DPO) — 4-eyes: STOP (8 blokerende + 6 betingede); final-check: GO (alle fund adresseret i v2)
- **Udkast-spor**: Laila v1 → v2 (Mads+Dorthe 4-eyes) → v2.1 (FC2+FC4-rettelser)
- **Strategiske FGD-beslutninger** (2026-05-09):
  - PKI scope: Intern PKI (ikke CABF public trust)
  - HSM-operatør: Ny dedikeret person til FGDCorePKI (Bjarne kører ikke begge)
  - AO: Delegeret intern FGD-person (navn TBD)
  - AP vs. Engineering Team Lead: Option B — to separate fra dag 1
- **FGD GO**: "/loop 10" — FGD, 2026-05-09 (fortolket som GO i kontekst af direkte spørgsmål)
- **Dorthes DPIA-krav i Fase 0** (ikke blokkerende for aktivering):
  1. Key ceremony video/lydoptagelse med lang retention
  2. OCSP/CRL CDN-hosting (afhænger af arkitekturvalg)
- **Logget af**: Stefan (Camilla committer)

## 2026-05-09 — Aktivering af Application Owner + Application Provider projekt-roller

- **Beslutning**: Aktivering af to nye projekt-roller for applikationsudvikling i Knowlagecentral:
  - `.claude/agents/application-owner.md` (sonnet)
  - `.claude/agents/application-provider.md` (opus)
- **Reviewer 1**: Mads (CISO) — 4-eyes: CONDITIONAL GO; final-check: GO (5 blokerende rettelser indarbejdet i v2)
- **Reviewer 2**: Dorthe (DPO) — 4-eyes: CONDITIONAL GO; final-check: GO (8+6 hard rules indarbejdet i v2)
- **Udkast-spor**: Laila v1 → v2 (10 blokerende rettelser indarbejdet samlet på tværs af Mads + Dorthe)
- **FGD GO**: "ja" — FGD, 2026-05-09
- **Logget af**: Camilla

## 2026-05-08 — Dorthe DPO mandat

- **Beslutning**: Dorthe DPO mandat godkendt og aktiveret som permanent rolle (`.claude/agents/dorthe-dpo.md`)
- **Reviewer 1**: Mads (CISO) — CONDITIONAL GO; 6 fund indarbejdet
- **Reviewer 2**: Poul (analytiker, substitueret for Dorthe pga. SOD-konflikt) — CONDITIONAL GO; 5 fund indarbejdet
- **FGD GO**: "Ja og noter samtidigt at naar jeg godkender skal filen fjernes fra min inbox." — FGD, 2026-05-08
- **Logget af**: Camilla

## 2026-05-08 — Stefan-organisator mandat

- **Beslutning**: Stefan-organisator mandat godkendt og aktiveret som permanent rolle (`.claude/agents/stefan-organisator.md`)
- **Reviewer 1**: Mads (CISO) — CONDITIONAL GO; fund indarbejdet (CEO-titel bekræftet af FGD, 4-eyes tilføjet, DORA-placering noteret)
- **Reviewer 2**: Dorthe (DPO) — CONDITIONAL GO; fund indarbejdet (DPO-routing art. 38(3) rettet, dataminimering tilføjet, memory-forbud mod persondata om rigtige mennesker)
- **FGD GO**: "Ja jeg godkender Stefans rolle beskrivelse og mandat" — FGD, 2026-05-08
- **Logget af**: Camilla

## 2026-05-08 — Camilla bibliotekar v2 mandat

- **Beslutning**: Camilla v2-mandat aktiveret (`.claude/agents/camilla-bibliotekar.md` opdateret)
- **Reviewer 1**: Mads (CISO) — GO
- **Reviewer 2**: Poul (analytiker, substitueret for Camilla pga. SOD) — GO efter stavefejl + minor rettet
- **FGD GO**: "godkendt" — FGD, 2026-05-08
- **Logget af**: Camilla

## 2026-05-08 — Vibeke commit-reviewer mandat

- **Beslutning**: Vibeke aktiveret som permanent commit-gate (`.claude/agents/vibeke-commit-reviewer.md`)
- **Reviewer 1**: Mads (CISO) — GO
- **Reviewer 2**: Dorthe (DPO) — GO
- **FGD GO**: "godkendt" — FGD, 2026-05-08
- **Logget af**: Camilla

## 2026-05-08 — Trine compliance-auditor mandat

- **Beslutning**: Trine aktiveret som permanent compliance-auditor (`.claude/agents/trine-compliance-auditor.md`)
- **Reviewer 1**: Mads (CISO) — GO efter rapporteringslinje + 4-eyes scorecard rettet
- **Reviewer 2**: Dorthe (DPO) — GO efter arbejdsgang-inkonsistens rettet
- **FGD GO**: "godkendt" — FGD, 2026-05-08
- **Logget af**: Camilla

## 2026-05-08 — Governance-tightening: invalidity-klausul + Hard Rule 8

- **Beslutning**: FGD-godkendelse af mandat-ændringer ugyldig uden Mads + Dorthe skriftlig vurdering — invalidity-klausul tilføjet til CLAUDE.md og alle 8 permanent-mandate. Hard Rule 8 (README.md-vedligehold) formaliseret i Camilla-mandat.
- **Reviewer 1**: Mads (CISO) — GO med to betingede præciseringer (eskalerings-undtagelse kryds-reference, Dorthe-aktiverings-note), begge inkorporeret
- **Reviewer 2**: Dorthe (DPO) — CONDITIONAL GO med betingelser (svarfrist, nødsituations-undtagelse), begge inkorporeret
- **FGD GO**: "Ja — FGD-godkendelse af mandat-ændringer er ugyldig uden begge vurderinger foreligger i governance/change-requests/. Camilla må ikke committe på baggrund af en GO der mangler Mads+Dorthe dokumentation. Svarfristen og nødsituations-undtagelse skal være klar." — FGD, 2026-05-08
- **Berørte filer**: `.claude/agents/stefan-organisator.md`, `.claude/agents/camilla-bibliotekar.md`, `.claude/agents/laila-hr.md`, `.claude/agents/poul-analytiker.md`, `.claude/agents/mads-ciso.md`, `.claude/agents/dorthe-dpo.md`, `.claude/agents/trine-compliance-auditor.md`, `.claude/agents/vibeke-commit-reviewer.md`, `CLAUDE.md`
- **Logget af**: Camilla
