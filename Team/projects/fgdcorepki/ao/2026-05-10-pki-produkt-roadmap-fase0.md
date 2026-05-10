---
dato: 2026-05-10
type: pki-produkt-roadmap
projekt: FGDCorePKI
fase: 0
forfatter: Application Owner (FGDCorePKI)
status: udkast
---

# FGDCorePKI — PKI Produkt Roadmap Fase 0

**Versionering:** Udkast til FGD-review og Mads-compliance-scorecard.
**Eskalering:** Compliance-konklusioner er ikke givet her. Mads og Dorthe leverer uafhængige vurderinger; dette dokument er AO-input til den proces.

---

## 1. Fase 0 Scope

### Formål

Fase 0 er forberedels- og etableringsfasen for FGDCorePKI. Fasen slutter ikke med produktion af certifikater — den slutter med at alt personale, al dokumentation og al compliance-screening er på plads til at Root CA key ceremony kan afvikles forsvarligt.

### MVP-mål for Fase 0

- Alle kritiske PKI-roller er bemandet (AP, Engineering Team Lead, CA Administrator, HSM-operatør, Key Ceremony Officer)
- DPIA-1 (ceremony-recording) er afsluttet og resultat foreligger
- ROPA for FGDCorePKI er oprettet og afleveret til Dorthe
- CP/CPS-udkast er leveret (Mads + CA Administrator ejer dette)
- HSM-procurement er afklaret og hardware bestilt
- Ceremony-procedure er godkendt af Mads inden KCO-eksekvering
- SOD-design for alle roller er valideret af Mads

### In-scope for Fase 0

| Element | Ansvarlig | Leverance |
|---|---|---|
| AP-onboarding | AO koordinerer; AP bemandes af Laila/FGD | AP i rolle |
| Engineering Team Lead-onboarding | AO koordinerer; Laila/FGD bemander | ETL i rolle |
| CA Administrator-onboarding | AO koordinerer; Mads godkender profil | CA Admin i rolle |
| HSM-operatør-onboarding | AO koordinerer; Mads godkender profil | HSM-op i rolle |
| Key Ceremony Officer-onboarding | AO koordinerer; Mads godkender profil | KCO i rolle |
| DPIA-1 initiering og gennemførelse | AO initierer; Dorthe ejer | DPIA-1-rapport |
| ROPA-oprettelse | AO trigge; Camilla opretter | ROPA-sektion |
| CP/CPS-udkast | Mads + CA Administrator ejer | Udkast klar til review |
| HSM-procurement-plan | AP leverer teknisk input; AO eskalerer budget til FGD | Procurement-beslutning |
| Ceremony-procedure-godkendelse | Mads godkender; KCO eksekverer | Skriftlig Mads-godkendelse |
| SOD-validering | Mads vurderer | Scorecard-punkt |

### Out-of-scope for Fase 0

- Root CA key generation (ceremony selv — kræver Fase 0 komplet)
- Intermediate CA og Issuing CA opsætning
- Produktionsdeployment af OCSP/CRL-infrastruktur
- Certifikat-udstedelse (issuance)
- ACME/SCEP/EST-protokol-integrationer
- FIPS 140-3-transition (planlagt til en fremtidig fase når NIST CMVP certificerer YubiHSM2 FIPS-variant)
- RA-bemanding (relevant fra issuance-fase)
- Ekstern PKI-audit (Trine)

### Success-kriterier

Fase 0 er afsluttet når samtlige følgende betingelser er opfyldt — alle kræver formel bekræftelse:

1. AP i rolle (skriftlig bekræftelse fra Stefan/FGD)
2. Engineering Team Lead i rolle (skriftlig bekræftelse fra Stefan/FGD)
3. CA Administrator i rolle og godkendt af Mads (skriftlig)
4. HSM-operatør i rolle og godkendt af Mads (skriftlig)
5. Key Ceremony Officer i rolle og godkendt af Mads (skriftlig)
6. DPIA-1 rapport foreligger og er signeret af Dorthe (og FGD ved DPIA-krav)
7. ROPA-sektion oprettet og attesteret af Dorthe
8. CP/CPS-udkast leveret og indledende Mads-review gennemført
9. HSM-hardware bestilt eller aftalt leveringsplan
10. Ceremony-procedure skriftligt godkendt af Mads
11. SOD-design valideret i Mads' compliance-scorecard
12. Compliance-scorecard samlet og præsenteret til FGD af Stefan

---

## 2. Team-komposition og Bemandingsrækkefølge

Rækkefølgen er dikteret af den kritiske sti: AP-onboarding blokerer DPIA-1's tekniske F-spørgsmål; DPIA-1 blokerer key ceremony. Engineering Team Lead onboardes parallelt med AP fra dag 1 (Option B — separat fra AP, som besluttet af FGD 2026-05-09).

### Kritisk sti

```
AP-onboarding
    │
    ▼
DPIA-1 F-spørgsmål (tekniske dataflow-detaljer fra AP)
    │
    ▼
DPIA-1 afsluttet (Dorthe)
    │
    ▼
Root CA Key Ceremony (kun efter Fase 0 komplet)
```

### Bemandingsplan

| Prioritet | Rolle | Bemandingstidspunkt | Blokerer |
|---|---|---|---|
| **1 (straks)** | Application Provider (AP) | Dag 0 — blokerende | DPIA-1 F-spørgsmål; HSM-procurement-input; ceremony-procedure-input |
| **1 (straks, parallelt)** | Engineering Team Lead | Dag 0 — parallelt med AP | Tooling-design kan starte; Option B kræver separat fra AP fra dag 1 |
| **2 (tidligt)** | CA Administrator | Uge 2-4 — inden CP/CPS-udkast behøver operationel input | CP/CPS-operationelle detaljer; ceremony-forberedelse |
| **2 (tidligt)** | HSM-operatør | Uge 2-4 — inden HSM-procurement afsluttes | Hardware-spec-input; ceremony-forberedelse |
| **3 (mid-fase)** | Key Ceremony Officer | Uge 4-8 — skal have tid til ceremony-procedure-gennemgang inden ceremony | Ceremony-afvikling (kræver DPIA-1 afsluttet) |
| **Fase 1+** | RA (Registreringsmyndighed) | Ud af Fase 0 scope | Relevant fra issuance-fase |

**Note:** Alle rolleprofiler godkendes af Mads inden onboarding. AO koordinerer bemandingsprocess med Laila og Stefan — AO ansætter ikke selv.

### SOD-krav i team-komposition

Følgende SOD-krav er absolutte og skal reflekteres i bemandingen (valideres af Mads i scorecard):

- AO og AP: aldrig samme person
- AP og Engineering Team Lead: aldrig samme person (Option B-beslutning)
- HSM-operatør og CA Administrator: aldrig samme person
- KCO må ikke varetages af CA Administrator eller HSM-operatør i samme ceremony
- AO deltager ikke i key ceremony i nogen kapacitet

---

## 3. Compliance-milestones

### Oversigt

| Milestone | Ansvarlig (ejer) | AO-rolle | Deadline |
|---|---|---|---|
| DPIA-1 initiering | AO trigger; Dorthe ejer | Send screening-formular straks ved AP-onboarding | Straks ved AP-onboarding |
| DPIA-1 rapport | Dorthe | Modtage og formidle til Stefan | Inden Root CA key ceremony — hårdt krav |
| ROPA-oprettelse | Camilla opretter; Dorthe attesterer | Trigge Camilla; sende info om dataflows | Inden DPIA-1 kan afsluttes |
| CP/CPS-udkast | Mads + CA Administrator ejer | Sætte deadline; sende til Mads 2 uger inden behov | Uge 4-6 (afhænger af CA Admin-onboarding) |
| Ceremony-procedure-godkendelse | Mads godkender | Koordinere tidsplan med KCO og Mads | Minimum 4 uger inden planlagt ceremony |
| SOD-design-validering | Mads | Sende team-komposition til Mads' scorecard | Inden CA Admin / HSM-op / KCO onboardes |
| Kvartalsvis AO-attest | AO | Bekræfte at CP/CPS reflekterer faktisk arkitektur | Første attest: 3 måneder efter Fase 0 afslutning |

### DPIA-1 detaljer

**Scope:** Ceremony-recording — optagelse eller logning af Root CA key generation.

**Status:** Fuld DPIA kræves (besluttet inden Fase 0). Dorthe ejer vurderingen.

**Blokerende dependency:** DPIA-1 kan ikke afsluttes uden AP's tekniske input om:
- Hvilke systemer optager eller logger under ceremony
- Dataflow for ceremony-recordings (hvem har adgang, hvor opbevares, retention)
- Hvem er present og i hvilken rolle (dataminimering)

**AO-handling:** Straks ved AP-onboarding sender AO DPIA-screening til Dorthe og notificerer AP om at teknisk dataflow-input er nødvendigt. AO sætter ikke DPIA-konklusionen — Dorthe ejer den.

**Hård grænse:** Root CA key ceremony må ikke afvikles før DPIA-1-rapport foreligger og er godkendt (ved DPIA-krav: FGD-signering; ved art. 36-krav: Datatilsynets svar). AO kan ikke give dispensation fra dette krav.

### GDPR art. 36 — Forudgående høring

Hvis Dorthes DPIA-1-rapport konkluderer "høj risiko uden tilfredsstillende mitigation", kræver ceremony forudgående høring til Datatilsynet. Høringen koordineres af Dorthe; FGD indsender. AO deployer/afvikler ikke ceremony før Datatilsynets svar foreligger eller frist er udløbet. AO kan ikke give dispensation.

---

## 4. Roadmap-afhængigheder

### Dependency-kort

```
FGD-strategiske beslutninger (2026-05-09)
    │
    ├─► AP-onboarding (BLOKERENDE — Fase 0 starter ikke for alvor uden AP)
    │       │
    │       ├─► DPIA-1 F-spørgsmål kan besvares (teknisk dataflow fra AP)
    │       │       │
    │       │       └─► DPIA-1 rapport fra Dorthe
    │       │               │
    │       │               └─► Root CA Key Ceremony (kun efter Fase 0 komplet)
    │       │
    │       └─► HSM-procurement-input (AP leverer teknisk spec)
    │
    ├─► Engineering Team Lead-onboarding (parallelt med AP, Option B)
    │       │
    │       └─► Tooling-design starter; ikke blokerende for ceremony
    │
    ├─► ROPA-oprettelse (Camilla; input fra AO om dataflows)
    │       │
    │       └─► DPIA-1 kan informeres af ROPA-scope
    │
    └─► CP/CPS-udkast (Mads + CA Administrator)
            │
            └─► Forudsætter CA Administrator onboardet
                    │
                    └─► Ceremony-procedure-godkendelse (Mads)
                            │
                            └─► Key Ceremony Officer klar til eksekvering
```

### Blokerende dependencies (stopper Fase 0)

| Bloker | Hvad der ikke kan ske | Hvem løser |
|---|---|---|
| AP ikke onboardet | DPIA-1 kan ikke afsluttes; HSM-procurement-spec mangler; ceremony-procedure mangler teknisk input | Stefan/FGD bemander AP |
| DPIA-1 ikke afsluttet | Root CA key ceremony må ikke afvikles | Dorthe afslutter; AO kan ikke fremskynde konklusionen |
| CA Administrator ikke onboardet | CP/CPS-udkast mangler operationelt input | AO koordinerer onboarding |
| Mads-godkendelse af ceremony-procedure ikke givet | Ceremony-afvikling blokeret | AO sender procedure til Mads minimum 4 uger forinden |

### Ikke-blokerende dependencies (kan køre parallelt)

- Engineering Team Lead tooling-design (kører parallelt med DPIA-1-proces)
- ROPA-oprettelse (kører parallelt med AP-onboarding)
- HSM-procurement-proces (kører parallelt med DPIA-1; hardware skal dog foreligge inden ceremony)

---

## 5. AO-to-Mads Hand-off — Compliance-scorecard-punkter

Nedenstående punkter sendes til Mads som input til compliance-scorecard. AO stiller spørgsmål og vurderingsanmodninger — Mads ejer konklusionerne.

**Hand-off-frist:** Sendes til Mads straks ved Fase 0-kick-off (inden CA Administrator, HSM-op og KCO onboardes).

### Scorecard-punkter til Mads

**SOD-design**
- Vurder om den planlagte team-komposition (AO, AP, Engineering Team Lead, CA Administrator, HSM-operatør, KCO) sikrer tilstrækkelig separation of duties i samtlige PKI-operationer
- Særlig opmærksomhed: er der scenarier hvor samme person midlertidigt ville varetage to SOD-kritiske roller?

**CP/CPS-scope og policy-OID-strategi**
- Vurder om Fase 0's CP/CPS-udkast-scope er tilstrækkeligt til at dække tre-lags CA-hierarkiet (Root CA offline, Intermediate CA HA-par, Issuing CA online)
- Policy-OID-tildelingsproces: skal FGD registrere eget OID-arc, eller anvendes private-arc?

**HSM-procurement og FIPS-transition**
- Vurder om valgt HSM-løsning til Fase 0 er egnet til fremtidig FIPS 140-3-transition (YubiHSM2 FIPS-variant når NIST CMVP-certificering foreligger)
- Vurder procurement-tidsplan ift. key ceremony-deadline

**Audit-log-arkitektur**
- Vurder om planlagt audit-log-scope (hvem logger hvad under ceremony og daglig drift) er tilstrækkeligt til SOC 2 / ISO 27001
- Note: audit-log-arkitektur krydser GDPR — koordinér med Dorthe om dataminimering

**Ceremony-procedure-godkendelse**
- Mads godkender formelt ceremony-procedure inden KCO kan eksekvere. AO anmoder om at dette godkendelses-punkt indgår i scorecarden med klar deadline

**OCSP/CRL EU-only hosting (Scenario B)**
- Bekræft at planlagt OCSP/CRL-hosting-design overholder EU-only-kravet og er teknisk tilstrækkeligt robust (HA)
- Note: Dorthe vurderer parallelt om Scenario B opfylder GDPR art. 44-49

**Key Ceremony Officer — rolleprofil**
- Mads godkender KCO-profil og qualifikations-krav inden onboarding. Anmodning om at dette sker inden Fase 0 midtvejs

**Engineering Team Lead og AP — SOD Fase 0**
- Bekræft at Option B (ETL separat fra AP fra dag 1) er tilstrækkeligt til governance-kravet; ingen midlertidige undtagelser

### Koordinering med Dorthe (AO noterer til Mads)

Følgende punkter kræver parallelt Dorthe-input og bør koordineres:
- DPIA-1-status og F-spørgsmål (Dorthe ejer; AP leverer teknisk input)
- ROPA-oprettelse (Camilla opretter; Dorthe attesterer)
- Audit-log-dataminimering (Dorthe vurderer GDPR-scope parallelt med Mads' ISO/SOC 2-vurdering)
- OCSP/CRL-hosting GDPR art. 44-49 (EU-only Scenario B — Dorthe bekræfter GDPR-compliance)
- Nye certifikatprofiler med persondata i Subject-felter: AO notificerer Dorthe inden implementering (ud af Fase 0 scope, men princippet gælder fra Fase 1)

---

*Dette dokument er AO's Fase 0-roadmap-udkast. Compliance-konklusioner er ikke givet — de leveres af Mads (governance/ISO/SOC 2) og Dorthe (GDPR) som selvstændige scorecards. Dokumentet er input til Stefan's FGD-koordinering.*
