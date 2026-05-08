---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: 4-eyes review af Lailas mandat-udkast — Application Owner og Application Provider
status: leveret
modtagere: Stefan (orkestrator), Laila (HR), FGD (godkender), Mads (CISO co-review)
type: 4-eyes-review
review-resultat: CONDITIONAL GO
referencer:
  - Team/Dorthe/2026-05-08-dorthe-vurdering-app-owner-provider.md (min oprindelige GDPR-vurdering)
  - Team/Laila/2026-05-08-udkast-application-owner-mandat.md (under review)
  - Team/Laila/2026-05-08-udkast-application-provider-mandat.md (under review)
  - GDPR art. 30 (ROPA), art. 35 (DPIA), art. 38(3) (DPO-uafhængighed), art. 38(6) (interessekonflikt)
  - EDPB Guidelines 04/2017 (DPIA, WP248)
---

# 4-eyes review: Application Owner + Application Provider mandat-udkast

## 0. Sammenfatning

**Resultat: CONDITIONAL GO**

Lailas udkast er solide og afspejler i hovedsagen mine 8 + 6 hard rules + cross-rolle constraints korrekt. Strukturen er ren, SOD-kravet er eksplicit, og DPIA-flowet er operationelt. Men der er **5 specifikke rettelser** der skal foretages inden FGD-godkendelse + aktivering, plus **3 anbefalinger** som forbedrer GDPR-præcision men ikke blokerer for go.

**Blokerende rettelser (skal ind før aktivering):**
1. Application Owner-mandatet mangler én af mine 8 hard rules (cross-rolle constraint om mandat-stabilitet er flettet ind, men cross-rolle constraint om "ingen ad-hoc-undtagelser" er udeladt eller for svagt formuleret).
2. Provider-mandatet udelader eksplicit henvisning til DPO-uafhængighed (art. 38(3)) — ingen af rollerne må kunne instruere mig om DPO-konklusioner.
3. Provider-hard rule §"Audit-trail på alt dataflow-relevance" mangler eksplicit konsekvens hvis ikke logget (jeg foreslog "manuelle handlinger uden audit-trail er forbudt på prod" + at det er GDPR art. 30/32-brud).
4. Provider-mandatet mangler eksplicit cross-rolle eskalations-pligt ved tvivl ("default: hvis i tvivl, vent på DPO-svar — implementer ikke").
5. ROPA-notifikationspligten (5 hverdage) er beskrevet, men selve notifikations-formatet (filsti `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med specifikke felter) er udeladt i Provider-mandatet.

**Ikke-blokerende anbefalinger:**
- Application Owner-mandat: tilføj eksplicit forbud mod at instruere DPO om konklusioner.
- Provider-mandat: præcisér at "dataflow-ændring" inkluderer mindre ændringer (ny env-var pegende på ny endpoint).
- Begge mandater: stavefejl og uklar tegnsætning (Camilla retter; ikke compliance-blokerende).

---

## 1. Spørgsmål 1 — Application Owner: er mine 8 hard rules korrekt indarbejdet?

Mine 8 hard rules fra §4.1 i min vurdering, krydstjekket mod Lailas udkast §"Hard rules":

| # | Min hard rule (§4.1) | Lailas indarbejdelse | Status |
|---|---|---|---|
| 1 | Juridisk afgrænsning (ikke "data controller") | "Du er ikke 'data controller' i GDPR-juridisk forstand. FGD som juridisk enhed (CVR) er controller..." | **OK** |
| 2 | DPO-notifikation før tredjelands-overførsler | "Du må ikke godkende tredjelandsoverførsler... Dorthe (DPO) notificeres og vurdere GDPR art. 44-49 inden du godkender" | **OK** |
| 3 | DPA før integration | "DPA før integration i prod. Ny leverandør med databehandling-relevans... må ikke aktiveres i production før DPA er gennemgået af Dorthe og Mads" | **OK** |
| 4 | DPIA-screening-pligt | "Du initierer DPIA-screening for alle nye features før implementeringsbeslutning. Hvis screening udløser DPIA-krav, deployes feature ikke til prod uden FGD-signeret DPIA." | **OK** |
| 5 | Kvartalsvis ROPA-attest | "Hver 3. måned signerer du attest til Dorthe + Trine..." inkl. 14-dages-eskalation | **OK** |
| 6 | Compliance-claims | "Du må ikke erklære produktet 'GDPR-compliant'... uden skriftlig anbefaling fra Dorthe (GDPR) eller Trine (ISO/SOC 2)" | **OK med bemærkning** — jeg skrev oprindeligt "DPO (GDPR) eller compliance-auditor (ISO/SOC 2)"; Lailas formulering er rolle-specifik ("Trine") frem for funktion-specifik. Da rollen er navngivet, er det acceptabelt, men hvis Trine skifter, skal mandatet også opdateres. **Rettelse-anbefaling**: tilføj parentes "(eller den til enhver tid værende compliance-auditor)". |
| 7 | Eskalations-pligt ved data-subject-anmodninger, breach, Datatilsynet | "Du eskalerer alle art. 15-22-anmodninger... breach-mistanker, og Datatilsynets henvendelser til Dorthe inden 24 timer." | **OK** |
| 8 | Retention-beslutninger | "Du kan foreslå retention-perioder for data; godkendelse tilhører Dorthe (GDPR), Mads (sikkerhed), og Trine (ISO/SOC 2-overhold)." | **OK** |

**Cross-rolle constraints (§4.3) — også relevante for Owner-mandatet**:

| # | Cross-rolle constraint | Indarbejdelse | Status |
|---|---|---|---|
| C1 | Ingen rolleblanding (SOD) | "Du er ALDRIG samme person som Application Provider" | **OK** (eksplicit + tydelig) |
| C2 | Ingen ad-hoc-undtagelser | **MANGLER eksplicit** — jeg ser ikke en hard rule om at Owner og Provider ikke i fællesskab kan godkende afvigelser fra GDPR-policies | **BLOKERENDE rettelse** |
| C3 | Eskalations-pligt ved tvivl ("hvis i tvivl, vent — implementer ikke") | Implicit men ikke eksplicit hard rule | **Anbefalet rettelse** |
| C4 | Mandat-stabilitet under change-management | "Reglerne kan ikke ændres uden formel change-management-proces" | **OK** |

**Vurdering Owner**: 8/8 hard rules + 2/4 cross-rolle constraints korrekt. C2 er blokerende; C3 er anbefalet.

---

## 2. Spørgsmål 2 — Application Provider: er mine 6 hard rules korrekt indarbejdet?

Mine 6 hard rules fra §4.2 i min vurdering, krydstjekket:

| # | Min hard rule (§4.2) | Lailas indarbejdelse | Status |
|---|---|---|---|
| 1 | Dataflow-notifikations-pligt (5 hverdage) | "Du notificerer Dorthe inden 5 hverdage ved any af følgende: ny third-party-integration, nyt PII-felt, ændret retention-periode, ny data-kategori, ny tredjelands-overførsel, processor-status-ændring, dataflow-arkivering, ny audit-log-kategori med PII." | **OK med bemærkning** — listen er der, men **konsekvens** ("manglende notifikation behandles som GDPR art. 30-brud") er udeladt. **Rettelse-anbefaling**. |
| 2 | Ingen direkte prod-PII-adgang som standard | "Prod-PII-adgang is restricted... break-glass-procedure — skriftlig begrundelse, CISO-godkendelse, automatisk audit-trail, maksimal session 4 timer." | **OK** |
| 3 | Ingen DPA-forhandling | "Ingen DPA-forhandling eller compliance-claims. Du må ikke forhandle DPA'er med leverandører..." | **OK** |
| 4 | Ingen compliance-vurderinger udadtil | "...eller udtale dig om GDPR/ISO/SOC 2-compliance til kunder/revisorer på vegne af FGD." | **OK** |
| 5 | Audit-trail på alle dataflow-relaterede handlinger | "Audit-trail på alt dataflow-relevance. Alle dine handlinger der berører dataflows (env-vars, konfigurationer, integration-credentials, API-endpoints) skal være loggede til `audit.events` med 3-års retention. Manuelle handlinger uden audit-trail på prod er forbudt." | **OK med bemærkning** — mangler eksplicit at det er GDPR art. 30/32-brud + at der er konsekvens (Trine fanger ved audit). **Rettelse-anbefaling**. |
| 6 | DPIA-input-pligt (5 hverdage) | "Hvis Dorthe beder om teknisk dataflow-input til DPIA, leverer du inden 5 hverdage. Manglende input blokerer feature-deploy." | **OK** |

**Cross-rolle constraints (§4.3) for Provider**:

| # | Cross-rolle constraint | Indarbejdelse | Status |
|---|---|---|---|
| C1 | Ingen rolleblanding (SOD) | "Du er ALDRIG samme person som Application Owner" | **OK** |
| C2 | Ingen ad-hoc-undtagelser (Owner+Provider sammen kan ikke godkende afvigelser) | **MANGLER** | **BLOKERENDE rettelse** |
| C3 | Eskalations-pligt ved tvivl | **MANGLER eksplicit** | **BLOKERENDE rettelse** for Provider (Provider har dyb teknisk adgang og kan implementere uden at vente — risiko er højere end for Owner) |
| C4 | Mandat-stabilitet | "Reglerne kan ikke ændres uden formel change-management-proces" | **OK** |

**Vurdering Provider**: 6/6 hard rules indholdsmæssigt OK med 2 mindre præciseringer. Cross-rolle constraints C2 + C3 er blokerende.

---

## 3. Spørgsmål 3 — DPIA-flow: er Owner→Dorthe→Provider operationelt beskrevet?

Min DPIA-flow fra §3.2-3.3 krydstjekket:

**Min specifikation**:
- Owner initierer DPIA-screening ved feature-roadmap-beslutning
- Dorthe afgør DPIA-krav (~3 hverdage)
- Provider leverer teknisk dataflow-input (~5 hverdage)
- Dorthe + Mads + Provider designer mitigations (~5-10 hverdage)
- Dorthe leverer DPIA-konklusion til FGD
- FGD signerer eller anmoder om revision
- Gate: ingen feature med DPIA-trigger må deployes til prod uden FGD-signeret DPIA

**Lailas indarbejdelse**:

I Owner-mandat §"Arbejdsgang" trin 3: "GDPR-screening (hvis relevant): Du initierer DPIA-screening med Dorthe (jf. `governance/dpa/dpia-screening-template.md`)". I §"Hard rules" — DPIA-screening-pligt + gate "deployes feature ikke til prod uden FGD-signeret DPIA". I §"Hand-off-aftaler": "Dorthe afgør DPIA-behov inden 3 hverdage. Hvis DPIA påkrævet: du sikrer Application Provider leverer dataflow-input inden 5 hverdage."

I Provider-mandat §"Arbejdsgang" trin 3: "DPIA-input: Hvis feature har GDPR-relevans, leverer du teknisk dataflow-input til Dorthe inden 5 hverdage". I §"Hard rules" — DPIA-input-pligt + "Manglende input blokerer feature-deploy". I §"Hand-off-aftaler": "DPIA-input leveret inden 5 hverdage efter anmodning (blokerer feature-deploy hvis forsinket)."

**Vurdering**: Flowet er **operationelt godt beskrevet**. Trin 1 (Owner initierer) og trin 4 (Provider leverer) er klart placeret. Min afgørelses-rolle (trin 2 — Dorthe afgør) er tydelig i Owner's hand-off-aftaler, men **udelades** i Provider-mandatet — Provider ser kun "leverer input efter anmodning fra Dorthe" uden eksplicit at vide at Dorthe ejer DPIA-konklusionen.

**Rettelse-anbefaling (ikke-blokerende)**: Tilføj én sætning i Provider-mandat §"Ansvarsfordeling med andre roller" under "Dorthe (DPO)": "Dorthe ejer DPIA-konklusion og mitigation-godkendelse; Provider leverer teknisk input men træffer ikke selv DPIA-beslutning."

Også: i Owner-mandat §"Arbejdsgang" trin 5 står "FGD-GO: Stefan koordinerer komplet scorecard-set til FGD". For DPIA specifikt skal det være FGD som signerer DPIA (juridisk DPO-registrant) — det er beskrevet i hard rule, men kunne styrkes i flow-beskrivelsen.

**Status spørgsmål 3**: GO med små anbefalinger.

---

## 4. Spørgsmål 4 — ROPA-notifikationspligt (5 hverdage): tydelig nok?

Min specifikation fra §2.2:

**Application Provider skal notificere Dorthe inden 5 hverdage ved 8 specifikke triggers**:
1. Ny tredjeparts-integration
2. Nyt PII-felt
3. Ændret retention-periode
4. Ny data-kategori opsamles
5. Ny tredjelands-overførsel
6. Processor-status-ændring
7. Sletning/arkivering af processing-aktivitet
8. Ny audit-log-kategori med PII

**Notifikations-format**: Note i `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med specifikke felter (hvad ændrer sig, hvilken processing-aktivitet i ROPA berøres, forventet implementeringsdato, drevet af hvem).

**Konsekvens ved manglende notifikation**: GDPR art. 30-brud, change-request blokeres, Provider skal redegøre.

**Lailas indarbejdelse i Provider-mandat**:

§"Hard rules" → "GDPR-relateret (fra Dorthe)": "Dataflow-notifikations-pligt: Du notificerer Dorthe inden 5 hverdage ved any af følgende: ny third-party-integration, nyt PII-felt, ændret retention-periode, ny data-kategori, ny tredjelands-overførsel, processor-status-ændring, dataflow-arkivering, ny audit-log-kategori med PII."

Også: §"Hard rules" → "Leverandør-integration & DPA": "Dataflow-notifikation til Dorthe. Når du ændrer existerende integration (fx ændring af hvilke felter der sendes til Voyage), notificerer du Dorthe inden 5 hverdage (jf. Dorthes procedure `Team/Dorthe/inbox/`)."

**Vurdering**:

- **Listen af 8 triggers**: 8/8 inkluderet. **OK**.
- **Notifikations-format (specifik filsti + felter)**: Filstien `Team/Dorthe/inbox/` er nævnt, men de specifikke felter ("hvad ændrer sig, hvilken processing-aktivitet i ROPA berøres, forventet implementeringsdato, drevet af hvem") er **udeladt**. Dette gør formatet uklart for Provider — kan resultere i sub-standard notifikationer der ikke kan bruges direkte til ROPA-vedligehold. **BLOKERENDE rettelse**.
- **Konsekvens (GDPR art. 30-brud)**: **MANGLER**. Uden eksplicit konsekvens er notifikationspligten "blød" — Provider kan rationalisere "det var bare en lille ændring". **BLOKERENDE rettelse**.

**Rettelses-tekst (anbefales tilføjet til Provider-hard rule)**:

> Notifikations-format: kort note i `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med følgende minimum-felter: (a) hvad ændrer sig, (b) hvilken processing-aktivitet i ROPA berøres, (c) forventet implementeringsdato, (d) hvem der har drevet ændringen. Manglende eller ufuldstændig notifikation behandles som GDPR art. 30-brud, blokerer change-request, og Provider skal redegøre over for DPO + Trine.

**Status spørgsmål 4**: CONDITIONAL — listen er der, men format og konsekvens skal præciseres.

---

## 5. Spørgsmål 5 — DPO-uafhængighed (art. 38(3)) tilstrækkeligt beskyttet?

Min vurdering §1.3 kræver fire grænser som hard rules:

1. Owner (eller Provider) instruerer ikke DPO om konklusioner — kan bestille vurdering, ikke diktere svar
2. DPO-uafhængighed bevares ved eskalation (jeg eskalerer direkte til FGD, ikke gennem Owner)
3. Owner kan ikke fyre/sanktionere DPO
4. Compliance-rapportering går ikke gennem Owner (Owner får kopi, ikke filter-position)

**Lailas indarbejdelse**:

I **Owner-mandat**:

- §"Mission" siger "Du er ikke compliance-vurderer — det er Mads (CISO) og Dorthe (DPO)" — implicit beskytter §1.3-grænse 1, men ikke eksplicit.
- §"Hard rules" → "Compliance-grænser": "Compliance-vurdering er IKKE dit ansvar. Du er ansvarlig for at få det gjort; Mads og Dorthe er ansvarlige for resultatet. Du må ikke erklære en feature 'compliant' uden skriftlig anbefaling fra hhv. Dorthe (GDPR) og Mads (ISO/SOC 2)." — beskytter grænse 1.
- §"Hard rules" → "Du må ikke deprioritere compliance-krav" + "team-members kan eskalere compliance-fund direkte til Mads uden at gå gennem dig" — beskytter grænse 4 delvist (men eksplicit kun for Mads, ikke Dorthe).
- **MANGLER eksplicit**: Hard rule om at Owner ikke kan instruere DPO om konklusioner; at DPO eskalerer direkte til FGD ved uenighed; at compliance-rapportering ikke filtreres gennem Owner.

I **Provider-mandat**:

- §"Hard rules" → "Ingen DPA-forhandling eller compliance-claims" — beskytter grænse 1 delvist.
- **MANGLER eksplicit**: Tilsvarende DPO-uafhængighedsbeskyttelse — Provider har dyb teknisk adgang og kunne i teorien modificere data eller konfigurationer i et forsøg på at "påvirke" min vurdering. Eksplicit hard rule om at Provider ikke må forsøge at påvirke DPO-konklusion er fraværende.

**Vurdering**: DPO-uafhængigheden er **delvist** beskyttet via "compliance-vurdering tilhører Mads/Dorthe" og "compliance-claims kræver skriftlig anbefaling", men **ikke eksplicit**. Dette er **BLOKERENDE rettelse** — art. 38(3) er en kerne-GDPR-rettighed, og hvis ikke eksplicit beskyttet i mandaterne, risikerer vi at Owner/Provider med tiden udøver pres uden at vide det er forbudt.

**Rettelses-tekst (anbefales tilføjet til BEGGE mandater under "Hard rules" som ny gruppe "DPO-uafhængighed")**:

> **DPO-uafhængighed (GDPR art. 38(3))**:
> - Du må ikke instruere DPO (Dorthe) om konklusionen i en GDPR-vurdering. Du kan bestille en vurdering eller anmode om uddybning, men ikke diktere svaret.
> - Hvis du og DPO er uenige om en GDPR-konklusion, eskalerer DPO direkte til FGD (juridisk DPO-registrant) — ikke gennem dig.
> - Du må ikke filtrere, redigere eller forsinke DPO's compliance-rapportering til FGD. Du modtager kopi af relevante leverancer, ikke filter-position.
> - Du må ikke sanktionere, deprioritere eller på anden måde lægge pres på team-members eller DPO for at have rapporteret GDPR-fund.

**Status spørgsmål 5**: BLOKERENDE rettelse nødvendig — eksplicit DPO-uafhængighedssektion mangler i begge mandater.

---

## 6. Andre observationer (ikke-blokerende, men værd at nævne)

### 6.1 Sproglige/redaktionelle bemærkninger

Begge udkast indeholder mindre sproglige fejl der bør rettes inden aktivering. Eksempler:
- Owner: "Kvadratsvis attest" (skal være "Kvartalsvis"), "tilbuddet" (skal være "tilbudt"), "Lagd af" (skal være "Lagt af" eller "Forfattet af"), "pejlmarker" (uklart — formentlig "pejlemærker"), "ansvarlig" gentages mistænkeligt ofte uden at være skrevet helt rigtigt.
- Provider: "Application Provider's" (apostrof-brug på dansk skal være "Application Providers"), "krænkedes" i §"Spørgsmål til FGD" (uklart — formentlig "kræver"), "Keine secrets" (tysk indsmuttet — skal være "Ingen secrets"), "tilsidesat går ikke" (uklart sætning).

Camilla retter disse rutinemæssigt før commit (jf. memory: "Camilla retter stavefejl og grammatik"). Ikke compliance-blokerende, men leverer dårligt indtryk hvis aktiveret med fejl.

### 6.2 Mandat-stabilitet — referenceret §3.3

Begge udkast siger "Reglerne kan ikke ændres uden formel change-management-proces (jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md § 3.3)". 

**Bemærkning**: Per FGD-override 2026-05-06 (citeret i CLAUDE.md) er §3.3 superseded — alle mandat-ændringer kræver Mads + Dorthe sammen, ikke trigger-baseret skift. Mandat-tekst kan beholde §3.3-reference forudsat policy-banneret er opdateret. Dette skal Mads bekræfte i hans 4-eyes; jeg flagger det som koordinations-punkt mellem mit og Mads' review.

### 6.3 ROPA-multi-app-skalering nævnes ikke i Owner-mandatet

Min §2.4 anbefaler at ROPA struktureres som én master-ROPA pr. juridisk enhed med kapitler pr. applikation. Owner's kvartalsvise attest er beskrevet, men det er ikke gjort eksplicit at Owner ejer **sit kapitel** i master-ROPA — ikke en separat ROPA-fil pr. applikation. Dette kan misforstås når flere applikationer tilføjes.

**Anbefaling**: Tilføj i Owner-mandat §"Mission" eller §"Arbejdsgang": "Application Owner ejer ROPA-kapitlet for sin applikation; master-ROPA-strukturen ejes af DPO. Owner attesterer kapitlet kvartalsvis."

Ikke-blokerende, men gør skala-håndtering renere når Owner-rolle udvides til andre apps.

### 6.4 Forudgående høring til Datatilsynet (art. 36) ikke nævnt

Min §3.2 nævner "Forudgående høring (art. 36) — FGD indsender til Datatilsynet hvis høj-risk uden tilfredsstillende mitigation". Ingen af mandaterne nævner dette scenarie eksplicit — Owner kunne i teorien træffe beslutning om at gå videre med en feature der egentlig krævede art. 36-høring.

**Anbefaling (ikke-blokerende)**: Tilføj i Owner-mandat §"Hard rules": "Ved DPIA-resultat 'høj risiko uden tilfredsstillende mitigation' kræver feature forudgående høring til Datatilsynet (GDPR art. 36) inden deploy. Høring koordineres af DPO; FGD indsender. Owner må ikke deploye feature før Datatilsynets svar foreligger eller frist er udløbet."

---

## 7. Konkluderende vurdering pr. spørgsmål

| # | Spørgsmål | Status |
|---|---|---|
| 1 | Owner: 8 hard rules korrekt indarbejdet? | **GO** med 1 blokerende cross-rolle-rettelse (C2 mangler) |
| 2 | Provider: 6 hard rules korrekt indarbejdet? | **GO** med 2 blokerende cross-rolle-rettelser (C2 + C3) + 2 præciseringer (konsekvens-tekst) |
| 3 | DPIA-flow operationelt beskrevet? | **GO** med små anbefalede præciseringer |
| 4 | ROPA-notifikationspligt (5 hverdage) tydelig? | **CONDITIONAL** — listen er der; format + konsekvens skal præciseres (BLOKERENDE) |
| 5 | DPO-uafhængighed (art. 38(3)) tilstrækkeligt beskyttet? | **BLOKERENDE** — eksplicit DPO-uafhængighedssektion mangler i begge mandater |

**Samlet review-resultat: CONDITIONAL GO**

---

## 8. Konkrete rettelser der skal foretages inden aktivering (blokerende)

Til Laila (via Stefan):

### 8.1 Application Owner-mandat — rettelser

**R1 (blokerende)** — tilføj cross-rolle constraint C2 som ny hard rule:
> Ingen ad-hoc-undtagelser i fællesskab med Application Provider: Owner og Provider må ikke i fællesskab godkende afvigelser fra GDPR/sikkerheds-policies. Undtagelser kræver eksplicit godkendelse fra DPO + CISO + Trine.

**R2 (blokerende)** — tilføj DPO-uafhængighedssektion (4 punkter, se §5 ovenfor).

**R3 (anbefalet)** — præcisér compliance-claims-rule: "...uden skriftlig anbefaling fra hhv. Dorthe (GDPR) eller Trine (ISO/SOC 2) — eller den til enhver tid værende compliance-auditor".

**R4 (anbefalet)** — tilføj art. 36 forudgående høring til hard rules (se §6.4).

**R5 (anbefalet)** — tilføj eskalations-pligt ved tvivl som hard rule:
> Eskalations-pligt ved tvivl: Hvis du er i tvivl om GDPR-implikation eller compliance-konsekvens af en feature-beslutning, eskalerer du til DPO inden implementering. Default: hvis i tvivl, vent på DPO-svar — implementer ikke.

**R6 (anbefalet)** — præcisér ROPA-multi-app-struktur: Owner ejer kapitlet, ikke separat fil (se §6.3).

### 8.2 Application Provider-mandat — rettelser

**R7 (blokerende)** — tilføj cross-rolle constraint C2 (ad-hoc-undtagelser, samme tekst som R1).

**R8 (blokerende)** — tilføj cross-rolle constraint C3 (eskalations-pligt ved tvivl):
> Eskalations-pligt ved tvivl: Hvis du er i tvivl om GDPR-implikation eller compliance-konsekvens af en teknisk beslutning, eskalerer du til DPO inden implementering. Default: hvis i tvivl, vent på DPO-svar — implementer ikke.

**R9 (blokerende)** — tilføj DPO-uafhængighedssektion (samme 4 punkter som R2).

**R10 (blokerende)** — præcisér ROPA-notifikations-format og konsekvens:
> Notifikations-format: kort note i `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med minimum-felter: (a) hvad ændrer sig, (b) hvilken processing-aktivitet i ROPA berøres, (c) forventet implementeringsdato, (d) hvem der har drevet ændringen. Manglende eller ufuldstændig notifikation behandles som GDPR art. 30-brud, blokerer change-request, og Provider skal redegøre over for DPO + Trine.

**R11 (anbefalet)** — præcisér audit-trail-konsekvens:
> Manuelle handlinger uden audit-trail på prod er forbudt og behandles som GDPR art. 30/32-brud (manglende dokumentation af processing + utilstrækkelige sikkerhedsforanstaltninger).

**R12 (anbefalet)** — tilføj i §"Ansvarsfordeling" under Dorthe: "Dorthe ejer DPIA-konklusion og mitigation-godkendelse; Provider leverer teknisk input men træffer ikke selv DPIA-beslutning."

### 8.3 Begge mandater — sproglige rettelser (Camilla/Laila)

**R13 (ikke-blokerende, men leverer indtryk)** — Laila eller Camilla retter sproglige fejl listet i §6.1.

---

## 9. Næste skridt

1. **Stefan** modtager dette 4-eyes review + Mads' parallelle 4-eyes (separat fil).
2. **Stefan** koordinerer: Laila reviderer udkastene baseret på begge 4-eyes-reviews.
3. **Laila** producerer revideret v2 i `Team/Laila/2026-05-08-udkast-application-owner-mandat-v2.md` + tilsvarende for Provider.
4. **Mads + Dorthe** laver final review (kort, kun for at bekræfte rettelser er korrekt indarbejdet).
5. **FGD** modtager final v2 + samlet 4-eyes-godkendelse fra Mads + Dorthe → signoff.
6. **Camilla** opretter mandat-filer i `.claude/agents/` efter FGD-go.
7. **Trine** logger change-event i audit-trail.

---

## 10. DPO-uafhængighedserklæring

Dette 4-eyes review er udført uafhængigt af Mads' SOD-vurdering (jeg har ikke set Mads' parallelle 4-eyes ved skrivetidspunkt). Min vurdering er udelukkende baseret på:
- Min egen oprindelige GDPR-vurdering (`2026-05-08-dorthe-vurdering-app-owner-provider.md`)
- Lailas to udkast
- GDPR-tekst og EDPB Guidelines

Eventuelle uoverensstemmelser med Mads' vurdering eskaleres til Stefan, der koordinerer afklaring uden at instruere mig om konklusionen (art. 38(3)).

---

**Underskrevet**: Dorthe (DPO)
**Dato**: 2026-05-08
**Review-resultat**: CONDITIONAL GO — 5 blokerende rettelser (R1, R2, R7, R8, R9, R10) + 6 anbefalede (R3, R4, R5, R6, R11, R12) + sproglige (R13)
**Distribution**: Stefan (orkestrator), Laila (HR), Mads (CISO co-review), FGD (godkender)
