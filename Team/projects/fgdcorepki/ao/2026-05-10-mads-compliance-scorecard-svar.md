---
dato: 2026-05-10
fra: Mads (CISO)
til: Application Owner (FGDCorePKI)
cc: Stefan (orkestrator), Dorthe (DPO)
emne: FGDCorePKI Fase 0 — compliance-scorecard-svar
projekt: FGDCorePKI
type: compliance-scorecard-svar
---

# FGDCorePKI Fase 0 — Compliance-scorecard-svar

AO,

Hermed Mads' vurdering af de 8 scorecard-punkter for Fase 0. Compliance-konklusioner ejes af Mads (ISO 27001:2022, SOC 2 Type II, FIPS 140-3) og Dorthe (GDPR). AO og AP erklærer ingen af nedenstående punkter "compliant" på vegne af Mads.

Scorecarden er baseret på AP's tekniske arkitektur-dokument (`Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md`) samt AO's anmodning. Tre tværgående noter behandles efter de 8 punkter.

---

## Samlet verdict (toplinje til Stefan/FGD)

**CONDITIONAL GO på alle 8 punkter — ingen STOPs.**

Fase 0-arkitekturen er velfunderet. Betingelserne nedenfor er forudsætninger for ceremony-godkendelse, ikke blokkere for design-arbejdet nu. Alle betingelser er opløselige inden første ceremony.

---

## Punkt 1 — SOD-design

**Verdict: CONDITIONAL GO**

**Begrundelse:** Design-niveau SOD er solid. AP's §2.2 definerer seks authentication-keys med capability-isolation og Domain-separation på HSM-niveau, hvilket teknisk håndhæver SOD maskinelt (en CA Admin-session kan ikke udstede slette-operationer, en OCSP-responder-session kan ikke tilgå Root-Domain). SOD-rollerne AO, AP, Engineering Team Lead (ETL) og CA Administrator er som udgangspunkt distinkte. Kontroller: SOC 2 CC6.1 (logical access), CC6.2 (provisioning), ISO 27001:2022 A.5.3 (SoD), A.5.18 (adgangsrettigheder).

**Betingelse — manglende rolle:** AP's Tier 1 ceremony-design (§1.2) kræver fire-personers dual-control: CA Admin + HSM-operatør + Security Officer + KCO. **Security Officer er ikke i AO's nuværende onboarding-plan** og optræder ikke som formaliseret rolle i scorecard-anmodningen. Security Officer er heller ikke i auth-key-mapping som separat identitet — rollen er nævnt i ceremony-tabellen, men AO's rolleliste har kun 6 poster der ikke inkluderer Security Officer eksplicit. Betingelse: AO formaliserer Security Officer som distinkt rolle med separat authentication-key, separat identitet (må ikke være samme person som CA Admin, HSM-operatør eller KCO), og planlagt onboarding parallelt med øvrige roller. Ceremony kan ikke eksekveres som designet uden denne rolle på plads.

**Afklaringsspørgsmål til AO (SOD-scenarie):** AO spørger specifikt om scenarier for midlertidig overlap. Mads' svar: ved forfald af én SOD-kritisk rolle (fx HSM-operatør) er proceduren suspension af drift-operationer der kræver den rolle — ikke midlertidig konsolidering i en anden rolle. Undtagelse kræver skriftlig Mads + FGD-godkendelse per incident. Ingen "temp SOD-undtagelse" er standard-procedure.

---

## Punkt 2 — CP/CPS-scope og policy-OID-strategi

**Verdict: CONDITIONAL GO**

**Begrundelse:** Tre-lags hierarki (Root offline / Intermediate HA-par / Issuing online) er teknisk veldefineret i AP's §1. CP/CPS-dokumentet, der formaliserer dette juridisk og operationelt, er endnu ikke skrevet — AP §7.3 milestone 1 identificerer korrekt CP/CPS-draft som pre-ceremony-forudsætning. ISO 27001:2022 A.5.31 (juridiske krav), A.8.9 (konfigurationsstyring) og SOC 2 CC8.1 (change management) kræver at policy-dokumentet er godkendt og versionstyret inden ceremony.

**Betingelse — CP/CPS-indhold:** CP/CPS-udkastet skal som minimum dække: (a) alle tre tiers' certificate profiles og validity-vinduer (inkl. åbent spørgsmål O3 om Root validity 10-12 vs. 15-20 år, som skal afgøres inden CP/CPS-draft færdiggøres), (b) WORM/7-år retention-policy som normativt krav, (c) ceremony-procedure som normativt krav med reference til ceremony-script, (d) revocation-procedure (OCSP + CRL-frekvenser), (e) HSM-capability-mapping (AP §2.2 — input til Mads + HSM-operatør per O10), (f) Dual-control-krav eksplicit for alle Tier 1-operationer. Mads godkender CP/CPS inden ceremony-script-godkendelse.

**OID-strategi:** Mads anbefaler at FGD registrerer eget **IANA Private Enterprise Number (PEN)** som OID-arc-rod. Privat-PKI kan teknisk fungere med private-arc (1.3.6.1.4.1.xxx), men PEN-registrering er gratis, permanent og fremtidssikker ved evt. public-trust-overgang (O4). Uden registreret PEN er der risiko for OID-kollision ved fremtidig integration. Anmodning om PEN-registrering er AO's ansvar; processen tager typisk 2-4 uger og er ikke en ceremony-blocker hvis startet nu.

---

## Punkt 3 — HSM-procurement og FIPS-transition

**Verdict: CONDITIONAL GO**

**Begrundelse:** Algoritme-baseline (ECDSA P-384, AES-256-CCM, SHA-384/512, RSA 4096 sekundær) er fuldt FIPS-approved (NIST SP 800-131A Rev 2, FIPS 197, FIPS 180-4). AP's §2.6 dokumenterer FIPS-ready design: ingen FIPS-forbudte algoritmer, ceremony-procedure der understøtter re-keying til FIPS-variant uden CP/CPS-ændringer. Kontrol: ISO 27001:2022 A.8.24 (kryptografi-policy), FIPS 140-3 readiness.

**Betingelse — spare-HSM og procurement-lead-time:** Tre-tiers kræver minimum 4 YubiHSM2-enheder (1× Root, 2× Intermediate HA-par, 1× Issuing) + mindst 1 spare per tier til break-glass-restore (total: 7 enheder anbefalet inden ceremony). Procurement-lead-time for YubiHSM2 kan variere; AO skal bekræfte at bestilling er afgivet med tilstrækkelig buffer til ceremony-deadline samt at firmware-version verificeres mod Yubico advisory-liste inden produktiv brug.

**Betingelse — FIPS re-keying trigger-policy:** AP §2.6 noterer O6 (CMVP-grant-trigger) som Mads-overvågning. Mads accepterer dette ansvar. Betingelse: AO og Mads formaliserer trigger-policy-dokument (kan være et afsnit i governance-dokumentet eller separat note i `Team/Mads/`) der definerer: (a) hvornår Mads erklærer re-keying-event åbnet, (b) FGD-go-krav, (c) change-management-kategori (CC8.1 major change), (d) kommunikationsplan til aktive cert-aftagere. Dette dokument er ikke ceremony-blocker men skal foreligge inden Fase 1.

**Shamir-depot forudsætning:** AP §2.3 kræver "ekstern juridisk depot" som femte Shamir-share-holder. Mads noterer at denne leverandør/part endnu ikke er identificeret. Dette er en pre-ceremony-blocker — fem-personers Shamir-split kan ikke gennemføres uden den femte part. AO skal identificere leverandør og indgå nødvendig NDA/depot-aftale inden ceremony-script-godkendelse.

---

## Punkt 4 — Audit-log-arkitektur

**Verdict: CONDITIONAL GO**

**Begrundelse:** WORM/Merkle-tree-design med 7-års retention på issuance-events er stærkt og compliant med SOC 2 CC7.2 (monitoring) og ISO 27001:2022 A.8.15 (logging). Tre-lags tamper-evidence (HSM-internal signed log + cryptographic chained log + WORM-persistens) overstiger hvad de fleste PKI-implementeringer leverer. AP's log-kilde-matrice (§5.1) dækker alle nødvendige audit-trails inkl. PACS og ceremony-protokol. Kontroller: SOC 2 CC7.2, ISO 27001:2022 A.8.15, A.8.16, GDPR art. 32 (tekniske sikkerhedsforanstaltninger).

**Betingelse — Dorthe-sign-off på PII-minimering:** Audit-log indeholder PII i mindst fire kategorier: authentication-key-ID (korrelerer til operatør-ID), Subject-DN i issuance-log, requester-ID i RA-portal-log, klient-IP i OCSP/CRL-request-log. Dorthe ejer GDPR art. 5(1)(c)-vurdering (dataminimering) og art. 32-vurdering for disse. Mads' betingelse: Dorthe skriftligt godkender log-scope og PII-håndtering inden go-live. Koordinering sker parallelt.

**Betingelse — OCSP-log-retention låses:** Retention-vinduet er åbent på 30-90 dage (§5.3). Mads anbefaler 30 dage som udgangspunkt (tilstrækkeligt for DDoS-detection og incident-response; minimerer PII-eksponering per art. 5(1)(e)). Forlængelse til 90 dage kræver dokumenteret operationelt behov godkendt af Mads + Dorthe. Konkret værdi låses inden go-live.

**Betingelse — WORM-provider DPA:** S3 Object Lock (EU-region) eller on-prem WORM-appliance (NetApp SnapLock) er begge acceptable. Valget skal foreligge med underskrevet DPA inden go-live. Leverandøren skal garantere at ingen administratorrolle (inkl. AP, Camilla, FGD) kan slette inden retention-udløb — Object Lock compliance-mode opfylder dette; governance-lock-tagging verificeres af Trine ved første audit.

**Betingelse — Trine-rolle-aktivering:** AP §5.4 + §2.2 forudsætter Trine som aktiv auditor med `auth-auditor`-key på HSM. Trine er endnu ikke fuldt aktiveret som permanent rolle. Indtil Trine er på plads: kvartalvis log-review kan ikke udføres med den nødvendige audit-uafhængighed. Mads flag dette som koordineringspunkt til Stefan: Trine-aktivering skal koordineres inden first go-live for at SOC 2 CC7.2-evidens kan indsamles uafhængigt fra dag 1.

---

## Punkt 5 — Ceremony-procedure-godkendelse

**Verdict: CONDITIONAL GO (proces-commit)**

**Begrundelse:** Mads bekræfter at ceremony-procedure-godkendelse er en hard gate per governance-mandat. Ingen KCO-eksekvering uden Mads' skriftlige godkendelse. ISO 27001:2022 A.5.31, A.8.9 og SOC 2 CC8.1 kræver dokumenteret change-autorisation for PKI-ceremony som kritisk infrastruktur-etablering.

**Godkendelsesproces og forudsætninger:** Mads godkender ceremony-script senest 5 arbejdsdage inden eksekvering, forudsat at alle nedenstående er på plads:

1. CP/CPS-udkast godkendt af Mads (se P2)
2. DPIA-1-konklusion (F1, F5, F9) foreligger fra Dorthe
3. Datacenter-tier + safe-grade besluttet (O7, Mads + FGD)
4. Network zone-model A-F formaliseret (O9, Mads + Engineering Lead)
5. CA-software-valg truffet (O5, Poul Track A → Mads → FGD)
6. HSM-throughput-test gennemført af Engineering Lead
7. Security Officer-rolle formaliseret og onboarded (se P1)
8. Ekstern juridisk depot (Shamir 5. share) identificeret og NDA underskrevet (se P3)
9. AP's ceremony-script-udkast gennemgået af KCO og leveret til Mads

**Deadline-ansvarlighed:** AO styrer ceremony-deadline. Mads' 5-arbejdsdages frist starter fra det tidspunkt alle 9 forudsætninger er opfyldt og script er i Mads' inbox.

---

## Punkt 6 — OCSP/CRL EU-only hosting (Scenario B)

**Verdict: GO (Mads' ISO 27001/SOC 2-domæne) — GDPR art. 44-49 refer til Dorthe**

**Begrundelse (Mads' domæne):** AP §4 implementerer EU-only-residency (L1) stringent og konsekvent. Tabellen i §4.1 definerer acceptable og ikke-acceptable leverandører eksplicit. OCSP-responder placeres i Zone E (DMZ EU-only); CRL Distribution Points hoste på EU-residency-garanteret infrastruktur; sub-processor-klausul kræver DPA-garanti fra leverandør. Designet er teknisk robust: OCSP-stapling som primær reducerer direkte responder-eksponering og DDoS-overflade markant; delta-CRL fra dag 1 håndterer mass-revocation-events. Kontroller: ISO 27001:2022 A.5.34 (privatlivs-beskyttelse), A.8.9 (konfigurationsstyring), SOC 2 CC6.1.

**HA-robusthed:** To-HSM HA-par i Tier 2 med kvartalvis failover-test er tilstrækkeligt for Intermediate-niveau. Tier 3 single-HSM (Fase 0) er en accepteret risiko med dokumenteret RTO-estimat (4-24 timer, O8) — acceptabelt for Fase 0 privat-PKI; Fase 1 HA-udvidelse er korrekt prioriteret.

**Betingelse — konkret leverandør + DPA:** GO er betinget af at en konkret EU-leverandør (med verificeret EU-only-konfiguration og tilgængelig DPA) vælges og DPA underskrives inden go-live. Dorthe ejer DPA-vurdering; AP leverer teknisk-fit-input (O13). EU-CDN-shortlist (Bunny.net EU-zones, Hetzner Cloud Storage, Scaleway Object Storage) er acceptable teknisk — Dorthe verificerer CLOUD Act-eksponering og GDPR art. 44-49 for den valgte leverandør.

**GDPR art. 44-49-vurdering:** Refer til Dorthe. Mads noterer at per FGD-beslutning 2026-05-10 (DPIA-2 lukket Scenario B) er EU-only et hard requirement — art. 44-49-spørgsmålet er derfor om Scenario B's leverandør-kontraktion er tilstrækkelig, ikke om kravet skal lempes.

---

## Punkt 7 — Key Ceremony Officer — rolleprofil

**Verdict: CONDITIONAL GO**

**Begrundelse:** KCO er en kritisk SOD-funktion. KCO er den eneste rolle der eksekverer ceremony-script-procedurerne — hverken AP, AO eller CA Admin eksekverer selv. Mads' godkendelse af KCO-profil er nødvendig inden onboarding per governance-mandat. Kontroller: ISO 27001:2022 A.6.1 (screening), A.5.3 (SoD), SOC 2 CC6.2 (onboarding).

**Minimum-kvalifikationskrav (Mads' anbefaling til Laila):**

- Dokumenteret erfaring med PKI key ceremony (minimum én gennemført Root CA-ceremony som aktiv deltager eller leder)
- Erfaring med HSM-håndtering (YubiHSM2 eller tilsvarende — PKCS#11, audit-log-eksport, wrap-key-administration)
- Ren straffeattest (kriminalregister — dansk eller relevant national)
- Baggrundstjek gennemført (reference-check minimum, evt. udvidet ved FGD's vurdering)
- Signeret NDA + rolle-acceptance-dokument inden ceremony-adgang
- **SOD-absolut krav:** KCO må ikke simultaneously varetage rollen som CA Administrator, HSM-operatør eller Security Officer — hverken på Fase 0-ceremony-dato eller i efterfølgende drift

**Deadline:** KCO-profil godkendt af Mads minimum 4 uger inden planlagt ceremony for at give tilstrækkelig rekrutteringstid. Laila designer rolle-format; Mads godkender indhold.

---

## Punkt 8 — Engineering Team Lead og AP — SOD Fase 0

**Verdict: GO**

**Begrundelse:** Option B (ETL separat fra AP fra dag 1) er den korrekte SOD-posture og Mads bekræfter dette som governance-krav. AP ejer arkitektur-design; ETL ejer build-/release-plan og implementerings-specificering. Sammenblanding ville bryde ISO 27001:2022 A.5.3 (SoD) og SOC 2 CC6.1 (logical access) ved at én rolle kontrollerede både design og implementering af kritisk sikkerhedsinfrastruktur.

**Ingen midlertidig undtagelse:** AO spørger om "ingen midlertidig undtagelse" er governance-kravet. Bekræftet. Hvis ETL-rollen ikke er besat på ceremony-datoen, er konsekvensen at implementerings-fasen (milestone 1-7 i AP §7.3) udskydes — ikke at AP overtager ETL-funktioner midlertidigt. AO eskalerer til Stefan/FGD hvis ETL-bemanding er truet.

---

## Tværgående noter (tre blinde vinkler — Mads flag eksplicit)

### T1 — `auth-hsm-operator` delete-capability og Mads' hard rule

AP §2.2 tildeler `auth-hsm-operator` capabilities inkl. `delete-asymmetric-key`. Dette aktiverer mulighed for destruktiv HSM-operation. Per Mads' governance-mandat er destruktive HSM-operationer (herunder `delete-object`, `factory-reset`, `delete-asymmetric-key`) underlagt **Mads + FGD-godkendelse** inden eksekvering.

**Krav til CP/CPS og ceremony-script:** `delete-asymmetric-key`-aktivering af `auth-hsm-operator` authentication-key er teknisk nødvendig (backup-restore, key-rotation). Den må kun aktiveres via dokumenteret break-glass-procedure med skriftlig pre-autorisation fra Mads + FGD. Rutinemæssig brug af delete-capability er forbudt. HSM-operatørens break-glass-procedure dokumenteres eksplicit i CP/CPS §6 og i HSM-operatør-runbogen. Trine auditerer break-glass-log halvårligt.

### T2 — Ekstern juridisk depot (Shamir 5. share) er pre-ceremony-blocker

AP §2.3 specificerer Shamir 3-of-5 split med "ekstern juridisk depot" som femte share-holder. Denne part er ikke identificeret i AO's roadmap eller scorecard-anmodningen. Mads betragter dette som en **hård pre-ceremony-blocker** — den praktiske konsekvens er at Tier 1 key ceremony ikke kan gennemføres med designet split-arkitektur uden denne part.

Acceptabelt alternativ: Mads godkender 4-of-5 split med fire internt/FGD-tilknyttede share-holdere, forudsat at de fire parter er geografisk adskilt og ingen enkelt part kan samle tre shares. En sådan ændring kræver CP/CPS-ajourføring og Mads-godkendelse. AO bekræfter præference inden CP/CPS-draft.

### T3 — Trine-aktivering som SOC 2-forudsætning

AP §5.4 og §2.2 (auth-key-mapping) forudsætter Trine som aktiv, uafhængig auditor med `auth-auditor`-key og read-only SIEM-adgang. Trine er endnu ikke fuldt aktiveret som permanent rolle.

Mads flag til Stefan: SOC 2 CC7.2 (monitoring of controls) kræver uafhængig audit-funktion. Hvis Trine ikke er aktiv ved go-live, kan kvartalvis log-review og halvårlig break-glass-audit ikke gennemføres med tilstrækkelig uafhængighed. Mads' anbefaling: Stefan koordinerer Trine-aktivering som Fase 0-milestone parallelt med øvrige onboardings — ikke som Fase 1-opgave.

---

## Dorthe-koordinering (Mads' videreformidling)

Per AO's koordinerings-note formidler Mads følgende til Dorthe (cc i dette svar):

- **DPIA-1 F1/F5/F9** er pre-ceremony-forudsætning (P5 betingelse 2). AP har leveret teknisk input; Dorthe ejer konklusion.
- **Audit-log PII-minimering** (T i P4): Dorthe-sign-off afventes inden go-live.
- **OCSP/CRL-hosting GDPR art. 44-49** (P6): Dorthe bekræfter GDPR-compliance for valgt EU-leverandør.
- **ROPA-oprettelse** for FGDCorePKI-aktiviteter (cert-issuance, OCSP/CRL, audit-log, ceremony-recording): AO har noteret til Dorthe. Camilla ejer fil-oprettelse; Dorthe attesterer.
- **OCSP-log-retention endelig værdi** (P4): Mads + Dorthe låser inden go-live.

---

## Åbne spørgsmål til AO/AP (Mads' returnering)

| # | Spørgsmål | Modtager | Frist-forslag |
|---|---|---|---|
| M1 | Security Officer formaliseres som distinkt onboarding-rolle med separat authentication-key — AO bekræfter plan? | AO | Inden CP/CPS-draft |
| M2 | OID-strategi: PEN-registrering (anbefalet) eller privat-arc? AO beslutter | AO | Inden CP/CPS-draft |
| M3 | Shamir 5. share: ekstern juridisk depot identificeret? Alternativt: 4-of-5 split? | AO + FGD | Pre-ceremony-blocker |
| M4 | Procurement-status: er alle 4+ HSM-enheder + spares bestilt? Firmware-verifikation planlagt? | AO | Bekræftelse til Mads inden ceremony-datum fastsættes |
| M5 | Root validity åbent (O3): 10-12 år vs. 15-20 år. AO bringer til FGD-beslutning for CP/CPS-draft | AO | Inden CP/CPS-draft |
| M6 | KCO-rekruttering: Laila kontaktes for rolle-design; AO bekræfter 4-ugers lead-time ift. ceremony-plan | AO + Laila | Straks |
| M7 | Trine-aktivering: Stefan koordinerer. AO notificerer Stefan om Fase 0-dependency | AO → Stefan | Straks |

---

*Mads (CISO), 2026-05-10*
*Compliance-konklusioner i dette dokument ejes af Mads (ISO 27001:2022, SOC 2 Type II, FIPS 140-3) og Dorthe (GDPR). AO og AP erklærer ingen compliance-status på egne vegne.*
*Næste step: Stefan forwarder scorecard til FGD. FGD's GO på Fase 0 forudsætter at betingelserne i P1-P8 og T1-T3 adresseres inden ceremony-godkendelse.*
