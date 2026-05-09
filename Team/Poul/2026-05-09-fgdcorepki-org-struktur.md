---
dato: 2026-05-09
forfatter: Poul (senior analytiker)
emne: FGDCorePKI — komplet organisations-struktur, rapportering, SOD-matrix, AO/AP-anbefalinger
bestilt-af: Stefan (på vegne af FGD)
input-dokumenter:
  - Team/Mads/2026-05-09-pki-org-krav.md
  - Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md
  - .claude/agents/application-owner.md (template, applikations-agnostisk)
  - .claude/agents/application-provider.md (template, applikations-agnostisk)
fgd-trufne-beslutninger-2026-05-09:
  - FIPS-strategi: Non-FIPS MVP nu (€728 YubiHSM2) + re-keying-plan ved FIPS 140-3-cert; Mads dokumenterer risiko formelt
  - Tier 2 HA: HA-pair med wrap-key sync (~€2.400 ekstra) — SPoF elimineret
  - Root CA validity: 10-12 år (PQC-headroom for ML-DSA-migration)
status: UDKAST — input til Laila (rolle-design), Stefan (koordinering), Mads (peer-review), FGD (godkendelse)
kopi: governance/design-docs/ (compliance-transparens for Trine + Dorthe)
---

# FGDCorePKI — Komplet organisations-struktur

## Sammendrag

Denne rapport leverer den fulde organisations-tegning for FGDCorePKI bygget på Mads' krav-dokument og indarbejder FGDs tre strategiske beslutninger fra 2026-05-09. Rapporten dækker (1) komplet org-chart med 16 distinkte roller, (2) rapporterings-linjer og eskalations-stier, (3) minimumsbemanding for fase 0 med tidsbegrænsede SOD-undtagelser, (4) udvidet SOD-matrix der inkluderer engineering-team-roller, (5) kompetence-krav for AO og AP, og (6) konkret brief til Laila om PKI-specifikke deltas til de eksisterende AO/AP-mandater.

**Centralt budskab til FGD**: Fuldt compliant FGDCorePKI kræver 8 separate personer. Fase 0-bemanding kan reduceres til **5-6 personer** med 4 dokumenterede tidsbegrænsede SOD-undtagelser, men 4 SOD-krav er **absolutte og kan aldrig kombineres**: (1) CA Administrator × PKI Auditor, (2) CA Administrator × Registration Authority, (3) PKI Auditor × enhver anden operationel rolle, (4) AO × CA Administrator. Re-keying-event når FIPS 140-3-cert lander vil kræve fuld key ceremony — Key Ceremony Officer-rollen skal designes med dette in mind.

---

## 1. Komplet org-chart for FGDCorePKI

```
FGD (ejer / dataansvarlig / GDPR-controller)
│
│   Direkte rapporterings-linjer (uafhængige af Stefan-orkestrering):
│   ├── Mads (CA Manager, CISO)         — eskalation ved CA-incidents
│   ├── Trine (PKI Auditor, audit)       — uafhængig audit-linje
│   └── Dorthe (DPO)                     — GDPR art. 38(3) uafhængighed
│
└── Stefan (organisator, orkestrering — ikke autoritet)
    │
    ├─[GOVERNANCE-LAG — strategi, policy, audit]
    │
    ├── Mads — CISO + CA Manager (PKI-governance) [PERMANENT ROLLE]
    │   ├── Funktion: PKI policy-ejerskab (CP/CPS), budget, leverandør-relationer,
    │   │             eskalations-knude ved CA-incidents, godkender CA-rotation
    │   │             og re-keying-events (NB: re-keying kommer ved FIPS 140-3-cert
    │   │             pr. FGDs MVP-strategi 2026-05-09)
    │   ├── SOD: Ingen direkte CA-system-/HSM-adgang. Må IKKE være CA Admin,
    │   │        HSM-op, PKI Auditor eller AO/AP for FGDCorePKI
    │   ├── Reports to: FGD direkte
    │   └── Note: Kan midlertidigt varetage Key Ceremony Officer + Security
    │             Officer i fase 0 (dokumenteret undtagelse, normalisering inden
    │             1. eksterne kunde)
    │
    ├── Trine — Compliance-auditor + PKI Auditor [PERMANENT ROLLE, UDVIDES]
    │   ├── Funktion: Uafhængig audit af CA-drift, ISO 27001/SOC 2 Type II,
    │   │             review af certificate issuance logs, HSM audit logs,
    │   │             ceremony-protokoller, CP/CPS-efterlevelse, kvartalvis
    │   │             access-review, attesterer DR-tests
    │   ├── SOD: ABSOLUT separation fra alle øvrige PKI-roller. Ingen
    │   │        operationel CA-adgang. Read-only audit logs. Kan ikke have
    │   │        været CA Admin de seneste 12 måneder
    │   ├── Reports to: FGD direkte (IKKE via Mads — uafhængighed)
    │   └── Kompetence-gap: PKI-audit-metodologi (WebTrust for CAs eller intern
    │                       tilpasning) — Mads identificerer træningsbehov
    │
    ├── Dorthe — DPO (GDPR-data-subject-pligt for PKI) [PERMANENT ROLLE]
    │   ├── Funktion: GDPR art. 33 72-timers vurdering ved CA-incidents der
    │   │             berører persondata-flows; ROPA-attestation; DPIA ved
    │   │             nye PKI-data-flows
    │   ├── SOD: Uafhængig af Mads/CA Manager (GDPR art. 38(3))
    │   ├── Reports to: FGD direkte
    │   └── Trigger: Auto-notification ved alle "høj"/"kritisk" CA-incidents
    │
    ├─[APPLIKATIONS-LAG — ejerskab og leverance for FGDCorePKI]
    │
    ├── Application Owner (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Produkt-ejerskab for FGDCorePKI som applikation
    │   │             (ikke som CA-drift): roadmap, feature-prioritering,
    │   │             kunde-relationer, compliance-koordinering. Fungerer
    │   │             som forretnings-trigger for Mads/Trine/Dorthe
    │   ├── SOD: Må IKKE have adgang til CA-software, HSM, key ceremony.
    │   │        Må IKKE være Application Provider (standard SOD).
    │   │        Må IKKE være CA Admin, HSM-op, RA, PKI Auditor
    │   ├── Reports to: Stefan (orkestrering); compliance til Mads + Dorthe
    │   └── Hvem dette er: Forretnings-/produkt-orienteret person; ikke
    │                      teknisk PKI-operatør
    │
    ├── Application Provider (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Teknisk arkitektur-ejer for FGDCorePKI: integrations-
    │   │             arkitektur, dataflow-design, server-infrastruktur (OS,
    │   │             netværk, Tailscale, monitoring), HSM-bridge-design.
    │   │             Pouls primære tekniske dybde-ressource. Designer
    │   │             tekniske krav; eksekverer IKKE CA-operationer
    │   ├── SOD: Må IKKE have direkte CA-software-runtime-adgang (CA Admin
    │   │        er eneste autoriserede CA-operatør). Må IKKE være HSM-op
    │   │        (specificerer HSM-arkitektur; eksekverer ikke). Må IKKE
    │   │        være Application Owner. Må IKKE være PKI Auditor
    │   ├── Reports to: AO (funktionelt), Stefan (orkestrering)
    │   └── Hvem dette er: Senior teknisk arkitekt med X.509/PKCS#11/ACME-
    │                      kompetence; *forskellig* fra CA Administrator
    │                      som er drifts-rolle
    │
    ├─[PKI-OPERATIONELT LAG — daglig CA-drift]
    │
    ├── CA Administrator (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Day-to-day drift af CA-software (Tier 2 ICA + Tier 3
    │   │             Issuing CA): certificate issuance-eksekvering, CRL-
    │   │             generering, OCSP-responder-drift, policy enforcement
    │   │             i CA-software. Ingen Root CA-adgang udenfor ceremony
    │   ├── SOD: Må IKKE være PKI Auditor, RA, HSM-op, AO. Dual-control
    │   │        kræves ved alle Root CA-operationer
    │   ├── Reports to: CA Manager (Mads) — funktionelt og compliance-mæssigt
    │   └── Eskalations-sti: CA Admin → CA Manager (Mads) → FGD
    │
    ├── HSM-operatør (FGDCorePKI) [PROJECT-ROLLE — separat tillæg]
    │   ├── Funktion: YubiHSM2 hardware-administration: initialisering, key
    │   │             generation under ceremony, wrap-key management, backup
    │   │             (encrypted key export), firmware-opdatering, session-
    │   │             authentication, DR-test. Eksekverer HSM-operationer
    │   │             efter Mads' godkendelse
    │   ├── SOD: Må IKKE være CA Admin, PKI Auditor. X under selve key
    │   │        ceremony (Key Ceremony Officer er separat person der)
    │   ├── Reports to: CA Manager (Mads)
    │   └── Person-anbefaling (Mads §6.1): Separat tillæg, ikke Bjarne fra
    │                                      Knowlagecentral (PKI-blast-radius
    │                                      større; cross-application risiko)
    │
    ├── Key Ceremony Officer (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Formel ceremoni-autoritet ved Root CA key ceremony og
    │   │             re-keying: script-ejer, tidslog, command authorization
    │   │             (ikke eksekvering — det er HSM-operatøren). Koordinerer
    │   │             m-of-n smartcard holders, kalder witnesses
    │   ├── SOD: Må IKKE varetage HSM-operatør-rollen UNDER samme ceremoni.
    │   │        Bør ikke være CA Admin. Kan ikke være witness for egen ceremoni
    │   ├── Reports to: CA Manager (Mads)
    │   └── Note: Kan i fase 0 varetages af Mads (dokumenteret undtagelse).
    │             Kritisk vigtig rolle ved planlagt re-keying når FIPS 140-3-
    │             cert lander
    │
    ├── Registration Authority (RA) (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Godkendelse af CSRs før de videresendes til Issuing CA.
    │   │             Identitetsverifikation, policy-tjek (algoritme, navne,
    │   │             validity period), approve/reject med begrundelse i
    │   │             audit-log. Adgang til RA-portal — IKKE CA-software
    │   ├── SOD: Må IKKE være CA Admin (klassisk "mark and approve your own
    │   │        work"-konflikt). Bør have uafhængig rapporterings-linje
    │   │        fra CA Admin
    │   ├── Reports to: CA Manager (Mads) — for compliance-uafhængighed
    │   └── Fase 0-mulighed: Kan kombineres med Security Officer (lavt volume),
    │                        men ALDRIG med CA Admin
    │
    ├── Security Officer (FGDCorePKI) [PROJECT-ROLLE — ny besættes]
    │   ├── Funktion: Second authorizer ved kritiske operationer: Root CA-
    │   │             aktivering, CA-revokering, HSM factory reset, wrap-key
    │   │             rotation, emergency CA shutdown. "Second pair of eyes"
    │   │             i to-mands-princippet
    │   ├── SOD: Må IKKE være HSM-op ved samme operation. Funktionelt
    │   │        uafhængig af CA Admin
    │   ├── Reports to: CA Manager (Mads)
    │   └── Note: I fase 0 kan Mads varetage rollen (dokumenteret undtagelse;
    │             dog problematisk fordi CA Manager + Security Officer skaber
    │             review-konflikt — på sigt separat)
    │
    ├─[ENGINEERING-LAG — bygger og vedligeholder PKI-integrationer]
    │
    └── Engineering Team (FGDCorePKI) [PROJECT-ROLLER]
        │
        ├── PKI Tech Lead / Architect (1 person)
        │   ├── Funktion: PKI-arkitektur, X.509-standards, PKCS#11, RFC 3647-
        │   │             aligned CP/CPS-udkast (§6 technical security
        │   │             controls), HSM-bridge design, threat-modeling
        │   ├── SOD: Må IKKE være CA Admin, HSM-op, PKI Auditor, RA. Kan
        │   │        teoretisk overlappe med Application Provider-rollen,
        │   │        men er IKKE samme rolle (AP er applikations-ejer; Tech
        │   │        Lead er engineering-leder under AP)
        │   ├── Reports to: Application Provider (funktionelt)
        │   └── Compliance-gate: Mads godkender alle arkitektur-kritiske
        │                        designs inden Engineering implementerer
        │
        ├── Senior PKI Engineer (1-2 personer)
        │   ├── Funktion: HSM-bridge-implementering (Go/Rust), ACME-server/
        │   │             klient, PKCS#11/PKCS#12-integration, certificate
        │   │             lifecycle automation, API-design (REST + OpenAPI 3.1)
        │   ├── SOD: Må IKKE være CA Admin, HSM-op, PKI Auditor, RA. Code-
        │   │        review obligatorisk inden merge (Mads + Tech Lead)
        │   ├── Reports to: PKI Tech Lead
        │   └── Adgangsbegrænsning: Ingen prod-CA-runtime-adgang; deploy via
        │                           CA Administrator
        │
        └── SRE / Platform Engineer (1 person)
            ├── Funktion: Cert-lifecycle monitoring, CRL/OCSP health checks,
            │             alerting, rotation automation, on-prem infra
            │             (Tailscale, service mesh, backup), HSM-throughput-
            │             test, SIEM-integration, network zone-model A-F-drift
            ├── SOD: Må IKKE være CA Admin, HSM-op, PKI Auditor. Kan have
            │        read-only-overvågnings-adgang men ikke runtime-CA-control
            ├── Reports to: PKI Tech Lead
            └── Vagt-dækning: Sammen med Senior PKI Engineer dækker SRE on-
                              call for non-crypto-incidents (CA-software-
                              crash er CA Admin)
```

### Markering: Permanent vs. Project-rolle

| Rolle | Type | Eksisterer i dag? |
|---|---|---|
| FGD | Ejer (juridisk) | Ja |
| Stefan | Permanent | Ja |
| Mads (CA Manager-funktion) | Permanent | Ja (CISO) — funktion udvides |
| Trine (PKI Auditor-udvidelse) | Permanent | Ja (compliance-auditor) — funktion udvides |
| Dorthe | Permanent | Ja (DPO) |
| Application Owner (FGDCorePKI) | Project | Nej — skal besættes |
| Application Provider (FGDCorePKI) | Project | Nej — skal besættes |
| CA Administrator | Project | Nej — skal besættes |
| HSM-operatør (FGDCorePKI) | Project | Bjarne findes for Knowlagecentral; FGDCorePKI separat tillæg |
| Key Ceremony Officer | Project | Nej — Mads midlertidigt i fase 0 |
| Registration Authority | Project | Nej — skal besættes |
| Security Officer | Project | Nej — Mads midlertidigt i fase 0 |
| PKI Tech Lead / Architect | Project | Nej — skal besættes |
| Senior PKI Engineer (1-2) | Project | Nej — skal besættes |
| SRE / Platform Engineer | Project | Nej — skal besættes |

---

## 2. Rapporterings-linjer, samarbejde og eskalations-stier

For hver rolle: hvem rapporterer de til, hvem samarbejder de med, hvad er eskalerings-stien?

### 2.1 Governance-lag

| Rolle | Reports to | Samarbejder med | Eskalations-sti |
|---|---|---|---|
| **Mads (CA Manager + CISO)** | FGD direkte | Trine, Dorthe, Stefan, Application Owner, alle PKI-operationelle roller | Ved CA-incident: Mads → FGD inden 1 time (kritisk) eller 4 timer (høj). Ved compliance-konflikt med AO: Mads kan eskalere uden om AO til FGD |
| **Trine (PKI Auditor)** | FGD direkte (audit-uafhængighed) | Mads (notification ved fund), Dorthe (GDPR-evidence), engineering team (audit-evidence-collection) | Ved audit-fund: Trine → CA Manager med kopi til FGD. Ved tegn på fraud/cover-up: Trine → FGD direkte uden om Mads |
| **Dorthe (DPO)** | FGD direkte (GDPR art. 38(3) uafhængighed) | Mads (security-vurderinger), Trine (audit-koordination), Application Owner (DPIA-screening) | GDPR art. 33-incident: Dorthe → FGD inden 24 timer; Datatilsynet inden 72 timer (FGD indsender) |

### 2.2 Applikations-lag

| Rolle | Reports to | Samarbejder med | Eskalations-sti |
|---|---|---|---|
| **Application Owner (FGDCorePKI)** | Stefan (orkestrering); compliance til Mads + Dorthe | Application Provider, PKI Tech Lead, Mads, Trine, Dorthe | Roadmap-blokere: AO → Stefan → FGD. Compliance-konflikt: Mads/Trine/Dorthe → FGD uden om AO. AO kan ikke filtrere compliance-rapportering |
| **Application Provider (FGDCorePKI)** | AO (funktionelt); Stefan (orkestrering) | PKI Tech Lead (engineering-leder), Poul (analytisk dybgang), Mads (security-design-review), Camilla (git-execution) | Tekniske blokere: AP → AO. Security-fund: AP → Mads direkte (ikke via AO). HSM-konfigurations-spørgsmål: AP → Mads (AP rører ikke HSM selv) |

### 2.3 PKI-operationelt lag

| Rolle | Reports to | Samarbejder med | Eskalations-sti |
|---|---|---|---|
| **CA Administrator** | Mads (CA Manager) | HSM-op (men separation under ceremony), RA, Security Officer, SRE | Drifts-incident: CA Admin → Mads inden 1 time (kritisk). Mistænkt CA-kompromittering: CA Admin → Mads + Security Officer parallelt; CA tages offline inden Mads-svar |
| **HSM-operatør (FGDCorePKI)** | Mads (CA Manager) | Key Ceremony Officer (under ceremoni), CA Admin (men separation), Security Officer (dual-control) | HSM-fund (tamper, fejlfunktion): HSM-op → Mads + Security Officer inden 1 time. Bjarne (Knowlagecentral-HSM-op) er IKKE en eskalations-vej for FGDCorePKI |
| **Key Ceremony Officer** | Mads (CA Manager) | HSM-op (eksekverer ceremoni-kommandoer), Security Officer (witness/authorizer), Trine (audit-observatør) | Ceremoni-afbrydelse eller -afvigelse: KCO → Mads øjeblikkeligt; Trine notificeres for audit-protokol |
| **Registration Authority** | Mads (CA Manager — for compliance-uafhængighed fra CA Admin) | CA Admin (modtager godkendte CSRs), Application Owner (forretnings-side), kunder/applikationer (rekvirenter) | Mistænkt rogue-CSR: RA → Mads + CA Admin parallelt; CSR holdes i kø. Brute-force: RA → SRE for rate-limit + Mads |
| **Security Officer** | Mads (CA Manager) | CA Admin (review af kritiske operationer), HSM-op (dual-control), Key Ceremony Officer (witness) | Detekteret SOD-brud eller dual-control-bypass: SO → Mads + FGD parallelt |

### 2.4 Engineering-lag

| Rolle | Reports to | Samarbejder med | Eskalations-sti |
|---|---|---|---|
| **PKI Tech Lead / Architect** | Application Provider | Senior PKI Engineers (leder), SRE, Mads (security-design-review), Poul (analytisk peer) | Arkitektur-uenighed med AP: Tech Lead → AP → Stefan. Security-konflikt med Mads: Tech Lead → AP → Mads → FGD |
| **Senior PKI Engineer** | PKI Tech Lead | Tech Lead, SRE, andre engineers | Kode/sikkerheds-fund: Engineer → Tech Lead → Mads. Whistle-blow: Engineer → Mads direkte uden om Tech Lead (anti-retaliation) |
| **SRE / Platform Engineer** | PKI Tech Lead | CA Admin (drifts-koordination), Tech Lead, Mads (alert-design) | Prod-incident (non-crypto): SRE → Tech Lead → AP. Crypto/CA-incident: SRE → CA Admin + Mads inden 15 min |

### 2.5 Diagram over eskalations-veje

```
                    FGD
                     ▲
   ┌─────────────────┼─────────────────┐
   │                 │                 │
   │    [uafhængige direkte linjer]   │
   │                 │                 │
 Mads             Trine             Dorthe
 (CA Mgr)       (PKI Auditor)       (DPO)
   ▲                 ▲                 ▲
   │                 │                 │
   ├── CA Admin      │                 │
   ├── HSM-op        │  (audit-evidence│
   ├── KCO           │   fra alle)     │  (GDPR-incidents
   ├── RA            │                 │   fra alle)
   └── Security Off  │                 │

                  Stefan
                  (orkestrering)
                     │
                     ├── Application Owner
                     │      │
                     │      ▼ funktionelt
                     ├── Application Provider
                     │      │
                     │      ▼
                     │   PKI Tech Lead
                     │      │
                     │      ▼
                     │   Senior PKI Engineers, SRE
```

**Kritisk uafhængigheds-princip**: Mads, Trine, Dorthe har alle direkte eskalation til FGD uden om Stefan eller Application Owner. Dette er compliance-forankret (GDPR art. 38(3) for Dorthe; SOC 2 CC4.1 og ISO 27001 A.5.3 for Trine; ISO 27001 A.5.2 for Mads som CISO).

---

## 3. Minimumsbemanding fase 0-1

Mads' krav-dokument §6.3 angiver **8 separate personer** for fuldt compliant PKI. Hvad er realistisk fase 0-bemanding?

### 3.1 Fase 0 (MVP-opstart, 0-6 måneder fra ceremony)

**Realistisk antal personer**: 5-6, med fire dokumenterede tidsbegrænsede SOD-undtagelser.

| Person | Roller varetaget | Begrundelse | Normaliserings-deadline |
|---|---|---|---|
| **Person A (Mads)** | CA Manager + Key Ceremony Officer + Security Officer | Mads har autoritet til alle tre; midlertidigt acceptabelt fordi der ikke udstedes mod eksterne kunder endnu | **Inden 1. eksterne kunde** — Key Ceremony Officer skal være separat person inden første re-keying-event (forventet ved FIPS 140-3-cert) |
| **Person B** | CA Administrator | Dedikeret rolle fra dag 1 — kan ikke kombineres med RA, PKI Auditor eller HSM-op | (ingen normalisering nødvendig — rollen er allerede separat) |
| **Person C** | HSM-operatør (FGDCorePKI) | Dedikeret separat fra Bjarne (Knowlagecentral) per Mads' anbefaling §6.1 | (ingen normalisering nødvendig) |
| **Person D** | Registration Authority + (evt. Security Officer ved lavt volume, men IKKE samtidigt med Mads-Security Officer) | RA + SO kombineres kun ved meget lavt cert-volume; ALDRIG kombineret med CA Admin | Inden 100 cert-issuances/måned: RA og SO separates |
| **Person E (Trine)** | PKI Auditor | Eksisterende rolle udvides; ABSOLUT separation fra alt operationelt | (ingen normalisering — absolut SOD) |
| **Person F (Dorthe)** | DPO | Eksisterende rolle; auto-notification ved kritiske CA-incidents | (ingen normalisering) |

Engineering-team (yderligere 3 personer ved fuld bemanding):
- I fase 0 kan PKI Tech Lead + 1 Senior Engineer + 1 SRE være 2 personer hvis Tech Lead også er senior engineer — men ALDRIG 1 person alene (ingen vagt-dækning, ingen code-review-SOD).

**Minimum fase 0-bemanding: 5 personer (PKI-roller) + 2 personer (engineering minimum) = 7 personer total**, ikke 8 som Mads angiver. Diskrepansen forklares af at Mads' 8 var forudsætning for fuldt compliant produktions-drift med eksterne kunder; fase 0-MVP uden eksterne kunder kan acceptere midlertidige kombinationer.

Application Owner og Application Provider lægges *ovenpå* dette: AO + AP = 2 yderligere personer der hverken må overlappe med hinanden eller med PKI-operationelle roller. Praktisk: AO kan være FGD selv eller en delegeret forretnings-leder; AP er teknisk arkitekt der ikke selv drifter CA. **Total fase 0: 9-10 personer**.

### 3.2 Tilladte midlertidige kombinationer (med dokumenterede undtagelser)

Følgende kombinationer er **acceptable i fase 0** med formel undtagelses-godkendelse (Mads + FGD + Dorthe):

1. **Mads = CA Manager + Key Ceremony Officer**: OK i opstart; risk: ceremoni-script-ejer er også policy-ejer (mild SOD-issue, men gennemskuelig)
2. **Mads = CA Manager + Security Officer**: PROBLEMATISK fordi Security Officer skal review CA Manager's beslutninger; midlertidig dækning kan ske i 0-3 måneder mod dokumenteret forpligtelse til separation. Trine attesterer kvartalsvis at undtagelsen overvåges
3. **RA = Security Officer**: OK ved <100 issuances/måned; SOD-grænsen er at RA ALDRIG må være CA Admin
4. **PKI Tech Lead = Senior PKI Engineer**: Acceptabelt midlertidigt hvis 2 separate engineers + SRE er på holdet (dvs. minimum 3-personers engineering-hold)

**Hver undtagelse kræver**:
- Formel skriftlig godkendelse fra Mads + Dorthe (FGD-override 2026-05-06)
- Tidsbegrænset gyldighed (max 6 måneder, evt. forlængelse efter ny vurdering)
- Trine kvartalsvis attestation
- Eksplicit normaliserings-plan med deadline
- Logning i `governance/sod-undtagelser/<dato>-<rolle>-<periode>.md`

### 3.3 ABSOLUTTE SOD-krav (aldrig kombinerbare — heller ikke midlertidigt)

Disse fire kombinationer er **forbudte under alle omstændigheder**, også i fase 0:

| Kombination | Begrundelse | Reference |
|---|---|---|
| **CA Administrator × PKI Auditor** | Klassisk "audit your own work"-konflikt; bryder SOC 2 CC4.1, ISO 27001 A.5.3, og PKI-industri-standard (WebTrust 4.1) | Mads §5 nøgle SOD-konflikt #1 |
| **CA Administrator × Registration Authority** | "Mark and approve your own work"-konflikt; CA Admin der godkender egne CSRs er bypass af RA-gate | Mads §5 nøgle SOD-konflikt #2 |
| **PKI Auditor × enhver anden operationel rolle** | Audit-uafhængighed er fundamental; "kan ikke have været CA Admin de seneste 12 måneder" gælder også for kombinationer | Mads §1.6 SOD-krav |
| **Application Owner × CA Administrator** (eller HSM-op, RA, PKI Auditor, Security Officer) | AO er forretnings-/produkt-rolle; operationel CA-adgang skaber compliance-attestation-konflikt og triple-risk-mønster | Mads §5 nøgle SOD-konflikt #3; AO-mandat-template hard rule |

Disse er teknisk obligatoriske og må aldrig opblødes — heller ikke ved nødsituation. Ved aktiv databreach kan andre SOD-undtagelser midlertidigt accepteres (per nødsituations-klausulen i CLAUDE.md), men ikke disse fire.

---

## 4. SOD-matrix udvidet med engineering-team

Mads' SOD-matrix (krav-dok §5) har "Engineering Lead" som aggregat. Jeg udvider med de tre engineering-roller separat (Tech Lead/Architect, Senior PKI Engineer, SRE/Platform Engineer) og bevarer Mads' øvrige struktur. **X** = må aldrig kombineres; **D** = dual-control; **OK** = tilladt; **OK*** = tilladt med tidsbegrænset undtagelse; **—** = ikke relevant.

| | CA Mgr (Mads) | CA Admin | HSM-op | KCO | RA | PKI Auditor (Trine) | Sec Off | AO | AP | Tech Lead | Sr Engineer | SRE |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **CA Manager** | — | X | X | OK* | X | X | OK* | X | X | X | X | OK |
| **CA Administrator** | X | — | X | X | X | X | X | X | X | X | X | X |
| **HSM-operatør** | X | X | — | X† | OK | X | X | X | X | X | X | X |
| **Key Ceremony Officer** | OK* | X | X† | — | OK | X | X | X | X | X | X | X |
| **Registration Authority** | X | X | OK | OK | — | X | OK* | X | X | X | OK | OK |
| **PKI Auditor** | X | X | X | X | X | — | X | X | X | X | X | X |
| **Security Officer** | OK* | X | X | X | OK* | X | — | X | X | X | OK | OK |
| **Application Owner** | X | X | X | X | X | X | X | — | X | X | X | X |
| **Application Provider** | X | X | X | X | X | X | X | X | — | OK‡ | X | X |
| **PKI Tech Lead** | X | X | X | X | X | X | X | X | OK‡ | — | OK | OK |
| **Senior PKI Engineer** | X | X | X | X | OK | X | OK | X | X | OK | — | OK |
| **SRE / Platform Eng** | OK | X | X | X | OK | X | OK | X | X | OK | OK | — |

**Fodnoter:**
- **†** = HSM-operatør og Key Ceremony Officer: X under selve ceremonien; OK i øvrige perioder.
- **‡** = AP og Tech Lead: AP er applikations-arkitektur-ejer; Tech Lead er engineering-team-leder. De kan teoretisk være samme person i meget lille team, men det skaber engineering-management × applikations-ejerskabs-konflikt. **Anbefaling**: hold dem separate. Hvis kombineret: dokumenteret undtagelse, normaliserings-plan, Trine attesterer kvartalsvis.

### 4.1 Nye engineering-specifikke SOD-konflikter at fremhæve

1. **Tech Lead × CA Administrator**: Tech Lead designer integrationer mod CA-software; må ikke selv eksekvere CA-operationer i prod. Ellers bypass af change-management.
2. **Senior Engineer × PKI Auditor**: Engineers har commit-rettigheder til CA-relateret kode; PKI Auditor reviewer at koden følger CP/CPS. Samme person bryder code-review-SOD.
3. **SRE × HSM-operatør**: SRE har monitor-adgang til HSM-metrics; må ikke have runtime-control. Adskillelse forhindrer at SRE "lige tjekker" HSM-state og udfører operation.
4. **AP × Tech Lead**: Beskrevet ovenfor — best practice separat.
5. **Engineering Team × Production CA Operations**: Hele engineering-teamet er under code-review-gate (Mads + Tech Lead + commit-reviewer Vibeke når aktiv); ingen direkte deploy uden CA Administrator sign-off.

### 4.2 Nye dual-control (D)-operationer med engineering-deltagelse

Tilføjes til Mads' liste i §5:

| Operation | Person 1 | Person 2 | Person 3 (witness) |
|---|---|---|---|
| Deploy af CA-software-opdatering til prod | CA Administrator | PKI Tech Lead (verifikation) | Mads (godkendelse) |
| HSM-firmware-opdatering | HSM-operatør | SRE (rollback-readiness) | Mads (godkendelse) |
| Network zone-model A-F-ændring | SRE | Mads | Trine (audit-observation) |
| Ny ACME/EST/SCEP-endpoint i prod | Senior Engineer (PR) | Mads (compliance-review) | CA Admin (deploy) |
| CA-software-konfigurations-ændring (CRL/OCSP-policy) | CA Administrator | Mads | Trine (notification) |

---

## 5. Kritiske kompetence-krav for AO og AP

### 5.1 Application Owner (FGDCorePKI) — kompetence-krav

**Minimale tekniske kompetencer (dag 1):**

- **Forståelse for PKI's forretnings-rolle** (ikke teknik-dybde): hvad er en CA, hvorfor trust hierarki, hvilke kunder bruger certifikater til hvad
- **Compliance-koordinering**: kunne læse en compliance-scorecard fra Mads, en DPIA fra Dorthe, en audit-rapport fra Trine — og oversætte til roadmap-konsekvenser
- **Risk-bevidsthed**: forståelse for at PKI-fejl er nukleart-niveau (en kompromitteret Root påvirker alle udstedte certifikater nogensinde)
- **Roadmap- og produkt-management-erfaring**: agile/kanban, prioritering, stakeholder-håndtering, kunde-feedback-håndtering

**Minimale governance-kompetencer (dag 1):**

- **GDPR-grundforståelse**: art. 30 ROPA, art. 33 breach-notification, art. 35 DPIA-trigger
- **ISO 27001 A.5.3 SOD-princip**: hvorfor AO ikke selv kan attestere compliance
- **Eskalations-disciplin**: forståelse for at Mads/Trine/Dorthe kan eskalere uden om AO til FGD — og det er korrekt adfærd, ikke loyalitets-brud

**Hvad AO IKKE skal kunne (eksplicit forbudt):**

- Eksekvere CA-software-kommandoer
- Tilgå YubiHSM2 fysisk eller logisk
- Deltage i key ceremony (heller ikke som witness)
- Forhandle DPA'er med kunder eller leverandører (Dorthe ejer dette)

**Erfaring kan komme fra:**

- Tidligere produkt-leder for sikkerheds- eller compliance-tunge produkter (SaaS-platforme med GDPR-impact, fintech, healthtech, gov-tech)
- Tidligere program-manager for ISO 27001/SOC 2-implementeringer (FGD's egne governance-roller var en god skole)
- Tidligere kunde-success-leder for B2B-produkter med compliance-krav
- Forretnings-/jura-baggrund med teknisk interesse (men ikke teknisk implementator-rolle)

**Erfaring fra rene CA-administrator-/ingeniør-roller er ikke optimalt** — det skaber bias mod operationel involvering, hvilket SOD-mæssigt er forbudt.

### 5.2 Application Provider (FGDCorePKI) — kompetence-krav

**Minimale tekniske kompetencer (dag 1):**

- **PKI-arkitektur i dybden**: three-tier hierarki, X.509 v3 (RFC 5280) — herunder Name Constraints, pathLenConstraint, Policy Constraints, AIA/CDP-extensions; CRL/OCSP-flows
- **PKCS#11 v2.40+** og YubiHSM2's PKCS#11-profil specifikt (forskelle fra Thales/Entrust); session-management, object-handling, capability-model
- **Protokoller**: ACME (RFC 8555), EST (RFC 7030), SCEP (RFC 8894), CMP (RFC 4210); HTTPS/TLS 1.3-design med must-staple (RFC 7633)
- **Kryptografi-grundlag**: RSA, ECDSA P-384/P-521, EdDSA (Ed25519), SHA-2-familien; bevidsthed om PQC-roadmap (ML-DSA FIPS 204 finaliseret aug 2024)
- **HSM-bridge-design**: arkitektur for sikker remote HSM-adgang (Tailscale eller mTLS), session-isolation, capability-restrictions
- **Threat-modeling for CA-angreb**: STRIDE applied til PKI — rogue certificates, OCSP-spoofing, key compromise, signing-oracle-angreb, downgrade-angreb
- **Network-arkitektur**: zone-model A-F-design (per Pouls 10-vinkler-analyse vinkel 6), firewall-rules, mTLS-mesh
- **Observability**: OpenTelemetry, signed log shipping, audit-log immutability via WORM eller kryptografisk chained log
- **Cloud/on-prem-platforme**: Tailscale, Linux server-administration på et arkitektur-niveau (eksekverer ikke selv konfigurationen — det er SRE/Camilla)

**Minimale governance-kompetencer (dag 1):**

- **RFC 3647 CP/CPS-struktur**: AP er primær udkast-forfatter af CP/CPS §6 (technical security controls); skal kunne strukturere policy-dokument
- **NIST SP 800-57 Part 1 Rev 5**: key management lifecycle (pre-activation → active → suspended/deactivated → compromised → destroyed)
- **NIST SP 800-131A Rev 2 (Final)**: cryptographic algorithm transitions; bevidsthed om Rev 3 draft (Oct 2024)
- **OWASP API Security Top 10 (2023)**: især BOLA/BOPLA, SSRF, broken authentication
- **Forståelse for FIPS 140-3 Level 3-krav** (per FGDs 2026-05-09-beslutning: Non-FIPS MVP nu, re-keying-plan ved cert): hvad ændrer sig ved migration, hvilke key-objects skal re-genereres
- **Code-review-disciplin**: forståelse for at AP's tekniske beslutninger skal gennem Mads' compliance-gate

**Hvad AP IKKE skal kunne (eksplicit forbudt):**

- Eksekvere CA-software-kommandoer i prod (det er CA Administrator)
- Røre YubiHSM2 fysisk eller logisk (det er HSM-operatør)
- Deltage som autoriserende part i key ceremony (KCO + Security Officer ejer det)
- Self-deploy til prod (skal gennem CA Admin + change-management)
- Forhandle DPA'er med leverandører (Dorthe)
- Erklære feature "compliant" (Mads + Trine + Dorthe)

**Erfaring kan komme fra:**

- Senior security-arkitekt-rolle hos en CA-leverandør (DigiCert, Sectigo, Entrust, GlobalSign) eller stor enterprise med intern PKI
- Senior platform-engineer-rolle med stærk fokus på X.509/TLS-infrastruktur (workload identity, service mesh, mutual TLS)
- Cryptography engineer fra HSM-leverandør (Thales, Utimaco, AWS CloudHSM-team) — men cross-trained i CA-side
- IETF/RFC-bidragyder i LAMPS WG eller ACME WG (sjældent men optimalt)
- Tidligere FGD-rolle Bjarne (Knowlagecentral HSM-operatør) — **er IKKE optimal til AP** fordi (a) AP for FGDCorePKI må ikke være HSM-operatør, og (b) cross-application AP-rolle ville kombinere Knowlagecentral- og FGDCorePKI-arkitektur-ejerskab (governance-spændende)

**Erfaring der ikke er tilstrækkelig**: ren applikations-udvikler uden PKI-baggrund, generel DevOps uden kryptografi-erfaring, audit-/compliance-baggrund uden teknisk dybde.

### 5.3 Sammenligning: AO vs. AP for FGDCorePKI

| Aspekt | Application Owner | Application Provider |
|---|---|---|
| Forretnings-fokus | Primær | Sekundær (forstår, men ejer ikke) |
| Teknisk dybde | Lav-medium | Høj-meget høj (PKI-specialist-niveau) |
| Compliance-koordinering | Trigger og koordinering | Teknisk input til DPIA/scorecard |
| Beslutnings-autoritet | Roadmap, prioritering, kunde | Arkitektur, integration, tekniske krav |
| SOD-rod | Må ikke være CA-operatør | Må ikke være CA-operatør eller HSM-op |
| Kompetence-spektrum | Bredde + governance | Dybde + arkitektur |
| Eksempel-titel | Senior Product Manager (PKI), Head of PKI Product | Principal PKI Architect, Distinguished Engineer (PKI) |

---

## 6. Brief til Laila — PKI-specifikke deltas til AO og AP-mandater

Laila har allerede leveret applikations-agnostiske AO/AP-template-mandater (`.claude/agents/application-owner.md` og `application-provider.md`, godkendt og aktiveret). De er korrekt designede generisk. For FGDCorePKI skal Laila lave to **project-specifikke instans-mandater** der bygger ovenpå template'erne.

### 6.1 PKI-specifikke deltas til AO-mandatet for FGDCorePKI

**Tilføjelser til mandat-body:**

1. **PKI-domæne-specifik mission-tilføjelse**: AO for FGDCorePKI har ekstra ansvar for at koordinere CA-policy-godkendelse (CP/CPS) med CA Manager (Mads) og PKI Auditor (Trine). AO ejer ikke CP/CPS-indhold men ejer at processen kører.
2. **PKI-specifik SOD (hard rule)**:
   - AO må IKKE have adgang til CA-software (Tier 2 ICA, Tier 3 Issuing CA)
   - AO må IKKE have adgang til YubiHSM2 (fysisk eller logisk)
   - AO må IKKE deltage i key ceremony i nogen rolle (heller ikke som witness)
   - AO må IKKE udstede sin egen CSR uden gennem RA-gate
   - AO må IKKE være CA Administrator, HSM-operatør, RA, PKI Auditor eller Security Officer
3. **PKI-specifik compliance-koordinering**: AO initierer CP/CPS-review-cyklus årligt med Mads + Trine; AO sikrer ROPA-attestation kvartalvis (eksisterende standard) men med eksplicit nævnelse af PKI-data-flows (CSR-data, certifikat-metadata, audit-trails)
4. **CA-incident-håndtering**: AO modtager status fra Mads ved alle "høj"/"kritisk" CA-incidents inden 4 timer; AO koordinerer kunde-kommunikation under Mads' og Dorthes sikkerheds-/GDPR-direktiver. AO må IKKE selv kontakte regulators (Datatilsynet) — det er Dorthe + FGD
5. **PKI-specifik anti-pattern**: AO må ikke acceptere kunde-kontrakts-forpligtelser om public-trust-status uden eksplicit FGD-beslutning om CT-logging og WebTrust-audit
6. **Re-keying-event-koordinering**: Når FIPS 140-3-cert lander og re-keying-event planlægges, koordinerer AO kunde-kommunikation og overlap-perioder; AO ejer ikke ceremoni-eksekvering (det er Mads/KCO/HSM-op)

**Hvor template'en allerede dækker behovet:**

- Standard SOD AO × AP — gælder også for PKI; ingen ændring
- Standard compliance-grænser (må ikke selv erklære compliant) — fungerer perfekt for PKI
- Kvartalsvis ROPA-attest — gælder også, blot med PKI-data-tilføjet

### 6.2 PKI-specifikke deltas til AP-mandatet for FGDCorePKI

**Tilføjelser til mandat-body:**

1. **PKI-domæne-specifik teknisk dybde-krav**: AP for FGDCorePKI skal have specialist-niveau kompetencer i (per §5.2): X.509, PKCS#11, ACME/EST/SCEP, NIST SP 800-57, RFC 3647 CP/CPS-struktur, three-tier CA-arkitektur. Ikke bare "kender til" — kan designe og argumentere
2. **PKI-specifik SOD (hard rule)**:
   - AP må IKKE være CA Administrator (drifts-rolle)
   - AP må IKKE være HSM-operatør (specificerer arkitektur; eksekverer ikke)
   - AP må IKKE deltage som autoriserende part i key ceremony (KCO + SO ejer)
   - AP må IKKE være PKI Tech Lead i samme person uden formel undtagelse (best practice separat)
   - AP har ingen direkte CA-software-runtime-adgang i prod
3. **CP/CPS-udkast-pligt**: AP er primær teknisk udkast-forfatter af CP/CPS §6 (technical security controls); leverer udkast til Mads (CA Manager) for governance-review; Trine (PKI Auditor) reviewer inden FGD-godkendelse
4. **HSM-bridge-arkitektur-ejerskab**: AP designer HSM-bridge-arkitektur (Tailscale tail-net, mTLS, capability-mapping per rolle); AP eksekverer ikke YubiHSM2-konfiguration (HSM-operatør med Mads-godkendelse)
5. **Network zone-model A-F-arkitektur-ejerskab**: AP designer zone-model (per Pouls 10-vinkler-vinkel 6: Zone A air-gap → Zone F audit/SIEM); SRE implementerer; Mads godkender
6. **Algorithm- og crypto-policy-ejerskab**: AP foreslår algorithm-policy (RSA-4096 vs. ECDSA P-384, SHA-384/512); Mads godkender; CP/CPS dokumenterer; AP designer crypto-agility-plan (PQC-roadmap mod ML-DSA inden 2032-2035)
7. **Re-keying-arkitektur-design**: Når FIPS 140-3-cert lander, designer AP re-keying-event (key generation på FIPS-validerede HSMs, certifikat-overlap-strategi, klient-rolling-update); KCO + HSM-op + Mads eksekverer ceremonien
8. **HSM-throughput-test og sizing**: AP designer load-test-plan; SRE eksekverer; AP justerer arkitektur baseret på resultater (jfr. Pouls 10-vinkler-vinkel 9: YubiHSM2 ~7 RSA-4096-sign/sek per HSM)
9. **PKI-specifik anti-pattern**: AP må ikke deploye CA-software-opdatering selv (CA Admin + Mads-dual-control); AP må ikke ændre CRL/OCSP-policy direkte (RFC 5280 §5 + dual-control)

**Hvor template'en allerede dækker behovet:**

- Code-review-gate (Mads + commit-reviewer) — fungerer perfekt for PKI
- Break-glass-procedure for prod-data-adgang — gælder også for CA-audit-data (hvis AP debugger live audit-events)
- Dataflow-notifikations-pligt til Dorthe — gælder også, blot med PKI-data (CSR-data, audit-events der indeholder Subject-felter)
- Standard SOD AP × AO — gælder også
- HSM-rule "tilgår aldrig YubiHSM direkte" i template — er allerede bygget korrekt for PKI

### 6.3 Praktiske fil-anbefalinger til Laila

Laila skal lave to nye filer (eller en model der kombinerer begge tilgange):

**Option 1 — Project-specifikke fulde mandater:**
- `.claude/agents/application-owner-fgdcorepki.md` (full mandate, men uden self-modification af template-baseret common-tekst)
- `.claude/agents/application-provider-fgdcorepki.md`

**Option 2 — Project-specifikke tillæg-dokumenter:**
- `Team/projects/FGDCorePKI/application-owner-tillaeg.md` (refererer til template + lister deltas)
- `Team/projects/FGDCorePKI/application-provider-tillaeg.md`

**Pouls anbefaling**: Option 1 (fulde mandater) for at undgå runtime-tvivl om hvilke regler der gælder. Camilla committer; Vibeke (commit-reviewer, når aktiv) reviewer; Mads + Dorthe godkender per FGD-override 2026-05-06; FGD giver final GO.

### 6.4 Andre PKI-specifikke roller Laila også skal designe

Udover AO og AP for FGDCorePKI har Mads' krav-dok §6.1 angivet (gentaget her for fuldstændighed):

1. **CA Administrator (FGDCorePKI)** — ny project-rolle (Prioritet 1)
2. **HSM-operatør (FGDCorePKI tillæg)** — separat fra Bjarne per Mads' anbefaling
3. **Key Ceremony Officer (FGDCorePKI)** — i fase 0 varetaget af Mads, men rollen designes med separation in mind
4. **Registration Authority (FGDCorePKI)** — ny project-rolle
5. **Security Officer (FGDCorePKI)** — i fase 0 varetaget af Mads, separat på sigt
6. **Trine PKI Auditor-udvidelse** — opdatering af eksisterende Trine-mandat

Engineering-roller (Tech Lead, Senior Engineers, SRE) kan designes som generiske project-roller eller som ad-hoc beskrivelser i `Team/projects/FGDCorePKI/engineering-team.md` afhængigt af om de skal have AI-agent-mandater eller blot er menneske-roller. Pouls anbefaling: ad-hoc beskrivelser i project-folder, ikke fulde Claude-agents (engineering-arbejdet er praktisk udvikling, ikke AI-agent-koordineret).

---

## 7. Risici og åbne afklaringer

### 7.1 Risici med fase 0-bemanding < 8 personer

| Risiko | Konsekvens | Mitigation |
|---|---|---|
| Mads = CA Manager + Key Ceremony Officer | Policy-ejer er også ceremoni-script-ejer; mild SOD-issue | Trine kvartalsvis attestation; separation senest ved 1. eksterne kunde |
| Mads = CA Manager + Security Officer | CA Manager review egne beslutninger via SO-rolle | Maksimal 6-måneders periode; Trine attesterer; FGD-deadline |
| Engineering Team = 2 personer (Tech Lead + Senior Engineer) i stedet for 3+ | Ingen vagt-dækning for non-crypto-incidents; code-review-SOD svækket | Tilføj SRE inden go-live for eksterne kunder |
| Bjarne ikke separat fra FGDCorePKI-HSM-operatør | Cross-application blast-radius hvis kompromittering | Mads' eksplicit anbefaling: separat tillæg; FGD bekræfter |

### 7.2 Åbne afklaringer til FGD (udover de 3 trufne 2026-05-09-beslutninger)

Disse afklaringer er ikke truffet endnu og påvirker org-strukturen:

1. **Public-trust eller privat-only PKI?** Påvirker WebTrust-audit-firma-behov, CT-logging-pligt, leaf-validity-trajektorie (CABF SC-081v3). Hvis public-trust: ekstra rolle-behov for ekstern WebTrust-akkrediteret revisor (ikke Trine alene)
2. **Bjarne udvidet mandat eller separat HSM-operatør for FGDCorePKI?** Mads' anbefaling: separat. Påvirker bemanding +1 person hvis separat
3. **Application Owner for FGDCorePKI = FGD selv eller delegeret?** Hvis FGD selv: AO-rollen er light-weight orkestrering; hvis delegeret: full project-rolle med kvartalsvis attestation
4. **Re-keying-event-tidspunkt**: når FIPS 140-3-cert lander (uvist tidspunkt), skal KCO være separat person inden første re-keying — afhænger af Yubico's CMVP-status
5. **CP/CPS-publicerings-grad**: intern eller publiceret? Påvirker Trines audit-scope og Dorthe's review af PII-eksponering i CP/CPS

### 7.3 Implikationer af FGDs 2026-05-09-beslutninger for org-struktur

| Beslutning | Org-impact |
|---|---|
| **Non-FIPS MVP nu med re-keying-plan** | KCO-rollen får eksplicit re-keying-event-ansvar; AP designer re-keying-arkitektur fra dag 1; Mads dokumenterer formel risk-acceptance med plan |
| **Tier 2 HA-pair (~€2.400 ekstra)** | HSM-operatør får udvidet ansvar for HA-sync (wrap-key replication mellem 2× YubiHSM2); SRE får monitoring-ansvar for HA-failover; Mads + AP designer HA-procedure |
| **Root CA validity 10-12 år (PQC-headroom)** | AP får eksplicit ansvar for crypto-agility-plan i CP/CPS; ML-DSA-migration-roadmap-ejer er AP (teknisk) + Mads (compliance); Engineering Team monitorerer Yubico's PQC-firmware-roadmap |

---

## 8. Konklusion og næste skridt

### 8.1 Konklusion

FGDCorePKI's organisations-struktur kan ikke under-bemandes uden at bryde fundamentale SOD-krav. **Fase 0 minimum: 9-10 personer** (5-6 PKI-roller + 2 engineering minimum + AO + AP). Fire SOD-kombinationer er absolutte og kan aldrig opblødes; fire andre kombinationer kan accepteres midlertidigt med dokumenterede tidsbegrænsede undtagelser og normaliserings-deadlines.

FGDs tre 2026-05-09-beslutninger (Non-FIPS MVP, HA-pair, 10-12 års Root) er korrekt indarbejdet i org-strukturen og påvirker især Key Ceremony Officer-rollen (re-keying-event), HSM-operatør-rollen (HA-sync) og Application Provider-rollen (crypto-agility-design).

AO og AP-mandaterne fra Laila's eksisterende template fungerer korrekt for PKI; PKI-specifikke deltas er moderate og fokuserer på at gøre SOD-kravene eksplicit hardrules (ikke kun anti-patterns), tilføje teknisk dybde-krav for AP, og indarbejde re-keying-event-koordinerings-ansvar.

### 8.2 Konkrete næste skridt

1. **Stefan formidler denne rapport** til Laila + Mads + Dorthe parallelt
2. **Mads peer-reviewer** org-struktur, SOD-matrix-udvidelse og fase 0-undtagelses-listen
3. **Dorthe vurderer** GDPR-impact af AO/AP-PKI-deltas (dataflow-notifikationer for CSR-data, audit-events med Subject-felter)
4. **Laila designer** PKI-specifikke AO/AP-instans-mandater + 5 nye PKI-rolle-mandater (CA Admin, HSM-op, KCO, RA, Security Officer) + Trine PKI Auditor-udvidelse
5. **FGD beslutter** de 5 åbne afklaringer i §7.2
6. **Camilla committer** godkendte mandater (efter Mads + Dorthe-vurderinger og FGD-GO per FGD-override 2026-05-06)
7. **Trine logger** alle ny-rolle-aktiveringer i audit-trail

---

## Kilder og referencer

**Primære input-dokumenter:**
- Team/Mads/2026-05-09-pki-org-krav.md — Mads' krav-dokument
- Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md — Pouls 10-vinkler-analyse
- .claude/agents/application-owner.md — eksisterende AO-template
- .claude/agents/application-provider.md — eksisterende AP-template
- governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md — change-management-process
- CLAUDE.md — Stefans orkestrerings-mandat og governance-flow

**Standarder:**
- RFC 3647 — Internet X.509 PKI Certificate Policy and CPS Framework
- RFC 5280 — Internet X.509 PKI Certificate and CRL Profile
- RFC 7633 — TLS Feature Extension (must-staple)
- RFC 8555 — ACME
- RFC 7030 — EST
- RFC 8894 — SCEP
- NIST SP 800-57 Part 1 Rev 5 — Key Management
- NIST SP 800-131A Rev 2 (Final) — Cryptographic Algorithm Transitions
- FIPS 204 — ML-DSA (Module-Lattice-Based Digital Signature, finaliseret aug 2024)
- ISO/IEC 27001:2022 — A.5.2 (CISO), A.5.3 (Segregation of Duties), A.8.32 (Change Management)
- AICPA SOC 2 Type II Trust Services Criteria — CC4.1 (Audit independence), CC6.1 (Logical access), CC6.3 (Access removal), CC7.2 (Change management)
- GDPR art. 30 (ROPA), art. 33 (Breach notification), art. 35 (DPIA), art. 38(3) (DPO independence)
- WebTrust for Certification Authorities (CPA Canada/AICPA)

---

*Dokument-ejer: Poul (analytiker)*
*Distribueret til: Stefan (koordinering), Laila (rolle-design), Mads (peer-review), Dorthe (GDPR-vurdering), FGD (godkendelse)*
*Næste revision: Efter Lailas mandat-design og Mads + Dorthes vurderinger*
*Kopi: governance/design-docs/ (compliance-transparens for Trine + Dorthe)*
