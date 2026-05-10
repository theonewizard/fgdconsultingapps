---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: 4-eyes GDPR-review af Trine compliance-auditor-mandat
reviewet-dokument: Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md
status: CONDITIONAL GO
---

# 4-eyes Review: Trine — Compliance Auditor

**Dorthe (DPO) | GDPR-vinkel**

---

## Konklusion

**CONDITIONAL GO** — med 3 krav der skal adresseres før Camilla committer `.claude/agents/trine-compliance-auditor.md`.

---

## Fund

### 1. GDPR vs. ISO 27001 — ansvarsgrænse

**Vurdering: TILFREDSSTILLENDE — med én præcisering.**

Mandatet er tydeligt: Trine ejer ISO 27001 + SOC 2; Dorthe ejer GDPR. Der er ingen overlap i ejerforhold. Ansvarsfordelings-tabellen nævner ikke Dorthe — det er korrekt; Trine og Dorthe arbejder parallelt under Mads' governance-ramme, og det er Mads der koordinerer.

**Præcisering krævet (men ikke blocker):** SOC 2 Trust Service Criteria P1-P8 (Privacy) overlapper med GDPR. Trine nævner i §Mission at "PI + Privacy overvejes senere" — det er fint, men når Privacy-kriterierne aktiveres, skal mandatet eksplicit kræve at Dorthe co-reviewer SOC 2 Privacy-mapping. Anbefaling: tilføj en note i Trines mandat under `## Hard rules` om at SOC 2 Privacy-kriterierne (P1-P8) kræver Dorthes input før de markeres ✅.

---

### 2. Audit-log med persondata — GDPR-krav

**Vurdering: KRITISK MANGEL — skal adresseres. (Krav 1)**

Trine indsamler evidens fra "CI/CD logs, audit-logs, access-reviews, test-rapporter". Audit-logs og access-reviews vil i praksis indeholde persondata: bruger-IDs, email-adresser, IP-adresser, timestamps koblet til identificerbare brugere.

Mandatet adresserer ikke:

- **Retsgrundlag** for Trines behandling af persondata i evidens-indsamling (GDPR art. 6). Trine er data-processor (intern), men behandlingen skal beskrives i ROPA med formål "compliance-auditering", retsgrundlag "legitim interesse / retlig forpligtelse (ISO 27001/SOC 2)", og retention begrænset til audit-periodens behov.
- **Retention-begrænsning** (art. 5(1)(e)): Mandatet nævner evidens-arkivering uden at sætte grænser. SOC 2-evidens kræver typisk 12 måneder observation + buffer; evidens med persondata må ikke opbevares længere end nødvendigt.
- **Dataminimering** (art. 5(1)(c)): Trine bør anvende pseudonymiserede eller aggregerede logs til evidens, hvor det er muligt — persondata skal kun indgå hvis det er nødvendigt for at bevise kontroleffektivitet.

**Krav 1:** Tilføj en `## GDPR-grænseflader`-sektion i Trines mandat der præciserer: (a) evidens med persondata behandles under Dorthes ROPA-ansvar, (b) Trine anvender dataminimering og pseudonymisering hvor muligt, (c) retention for persondata-evidens sættes til Dorthe-anbefalet max (foreløbig anbefaling: 24 måneder fra log-dato, reviseres ved SOC 2-periodestart).

---

### 3. Breach-respons — overlap med Dorthe

**Vurdering: POTENTIELT OVERLAP — kræver afklaring. (Krav 2)**

Trines mandat nævner ikke breach-respons eksplicit — det er positivt. Men Trines `Stack-kompetencer` inkluderer "Security Incident Management" (ISO 27001 Annex A.5.24-5.28), og Trine vil uundgåeligt modtage adgang til incident-logs ved audit.

Risiko: Trine opdager under sin interne audit et aktivt sikkerhedsbrud — hvad gør hun? Mandatet specificerer ikke eskalerings-sti til Dorthe ved breach-mistanke.

**Krav 2:** Tilføj til `## Hard rules`: "Hvis Trine ved evidens-indsamling opdager indikation på sikkerhedsbrud eller uautoriseret adgang til persondata, eskalerer hun **omgående** til Mads + Dorthe parallelt (ikke sekventielt). Dorthe vurderer GDPR art. 33-notifikationspligt inden 72 timer. Trine stopper ikke sin audit-dokumentation, men afventer Dorthes vurdering inden evidens for det konkrete incident arkiveres."

---

### 4. Rapporteringsstruktur — DPO-uafhængighed

**Vurdering: TILFREDSSTILLENDE.**

Trine rapporterer til Mads (compliance-status) og Stefan (blockere). Dette er korrekt: Trine er auditor, ikke DPO — GDPR art. 38(3)-uafhængighedskravet gælder specifikt DPO-rollen (Dorthe), ikke compliance-auditor-rollen.

At Trine rapporterer til Mads er strukturelt rigtigt: Mads definerer kontrol-rammer, Trine verificerer dem. Dorthe er uafhængig af Mads; Trine er det ikke, og behøver ikke at være det. Strukturen er sund.

---

### 5. Projekt-rolle-navne i mandatet

**Vurdering: TILFREDSSTILLENDE.**

Mandatet overholder konventionen fra `memory/_crosscutting/feedback_permanent_vs_project_separation.md`. Ansvarsfordelings-tabellen bruger udelukkende generiske termer: "backend-udvikler", "frontend-udvikler", "AI-pipeline-udvikler", "security-reviewer", "HSM-operatør" — alle med noten "(projekt-rolle, defineret af Mads)". Ingen konkrete projekt-rolle-navne (Karen, Bjarne, Henrik, Anders, Sofie) forekommer i mandatteksten.

**En enkelt observation** (ikke blocker): I `## Anti-patterns` skrives "Trine auditer; Karen adviserer på sikkerhed". Karen er et konkret projekt-rolle-navn. **(Krav 3):** Erstat "Karen" med "security-reviewer (project-rolle)".

---

## Opsummering af krav

| # | Krav | Prioritet |
|---|---|---|
| 1 | Tilføj GDPR-grænseflader-sektion: ROPA-ansvar, dataminimering, retention-max for evidens med persondata | **BLOCKER** |
| 2 | Tilføj breach-eskalerings-regel til hard rules: Trine → Mads + Dorthe parallelt ved breach-indikation | **BLOCKER** |
| 3 | Erstat "Karen" med "security-reviewer (project-rolle)" i Anti-patterns | Minor |

Krav 1 og 2 er GDPR-retlige krav og ikke valgfrie. Krav 3 er en konventions-overtrædelse.

---

## Anbefalet næste skridt

Laila (eller Mads) opdaterer Trine-mandatudkastet med ovenstående 3 krav. Dorthe re-reviewer opdateret version inden Camilla committer.

*Dorthe — DPO, 2026-05-08*
