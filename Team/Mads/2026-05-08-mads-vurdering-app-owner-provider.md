---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: Compliance-vurdering — Application Owner og Application Provider (nye projekt-roller)
bestilt-af: Stefan (på vegne af FGD)
status: UDKAST — afventer FGD-godkendelse
relateret-arkitektur: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
relateret-governance: governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md
---

# Compliance-vurdering: Application Owner og Application Provider

**Vurderet af:** Mads (CISO)
**Dato:** 2026-05-08
**Scope:** SOD-analyse, governance-flow, security-krav, anbefalinger til Laila
**Gælder for:** Projekt-roller pr. applikation (Knowlagecentral som primært referenceeksempel)

---

## 1. SOD-analyse (Separation of Duties)

### 1.1 Rollens samlede mandat som beskrevet

Application Owner ejer tre ting simultant:

- Features og aftaler (produkt-beslutninger)
- Team-members tilknyttet applikationen (personalemæssig autoritet)
- Den overordnede roadmap (prioritering og timing)
- Ansvar for compliance (produktets compliance-posture)

Dette er et klassisk **triple-risk SOD-mønster**: én person har myndighed til at (a) beslutte hvad der bygges, (b) hvem der bygger det, og (c) afgøre om det er compliant. Kombinationen fjerner de interne checks der normalt identificerer fejl og compliance-afvigelser, inden de rammer produktion.

### 1.2 Konkrete SOD-problemer

**Problem 1 — Compliance-self-certification (kritisk)**

Hvis Application Owner ejer compliance-posture OG roadmap, kan Owner:
- Deprioritere compliance-tiltag under tidspres uden ekstern modvægt
- Erklære en feature "compliant" uden uafhængig validering
- Kontrollere hvornår og om Mads/Trine involveres

Under SOC 2 Type II CC2.2 (kommunikation af ansvar) og CC4.1 (risiko-assessment) er det et krav at compliance-vurdering er funktionelt adskilt fra dem der har incitament til at prioritere leverance over compliance. En Application Owner med begge mandater opfylder ikke dette krav.

**ISO 27001:2022 Annex A 5.3** (Segregation of duties) specificerer: "Conflicting duties and areas of responsibility shall be segregated." Roadmap-myndighed og compliance-godkendelse er per definition i konflikt — den der er presset på deadline har incitament til at godkende for hurtigt.

**Problem 2 — Team-autoritet kombineret med review-ansvar (moderat)**

Application Owner som personale-ansvarlig for team-members og compliance-ansvarlig skaber et subtilt magtmønster: teammedlemmer der finder compliance-problemer kan have svært ved at eskalere forbi Owner, fordi Owner kontrollerer deres tilknytning til projektet. Dette er en variant af den rapporteringslinje-svaghed Mads pegede på i Trine-review (fund 1, 2026-05-08-mads-4eyes-trine.md) — her på projekt-niveau.

**Problem 3 — Rolledoublement mod Mads' governance (moderat)**

Mads ejer compliance-governance for hele FGD-teamet. Trine ejer uafhængig compliance-audit. Hvis Application Owner ejer "compliance-posture" for produktet, skabes en parallel governance-struktur med uklare autoritetslinjer:

- Hvem har forrang ved konflikt: Mads' governance-rammer eller Owner's compliance-vurdering?
- Kan Owner override Mads' compliance-krav med reference til "sin" compliance-posture?
- Kan Owner afgøre at Trine ikke behøver involveres fordi compliance-posture er "Owner's ansvar"?

Dette er ikke hypotetisk — det er en forudsigelig eskalering ved tidspres eller ressourcekonflikter.

### 1.3 SOD-vurdering: samlet

Rollen som beskrevet er **ikke SOD-sund** i sin nuværende form. Den kan gøres SOD-sund med to afgrænsninger (se afsnit 4). Rollens kern — produkt-ejerskab, roadmap, team — er velfunderet og hensigtsmæssig. Det er udelukkende kombinationen med ubegrænset compliance-posture-myndighed der skaber problemerne.

---

## 2. Governance-flow

### 2.1 Korrekt placering af "compliance-posture"-ansvaret

Kompromis-formuleringen der løser SOD-problemet er:

**Application Owner er *accountable* for at compliance-input indhentes — ikke for at *vurdere* compliance.**

Distinction: Owner er den der sikrer at Mads og Trine bliver involveret til rette tid. Owner er ikke den der afgør om resultatet er tilstrækkeligt.

Dette svarer til ISO 27001:2022 Annex A 5.2 (Information security roles and responsibilities): "These responsibilities shall be defined and allocated." Owner definerer scope og tidsplan; Mads definerer compliance-krav; Trine verificerer efterlevelse.

### 2.2 Flowdiagram — compliance-hændelse

```
Application Owner (opdager/modtager compliance-hændelse eller feature med compliance-relevans)
  |
  v
Eskalér til Mads (CISO) + Dorthe (DPO) ved GDPR-relevans
  |
  v
Mads vurderer (governance/ISO 27001/SOC 2) + Dorthe vurderer (GDPR)
  |
  v
Mads/Dorthe leverer compliance-scorecard til Stefan
  |
  v
Stefan præsenterer samlet anbefaling til FGD
  |
  v
FGD giver GO/NO-GO (eller betinget GO med krav)
  |
  v
Application Owner implementerer beslutningen i roadmap
```

Owner er trigger og implementor — ikke beslutningstageren i compliance-spørgsmål.

### 2.3 Rapporteringslinjer ved compliance-hændelser

| Hændelsestype | Primær eskalering | Sekundær | Beslutning |
|---|---|---|---|
| GDPR-hændelse (databrud, DPIA-behov) | Dorthe (DPO) → Stefan | Mads cc | FGD via Stefan; Dorthe notificerer Datatilsynet selvstændigt ved tidspres (jf. art. 33) |
| Security-incident (OWASP-niveau, brud i auth/RLS) | Mads (CISO) → Stefan | Application Provider teknisk input | FGD via Stefan |
| Compliance-afvigelse (SOC 2 kontrol fejler) | Trine → Stefan → Mads cc | Application Owner informeres om findings | Trine rapporterer til FGD; Owner implementerer remediation |
| Feature med compliance-relevans (ny integration, ny datatype) | Application Owner → Mads | Dorthe ved GDPR-impact | FGD GO via Stefan efter compliance-scorecard |
| Change-management (mandat-ændring) | Standard change-request-flow | Mads + Dorthe vurderer | FGD GO |

**Nøgleprincip:** Application Owner rapporterer aldrig *til* Mads i traditionel hierarkisk forstand. Mads er governance-autoritet, ikke linjechef. Owner rapporterer compliance-triggere *til* Mads som faglig instans, og modtager governance-krav retur. Owner rapporterer fremdrift og produkt-beslutninger til FGD via Stefan.

### 2.4 Application Provider i governance-flowet

Application Provider er teknisk ressource, ikke governance-aktør. Provider:

- Leverer teknisk input til Mads' compliance-vurderinger (arkitektur, integrationer, dataflows)
- Indgår i Pouls tekniske analyser som primær teknisk sparringspartner
- Implementerer security-krav defineret af Mads efter Owner's godkendelse
- Rapporterer tekniske findings med compliance-relevans til Application Owner (som eskalerer til Mads)

Provider har **ingen selvstændig compliance-autoritet**. Provider kan og bør råbe op ved tekniske fund — men via Owner, ikke direkte til FGD.

---

## 3. Security-krav til Application Provider

Application Provider kombinerer dyb teknisk adgang med integrations-ejerskab. Det er en høj-risiko-profil under OWASP API Security Top 10 og ISO 27001:2022 Annex A 8.x (Asset management + Access control). Følgende krav er ikke forhandlingsbare.

### 3.1 Least Privilege

Provider skal tildeles adgang pr. integration, ikke blanket-adgang. Konkret for Knowlagecentral-stacken:

- Neon: read/write på relevante branches (dev, test); *ikke* direkte prod-adgang uden Change Management-godkendelse
- Vercel: kun project-level adgang, ikke team-level (fakturering, andre projekter)
- Clerk: read-only på bruger-admin; ændringer til JWT-templates kræver Camilla + Mads-godkendelse
- YubiHSM/bridge: Provider må aldrig have direkte HSM-adgang; kun via HSM-operatør (Bjarne-ækvivalent) med 4-eyes (Mads + FGD)
- Anthropic-workspaces: spend-cap-ændringer kræver FGD-godkendelse; Provider er ikke spend-cap-ejer
- Tailscale ACL: Provider kan foreslå ændringer; Camilla committer; Mads godkender ved security-impact

### 3.2 Audit Trail

Alle handlinger med compliance-relevans skal logges med attribuerbart spor. Konkret:

- Alle infrastruktur-ændringer (Vercel, Neon, Clerk-config) skal commites via Camillas git-workflow med eksplicit reference til godkendelse
- HSM-operationer: fuld audit-log med timestamp, operator-id, kommandotype — Provider må aldrig tilgå dette log direkte
- Adgangstildeling/-fratagelse: Camilla committer; Vibeke reviewer; Trine auditerer post-hoc
- API-nøgle-rotation: logges i `governance/`-mappe med dato, rotationsårsag, godkender

Immutabilitet-krav (jf. Mads' Trine-review, fund 3): logs der bruges som SOC 2-evidens skal gemmes i write-once-format eller eksporteres til separat arkiv, som hverken Provider eller producent-rollerne kan modificere.

### 3.3 Code Review Gate

Provider er per definition en person med dyb teknisk viden og potentielt bred ændrings-adgang. Code review-gate er obligatorisk:

- **Alle** PRs fra Provider-rollen reviewes af security-reviewer (projekt-rolle defineret af Mads) inden merge til `test` eller `prod`
- Security-reviewer kigger specifikt på: nye API-integrationer, ændringer til auth-flow, RLS-policies, secrets-håndtering, dataflow-ændringer
- Providers ændringer til arkitektur-kritiske komponenter (bridge, HSM-integration, Clerk JWT-template, RLS-policies) kræver eksplicit Mads-godkendelse inden deploy til prod
- PRs med compliance-relevans flagges med label og forbliver åbne til Mads har kvitteret — Provider er ansvarlig for at flagge korrekt

### 3.4 Change Management

Provider skal operere inden for Camillas change management-ramme, ikke ved siden af den:

- Ingen direkte prod-ændringer uden PR-flow (hverken via Vercel-dashboard eller CLI)
- Feature-ændringer med integrations-impact (nye leverandører, nye datatyper, ny AI-model) kræver Application Owner-godkendelse + Mads-compliance-scorecard + FGD GO inden deploy
- Infrastruktur-ændringer (branch-struktur, Neon-konfiguration, Vercel-settings) kræver Camilla som executor — Provider designer, Camilla implementerer
- Destruktive handlinger (slet data, reset HSM, drop tabel) kræver eksplicit Mads + FGD-godkendelse per hændelse

### 3.5 Leverandør-afhængigheder og GDPR-impact

Providers integrations-ejerskab medfører et ansvar: enhver ny integration eller eksisterende integrationsændring med databehandlings-relevans skal trigge DPA-vurdering via Dorthe. Provider kan ikke godkende en ny leverandørtilkobling uden DPA-afklaring. Konkret:

- Ny AI-model/leverandør: Dorthe vurderer GDPR-impact; Mads vurderer security-impact; FGD godkender
- Ændring af embeddings-pipeline (fx ny Voyage-version, ny model): Provider vurderer teknisk; Mads vurderer om zero-data-retention fortsat gælder; Dorthe vurderer databehandler-kæde
- Ekstern API kaldt med brugerdata: DPIA-trigger; Dorthe involveres fra dag 1

---

## 4. Anbefalinger til Laila

Laila skal have disse konkrete inputs med i rolle-mandaterne. De er listet som felter/begrænsninger.

### 4.1 Application Owner — påkrævede begrænsninger i mandatet

**Felt: compliance-posture (omformulér)**

Nuværende beskrivelse: "Ansvarlig for compliance (produktets compliance-posture)"

Korrekt formulering til Laila:
> "Accountable for at compliance-krav indhentes og indarbejdes — ikke for at vurdere compliance. Application Owner er obligatorisk trigger-punkt: ingen feature med compliance-relevans må gå i produktion uden at Mads (CISO) og Dorthe (DPO, ved GDPR-relevans) har leveret compliance-scorecard. Owner implementerer resultatet i roadmap; Owner afgør ikke resultatet."

**Felt: Hard rule — compliance-autoritet (tilføj)**
> "Application Owner må ikke erklære en feature, integration eller ændring som compliant. Compliance-vurdering tilhører Mads (governance/ISO 27001/SOC 2) og Dorthe (GDPR). Owner kan komme med teknisk kontekst; beslutningen er Mads'/Dorthes."

**Felt: Hard rule — team-autoritet og eskalering (tilføj)**
> "Team-members tilknyttet applikationen kan eskalere compliance-fund direkte til Mads uden at gå via Application Owner — dette er en beskyttet kanal. Owner må ikke sanktionere eller deprioritere team-members der bruger denne kanal."

**Felt: rapporteringslinje (specificér)**
> "Application Owner rapporterer produkt-beslutninger og fremdrift til FGD via Stefan. Compliance-hændelser rapporteres til Mads (og Dorthe ved GDPR-relevans). Owner modtager governance-krav fra Mads — ikke omvendt."

**Felt: Anti-patterns (tilføj)**
> - Ejer ikke compliance-vurdering — det er Mads' og Dorthes domæne
> - Kan ikke deprioritere compliance-krav i roadmap uden eksplicit FGD-godkendelse med Mads' anbefaling
> - Kan ikke involvere ny leverandør med databehandlings-relevans uden Dorthes DPA-vurdering
> - Kan ikke afgøre at Trine ikke behøver involveres i audit af produktet

### 4.2 Application Provider — påkrævede begrænsninger i mandatet

**Felt: Adgangs-profil (specificér)**
> "Least privilege pr. integration. Ingen blanket-adgang til prod-infrastruktur. Alle adgangsrettigheder specificeres ved rolle-aktivering og dokumenteres i `Team/<projektmappe>/adgangsprofil.md`. Ændringer til adgangsprofil kræver Mads-godkendelse."

**Felt: Hard rule — code review gate (tilføj)**
> "Alle PRs reviewes af security-reviewer inden merge til test eller prod. PRs med arkitektur-kritisk impact (auth, RLS, HSM, secrets, dataflow) kræver eksplicit Mads-godkendelse inden deploy til prod. Provider er ansvarlig for korrekt flagning."

**Felt: Hard rule — change management (tilføj)**
> "Ingen direkte prod-ændringer uden PR-flow. Infrastruktur-ændringer eksekuteres af Camilla — Provider designer og specificerer, Camilla implementerer. Destruktive handlinger kræver Mads + FGD-godkendelse per hændelse."

**Felt: Hard rule — leverandør-integration (tilføj)**
> "Ny integration eller ændret dataflow med brugerdata trigger obligatorisk DPA-vurdering (Dorthe) og security-vurdering (Mads) inden implementering. Provider kan ikke selv godkende leverandørtilkobling."

**Felt: compliance-eskalering (specificér)**
> "Provider rapporterer compliance-relevante tekniske fund til Application Owner, der eskalerer til Mads. Ved akut sikkerhedsrisiko (aktiv brud, secret-eksponering) kan Provider eskalere direkte til Mads — ikke til FGD."

**Felt: HSM og kryptografi-specifikke begrænsninger (tilføj)**
> "Provider må aldrig tilgå YubiHSM direkte. Alle crypto-operationer går via HSM-operatør med Mads-godkendelse. Provider kan designe og specificere kryptografi-krav; eksekvering tilhører HSM-operatør. Key-ceremony-deltagelse kræver eksplicit Mads + FGD-godkendelse."

### 4.3 Fælles for begge roller

**Felt: Audit-trail-pligt (tilføj til begge)**
> "Alle handlinger med compliance-relevans commites via Camillas git-workflow med reference til godkendelse. Ingen ad-hoc-ændringer til infrastruktur, konfiguration eller leverandøraftaler der ikke har et spor i git eller governance-mappen."

**Felt: SOD-klausul (tilføj til begge)**
> "Application Owner og Application Provider bør ikke besættes af samme person. Sammenfaldet af produkt-ejerskab og teknisk eksekvering i én person fjerner den interne kontrol der identificerer compliance-afvigelser tidligt."

---

## 5. Åbne spørgsmål til FGD/Stefan

Disse punkter kan ikke afgøres af Mads alene — de kræver FGD-beslutning eller Laila-input:

1. **Én eller to roller pr. applikation?** Roller kan i princippet besættes af to agenter, to menneskelige medarbejdere, én agent + ét menneske, osv. Mads anbefaler ikke at rolle-besætning afgøres her, men at mandaterne er skrevet til at kunne fungere med klart SOD uanset besætning.

2. **Rapporteringslinje til FGD**: Hvem af de to rapporterer fremdrift til FGD direkte? Anbefalingen er Application Owner som primær. Provider rapporterer teknisk til Owner og eskalerer compliance direkte til Mads.

3. **Feature-godkendelsesflow**: Skal Application Owner have formel godkendelses-myndighed over features (dvs. kan Owner alene sige GO til en feature) — eller kræves Stefan-koordinering for features med compliance-relevans? Mads' anbefaling: Owner kan GO features uden compliance-relevans; features MED compliance-relevans kræver compliance-scorecard (Mads/Dorthe) via Stefan til FGD.

4. **Projekt-rolle-livscyklus**: Hvornår og hvordan deaktiveres roller ved projekt-afslutning? Adgangsprofiler skal udtrykkeligt tilbagekaldes. Mads anbefaler at Laila designer en afviklings-procedure som del af rolle-mandaterne.

---

## 6. Compliance-tjekliste (SOC 2 / ISO 27001 / GDPR)

| Standard | Krav | Status med rettelser fra afsnit 4 | Krav uden rettelser |
|---|---|---|---|
| SOC 2 CC4.1 | Risk assessment uafhængig af operationel ledelse | Opfyldt (Mads/Trine uafhængige af Owner) | Ikke opfyldt |
| SOC 2 CC8.1 | Change management med separation | Opfyldt (Provider via Camilla; Mads godkender kritiske) | Delvist |
| ISO 27001 A.5.3 | Segregation of duties | Opfyldt (compliance-vurdering adskilt fra Owner) | Ikke opfyldt |
| ISO 27001 A.8.2 | Privileged access rights | Opfyldt (least privilege-felt i mandat) | Ikke opfyldt |
| ISO 27001 A.8.32 | Change management | Opfyldt (PR-flow + Camilla-executor) | Delvist |
| GDPR art. 25 | Privacy by design | Opfyldt (DPA-trigger ved ny integration) | Ikke opfyldt (Provider kan godkende leverandør selvstændigt) |
| GDPR art. 28 | Databehandleraftaler | Opfyldt (Dorthe DPA-vurdering obligatorisk) | Ikke opfyldt |
| GDPR art. 38(3) | DPO-uafhængighed | Opfyldt (Owner kan ikke override Dorthe) | Ikke specificeret |

---

## 7. Samlet anbefaling

**CONDITIONAL GO** til begge roller — med betingelse om at Laila inkorporerer begrænsningerne fra afsnit 4 i mandaterne inden aktivering.

Kernedesignet er godt: Application Owner som produkt-ansvarlig og Application Provider som teknisk ressource er en sund og praktisk model. Det er en naturlig pendant til developer-ejer og teknisk lead i traditionelle teams.

Justeringerne handler ikke om at svække rollerne — de handler om at give dem præcise grænser, så de kan fungere inden for FGD's compliance-ramme uden at skabe parallelle governance-strukturer der underminerer Mads' og Trines uafhængighed.

Den eneste alternative model, der ville løse SOD fuldt ud uden begrænsningerne, er at fjerne "compliance-posture" helt fra Application Owners mandat og erstatte det med "compliance-koordinering" (at sikre involvering — ikke at vurdere resultater). Det er hvad rettelsen i afsnit 4.1 reelt gør.

**Næste skridt:**

1. Stefan præsenterer denne vurdering for FGD og afventer GO.
2. FGD giver GO → Laila designer rolle-mandater med begrænsningerne fra afsnit 4.
3. Mads laver 4-eyes review af Lailas mandatudkast inden aktivering.
4. Trine logger rolle-aktivering i audit-trail (når Trine er aktiveret).
5. Camilla committer mandaterne efter FGD-godkendelse.

---

*Mads (CISO) — 2026-05-08*
