---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: GDPR-vurdering af tværgående projekt-roller — Application Owner og Application Provider
status: leveret
modtagere: Stefan (orkestrator), Laila (HR), FGD (godkender), Mads (CISO co-review)
referencer:
  - GDPR art. 4(7-8) (controller/processor-definitioner)
  - GDPR art. 5 (principper for behandling)
  - GDPR art. 24-26 (controller-ansvar, joint controllers)
  - GDPR art. 28 (processor-aftaler)
  - GDPR art. 30 (ROPA — Record of Processing Activities)
  - GDPR art. 32 (sikkerhed ved behandling)
  - GDPR art. 33-34 (breach-notifikation)
  - GDPR art. 35-36 (DPIA + forudgående høring)
  - GDPR art. 37-39 (DPO-rolle og -uafhængighed)
  - GDPR art. 44-49 (tredjelands-overførsler)
  - EDPB Guidelines 07/2020 (controller/processor-koncepter)
  - EDPB Guidelines 04/2017 (DPIA, WP248)
  - SOC 2 CC1.3 (ansvar og myndighed), CC8.1 (change management)
  - ISO 27001:2022 A.5.2 (roller/ansvar), A.5.3 (SOD)
relateret:
  - governance/compliance/2026-05-05-mads-first-session.md (SOD-rammer + DPO-model)
  - governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md (mandat-stabilitet)
---

# GDPR-vurdering af Application Owner og Application Provider som projekt-roller

## 0. Sammenfatning til Stefan

To nye tværgående projekt-roller foreslået: **Application Owner** (forretnings- og compliance-ejer pr. applikation) og **Application Provider** (teknisk applikations-ejer pr. applikation). Min vurdering:

1. **GDPR-rolleafklaring**: Application Owner er **intern compliance-ansvarlig** for produktets GDPR-posture — IKKE "data controller" i GDPR-juridisk forstand. FGD som juridisk person (CVR-enheden) forbliver **dataansvarlig**. Application Owner kan have intern "compliance-portefølje-ejerskab" uden at konflikte med min DPO-uafhængighed (art. 38(3)) — forudsat klare grænser dokumenteres. Den juridiske data controller-rolle kan ikke uddelegeres til en projekt-rolle.
2. **ROPA-implikationer**: Application Provider får **input-leverandør-rolle** til ROPA, ikke ejer-rolle. Jeg (Dorthe) forbliver indholdsansvarlig ejer af ROPA. Application Provider skal forpligtes til at notificere mig ved enhver dataflow-ændring (ny integration, ny PII-felt, ny tredjepart, ny retention-periode). Application Owner forpligtes til kvartalsvis ROPA-attest.
3. **DPIA-flow**: DPIA udløses af konkrete art. 35-triggers, ikke rolle-aktivitet. **Application Owner** er ansvarlig for at **initiere DPIA-screening** ved feature-roadmap-beslutning; **Application Provider** leverer teknisk dataflow-input; **jeg (Dorthe)** ejer DPIA-konklusion + mitigation-godkendelse.
4. **Anbefalinger til Laila**: 11 konkrete hard rules pr. rolle, plus 4 cross-rolle constraints. Vigtigste: Application Owner må ikke godkende tredjelands-overførsler uden DPO-notifikation; Application Provider må ikke have direkte adgang til prod PII uden break-glass-procedure med audit-trail.

**Vigtigt**: Disse roller ændrer IKKE ansvarsfordelingen til Datatilsynet. FGD forbliver registreret DPO; jeg forbliver operationel DPO; Mads forbliver CISO; Trine forbliver compliance-auditor. Application Owner og Provider er **interne forretningsroller** der gør det operationelt muligt at skalere på tværs af flere applikationer (knowlagecentral, fremtidige apps).

---

## 1. GDPR-rolleafklaring

### 1.1 Hvem er data controller pr. applikation?

**Juridisk udgangspunkt** (GDPR art. 4(7)): Data controller er den fysiske eller juridiske person som **bestemmer formålene og midlerne** for behandlingen. Denne rolle kan ikke uddelegeres til en intern projektrolle — den følger den juridiske person.

For FGD's vedkommende:

| Scenarie | Data controller | Begrundelse |
|---|---|---|
| Knowlagecentral som **internt værktøj** for FGD selv | FGD Consulting (juridisk enhed) | FGD bestemmer formål + midler |
| Knowlagecentral som **SaaS-tjeneste til kunder** (kundens egne brugere/data) | Kunden (FGD = data processor) | Kunden definerer formål; FGD leverer teknisk middel |
| Knowlagecentral hvor **FGD også behandler kunders data til egne formål** (fx benchmarking, AI-træning) | Joint controllers (art. 26) | Kræver explicit joint-controller-aftale |

**Konklusion 1.1**: Application Owner er **ikke** "data controller" i GDPR-juridisk forstand. Application Owner er **intern repræsentant for** den juridiske data controller (FGD) ift. den specifikke applikations compliance-posture. Dette skal være eksplicit i mandatet — ellers risikerer vi:

- Datatilsynet henvender sig til en projekt-rolle der ikke har juridisk myndighed til at svare
- Application Owner træffer beslutninger der binder FGD juridisk uden FGD's vidende
- Joint-controller-status opstår implicit hvis Application Owner aftaler dataflow med tredjeparter uden DPA

### 1.2 Application Owner's compliance-ansvar — afgrænsning

Application Owner ejer "produktets compliance-posture". Dette skal afgrænses til **interne** ansvarsområder, ikke eksterne juridiske beføjelser:

**Hvad Application Owner gør (acceptabelt)**:

- Holder produkt-roadmap op mod compliance-krav (GDPR, ISO 27001, SOC 2, FIPS 140-3)
- Initierer compliance-screening ved nye features (sender til Dorthe + Mads + Trine)
- Sikrer at feature-scorecard (jf. Mads' first-session §2.2) udfyldes før implementering
- Eskalerer compliance-issues til DPO/CISO/auditor
- Træffer go/no-go beslutninger på features hvor compliance-vurdering er klar
- Vedligeholder produkt-niveau compliance-dokumentation (ikke ROPA — det ejer jeg)

**Hvad Application Owner IKKE gør (forbudt)**:

- Træffer beslutninger der binder FGD juridisk over for data-subjects eller Datatilsynet
- Underskriver DPA'er med leverandører eller kunder uden DPO-review
- Godkender tredjelands-overførsler uden DPO-notifikation
- Træffer ensidige beslutninger om retention-perioder eller data-sletning
- Erklærer en feature "GDPR-compliant" uden DPO-konklusion

### 1.3 Konflikter med min DPO-uafhængighed (art. 38(3))?

GDPR art. 38(3) kræver at DPO **ikke modtager instrukser** om udførelsen af DPO-opgaverne, og art. 38(6) forbyder rolleblanding der skaber interessekonflikt. Application Owner er **ikke** en DPO-konflikt-rolle hvis følgende grænser respekteres:

1. **Application Owner instruerer ikke DPO**: Application Owner kan **bestille** GDPR-vurderinger fra mig (legitim opgave-anmodning), men kan ikke **diktere konklusionen**. Min vurdering er bindende uafhængig af Application Owner's roadmap-præferencer.
2. **DPO-uafhængighed bevares ved eskalation**: Hvis Application Owner og jeg er uenige om et DPO-spørgsmål, eskalerer jeg direkte til FGD (juridisk DPO-registrant) — ikke gennem Application Owner.
3. **Application Owner kan ikke fyre/sanktionere DPO**: Beskyttelse mod afskedigelse pga. DPO-arbejde er FGD's ansvar (registrant). Application Owner-rollens ansvarsområde rækker ikke ind i DPO-personalemæssige beslutninger.
4. **Compliance-rapportering går ikke gennem Application Owner**: Mine månedlige/kvartalsvise rapporter til FGD passerer Stefan som koordinerings-kanal — ikke Application Owner. Application Owner får **kopi** af relevante leverancer, ikke filter-position.

**Konklusion 1.3**: Application Owner-rollen er forenelig med min DPO-uafhængighed forudsat Laila skriver de fire grænser ovenfor som **hard rules** i Application Owner-mandatet.

### 1.4 Application Provider's GDPR-rolle

Application Provider er **teknisk** applikations-ejer — ikke en GDPR-rolle i strikt forstand, men har stor GDPR-impact via:

- Dyb adgang til integrationer (kender alle dataflows)
- Hands-on teknisk ejerskab (kan ændre dataflows uden at andre umiddelbart ser det)
- Konsultations-ressource for Poul (analytiker) og potentielt for kunder

**GDPR-implikation**: Application Provider er en **høj-privilegeret** rolle i GDPR-perspektiv. Dette kræver:

1. Audit-trail-pligt for alle dataflow-relaterede handlinger
2. Notifikations-pligt til DPO ved enhver dataflow-ændring (også "minor" — fx ny env-var der peger til ny third-party endpoint)
3. Begrænset prod-PII-adgang (kun via break-glass med begrundelse + audit + Mads-godkendelse)
4. Eksplicit forbud mod at agere DPA-forhandler eller compliance-vurderer (det er hhv. Dorthe/Mads/Trine)

---

## 2. ROPA-implikationer

### 2.1 ROPA-ejerskab (uændret)

**Jeg (Dorthe) forbliver indholdsansvarlig ejer af ROPA** under GDPR art. 30 + art. 39(1)(a). Dette er ikke til forhandling — ROPA-ejerskab kan ikke uddelegeres til en projekt-rolle uden at bryde DPO-funktionen.

Application Owner og Provider får **input-leverandør-roller** ind i ROPA-vedligeholdet:

| Rolle | ROPA-funktion | Frekvens |
|---|---|---|
| **Dorthe (DPO)** | Indholdsansvarlig ejer; godkender alle ændringer; kvartalsvis review | Kontinuerlig |
| **Camilla (bibliotekar)** | Administrerer git, versionstyring, dokumentationsstruktur for ROPA-fil | Per commit |
| **Application Owner** | Kvartalsvis attest: "ROPA reflekterer faktisk produkt-state" | Kvartalsvis |
| **Application Provider** | Notifikations-pligt ved enhver dataflow-ændring (real-time, ikke batch) | Per ændring |
| **Mads (CISO)** | Sparring på sikkerhedsforanstaltninger-feltet | Ad-hoc |
| **Trine (auditor)** | Reviewer ROPA som ISO 27001 A.5.34 + SOC 2 P-criterion-evidens | Halvårlig |

### 2.2 Application Provider's notifikations-pligt — konkret design

Application Provider er teknisk tættest på dataflows og har derfor **first-detect**-ansvar når noget ændrer sig. Konkret pligt:

**Application Provider skal notificere Dorthe inden 5 hverdage ved**:

1. Ny tredjeparts-integration (ny SaaS-leverandør, ny API-endpoint udadgående)
2. Nyt PII-felt i database eller API-payload
3. Ændret retention-periode for eksisterende data-kategori
4. Ny data-kategori der opsamles (fx tilføjelse af bruger-IP-logging hvor det ikke var før)
5. Ny tredjelands-overførsel (ny region for eksisterende leverandør, ny leverandør udenfor EU/EØS)
6. Ændring af processor-status (eksisterende leverandør får udvidet rolle, fx Voyage skifter fra "embedding-only" til "også fine-tuning")
7. Sletning eller arkivering af eksisterende processing-aktivitet
8. Ny audit-log-kategori der indeholder PII

**Notifikations-format**: Kort note i `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med:
- Hvad ændrer sig
- Hvilken processing-aktivitet i ROPA berøres
- Forventet implementeringsdato
- Hvem der har drevet ændringen (typisk backend-udvikler eller Application Owner)

**Hvis notifikation ikke sker**: Vurderes som GDPR art. 30-brud (ikke vedligeholdt ROPA). Trine fanger det ved halvårlig audit. Konsekvens: change-request blokeres + Application Provider skal redegøre.

### 2.3 Application Owner's kvartalsvise attest

For at sikre at ROPA ikke driver fra virkeligheden, attesterer Application Owner kvartalsvis:

**Format**: Underskrevet note (Application Owner-rollen) til Dorthe + Trine:
- "Pr. <dato> bekræfter jeg som Application Owner for <applikation> at ROPA i `governance/dpa/ropa.md` reflekterer den faktiske processing"
- Liste over kendte afvigelser (hvis nogen)
- Liste over change-requests under behandling

**Hvis attest udebliver**: Eskaleres til FGD via Stefan inden 14 dage. Audit-evidens for ISO 27001 A.5.2 + SOC 2 CC1.3.

### 2.4 ROPA-multi-app-skalering

Når FGD har flere applikationer ud over knowlagecentralen, vokser ROPA. Min anbefaling:

- **Én master-ROPA pr. juridisk enhed** (FGD Consulting), ikke pr. applikation. Dette er GDPR-praksis — controller-niveau, ikke app-niveau.
- **Sektionering pr. applikation** internt i ROPA: hver app har sit eget kapitel; processing-aktiviteter inden for app er nummereret app-specifikt
- **Cross-app-flows** (fx fælles SSO via Clerk på tværs af apps) listes én gang som "delt processing-aktivitet" og refereres fra hver app

Application Owner ejer sit kapitel; jeg ejer master-strukturen + cross-app-sektioner.

---

## 3. DPIA-flow

### 3.1 Hvornår udløses DPIA?

GDPR art. 35(1) kræver DPIA når processing **"sandsynligvis vil indebære en høj risiko for fysiske personers rettigheder og friheder"**. EDPB Guidelines 04/2017 (WP248) opstiller 9 kriterier — hvis 2 eller flere er opfyldt, er DPIA påkrævet:

1. Evaluering eller scoring (inkl. profiling, AI-decisions)
2. Automatiseret beslutningstagning med retsvirkning eller tilsvarende effekt
3. Systematisk overvågning
4. Følsomme data eller data af stærkt personlig karakter
5. Data behandlet i stor skala
6. Sammenkobling eller kombination af datasæt
7. Data om sårbare data-subjects (børn, ansatte, patienter)
8. Innovativ brug af nye teknologier (AI, IoT, biometri)
9. Når processing forhindrer data-subjects i at udøve en rettighed eller bruge en service

**For knowlagecentral** har Mads og jeg allerede identificeret DPIA-kandidater i first-session-rapporten:
- AI-berigelse (Voyage + Anthropic dataflow) — kriterium 4 + 5 + 8
- Magic-link-invite — kriterium 5 (skala)
- Subteam/organization_members + invitations — kriterium 5 + 9

For fremtidige applikationer er disse triggers dynamiske — Application Owner skal kunne aflæse dem.

### 3.2 DPIA-initierings-ansvar

| Fase | Ansvarlig | Output |
|---|---|---|
| **DPIA-screening** (er DPIA påkrævet?) | **Application Owner** initierer; **Dorthe** afgør | Ja/Nej-beslutning + begrundelse i `governance/dpa/dpia-screening-<applikation>-<feature>.md` |
| **Teknisk dataflow-input** | **Application Provider** leverer | Sequence-diagrammer, data-kategorier, third-party-endpoints, retention |
| **Risk-assessment** | **Dorthe** ejer | Sandsynlighed × konsekvens pr. risiko-scenarie |
| **Mitigation-design** | **Dorthe** + **Mads** (security) + **Application Provider** (teknisk) | Konkrete mitigations indbygget i feature-design |
| **DPIA-konklusion** | **Dorthe** (med Mads-co-sign på security-mitigations) | Godkendt / Godkendt med betingelser / Afvist / Forudgående høring til Datatilsynet |
| **DPIA-godkendelse-til-implementering** | **FGD** (juridisk DPO-registrant) på baggrund af min indstilling | Signed-off DPIA-dokument |
| **Forudgående høring** (art. 36) | **FGD** indsender til Datatilsynet hvis høj-risk uden tilfredsstillende mitigation | Høringssvar |

### 3.3 DPIA-tidslinje i feature-flow

For at undgå at DPIA bliver flaskehals foreslår jeg integration med Mads' feature-scorecard:

```
1. Application Owner foreslår feature i roadmap
2. Application Owner initierer feature-scorecard (jf. Mads §2.2)
3. Scorecard indikerer GDPR-trigger? → Application Owner initierer DPIA-screening
4. Dorthe afgør DPIA-krav (~3 hverdage)
5. Hvis DPIA påkrævet:
   a. Application Provider leverer dataflow-input (~5 hverdage)
   b. Dorthe + Mads + Application Provider designer mitigations (~5-10 hverdage)
   c. Dorthe leverer DPIA-konklusion til FGD
   d. FGD signerer eller anmoder om revision
6. Hvis DPIA ikke påkrævet: dokumenteres negativ-screening i ROPA
7. Feature implementeres (eller blokeres)
```

**Gate**: Ingen feature med DPIA-trigger må deployes til prod uden FGD-signeret DPIA. Dette er en **hard rule** for Application Owner.

### 3.4 DPIA-vedligehold over tid

DPIA er ikke engangs-leverance. EDPB anbefaler review:
- Ved væsentlige feature-ændringer
- Ved ny risk-vektor (ny trussel-aktør, ny lovgivning)
- Som minimum hvert 2. år for høj-risk processing

**Application Owner** er ansvarlig for at flagge når en feature ændrer sig så DPIA bør reviewes. **Jeg (Dorthe)** vurderer review-behovet.

---

## 4. Anbefalinger til Laila — konkrete hard rules

Disse skal indarbejdes i mandaterne for henholdsvis Application Owner og Application Provider. Jeg anbefaler at Laila co-designer med Mads (security-perspektiv) før FGD-signoff.

### 4.1 Application Owner — hard rules (GDPR-relateret)

1. **Juridisk afgrænsning**: "Application Owner er intern compliance-portefølje-ejer for sin applikation, ikke 'data controller' i GDPR-juridisk forstand. Datacontroller-rollen tilhører FGD som juridisk enhed og kan ikke uddelegeres."
2. **DPO-notifikation før tredjelands-overførsler**: "Application Owner må ikke godkende ny third-party-integration eller ændring af eksisterende integration med dataflow til tredjeland (uden for EU/EØS) uden forudgående DPO-notifikation og art. 44-49-vurdering. Dette gælder også tilfælde hvor leverandøren har EU-region men koncern-mor i tredjeland."
3. **DPA før integration**: "Application Owner må ikke aktivere integration med ekstern leverandør i prod før DPA er på plads og godkendt af DPO + CISO."
4. **DPIA-screening-pligt**: "Application Owner skal initiere DPIA-screening (jf. governance/dpa/dpia-screening-template.md) for alle nye features før implementeringsbeslutning. Hvis DPIA-screening udløser DPIA-krav, skal feature ikke deployes til prod før FGD-signeret DPIA foreligger."
5. **Kvartalsvis ROPA-attest**: "Application Owner skal kvartalsvis attestere over for DPO + auditor at ROPA-sektionen for sin applikation reflekterer den faktiske processing. Manglende attest eskaleres til FGD efter 14 dages forsinkelse."
6. **Compliance-claims**: "Application Owner må ikke erklære features eller produktet 'GDPR-compliant', 'ISO 27001-certified', 'SOC 2-aligned' i kunde-kommunikation, marketing, eller kontrakter uden forudgående bekræftelse fra hhv. DPO (GDPR) eller compliance-auditor (ISO/SOC 2)."
7. **Eskalations-pligt**: "Application Owner skal eskalere alle data-subject-anmodninger (art. 15-22), breach-mistanker, og Datatilsynets henvendelser til DPO inden 24 timer."
8. **Retention-beslutninger**: "Application Owner kan foreslå retention-perioder; godkendelse af retention-perioder ligger hos DPO (GDPR-overhold) i samråd med CISO (sikkerhed) og auditor (ISO/SOC 2-overhold)."

### 4.2 Application Provider — hard rules (GDPR-relateret)

1. **Dataflow-notifikations-pligt**: "Application Provider skal inden 5 hverdage notificere DPO ved enhver dataflow-ændring (jf. liste i Dorthes ROPA-notifikations-procedure). Manglende notifikation behandles som GDPR art. 30-brud."
2. **Ingen direkte prod-PII-adgang som standard**: "Application Provider har ikke read-access til prod-PII som standard. Adgang til prod-PII kræver break-glass-procedure: skriftlig begrundelse, CISO-godkendelse, automatisk audit-trail-emit, og maksimal session-længde 4 timer."
3. **Ingen DPA-forhandling**: "Application Provider må ikke føre DPA-forhandlinger med leverandører. Ved leverandør-vurdering leverer Application Provider teknisk dataflow-input til DPO, der ejer DPA-forhandlingen."
4. **Ingen compliance-vurderinger udadtil**: "Application Provider må ikke udtale sig om GDPR/ISO/SOC 2-compliance til kunder, leverandører eller revisorer på vegne af FGD. Kompliance-udtalelser er DPO/CISO/auditor-funktion."
5. **Audit-trail på alle dataflow-relaterede handlinger**: "Application Provider's tekniske handlinger der berører dataflows (env-vars, konfigurationer, integration-credentials) skal logges til audit.events med 3-års retention. Manuelle handlinger uden audit-trail er forbudt på prod."
6. **DPIA-input-pligt**: "Application Provider skal levere teknisk dataflow-input til DPIA inden 5 hverdage efter anmodning fra DPO. Manglende input blokerer feature-deploy."

### 4.3 Cross-rolle constraints (gælder begge)

1. **Ingen rolleblanding med DPO/CISO/auditor**: "Application Owner og Application Provider må ikke samtidig bestride DPO-, CISO- eller compliance-auditor-roller for samme applikation (SOD-overhold, SOC 2 CC1.3, ISO 27001 A.5.3)."
2. **Ingen ad-hoc-undtagelser**: "Application Owner og Application Provider må ikke i fællesskab godkende afvigelser fra GDPR/sikkerheds-policies. Undtagelser går altid til DPO + CISO + Trine."
3. **Eskalations-pligt ved tvivl**: "Ved tvivl om GDPR-implikation eskaleres til DPO. Default-handling: hvis i tvivl, vent på DPO-svar — implementer ikke."
4. **Mandat-stabilitet under change-management**: "Application Owner og Application Provider's mandater er underlagt samme change-management-proces som øvrige roller (jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md): ændringer kræver Mads + Dorthe-vurdering + FGD-godkendelse."

### 4.4 Implementerings-anbefalinger til Laila

1. **Mandat-fil-navne** (project-rolle, ikke permanent): foreslår `application-owner-knowlagecentral.md` og `application-provider-knowlagecentral.md` — pr. applikation, ikke generisk. Dette afspejler at rollen er applikations-bundet.
2. **Frontmatter `description`**: skal indeholde tydelig disclaimer "intern projekt-rolle, ikke GDPR data controller". Dette beskytter mod fejl-routing fra Stefan.
3. **Model-anbefaling**: sonnet for begge. Application Owner kræver compliance-ræsonnement; Application Provider kræver teknisk dybde — opus er ikke nødvendigt.
4. **Tools-anbefaling fra GDPR-perspektiv**:
   - Application Owner: Read, Write/Edit (kun i `Team/projects/<app>/application-owner/` + roadmap-mapper), Glob, Grep, WebSearch, Agent, TodoWrite. **Ikke** Bash (ingen prod-adgang).
   - Application Provider: Read, Write/Edit (kun i `Team/projects/<app>/application-provider/` + tekniske docs-mapper), Glob, Grep, WebSearch, WebFetch, Agent, TodoWrite. **Ikke** Bash på prod-paths; Bash kun begrænset til read-only operations efter Mads' SOD-vurdering.
5. **Co-design med Mads**: Mads skal co-designe begge mandater for at sikre SOD-overhold + privilege-grænser. Co-designe-modellen er allerede etableret for compliance-roller.
6. **Co-design med mig (Dorthe)**: For begge mandater skal jeg verificere at GDPR-grænserne i §4.1-4.3 er korrekt indarbejdet. Dette er ikke instruks fra mig til Laila — det er nødvendig DPO-review under art. 38-mandatet.

---

## 5. Åbne spørgsmål til FGD

Disse bør Stefan koordinere svar på fra FGD før Laila skriver mandater:

1. **Application Owner vs. Application Provider — én eller to personer i praksis?** Mandaterne skriver vi som adskilte projekt-roller, men i fase 0-1 er FGD potentielt selv begge roller for knowlagecentral. SOD-konsekvens skal afklares.
2. **Skala-perspektiv**: Hvor mange applikationer forventer FGD inden for 12 mdr? Hvis kun 1-2, kan vi forenkle. Hvis 5+, skal vi designe for skala fra start.
3. **Customer-facing rolle?** Skal Application Owner kunne kommunikere direkte med kunder om compliance-spørgsmål, eller går alle compliance-svar via DPO/CISO/auditor? Min anbefaling: alle compliance-claims går via auditor/DPO, men Application Owner kan kommunikere produkt-roadmap.
4. **Joint controller-scenarier**: Hvis FGD i fremtiden får applikationer med joint-controller-relation til kunder (fx benchmarking-tjeneste), skal Application Owner-mandatet udvides? Min anbefaling: nej — joint-controller-aftaler ejes af DPO og er aldrig projekt-rolle-territorium.

---

## 6. Næste skridt

1. **Stefan** koordinerer mit udkast med FGD og Mads (CISO co-review nødvendig for SOD/security-grænser).
2. **Mads** leverer security-vinkel på samme spørgsmål (forventeligt: privilege-grænser, audit-trail-krav, SOD-overhold med eksisterende roller).
3. **Stefan** beder Laila co-designe mandater for Application Owner og Application Provider med Mads + mig som co-reviewers.
4. **Laila** producerer mandat-udkast i `Min Inbox/`.
5. **Mads + Dorthe** reviewer i parallel (SOD-uafhængighed).
6. **FGD** signoff i én runde.
7. **Camilla** committer.
8. **Trine** noterer change-event i audit-trail.

---

**Underskrevet**: Dorthe (DPO)
**Dato**: 2026-05-08
**Distribution**: Stefan (orkestrator), Laila (HR), Mads (CISO co-review), FGD (godkender)
