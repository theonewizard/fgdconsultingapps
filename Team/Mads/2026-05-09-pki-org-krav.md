---
dato: 2026-05-09
forfatter: Mads (CISO)
emne: PKI-organisationskrav — FGDCorePKI (Three-Tier CA Hierarchy)
bestilt-af: Stefan (på vegne af FGD)
status: UDKAST — input til Laila og Poul
relateret-arkitektur: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
relateret-fips: Team/Poul/2026-05-05-secrets-fips-analyse.md
relateret-hsm: Team/Poul/2026-05-05-yubihsm-integration-analyse.md
relateret-sod: governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md
relateret-ao-vurdering: Team/Mads/2026-05-08-mads-vurdering-app-owner-provider.md
---

# PKI-organisationskrav — FGDCorePKI

**Udstedt af:** Mads (CISO)
**Dato:** 2026-05-09
**Scope:** Three-Tier CA Hierarchy, on-premises deployment, YubiHSM2
**Primære modtagere:** Laila (rolle-design), Poul (arkitektur-research)
**Sekundære modtagere:** Stefan (koordinering), FGD (godkendelse)

---

## Compliance-note om FIPS 140-2 vs. FIPS 140-3 (VIGTIGT — læs forst)

Briefen beskriver YubiHSM2 som "FIPS 140-2 Level 3 valideret". Dette er terminologisk forældet og kræver eksplicit stillingtagen:

- CMVP accepterede den 1. april 2022 ikke længere nye FIPS 140-2-valideringer
- Alle eksisterende FIPS 140-2-certificeringer går på "Historical List" den 21. september 2026 — 4 måneder herfra
- YubiHSM2 FIPS-variant certificeres til FIPS 140-3 (forventet Yubico Q2 2026)

**Mads' anbefaling:** FGDCorePKI skal designes til FIPS 140-3 Level 3 fra dag 1. At bygge en ny PKI-infrastruktur på en standard der udløber i 4 måneder er ikke forsvarligt — hverken teknisk, compliance-mæssigt eller kommercielt. YubiHSM2 FIPS-variant understøtter FIPS 140-3; det er dette dokuments reference.

**Handling påkrævet af FGD:** Bekræft at FIPS 140-3 Level 3 er det operative krav for FGDCorePKI, og at "FIPS 140-2" i briefs behandles som overgangsterminologi. Alternativt: FGD angiver en forretningsbegrundelse for 140-2 (fx eksisterende kundekrav), hvorefter Mads dokumenterer risikoen formelt og eskalerer til Stefan.

---

## 1. PKI-governance-roller

Nedenstående roller er nødvendige for compliant drift af en three-tier CA-hierarki. For hver rolle angives: ansvar, privilege-level, SOD-krav, FIPS 140-3-relevans og mapping til eksisterende FGD-roller.

### 1.1 CA Manager (overordnet PKI-governance)

| Felt | Indhold |
|---|---|
| **Ansvar** | Overordnet ansvar for PKI-programmet: policy-ejerskab (CP/CPS), budgetgodkendelse, leverandørrelationer, eskalationslinje ved CA-incidents, godkendelse af CA-rotation og re-keying |
| **Privilege-level** | Governance-autoritet; ingen direkte operationel adgang til CA-systemer eller HSM |
| **SOD-krav** | Må IKKE varetage CA Administrator, HSM-operatør eller PKI Auditor. Skal rapportere til FGD direkte — ikke til Application Owner |
| **FIPS 140-3-relevans** | Indirekte: godkender policy der specificerer FIPS 140-3-krav til hardware og algoritmer |
| **Mapping til eksisterende FGD-rolle** | Mads (CISO) overtager CA Manager-funktionen. Ingen ny rolle nødvendig — dette er Mads' governance-mandat i praksis |

### 1.2 CA Administrator

| Felt | Indhold |
|---|---|
| **Ansvar** | Day-to-day drift af CA-infrastruktur: certificate issuance, policy enforcement, CRL-generering, OCSP-responder, Intermediate CA-konfiguration, Issuing CA-opstart og nedlukning |
| **Privilege-level** | Fuld administrativ adgang til CA-software og Tier 2/Tier 3-systemer. Ingen direkte Root CA-adgang (kræver Key Ceremony) |
| **SOD-krav** | Må IKKE kombineres med: PKI Auditor, Registration Authority (RA), HSM-operatør, Application Owner for FGDCorePKI. Dual-control kræves ved Root CA-operationer |
| **FIPS 140-3-relevans** | Direkte: CA Administrator er ansvarlig for at CA-software konfigureres til kun at bruge FIPS-godkendte algoritmer (RSA-4096 eller ECDSA P-384 for CA-nøgler; SHA-384/SHA-512 som minimum) |
| **Mapping til eksisterende FGD-rolle** | Ny dedikeret project-rolle under FGDCorePKI. Laila skal designe rollen |

### 1.3 HSM-operatør

| Felt | Indhold |
|---|---|
| **Ansvar** | Håndtering af YubiHSM2-enheder: initialisering, key generation, wrap-key management, backup (encrypted key export), firmware-opdatering, session-authentication, DR-test. Eksekverer HSM-operationer efter Mads' godkendelse |
| **Privilege-level** | Direkte fysisk og logisk adgang til YubiHSM2. Admin-password + authentication key. Adgang til offline backup-media |
| **SOD-krav** | Må IKKE kombineres med: CA Administrator, PKI Auditor, Key Ceremony Officer (under selve ceremonien). Alle destruktive operationer kræver Mads + FGD-godkendelse skriftligt inden eksekvering |
| **FIPS 140-3-relevans** | Kritisk: HSM-operatøren er den person der fysisk og logisk garanterer at nøglematerialet aldrig forlader FIPS 140-3-valideret hardware i ukrypteret form |
| **Mapping til eksisterende FGD-rolle** | Bjarne er defineret som HSM-operatør for Knowlagecentral. FGDCorePKI kræver et separat mandat (begrundelse: PKI-nøgler er trust anchor for hele infrastrukturen; cross-application blast radius). Vurdering: Bjarne KAN udvide mandatet til FGDCorePKI, men Laila skal designe separat FGDCorePKI-HSM-operatør-tillæg med eksplicit SOD-afklaring. Mads godkender inden aktivering |

### 1.4 Key Ceremony Officer

| Felt | Indhold |
|---|---|
| **Ansvar** | Formel autoritet ved Root CA key ceremony og re-keying: script-ejer, tidslog, command execution authorization (ikke eksekvering — det er HSM-operatøren). Koordinerer m-of-n smartcard holders, kaller witnesses, godkender ceremony-afholdelse |
| **Privilege-level** | Funktionel autoritet; ingen direkte HSM-adgang under ceremonien (SOD-krav) |
| **SOD-krav** | Må IKKE varetage HSM-operatør-rollen under samme ceremoni. Kan ikke være witness for sin egen ceremoni. Bør ikke være CA Administrator |
| **FIPS 140-3-relevans** | Sikrer at Root CA key generation følger NIST SP 800-57 Part 1 og sker på FIPS 140-3-valideret hardware uden eksponering af key material |
| **Mapping til eksisterende FGD-rolle** | Ny dedikeret project-rolle. Kan varetages af CA Manager (Mads) i en startup-fase, men bør på sigt adskilles. Laila designer rolle |

### 1.5 Registration Authority (RA)

| Felt | Indhold |
|---|---|
| **Ansvar** | Godkendelse af certificate signing requests (CSRs) før de videresendes til Issuing CA. Identitetsverifikation af rekvirenter, policy-tjek (algoritme-krav, navne-konventioner, validity period), approve/reject med begrundelse i audit-log |
| **Privilege-level** | Adgang til RA-portal og CSR-kø. Ingen direkte CA-software-adgang. Ingen HSM-adgang |
| **SOD-krav** | Må IKKE kombineres med CA Administrator (dette er det klassiske "mark and approve your own work"-problem). Bør have uafhængig rapporteringslinje fra CA Administrator |
| **FIPS 140-3-relevans** | Indirekte: RA er første kontrolpunkt for at sikre at kun FIPS-konform nøglemateriale (algoritmetype, nøgellængde) godkendes til signering |
| **Mapping til eksisterende FGD-rolle** | Ny dedikeret project-rolle. For lavt trafik-volume: kan kombineres med Security Officer, men ikke CA Administrator. Laila designer rolle |

### 1.6 PKI Auditor

| Felt | Indhold |
|---|---|
| **Ansvar** | Uafhængig audit af CA-drift: review af certificate issuance logs, HSM audit logs, key ceremony protokoller, CP/CPS-efterlevelse, adgangskontrol-review, rapportering til CA Manager og FGD |
| **Privilege-level** | Read-only adgang til audit logs og CA-rapporter. Ingen operationel CA-adgang. Ingen HSM-adgang |
| **SOD-krav** | Komplet operationel separation fra alle øvrige PKI-roller. Rapporterer direkte til FGD — ikke via CA Manager. Kan ikke have været CA Administrator inden for de seneste 12 måneder |
| **FIPS 140-3-relevans** | Verificerer at FIPS 140-3-krav er efterlevet i alle operationer: korrekte algoritmer brugt, nøglemateriale aldrig eksporteret ukrypteret, session logs intakte |
| **Mapping til eksisterende FGD-rolle** | Trine (compliance-auditor) overtager PKI Auditor-funktionen. Triners eksisterende mandat dækker ISO 27001:2022 og SOC 2 Type II audit — PKI-audit er en natural extension. Mads koordinerer med Trine om kompetence-gap (PKI-specifik audit-metodologi) |

### 1.7 Security Officer (dual-control)

| Felt | Indhold |
|---|---|
| **Ansvar** | Second authorizer ved kritiske PKI-operationer: Root CA-aktivering, CA-revokering, HSM factory reset, wrap-key rotation. Fungerer som "second pair of eyes" i to-mands-princippet. Signerer ceremony-protokol |
| **Privilege-level** | Authorizations-rettighed til at godkende kritiske operationer; ingen selvstændig eksekvering. Adgang til Security Officer smartcard (én af m i m-of-n scheme) |
| **SOD-krav** | Må IKKE kombineres med HSM-operatøren ved samme operation. Skal være funktionelt uafhængig af CA Administrator |
| **FIPS 140-3-relevans** | Sikrer at to-mands-princippet er operationaliseret for de mest kritiske FIPS 140-3-relaterede operationer |
| **Mapping til eksisterende FGD-rolle** | I en startup-fase: kan varetages af Mads (som CA Manager). På sigt: separat rolle for at undgå CA Manager + Security Officer kombination. Laila vurderer timing |

---

## 2. Engineering Team (Application Development)

Engineering-teamet til FGDCorePKI er ansvarlig for at bygge og vedligeholde de integrationsflader der giver andre applikationer adgang til PKI-infrastrukturen. Teamet opererer i lag 3-4 (integration + tooling) — de rører ikke Root CA eller Intermediate CA direkte.

### 2.1 Minimums-besætning

Minimum 3 personer for compliant drift uden single-person-dependency:

| Rolle | Antal | Kernekompetencer |
|---|---|---|
| **PKI Tech Lead / Architect** | 1 | PKI-arkitektur, X.509 standards, PKCS#11, RFC 3647, ACME/SCEP/EST-protokoller, HSM-bridge design, threat modeling |
| **Senior PKI Engineer** | 1–2 | HSM-bridge implementering (Go/Rust foretrukket), ACME-klient/server, PKCS#12/PKCS#11-integration, certificate lifecycle automation, API-design (REST + OpenAPI 3.1) |
| **SRE / Platform Engineer** | 1 | Cert-lifecycle monitoring, CRL/OCSP health checks, alerting, rotation automation, on-prem infra (Tailscale, service mesh, backup) |

Under 3 personer: ingen vagt-dækning, ingen SOD på code-review, ingen vakant-håndtering ved sygdom. Det er ikke acceptabelt for en trust anchor-infrastruktur.

Anbefalede kompetencer på holdet samlet:

- RFC 3647 (CP/CPS-struktur), RFC 5280 (X.509), RFC 8555 (ACME), RFC 8894 (SCEP), RFC 7030 (EST)
- PKCS#11 v2.40+ og YubiHSM2 KSP/PKCS#11 provider
- NIST SP 800-57 Part 1 (key management) og SP 800-52 Rev 2 (TLS)
- Secure code review (OWASP, SAST/DAST tooling)
- PKI-specifik threat modeling (STRIDE mod CA-angreb: rogue certificates, OCSP spoofing, key compromise)

### 2.2 Engineering team SOD-krav

Engineering-teamet må **ikke** have adgang til produktion CA-operationer under normal drift. Specifikt:

- Ingen direkte adgang til Root CA eller YubiHSM2 production keys
- Code changes til PKI-integrationer kræver security-review (Mads/security-reviewer godkender pipeline-gates)
- Deploy til produktion kræver CA Administrator sign-off (ikke Engineering Lead)

Poul skal researche: hvilke ACME/EST/SCEP-stacks og cert-lifecycle-management-tools der understøtter YubiHSM2 PKCS#11. Dette er Pouls research-track — Engineering-teamet implementerer ikke blindt, men baserer sig på Pouls analyse.

---

## 3. Processer der skal implementeres

### 3.1 Key Ceremony procedure (Root CA)

Root CA key ceremony er den mest kritiske operation i hele PKI-infrastrukturen. Fejl her kan ikke rettes uden komplet CA-genbygning.

**Krav til ceremony:**

- **m-of-n smartcard scheme:** 3-of-5 for Root CA key activation. Smartcards distribueres til 5 separate key holders (Key Ceremony Officer + 4 secondary holders). Ingen enkeltperson kan aktivere Root CA alene
- **Intermediate CA:** 2-of-3 scheme (less kritisk, men dual-control obligatorisk)
- **Witnesses:** Minimum 2 uafhængige witnesses (ikke CA-operatører). Witnesses signerer ceremony-protokol
- **Ceremony script:** Fuldt udskrevet step-by-step script godkendt af Mads inden ceremony. Ingen ad-hoc-beslutninger under ceremony
- **Video/audio recording:** Hele ceremony optages; optagelse gemmes krypteret med WORM-garanti i minimum 20 år
- **Audit log:** Signeret audit log med timestamp pr. trin; log signeres af Key Ceremony Officer + alle witnesses efter ceremony
- **Offline environment:** Root CA operationer foregår på air-gapped maskine; YubiHSM2 er den eneste hardware der er online
- **Scripted verification:** Umiddelbart efter key generation: verify public key, verify HSM object ID, verify algorithm (ECDSA P-384 eller RSA 4096 minimum)

**NIST-reference:** NIST SP 800-57 Part 1 Rev 5 §5.6 (Key Management Lifecycle)

### 3.2 Certificate issuance flow

```
Rekvirent (applikation)
    → genererer CSR lokalt (private key forlader ALDRIG rekvirent-miljø)
    → indsender CSR via RA-portal (ACME/SCEP/EST eller manuel API)

RA (Registration Authority)
    → verificerer rekvirent-identitet (API-authentication, organization membership)
    → validerer CSR: algoritme, nøglelængde, Subject-felter, SAN
    → approve → vidersend til Issuing CA queue
    → reject → begrundelse til rekvirent i audit-log

Issuing CA (Tier 3)
    → modtager godkendt CSR fra RA
    → signerer med Issuing CA private key (på YubiHSM2)
    → genererer certifikat med validity ≤ 1 år (leaf certificates)
    → leverer certifikat til rekvirent + logger issuance i audit trail

Rekvirent
    → installerer certifikat
    → ansvarlig for renewal inden expiry (ACME autorenew anbefalet)
```

**Validity periods (fastsættes i CP/CPS):**

| CA-niveau | Max validity |
|---|---|
| Root CA | 15–20 år |
| Intermediate CA (Tier 2) | 10 år |
| Issuing CA (Tier 3) | 5 år |
| Leaf certificate | Maksimum 1 år (branchekrav) |

### 3.3 Certificate revocation procedure (CRL + OCSP)

- **CRL:** Udstedes af Issuing CA minimum én gang dagligt; signes af Issuing CA private key; publiceres på internt tilgængeligt distribution point (HTTP, ikke HTTPS — tilgængelighed er kritisk)
- **OCSP:** OCSP-responder kørende 24/7; respons-signering med delegated OCSP-signing key (separat fra Issuing CA key); max response-alder 4 timer
- **Emergency revocation:** Procedure for kompromitteret certifikat: RA modtager anmodning → CA Administrator godkender revokering → CRL genereres inden for 1 time → OCSP opdateres
- **Root/ICA revocation:** Kræver Key Ceremony (m-of-n aktivering) + Mads-godkendelse + FGD-godkendelse + PKI Auditor-notifikation

### 3.4 HSM backup og DR-procedure

**Backup-princip:** YubiHSM2 tillader encrypted key export (wrap-key scheme). Private keys forlader aldrig HSM ukrypteret.

- **Wrap-key:** Separat AES-256 wrap-key genereret under initial setup; opbevares i offline encrypted storage (minimum 2 geografisk adskilte lokationer)
- **Backup-frequency:** Ved hver key ceremony, ved CA-rotation, og månedligt for operational keys
- **Backup-test:** Kvartalvis restore-test til identisk YubiHSM2-enhed i isoleret testmiljø; HSM-operatør eksekverer; PKI Auditor observerer
- **DR-procedure:**
  1. Incident erkendt → CA Administrator + HSM-operatør notificeres
  2. Replacement YubiHSM2 hentes fra secure storage
  3. Wrap-key hentes fra offline backup (2-person-regel)
  4. Keys importeres til replacement HSM
  5. CA-software rekonfigureres mod ny HSM
  6. Verify: OCSP og CRL fungerer
  7. PKI Auditor attesterer restore-completion

**HA-observation:** Briefen specificerer "shared YubiHSM2" mellem 2 Intermediate CA-servere i Tier 2. Dette er en single point of failure. **Mads' anbefaling:** Anskaf HA-pair (2 × YubiHSM2 for Tier 2/3) med synkroniserede keys via wrap-export. Alternativt: dokumentér eksplicit accepteret risiko med RTO/RPO (forventet: RTO 4–8 timer ved HSM-svigt, afhænger af backup-procedure). FGD skal træffe dette valg inden arkitektur-godkendelse.

### 3.5 CA rotation og re-keying procedure

- **Planlagt rotation:** Initieres minimum 12 måneder inden CA-certifikatudløb
- **Root CA re-keying:** Full key ceremony (§3.1); ny Root CA-nøgle genereres; gammel Root CA forbliver aktiv i overlap-periode (typisk 2 år) til eksisterende certifikater udløber
- **Intermediate/Issuing CA rotation:** Dual-control; ny nøgle genereres på HSM; ny CA-certifikat signeres af overliggende CA; overlap-periode 6 måneder
- **Notification:** Rekvirenter notificeres minimum 90 dage inden CA-udløb

### 3.6 Incident response ved CA-kompromittering

| Alvorlighedsgrad | Trigger | Response |
|---|---|---|
| **Kritisk (Root CA kompromitteret)** | Mistanke om Root CA key-eksponering | Umiddelbar shutdown af alle CA-systemer. Mads + FGD notificeres inden for 1 time. Key Ceremony for ny Root CA inden 72 timer. Alle udstedte certifikater potentielt ugyldiggjort |
| **Høj (Intermediate/Issuing CA kompromitteret)** | Mistanke om ICA-key-eksponering | ICA tages offline inden for 1 time. Revokering af ICA-certifikat. Replacement ICA key generation (dual-control). Alle blade certifikater udstedt af kompromitteret ICA revokeres |
| **Medium (RA-kompromittering)** | Uautoriseret adgang til RA-portal | RA-portal lukkes. Audit-review af alle approvals inden for 30 dage. Revokering af eventuelle rogue-godkendede certifikater |
| **Lav (individual certifikat)** | Rekvirent rapporterer nøglekompromittering | Emergency revocation (§3.3) inden for 1 time |

GDPR-relevans: Hvis kompromitterede certifikater er knyttet til persondata-flows, notificerer Dorthe (DPO) relevante myndigheder inden for 72 timer (GDPR art. 33).

### 3.7 Access review (kvartalvis SOD-verificering)

- **Frekvens:** Kvartalvis (Q1/Q2/Q3/Q4)
- **Ansvarlig:** PKI Auditor (Trine) gennemfører; CA Manager (Mads) godkender rapport
- **Scope:** Alle PKI-rolle-assignments; HSM authentication-key holders; smartcard holders (m-of-n); RA-portal adgange; audit log read-access
- **Output:** Signeret access-review-rapport; eventuelle afvigelser eskaleres til Mads inden for 5 arbejdsdage
- **SOC 2-reference:** CC6.3 (Access removal upon role change)

### 3.8 Change management for PKI-infrastruktur

PKI-infrastruktur er en trust anchor — ændringer er højrisiko.

- **Alle ændringer til CA-software, HSM-konfiguration eller CP/CPS:** Kræver change request → CA Manager (Mads) review → PKI Auditor notifikation → godkendelse inden implementering
- **Maintenance window:** Planned changes gennemføres i forud-annoncerede vedligeholdelsesvinduer
- **Emergency changes:** Godkendes af CA Manager + Security Officer; dokumenteres i audit-log inden for 2 timer
- **Reference:** SOC 2 CC7.2 (Change Management)

---

## 4. Compliance-krav

### 4.1 FIPS 140-3 Level 3 (YubiHSM2)

YubiHSM2 FIPS-variant er reference-hardware. Specifikke krav:

| Krav | Detalje |
|---|---|
| **Key storage** | Alle private CA-nøgler genereres og opbevares udelukkende på YubiHSM2. Ingen software-key-copies i produktion |
| **Authenticated sessions** | YubiHSM2 session-authentication kræver authentication key (password) + optionally asymmetric session (ECDHP256). Alle sessioner logges af HSM |
| **Algoritmer (FIPS-approved only)** | RSA ≥ 4096 bit eller ECDSA P-384/P-521 for CA-nøgler; SHA-384/SHA-512 som minimum for signering; AES-256-CCM for wrap operations |
| **Audit log** | YubiHSM2 hardware audit log er immutable og HSM-intern; exporteres til eksternt WORM-system ved regelmæssige intervaller |
| **Physical security** | YubiHSM2-enheder opbevares i låst rack eller safe (on-premises); access log til fysisk lokation |
| **Firmware integrity** | Firmware-opdateringer verificeres mod Yubico-signatur inden installation |

### 4.2 ISO 27001:2022

| Control | Applicability |
|---|---|
| **A.10.1 / A.8.24** (Cryptographic controls / Use of cryptography) | CP/CPS skal dokumentere godkendte algoritmer, nøgellængder og validity periods; cryptographic policy reviewed årligt |
| **A.5.3** (Segregation of duties) | SOD-matrix (§5) er direkte implementering af A.5.3 for PKI |
| **A.8.18** (Use of privileged utility programs) | HSM management tools (yubihsm-setup, yubihsm-shell) er privileged utilities; adgang begrænses til HSM-operatør |
| **A.8.20** (Networks security) | Tier 2/3 CA-servere på isoleret netværkssegment; Tailscale VPN med ACL-kontrol |
| **A.5.33** (Protection of records) | Audit logs: WORM, minimum 7 år, kryptografisk signerede |
| **A.8.32** (Change management) | §3.8 ovenfor implementerer dette control |

### 4.3 SOC 2 Type II

| Control | Implementation |
|---|---|
| **CC6.1** (Logical access security) | m-of-n smartcard scheme; role-based access til CA-systemer; ingen shared passwords |
| **CC6.3** (Access removal) | Kvartalvis access review (§3.7); off-boarding procedure ved rolle-ændring |
| **CC7.2** (Change management) | §3.8; alle changes dokumenteret og godkendt |
| **CC9.2** (Risk mitigation) | DR-procedure (§3.4); HA-vurdering for HSM (§3.4 observation) |
| **A1.2** (Availability) | CRL/OCSP tilgængelighed 24/7; monitoring via SRE (Engineering team) |

### 4.4 Certificate Policy (CP) og Certification Practice Statement (CPS)

CP og CPS er obligatoriske dokumenter for enhver CA-drift. De er ikke valgfri dokumentation — de er juridisk og compliance-mæssigt forpligtende rammer.

**Template:** CP/CPS skal følge **RFC 3647** (Internet X.509 PKI Certificate Policy and Certification Practices Framework). Strukturen i RFC 3647 §6 er mandatorisk at følge.

**Minimums-indhold (RFC 3647-aligned):**

1. Introduction (OID, community, applicability)
2. Publication and Repository Responsibilities
3. Identification and Authentication
4. Certificate Life-Cycle Operational Requirements
5. Facility, Management and Operational Controls
6. Technical Security Controls (algoritmer, nøgellængder, YubiHSM2-krav)
7. Certificate and CRL Profiles
8. Compliance Audit and Other Assessment
9. Other Business and Legal Matters

**Ansvar:** CA Manager (Mads) godkender CP/CPS; Engineering team (PKI Architect) udkaster det tekniske indhold; PKI Auditor (Trine) reviewer inden godkendelse.

### 4.5 Audit trail-krav

| Krav | Specifikation |
|---|---|
| **Retention** | Minimum 7 år (PKI-branchestandard; matcher ISO 27001 A.5.33 og SOC 2 krav) |
| **Immutabilitet** | WORM (Write Once Read Many) storage eller kryptografisk chained log (hash-kæde pr. log-entry) |
| **Signering** | Log-entries signeres med dedikeret log-signing key (separat fra CA-nøgler) |
| **Scope** | Alle certificate issuance-events, revocations, HSM-operationer (med operator-ID), CA-system logins, access reviews, ceremony-protokoller, RA approve/reject-beslutninger |
| **Adskillelse** | PKI-operatører (CA Administrator, HSM-operatør) har ingen skriveadgang til audit-logs. Logs skrives af systemet; læseadgang kun for PKI Auditor og CA Manager |
| **OpenTelemetry** | Strukturerede logs eksporteres til OpenTelemetry-pipeline (i tråd med CLAUDE.md §1); traces korreleres via W3C Trace Context |

---

## 5. SOD-matrix

Matrixen viser: **X** = må ikke kombineres i én person, **D** = kræver dual-control (to-mands-princip), **OK** = tilladt kombination, **—** = ikke relevant.

| | CA Mgr (Mads) | CA Admin | HSM-op | Key Cer. Off. | RA | PKI Auditor (Trine) | Security Off. | AO (FGDCorePKI) | Engineering Lead |
|---|---|---|---|---|---|---|---|---|---|
| **CA Manager** | — | X | X | OK | X | X | OK | X | OK |
| **CA Administrator** | X | — | X | X | X | X | X | X | X |
| **HSM-operatør** | X | X | — | X* | OK | X | X | X | X |
| **Key Ceremony Officer** | OK | X | X* | — | OK | X | X | X | X |
| **Registration Authority** | X | X | OK | OK | — | X | OK | X | OK |
| **PKI Auditor** | X | X | X | X | X | — | X | X | X |
| **Security Officer** | OK | X | X | X | OK | X | — | X | OK |
| **AO (FGDCorePKI)** | X | X | X | X | X | X | X | — | X |
| **Engineering Lead** | OK | X | X | X | OK | X | OK | X | — |

*HSM-operatør og Key Ceremony Officer: X under selve ceremonien; OK i øvrige perioder.

**Dual-control (D) kræves ved følgende operationer:**

| Operation | Person 1 | Person 2 |
|---|---|---|
| Root CA aktivering | Key Ceremony Officer | Security Officer |
| Root CA key generation | HSM-operatør (eksekverer) | Key Ceremony Officer (autoriserer) |
| ICA private key generation | HSM-operatør | CA Administrator |
| Wrap-key retrieval (backup) | HSM-operatør | CA Manager |
| CA-certifikat revokering | CA Administrator | CA Manager |
| HSM factory reset | HSM-operatør | CA Manager + FGD-godkendelse |
| Emergency CA shutdown | CA Administrator | Security Officer |

**Nøgle SOD-konflikter at fremhæve særligt:**

1. **CA Administrator × PKI Auditor:** Den person der drifter CA kan ikke audite sig selv — klassisk SOD-brud; bryder SOC 2 CC4.1 og ISO 27001 A.5.3
2. **CA Administrator × RA:** Den der godkender certifikat-anmodninger kan ikke selv udstede dem
3. **AO × CA Administrator:** Application Owner for FGDCorePKI må aldrig have operationel CA-adgang (se §1.1 SOD-analyse; samme triple-risk mønster som i `2026-05-08-mads-vurdering-app-owner-provider.md`)
4. **Engineering Lead × Production CA:** Engineering-teamet bygger integrationer; de drifter ikke produktions-CA

---

## 6. Anbefalinger til Laila og Poul

### 6.1 Til Laila: Rolle-design-anmodninger

Laila skal designe følgende projekt-roller for FGDCorePKI. For hver rolle angives Mads' kompetence- og ansvarsindhold; Laila formatterer i FGD's standard rolle-format.

**Prioritet 1 — Design disse inden FGDCorePKI-projekt-start:**

1. **CA Administrator (FGDCorePKI)**
   - Kompetencer: PKI-administration, X.509, CRL/OCSP, CA-software (f.eks. EJBCA, Dogtag, Step-CA — Pouls research afgør valget), YubiHSM2 PKCS#11 integration
   - SOD-begrænsninger: Ingen RA, PKI Auditor, HSM-operatør eller AO-kombination
   - Rapporterer til: CA Manager (Mads)
   - Autorisationsniveau: Fuld adgang til Tier 2 og Tier 3 CA-systemer; ingen Root CA-adgang udenfor ceremony

2. **HSM-operatør — FGDCorePKI tillæg (evt. Bjarne)**
   - Kompetencer: YubiHSM2 hardware-administration, PKCS#11, wrap-key management, backup/restore
   - SOD-begrænsninger: Ingen CA Administrator, PKI Auditor-kombination
   - Rapporterer til: CA Manager (Mads)
   - Eksisterende mandat (Bjarne/Knowlagecentral): Laila skal vurdere om FGDCorePKI-HSM kræver separat person eller tillæg til eksisterende mandat med eksplicit SOD-afklaring. Mads' præference: separat tillæg, da PKI-nøgler er højere kritikalitet end Knowlagecentral-nøgler

3. **Key Ceremony Officer (FGDCorePKI)**
   - Kompetencer: key ceremony-protokol (NIST SP 800-57), m-of-n scheme management, scripted ceremony execution
   - SOD-begrænsninger: Kan ikke varetage HSM-operatør-rollen under ceremoni
   - Rapporterer til: CA Manager (Mads)
   - Note: I opstartsfase kan Mads varetage denne rolle, men Laila skal designe rollen med henblik på at adskille den

4. **Registration Authority (FGDCorePKI)**
   - Kompetencer: CSR-validering, identitetsverifikation, PKI-policy-forståelse
   - SOD-begrænsninger: Ingen CA Administrator-kombination
   - Rapporterer til: CA Administrator (funktionelt) + CA Manager (compliance-eskalation)

**Prioritet 2 — Eksisterende roller der skal opdateres:**

5. **Trine (PKI Auditor-udvidelse)**
   - Triners eksisterende compliance-auditor-mandat udvides med PKI-specifik audit
   - Kompetence-gap: Trine skal have PKI audit-metodologi (WebTrust for CAs eller intern tilpasning). Mads identificerer træningsbehov og bringer anbefaling til Laila + Stefan

6. **Application Owner for FGDCorePKI**
   - AO-rollen følger samme SOD-rammer som `2026-05-08-mads-vurdering-app-owner-provider.md`
   - Yderligere begrænsning specifik for FGDCorePKI: AO må IKKE have adgang til CA-systemer, HSM eller key ceremony. AO's ansvar er forretnings- og produkt-ejerskab, ikke operationel PKI-drift
   - AO kan ikke self-certifiere compliance — Trine (PKI Auditor) + Mads er de eneste der kan attestere PKI compliance-status

7. **Application Provider for FGDCorePKI**
   - AP (infrastruktur-ansvar) er separat fra CA Administrator og HSM-operatør
   - AP håndterer server-infrastruktur (OS, netværk, Tailscale, monitoring) — ikke CA-software eller HSM
   - SOD: AP har ingen direkte CA-software-adgang; CA Administrator er eneste autoriserede CA-operatør

### 6.2 Til Poul: Research-anmodninger

**Track A: CA-software-vurdering**

Poul skal researche og anbefale CA-software der passer til FGDCorePKI's krav:
- On-premises deployment
- YubiHSM2 PKCS#11-integration (verificer at valgt software understøtter YubiHSM2's PKCS#11-profil)
- Three-tier CA-hierarki support
- ACME (RFC 8555) server-side support (til Tier 3 Issuing CA)
- SCEP (RFC 8894) og EST (RFC 7030) support (til enterprise-klienter)
- Open source foretrukket (EJBCA Community, Step-CA, Dogtag) vs. commercial (EJBCA Enterprise, DigiCert PKI Platform)
- Verificer aktuelle versioner + FIPS 140-3-kompatibilitet med YubiHSM2

**Track B: HA-arkitektur for HSM (Tier 2)**

Briefens "shared YubiHSM2" er en SPoF. Poul skal undersøge:
- YubiHSM2 key replication/sync-muligheder (wrap-export til secondary HSM)
- Operational cost ved HA-pair (hardware + vedligeholdelses-kompleksitet)
- Alternativ: accept af SPoF med dokumenteret RTO/RPO
- Levér anbefaling til FGD med cost/risk-matrix

**Track C: Certificate lifecycle management tooling**

Engineering-teamet skal bruge cert-management tools. Poul researcher:
- Certbot/ACME.sh (klient-side automation)
- Keyfactor, Venafi, cert-manager (Kubernetes-kontekst ved behov)
- HashiCorp Vault PKI Secrets Engine som mulig middleware
- Kompatibilitet med on-premises deployment og YubiHSM2

### 6.3 Minimums-teamstørrelse for compliant PKI

For at køre FGDCorePKI compliant med FIPS 140-3, ISO 27001, SOC 2 Type II og SOD-kravene ovenfor:

| Funktion | Minimum antal separate personer |
|---|---|
| CA Manager | 1 (Mads) |
| CA Administrator | 1 |
| HSM-operatør | 1 |
| Key Ceremony Officer | 1 (kan overlap med CA Manager i opstartsfase) |
| Security Officer | 1 (kan overlap med CA Manager i opstartsfase) |
| Registration Authority | 1 (kan kombineres med Security Officer) |
| PKI Auditor | 1 (Trine) |
| Engineering team | Minimum 3 |
| **Total (med overlaps)** | **Minimum 8 separate personer** |

Under 8 er muligt i opstartsfase med dokumenterede, tidsbegrænsede undtagelser godkendt af FGD — men det skal være eksplicit accepteret risiko med plan for normalisering inden første eksterne kunde på FGDCorePKI.

---

## 7. Åbne spørgsmål til FGD

Disse kræver FGD's stillingtagen inden arkitektur-godkendelse:

1. **FIPS 140-3 vs. 140-2:** Bekræft at FIPS 140-3 Level 3 er det operative krav (se compliance-note øverst i dette dokument)
2. **HA for HSM (Tier 2):** Accepter SPoF eller investér i HA-pair? Poul leverer cost/risk-anbefaling
3. **Scope for FGDCorePKI i fase 1:** Kun intern FGD-brug, eller skal eksterne kunder fra start stole på FGD's CA-hierarki? (påvirker CP/CPS-krav markant)
4. **Bjarne / HSM-operatør:** Separat person for FGDCorePKI-HSM eller Bjarne med udvidet mandat? Laila og Mads anbefaler separat tillæg
5. **CP/CPS-publicering:** Skal FGD's CP/CPS publiceres (kræves ved public-trust CA); eller intern CA til private PKI? (påvirker audit-krav)

---

*Dokument-ejer: Mads (CISO)*
*Næste revision: Ved FGD-godkendelse af åbne spørgsmål (§7) eller ved væsentlig arkitekturændring*
*Distribueret til: Laila (rolle-design), Poul (research-tracks), Stefan (koordinering)*
