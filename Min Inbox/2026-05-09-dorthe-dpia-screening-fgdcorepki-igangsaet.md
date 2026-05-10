---
dato: 2026-05-09
fra: Dorthe (DPO)
til: FGD
cc: Stefan (orkestrator), Mads (CISO)
emne: To DPIA-screeninger initieret for FGDCorePKI — kræver FGD-stillingtagen
projekt: FGDCorePKI
status: SCREENING-IGANGSAT
---

# DPIA-screeninger igangsat — FGDCorePKI

Som flagget i min final-check af FGDCorePKI-mandaterne (2026-05-09, GO) har jeg nu initieret de to DPIA-screeninger der skal gennemføres inden Fase 0 produktionstag.

## En-linje-opsummering

1. **Ceremony-recording DPIA**: kræver fuld DPIA — biometri (art. 9) + 20-årsbevaring udløser DPIA-pligt af sig selv. Skal være afsluttet inden første Root CA key generation.
2. **OCSP/CRL CDN DPIA**: conditional — kræver fuld DPIA + Schrems II-TIA hvis tredjelands-CDN anvendes; bortfalder hvis EU-only-arkitektur vælges.

## Screening-filer

- `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ceremony-recording.md` (status: kræves)
- `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ocsp-crl-cdn.md` (status: TBD — conditional)

## Hvad FGD skal tage stilling til

| # | Beslutning | Konsekvens | Hastværd |
|---|---|---|---|
| 1 | Bekræfte at de to DPIA'er skal igangsættes som planlagt (Fase 0-blokkere) | Standard governance-flow; jeg kører videre med F1-F9 (ceremony) og F1-F8 (CDN) | Inden 5 hverdage |
| 2 | **OCSP/CRL-arkitektur — EU-only som default?** Hvis FGD beslutter EU-only-hosting (intern eller EU-CDN), bortfalder DPIA #2 og erstattes af ROPA-opdatering. Hvis FGD ønsker global CDN åben som mulighed, kører jeg fuld DPIA + Schrems II-TIA parallelt | Stort scope-impact: EU-only sparer 2-8 ugers DPIA-arbejde og fjerner art. 36-risiko | Inden arkitekturvalg (peg gerne tidligt) |
| 3 | **Ceremony-recording-format** — fuld video, eller alternativ form (audio + screen + hænder uden ansigt; ansigtsmaskering)? | Reducerer biometri-eksponering og potentielt retention-krav. Ikke en låst beslutning nu, men FGD's holdning til "minimum optage-format der opfylder non-repudiation" hjælper DPIA-mitigation-fase | Inden ceremony-script låses (typisk uger før ceremoni) |
| 4 | **20-årsbevaring vs. 7-10 år** for ceremony-arkiv — er 20 år drevet af WebTrust/ETSI-audit eller intern policy? Hvis ikke regulatorisk pålagt, kan retention reduceres | Direkte input til DPIA-proportionalitets-vurdering (art. 5(1)(e)) | Inden DPIA-konklusion |

## Mit anbefalede flow

1. FGD bekræfter punkt 1 (DPIA-er igangsættes) — jeg kører videre uanset 2-4
2. Stefan koordinerer at AP + Mads forelægger arkitektur-options for OCSP/CRL hurtigst muligt (driver punkt 2)
3. Når Fase 0 nærmer sig produktionstag (estimeret 4-6 uger ud), har jeg fuld DPIA(er) klar; FGD modtager DPIA-konklusion med GO/GO-MED-MITIGATION/art. 36-anbefaling
4. ROPA `governance/dpa/ropa.md` oprettes parallelt af Camilla med mine 9 behandlinger fra final-check'en

## Ingen blokerende risici lige nu

Mandat-strukturen er GO. Disse screeninger er forventede og dokumenterede pre-requisits for Fase 0 produktionstag — ikke en forsinkelse, men planlagt arbejde der nu er sat i gang.

---

**Næste skridt fra mig:**
- Afventer FGD-bekræftelse på punkt 1 (gennem Stefan)
- Indhenter F-spørgsmål-svar fra interessenter (AP, Mads, KCO når bemandet)
- Holder Stefan løbende orienteret om DPIA-fremdrift
- Eskalerer direkte hvis Schrems II-TIA-konklusion peger mod art. 36-konsultation

— Dorthe (DPO)

---

**Opdatering 2026-05-10:** DPIA-2 (OCSP/CRL CDN) er lukket — FGD besluttede EU-only hosting (Scenario B). Kun DPIA-1 (ceremony-recording) er fortsat åben.
