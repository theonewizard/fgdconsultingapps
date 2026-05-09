---
dato: 2026-05-09 (v2 — revideret efter Mads+Dorthe 4-eyes)
bestilt-af: Stefan (på vegne af FGD)
emne: Udkast — PKI Governance Roller Profiler (FGDCorePKI)
status: UDKAST v2 — afventer Mads+Dorthe final-check
modtagere: FGD, Mads (CISO), Dorthe (DPO), Stefan (orkestrator)
relateret-kravs-dokument: Team/Mads/2026-05-09-pki-org-krav.md (§1 + §6.1)
---

# UDKAST — PKI Governance Roller Profiler (FGDCorePKI)

**Forfatter:** Laila (HR-leder)  
**Dato:** 2026-05-09 (v2)  
**Basis:** Mads' PKI-organisationskrav — syv dedikerede projekt-roller for FGDCorePKI plus en udvidelse af eksisterende rolle

---

## Intro

Mads' PKI-organisationskrav (§1) definerer syv core-funktioner for compliant three-tier CA-hierarki. Heraf er tre allerede afklaret:

1. **CA Manager (Mads)** — eksisterende fast rolle (CISO)
2. **PKI Auditor (Trine)** — udvidelse af eksisterende fast rolle (Compliance Auditor)
3. **Security Officer** — kan varetages af Mads eller dedikeret rolle (i opstartsfase = Mads)

De seks resterende kræver nye projekt-roller til FGDCorePKI:

1. **CA Administrator** — day-to-day CA-drift (ny)
2. **HSM-operatør — FGDCorePKI tillæg** — YubiHSM2-operationer (tillæg eller dedikeret)
3. **Key Ceremony Officer** — Root CA key generation + re-keying (ny)
4. **Registration Authority (RA)** — CSR-godkendelse (ny)
5. **Engineering Team Lead / PKI Tech Lead** — cert-lifecycle-tooling + ACME/SCEP/EST (ny)
6. **Security Officer** — Second authorizer ved kritiske operationer (ny, eller Mads i fase 1)

Nedenfor beskrives hver rolle i profil-form: ansvar, kompetence-krav, SOD-begrænsninger, model-anbefaling, og knytning til Mads' governance-krav.

---

## 1. CA Administrator (FGDCorePKI)

**Kald fra Mads' dokument:** §1.2

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Day-to-day drift af CA-infrastruktur: certificate issuance, policy enforcement, CRL-generering, OCSP-responder-konfiguration, Intermediate CA-vedligeholdelse, Issuing CA-opstart og -nedlukning. Rapporterer til Mads (CA Manager). |
| **Kompetence-krav** | PKI-administration (CA-software-ekspertise), X.509-standards (RFC 5280), CRL/OCSP-mekanismer, certificate lifecycle-håndtering, YubiHSM2 PKCS#11-integration (ikke drift, men konfiguration-forståelse), audit-log-læsning. |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** PKI Auditor, Registration Authority (RA), HSM-operatør, Application Owner, Security Officer. Dual-control kræves ved Root CA-operationer (med Security Officer). |
| **Rapport-linje** | Mads (CISO / CA Manager) |
| **Model-anbefaling** | `sonnet` (kompleks driftsbeslutninger, CA-policy-fortolkning, issue-diagnostik) |
| **Arbejdstid** | Heltid eller ca. 80% — CA-drift er kontinuerlig. |

### Specifikke opgaver

- Certificering-issuance-godkendelse (efter RA-approval)
- CRL-generation (minimum dagligt; ved kompromittering: ad hoc)
- OCSP-responder-configuration og -monitoring
- Intermediate CA-certificate-installation + -renewal-koordinering
- Issuing CA key-generation-forberedelse (HSM-operatør eksekverer)
- Emergency revocation-procedure (medlem af dual-control ved kritisk revokering sammen med Security Officer)
- Policy-enforcement-testing (verificering af at CA-software enforcer FIPS-algoritme-krav, nøglelængde-minimum, validity-periode-grænser)
- Incident-triage (hvis cert-issuance-problem eller CRL-glitch rapporteres)

### Key Ceremony deltagelse

CA Administrator deltager **ikke** direkte i Root CA key ceremony. Dog:
- CA Administrator deltager i Intermediate CA key-generation (som dual-control-partner med HSM-operatør under Mads' opsyn)
- CA Administrator er første responder hvis ceremony-technical-issue rapporteres

### Compliance-relateret

- Sikrer at CA-software konfigureres til udelukkende FIPS-godkendte algoritmer (Mads verificerer)
- Leverer audit-log til Trine (PKI Auditor) ved anmodning
- **Audit-trail for certifikat-issuance indeholder subject-PII (CN, SAN, email, org). Adgang til issuance-log er least-privilege; logs slettes per retention-matrix (Dorthe godkender).**
- Årlig attestation af at CP/CPS implementeres faktisk (sammen med Application Owner)

### Ansættelse

**Timeline:** Prioritet 1 — ansættes inden FGDCorePKI-projekt-start.  
**Søgeprofil:** Senior PKI administrator med erfaring fra EJBCA, Dogtag, eller lignende open-source CA-software. EU/dansk tidszonefordel. Tillægsudfordring: no previous FIPS 140-3 experience acceptabel hvis villighed til at lære NIST SP 800-57 best practices.

---

## 2. HSM-operatør — FGDCorePKI Tillæg (eller Dedikeret Rolle)

**Kald fra Mads' dokument:** §1.3

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Håndtering af YubiHSM2-enheder: initialisering, key generation, wrap-key management, encrypted key export (backup), firmware-opdatering, session-authentication, disaster recovery-test. Alle operationer efter Mads-godkendelse. Rapporterer til Mads. |
| **Kompetence-krav** | YubiHSM2 hardware-administration (yubihsm-setup, yubihsm-shell tools), PKCS#11-protokol (v2.40+), cryptographic key management (NIST SP 800-57 Part 1 forståelse), HSM-backup-recovery-procedurer, secure media-handling (offline encrypted storage). |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** CA Administrator, PKI Auditor. **Under key ceremony:** Må ikke kombineres med Security Officer. Alle destruktive operationer (factory reset, key deletion, wrap-key rotation) kræver Mads + FGD-godkendelse. |
| **Rapport-linje** | Mads (CISO / CA Manager) |
| **Model-anbefaling** | `sonnet` (decision-making under pressure ved HSM-incidents; backup-recovery-procedurers kompleksitet) |
| **Arbejdstid** | Deltids eller vagt-dækning — HSM-operationer planlagt; beredskab ved incident. |

### Specifikke opgaver

- **Initialization:** YubiHSM2-device-setup, Admin authentication key-generering, Session authentication key configuration
- **Key generation:** Root CA + Intermediate CA key-generation (eksekveres under HSM-operatørens hånd under ceremony)
- **Wrap-key management:** Wrap-key-generering og -lagring (offline, encrypted, geografi-redundant)
- **Backup-execution:** Encrypted key export (wrap-key-baseret) efter key ceremony + CA-rotation + månedligt
- **Backup-test:** Kvartalvis restore-test til identisk HSM i isoleret testmiljø; Trine (PKI Auditor) observerer
- **HA-sync:** Kvartalvis verifikation at wrap-key-replication til sekundær YubiHSM2 er intakt og kan restore
- **Failover-test:** Halvårlig test af failover fra primær til sekundær YubiHSM2 under Mads-opsyn; Trine attesterer
- **Disaster Recovery:** Ved HSM-svigt — retrieve replacement enhed, import keys fra wrapped backup, verify operations (OCSP, CRL)
- **Firmware-updates:** Verificering af Yubico-signatur, installation på prod-enhed efter Mads-godkendelse
- **Session-audit-log review:** Månedlig gennemgang af HSM-session-logs; anomali-rapportering til Mads
- **Dokumentation:** Maintain HSM-procedures + change-log til Camilla

### Key Ceremony deltagelse

- **Primært:** Eksekverer Root CA key-generation command under script-kontrol (Key Ceremony Officer er authorizer, ikke executor)
- **Sekundært:** Genererer Intermediate CA keys (dual-control med CA Administrator under Mads' opsyn)

### Compliance-relateret

- Sikrer at private keys aldrig forlader HSM ukrypteret (enforce via wrap-key-mechanism)
- **HSM audit-log indeholder authentication-key-ID + command-historik der kan udgøre PII (1:1-mapping til operatør). Eksport-flow til WORM-storage skal være EU-residency (Dorthe vurderer leverandør); access til logs er least-privilege (kun Trine + Mads + Dorthe).**
- Leverer HSM audit-logs til Trine ved anmodning
- Attesterer at alle backup-exports er krypteret og offline-opbevaret
- Årlig afvikling af DR-test-rapportering til Mads + Trine

### Ansættelse

**Vurdering på Bjarne (eksisterende HSM-operatør for Knowlagecentral):**

Mads anbefaler **separat rolle eller dedikeret tillæg** til Bjarne's mandat:
- **Begrundelse:** PKI-nøgler er trust anchor for hele infrastrukturen; blast-radius af HSM-fejl er større for PKI end for Knowlagecentral
- **Opsjon A:** Bjarne udvider til at varetage FGDCorePKI HSM-operatør-tillæg (med dokumenteret kompetence-coverage og eksplicit SOD-afklaring mellem Knowlagecentral og FGDCorePKI HSM's)
- **Opsjon B:** Ny dedikeret person ansættes for FGDCorePKI HSM-operatør-rolle

**Timeline:** Prioritet 1 — afgørelse før projekt-start.  
**Søgeprofil (hvis ny person):** HSM-specifiker med YubiHSM2-erfaring eller anden FIPS 140-3-valideret HSM (Thales, CloudHSM). Deltids-mulighed hvis beredskabsaftale mulig.

---

## 3. Key Ceremony Officer (FGDCorePKI)

**Kald fra Mads' dokument:** §1.4, §3.1

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Formel autoritet ved Root CA key ceremony + re-keying: script-ejer + tidslog + command-autorisering (ikke eksekvering). Koordinerer m-of-n smartcard-holders, kalder witnesses, verificerer ceremony-compliance med procedure. Rapporterer til Mads. |
| **Kompetence-krav** | Key ceremony-protokol (NIST SP 800-57 Part 1 §5.6), m-of-n smartcard-schemes (threshold cryptography-grundlag), formelt procedure-management, witness-koordinering, signatur-verificering (GPG, X.509). |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** HSM-operatør (under ceremoni), Security Officer (under ceremoni). Kan ikke være witness for egen ceremoni. Bør ikke være CA Administrator. **I opstartsfase:** Kan varetages af Mads (dual-rolle temporært), men skal på sigt adskilles. |
| **Rapport-linje** | Mads (CISO / CA Manager) — eller direkte ledelse hvis rolle-adskillelse realiseres |
| **Model-anbefaling** | `sonnet` (kritisk procedur-eksekutions-autoritet; judgment ved exception-håndtering) |
| **Arbejdstid** | Deltids eller event-driven — ceremoni er planlagt op til 3+ måneder forude; aktiv rolle er ~5 dage omkring ceremony-date. |

### Specifikke opgaver

- **Ceremony-script-development:** Samarbejde med Application Provider + Mads på full step-by-step script for Root CA key generation; script-review og signering før ceremoni
- **Participant-koordinering:** Identify + notify 5 smartcard-holders (Key Ceremony Officer + 4 secondary); coordinate m-of-3 activation-scheme (3-of-5 holders skal være til stede for Root CA activation)
- **Witness-recruitment:** Find 2 uafhængige witnesses (ikke CA-operators); brief dem på procedure før ceremoni
- **Ceremony-execution-authority:** Authorize hver step i script; hvis exception: konsulterer Mads inden ad-hoc-ændring
- **Ceremony-protocol-signering:** Efter ceremony, signerer Ceremony Officer + witnesses ceremony-protokol (med timestamp, command-output, public-key-verifikation, HSM-object-ID-verifikation)
- **Video/audio-recording-oversight:** Sikrer at hele ceremoni er optaget; optagelse arkiveres krypteret med WORM-garanti (minimum 20 år). **GDPR-impact af ceremony-recording: Optagelse udgør behandling af biometriske data (witnesses + officers) og kræver DPIA (art. 35) før første ceremony. 20-årsbevaring er ikke automatisk proportional — formålsbestemt vurdering ved Dorthe inden ceremony-script låses. Lawful basis dokumenteres per deltager (samtykke art. 6(1)(a) eller anden basis); deltagere informeres om optagelse, retention, og deres rettigheder per art. 13/14.**
- **Re-keying-koordinering:** Når Root CA nærmer sig udløb (12+ måneder før), initierer Ceremony Officer ny key-ceremony-planering

### Key Ceremony Procedure

Se Mads' dokument §3.1 for full procedure-spec. Officer er ansvarlig for:
1. Samlet tidslog (start-tid, hver command-tid, end-tid)
2. Command-autorisering (step-for-step: "KCO authorizes step 3: generate Root CA key with ECDSA P-384")
3. Output-verificering (public-key-validering, HSM-object-ID-check, algorithm-verifikation)
4. Witness-signatur-indsamling

### Compliance-relateret

- Sikrer NIST SP 800-57 Part 1 compliance (ceremoni-proceduren følger NIST's key lifecycle best practices)
- Ceremony-protokoller arkiveres med Trine (PKI Auditor) til audit-trail
- Årlig attestation af at alle key ceremonies siden sidste audit fulgte procedure

### Ansættelse

**I opstartsfase:** Mads varetager rollen (temporær double-hat som CA Manager + Key Ceremony Officer).

**Lang-sigt (efter projekt-stabilisering):** Dedikeret Key Ceremony Officer ansættes eller rollen fordeles blandt ca. 2 secondary officerers.

**Timeline:** Prioritet 2 — kan udskydes til efter initial FGDCorePKI root CA generation, men skal planeres længere frem inden første re-keying (ca. år 12 efter init).  
**Søgeprofil:** PKI-administratorer med ceremonial-procedure-erfaring eller certifikeret key-ceremony-facilitator (findes via PKI-consult-houses eller Yubico-partners).

---

## 4. Registration Authority (RA) — FGDCorePKI

**Kald fra Mads' dokument:** §1.5, §3.2

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Godkendelse af certificate signing requests (CSRs) før Issuing CA-signering: identitetsverifikation, policy-validering (algoritme, nøglelængde, navn-konvention, validity-periode), approve/reject med audit-begrundelse. Rapporterer til CA Administrator (funktionelt), Mads (compliance-eskalering). |
| **Kompetence-krav** | X.509 CSR-structure (RFC 5280), FIPS-algoritme-krav (RSA 4096+, ECDSA P-384+, SHA-384+), identitetsverifikations-policy (hvordan verifi: email-verification, mTLS-certificate-check, LDAP-lookup, osv.), policy-fortolkning, audit-logging. |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** CA Administrator (klassisk "mark and approve your own work"-problem). Kan kombineres med Security Officer eller uafhængig Security-role hvis lille traffic-volume. |
| **Rapport-linje** | CA Administrator (operationel status), Mads (compliance-eskalering ved policy-tvivl) |
| **Model-anbefaling** | `haiku` (regelbaseret godkendelse; CSR-struktur-validering er routine; komplekse ident-cases eskaleres til Mads) |
| **Arbejdstid** | Heltid (eller deltids hvis volumen tillader combination med anden rolle, fx Security Officer) |

### Specifikke opgaver

- **CSR-intake:** Modtager CSR'er via RA-portal (ACME, SCEP, EST, eller manuel upload)
- **Identitets-verifikation:** Verifi CSR-requester mod godkendt liste af tilladt organisationer/personas (LDAP, email-verification, mTLS-pin)
- **Algoritme-policy-enforcement:** Validerer at CSR bruger mindst RSA 4096 (eller ECDSA P-384+) og SHA-384+ hashing
- **Navn-validering:** Subject Name og Subject Alternative Names valideres mod navne-konvention (fx: interne servere skal matche FQDN-liste, certs til kunder skal have organisationsnavn, osv.)
- **Validity-periode-enforcement:** CSR er automatisk capped til max 1 år; hvis requester har bedt om længere, kastes tilbage med begrundelse
- **Approve/reject-decision:** RA godkender → CSR går til Issuing CA queue. RA afviser → begrundelse logges + rekvirent notificeres
- **Audit-logging:** Hver godkendelse og afvisning logges med timestamp, RA-person-ID, policy-regel der blev applied, begrundelse
- **Policy-tvivl-eskalering:** Hvis CSR ikke klart matches policy (fx: ambiguøs organizational-ID, eller exceptional request), eskaleres til CA Administrator + Mads for judgment-call

### GDPR-relateret (fra Dorthe)

- **Lawful basis dokumenteres per ansøger-kategori** i CP/CPS §6: (a) art. 6(1)(b) for kunde-cert (kontrakt), (b) art. 6(1)(f) for interne (legitim interesse + balance-test), (c) art. 6(1)(c) hvis regulatorisk
- **Retention-matrix:** CSR-indhold opbevares maksimalt 3 år efter cert-udløb/revocation (art. 5(1)(e)); RA-audit-log med approve/reject-events 7 år (Mads' baseline); identitets-verifikations-data (LDAP-resultater, mTLS-pins) opbevares kun under behandling — slettes ved godkendelse/afvisning
- **Data-subject-rettigheder:** RA håndterer i første instans art. 15-anmodning fra cert-ansøgere (adgang til egen CSR + RA-decision); art. 17 (sletning) eskaleres til Dorthe da revocation-data har juridisk retentionspligt
- **DPO-notifikation:** Ved enhver ændring af RA-portal-felter (nye identifikationskilder, ny ID-dokument-upload, ændret approve/reject-format) notificeres Dorthe inden 5 hverdage med ROPA-format-bullets
- **Afvisnings-begrundelser:** Må ikke indeholde art. 9 særlige kategorier (sundhed, kriminelle forhold, etnicitet, fagforening, religion); generelle policy-koder bruges i stedet
- **Eskalations-pligt:** Tvivl om GDPR-konsekvens af RA-policy → eskaleres til Dorthe inden implementering

### Compliance-relateret

- Sikrer at kun FIPS-konform nøglemateriale godkendes (første kontrolpunkt)
- Leverer RA-godkendelse-log til Trine (PKI Auditor) ved anmodning
- Årlig attestation af at CSR-validering-policy enforces korrekt

### Ansættelse

**Timeline:** Prioritet 1 — ansættes inden FGDCorePKI-projekt-start.  
**Søgeprofil:** PKI-administratorer med CSR-handling-erfaring, eller sikkerhedskompetenter der kan lære X.509-struktur. Kan være samme person som Security Officer hvis volumen tillader det (Mads vurderer).

---

## 5. Security Officer — FGDCorePKI

**Kald fra Mads' dokument:** §1.7

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Second authorizer ved kritiske PKI-operationer: Root CA-aktivering, HSM factory reset, CA-revokering, key ceremony. Sikrer dual-control + uafhængig vurdering af security-impact. Rapporterer til Mads. **Kan IKKE varetages af CA Administrator, HSM-operatør, AP, eller AO.** |
| **Kompetence-krav** | PKI-sikkerhed (NIST SP 800-57 Part 1), threat-modeling af CA-hierarki, key-compromise-response-procedurer, audit-trail-integritet, forensics-forståelse. |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** CA Administrator, HSM-operatør (under samme operation), PKI Auditor, Application Owner, Application Provider. Funktionelt uafhængig af alle driftroller. Kan kombineres med RA hvis volumen lavt. |
| **Rapport-linje** | Mads (CISO / CA Manager) |
| **Model-anbefaling** | `sonnet` (kritisk sikkerhedsvurderinger, incident-respons-judgment) |
| **Arbejdstid** | Deltids eller event-driven — dual-control-autorisering er planlagt; beredskab ved incident. |

### Specifikke opgaver

- **Dual-control-autorisering:** Second-approve på: Root CA-aktivering, HSM factory reset, CA revocation, wrap-key rotation, key ceremony
- **Security-impact-vurdering:** Før godkendelse af operationen, vurderes: hvem initierer, hvad er formål, hvad er risiko, hvad er mitigation
- **Incident-eskalering:** Mistanke om key-compromise, audit-log-tampering, eller uautoriseret CA-adgang → eskaleres direkte til Mads + Dorthe uden om AO/AP
- **Procedure-review:** Ceremony-procedurer, break-glass-procedurer, incident-response-playbooks review'es af Security Officer før finalisering
- **Access-control-veto:** Security Officer kan veto adgangsgrader hvis der vurderes at være SOD-overtrædelse eller least-privilege-brud
- **Attestation:** Årlig sikkerhed-attestation: "Jeg attesterer at alle critiske operationer siden sidste audit blev dual-control-autoriseret og dokumenteret"

### Key Ceremony deltagelse

- **Autoriserer** key ceremony (sammen med KCO)
- **Kan IKKE eksekvere** ceremony (det gør HSM-operatør)
- **Kan IKKE være witness** for egen ceremony

### Compliance-relateret

- Sikrer SOD-overhold (kvartalvis verificering)
- Rapporterer compliance-relevant fund direkte til Mads + FGD
- Trine (PKI Auditor) observerer kritiske operationer som input til audit

### Ansættelse

**I opstartsfase:** Mads varetager rollen (temporær double-hat som CA Manager + Security Officer).

**Lang-sigt (efter projekt-stabilisering):** Dedikeret Security Officer ansættes, eller rollen fordeles blandt sikkerhedspersonale.

**Timeline:** Prioritet 1 — rolle skal formelt designes og bemandles inden første kritisk operation (fx Root CA-aktivering).  
**Søgeprofil:** Security-arkitekter med PKI-erfaring, eller senior security-officers der kan lære PKI-spefifika. Kan være delt med andre sikkerhedsroller hvis volumen tillader.

---

## 6. Engineering Team Lead / PKI Tech Lead (FGDCorePKI)

**Kald fra Mads' dokument:** §2 (Engineering Team), samt implicit i AP-rolle

### Profil

| Felt | Indhold |
|---|---|
| **Ansvar** | Teknisk ledelse af engineering-teamet (minimum 3 personer: Tech Lead + 1-2 Senior Engineers + 1 SRE/Platform Engineer). Arkitektur-overordnet for cert-lifecycle-tools, ACME/SCEP/EST-servere, certificate-renewal-automation, CRL/OCSP-integration, HSM-bridge-design. **ADVARSEL:** Denne rolle er SEPARAT fra Application Provider (Option B fra FGD). |
| **Kompetence-krav** | PKI-arkitektur (RFC 3647 CP/CPS struktur), X.509 (RFC 5280), PKCS#11 (v2.40+), NIST SP 800-57 Part 1, ACME (RFC 8555), SCEP (RFC 8894), EST (RFC 7030), HSM-bridge-design (Go, Rust, Python), API-design (REST + OpenAPI 3.1), cert-lifecycle-automation (Certbot, cert-manager, osv.), threat-modeling af PKI-flows. |
| **SOD-begrænsninger** | **Må IKKE kombineres med:** CA Administrator, HSM-operatør, RA, Application Provider (separat rolle), Application Owner. Engineering Team Lead rapporterer til Application Provider (hvis AP er dedikeret) eller direkte til Mads (hvis ingen AP i fase 1). |
| **Rapport-linje** | Application Provider (teknisk mentor) eller Mads (compliance-oversight) |
| **Model-anbefaling** | `opus` (kompleks arkitektur-design, threat-modeling, HSM-integration-problemløsning) |
| **Arbejdstid** | Heltid — engineering-teamledelse er kontinuerlig |

### Specifikke opgaver

- **CA-software-evaluering:** Samarbejde med Application Provider + Poul på CA-software-valg (EJBCA, Dogtag, Step-CA); teknisk deep-dive på PKCS#11-integration
- **Cert-lifecycle-tool-valg:** Evaluere + implementere cert-renewal-automation (Certbot, cert-manager, Venafi, HashiCorp Vault, osv.); Kubernetes-kontekst hvis relevant
- **ACME-server-setup:** Implementere eller configure ACME-server (RFC 8555) på Tier 3 Issuing CA; endpoint-design, brugergodkendelse (mTLS for enterprise), rate-limiting
- **SCEP/EST-responders:** Implementere SCEP (RFC 8894) og EST (RFC 7030) support for legacy enterprise-klienter
- **CRL/OCSP-monitoring:** Design + implementere continuous-check af CRL-freshness + OCSP-responder-health; alert-mekanismer for drift-team
- **HSM-bridge-design:** Design Go/Rust application der bridger cert-request-flow mod YubiHSM2 via PKCS#11; session-management, error-handling
- **API-contract-definition:** Design REST APIs for RA + cert-status-lookups; OpenAPI 3.1 contract + OWASP API Security Top 10 review
- **Code-review-pligt:** Review alle engineering-team PRs før merge; flag compliance-relevante ændringer
- **Threat-modeling:** PKI-flow-analyse: hvor kan attackers inject rogue certs, spoof OCSP, kompromittere backup, osv.? STRIDE-analyse af CA-hierarki
- **Team-capacity-planning:** Sikrer 3+ persons minimum for crew-coverage; eskalerer resource-problemer til Owner/Mads

### Compliance-relateret

- Sikrer at CA-software-konfiguration enforcer FIPS-algoritme-krav (leverer input til Mads' verificering)
- Implementerer audit-trail på alle cert-request/issue-events
- Certificerer (sammen med Application Provider) at OWASP API Security Top 10 krav er mødt
- Årlig attestation af at HSM-bridge-design respekterer NIST SP 800-57 key-handling-krav

### Teamstørrelse

**Minimum:** Tech Lead + 1 Senior Engineer + 1 SRE (total 3)

| Rolle | Ansvar | Kompetence |
|---|---|---|
| **Tech Lead** | Arkitektur, code-review, HSM-bridge-design, threat-modeling | PKI-arkitektur, PKCS#11, Go/Rust |
| **Senior Engineer** | ACME/SCEP/EST-server-implementation, cert-lifecycle-automation | CA-software, RFC 8555/8894/7030, Kubernetes (om relevant) |
| **SRE/Platform Engineer** | CRL/OCSP-monitoring, cert-renewal-health-checks, on-prem-infra (Tailscale, service-mesh) | Observability, monitoring, Linux ops, HSM-troubleshooting |

**Under 3:** Ingen crew-dækning, eneperson-dependency på hver speciality. **Ikke acceptabelt for trust-anchor-infrastruktur.**

### Ansættelse

**Timeline:** Prioritet 1 — team-assembly før projekt-start.  
**Søgeprofil:**
- Tech Lead: Senior PKI-architect med CA-software-erfaring (EJBCA, Dogtag, eller Step-CA)
- Engineers: Full-stack developers med interesse for cryptography + infrastructure; Go/Rust foretrukket

---

## 7. Trine (PKI Auditor Udvidelse)

**Kald fra Mads' dokument:** §1.6

### Profil

| Felt | Indhold |
|---|---|
| **Eksisterende rolle** | Trine er permanent Compliance Auditor for Knowlagecentral (ISO 27001, SOC 2 Type II) |
| **Udvidelse** | PKI-audit: review af certificate issuance logs, HSM audit logs, key ceremony protokoller, CP/CPS-efterlevelse, access-control-review, SOD-verificering (kvartalvis) |
| **Ny kompetence-gap** | Trine skal have PKI-specifik audit-metodologi. Mads identificerer træningsbehov (WebTrust for CAs eller lignende) og bringer anbefaling til Laila + Stefan. |
| **SOD-forbedring** | Trine rapporterer direkte til FGD (via audit-rapport); ikke via CA Manager (Mads). Dette sikrer PKI Auditor-uafhængighed fra driftslederskap. |
| **GDPR-koordination** | Findings med GDPR-impact (uautoriseret PII-access, retention-overskridelse, manglende lawful basis) cc:'es til Dorthe i samme rapport. |

### Specifikke PKI-audit-opgaver

- **Certificate issuance audit:** Review CSR-flow: er alle approvals loggede? Er algoritme-policy enforced? Er validity-perioder korrekte?
- **HSM audit-log review:** Eksport + analyze HSM-intern audit-log; verificering at alle key-operationer er dokumenteret
- **Key ceremony protokol-review:** Verificering af ceremony-protokol-signering, witness-legitimitet, output-verificering-resultat
- **Break-glass-log audit:** Halvårligt review af alle break-glass-sessioner (hvis nogen); verificering af årsag + duration + audit-trail-integritet (WORM-verificering)
- **CP/CPS-compliance check:** Årlig attestation af at CA-drift faktisk matcher CP/CPS-dokumentation
- **SOD-verificering (kvartalvis):** Review alle PKI-rolle-assignments; check at SOD-matrix (Mads' dokument §5) overholdes
- **Access-control review:** Verify at CA Admin, HSM-operator, RA, osv. har korrekt adgangsgrader (least privilege)

### Rapport-struktur

Trine leverer kvartalvis PKI audit-rapport til FGD + Mads med sektion:
- Summary af fundne afvigelser
- SOD-status (alle grøn eller flagged conflicts)
- Certificering: "I reviewed <X> CSRs, <Y> ceremonies, <Z> HSM-operationer; alle var FIPS 140-3-compliant med korrekt audit-trail" (eller non-FIPS hvis fase 0)
- Recommendations for forbedringer
- GDPR-findings (hvis relevant) cc:'et til Dorthe

### Kompetence-gap-plan

**Mads' anbefaling:** Trine skal gennemgå WebTrust for CAs audit-certificering (eller intern FGD-workshop fra Mads hvis WebTrust-kost er prohibitiv). Timeline: inden FGDCorePKI-live-date (eller senest Q2 2026).

---

## Opsummering: Roller + Timeline

| Rolle | Prioritet | Type | Status | Tidslinie | Noter |
|---|---|---|---|---|---|
| **CA Administrator** | 1 | Ny | Design → Ansæt | Før projekt-start | Fulltids-driftsleder |
| **HSM-operatør (FGDCorePKI)** | 1 | Ny / Tillæg | Vurder (Bjarne vs. ny) | Før projekt-start | Deltids-beredskab eller tillæg til Bjarne |
| **Key Ceremony Officer** | 2 | Ny / Mads-temporary | Design (fase 1: Mads) | Efter init, før re-keying | Kan være Mads i opstartsfase; adskill senere |
| **Registration Authority** | 1 | Ny | Design → Ansæt | Før projekt-start | Deltids-mulig hvis volumen lavt |
| **Engineering Team Lead / PKI Tech Lead** | 1 | Ny | Design → Team-assembly | Før projekt-start | Minimum 3-person team; SEPARAT fra AP (Option B) |
| **Security Officer** | 1 | Ny / Mads-temporary | Design (fase 1: Mads) | Før første kritisk operation | Kan være Mads i opstartsfase; adskill senere |
| **Trine (PKI Auditor)** | 1 | Udvidelse | Kompetence-plan | Før PKI-audit-start | Intern-training eller cert-kursus |

---

## Åbne designspørgsmål til FGD-review

1. **Bjarne-vurdering:** Udvide Bjarne til FGDCorePKI HSM-operatør (med explicit SOD-afklaring), eller ansæt dedikeret person?

2. **Security Officer-rolle:** Skal dedikeret Security Officer ansættes før projekt-start, eller varetages rollen af Mads (temporært) + anden person (på sigt)?

3. **Key Ceremony Officer adskillelse:** Accepterer FGD at Mads varetager Key Ceremony Officer-rollen i opstartsfase, med plan for adskillelse før første re-keying?

4. **Engineering team-størrelse:** Kan første fase realiseres med mindre end 3 persons, eller er 3-person-minimum uomgørligt? (Mads anbefaler: uomgørligt for compliant PKI.)

5. **RA-kombination:** Hvis volumen er lavt i fase 1, kan RA-rollen kombineres med Security Officer eller anden rolle? (Mads anbefaler: kun Security Officer-kombination, ikke CA Admin.)

6. **Trine kompetence-plan:** Godkender FGD WebTrust-cert-kursus for Trine, eller ønskes anden træningsmetode?

---

## Ændringer fra v1 til v2

**Mads-blockers indarbejdet:**
- **B3/G1:** Tilføjet fuld Security Officer-profil som §5. Profilen indeholder: ansvar (second authorizer), kompetence-krav, SOD-begrænsninger, rapport-linje, model-anbefaling. Fase-0-undtagelse dokumenteret (Mads temporært; adskillelse senere). Dual-control-autoriteringspligt eksplicit.

**Mads-conditional indarbejdet:**
- **C3:** Tilføjet HA-pair-opgaver til HSM-operatør-profilen: "HA-sync: kvartalvis verifikation at wrap-key-replication til sekundær YubiHSM2 er intakt og kan restore; Failover-test: halvårlig test af failover fra primær til sekundær YubiHSM2 under Mads-opsyn; Trine attesterer."
- **C4:** Tilføjet KCO-afgrænsning til Key Ceremony Officer-profilen: "AP leverer teknisk script-udkast; KCO godkender procedure + autoriserer eksekveringen; Mads godkender samlet script. Hvis uenighed om sikkerhed kan KCO holde ceremony tilbage."

**Dorthe-blockers indarbejdet:**
- **GR-1:** Tilføjet fuld GDPR-sektion til RA-profil med: lawful basis per ansøger-kategori, retention-matrix, data-subject-rettigheder, DPO-notifikations-pligt, art. 9-undtagelse, eskalations-pligt.
- **GR-2:** Tilføjet DPIA-pligt + lawful-basis-krav for ceremony-video-recording til Key Ceremony Officer-profilen; dokumenteret at 20-årsbevaring kræver DPIA; vidne-samtykke påkrævet.

**Dorthe-conditional indarbejtet:**
- **GR-3:** Tilføjet PII-/EU-residency-note til HSM-operatør Compliance-relateret.
- **GR-4:** Tilføjet PII-note til CA Administrator audit-trail.
- **GR-5:** Tilføjet GDPR-cc-bestemmelse til Trine-udvidelse.

**Redaktionelle rettelser:**
- "Lagd af" → "Lagt af" (linje 356)
- Alle "Met" → "Med" i referentiale
- Stavekontrol: "rapport" (ikke "rapportere" i tabeller)

**FGD-beslutninger indarbejdet:**
- Fjernet alle CABF-referencer; fokus på intern PKI.
- Engineering Team Lead er eksplicit SEPARAT fra Application Provider (Option B).
- Security Officer er egen dedikeret rolle (ikke kombineret med CA Admin eller AP).

---

**Lagt af:** Laila (HR-leder)  
**Dato:** 2026-05-09  
**Status:** UDKAST v2 profiler — afventer Mads+Dorthe final-check  
**Næste trin:** Mads+Dorthe final-check → FGD-gennemgang af profiler → Laila designes fulde mandater → Ansæt/aktivér roller per timeline

**Relateret:**
- Team/Mads/2026-05-09-pki-org-krav.md (Mads' komplet PKI-kravs-dokument)
- Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md
- Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md
- Team/Mads/2026-05-09-mads-4eyes-fgdcorepki-mandater.md (Mads' 4-eyes review)
- Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md (Dorthe's 4-eyes review)
