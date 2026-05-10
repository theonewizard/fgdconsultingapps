---
dato: 2026-05-09
forfatter: Laila (HR-leder)
emne: Kvittering — Mads FC2+FC4-rettelser implementeret (v2.1)
status: UDKAST v2.1 færdig, klar til Mads stikprøvekontrol
relateret-final-check: Team/Mads/2026-05-09-mads-final-check-fgdcorepki-mandater.md
---

# Laila — v2.1 Rettelser: FC2 + FC4 + Redaktionelle fejl

**Status:** Implementeret og afsluttet 2026-05-09.

Begge FGDCorePKI-mandat-udkast er opdateret med Mads' FC2+FC4-krav + redaktionelle fejlrettelser fra final-check.

---

## Implementerede rettelser

### File 1: `Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md` (v2 → v2.1)

**FC2 — Break-glass for PII-bærende systemer:**
- ✓ Tilføjet nyt afsnit "Break-glass for PII-bærende systemer" under **Hard rules** (efter Least Privilege-afsnit).
- ✓ Specificerer at break-glass dækker CSR-database + audit-logs der indeholder PII (Subject CN/SAN, email, organisation).
- ✓ Krav: skriftlig begrundelse + Mads-godkendelse + automatisk audit-trail + max 4-timers session + WORM-format.
- ✓ Dorthe-notifikation inden 24 timer af break-glass-adgang til PII-systemer.
- ✓ Koordination med Dorthe for GDPR art. 32-vurdering (uautoriseret behandling-risiko).

**Redaktionelle fejl:**
- ✓ Linje 97: "`audit.events`" → "PKI audit-log" (GDPR-relateret sektion)
- ✓ Linje 144 ansvarsfordeling: "du designes arkitektur/integration" → "du designer arkitektur/integration"
- ✓ Linje 149 ansvarsfordeling: "Du designers HSM-konfiguration" → "Du designer HSM-konfiguration"
- ✓ Linje 163 hand-off-aftaler: "Du designes procedure" → "Du designer procedure"

**Changelog opdateret:**
- ✓ Ændringer fra v1 til v2.1 sektion tilføjet med eksplicit "Mads FC2+FC4-rettelser (v2.1)" med detaljer.

---

### File 2: `Team/Laila/2026-05-09-udkast-pki-governance-roller.md` (v2 → v2.1)

**FC4 — KCO rolleafgrænsning:**
- ✓ Tilføjet eksplicit rolleafgrænsnings-tekst til **KCO "Ceremony-script-development"** opgave (linje 159-162).
- ✓ Tekst specificerer:
  - Application Provider leverer teknisk script-udkast
  - Key Ceremony Officer godkender proceduren for ceremoniuel sikkerhed
  - Mads godkender det færdige script før ceremony eksekveres
  - KCO kan holde ceremony tilbage ved uenighed om sikkerhed; Mads megler
- ✓ Dette sikrer at afgrænsningen fremgår af KCO-profilens egen tekst, ikke blot AP-dokumentet.

**Redaktionel fejl:**
- ✓ Linje 296 SOD-begrænsninger (Engineering Team Lead): Fjernet forældet "hvis ingen AP i fase 1"-formulering.
  - Før: "Engineering Team Lead rapporterer til Application Provider (hvis AP er dedikeret) eller direkte til Mads (hvis ingen AP i fase 1)."
  - Efter: "Engineering Team Lead rapporterer til Application Provider (teknisk mentor) og til Mads (compliance-oversight)."
- ✓ Rapport-linje-tabel opdateret tilsvarende (linje 297).

**Changelog opdateret:**
- ✓ Ændringer fra v1 til v2.1 sektion tilføjet med eksplicit "Mads FC4-rettelse (v2.1)" + "Redaktionel fejl udbedret (v2.1)".

---

## Status per rettelse (Mads' final-check)

| Rettelse | Status | Lokation |
|---|---|---|
| **FC2** — Break-glass PII-dækning | ✓ Implementeret | AP-udkast, Hard rules, nyt afsnit |
| **FC4** — KCO rolleafgrænsning | ✓ Implementeret | PKI-governance §3 (KCO), Ceremony-script-development |
| **Redaktionel: `audit.events`** | ✓ Rettet | AP linje 97 → "PKI audit-log" |
| **Redaktionel: Verbformer** | ✓ Rettet | AP linje 144, 149, 163 (designes → designer) |
| **Redaktionel: Forældet formulering** | ✓ Rettet | PKI-governance linje 296 ("hvis ingen AP i fase 1" fjernet) |

---

## Næste trin

1. **Mads stikprøvekontrol** af FC2+FC4-afsnittene + redaktionelle rettelser (response-tid: max 24 timer per final-check-instruks).
2. **Stefan → FGD-GO** når Mads godkender stikprøven.
3. **Camilla committer** `.claude/agents/<navn>-application-provider-fgdcorepki.md` + `.claude/agents/<navn>-ca-administrator-fgdcorepki.md` osv. (fulde mandater efter FGD-godkendelse).

---

**Lagt af:** Laila (HR-leder)  
**Dato:** 2026-05-09  
**Status:** v2.1 implementeret og testet; afventer Mads stikprøvekontrol
