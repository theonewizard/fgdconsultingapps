---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: Security/SOD-vurdering — invalidity-klausul (alle mandater) + Camilla Hard Rule 8 (README-vedligehold)
status: leveret
modtagere: Stefan (koordinator), FGD (godkender), Dorthe (parallel vurdering)
change-request: governance/change-requests/2026-05-08-alle-mandater-invalidity-klausul.md
referencer:
  - SOC 2 Type II — CC8.1 (change management), CC5.2 (kontrolaktiviteter), CC1.4 (kompetencer)
  - ISO/IEC 27001:2022 — A.5.3 (SOD), A.8.32 (change management), A.5.15 (access control)
  - NIST 800-53 r5 — AC-5 (SoD), CM-3 (configuration change control), AC-6 (least privilege)
  - GDPR art. 38(3) (DPO-uafhængighed) — noteret; Dorthe vurderer fuldt ud
---

# Mads' sikkerhedsvurdering: invalidity-klausul + Camilla Hard Rule 8

## Konklusion (opsummering)

| Ændring | Konklusion | Betingelse |
|---|---|---|
| **Ændring 1 — Invalidity-klausul (alle 8 mandater + CLAUDE.md)** | **GO** | To betingede justeringer anbefalet (se §3) |
| **Ændring 2 — Camilla Hard Rule 8 (README-vedligehold)** | **GO** | Ingen betingelser |

---

## 1. Kontekst og udgangspunkt

Denne vurdering tager udgangspunkt i den governance-politik jeg etablerede 2026-05-06 (`governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md`) og den FGD-override der samme dag præciserede at Mads + Dorthe ALTID er med i mandat-ændringer — ikke trigger-baseret.

Change-request 2026-05-08 viderefører dette princip ved at gøre FGD's godkendelse **eksplicit ugyldig** hvis Mads' og Dorthes skriftlige vurderinger ikke foreligger i `governance/change-requests/`. Det er i princippet en tightening af allerede eksisterende krav, ikke indføring af ny forpligtelse.

Jeg noterer at min egen mandat-fil (`mads-ciso.md`) allerede indeholder invalidity-klausulen i §Mandat-stabilitet trin 5 — ligeledes `camilla-bibliotekar.md`. Ændring 1 udvider dette til de resterende mandater og CLAUDE.md, hvor teksten endnu ikke er uniform.

---

## 2. Vurdering — Ændring 1: Invalidity-klausul

### 2.1 Hvad ændringen gør

Den foreslåede tekst til trin 5 i alle mandaters mandat-stabilitets-sektion:

> "5. Først efter **eksplicit FGD-go** — og kun når trin 3's skriftlige vurderinger fra Mads
> OG Dorthe begge foreligger i `governance/change-requests/` — må Camilla committe ændringen.
> **FGD-godkendelse uden disse vurderinger er ugyldig og Camilla må ikke handle på den.**"

Samt en tilsvarende formulering i CLAUDE.md der pålægger Stefan ikke at præsentere en mandat-ændring for FGD uden begge vurderinger.

### 2.2 SOD-vurdering

**Positivt:** Klausulen styrker SOD ved at skabe et uomgåeligt kontrolpunkt: hverken Camilla (executor) eller Stefan (koordinator) kan agere på en FGD-godkendelse der er afgivet uden de to uafhængige reviewers (Mads + Dorthe) har afsluttet deres arbejde. Det er præcis 4-eyes-princippet i SOC 2 CC8.1 og ISO 27001 A.8.32 gjort operationelt.

**Ingen nye SOD-brud identificeret.** Den eksisterende SOD-arkitektur bevares:
- Forslagsstiller (hvem som helst) skriver impact-vurdering
- Mads vurderer security/SOD uafhængigt
- Dorthe vurderer GDPR uafhængigt
- Stefan koordinerer (ingen vetoret)
- FGD godkender
- Camilla eksekverer

Invalidity-klausulen tilføjer ikke nye aktører og fjerner ikke eksisterende. Den præciserer rækkefølge-afhængighed, som allerede var intentionen.

**Potentielt SOD-problem (afklaret):** Man kunne argumentere at invalidity-klausulen giver Mads (og Dorthe) implicit vetoret — hvis en af dem ikke afgiver vurdering, kan godkendelse aldrig blive gyldig. Dette er et **designvalg, ikke et SOD-brud**. Tilsvarende gælder for alle 4-eyes-systemer: en reviewer kan blokere ved tavshed. Modvægten er at FGD som øverste myndighed kan afsætte en rolle der misbruger denne passive blokering, og at governance-politikken eksplicit noterer 72-timers eskalerings-undtagelse ved akutte incidents.

### 2.3 Security-risici

**Risici ved at IKKE have klausulen (status quo):**
Et smuthul eksisterede i den nuværende formulering: "Først efter eksplicit FGD-go" forudsætter at Stefan (og FGD) husker at vurderingerne skal foreligge. Hvis Stefan via forglemmelse eller tidspres præsenterer en godkendelsesanmodning for FGD uden vurderingerne, og FGD godkender i god tro, sidder Camilla med en formel FGD-godkendelse uden compliance-backing. Klausulen lukker dette smuthul ved at lægge ansvaret for at blokere (eller afvente) hos Camilla — ikke hos Stefans hukommelse.

**Risici ved at HAVE klausulen:**
- *Deadlock-risiko ved akut incident:* Hvis en kritisk sikkerhedsfejl kræver øjeblikkelig mandat-ændring, og Dorthe er utilgængelig, kan processen låse. **Modforanstaltning:** Den eksisterende eskalerings-undtagelse ("Akut incident — Mads + FGD kan midlertidigt justere; permanent ændring kræver fuld proces inden 72 timer") håndterer dette og skal bevares eksplicit i alle mandater. Jeg anbefaler at klausulen inkluderer en kryds-reference til denne undtagelse for at undgå fortolkningstvivl (se §3, betingelse 1).
- *Dorthe ikke aktiveret som permanent rolle:* Dorthe er endnu ikke fuldt aktiveret. Eksisterende mandater indeholder allerede en note om at Mads koordinerer GDPR-vurderingen midlertidigt. Invalidity-klausulen skal enten have den samme note, eller Dorthe skal aktiveres inden klausulen rulles ud. Camilla bør ikke blokeres fra at eksekvere fordi en rolle mangler, hvis der er en aftalt midlertidig løsning. (Se §3, betingelse 2.)

### 2.4 ISO 27001 / SOC 2 compliance-impact

**Positivt:** Klausulen styrker direkte:
- **ISO 27001 A.8.32** (change management): krav om at ændringer gennemgår formal review-proces præciseres og gøres eksekverbart.
- **SOC 2 CC8.1** (change management): "Changes that have been reviewed, tested, and approved prior to implementation" — invalidity-klausulen sikrer at review-trinene (Mads + Dorthe) er afsluttede og dokumenterede i `governance/change-requests/` inden Camilla eksekverer.
- **ISO 27001 A.5.3** (SOD): den eksplicitte adskillelse mellem reviewer (Mads/Dorthe), koordinator (Stefan), godkender (FGD), og executor (Camilla) er endnu klarere.

Ingen negative compliance-effekter identificeret under ISO 27001 eller SOC 2. Tværtimod: dette er det niveau af kontrol-eksplicitering som en SOC 2 Type II-auditor vil forvente.

### 2.5 Styrker klausulen governance-niveauet?

**Ja, entydigt.** Den konverterer et princip ("vurderinger skal foreligge") til en mekanisk kontrol ("Camilla kan ikke agere uden dokumentation"). Det er defensiv programmering for governance: fejl via forglemmelse elimineres.

---

## 3. Betingede anbefalinger til Ændring 1

**Betingelse 1 — Kryds-reference til eskalerings-undtagelse:**

Den foreslåede klausulen bør suppleres med en halvlinje der eksplicit undtager akut-incident-scenariet, så det er tydeligt at invalidity-klausulen ikke blokerer nød-proceduren. Anbefalet tilføjelse (kursiv markerer nyt):

> "... Camilla må ikke handle på den. *Undtagen: akut sikkerheds-incident — se eskalerings-undtagelse nedenfor.*"

Alternativt kan kryds-referencen stå som et parentetisk i klausulen. Formuleringen er et detaljespørgsmål; pointen er at undtagelsen skal fremgå samme sted som invalidity-klausulen.

**Betingelse 2 — Dorthe-aktiverings-note:**

Eftersom Dorthe endnu ikke er aktiveret som permanent rolle, anbefaler jeg at invalidity-klausulen i alle mandater inkluderer den note der allerede findes i `camilla-bibliotekar.md`'s mandat-stabilitets-sektion:

> "Hvis Dorthe endnu ikke er aktiveret som permanent rolle, koordinerer Mads GDPR-vurderingen midlertidigt."

Alternativt: invalidity-klausulens rollout kobles til Dorthes aktivering. Jeg foretrækker at rollout sker nu med noten, frem for at vente — Dorthes mandat er under udformning og aktivering er nær forestående.

Disse to betingelser er **ikke blokkerende** for GO. De er præciseringer der forhindrer fortolkningstvivl i kanten. Ændringen kan committes med betingelserne inkorporeret direkte i teksten.

---

## 4. Vurdering — Ændring 2: Camilla Hard Rule 8 (README-vedligehold)

### 4.1 Hvad ændringen gør

Ny Hard Rule 8 i `camilla-bibliotekar.md`: inden `git push` til origin læser Camilla README.md og opdaterer den hvis team-tabel, mappestruktur eller aktivt projekt er forældet. Resultat rapporteres til Stefan.

Jeg bemærker at Camillas mandat allerede indeholder Hard Rule 8 (README.md-vedligehold) som det ses af den aktuelle `camilla-bibliotekar.md`. Ændringen er altså i praksis allerede implementeret i Camillas mandat. Ændringsforespørgslen bekræfter denne regel formelt og sikrer den rulles ud til CLAUDE.md-registreringen af Hard Rules.

### 4.2 SOD-vurdering

Ingen SOD-problemer identificeret.

Camilla har allerede Write-adgang til dokumentations-filer inkl. README.md. At tilføje en **obligatorisk kontrolhandling** (læs + vurder + opdatér) inden push til origin er en interne procedurale skærpelse, ikke en privileges-udvidelse. README.md behandler ikke persondata og indeholder ikke systemhemmeligheder.

Ansvarskæden er klar: Camilla opdaterer (executor) → rapporterer til Stefan (koordinator). FGD er ikke i løkken for hvert push — det ville skabe overdreven friktion for en lav-risiko rutineoperation.

### 4.3 Security-risici

**Lav risiko.** README.md er et offentligt orienterings-dokument (antager jeg — ingen fortrolige data). Opdatering inden push er en kvalitets- og konsistens-kontrol, ikke en sikkerheds-kritisk operation.

Én potentiel risiko identificeret: hvis README.md indeholder fejlagtige oplysninger om team-sammensætning eller arkitektur, og disse læses af eksterne parter, kan det give et misvisende billede. Hard Rule 8 reducerer denne risiko ved at holde README.md aktualiseret.

**Ingen risici ved at have reglen.** Den tilføjer en lille mængde arbejde til Camillas push-flow, men det er proportionalt med dokumentationens rolle som primær indgang for nye teammedlemmer og revisorer.

### 4.4 ISO 27001 / SOC 2 compliance-impact

**Positivt, om end begrænset:**
- **ISO 27001 A.5.1** (policies for information security): dokumentationen holdes ajour.
- **SOC 2 Availability / Confidentiality**: ikke direkte berørt af README.md-indhold, men god dokumentations-hygiejne understøtter audit-klarberedskab.

Ingen negative compliance-effekter.

### 4.5 Uintenderede konsekvenser

En mulig konsekvens: hvis README.md er kompleks og kræver substantielle opdateringer ved hvert push, kan reglen skabe forsinkelse i push-flowet. Jeg vurderer dette som usandsynligt — README.md skal holdes kort og korrekt (som det fremgår af Hard Rule 8's egne instruktioner), og opdateringer forventes sjældne.

Reglens formulering er fornuftig: den kræver kun opdatering "hvis nødvendigt", ikke at Camilla altid skriver noget. Rapporteringen til Stefan ("README.md: ingen ændringer / opdateret: X") er lav-overhead.

---

## 5. Samlet vurdering

### Ændring 1 — GO (med betingede præciseringer)

Invalidity-klausulen er en **compliance-forbedring uden uhensigtsmæssige sideeffekter**. Den eliminerer et smuthul i den eksisterende governance, styrker SOD, og er direkte aligned med SOC 2 CC8.1 og ISO 27001 A.8.32. De to betingede præciseringer (kryds-reference til eskalerings-undtagelse; Dorthe-aktiverings-note) er anbefalede, ikke blokkerende. Ændringen kan godkendes og committes med disse præciseringer inkorporeret.

### Ændring 2 — GO (ingen betingelser)

Hard Rule 8 til Camillas mandat er en fornuftig og proportional procedurel skærpelse. Ingen SOD-problemer, ingen sikkerhedsrisici, positiv compliance-effekt på dokumentations-hygiejne. Camillas mandat indeholder allerede denne regel — formalisering i change-request-processen bekræfter og dokumenterer den korrekt.

---

## 6. Handlings-anbefalinger til Stefan

1. **Parallel Dorthe-vurdering**: Stefan sikrer at Dorthes GDPR-vurdering (`Team/Dorthe/2026-05-08-vurdering-invalidity-klausul.md`) foreligger inden præsentation for FGD.
2. **Formulating og rollout**: ved FGD-go bedes Camilla inkorporere de to betingede præciseringer fra §3 i klausul-teksten.
3. **Dorthe-aktivering timing**: Stefan koordinerer med FGD om Dorthes aktiverings-tidspunkt i relation til invalidity-klausulens rollout — parallel aktivering er optimal.
4. **Audit-trail**: Trine notificeres om ændringen til brug for SOC 2 CC8.1 change-management-evidence.

---

**Mads (CISO)**
*FGD-teamets Chief Information Security Officer*
*2026-05-08*
