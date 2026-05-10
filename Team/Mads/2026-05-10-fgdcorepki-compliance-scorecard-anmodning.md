---
dato: 2026-05-10
fra: Application Owner (FGDCorePKI)
til: Mads (CISO)
emne: FGDCorePKI Fase 0 — compliance-scorecard-anmodning
projekt: FGDCorePKI
type: compliance-scorecard-anmodning
svarfrist: 2026-05-17
---

# FGDCorePKI Fase 0 — Compliance-scorecard-anmodning

Mads,

Per hand-off-aftalen fremsender Application Owner (FGDCorePKI) hermed compliance-scorecard-punkter for Fase 0. Anmodningen er genereret på baggrund af Fase 0-roadmap-dokumentet (`Team/projects/fgdcorepki/ao/2026-05-10-pki-produkt-roadmap-fase0.md`, sektion 5).

AO stiller spørgsmål og vurderingsanmodninger — Mads ejer konklusionerne. AO har ikke foretaget compliance-vurdering og erklærer ingen af nedenstående punkter som "compliant" eller "godkendt".

---

## Scorecard-punkter til Mads' vurdering

### 1. SOD-design

Vurder om den planlagte team-komposition (AO, AP, Engineering Team Lead, CA Administrator, HSM-operatør, KCO) sikrer tilstrækkelig separation of duties i samtlige PKI-operationer.

Særlig opmærksomhed: er der scenarier hvor samme person midlertidigt ville varetage to SOD-kritiske roller?

---

### 2. CP/CPS-scope og policy-OID-strategi

Vurder om Fase 0's CP/CPS-udkast-scope er tilstrækkeligt til at dække tre-lags CA-hierarkiet (Root CA offline, Intermediate CA HA-par, Issuing CA online).

Policy-OID-tildelingsproces: skal FGD registrere eget OID-arc, eller anvendes private-arc?

---

### 3. HSM-procurement og FIPS-transition

Vurder om valgt HSM-løsning til Fase 0 er egnet til fremtidig FIPS 140-3-transition (YubiHSM2 FIPS-variant når NIST CMVP-certificering foreligger).

Vurder procurement-tidsplan ift. key ceremony-deadline.

---

### 4. Audit-log-arkitektur

Vurder om planlagt audit-log-scope (hvem logger hvad under ceremony og daglig drift) er tilstrækkeligt til SOC 2 / ISO 27001.

Note til Mads: audit-log-arkitektur krydser GDPR — koordinering med Dorthe om dataminimering bør ske parallelt.

---

### 5. Ceremony-procedure-godkendelse

Mads godkender formelt ceremony-procedure inden KCO kan eksekvere. AO anmoder om at dette godkendelses-punkt indgår i scorecarden med klar deadline.

---

### 6. OCSP/CRL EU-only hosting (Scenario B)

Bekræft at planlagt OCSP/CRL-hosting-design overholder EU-only-kravet og er teknisk tilstrækkeligt robust (HA).

Note til Mads: Dorthe vurderer parallelt om Scenario B opfylder GDPR art. 44-49.

---

### 7. Key Ceremony Officer — rolleprofil

Mads godkender KCO-profil og kvalifikationskrav inden onboarding. AO anmoder om at denne godkendelse sker inden Fase 0 midtvejs.

---

### 8. Engineering Team Lead og AP — SOD Fase 0

Bekræft at Option B (ETL separat fra AP fra dag 1) er tilstrækkeligt til governance-kravet; ingen midlertidige undtagelser.

---

## Koordinering med Dorthe (til Mads' orientering)

Følgende punkter kræver parallelt Dorthe-input. AO noterer disse til Mads' orientering, da de krydser Mads' og Dorthes respektive domæner:

- DPIA-1-status og F-spørgsmål (Dorthe ejer; AP leverer teknisk input)
- ROPA-oprettelse (Camilla opretter; Dorthe attesterer)
- Audit-log-dataminimering (Dorthe vurderer GDPR-scope parallelt med Mads' ISO/SOC 2-vurdering — se punkt 4 ovenfor)
- OCSP/CRL-hosting GDPR art. 44-49 (EU-only Scenario B — Dorthe bekræfter GDPR-compliance — se punkt 6 ovenfor)
- Nye certifikatprofiler med persondata i Subject-felter: AO notificerer Dorthe inden implementering (ud af Fase 0 scope, men princippet gælder fra Fase 1)

---

## Responsfrist og videre flow

**Svarfrist: 2026-05-17** (per hand-off-aftale: Mads leverer scorecard inden 1 uge).

Mads' samlede compliance-scorecard videresendes af Stefan til FGD. FGD's GO på Fase 0 forudsætter at scorecarden foreligger og er behandlet.

AO afventer Mads' vurdering inden CA Administrator, HSM-operatør og Key Ceremony Officer onboardes — dette er et hårdt krav per Fase 0-roadmap.

---

*Fremsendelse fra Application Owner (FGDCorePKI), 2026-05-10.*
*Compliance-konklusioner ejes af Mads (governance/ISO/SOC 2) og Dorthe (GDPR) — ikke af AO.*
