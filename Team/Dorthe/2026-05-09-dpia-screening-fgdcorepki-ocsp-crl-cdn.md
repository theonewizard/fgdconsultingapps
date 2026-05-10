---
dato: 2026-05-09
type: dpia-screening
projekt: FGDCorePKI
behandlings-aktivitet: OCSP-responder + CRL-distribution via tredjelands-CDN (conditional)
initieret-af: Dorthe (DPO)
status: SCREENING-IGANGSAT
kræver-fuld-dpia: TBD (conditional — afhænger af arkitekturvalg)
betinget-af:
  - Arkitekturbeslutning: anvendes tredjelands-CDN (Cloudflare US, Fastly US, Akamai global) til OCSP/CRL distribution?
  - Hvis JA → fuld DPIA påkrævet
  - Hvis NEJ (intern EU-hosting eller EU-only-CDN) → screening lukkes med "ingen DPIA påkrævet, ROPA-opdatering tilstrækkelig"
basis:
  - Team/Dorthe/2026-05-09-dorthe-final-check-fgdcorepki-mandater.md (AP-3, ROPA #6+#7)
  - Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md (Fund AP-3, AP-4)
  - Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md (Vinkel 3 + Vinkel 6)
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md v2 (linje 96)
referencer-gdpr:
  - art. 44 (generelt princip for tredjelands-overførsel)
  - art. 45 (adequacy-decisions — EU-Commission)
  - art. 46 (passende sikkerhedsforanstaltninger — SCCs, BCRs)
  - art. 49 (specifikke situationer — undtagelser)
  - art. 5(1)(c) (data-minimering)
  - art. 5(1)(e) (opbevaringsbegrænsning)
  - art. 6(1)(f) (legitim interesse — sikkerhed)
  - art. 35 (DPIA hvis høj risiko)
  - Schrems II (C-311/18) — TIA påkrævet for US-overførsler
edpb-references:
  - EDPB Recommendations 01/2020 (transfer impact assessment — TIA)
  - EU-US Data Privacy Framework (DPF) 2023 — adequacy-decision for visse US-leverandører
modtagere: FGD, Mads (CISO), AP (når rolle bemandes), Stefan (orkestrator)
---

# DPIA-screening: OCSP/CRL CDN-hosting (conditional)

## 1. Behandlings-beskrivelse

**Hvad behandles (hvis tredjelands-CDN anvendes):**

OCSP og CRL er **availability-kritiske** PKI-services. Hver gang en klient (browser, server, IoT-enhed) verificerer et FGDCorePKI-udstedt certifikat, foretages opslag mod:
- **OCSP-responder**: real-time forespørgsel "er cert med serial X tilbagekaldt?" → svar OK/REVOKED/UNKNOWN
- **CRL Distribution Point (CDP)**: hentes signeret CRL-fil med liste af tilbagekaldte certifikater

**Data der overføres til CDN:**

| Data-element | PII-status | Begrundelse |
|---|---|---|
| Klientens IP-adresse (request-source) | **PII** (art. 4(1) + Recital 30; EDPB-praksis) | IP er identificerbar i kombination med andre data |
| Cert-serial-number i OCSP-request | Conditional PII | Serial er ikke direkte PII, men korrelerer med specifik cert-indehaver hvis CDN logger serial+IP+timestamp sammen |
| User-Agent header | Mulig PII (sjældent unik) | Browser/OS-fingerprint |
| Request-timestamp | Metadata | I kombination med IP/serial: adfærdsmønster |
| TLS-handshake-data | Mulig PII | Cipher-suite-rækkefølge kan fingerprintes |
| CRL-fetch-pattern | Metadata | Hvilke organisationer der verificerer mod FGDCorePKI |

**Behandlingens omfang:**
- OCSP-requests genereres automatisk af klient ved hver TLS-handshake hvor cert-validering kræves
- Volumen: **høj** — potentielt millioner af requests/dag hvis FGDCorePKI deployer leaf-certs til offentligt-tilgængelige services
- Geografisk spredning: requesterens IP afhænger af cert-bruger; for FGDCorePKI med dansk/EU-fokus primært EU-IP'er, men globalt muligt

**Af hvem (hvis CDN anvendes):**
- Dataansvarlig: FGD (CVR)
- Behandler: FGDCorePKI-teamet (AP designer arkitektur)
- Sub-processor: CDN-leverandør (Cloudflare Inc., Fastly Inc., Akamai etc.) — ofte US-baseret med globalt edge-netværk

**Til hvilket formål:**
1. **Certifikat-validering** (art. 6(1)(f) — legitim interesse: sikker autentificering)
2. **Availability** — CDN sikrer global lav-latency og DDoS-resilience for OCSP/CRL (kompromittering af OCSP-availability = de facto cert-revocation-problem)
3. **Skalering** — OCSP/CRL-load er burst-præget; CDN-caching reducerer load på origin

---

## 2. Vurdering af DPIA-trigger (GDPR art. 35)

DPIA-pligt udløses primært af **tredjelands-overførsel + storskala-behandling + automatiseret IP-logning** snarere end af særlige kategorier.

| # | WP29-kriterium | Opfyldt? | Begrundelse |
|---|---|---|---|
| 1 | Evaluering eller scoring | Nej | Ingen profilering |
| 2 | Automatiseret beslutningstagning | Nej | OCSP-svar er teknisk, ikke retsvirkning |
| 3 | Systematisk overvågning | **Delvist** | OCSP-trafik kan teoretisk korreleres til adfærdsmønstre, men typisk ikke aktivt overvåget |
| 4 | Sensitive data | Nej | Ikke art. 9-data |
| 5 | Behandling i stor skala | **JA** | Millioner af requests potentielt; bredt geografisk |
| 6 | Matching/kombination af datasæt | **Delvist** | Hvis CDN-logs kombineres med cert-serial: kan udlede hvilken klient der bruger hvilken service |
| 7 | Sårbare personer | Nej | Generel cert-validering |
| 8 | Innovativ teknologi | Nej | Standard OCSP/CRL |
| 9 | Forhindrer udøvelse af ret | **Delvist** | OCSP-availability er kritisk for cert-validering; tab kan blokere services |

**Antal opfyldte kriterier:** 1 fuld (#5 storskala) + 3 delvise (#3, #6, #9)

**Yderligere DPIA-pligt-grundlag (uafhængigt af WP29-kriterier):**
- **Tredjelands-overførsel** kræver per art. 44-49 en **transfer impact assessment (TIA)** efter Schrems II for US-leverandører — TIA er ikke det samme som DPIA, men de overlapper væsentligt og udføres typisk som integreret dokument når CDN-arkitektur er ny
- **EDPB Recommendations 01/2020** kræver "supplementary measures" hvis art. 46-mekanisme alene ikke giver "essentially equivalent" beskyttelse
- Datatilsynets generelle praksis: storskala-behandling af IP-adresser med tredjelands-overførsel er DPIA-pligtig

---

## 3. Foreløbig konklusion

**Kræver fuld DPIA: TBD — conditional på arkitekturbeslutning.**

To scenarier:

### Scenario A: Tredjelands-CDN anvendes (Cloudflare, Fastly, Akamai US-baseret)

**→ Fuld DPIA + TIA påkrævet inden produktionsdeployment.**

Begrundelser:
1. Storskala IP-logning (WP29 #5)
2. Tredjelands-overførsel (art. 44-49) med Schrems II-implikationer for US
3. Sub-processor-DPA + SCCs påkrævet (DPA-tracker-opdatering)
4. Supplementary measures kan være nødvendige (kryptering, pseudonymisering, kontraktlige forpligtelser)
5. Availability-kritikalitet (#9 delvist) — kombineret med tredjelands-risiko (US CLOUD Act, US FISA 702) giver høj rest-risiko der potentielt kræver art. 36-konsultation

### Scenario B: Intern EU-hosting eller EU-only-CDN (fx Bunny.net EU-config, intern Vercel EU-region, on-prem)

**→ DPIA ikke påkrævet; ROPA-opdatering er tilstrækkelig (#6+#7 i ROPA).**

Begrundelser:
1. Ingen tredjelands-overførsel
2. Storskala-aspektet alene (#5) udgør ikke nok til DPIA-pligt uden andre faktorer
3. Standard art. 6(1)(f)-vurdering + retention-matrix er tilstrækkeligt
4. Krav: leverandør-DPA på plads (DPA-tracker), retention 30-90 dage for IP-logs

### Scenario C: Hybrid (EU-CDN med tredjelands-failover)

**→ Fuld DPIA påkrævet (samme som Scenario A; failover-aktivering = tredjelands-overførsel).**

---

## 4. Afventende information

Inden DPIA kan endeligt scopes (eller lukkes):

| # | Spørgsmål | Ansvar | Frist |
|---|---|---|---|
| F1 | Arkitekturbeslutning: hvilken OCSP/CRL-distributionsmodel vælges? Intern hosting / EU-CDN / global CDN? | AP + Mads + Poul | Inden Fase 0 produktionstag |
| F2 | Hvis CDN: hvilke specifikke leverandører er på shortlist? Adequacy-status (DPF, andre)? | AP + Dorthe | Parallelt med F1 |
| F3 | Forventet request-volumen (OCSP/dag) i Fase 0/1/2? — driver storskala-vurdering | AP + Poul | Ved arkitekturvalg |
| F4 | Hvilke OCSP/CRL-services skal cache-hostes? OCSP-stapling som alternativ (cert-issuer leverer signed OCSP-response sammen med cert; ingen CDN-fetch fra klient) | AP + Mads | Ved arkitekturvalg |
| F5 | Hvis tredjelands-CDN: hvilke supplementary measures er teknisk mulige? IP-anonymisering på CDN-edge, request-aggregation, DNSSEC-OCSP-must-staple | AP + Mads | Ved DPIA-mitigation-fase (kun hvis Scenario A) |
| F6 | Logning på CDN-side: hvilke logs gemmer leverandøren? Retention? Adgang for sub-processor's medarbejdere? | AP + Dorthe (DPA-vurdering) | Ved leverandørvalg |
| F7 | Sub-processor-DPA + SCC-tilgængelighed for kandidat-leverandører | Dorthe + AO | Ved leverandørvalg |
| F8 | Hvis CT-logging (Vinkel 3 — public-trust): hvilke CT-loggers anvendes? Google/Cloudflare/Sectigo CT-logs er typisk US-hosted | AP + Mads | Hvis FGDCorePKI går public-trust |

---

## 5. Næste skridt

| # | Handling | Ansvar | Tidsramme |
|---|---|---|---|
| 1 | Stefan afklarer med FGD: ønskes EU-only-arkitektur som default? Det vil eliminere DPIA-pligt for #6+#7 | Stefan | Inden 5 hverdage |
| 2 | AP + Mads forelægger arkitekturoptioner til AO + Dorthe (intern / EU-CDN / global CDN) med GDPR-konsekvens-tabel | AP + Mads | 2-3 uger efter mandat-aktivering |
| 3 | Dorthe leverer kort GDPR-impact-vurdering pr. arkitekturoption (input til arkitekturbeslutning) | Dorthe | Parallelt med #2 |
| 4 | AO/FGD træffer arkitekturvalg | AO → FGD | Når options foreligger |
| 5a | **Hvis Scenario A eller C valgt:** Dorthe initierer fuld DPIA + integreret TIA | Dorthe | Inden produktionsdeployment |
| 5b | **Hvis Scenario B valgt:** Dorthe lukker screening med note "ingen DPIA påkrævet"; ROPA #6+#7 opdateres med EU-leverandør | Dorthe | Inden Fase 0 produktionstag |
| 6 | DPA-tracker-opdatering for valgt CDN-leverandør (DPA + SCCs hvis tredjelands) | Dorthe | Ved leverandørvalg |
| 7 | Update af ROPA #6 (OCSP) + #7 (CRL) med korrekt retention (30-90 dage anbefalet) og leverandør | Dorthe → Camilla | Ved aktivering af `governance/dpa/ropa.md` |
| 8 | Mads samarbejde: Schrems II-TIA hvis US-leverandør (kombineret med Mads' threat-model på CDN-bypass) | Dorthe + Mads | Kun ved Scenario A/C |

---

## 6. Foreløbige mitigation-anbefalinger (input til fuld DPIA hvis Scenario A/C)

Anbefalinger der reducerer rest-risiko hvis tredjelands-CDN er ufravigeligt:

1. **OCSP-stapling som primær model** — cert-issuer pre-signs OCSP-response og udsteder den sammen med cert; klient behøver ikke kontakte OCSP-responder. Eliminerer den kontinuerlige IP-logning næsten helt
2. **OCSP-must-staple** cert-extension — tvinger klient til at bruge stapling (ingen CDN-fetch)
3. **CRL i stedet for OCSP** for kort-levede certs — CRL kan caches lokalt af klient i timer/dage; reducerer CDN-fetch-frekvens
4. **IP-anonymisering på CDN-edge** — leverandører som Cloudflare og Fastly understøtter request-level IP-truncation før logging
5. **Kort retention** for CDN-logs (24-72 timer) i kontrakt; forhandling med leverandør
6. **Kontraktlige forpligtelser** ud over SCCs: forbud mod aggregation til andre formål, forbud mod sub-sub-processing uden samtykke
7. **Kryptering af request-payload** hvor teknisk muligt
8. **EU-region-pinning** — kontraktlig forpligtelse til kun at routene EU-IP'er gennem EU-edges (begrænser US-overførsel)
9. **Public transparency** — publicering af DPIA-resumé og hvilke CDN-leverandører der bruges (cert-aftagere kan vurdere)

---

## 7. Risiko-flagging (foreløbig)

**Høj-risiko-elementer (kun Scenario A/C):**
- Schrems II-implikationer for US-leverandører (CLOUD Act, FISA 702)
- Storskala IP-logning kombineret med cert-serial = identifikations-mulighed
- Sub-processor's egen logning (Cloudflare/Fastly har egne ToS for log-anvendelse)

**Mellem-risiko-elementer:**
- OCSP-fetch-pattern afslører "hvem verificerer hvilken service hvornår"
- DDoS-mitigation kræver ofte global edge-netværk (modarbejder EU-only)

**Lav-risiko-elementer:**
- Cert-serial alene er ikke PII
- CRL-data er offentlig (signeret revocation-liste)

---

## 8. Sammenhæng med øvrige DPIA-screeninger og governance-arbejde

Denne screening er parallel med:
- `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ceremony-recording.md` (uafhængig DPIA, biometri-fokus)

**Afhængigheder:**
- DPA-tracker (`Team/Dorthe/dpa-tracker.md`) — opdateres ved leverandørvalg uanset DPIA-udfald
- ROPA #6+#7 (`governance/dpa/ropa.md` når oprettet) — opdateres ved arkitekturvalg
- AP-mandat linje 96 — DPO-notifikations-trigger for "ny tredjelands-CDN-hosting" — er aktiveret nu

---

## 9. Procesnote — hvorfor screening initieres nu (ikke ved arkitekturvalg)

Selv om DPIA-pligt er conditional på arkitekturbeslutning, initieres screening nu af tre grunde:

1. **Forhåndsorientering af FGD** — FGD bør vide at arkitekturvalg har GDPR-konsekvenser **inden** valget træffes (ikke efter)
2. **Tidsbuffer** — fuld DPIA + evt. art. 36-konsultation tager 2-8 uger; hvis screening udskydes til arkitekturvalg, risikeres produktions-forsinkelse
3. **AP-mandat aktiveret nu** — AP-rollen har eksplicit pligt til DPO-notifikation ved tredjelands-CDN (linje 96); screening dokumenterer at processen er klar når trigger udløses

---

**Forfatter:** Dorthe (DPO)
**Dato igangsat:** 2026-05-09
**Status:** SCREENING-IGANGSAT — afventer arkitekturbeslutning (F1) som styrende variabel
**Næste milestone:** Conditional handoff — enten fuld DPIA (Scenario A/C) eller screening-lukning med ROPA-opdatering (Scenario B), trigger ved arkitekturvalg
