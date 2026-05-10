---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: Final 4-eyes GDPR-review — Trine compliance-auditor-mandat
reference: Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md
status: final
---

# Final GDPR-review: Trine — Compliance Auditor

## Samlet vurdering: CONDITIONAL GO

Alle tre specificerede GDPR-fund er løst. Et nyt, mindre inkonsistens-fund kræver rettelse før commit.

---

## Verificering af de tre specificerede fund

### 1. GDPR-grænseflade for evidence-logs (art. 6, retention, dataminimering)

**LØST**

Hard rule (linje 77 i mandatteksten) indeholder alle tre elementer:

- **Retsgrundlag**: art. 6(1)(f) legitim interesse — eksplicit angivet med begrundelse ("compliance-forpligtelse").
- **Retention**: "typisk 3–7 år; Dorthe fastsætter præcis retention" — delegation til DPO etableret korrekt. Dorthe accepterer dette ansvar.
- **Dataminimering**: "Trine indsamler kun det minimum der er nødvendigt for at dokumentere kontrol-effektiviteten" — eksplicit og proportionalitetsorienteret formulering.
- **ROPA-koordinering**: "Trine koordinerer med Dorthe (DPO) når nye evidence-kategorier tilføjes" — ROPA-opdateringsloop er etableret.

Vurdering: Alle fire GDPR-krav for dette punkt er dækket på tilfredsstillende vis.

---

### 2. Breach-eskalering: Trine → Mads + Dorthe parallelt (art. 33, 72-timer)

**LØST**

Hard rule (linje 78) specificerer:

- Eskalering sker "omgående og parallelt" til Mads (CISO) og Dorthe (DPO).
- 72-timers-fristen er eksplicit nævnt med korrekt forankring i GDPR art. 33.
- Forbuddet mod sequential afventen er eksplicit: "Trine må ikke vente på Mads' bekræftelse før Dorthe orienteres."

Vurdering: Parallel-kravet og tidsfristens løbetidspunkt er korrekt formuleret. Dette er i overensstemmelse med GDPR art. 33(1) og EDPB Guidelines 9/2022 om breach notification.

---

### 3. "Karen" fjernet

**LØST**

Grep over hele mandatteksten viser nul forekomster af "Karen". Ansvarsfordelings-tabellen bruger udelukkende generiske termer: "backend-udvikler", "frontend-udvikler", "AI-pipeline-udvikler", "security-reviewer", "HSM-operatør" — alle med "(projekt-rolle, defineret af Mads)"-markering.

---

## Nye fund

### Fund A: Intern inkonsistens i rapporteringslinje (arbejdsgang §4 vs. hard rule)

**Alvorlighedsgrad: LAV — kræver rettelse før commit**

Arbejdsgang-trin 4 (linje 66 i mandatteksten) angiver:

> "Rapportér: månedsrapport til **Mads** + kort opsummering til Stefan hvis blocker."

Dette er i konflikt med hard rule (linje 75):

> "Rapporterer til **Stefan** (primær linje) og dermed til FGD. Mads modtager kopi (cc). Ikke til udvikler-teamet direkte."

og med den etablerede SOD-begrundelse i Lailas forord:

> "Primær rapporteringslinje: Trine → Stefan → FGD. Mads modtager kopi (cc) af compliance-rapporter men er ikke primær modtager af Trines audit-konklusioner."

**Konsekvens**: Mandattet indeholder to modstridende instrukser om rapporteringskanal. Trine kan potentielt fortolke arbejdsgangen som at Mads er primær modtager og Stefan kun er cc ved blockere. Dette underminerer revisor-uafhængighed og SOD.

**Anbefalet rettelse** (Laila/Stefan eksekverer): Linje 66 skal rettes til:

> "Rapportér: månedsrapport til **Stefan** (primær) + cc til Mads; kort opsummering til Stefan ved blockere."

Dette er ikke et GDPR-problem i sig selv, men en styrings-inkonsistens der kan føre til ukorrekt behandling af compliance-rapporter, hvilket indirekte påvirker GDPR-overvågningsforpligtelsen (art. 39(1)(b)).

---

## Konklusion

| Punkt | Status |
|---|---|
| GDPR-grænseflade evidence-logs (art. 6, retention, dataminimering) | LØST |
| Breach-eskalering parallelt Mads + Dorthe (art. 33, 72-timer) | LØST |
| "Karen" fjernet | LØST |
| Nyt fund: rapporteringslinje-inkonsistens (§ arbejdsgang trin 4) | KRÆVER RETTELSE |

**CONDITIONAL GO**: Mandattet er klart til commit efter én tekstrettelse i arbejdsgang-trin 4 (rapporteringslinje). Rettelsen er teknisk triviel og kræver ikke ny GDPR-gennemgang — Dorthe bekræfter endeligt GO ved Camillas commit-besked med rettelsen dokumenteret.

---

*Dorthe — Data Protection Officer*
*Review udført: 2026-05-08*
