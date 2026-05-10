---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: Final 4-eyes review — Trine compliance-auditor mandat (efter rettelser)
kilde: Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md
foregående-review: Team/Mads/2026-05-08-mads-4eyes-trine.md
status: CONDITIONAL GO
---

# Final 4-eyes Review: Trine — Compliance Auditor

**Reviewer:** Mads (CISO)
**Dato:** 2026-05-08
**Objekt:** Revideret `.claude/agents/trine-compliance-auditor.md` (kodeblok i Min Inbox-fil)

---

## Verdict: CONDITIONAL GO

Tre ud af fire kritiske fund er løst. Et punkt er delvist løst med en residual inkonsistens der skal rettes inden commit.

---

## Verifikation af fire specificerede punkter

### 1. Rapporteringslinje: Trine→Stefan→FGD (Mads = cc)

**Status: DELVIST LØST**

Hard rules §6 er korrekt: "Rapporterer til Stefan (primær linje) og dermed til FGD. Mads modtager kopi (cc). Ikke til udvikler-teamet direkte."

Inkonsistens: Arbejdsgang §4 (rapportér-trin) lyder stadig: "Rapportér: månedsrapport til **Mads** + kort opsummering til Stefan hvis blocker."

Dette modstrider hard rule og den korrekte rapporteringslinje. Arbejdsgang §4 skal rettes til: "Rapportér: månedsrapport til **Stefan** (Mads cc) + eskalér til Stefan ved blocker." Den nuværende formulering genskaber det SOD-problem der var i foregående review.

**Påkrævet rettelse:** Arbejdsgang §4 — erstat "til Mads" med "til Stefan (Mads cc)".

### 2. Evidence-logs: immutabilitet/tamper-evidence specificeret

**Status: LØST**

Hard rules indeholder nu: "Evidence-logs skal være tamper-evident; når Trine indsamler og arkiverer evidence, anvendes write-once-metode (fx kryptografisk hash-log, immutable storage). Hverken Trine, Camilla eller andre roller må modificere arkiveret evidence. Camilla eksekverer arkivering; Trine verificerer integritet."

Dækker: write-once-metode, konkrete eksempler (kryptografisk hash-log, immutable storage), adgangsrestriktion (ingen roller må modificere), og SOD (Camilla eksekverer, Trine verificerer). Tilfredsstillende for SOC 2 Type II evidens-krav.

### 3. Trine som navngiven part i 4-eyes godkendelsesflow

**Status: DELVIST LØST**

Wrapper-filen (efter mandatblokken) indeholder korrekt: "4-Eyes Compliance Scorecard — Trine leverer compliance-scorecard til Stefan inden FGD's GO på alle compliance-kritiske ændringer."

Problem: Dette scorecard-krav er **ikke** inkluderet i mandatblokken (kodeblokken der committertes som `.claude/agents/trine-compliance-auditor.md`). Når Camilla committer, forsvinder wrapper-filens tekst — kun kodeblokken persisteres. En ekstern revisor der læser `.claude/agents/trine-compliance-auditor.md` vil ikke finde Trines navngivne pligt i godkendelsesflowet.

**Påkrævet rettelse:** Tilføj til Hard rules i kodeblokken: "Ved feature-godkendelser og integrationer med compliance-relevans leverer Trine compliance-scorecard til Stefan inden FGD's GO. Scorecard angiver kontrol-status (✅/⚠/❌) for påvirkede kontroller, åbne findings, og Trines anbefaling (klar/ikke klar til go)."

### 4. "Karen" fjernet fra anti-patterns

**Status: LØST**

Grep på hele filen returnerer nul hits på "Karen". Ingen project-rolle-navne i anti-patterns eller ansvarsfordelings-tabel. Generiske termer ("Backend-udvikler", "Security-reviewer", "HSM-operatør") bruges konsekvent med parentetisk "(projekt-rolle, defineret af Mads)".

---

## Nye fund

### A. Titel-fejl i §5 (Arbejdsgang)

Mission §5 lyder: "Internal audit-cycle — før SOC 2 Type II **Type II**-observation-periode". "Type II" optræder to gange. Kosmetisk, men bør rettes inden commit.

### B. Breach-eskalering-clause — korrekt

Hard rules indeholder: "Hvis Trine under evidence-indsamling opdager indikation på databrud, eskalerer hun omgående og parallelt til Mads (CISO) og Dorthe (DPO). GDPR art. 33's 72-timers-frist løber fra kendskab — Trine må ikke vente på Mads' bekræftelse."

Dette er korrekt og en forbedring ift. foregående version. Ingen rettelse.

### C. GDPR-evidence-clause er tilstrækkelig

Hard rules dækker: retsgrundlag (art. 6(1)(f)), retention-begrænsning, dataminimering, og koordinering med Dorthe ved nye evidence-kategorier. Tilfredsstillende.

---

## Samlet fund-tabel

| # | Punkt | Status | Handling |
|---|---|---|---|
| 1 | Rapporteringslinje Trine→Stefan→FGD | DELVIST LØST | Ret Arbejdsgang §4: "til Mads" → "til Stefan (Mads cc)" |
| 2 | Evidence-logs immutabilitet/tamper-evidence | LØST | Ingen rettelse |
| 3 | Trine navngiven i 4-eyes godkendelsesflow | DELVIST LØST | Flyt scorecard-krav ind i kodeblokken (Hard rules) |
| 4 | "Karen" fjernet fra anti-patterns | LØST | Ingen rettelse |
| A | Dubleret "Type II" i Mission §5 | Nyt | Kosmetisk rettelse |

---

## Næste skridt

1. **Laila** udfører to minimale rettelser i kodeblokken:
   - Arbejdsgang §4: "til Mads" → "til Stefan (Mads cc)"
   - Hard rules: tilføj scorecard-pligt (tekst ovenfor under punkt 3)
   - Kosmetisk: fjern dubleret "Type II" i Mission §5
2. **Mads** kvitterer GO ved næste forevisning (kan gøres i en linje — kun de tre linjer verificeres).
3. **FGD GO** → Camilla committer `.claude/agents/trine-compliance-auditor.md`.
