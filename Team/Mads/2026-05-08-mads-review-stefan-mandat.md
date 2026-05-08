---
date: 2026-05-08
author: mads-ciso
subject: Security- og governance-review af stefan-organisator mandat
status: CONDITIONAL GO
references:
  - Min Inbox/_crosscutting/2026-05-08-stefan-organisator-mandat.md
  - .claude/agents/mads-ciso.md
---

# CISO-review: Stefan-organisator mandat

**Dato:** 2026-05-08  
**Reviewer:** Mads (CISO)  
**Dokument reviewet:** `Min Inbox/_crosscutting/2026-05-08-stefan-organisator-mandat.md`  
**Overordnet anbefaling:** **CONDITIONAL GO**

---

## Overordnet vurdering

Mandatet er strukturelt solidt. Governance-sektionen (Mandat-stabilitet) er korrekt formuleret og aligned med SOC 2 CC8.1, ISO 27001 A.5.3 og NIST 800-53 AC-5. De syv fund nedenfor er en blanding af ét kritisk punkt, tre medium-alvorlige og tre minor/kosmetiske. Ingen enkelt-fund er i sig selv en NO-GO, men fund 1 og 3 skal rettes inden aktivering.

---

## Fund

### Fund 1 — KRITISK: Titel "CEO" skaber dataansvar-forvirring (linje 9)

**Reference:** Linje 9: `Du er **Stefan**, FGD-teamets CEO.`

**Problem:**  
"CEO" er i juridisk og regulatorisk kontekst normalt dataansvarlig (data controller) eller ultimativ beslutningsautoritet med personlig hæftelse. GDPR art. 4(7) definerer den dataansvarlige som den der bestemmer formål og midler — det er FGD-personen, ikke en AI-agent. Hvis Stefan tituleres CEO, skabes der en reelt risiko for:

- Uklarhed om hvem der er den egentlige dataansvarlige over for tilsynsmyndighed (Datatilsynet).
- Uklarhed om hvem der har beslutningskompetence ved compliance-eskalering — Stefan er per mandat afhængig af FGDs GO; en CEO er ikke det.
- SOD-konflikt: CEO-rollen som ultimativ autoritet er uforenelig med Stefans eksplicitte "Stefan venter på FGDs GO"-regel.
- ISO 27001 A.5.3 kræver at roller, ansvar og beføjelser er klart opdelt og kommunikeret. "CEO" sender et signal om kompetence der ikke matcher mandatets faktiske beføjelser.

**Anbefaling (blokkerende):**  
Ret linje 9 til `Du er **Stefan**, FGD-teamets AI-organisator.` eller tilsvarende neutral titel. Frontmatter `name: stefan-organisator` er korrekt — body skal matche. FGD er den eneste CEO/dataansvarlige i dette setup.

---

### Fund 2 — MEDIUM: Stefans "fuld adgang" og web-søgning vs. least privilege (linje 27-28)

**Reference:** Linje 27-28: "Stefan har fuld adgang til både global og project viden" og "Søge [...] på websites"

**Problem:**  
SOC 2 CC6.3 kræver least-privilege: adgangsrettigheder begrænset til hvad der er nødvendigt for opgaven. "Fuld adgang til al viden" er teknisk nødvendigt for Stefans orkestreringsrolle, men formuleringen mangler en eksplicit afgrænsning af hvad denne adgang *ikke* inkluderer (fx skrive-adgang, produktionskode, database). Uden afgrænsning kan "fuld adgang" fejlfortolkes som write-adgang.

Web-søgning (via `Bash` og `Grep`-toolset) er acceptabelt for en orkestreringsrolle, men mandatet bør eksplicit konstatere at Stefan *ikke* må handle på eksterne data selv — kun videregive til Poul for validering. Ellers er der en omgåelsesrisiko for Pouls analyse-monopol.

**Anbefaling (ikke-blokkerende, men bør rettes):**  
Tilføj præcisering: "Stefan har fuld *læse*-adgang til al projekt- og global viden. Stefan søger på websites udelukkende for at orientere sin orkestrering — al dyb research og kildevalidering delegeres til Poul." Dette lukker least-privilege-fortolkningsgabet og bevarer SOD.

---

### Fund 3 — KRITISK: 4-eyes for Poul er udefineret (linje 72)

**Reference:** Linje 72: "Implementer 4-eyes princippet" (Poul-rækken i Delegerings-hierarki-tabel)

**Problem:**  
4-eyes-princippet er et centralt SOD-kontrol (ISO 27001 A.5.3, NIST 800-53 AC-5). Formuleringen "Implementer 4-eyes princippet" er uden specifikation af:

- Hvem de 4 øjne er (fx "Poul + Mads" eller "Poul + Stefan" eller "Poul + FGD").
- Hvornår det aktiveres (alle Poul-analyser, kun arkitektur-analyser, kun FIPS/compliance-relevante?).
- Hvad review-gaten er (hvad skal godkendes inden Poul-analysen sendes videre?).

Et udefineret 4-eyes er kontrolmæssigt tomt — det kan ikke auditeres og vil ikke blive overholdt i praksis. Under SOC 2 Type II vil en auditor afvise en kontrol uden operationel definition.

**Anbefaling (blokkerende):**  
Præcisér: "4-eyes for Poul gælder for arkitektur-analyser og compliance-kritiske analyser: Poul producerer rapport → Mads peer-reviewer inden Stefan præsenterer til FGD. For øvrige analyser (behovs-kortlægning, vendor-research): Stefan godkender direkte. Mads noteres i audit-trail ved peer-review."

---

### Fund 4 — MEDIUM: DORA under Trine er forkert placeret (linje 76)

**Reference:** Linje 76, Trine-rækken: "Trine: DORA, ISO 27001 + SOC 2 Type II audit"

**Problem:**  
DORA (EU 2022/2554 — Digital Operational Resilience Act) er en finansiel sektors regulering under finanstilsynsmyndigheder (EBA, ESMA, ECB), ikke en generel IT-compliance-standard. DORA gælder for finansielle enheder og kritiske tredjepartsudbydere. Den adresserer:

- ICT-risikostyring (kræver CISO-niveau-ansvar).
- Incident-rapportering til finansmyndigheder.
- Digital operationel resilience-test (penetrationstest-krav kaldet TLPT).
- Tredjeparts-risikostyring (ICT-leverandørkontrakter).

Dette er CISO-kompetence (Mads), ikke compliance-auditor-kompetence (Trine). Trine ejer evidensindsamling og SoA-mapping; DORA-compliance kræver aktiv risikostyring, kontraktkrav og myndighedskontakt — det er Mads' domæne. Desuden: placering af DORA her forudsætter at FGD-projektet er en finansiel enhed eller kritisk tredjepartsudbyder under DORA — dette er ikke verificeret i mandatet.

**Anbefaling (ikke-blokkerende, men kræver afklaring):**  
Flyt DORA-ansvar til Mads-tabellen med note: "DORA kun relevant hvis FGDs kunder er finansielle enheder under DORA-scope." Trine kan varetage DORA-audit-evidens-indsamling, men aldrig DORA-risikostyringen selv. Alternativt: fjern DORA fra Trine-rækken og behold i Mads' faseliste.

---

### Fund 5 — MEDIUM: Ansvarsfordeling-inkonsistens — Dorthe kanal (linje 110 vs. linje 75)

**Reference:**  
- Linje 75 (Delegerings-hierarki): Dorthe er direkte kanal for GDPR-compliance.  
- Linje 110 (Ansvarsfordeling): "Stefan eskalerer GDPR-spørgsmål til Mads, som koordinerer med Dorthe."

**Problem:**  
De to linjer beskriver to forskellige flows. Delegerings-hierarki-tabellen antyder at Stefan kan kalde Dorthe direkte (via Agent). Ansvarsfordelingstabellen siger at GDPR altid går via Mads. Det er en SOD-tvetydighed: hvis Stefan omgår Mads og går direkte til Dorthe, mister Mads sit compliance-overblik. Mads ejer compliance-integrationen; Dorthe er specialist-input til Mads. Direkte Stefan→Dorthe-kanal for GDPR svækker Mads' oversigt og skaber risiko for inkonsistente vurderinger.

**Anbefaling (ikke-blokkerende, men bør rettes):**  
Linje 110 er korrekt fra SOD-perspektiv. Ret Delegerings-hierarki-tabellen til at specificere: "Dorthe: GDPR-compliance, *via Mads som koordinator*. Stefan kalder Dorthe direkte kun hvis Mads er eksplicit involveret eller ved selvstændig GDPR-forbruger-henvendelse." Ensret de to tabeller.

---

### Fund 6 — MINOR: Project-roller governance er underspecificeret (linje 77)

**Reference:** Linje 77: "Stefan opsætter project organisationer [...] Mads, og Dorte har en medarbejder med der sikrer compliance [...] og som sikrer Mads og Dorte godkender det endelige produkt"

**Problem:**  
Formuleringen er intentionelt bred, men mangler review-gate-definition for project-roller:

- Hvornår er en project-rolle "godkendt"? Er det nok at Mads og Dorthe har en repræsentant med, eller kræves eksplicit skriftlig godkendelse?
- Hvad er minimum-krav til en project-roles mandat for at passere Mads-gate (fx: hard rule om ingen direkte prod-adgang, audit-log-krav, tool-begrænsning)?
- Er project-rolle-mandater underlagt samme change management som permanente roller (CC8.1)?

Under SOC 2 CC8.1 skal change management-gaten for konfigurationsændringer være eksplicit og verificerbar. "Mads og Dorte godkender" uden form-krav kan ikke auditeres.

**Anbefaling (kosmetisk/minor):**  
Tilføj minimumskrav: "Project-roller skal have skriftlig compliance-godkendelse fra Mads (security/SOD-vurdering) og Dorthe (GDPR-vurdering) inden aktivering. Godkendelse dokumenteres i `governance/change-requests/`. Project-rolle-mandater er underlagt samme change management som permanente roller."

---

### Fund 7 — MINOR: Governance-sektionen (Mandat-stabilitet) er korrekt men mangler Trine-audit-trail-reference

**Reference:** Linje 195-199 (Mandat-stabilitet-processen)

**Problem:**  
Processen er korrekt og aligned med SOC 2 CC8.1, ISO 27001 A.5.3 SoD og NIST 800-53 AC-5. Ét mindre hul: punkt 6 ("Trine logger change-event i audit-trail") er identisk med Mads' eget mandat — dette er godt. Men "audit-trail" er ikke specificeret som et konkret dokument/mappe. Uden adresse kan Trine (og auditor) ikke finde evidens.

**Anbefaling (kosmetisk):**  
Tilføj: "Trine logger change-event i `governance/audit-trail/<dato>-stefan-<felt>-change.md`" — præcis sti, ikke abstrakt "audit-trail".

---

## Opsummering

| Fund | Sværhedsgrad | Blokkerende | Anbefalet handling |
|---|---|---|---|
| 1 — "CEO"-titel | KRITISK | JA | Ret til "AI-organisator" inden aktivering |
| 2 — "Fuld adgang" / web-søgning | MEDIUM | NEJ | Tilføj præcisering om læse-adgang og Poul-delegering |
| 3 — 4-eyes udefineret | KRITISK | JA | Definer hvem, hvornår, hvad for Poul-reviews |
| 4 — DORA under Trine | MEDIUM | NEJ | Flyt DORA til Mads; Trine beholder evidens-rolle |
| 5 — Dorthe-kanal-inkonsistens | MEDIUM | NEJ | Ensret til Mads-koordineret flow |
| 6 — Project-roller ingen review-gate | MINOR | NEJ | Tilføj minimum-krav og dokumentations-adresse |
| 7 — Audit-trail uden sti | MINOR | NEJ | Tilføj konkret filsti |

**Samlet anbefaling: CONDITIONAL GO.**  
Fund 1 og 3 skal rettes inden mandatet aktiveres. Fund 2, 4 og 5 bør rettes i samme omgang for at undgå fremtidige SOD-tvister. Fund 6 og 7 kan rettes i næste revision.

---

*Mads, CISO — 2026-05-08*  
*Kopi til: Dorthe (GDPR-review af fund 5 og 4), FGD (beslutning).*
