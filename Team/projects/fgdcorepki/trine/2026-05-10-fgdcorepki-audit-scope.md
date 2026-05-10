---
dato: 2026-05-10
type: audit-scope
projekt: FGDCorePKI
forfatter: Trine (compliance-auditor)
compliance-frameworks: [SOC 2 Type II, ISO 27001:2022]
status: initial-scope
---

# FGDCorePKI — Initial audit-scope (Fase 0)

**Formål:** Dette dokument definerer Trines audit-scope for FGDCorePKI Fase 0. Det er et verificerings-grundlag — ikke en compliance-konklusion. Compliance-status kan først fastslås når evidens er indsamlet efter go-live. Mads ejer kontrol-design; AP og øvrige projektroller implementerer; Trine verificerer.

**Kildemateriale for dette scope-dokument:**

- `Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md` (AP's tekniske arkitektur)
- `Team/projects/fgdcorepki/ao/2026-05-10-pki-produkt-roadmap-fase0.md` (AO's roadmap)
- `Team/projects/fgdcorepki/ao/2026-05-10-mads-compliance-scorecard-svar.md` (Mads' CONDITIONAL GO + T1-T3)

---

## 1. SOC 2 Type II — relevante Trust Service Criteria

Scope for FGDCorePKI Fase 0 er Security (CC) og Availability (A). Confidentiality-kriterierne (C1.1, C1.2) er ikke i dette scope-dokument men foreslås vurderet til SoA v2: PKI-key-material og audit-logs er klassisk C-territorium.

### 1.1 Security — CC6: Logical and Physical Access Controls

| Kriterium | Beskrivelse | FGDCorePKI-relevans | Anker |
|---|---|---|---|
| **CC6.1** | Logisk adgang begrænses til autoriserede brugere og tjenester | Six authentication-keys med capability-isolation og Domain-separation på YubiHSM2; session-timeout 30 min / 4 timer ceremony | AP §2.2 |
| **CC6.2** | Ny adgang provisioneres via godkendt process | Mads godkender alle rolle-onboardings (CA Admin, HSM-operatør, KCO, Security Officer); ingen ad-hoc-provisioning | Mads P7, AO §2 |
| **CC6.3** | Adgang fjernes ved rolleskifte/fratrædelse | Deprovisioning-procedure for authentication-keys på YubiHSM2; capability-reset ved rolleafslutning | Mads P1; AP §2.2 |

**Evidens Trine vil søge:** ceremony-protokol med navngivne deltagere og authentication-key-ID'er; Mads' skriftlige onboarding-godkendelser; log fra `auth-auditor`-key der viser hvem der har tilgået hvilke HSM-domains.

### 1.2 Security — CC7: System Operations

| Kriterium | Beskrivelse | FGDCorePKI-relevans | Anker |
|---|---|---|---|
| **CC7.2** | Anomalier og hændelser detekteres og undersøges | HSM internal log eksporteres hver time; Merkle-chained WORM-log for alle CA-events; break-glass-log halvårlig Trine-audit | Mads T3; AP §5.1-5.2 |
| **CC7.3** | Opdagede hændelser eskaleres og håndteres | Break-glass-aktivering kræver Mads + FGD-pre-autorisation; hsm-operator-runbook definerer incident-procedure | Mads T1; AP §2.2 |

**Evidens Trine vil søge:** SIEM read-only-adgang (afventer platform-valg, O11); WORM-bucket governance-lock-tag verifikation; break-glass-log-integritet (hash-kæde-check); halvårlig `get-log-entries`-eksport via `auth-auditor`-key.

### 1.3 Security — CC8: Change Management

| Kriterium | Beskrivelse | FGDCorePKI-relevans | Anker |
|---|---|---|---|
| **CC8.1** | Infrastrukturændringer godkendes og testes inden produktion | Ceremony-procedure kræver skriftlig Mads-godkendelse minimum 5 arbejdsdage inden eksekvering; CP/CPS versionsstyres; CA-software-valg godkendes Poul Track A → Mads → FGD | Mads P5; AP §7.3 |

**Evidens Trine vil søge:** Mads' skriftlige ceremony-godkendelse (dateret); CP/CPS versionsnummer og godkendelses-signatur; ændringslogs for CA-software-konfiguration.

### 1.4 Availability — A1: Availability

| Kriterium | Beskrivelse | FGDCorePKI-relevans | Anker |
|---|---|---|---|
| **A1.1** | Availability-commitments er defineret og kommunikeret | RTO/RPO-mål for Tier 3 single-HSM (foreløbig 4-24 timer, O8) formaliseres af Mads + FGD | Mads P6; AP §1.4 |
| **A1.2** | Systemet er tilgængeligt per commitment | Tier 2 HA-par (2× YubiHSM2) med kvartalvis failover-test; spare-HSM til Tier 3 break-glass-restore under 1 time; backup-wrapped-keys på offline media | Mads P6; AP §1.3 |

**Evidens Trine vil søge:** kvartalvis failover-test-rapport (Tier 2); dokumenteret RTO/RPO-mål (O8-afklaring); backup-wrapped-key inventory med chain-of-custody-protokol; spare-HSM-lagerplan.

---

## 2. ISO 27001:2022 — relevante Annex A-kontroller

Nedenstående er de primære in-scope-kontroller for FGDCorePKI Fase 0. Listen er ikke udtømmende; den udvides i takt med at systemet modnes og SoA for knowlagecentral-projektet tilpasses.

### 2.1 A.5.3 — Separation of Duties (SoD)

**Beskrivelse:** Ansvar og opgaver adskilles for at reducere risiko for uautoriseret eller utilsigtet ændring.

**FGDCorePKI-relevans:** Seks HSM authentication-keys med capability-isolation og Domain-separation håndhæver SoD maskinelt. Ceremony kræver fire separate personer (CA Admin + HSM-operatør + Security Officer + KCO). AO og AP er aldrig samme person. AP og ETL er aldrig samme person (Option B).

**Evidens Trine vil søge:**
- Ceremony-protokol: fire navngivne, distinkte personer med separate authentication-key-ID'er bekræftet via HSM-log
- Mads' skriftlige godkendelse af onboarding for alle SOD-kritiske roller
- Organisationsoversigt der dokumenterer at ingen person varetager to SOD-kritiske roller simultant
- `auth-auditor`-key log: ingen `auth-hsm-operator`-session har udstedt signaturer der tilhører `auth-ica-admin`-domæne

**Anker:** Mads scorecard P1, P8; AP §2.2.

### 2.2 A.5.18 — Access Rights (adgangsrettigheder)

**Beskrivelse:** Adgangsrettigheder tildeles, revideres og tilbagetrækkes systematisk.

**FGDCorePKI-relevans:** Alle HSM authentication-keys provisioneres med eksplicit capability-liste og Domain-tilhørsforhold (least privilege). `auth-auditor`-key giver Trine read-only (`get-log-entries`, `get-object-info`) uden skriveadgang.

**Evidens Trine vil søge:**
- YubiHSM2 `get-object-info` for hver authentication-key: bekræfter Domain, Capabilities matcher AP §2.2-tabellen
- Adgangsrevision: kvartalsvis — ejer (HSM-operatør under Mads' mandat) leverer liste; Trine verificerer mod godkendt konfiguration
- Bevis for at ingen authentication-key er tildelt capabilities ud over defineret sæt

**Anker:** AP §2.2; Mads P1; SOC 2 CC6.1.

### 2.3 A.5.31 — Juridiske, lovgivningsmæssige og kontraktlige krav

**Beskrivelse:** Lovgivnings- og kontraktmæssige krav identificeres og overholdes.

**FGDCorePKI-relevans:** CP/CPS-udkast formaliserer juridiske krav til PKI-drift. EU-only OCSP/CRL-hosting (L1) er hård beslutning med GDPR art. 44-49-dimension. GDPR-koordinering sker via Dorthe (DPO).

**Evidens Trine vil søge:**
- CP/CPS-dokument med Mads' signatur + versionsnummer
- DPA med OCSP/CRL-leverandør (EU-only-garanti; afventer Dorthe-godkendelse, O13)
- Mads' bekræftelse af at Root validity-beslutning (O3) er afspejlet i CP/CPS

**Anker:** Mads P2; AP §4.1.

### 2.4 A.8.9 — Konfigurationsstyring

**Beskrivelse:** Konfiguration af hardware, software og services dokumenteres, godkendes og vedligeholdes.

**FGDCorePKI-relevans:** YubiHSM2-konfiguration (authentication-keys, Domain-mapping, capability-sæt) dokumenteres i CP/CPS §6. CA-software-valg godkendes via formel proces (Poul Track A → Mads → FGD). Ændringer til HSM-konfiguration kræver change-management-autorisation (CC8.1).

**Evidens Trine vil søge:**
- `yubihsm-shell list-objects` output pr. HSM-enhed sammenlignet med konfigurationsbeskrivelse i CP/CPS §6
- Versionshistorik for CP/CPS
- Change-log for CA-software-konfigurationsændringer med godkendelsesstempel

**Anker:** Mads P2; AP §2.4; SOC 2 CC8.1.

### 2.5 A.8.10 — Sletning af information (key destruction)

**Beskrivelse:** Information lagret i systemer, enheder og medier slettes sikkert.

**FGDCorePKI-relevans:** `delete-asymmetric-key`-capability på `auth-hsm-operator`-key er en destruktiv operation. Per Mads T1 kræver aktivering af denne capability forudgående skriftlig autorisation fra Mads + FGD. Rutinemæssig brug er forbudt.

**Evidens Trine vil søge:**
- Break-glass-log: enhver forekomst af `delete-asymmetric-key` i HSM-log skal have korresponderende skriftlig pre-autorisation (dato + sign-off fra Mads + FGD)
- HSM-operatør-runbook: dokumenterer eksplicit at delete-capability kun aktiveres via break-glass-procedure
- Halvårlig Trine-gennemgang: cross-check HSM-log mod autorisationslog for ingen uautoriserede sletninger

**Anker:** Mads T1; AP §2.2.

### 2.6 A.8.12 — Forebyggelse af datalækage

**Beskrivelse:** Foranstaltninger implementeres for at forhindre uautoriseret eksponering af data.

**FGDCorePKI-relevans:** WORM-persistens og cryptographic chained log sikrer at audit-evidens ikke kan manipuleres. Ingen authentication-key (inkl. `auth-hsm-operator`) kan slette WORM-arkiveret evidens inden retention-udløb. EU-only OCSP/CRL-hosting forhindrer tredjelands-eksponering af PKI-trafik.

**Evidens Trine vil søge:**
- S3 Object Lock governance-mode-tag eller tilsvarende WORM-mekanisme verificeret aktiv via AWS/leverandør-console-screenshot eller API-output (read-only)
- Bekræftelse af at ingen admin-rolle (inkl. Camilla, FGD) kan slette inden retention-udløb — leverandørens compliance-mode-dokumentation
- OCSP/CRL-leverandørens DPA med EU-only-klausul (afventer Dorthe, O13)

**Anker:** Mads P4; AP §5.2.

### 2.7 A.8.15 — Logning

**Beskrivelse:** Logs der registrerer aktiviteter, undtagelser og sikkerhedshændelser produceres, beskyttes og analyseres.

**FGDCorePKI-relevans:** Tre-lags tamper-evidence: HSM internal signed log + Merkle-chained cryptographic log + WORM-persistens. Dækker alle log-kilder i AP §5.1 (HSM, CA-software, RA-portal, OCSP, CRL, OS, PACS, ceremony). Retention 7 år for issuance-events; 30-90 dage for OCSP/CRL-request-logs (Dorthe-koordinering afventer).

**Evidens Trine vil søge:**
- `get-log-entries` via `auth-auditor`-key: verificerer at alle signing-events og authentication-events er logget
- Hash-kæde-integritetscheck på Merkle-log (mindst halvårligt; ved incident ad hoc)
- Retention-konfiguration i SIEM og WORM-bucket verificeret mod godkendte værdier
- Ceremony-protokol (papir + screen-capture): arkiveret og tamper-evident

**Anker:** Mads P4; AP §5.1-5.3; SOC 2 CC7.2.

### 2.8 A.8.16 — Overvågningsaktiviteter

**Beskrivelse:** Netværk, systemer og applikationer overvåges for anomalier.

**FGDCorePKI-relevans:** SIEM-platform (afventer Mads + Engineering Lead valg, O11) aggregerer alle log-streams fra AP §5.1. Alarm ved CRL-størrelse > 5 MB. HSM-log-buffer 80%-alarm udløser manuel eksport.

**Evidens Trine vil søge:**
- SIEM-alarmkonfiguration: dokumenterede alert-regler for HSM-buffer-fylde, CRL-størrelse, uautoriserede adgangsforsøg
- Alert-historik (ingen tilgængelig ved Fase 0 start — baseline etableres)
- Bekræftelse af at Trine's SIEM read-only-rolle er provisioneret

**Anker:** AP §5.2; Mads P4.

### 2.9 A.8.24 — Brug af kryptografi

**Beskrivelse:** Regler for brug af kryptografi — nøglegenerering, distribution, opbevaring og destruktion — defineres og implementeres.

**FGDCorePKI-relevans:** Algoritme-konvolut: ECDSA P-384 primær, RSA 4096 sekundær, SHA-384/512 minimum, AES-256-CCM for wrap-key. Shamir 3-of-5 split for wrap-key med geografisk distribueret depot. Re-keying ved CMVP-grant (O6, Mads-overvågning). Ingen SHA-1, MD5, RSA-2048 i scope (L5).

**Evidens Trine vil søge:**
- `get-object-info` for alle CA-keys: Algorithm-felt bekræfter ECDSA P-384 eller RSA 4096 (aldrig P-256, RSA-2048, SHA-1)
- Ceremony-protokol: dokumenterer wrap-key-generation + Shamir-split-eksekvering (5 shares, 5 navngivne parter)
- Shamir-share-holders' bekræftelse: underskrevne kvitteringer for modtagelse og sikkert depot (geografisk distribueret)
- FIPS-readiness-erklæring: ingen forbudte algoritmer i produktionskonfiguration

**Anker:** AP §2.3, §2.6, §6; Mads P3.

### 2.10 A.6.1 — Screening

**Beskrivelse:** Baggrundstjek foretages for alle kandidater til ansvarlige stillinger.

**FGDCorePKI-relevans:** KCO kræver dokumenteret PKI ceremony-erfaring, HSM-erfaring, ren straffeattest og baggrundstjek. Alle SOD-kritiske roller (CA Admin, HSM-operatør, Security Officer) godkendes af Mads inden onboarding.

**Evidens Trine vil søge:**
- Mads' skriftlige godkendelse for hver SOD-kritisk rolle (dateret, refererer til rolle-profil)
- Bekræftelse fra AO at screening-krav i Mads' P7 er opfyldt (straffeattester, reference-check)
- Signerede NDA + rolle-acceptance-dokumenter for KCO og Security Officer

**Anker:** Mads P7; AO §2.

---

## 3. Evidens-krav — sammenfattet matrice

| Kontrol | Evidens-artefakt | Kilde-system | Frekvens | Trine-verifikation |
|---|---|---|---|---|
| CC6.1 / A.5.18 | `get-object-info` output pr. authentication-key | YubiHSM2 via `auth-auditor`-key | Kvartalvis + ved ændring | Sammenlign mod CP/CPS §6-konfiguration |
| CC6.2 / A.6.1 | Mads' skriftlige onboarding-godkendelser | Stefan/Mads-korrespondance | Per onboarding-event | Bekræft alle SOD-kritiske roller har dateret godkendelse |
| CC7.2 / A.8.15 | `get-log-entries` export + WORM-bucket inventory | YubiHSM2 + SIEM + WORM-storage | Halvårlig (break-glass-audit) + kvartalvis (log-review) | Hash-kæde-integritetscheck; governance-lock-tag bekræftet aktiv |
| CC7.3 / A.8.10 | Break-glass-log cross-check mod pre-autorisationslog | WORM break-glass-arkiv + Mads/FGD sign-off | Halvårlig | Ingen `delete-asymmetric-key` uden tilsvarende dateret Mads + FGD-autorisation |
| CC8.1 / A.8.9 | CP/CPS versionsnummer + Mads' ceremony-godkendelses-dato | CP/CPS-dokument + Stefan-inbox | Per ceremony + ved CP/CPS-ændring | Verificér at ceremony ikke er eksekveret uden forudgående skriftlig godkendelse |
| A1.2 | Failover-test-rapport (Tier 2 kvartalvis) + RTO/RPO-mål-dokument | HSM-operatør-runbook + testlog | Kvartalvis | Bekræft test gennemført + resultat dokumenteret; RTO/RPO-mål formaliseret |
| A.8.24 | `get-object-info` Algorithm-felt for alle CA-keys + ceremony-protokol | YubiHSM2 + KCO-arkiv | Ved ceremony + kvartalvis | Ingen forbudt algoritme; Shamir-share-holders-kvitteringer foreligger |
| A.5.3 | Ceremony-protokol: fire navngivne separate deltagere + HSM-log | KCO-arkiv + YubiHSM2 | Per ceremony | Cross-check: navne i protokol vs. authentication-key-ID i HSM-log |
| A.5.31 | CP/CPS med Mads-signatur + DPA med OCSP/CRL-leverandør | CP/CPS-arkiv + Dorthe/leverandør | Årlig review | Bekræft CP/CPS er aktuel + DPA dækker EU-only-krav |
| A.8.12 | S3 Object Lock compliance-mode-tag + leverandørens WORM-dokumentation | WORM-provider (afventer valg) + Mads P4-betingelse | Halvårlig | Governance-lock aktiv; ingen admin-rolle kan slette inden retention-udløb |
| A.8.16 | SIEM alert-konfiguration + alert-historik | SIEM (afventer platform-valg, O11) | Kvartalvis | Bekræft alarmregler dækker HSM-buffer, CRL-størrelse, uautoriseret adgang |

**GDPR-note:** Audit-artefakter der indeholder PII (klient-IP i OCSP-logs, Subject-DN i issuance-log, operatør-ID i HSM-log, witness-navne i ceremony-protokol) behandles under GDPR art. 6(1)(f) legitim interesse. Dorthe fastsætter præcis retention og godkender log-scope. Trine koordinerer med Dorthe inden PII-holdige evidensbaser tilgås for første gang. Dataminimeringsprincip: Trine indsamler kun hvad der er nødvendigt for at dokumentere kontrol-effektiviteten.

---

## 4. Audit-tidsplan

Trine kan ikke fastlægge konkrete datoer — tidsplanen er betinget af projektets milepæle. Nedenstående format er: **anker-milepæl → hvad Trine kan/skal gøre**.

| Audit-begivenhed | Afhænger af | Indhold | Output |
|---|---|---|---|
| **Trine aktiveres som permanent rolle** (T0-meta) | Stefan koordinerer Trine-aktivering (Mads T3, M7) | `auth-auditor`-key provisioneret på alle HSM-instanser; SIEM read-only-adgang etableret | Trine klar til at modtage evidens |
| **Soft internal audit 1** (~T0+30 dage efter Fase 0 go-live) | Fase 0 go-live: alle Mads P1-P8-betingelser opfyldt + ceremony gennemført | Verificér at audit-infrastruktur virker i praksis: `get-log-entries` fungerer, WORM-bucket tilgængelig, SIEM-adgang bekræftet | Audit-infrastruktur-rapport til Stefan (cc Mads) |
| **Formal internal audit 1** (~T0+90 dage) | Soft audit 1 gennemført + 90 dages observation-data | Kontrol-effektivitets-gennemgang for alle kontroller i §2; sampling af log-entries; cross-check ceremony-protokol mod HSM-log; break-glass-log-check (ingen events forventet på dette tidspunkt) | Månedsrapport 1 til Stefan (cc Mads): ✅/⚠/❌ pr. kontrol med evidens-citat |
| **Halvårlig break-glass-audit** (~T0+180 dage) | 180 dages WORM-data tilgængelig | Mads T1-krav: cross-check break-glass-log mod pre-autorisationslog; hash-kæde-integritetscheck; `delete-asymmetric-key`-verifikation | Break-glass-audit-rapport til Stefan |
| **SOC 2 Type II observation-periode start** | Formal internal audit 1 ren (ingen kritiske ❌) + Mads-godkendelse | Observation-vindue åbent: ekstern revisor observerer kontrol-virkelighed i minimum 6 måneder | Koordineres med Stefan og Mads; ekstern revisor-firm onboardes |
| **Kvartalvis adgangsrevision** (løbende fra T0) | `auth-auditor`-key aktiv | `get-object-info` pr. authentication-key; sammenlign mod CP/CPS §6-konfiguration; bekræft ingen capability-drift | Adgangsrevisions-notat til Stefan |

---

## 5. Blokerere — hvad mangler for at Trine kan auditere

### B0 — Meta-bloker: Trine ikke aktiveret som permanent rolle (Mads T3)

**Hvad blokeres:** Al audit-aktivitet. Uden `auth-auditor`-key og SIEM read-only-adgang kan Trine ikke tilgå HSM-logs eller SIEM-data. SOC 2 CC7.2-evidens kan ikke indsamles med tilstrækkelig uafhængighed.

**Ejer:** Stefan (koordinering) + Mads (flag til FGD).

**Eskaleringsvej:** Stefan eskalerer Trine-aktivering til FGD som Fase 0-milestone (jf. Mads M7: "straks"). AO notificerer Stefan.

**Konsekvens hvis uløst:** SOC 2 Type II observation-periode kan ikke starte med gyldig audit-funktion.

---

### B1 — Mads T1: `auth-hsm-operator` delete-capability ikke dokumenteret i runbook

**Hvad blokeres:** Trine kan ikke auditere break-glass-procedurens compliance (A.8.10, CC7.3). Halvårlig break-glass-audit (Mads T1-krav) kræver at runbook eksplicit definerer: hvornår delete-capability må aktiveres, hvem autoriserer (Mads + FGD), hvordan pre-autorisation dokumenteres, og at log-entry i WORM-arkivet korrelerer til autorisationspost.

**Ejer:** HSM-operatør (runbook-forfatter) + Mads (godkendelse af runbook-indhold). Trine verificerer — designerer ikke runbook.

**Eskaleringsvej:** Via Stefan til Mads. Runbook skal foreligge inden ceremony (Mads P5 betingelse — ceremony-script forudsætter documenteret procedure).

**Hvad Trine behøver fra runbook:** Eksplicit break-glass-afsnit der indeholder: (a) trigger-kriterier for delete-aktivering, (b) pre-autorisation-format (dateret, sign-off fra Mads + FGD), (c) log-correlation-procedure (WORM-entry ID matcher autorisationsdokument).

---

### B2 — Mads T2: Shamir 5. share juridisk depot ikke identificeret

**Hvad blokeres:** Trine kan ikke verificere A.8.24-compliance (Shamir 3-of-5 wrap-key-split) uden at de 5 share-holders og deres depotforhold er dokumenteret. Ceremony-protokollen skal indeholde 5 navngivne parters underskrevne kvitteringer — uden identificeret ekstern juridisk depot mangler én post.

**Ejer:** AO (identificerer leverandør og indgår NDA/depot-aftale) + Mads (godkender alternativ 4-of-5 split hvis eksternt depot ikke opnås, jf. Mads T2/M3).

**Eskaleringsvej:** AO bekræfter præference (ekstern depot vs. 4-of-5) til Mads inden CP/CPS-draft. Trine afventer Mads' godkendelse af valgt model.

**Konsekvens:** Denne bloker er pre-ceremony-bloker (Mads T2 eksplicit). Trine kan ikke auditere en ceremony der er gennemført uden at Shamir-arkitekturen er endeligt godkendt af Mads.

---

### B3 — WORM-provider DPA ikke indgået (Mads P4-betingelse)

**Hvad blokeres:** Trine kan ikke verificere WORM-immutabilitet (A.8.12, A.8.15) uden at WORM-provider er valgt og DPA er underskrevet. Governance-lock-tag-verifikation forudsætter at leverandøren er på plads og tilgængelig for read-only.

**Ejer:** Dorthe (DPA-vurdering) + AO (leverandørvalg med AP teknisk-fit-input, O13).

**Eskaleringsvej:** Via Stefan. Dorthe-godkendelse af DPA er pre-go-live-betingelse (Mads P4).

**Hvad Trine behøver:** Underskrevet DPA med EU-only-garanti + bevis for compliance-mode-aktivering (S3 Object Lock compliance-mode eller tilsvarende) inden første audit.

---

### B4 — SIEM-platform ikke valgt (O11)

**Hvad blokeres:** Trine's kvartalvise log-review og A.8.16-audit (SIEM alert-konfiguration) kræver at SIEM-platform er valgt og Trine's read-only-rolle provisioneret.

**Ejer:** Mads + Engineering Lead (platformsvalg, O11).

**Eskaleringsvej:** Via Stefan til Mads. Ikke en ceremony-bloker, men en go-live-forudsætning for SOC 2 CC7.2-evidens fra dag 1.

---

## 6. Koordinering med øvrige roller

| Rolle | Koordineringspunkt | Trine-handling |
|---|---|---|
| **Mads (CISO)** | Bekræftelse af kontrol-krav; godkendelse af ceremony-procedure (P5); break-glass-autorisation (T1) | Trine modtager kopi af godkendelser; verificerer at de foreligger inden audit |
| **Dorthe (DPO)** | PII-holdige audit-artefakter (OCSP-logs, issuance-logs, ceremony-protokol); WORM-provider DPA | Trine koordinerer med Dorthe inden første PII-holdig evidensbase tilgås; afventer Dorthes log-scope-godkendelse |
| **HSM-operatør** (projekt-rolle) | `get-log-entries`-eksport; runbook break-glass-afsnit; kvartalvis failover-test-rapport | Trine læser — HSM-operatør ejer ikke Trines adgang (auth-auditor-key er separat) |
| **Camilla (infra)** | CI-logs, backup-logs, WORM-bucket inventory | Camilla leverer raw data; Trine analyserer |
| **Stefan (orkestrering)** | Eskalering af blokerere (B0-B4); modtager månedsrapporter | Trine rapporterer til Stefan primært |

---

**Næste skridt fra Trine:** Dette scope-dokument er Trines indgang til FGDCorePKI-audit-arbejdet. Trine kan ikke producere evidens-baseret compliance-status (✅/⚠/❌) før B0 (Trine-aktivering) er løst og Fase 0 go-live har fundet sted. Trine afventer Stefan's koordinering af aktiveringsforløbet.

*Trine (compliance-auditor), 2026-05-10*
*Initial scope — ikke en compliance-konklusion. Evidens-indsamling starter ved Fase 0 go-live.*
