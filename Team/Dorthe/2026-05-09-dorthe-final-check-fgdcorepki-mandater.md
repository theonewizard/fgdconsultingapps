---
dato: 2026-05-09
type: final-check
emne: FGDCorePKI AO+AP+PKI-roller v2 — GDPR
vurderer: Dorthe (DPO)
status: GO
basis-filer:
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md (v2)
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md (v2)
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md (v2)
forrige-vurdering: Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md (STOP — 8 blokerende + 6 conditional)
præcedens-reference:
  - .claude/agents/application-owner.md (godkendt v2)
  - .claude/agents/application-provider.md (godkendt v2)
modtagere: FGD, Mads (CISO), Stefan (orkestrator), Laila (HR)
---

# Dorthe — Final-check: FGDCorePKI mandater v2 (GDPR)

## Samlet status: GO

Alle 8 blokerende fund fra min STOP-vurdering 2026-05-09 er reelt indarbejdet i v2-udkastene. Alle 6 conditional fund er ligeledes adresseret. To mindre observationer noteret nedenfor som "ROPA-noter" og er ikke blokerende — de håndteres ved aktivering, ikke som forudsætning for FGD-godkendelse af mandat-strukturen.

GDPR-substansen er nu på samme bar som AO/AP v2-præcedens-mandaterne i `.claude/agents/`. Sammenligning af 4-bullet DPO-uafhængigheds-blok mod præcedensen viser ordret match for alle 4 substantielle krav (ikke-instruktion, direkte FGD-eskalering, ikke-filtrering, ikke-sanktionering).

**Anbefaling:** Mandater kan gå til FGD-GO. ROPA-oprettelse for de 9 nye behandlingsaktiviteter (jf. min 4-eyes-tabel) eksekveres parallelt med rolle-aktivering.

---

## Regressionscheck

| Fund | Status | Note |
|------|--------|------|
| DPO-uafhængigheds-blok AO (4 bullets) | GO | AO linjer 91-96. Ordret match med præcedens i `.claude/agents/application-owner.md` linjer 59-64. Alle 4 substantielle krav til stede |
| DPO-uafhængigheds-blok AP (4 bullets) | GO | AP linjer 101-106. Ordret match med præcedens i `.claude/agents/application-provider.md` linjer 72-77 |
| §3.3-supersedance-note AO | GO | AO linje 100: "§3.3 er superseded af FGD-override 2026-05-06 — Mads + Dorthe vurderer SAMMEN alle rolle-konfigurations-ændringer (ingen 'Mads-engangs-undtagelser')." Korrekt placeret under Mandat-stabilitet |
| §3.3-supersedance-note AP | GO | AP linje 121: identisk formulering. Korrekt placeret |

Alle 4 regressioner fra forrige runde er retableret. Ingen v2-bar-glidning.

---

## GDPR-specifikke fund (PKI-substans)

### AP-3 — Tredjeland-trigger udvidet (var BLOKERENDE)

**Status: GO**

AP linje 96 lister nu eksplicit: *"ny tredjelands-CDN-hosting af OCSP/CRL/CT-data, ny managed-CA- eller cert-management-SaaS uden EU/EØS-residency"*. Dækker:
- OCSP/CRL CDN-hosting (Cloudflare, Fastly, Akamai)
- CT-logs i tredjeland (hvis FGDCorePKI går public-trust)
- Cert-management-SaaS uden EU-residency (Keyfactor, Venafi)

Art. 44-49-vurdering aktiveres korrekt før design låses. Adresserer Pouls Vinkel 3+6.

---

### AP-4 — Retention-differentiering med art. 5(1)(e) (var BLOKERENDE)

**Status: GO**

AP linje 97 kobler nu retention eksplicit til formålsbestemmelse: *"7-årsbevaring er PKI-baseline for issuance/revocation audit; kortere retention kan være nødvendigt for OCSP-request-logs (30-90 dage) og længere for ceremony-protokoller. Dorthe godkender retention-matrix per dataflow."*

Differentiering matcher mit krav:
- OCSP-request-logs: 30-90 dage (proportionalt; requesting-IP er PII med kort formål)
- Issuance/revocation audit: 7 år (PKI-baseline + Mads-krav)
- Ceremony-protokoller: længere (særskilt vurdering ved Dorthe)
- Eksplicit reference til DPO-godkendelse af retention-matrix før implementering

Note: 20-årsbevaring af ceremony-video håndteres separat under GR-2 (KCO-profil), korrekt placering.

---

### AP-5 / AO-3 — Breach-eskalering ved CA-kompromittering (var CONDITIONAL)

**Status: GO**

AP linje 99 har eksplicit: *"CA-nøgle-kompromittering (Issuing/Intermediate/Root) eskaleres direkte til Dorthe + Mads øjeblikkeligt — Dorthe initierer GDPR art. 33/34-vurdering med 72-timers-ur. Du behøver ikke afvente Application Owner-bekræftelse før eskalering."*

Korrekte elementer:
- Tre kompromis-niveauer (Issuing/Intermediate/Root) eksplicit
- 72-timers-ur til art. 33-Datatilsyn-notifikation
- Art. 34-vurdering (data-subject-notifikation hvis høj risiko)
- AO-filter eksplicit elimineret ved breach (DPO-uafhængighed bevares operationelt)

AO-3-elementet (24-timers breach-eskalering) er på AO-side adresseret via linje 110 i operationel rapportering: *"Mistanke om databrud eller audit-log-tampering → direkte eskalering til Mads + Dorthe inden 24 timer"*. Tilstrækkelig dækning.

---

### GR-1 — RA-rolle GDPR-sektion (var BLOKERENDE)

**Status: GO**

PKI-roller §4 (RA), linjer 217-224 har nu fuld 6-bullet GDPR-sektion:

| Krav fra mit fund GR-1 | Verifikation i v2 |
|---|---|
| Lawful basis per ansøger-kategori (art. 6) | Linje 219: art. 6(1)(b)/(f)/(c) differentieret per kunde/intern/regulatorisk |
| Retention-matrix differentieret | Linje 220: CSR 3 år post-cert-udløb; RA-audit 7 år; identitets-verifikations-data slettes ved decision |
| Data-subject-rettigheder (art. 15-22) | Linje 221: art. 15 håndteres af RA; art. 17 eskaleres til Dorthe |
| DPO-notifikation ved RA-portal-ændringer | Linje 222: 5 hverdage med ROPA-format-bullets |
| Art. 9-undtagelse i afvisnings-begrundelser | Linje 223: særlige kategorier forbudt; policy-koder bruges |
| Eskalations-pligt ved tvivl | Linje 224: tvivl → Dorthe inden implementering |

Den eneste rolle med direkte ansøger-PII-behandling har nu GDPR-sektion på samme niveau som AO/AP. Største enkelte gap fra forrige runde er lukket.

---

### GR-2 — DPIA-screening for ceremony-video (var BLOKERENDE)

**Status: GO**

PKI-roller §3 (KCO), linje 163 har nu inline GDPR-tekst i video/audio-recording-bulletten:

> "Optagelse udgør behandling af biometriske data (witnesses + officers) og kræver DPIA (art. 35) før første ceremony. 20-årsbevaring er ikke automatisk proportional — formålsbestemt vurdering ved Dorthe inden ceremony-script låses. Lawful basis dokumenteres per deltager (samtykke art. 6(1)(a) eller anden basis); deltagere informeres om optagelse, retention, og deres rettigheder per art. 13/14."

Korrekte elementer:
- DPIA art. 35 eksplicit krævet før første ceremony
- 20-årsbevarings-proportionalitet skal vurderes (ikke automatisk accepteret)
- Lawful basis per deltager (art. 6(1)(a) samtykke eller anden basis)
- Information til deltagere per art. 13/14
- Biometri-kategorisering anerkendt (art. 9-trigger)

Mine alternativer fra fund GR-2 (audio + papir, video uden lyd, ansigtsmaskering) er ikke eksplicit i mandatet, men det er acceptabelt — disse er DPIA-mitigation-options som Dorthe og KCO afgør i konkret DPIA-proces. Mandat-niveauet kræver kun at DPIA sker før ceremony låses, hvilket er på plads.

---

### GR-3 — HSM-operatør PII-/EU-residency (var CONDITIONAL)

**Status: GO**

PKI-roller §2 (HSM-operatør), linje 122: *"HSM audit-log indeholder authentication-key-ID + command-historik der kan udgøre PII (1:1-mapping til operatør). Eksport-flow til WORM-storage skal være EU-residency (Dorthe vurderer leverandør); access til logs er least-privilege (kun Trine + Mads + Dorthe)."*

Adresserer Pouls Vinkel 8 om hourly audit-log-eksport.

---

### GR-4 — CA Administrator PII-note (var CONDITIONAL)

**Status: GO**

PKI-roller §1 (CA Admin), linje 75: *"Audit-trail for certifikat-issuance indeholder subject-PII (CN, SAN, email, org). Adgang til issuance-log er least-privilege; logs slettes per retention-matrix (Dorthe godkender)."*

---

### GR-5 — Trine GDPR-koordination (var CONDITIONAL)

**Status: GO**

PKI-roller §7 (Trine udvidelse), linje 354 GDPR-koordination i profil-tabel: *"Findings med GDPR-impact (uautoriseret PII-access, retention-overskridelse, manglende lawful basis) cc:'es til Dorthe i samme rapport."*

Også reflekteret i rapport-struktur linje 373.

---

### AO-4 — Tredjelands-eksempel udvidet (var CONDITIONAL)

**Status: GO**

AO linje 81 indeholder nu: *"...ny tredjelandsservice (fx Langchain for prompt-chaining, eller tredjeland-CDN til OCSP/CRL distribution)..."*. Tredjeland-CDN er nu eksplicit triggernevnt udover Langchain.

Note: AO-3 (24-timers breach-eskalering) er adresseret som beskrevet ovenfor under AP-5/AO-3-bulletten.

---

### AP-6 — ROPA-notifikations-format (4 minimum-felter)

**Status: BEVARET (var ikke regredieret)**

AP linje 96 har stadig de 4 minimum-felter: (a) hvad ændrer sig, (b) hvilken processing-aktivitet i ROPA berøres, (c) forventet implementeringsdato, (d) hvem der drev ændringen. Korrekt bevaret fra v2-præcedens.

---

## Resterende krav (hvis GO ikke kan gives)

Ingen blokerende eller conditional fund tilbage. GO kan udstedes uden restriktioner på mandat-strukturen.

---

## Mindre observationer (ikke blokerende, til opfølgning ved aktivering)

**Obs 1 — RA model-anbefaling vs. art. 22:** RA-profilen anbefaler `haiku` med begrundelsen "regelbaseret godkendelse; CSR-struktur-validering er routine". Hvis RA-rollen i praksis bliver fuldautomatiseret AI-godkendelse uden human-in-the-loop, kan det udløse art. 22 (automatiseret individuel beslutningstagning med retsvirkning eller tilsvarende væsentlig påvirkning af cert-ansøger). Mandat-niveauet kræver ikke ændring — men ved aktivering: Dorthe vurderer art. 22-trigger sammen med RA-eskalations-flow (linjer 215, 224). Eskalationspligten ved policy-tvivl giver i praksis human-fallback, hvilket sandsynligvis tilfredsstiller art. 22(2)(b)/(c). Ikke blokerende; håndteres som DPIA-input ved RA-aktivering.

**Obs 2 — Break-glass session-logs som PII:** AP linje 97-98 og §2 §"break-glass" omtaler WORM/kryptografisk chaining. Break-glass-session-logs indeholder operator-ID + tidsstempel + command-historik = PII per art. 4(1). Dette er dækket implicit af GR-3 (samme princip), men en eksplicit AP-bullet ville være ryddeligere. Ikke blokerende; kan opfølges som ROPA-detalje ved første break-glass-procedure-implementering.

---

## ROPA-note

`governance/dpa/ropa.md` eksisterer ikke endnu (verificeret 2026-05-09 i forrige runde). Ved aktivering af FGDCorePKI-rollerne skal følgende 9 behandlingsaktiviteter oprettes i ROPA (uændret fra min 4-eyes-tabel):

| # | Behandling | Lawful basis | Retention | DPIA |
|---|---|---|---|---|
| 1 | Certifikat-udstedelse | art. 6(1)(b)/(f)/(c) | Cert-validity + 3 år | Nej |
| 2 | CSR-modtagelse + RA-identitets-verifikation | art. 6(1)(b)/(f) | Decision + 3 år | Conditional |
| 3 | Cert-issuance-audit-log | art. 6(1)(c) | 7 år | Nej |
| 4 | HSM-audit-log | art. 6(1)(f) | 7 år | Nej |
| 5 | Key-ceremony-optagelse | art. 6(1)(a)/(c) + art. 9 | 20 år | **JA** (mandat-krav) |
| 6 | OCSP-responder-requests | art. 6(1)(f) | 30-90 dage | Conditional |
| 7 | CRL-distribution | art. 6(1)(f) | 30-90 dage | **JA hvis tredjelands-CDN** |
| 8 | Break-glass-session-logs | art. 6(1)(c) | 7 år | Nej |
| 9 | Revocation-data + art. 34-notifikation ved CA-kompromis | art. 6(1)(c) | 7 år | Nej (men art. 34-flow påkrævet) |

**Aktiverings-trigger:** Når FGD godkender mandater og første rolle ansættes, beder jeg Stefan koordinere ROPA-oprettelse via Camilla. Jeg leverer indhold til de 9 behandlinger; Camilla strukturerer og committer.

**To DPIA'er skal initieres i Fase 0 (før første ceremony, før første tredjelands-CDN-deployment):**
1. **Ceremony-recording DPIA** — KCO + Dorthe; trigger ved første Root CA key generation
2. **OCSP/CRL CDN DPIA** — AP + Dorthe; trigger hvis FGDCorePKI-arkitektur vælger tredjelands-CDN

Yderligere conditional-DPIA (RA-flow #2 + #6 OCSP) afgøres ved arkitektur-låsning.

---

## Konsistens med Mads' parallelle vurdering

Min final-check dækker udelukkende GDPR-/data-subject-perspektivet. Mads laver parallel security/SOD-vurdering. Mit GO her er ikke betinget af Mads' vurdering, men FGD bør modtage begge vurderinger samtidigt før godkendelses-beslutning. Hvis Mads finder noget der kræver mandat-ændring som også berører GDPR (fx ny rolle-grænse, ændret eskalations-flow), revurderer jeg ad hoc.

---

## Anbefaling til FGD

**GO** på alle tre udkast (AO + AP + PKI-roller) som mandat-struktur. Alle 8 blokerende + 6 conditional GDPR-fund fra min STOP-vurdering er adresseret på samme niveau som AO/AP v2-præcedens-mandaterne.

**Næste skridt:**

1. **FGD godkender mandat-struktur** baseret på Mads + Dorthe samlet vurdering (forventer Mads' final-check er parallelt eller umiddelbart efter denne)
2. **Camilla aktiverer mandater** i `.claude/agents/` med projekt-rolle-naming-konvention (jf. de tre udkasts frontmatter-forslag)
3. **Camilla opretter `governance/dpa/ropa.md`** efter mandat-aktivering; Dorthe leverer indhold for de 9 behandlinger
4. **DPIA-trigger-watch:** Jeg holder øje med (a) første ceremony-planlægning → ceremony-recording DPIA, (b) OCSP/CRL-arkitektur-valg → tredjelands-CDN DPIA
5. **Art. 22-vurdering ved RA-aktivering:** Hvis RA-rollen skal være fuldautomatiseret, foretages art. 22-trigger-vurdering ved RA-bemanding (Obs 1)

**Anslået rettelse-arbejde fra dette punkt:** Ingen for mandat-strukturen. Aktiveringsarbejde (ROPA + 2 DPIA'er) er forventet og planlagt.

---

**Forfatter:** Dorthe (DPO)
**Dato:** 2026-05-09
**Status:** GO — alle 14 fund adresseret
**Foregående:** Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md (STOP)
**Næste:** Afventer FGD-godkendelse koordineret med Mads' parallelle vurdering
