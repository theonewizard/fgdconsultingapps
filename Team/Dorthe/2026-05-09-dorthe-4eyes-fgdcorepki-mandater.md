---
dato: 2026-05-09
type: 4-eyes-vurdering
emne: FGDCorePKI AO+AP+PKI-roller mandatudkast — GDPR-perspektiv
vurderer: Dorthe (DPO)
status: STOP
basis-filer:
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md
kontekst:
  - Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md
  - Team/Mads/2026-05-09-pki-org-krav.md
  - Team/Dorthe/2026-05-08-dorthe-final-review-app-owner-provider.md (præcedens fra v2)
modtagere: FGD, Mads (CISO), Stefan (orkestrator), Laila (HR)
---

# Dorthe — 4-eyes vurdering: FGDCorePKI mandater (GDPR)

## Samlet vurdering

**STOP** — to af mine egne 5 blokerende rettelser fra det tidligere AO/AP-review (2026-05-08, status `GO`) er **ikke ført med over** i FGDCorePKI-versionerne. Det er en regression der ville undergrave min v2-bar hvis jeg accepterede CONDITIONAL GO. Derudover har RA-rollen — den eneste rolle med direkte CSR-/identitets-behandling af persondata — **ingen GDPR-sektion overhovedet**, og 20-års-WORM-arkivering af key-ceremony-video (med witnesses' biometri) mangler en proportionalitets-/DPIA-vurdering.

Begrundelse for STOP frem for CONDITIONAL GO: præcedensen fra 2026-05-08 satte art. 38(3)-blokken og §3.3-supersedance som **blocker-niveau**. Hvis disse må falde ud når en rolle udvides til nyt projekt, er v2-rettelserne reelt ikke mandatets niveau — de er rolle-instans-niveau, hvilket bryder med governance-låsen i `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md`.

Når regressionerne er rettet og RA-/ceremony-PII-fundene adresseret, forventer jeg GO. Anslået rettelse-arbejde for Laila: 2-4 timers redigering.

---

## AO-udkast — GDPR-fund

### Fund AO-1 (BLOKERENDE) — DPO-uafhængighed art. 38(3) er regredieret

**Reference:** Brief-punkt 4. Min final-check 2026-05-08 §"Punkt 2", linjer 91-96 i AO v2.

**Hvad der mangler:** AO v2-mandatet havde en eksplicit DPO-uafhængigheds-sektion med 4 bullets:
1. Ikke-instruktions-pligt (DPO modtager ikke instruks om GDPR-konklusioner)
2. Eskalationspath direkte til FGD (DPO behøver ikke gå gennem AO/AP)
3. Ingen filter-position (AO/AP videreformidler DPO-rapport ufiltreret)
4. Ikke-sanktionering (AO/AP må ikke straffe DPO for upopulære vurderinger)

I FGDCorePKI-AO-udkastet er sektionen "GDPR-relateret (fra Dorthe)" (linjer 84-87) reduceret til 2 bullets om CP/CPS-vejledning og eskalations-pligt ved tvivl. Art. 38(3)-blokken er væk.

**Konsekvens:** Mandatet er ikke længere art. 38(3)-konformt for et CA-Manager-aligned product owner. CA-kontekst gør dette **mere** kritisk, ikke mindre — kompromis af CA-nøgle udløser GDPR-breach-flow med 72-timers frist hvor DPO skal kunne eskalere uden filter.

**Rettelse:** Genindsæt 4-bullet-blokken fra AO v2 linjer 91-96. Tilpas formulering til PKI-kontekst hvor relevant (fx erstat "feature-flow" med "CP/CPS-ændring eller cert-profil"), men de 4 substantielle krav skal være intakte.

---

### Fund AO-2 (BLOKERENDE) — §3.3-supersedance / FGD-override 2026-05-06 er regredieret

**Reference:** Min final-check 2026-05-08 §"Punkt 5", linje 106 i AO v2.

**Hvad der mangler:** AO v2 refererede eksplicit at `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3 er superseded af FGD-override 2026-05-06 (Mads + Dorthe SAMMEN vurderer alle mandat-ændringer — ingen "Mads-engangs-undtagelser").

I PKI-AO-udkastet linje 91 er der kun: *"Alle mandat-ændringer følger formel governance-proces (jf. ...2026-05-06-mads-mandat-stabilitet-governance.md)."* — uden supersedance-noten. En fremtidig læser falder direkte tilbage på den oprindelige policy-tekst hvor §3.3 stadig giver Mads engangs-undtagelses-power.

**Rettelse:** Tilføj 1-linje-note: *"§3.3 er superseded af FGD-override 2026-05-06; Mads + Dorthe vurderer SAMMEN alle rolle-konfigurations-ændringer."*

---

### Fund AO-3 (CONDITIONAL) — DPO-eskalations-SLA er ikke harmoniseret

AO linje 124 (ansvarsfordeling-tabel) og linje 136 (hand-off): *"Dorthe-respons: 5 hverdage."* Men AP-mandatet linje 96 og linje 149 har også 5 hverdage. Det er fint i sig selv, men når AP forpligter sig til at notificere DPO **inden** 5 hverdage, og DPO-respons også er 5 hverdage, er den samlede latency potentielt 10 hverdage før et nyt PII-flow kan implementeres. Dette er acceptabelt for ordinære cases, men det skal eksplicit være dokumenteret at break-glass-flow (audit-log-anomali, mistanke om uautoriseret HSM-adgang) går uden om denne SLA — hvilket AP linje 99 dækker (24 timer direkte til Dorthe + Mads).

**Rettelse:** Ingen blokerende ændring. Men overvej at tilføje 1-linje i AO der spejler AP linje 99: *"Mistanke om breach (kompromitteret CA-nøgle, audit-log-tampering, RA-portal-bypass) eskaleres direkte til Dorthe + Mads inden 24 timer — uden om ordinær SLA."*

---

### Fund AO-4 (mindre) — Sub-processor-clausul ved tredjelands-integration

AO linje 81 dækker DPA-vurdering før ny tredjelandsservice (eksemplet er Langchain). For PKI-kontekst er tredjelands-vinkler mere subtile end "Langchain":
- OCSP-responder hostet bag CDN (Cloudflare US, Fastly US) — Pouls Vinkel 3+6 anbefaler dette
- CRL Distribution Point bag CDN — samme problem
- Eventuelle managed-CA-tjenester (Sectigo, DigiCert som operations-partner)

**Rettelse:** Udvid AO linje 81-eksemplet eller tilføj note: *"Eksempler omfatter tredjelandshostet OCSP/CRL CDN, managed-CA-leverandør, eller cert-management-SaaS uden EU-residency. Art. 44-49-vurdering kræves."*

---

## AP-udkast — GDPR-fund

### Fund AP-1 (BLOKERENDE) — DPO-uafhængighed art. 38(3) er regredieret

Identisk problem som AO-1. AP v2 havde 4-bullet-blokken på linjer 112-117 (per min final-check 2026-05-08). I FGDCorePKI-AP-udkastet linjer 94-99 er sektionen reduceret. Art. 38(3)-blokken mangler.

**Rettelse:** Genindsæt 4-bullet-blokken fra AP v2 linjer 112-117. PKI-kontekst gør dette særlig kritisk fordi AP designer det tekniske dataflow der kan introducere PII-behandlinger uden at AP vurderer GDPR — DPO skal kunne stoppe et design uden at AP/Owner kan filtrere eller forsinke vurderingen.

---

### Fund AP-2 (BLOKERENDE) — §3.3-supersedance / FGD-override 2026-05-06 er regredieret

Identisk problem som AO-2. AP linje 110: *"Alle mandat-ændringer følger formel governance-proces (jf. ...2026-05-06-mads-mandat-stabilitet-governance.md)."* — uden supersedance-noten.

**Rettelse:** Samme 1-linje-note som AO-2.

---

### Fund AP-3 (BLOKERENDE) — Tredjeland-trigger dækker ikke OCSP/CRL CDN

**Reference:** Brief-punkt 6.

AP linje 96 lister notifikations-triggere: *"ny third-party-CA-integrations, ny CSR-opbevarings-kategori, ændret certificate-retention-periode, ny audit-log-kategori, ny tredjelands-HSM-integration."* Listen mangler de mest sandsynlige tredjelands-vinkler i PKI-arkitektur per Pouls Vinkel 3+6:

- **OCSP-responder hostet bag tredjelands-CDN** (Cloudflare US, Fastly US, Akamai US/global) — direkte art. 44-49-overførsel hvis CDN'en logger requesting-IP (som er PII for cert-verifikator)
- **CRL Distribution Point bag tredjelands-CDN** — samme problem; CRL-fetches afslører hvilken klient der verificerer hvilket cert
- **Certificate Transparency-logs hosted i tredjeland** (hvis FGDCorePKI nogensinde går public-trust per Pouls Vinkel 3 → CT-logging hos Sectigo/Google/Cloudflare)
- **Cert-management-SaaS uden EU-residency** (Keyfactor, Venafi som mulige integrationer per Pouls Vinkel 9)

**Rettelse:** Udvid linje 96-listen til at omfatte: *"...ny tredjelands-CDN-hosting af OCSP/CRL/CT-data, ny managed-CA- eller cert-management-SaaS uden EU/EØS-residency."* Dette aktiverer art. 44-49-vurdering før design låses.

---

### Fund AP-4 (BLOKERENDE) — Cert-/audit-retention er ikke koblet til GDPR art. 5(1)(e)

**Reference:** Brief-punkt 3.

AP linje 97 etablerer 7-årsbevaring som "PKI-standard" og kalder manglende audit-trail "GDPR art. 30/32-brud". Det er korrekt, men en kritisk manglende kobling: 7-årsbevaring **skal** stå proportional med formålet (art. 5(1)(e) opbevaringsbegrænsning). For visse PKI-data-kategorier er 7 år muligvis for langt:

- **CSR-indhold (subject + email + org)** efter cert er udløbet og revoked: hvad er formålet med fortsat opbevaring? Audit-evidence for issuance-correctness, ja — men det kræver ikke nødvendigvis fuld CSR; en hash + metadata kan være tilstrækkeligt
- **OCSP-request-logs** hvis disse logger requesting-IP: kort retention bør overvejes (30-90 dage) — ikke 7 år
- **Break-glass-session-logs** (AP linje 106): 7 år ok
- **Ceremony-protokoller** (signede papirer): 7 år matcher Mads' krav, men ceremony-video (PKI-roller §3, linje 157) er specificeret til **20 år** — separate behandling

**Rettelse:** AP linje 97 udvides til: *"Retention-perioder fastsættes per data-kategori i koordination med Dorthe (art. 5(1)(e)). 7-årsbevaring er PKI-baseline for issuance/revocation audit, men kortere retention kan være nødvendigt for OCSP-request-logs og længere for ceremony-protokoller. Dorthe godkender retention-matrix per dataflow."*

---

### Fund AP-5 (CONDITIONAL) — Art. 33-breach-eskalering kobles ikke eksplicit til CA-kompromis

**Reference:** Brief-punkt 5. Pouls Vinkel 10 dokumenterer dette risk-domain.

AP linje 99: *"Breach-mistanker rapporteres direkte til Dorthe + Mads inden 24 timer."* — godt. Men eksempel-listen dækker "audit-logs bliver korrupte, eller mistanke om uautoriseret HSM-adgang". Den dækker **ikke** eksplicit:

- **Kompromittering af Issuing CA-nøgle** — alle leaf-cert-indehavere kan potentielt være berørt (mass-revocation; data-subject-notifikation per art. 34 hvis høj risiko)
- **Kompromittering af Intermediate CA-nøgle** — bredere data-subject-population
- **Root CA-kompromis** — alle data-subjects med cert udstedt af FGDCorePKI

Disse scenarier udløser direkte 72-timers Datatilsyn-frist (art. 33) **og** sandsynligvis art. 34-data-subject-notifikation.

**Rettelse:** Tilføj 1 bullet til AP linje 99-blokken: *"CA-nøgle-kompromittering (Issuing/Intermediate/Root) eskaleres direkte til Dorthe + Mads øjeblikkeligt — Dorthe initierer art. 33/34-vurdering med 72-timers-ur."*

---

### Fund AP-6 (mindre) — ROPA-notifikations-format mangler

I AP v2 (per min final-check 2026-05-08) blev der etableret 4 minimum-felter for DPO-notifikation: (a) hvad ændrer sig, (b) hvilken processing-aktivitet i ROPA berøres, (c) forventet implementeringsdato, (d) hvem der drev ændringen. PKI-AP-udkastet linje 96 har dette format **bevaret** — det er korrekt og glæder mig. Ingen ændring nødvendig her; nævnt for at flagge at netop dette punkt **ikke** er regredieret (modsat AP-1 og AP-2).

---

## PKI-governance-roller — GDPR-fund

### Fund GR-1 (BLOKERENDE) — RA-rollen har ingen GDPR-sektion

**Reference:** Brief-punkt 7. PKI-governance-roller §4 (linjer 185-220).

RA-rollen er **den eneste rolle i PKI-governance-modellen der direkte behandler ansøger-PII**:
- CSR-indhold (subject CN, SAN, email, organisation)
- Identitets-verifikation via LDAP-lookup, email-verification, mTLS-pin
- Approve/reject med begrundelse (kan indeholde personlige oplysninger)

Sektionen "Compliance-relateret" (linjer 211-215) nævner **kun** FIPS-konformitet og Trine-audit. Ingen GDPR-substans. Det er den største enkelte gap i de tre udkast.

**Konkret manglende elementer:**

1. **Lawful basis (art. 6) for CSR-/identitets-behandling** — ikke specificeret. Sandsynligt: art. 6(1)(b) (kontrakt) for kunder, art. 6(1)(f) (legitim interesse) for interne medarbejdere, art. 6(1)(c) (retlig forpligtelse) hvis FIPS/regulatorisk udstedelse
2. **Retention af CSR-indhold + RA-audit-log** efter cert er udløbet/revoked — ikke specificeret. Bør være kortere end Mads' 7-årsbevaring for revocation-data; måske 3 år efter cert-udløb (data-minimering art. 5(1)(c)+(e))
3. **DPO-notifikations-pligt når RA-portal-felter ændres** — ikke specificeret. Hvis fx ny "ID-dokument-upload"-funktion tilføjes, skal Dorthe vurdere
4. **Data-subject-rettigheder (art. 15-22)** for cert-ansøgere — ikke specificeret. Hvem håndterer access-request fra en person der har ansøgt om cert? Hvem håndterer sletning hvis ansøger trækker sig?
5. **Identitets-verifikations-data** (LDAP-lookup-resultater, mTLS-pin-fingerprints) — opbevares de? Hvor længe?
6. **Afvisnings-begrundelser** kan indeholde sensitiv information (fx "afvist fordi requester ikke er employee per HR-system") — særlig kategori-data per art. 9 hvis sundhedsdata, kriminelle forhold, osv.

**Rettelse:** Tilføj GDPR-sektion til RA-profil (parallelt til Compliance-relateret), minimum:

```
### GDPR-relateret (fra Dorthe)

- **Lawful basis dokumenteres per ansøger-kategori** i CP/CPS §6: (a) art. 6(1)(b) for kunde-cert (kontrakt), (b) art. 6(1)(f) for interne (legitim interesse + balance-test), (c) art. 6(1)(c) hvis regulatorisk
- **Retention-matrix:** CSR-indhold opbevares maksimalt 3 år efter cert-udløb/revocation (art. 5(1)(e)); RA-audit-log med approve/reject-events 7 år (Mads' baseline); identitets-verifikations-data (LDAP-resultater, mTLS-pins) opbevares kun under behandling — slettes ved godkendelse/afvisning
- **Data-subject-rettigheder:** RA håndterer i første instans art. 15-anmodning fra cert-ansøgere (adgang til egen CSR + RA-decision); art. 17 (sletning) eskaleres til Dorthe da revocation-data har juridisk retentionspligt
- **DPO-notifikation:** Ved enhver ændring af RA-portal-felter (nye identifikationskilder, ny ID-dokument-upload, ændret approve/reject-format) notificeres Dorthe inden 5 hverdage med ROPA-format-bullets
- **Afvisnings-begrundelser:** Må ikke indeholde art. 9 særlige kategorier (sundhed, kriminelle forhold, etnicitet, fagforening, religion); generelle policy-koder bruges i stedet
- **Eskalations-pligt:** Tvivl om GDPR-konsekvens af RA-policy → eskaleres til Dorthe inden implementering
```

---

### Fund GR-2 (BLOKERENDE) — Ceremony-video 20-årsbevaring kræver DPIA

**Reference:** Brief-punkt 1+3. PKI-governance-roller §3 (Key Ceremony Officer), linje 157: *"optagelse arkiveres krypteret med WORM-garanti (minimum 20 år)"*.

Ceremony-video indeholder PII af særlig karakter:
- Witnesses og smartcard-holders' billede + stemme = biometriske kategorier per art. 9 (hvis brugt til identifikation)
- 20-årsbevaring er væsentligt længere end PKI-cert-baseline (7 år)
- Formålet (audit-evidence for ceremony-integritet) skal være proportional med 20-års-opbevaring

GDPR-art. 5(1)(c) data-minimering og art. 5(1)(e) opbevaringsbegrænsning kræver formålsbestemt vurdering. Dette er **DPIA-trigger** per art. 35(3)(b) (systematisk og omfattende behandling) og potentielt art. 35(3)(b)+(c) hvis biometri.

**Rettelse:**
1. Ceremony Officer-mandatet henviser eksplicit til at retention-perioden for video kræver Dorthe-DPIA inden første ceremony
2. Alternativt: skift fra video → audio + signeret papirprotokol + screen-capture (mindre PII-rich)
3. Eller: video uden lyd + maskering af ansigter (kun screen + hænder synlige) — reducerer art. 9-eksponering
4. Alle witnesses skal have informeret samtykke (art. 6/7) eller anden lawful basis dokumenteret før ceremony

**Tilføjes til PKI-governance-roller §3 "Compliance-relateret"-sektion:**

```
- **GDPR-impact af ceremony-recording:** Optagelse af ceremony udgør behandling af biometriske data (witnesses + officers) og kræver DPIA (art. 35) før første ceremony. 20-årsbevaring er ikke automatisk proportional — formålsbestemt vurdering ved Dorthe inden ceremony-script låses. Lawful basis dokumenteres per deltager (samtykke art. 6(1)(a) eller anden basis); deltagere informeres om optagelse, retention, og deres rettigheder per art. 13/14
```

---

### Fund GR-3 (CONDITIONAL) — HSM-operatør og audit-log-eksport mangler PII-vurdering

PKI-governance-roller §2 (HSM-operatør), linje 102: *"Backup-execution: Encrypted key export (wrap-key-baseret)"*.

HSM audit-logs indeholder potentielt:
- Authentication-key-ID for hver session = identificerer hvilken person (hvis 1:1-mapping)
- Timestamp + command-historik = adfærdsmønster
- Object-ID-references (kan kobles til cert-indehaver)

Per Pouls Vinkel 8 eksporteres audit-logs hver time til immutable WORM. Dette er PII-behandling som ikke aktuelt er adresseret i HSM-operatør-profilen.

**Rettelse:** Tilføj til §2 Compliance-relateret-sektion: *"HSM audit-log indeholder authentication-key-ID + command-historik der kan udgøre PII (1:1-mapping til operatør). Eksport-flow til WORM-storage skal være EU-residency (Dorthe vurderer leverandør); access til logs er least-privilege (kun Trine + Mads + Dorthe)."*

---

### Fund GR-4 (CONDITIONAL) — CA Administrator audit-trail-rolle ikke GDPR-koblet

PKI-governance-roller §1 (CA Administrator) — Compliance-relateret (linjer 71-74) nævner audit-log-leverance til Trine men ikke at certifikat-issuance-logs indeholder PII (subject = navn/email/org).

**Rettelse:** Tilføj 1 bullet: *"Cert-issuance-logs indeholder subject-PII (CN, SAN, email, org). Adgang til issuance-log er least-privilege; eksport til Trine sker via signed log-shipping; logs slettes per retention-matrix (Dorthe godkender)."*

---

### Fund GR-5 (mindre) — Trine PKI-auditor-udvidelse mangler GDPR-koordination

PKI-governance-roller §6 (Trine udvidelse) er beskrevet uden eksplicit interface mod Dorthe. Trine reviewer SOD og access-control kvartalsvis — men hvis Trine finder at en RA-bruger har read-access til andre brugeres CSR-data (PII-spillover), er det både en SOC 2/ISO 27001-finding **og** en GDPR art. 32-finding.

**Rettelse:** Tilføj note: *"Findings med GDPR-impact (uautoriseret PII-access, retention-overskridelse, manglende lawful basis) cc:'es til Dorthe i samme rapport."*

---

## ROPA-implikationer

FGDCorePKI introducerer minimum 9 nye behandlingsaktiviteter der skal i ROPA:

| # | Behandling | Data-kategorier | Lawful basis (forventet) | Retention | DPIA påkrævet? |
|---|---|---|---|---|---|
| 1 | Certifikat-udstedelse | CN, SAN, email, organisation, public key | art. 6(1)(b)/(f)/(c) per ansøger-type | Cert-validity + 3 år (post-revocation) | Nej (lav risiko) |
| 2 | CSR-modtagelse + RA-identitets-verifikation | CSR-indhold + LDAP/email/mTLS verification | art. 6(1)(b)/(f) | Beslutning + 3 år | Conditional (afhænger af verifikations-kanaler) |
| 3 | Cert-issuance-audit-log (CA Admin) | Subject-PII + timestamp + RA-decision-ref | art. 6(1)(c) (audit-pligt) | 7 år (PKI-baseline) | Nej |
| 4 | HSM-audit-log (operatør) | Authentication-key-ID + commands + timestamps | art. 6(1)(f) (sikkerhed) | 7 år | Nej (intern PII) |
| 5 | Key-ceremony-optagelse (video/audio) | Biometri (billede/stemme), navne, signaturer | art. 6(1)(a)/(c) + art. 9 | **20 år — kræver DPIA** | **JA — art. 35(3)(b)+(c)** |
| 6 | OCSP-responder-requests | Requesting IP + cert-serial | art. 6(1)(f) | **30-90 dage anbefalet (ikke 7 år)** | Conditional |
| 7 | CRL-distribution (incl. CDN-fetch-logs) | Requesting IP + CRL-version | art. 6(1)(f) | Som OCSP | **JA hvis tredjelands-CDN** (art. 44-49) |
| 8 | Break-glass-session-logs (Provider HSM-access) | Operator-ID, begrundelse, command-trace | art. 6(1)(c) | 7 år | Nej |
| 9 | Revocation-data + data-subject-notifikation ved CA-kompromis | Cert-indehavere, kontaktdata | art. 6(1)(c) (art. 34-pligt) | 7 år (revocation-evidence) | Nej, men art. 34-flow påkrævet |

**Note til ROPA-vedligehold:** `governance/dpa/ropa.md` eksisterer ikke endnu (verificeret 2026-05-09). Når FGDCorePKI godkendes til implementation, skal Camilla oprette dokumentet og jeg leverer indhold for de 9 behandlinger. Indtil da bibeholdes denne tabel som kilde.

---

## Krævede rettelser inden aktivering

**Alle skal rettes før jeg kan udstede GO. Listet i rækkefølge:**

### Blokerende (regression af v2-præcedens)

1. **AO-1**: Genindsæt 4-bullet DPO-uafhængigheds-blok (art. 38(3)) i AO-mandatet, parallelt til AO v2 linjer 91-96
2. **AO-2**: Tilføj §3.3-supersedance-note + reference til FGD-override 2026-05-06 i AO-mandatet
3. **AP-1**: Genindsæt 4-bullet DPO-uafhængigheds-blok i AP-mandatet, parallelt til AP v2 linjer 112-117
4. **AP-2**: Tilføj §3.3-supersedance-note + reference til FGD-override 2026-05-06 i AP-mandatet

### Blokerende (PKI-specifik GDPR-substans)

5. **AP-3**: Udvid AP-tredjeland-trigger-listen til at omfatte OCSP/CRL-CDN, CT-logs, cert-management-SaaS
6. **AP-4**: Kobl AP-retention-bullet til art. 5(1)(e) og specificer at Dorthe godkender retention-matrix per dataflow
7. **GR-1**: Tilføj fuld GDPR-sektion til RA-profil (lawful basis, retention, data-subject-rettigheder, DPO-notifikation, art. 9-undtagelse, eskalations-pligt) — se konkret tekst i fund GR-1
8. **GR-2**: Tilføj DPIA-pligt + lawful-basis-krav for ceremony-video-recording til Key Ceremony Officer-profil; overvej alternativ optage-format

### Conditional (kan rettes parallelt med ovenstående)

9. **AO-3**: Tilføj 24-timers breach-eskalation til AO-hand-off-aftaler
10. **AO-4**: Udvid AO-tredjelands-eksempel ud over Langchain
11. **AP-5**: Tilføj eksplicit CA-nøgle-kompromis til AP-breach-trigger-liste med 72-timers-ur
12. **GR-3**: Tilføj PII-/EU-residency-note til HSM-operatør Compliance-relateret
13. **GR-4**: Tilføj PII-note til CA Administrator audit-trail
14. **GR-5**: Tilføj GDPR-cc-bestemmelse til Trine-udvidelse

### Ikke blokerende (acceptabelt som beskrevet)

- AP linje 96 ROPA-notifikations-format med 4 minimum-felter — bevaret korrekt fra v2
- AP linje 97-98 audit-trail som GDPR art. 30/32-pligt — bevaret korrekt
- AO linje 87 eskalations-pligt ved tvivl — bevaret korrekt

---

## Process-note til Stefan

**Ved næste runde:** Når Laila har lavet rettelse-batch, vil jeg gerne lave **final-check** (verifikation, ikke fuldt review) parallelt til min 2026-05-08-final-check. Forventet leverance-tid: 1 hverdag efter rettelser modtaget.

**Konsistens med Mads' parallelle vurdering:** Mads laver sandsynligvis security/SOD-vurdering parallelt. Hans vurdering bør særligt teste: cross-rolle-constraint (AO+AP+RA+CA Admin SOD-matrix), break-glass-procedure-effektivitet, FIPS-algoritme-enforcement i mandat-tekst. Mit STOP-kald handler udelukkende om GDPR-/uafhængigheds-aspekter; Mads kan have parallelle eller divergerende fund.

**Reference til præcedens:** Min 2026-05-08-vurdering blev udstedt efter v2-iteration med samme mandat-struktur. Den runde tog 2 udkast + final-check at lukke. Hvis FGDCorePKI-udkastene starter fra v3-tilstand uden v2-rettelserne, bør Laila forvente samme antal iterationer med mig — alternativt at hun starter fra v2-skeleton og overlay'er PKI-specifika.

---

**Forfatter:** Dorthe (DPO)
**Dato:** 2026-05-09
**Status:** STOP — afventer Laila-rettelser jf. de 14 fund (8 blokerende + 6 conditional)
**Næste leverance:** Final-check efter Laila-rettelser (parallelt format til 2026-05-08-final-check)
