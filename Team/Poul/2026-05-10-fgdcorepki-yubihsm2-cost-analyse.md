---
dato: 2026-05-10
type: cost-analyse
projekt: FGDCorePKI
forfatter: Poul (analytiker)
emne: YubiHSM2 hardware-investering
status: udkast
---

# YubiHSM2 cost-analyse — FGDCorePKI Fase 0

## Sammendrag

Mads' anbefaling på 7 enheder er **security-minimum** men dækker ikke ceremony-spare ved hardware-svigt under aktiv key ceremony. Jeg anbefaler **Scenario B (9 enheder)** som risk/cost-balanceret minimum: marginal-cost €1.300 ex-VAT for at undgå at en ceremony skal aflyses og rescheduleres ved en defekt enhed (operationel disruption ≫ hardware-cost). Scenario C (11 enheder) er forsvarligt hvis FGD ønsker fuld geografisk redundans. Total beslutning til FGD: €5.850 ex-VAT (B) eller €7.150 ex-VAT (C) for Fase 0; FIPS-skifte i Fase 2+ tillægger ~€2.700-3.300 ex-VAT (re-buy af 9-11 FIPS-enheder).

---

## 1. YubiHSM2 priser (verificeret 2026-05-10)

Kilder: Yubico EU-store og Yubico US-store, hentet 2026-05-10.

| Variant | Pris ex-VAT | Pris incl. VAT (DK 25%) | FIPS-status |
|---|---|---|---|
| YubiHSM 2 v2.4 (non-FIPS) | **€650** | €812,50 | — |
| YubiHSM 2 FIPS v2.2 | **€950** | €1.187,50 | FIPS 140-2 Level 3 (validation udløbet 2026-05-02; FIPS 140-3 forventes Q2 2026 — afventer NIST CMVP-grant) |

**VAT-håndtering**: FGD Consulting Sweden er momsregistreret. Cash-cost er ex-VAT (moms refunderes via momsregnskab). Alle scenarie-tal nedenfor er ex-VAT.

**Volume-rabat**: Yubico EU-store viser ingen volume-pricing under 50 enheder. For >10 enheder anbefales Enterprise-quote direkte fra Yubico Salg — historisk 5-10% rabat ved >10 enheder, men ikke kontraktuel.

---

## 2. Scenarieanalyse

### Forudsætninger

- AP-arkitektur §1.1: Tier 1 (1 aktiv, offline) + Tier 2 (HA-par 2 aktive, online) + Tier 3 (1 aktiv, online) = 4 produktions-aktive HSMs
- "Backup" = wrapped-export-restore-targets (kolde HSMer til svigt-import), ikke hot-spares til ceremony-disruption
- Wrap-key-rotation hvert 5. år (AP §2.3)

### Tæller-diskrepans: 7 vs. 8

Mads' "7 enheder" matcher kun hvis Tier 2's 2 backup-HSMer tolkes asymmetrisk. Summen 2+4+2 (prompt-tal) = 8. **Stefan bør reconcile med Mads inden FGD-beslutning** — er Mads' 7 inkl. eller ekskl. fuld Tier 2 2+2-redundans?

### Scenarie-tabel

| Scenario | Tier 1 | Tier 2 | Tier 3 | Total | Cost ex-VAT |
|---|---|---|---|---|---|
| A (Mads minimum, 7) | 1+1=2 | 2+1=3 | 1+1=2 | **7** | €4.550 |
| A' (strikt 2+2 Tier 2) | 1+1=2 | 2+2=4 | 1+1=2 | **8** | €5.200 |
| **B (anbefalet — +ceremony-spare)** | 1+1+1=3 | 2+2=4 | 1+1=2 | **9** | **€5.850** |
| C (geografisk redundans) | 1+1+1=3 | 2+2+1=5 | 1+1+1=3 | **11** | €7.150 |

### Risiko-analyse pr. scenario

| Risiko | A (7) | B (9) | C (11) |
|---|---|---|---|
| HSM-defekt under Tier 1 ceremony → 4-8 ugers reschedule | **Høj** | Lav | Meget lav |
| HSM-defekt under Tier 2 produktion → degradation | Mellem | Lav | Meget lav |
| Geografisk disaster tager Tier 2 + backup | Høj | Mellem | **Lav** |
| Wrap-key-rotation kræver spare til import-test | Mellem | Lav | Meget lav |

---

## 3. Total Cost of Ownership (5-årig horisont)

**Hardware (år 0):** Se scenarietabel ovenfor.

### Vedligehold (estimat, årligt)

| Post | Estimat |
|---|---|
| HSM-firmware-opdateringer (8-9 enheder × 2 timer × 1.000 DKK/time) | 16-18.000 DKK/år |
| HSM-operatør drift (4 timer/uge × 1.000 DKK/time) | 200.000 DKK/år |
| Kvartalvis HA failover-test Tier 2 | 16.000 DKK/år |
| Kvartalvis audit-review (Trine + KCO) | 32.000 DKK/år |
| Wrap-key-rotation (hvert 5. år, engangs-event) | ~80.000 DKK |
| **Operatør+vedligehold, årligt** | **~250-260.000 DKK/år** |

**5-årig total TCO (Scenario B):**
- Hardware år 0: €5.850 ≈ 43.640 DKK
- Vedligehold år 1-5: 5 × 250.000 = 1.250.000 DKK
- Wrap-key-rotation år 5: 80.000 DKK
- **Total ≈ 1.374.000 DKK ≈ €184.000**

**Konklusion:** Hardware er ~3% af 5-års TCO. Operatør-tid dominerer. Scenario-valg handler om risk-mitigation, ikke hardware-budget.

---

## 4. FIPS-variant — overgang Fase 2+

- YubiHSM 2 FIPS v2.2: FIPS 140-2 Level 3 validation udløbet 2026-05-02
- FIPS 140-3 forventes Q2 2026 (Yubico prognose, ikke garanti)
- Re-keying-trigger: CMVP-grant + Mads' compliance-bedømmelse + FGD-go

### FIPS overgangs-delta (Scenario B, 9 enheder)

| Post | Cost ex-VAT |
|---|---|
| Re-buy 9 FIPS-enheder (€950 × 9) | €8.550 |
| Pris-delta vs. non-FIPS (€300 × 9) | €2.700 |
| Re-keying ceremony (~80 personhours + faciliteter) | ~€11.000 |
| Migration parallel-drift 30-90 dage | ~€5.400 |
| **Total Fase 2+ FIPS delta** | **~€19.000** |

Resterende non-FIPS Fase 0-enheder kan re-purposes til lab/test/rehearsal.

---

## 5. Anbefaling

**Scenario B (9 enheder, €5.850 ex-VAT) — bedst risk/cost-balance for intern PKI Fase 0.**

Begrundelser:
1. Marginal-cost €1.300 ex-VAT (A→B) er trivielt vs. ceremony-reschedule-cost på 80-120.000 DKK + 4-8 ugers projektforsinkelse
2. Tier 2 og Tier 3 har hot-spare i B — RTO under 1 time vs. 1-4 timer ved wrapped-key-import i A
3. Wrap-key-rotation år 5 lettere med spare-pool

**Scenario A** kun hvis budget er hard-limited i Q2 2026 og FGD accepterer ceremony-reschedule-risk.

**Scenario C** hvis geografisk redundans ønskes i Fase 0 (ellers Fase 1+).

---

## 6. Beslutningsgrundlag til FGD

**FGD godkender:**

| Post | Scenario B (anbefalet) |
|---|---|
| Antal enheder | 9 |
| Hardware-budget ex-VAT | **€5.850** |
| Cash-flow incl-VAT | **€7.312,50** (≈ 54.550 DKK) |
| Ordrekanal | Yubico EU-store direkte (én ordre, ensartet firmware) eller Enterprise-quote |

**FGD godkender IKKE nu:** Operatør-budget (250.000 DKK/år — eksisterende ressource), FIPS-overgang (afventer CMVP-grant), Tier 3 HA (Fase 1+).

**Anbefalet timing:** Ordre placeres når Engineering Team Lead bekræfter modtage-klar og KCO-rolle er bemandet. Forhandl Enterprise-quote (~€300-600 ex-VAT rabat mulig).

---

## 7. Risici og forbehold

- **Tæller-diskrepans 7 vs. 8**: Reconciler med Mads inden FGD-beslutning
- **Pris-volatilitet**: Yubico justerer priser ~1×/år; valider ved ordre-tidspunkt
- **CMVP-timeline-usikkerhed**: FIPS 140-3 Q2 2026 er prognose, kan forsinkes til H2 2026+
- **Volume-rabat ikke verificeret**: Kontakt Yubico Salg for konkret Enterprise-quote
- **Ceremony-spare-antagelse**: Scenario B forudsætter ceremony-spare tilgængeligt indenfor 1-2 timer fra ceremony-lokation; hvis off-site, vælg scenario C

---

## 8. Kilder

- Yubico EU-store: YubiHSM 2 v2.4 €812,50 incl-VAT; YubiHSM 2 FIPS v2.2 €1.187,50 incl-VAT (hentet 2026-05-10)
- AP-arkitektur Fase 0: `Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md` §1.1-1.4, §2.3, §2.6
- Memory: `projects/knowlagecentral/yubihsm-vej-2026-05-05.md`, `hsm-fips-140-produktionsmål-2026-05-05.md`, `kunde-vertikal-regulering-2026-05-05.md`
