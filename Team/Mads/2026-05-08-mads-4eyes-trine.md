---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: 4-eyes review — Trine compliance-auditor mandat
kilde: Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md
status: CONDITIONAL GO
relateret: governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md
---

# 4-eyes Review: Trine — Compliance Auditor

**Reviewer:** Mads (CISO)
**Dato:** 2026-05-08
**Objekt:** Foreslået `.claude/agents/trine-compliance-auditor.md` (indlejret i Min Inbox-fil)

---

## Verdict: CONDITIONAL GO

Trines mandat er solid i kernen. SOC 2 + ISO 27001 scope er dækkende; evidence-metodik er tilstrækkelig til intern audit. Tre punkter skal rettes/afklares **inden Camilla committer**.

---

## Nummererede fund

### 1. Audit-uafhængighed — DELVIS OVERHOLDT (kritisk justering nødvendig)

**Fund:** Mandatets Arbejdsgang §1 lyder "Modtag brief fra Mads eller Stefan". Trines mission-tekst siger hun "rapporterer til Mads på overordnet niveau". Kombinationen skaber et afhængighedsforhold: Trine modtager arbejdsordrer fra den person, hvis kontrol-rammer hun skal verificere.

**SOC 2-krav:** SOC 2 Type II kræver ikke fuld ekstern revisoruafhængighed for intern audit, men kræver at den interne auditors rapporteringslinje er fri fra operational-management-indflydelse på audit-resultaterne. COSO-framework (som Trine korrekt refererer) kræver "organizational objectivity".

**Problem:** Hvis Mads briefer Trine og Trine rapporterer til Mads, kan Mads i praksis styre hvad der audites og hvad der rapporteres opad — dette svækker kontrollen i en ekstern SOC 2-revisors øjne.

**Anbefalet rettelse:** Tilføj eksplicit tekst: "Trine rapporterer compliance-status **direkte til FGD via Stefan** — ikke primært til Mads. Mads modtager kopi. Trine kan modtage teknisk kontekst fra Mads, men bestillingsret til at ændre audit-scope eller konklusioner tilhører kun FGD/Stefan." Flowet: `Trine → Stefan → FGD (Mads cc)`.

### 2. SOC 2 + ISO 27001 scope — TILSTRÆKKELIGT

**Fund:** Trine dækker alle seks Trust Service Criteria-kategorier (S, C, A, PI, Privacy), 93 Annex A controls, og COSO-framework. Stack-kompetencer-listen er korrekt og fuldstændig. Fase 0/1/2-ansvarsfordeling er realistisk.

**Positivt:** Trine skelner korrekt mellem SOC 2 Type II observation-periode (6+ måneder) og intern audit. Dette er ofte misforstået — Trine har det rigtigt.

**Ingen rettelse nødvendig.**

### 3. Evidence-indsamling — TILSTRÆKKELIG MED ETT HULLER

**Fund:** Procedure for evidence-indsamling er beskrevet (CI/CD logs, audit-logs, access-reviews, test-rapporter, dokumentation). Fase 1 inkluderer "Internal audit 1 (soft): kørt uden at udvikler-teamet ved det" — dette er korrekt surprise-audit-metodik.

**Hul:** Mandatet nævner ikke **immutabilitet** og **tamper-evidence** for logs. En ekstern SOC 2-revisor vil spørge: "Kan Trine selv eller Camilla manipulere de logs Trine bruger som evidens?" Der er ingen procedure for at sikre log-integritet (fx write-once storage, hash-chaining, eller ekstern log-eksport).

**Anbefalet tilføjelse til Hard rules:** "Evidence-logs skal gemmes i uforanderlig form (write-once eller eksporteret til separat arkiv) og ikke kunne modificeres af de roller der producerer dem (Camilla, udvikler-roller) eller indsamler dem (Trine)."

### 4. SOD med Camilla — KORREKT

**Fund:** Mandatet beskriver korrekt SOD: Camilla leverer raw data (CI-logs, backup-logs, access-reviews); Trine analyserer. Camilla implementerer og committer; Trine auditerer.

**SOD-vurdering:** Dette er korrekt separation. Trine auditer Camillas git-historik og CI-logs, men kan ikke selv ændre dem. Camilla kan ikke styre hvad Trine konkluderer. Ansvarsfordelings-tabellen er klar og konsekvent.

**Ingen rettelse nødvendig.**

### 5. Projekt-rolle-navne — OVERHOLDT MED EN UNDTAGELSE

**Fund:** Hard rules §2 siger eksplicit "Trine's mandat refererer ikke specifikke project-rolle-navne." Dette overholdes i Hard rules-sektionen.

**Undtagelse:** Ansvarsfordelings-tabellen bruger generiske termer som "Backend-udvikler", "Frontend-udvikler", "AI-pipeline-udvikler" — og parentetisk "(projekt-rolle, defineret af Mads)". Dette er acceptabelt og overholdt efter konventionen.

**Positivt:** Brug af parentetisk "(projekt-rolle, defineret af Mads)" er den korrekte teknik til at markere project-scope uden at låse konkrete navne ind.

**Ingen rettelse nødvendig.**

### 6. Mandat-stabilitet — KORREKT

**Fund:** Mandat-stabilitets-sektionen er identisk struktureret med Mads' eget mandat. Låste kontrolpunkter (model, tools, hard rules, description) er alle listet med korrekt eskaleringspath (Mads + Dorthe + FGD). Self-modificerings-forbud er eksplicit. Trine logger change-events i audit-trail (korrekt — hun ejer audit-trail).

**Ingen rettelse nødvendig.**

### 7. 4-eyes integration i godkendelseslog-flowet — MANGEL (kritisk)

**Fund:** Mandatet beskriver Trines rolle som den der "logger change-event i audit-trail" i mandat-stabilitets-processen. Dette er korrekt. Men mandatet specificerer **ikke eksplicit** at Trine er en del af godkendelseslog-flowet for features/integrationer — dvs. at Trines compliance-status-rapport skal foreligge **inden** FGD giver GO til en ny fase eller feature med compliance-relevans.

**Problem:** Stefans CLAUDE.md (governance-sektion) siger "Stefan samler compliance-scorecard fra Mads/Trine, præsenterer samlet til FGD." Men Trines eget mandat nævner ikke dette flow. En ekstern revisor vil forvente at auditoren er en navngiven part i change-godkendelses-processen.

**Anbefalet tilføjelse til Hard rules:** "Ved feature-godkendelser med compliance-relevans leverer Trine et compliance-scorecard til Stefan inden FGD's GO. Scorecard angiver: kontrol-status (✅/⚠/❌) for de påvirkede kontroller, evt. åbne findings, og Trines anbefaling (klar/ikke klar)."

---

## Opsummering af påkrævede rettelser inden commit

| # | Fund | Kritikalitet | Rettelse |
|---|---|---|---|
| 1 | Rapporteringslinje Trine→Mads svækker audit-uafhængighed | Kritisk | Primær rapportering: Trine→Stefan→FGD; Mads cc |
| 3 | Manglende immutabilitets-krav til evidence-logs | Moderat | Tilføj hard rule om write-once/ekstern arkivering |
| 7 | Trine ikke eksplicit positioneret i godkendelseslog-flow | Moderat | Tilføj hard rule om compliance-scorecard til Stefan |

Fund 2, 4, 5, 6: ingen rettelser nødvendige.

---

## Næste skridt

1. **Laila** retter de tre punkter ovenfor i mandatudkastet.
2. **Mads 4-eyes re-check** (kan gøres hurtigt — kun de tre rettede sektioner).
3. **FGD GO** → Camilla committer `.claude/agents/trine-compliance-auditor.md` + opretter `Team/Trine/`.
4. Trine starter Fase 0: SoA + Trust Service Criteria mapping.
