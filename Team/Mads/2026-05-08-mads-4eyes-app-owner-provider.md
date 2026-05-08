---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: 4-eyes review — Application Owner og Application Provider mandat-udkast (Laila)
kilde-udkast:
  - Team/Laila/2026-05-08-udkast-application-owner-mandat.md
  - Team/Laila/2026-05-08-udkast-application-provider-mandat.md
reference-vurdering: Team/Mads/2026-05-08-mads-vurdering-app-owner-provider.md
reference-dpo: Team/Dorthe/2026-05-08-dorthe-vurdering-app-owner-provider.md
status: CONDITIONAL GO
---

# 4-eyes Review: Application Owner og Application Provider mandat-udkast

**Reviewer:** Mads (CISO)
**Dato:** 2026-05-08
**Objekt:** Lailas mandat-udkast til Application Owner og Application Provider — begge filer reviewes samlet da SOD-separation er et fælles reviewpunkt.

---

## Samlet verdict: CONDITIONAL GO

Begge mandater er substantielt stærkere end hvad der typisk produceres i en første udkastsrunde. Laila har indarbejdet størstedelen af mine krav fra CONDITIONAL GO-anbefalingen (afsnit 4) og Dorthes GDPR-krav (afsnit 4.1-4.3). SOD-separationen er eksplicit og gennemgående. De primære security-kontroller er på plads.

Tre specifikke huller kræver rettelse inden aktivering. Ingen af hullerne underminerer rolledesignets grundlæggende logik — de er præciserings-mangler, ikke design-fejl.

---

## Reviewspørgsmål 1: Er mine tre betingede krav korrekt indarbejdet?

Mine tre CONDITIONAL GO-krav fra afsnit 4.1 var:

### Krav A — Compliance-koordinering (ikke -vurdering)

**Krav (original formulering):**
> "Application Owner er *accountable* for at compliance-krav indhentes — ikke for at *vurdere* compliance."

**Status i udkastet: INDARBEJDET**

AO-mandatets Mission §4 siger "Du sikrer at Mads og Dorthe bliver involveret tidligt — ikke fordi du vurderer compliance, men fordi det er dit job at være trigger-punkt." Hard rules §Compliance-grænser siger eksplicit "Compliance-vurdering er IKKE dit ansvar. Du er ansvarlig for at få det gjort." Distinktionen er korrekt gjort og gentages konsekvent i ansvarsfordeling-tabellen og hand-off-aftalerne.

Ingen rettelse nødvendig.

### Krav B — Self-certify-forbud

**Krav (original formulering):**
> "Application Owner må ikke erklære en feature, integration eller ændring som compliant."

**Status i udkastet: INDARBEJDET — med et semantisk hul**

Hard rules §Compliance-grænser dækker dette. GDPR-sektionen §Compliance-claims-til-kunder specificerer forbuddet mod at erklære produktet "GDPR-compliant", "ISO 27001-certificeret", "SOC 2-aligned" i kundekommunikation.

**Hul (moderat):** Self-certify-forbuddet dækker kundekommunikation men formulerer ikke forbuddet i den interne beslutningsproces. En Application Owner kan læse teksten og konkludere at interne compliance-erklæringer (fx "denne feature er compliant, vi behøver ikke sende til Mads") er tilladte, blot fordi det ikke sker i "kundekommunikation, marketing eller kontrakter".

**Krævet rettelse (AO-mandat, Hard rules §Compliance-grænser):**

Tilføj som afsluttende sætning under dette felt:
> "Dette forbud gælder internt som eksternt. Du må ikke internt erklære en feature, integration eller ændring som compliant uden skriftlig anbefaling fra Mads (ISO 27001/SOC 2) og/eller Dorthe (GDPR) — også ved features der virker åbenlyst acceptable."

### Krav C — Protected escalation

**Krav (original formulering):**
> "Team-members tilknyttet applikationen kan eskalere compliance-fund direkte til Mads uden at gå via Application Owner — dette er en beskyttet kanal. Owner må ikke sanktionere eller deprioritere team-members der bruger denne kanal."

**Status i udkastet: INDARBEJDET**

Hard rules §Compliance-grænser afsluttende bullet: "Team-members kan eskalere compliance-fund direkte til Mads uden at gå gennem dig. Du må ikke sanktionere eller deprioritere dem for at bruge denne kanal." Teksten er ordret korrekt og er placeret i en section der gør den operationelt tydelig.

Ingen rettelse nødvendig.

---

## Reviewspørgsmål 2: Er Application Provider's security-krav tilstrækkeligt stramme?

Mine fire security-krav fra afsnit 4.2 var: least privilege, Camilla som executor, security-reviewer-gate, DPA-trigger.

### Least Privilege

**Status: INDARBEJDET — med et konkret hul**

Hard rules §Least Privilege & Access Control er solid: ingen blanket-adgang, adgang pr. integration dokumenteret i `adgangsprofil.md`, prod-PII-adgang via break-glass med CISO-godkendelse og 4-timers maks-session. Clerk, Neon, Vercel, HSM/bridge, Anthropic og Tailscale er navngivet specifikt som de skal (jf. min §3.1).

**Hul (moderat):** Break-glass-proceduren er beskrevet med de rette elementer ("skriftlig begrundelse, CISO-godkendelse, automatisk audit-trail, maksimal session 4 timer") men mangler en eksplicit specificering af immutabilitet for den automatiske audit-trail. Dette er det samme immutabilitets-krav jeg identificerede i Trine-review (fund 3). Hvis Provider kan redigere sin egne session-logs — eller hvis Camilla kan, fordi hun ejer git — taber break-glass-procedurens forensisk sporbarhed.

**Krævet rettelse (AP-mandat, Hard rules §Least Privilege & Access Control):**

Tilføj som punkt til break-glass-procedurens beskrivelse:
> "Audit-trail for break-glass-sessioner gemmes i uforanderlig form — write-once storage eller eksporteret til separat immutabelt arkiv. Hverken Application Provider, Camilla, eller andre roller der producerer eller eksekverer kan modificere dette log. Trine auditerer break-glass-log halvårligt som SOC 2-evidens."

### Camilla som executor

**Status: INDARBEJDET**

Hard rules §Least Privilege & Access Control, tredje bullet: "Du designer (specificerer hvad der skal ændres); Camilla implementerer (udfører ændringen i git/infrastructure)." Anti-patterns-listen bekræfter: "Committer infrastruktur-ændringer selv — Camilla ejer executionen". Change Management-sektionen gentager princippet. Ansvarsfordelings-tabellen viser korrekt "Provider designer ændring; Camilla implementerer".

Ingen rettelse nødvendig.

### Security-reviewer-gate

**Status: INDARBEJDET**

Hard rules §Code Review Gate er korrekt: alle PRs reviewes af security-reviewer inden merge til test eller prod. Scope for reviewer specificeret (API-integrationer, auth-flow, RLS, secrets, dataflows). Arkitektur-kritiske PRs kræver eksplicit Mads-godkendelse. Provider er ansvarlig for compliance-flagning med label.

Ingen rettelse nødvendig.

### DPA-trigger

**Status: INDARBEJDET**

Hard rules §Leverandør-integration & DPA og §GDPR-relateret (fra Dorthe) specificerer korrekt: Provider kan ikke godkende ny leverandørtilkobling, DPA-vurdering via Dorthe er obligatorisk, notifikations-pligt inden 5 hverdage ved dataflow-ændring. De otte konkrete notifikations-triggere fra Dorthes vurdering (§2.2) er inkluderet i GDPR-sektionen.

Ingen rettelse nødvendig.

### Yderligere security-observation (AP-mandat)

**Observation (ikke blokkerende, men kræver afklaring):**

AP-mandatets ansvarsfordelings-tabel refererer "Vibeke (doc-reviewer)" som commit-gate via security-reviewer. Vibeke er ikke en aktiveret rolle i det nuværende team. Hvis referensen er til en fremtidig rolle er det acceptabelt, men det bør markeres eksplicit med "(project-rolle, ikke aktiveret)" for at undgå forvirring ved aktivering. Alternativt erstattes Vibeke med den korrekte gate-rolle (security-reviewer direkte, eller Camilla som git-ejer).

**Krævet rettelse (AP-mandat, ansvarsfordelings-tabel og hand-off-aftaler):**

I rækken for "Vibeke (doc-reviewer)" tilføjes parentetisk "(project-rolle — aktiveres ved behov; security-reviewer fungerer som gate indtil aktivering)". Samme præcisering i hand-off-aftalerne.

---

## Reviewspørgsmål 3: Er SOD-separationen (AO ≠ AP) eksplicit og operationelt håndterbar?

### Eksplicithed

SOD-klausulen er indarbejdet i begge mandater som første punkt under Hard rules §Separation of Duties. Formuleringerne er gennemgående og konsistente: begge mandater siger "du er ALDRIG samme person" og eskalerings-instruks til Laila ved rolledoublement er identisk i begge.

Ansvarsfordelings-tabellerne i begge mandater gentager "Vigtig: du og [modpart] er ALTID to forskellige personer (SOD-krav)" — dette er god praksis; det gør SOD-kravet synligt i den operationelle arbejdsgang, ikke kun i hard rules.

### Operationel håndterbarhed

SOD er operationelt håndterbar med en enkelt præcisering, som jeg finder manglende:

**Hul (kritisk):** Hverken AO- eller AP-mandatet specificerer hvad der sker i den periode inden begge roller er besat. Knowlagecentral er i Fase 0-1 nu. FGD er i praksis potentielt den eneste operationelle aktør der kan udfylde begge roller. Dorthes §5, spørgsmål 1 peger på præcis denne problemstilling.

Mandaterne adresserer ikke dette overgangsscenarie. Hvis mandaterne aktiveres uden at begge roller er besat af separate personer, er SOD-kravet formelt overtrådt fra dag 1 — selv om mandaterne forbyder det.

**Krævet rettelse (begge mandater, afsnit om aktivering/livscyklus — Laila skal tilføje dette afsnit):**

Tilføj til begge mandater under Hard rules §Separation of Duties:
> "Overgangsscenarie: Hvis kun én af de to roller er besat ved aktivering (fx AO uden AP eller omvendt), opererer den besatte rolle med begrænset mandat. AO uden AP: AO kan drive roadmap og compliance-koordinering, men må ikke selv specificere teknisk design. AP uden AO: AP kan drive teknisk implementering under Mads' direkte compliance-oversight, men har ikke selvstændig roadmap-autoritet. FGD notificeres om overgangsperioden via Stefan; Laila besætter den manglende rolle inden for den aftalte tidsramme."

---

## Reviewspørgsmål 4: Er der governance-huller?

### Hul A — Mandat-stabilitet mangler i AO-mandatet (kritisk)

AP-mandatets afsluttende sætning under Hard rules lyder korrekt: "Reglerne kan ikke ændres uden formel change-management-proces (jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md § 3.3)."

AO-mandatet har samme sætning. Begge mandater har den. Dette er korrekt.

**Observation:** Hverken AO- eller AP-mandatet indeholder et eksplicit forbud mod self-modifikation (parallelt med Mads' og Trines mandater). For projekt-roller er dette ikke lige så kritisk som for permanente governance-roller, men da AO og AP er vektorer for Camilla-godkendelse af commits, bør forbuddet være eksplicit.

**Anbefalet tilføjelse (begge mandater, ny bullet under mandat-stabilitet-sætningen):**
> "Du må ikke selv foreslå eller bede andre om ad-hoc-rettelser til dit eget mandat. Alle mandat-ændringer følger den formelle change-management-proces."

Dette er en anbefaling, ikke en blokerende krav.

### Hul B — Breach-respons-flow mangler i AP-mandatet (moderat)

AO-mandatets Hard rules §GDPR-specifikke inkluderer: "Du eskalerer alle art. 15-22-anmodninger (ret til indsigt, sletning, osv.), breach-mistanker, og Datatilsynets henvendelser til Dorthe inden 24 timer."

AP-mandatets Hard rules §GDPR-relateret specificerer audit-trail-pligt og notifikations-pligt til Dorthe, men nævner ikke breach-mistanker specifikt. Application Provider er teknisk tættest på infrastrukturen og vil statistisk set oftere opdage et brud end Application Owner. Hvis Provider kun ved at rapportere til Owner (som derefter eskalerer), risikerer vi forsinkelse mod GDPR art. 33's 72-timers notifikationsgrænse.

**Krævet rettelse (AP-mandat, Hard rules §Security-specifikt):**

Tilføj afsluttende bullet:
> "Breach-mistanker rapporteres direkte til Dorthe (DPO) og Mads (CISO) inden 24 timer — ikke via Application Owner som mellemled. Tid er kritisk under GDPR art. 33 (72-timers notifikationsvindue). Du behøver ikke afvente Owner-bekræftelse før eskalering."

### Hul C — Audit-trail-krav: Immutabilitet (moderat — gentagelse fra SP2)

Som nævnt under Security-krav SP2: break-glass-procedurens audit-trail mangler immutabilitets-krav. Dette er identisk med fund 3 fra Trine-review. Konsistens på dette punkt er vigtig; et SOC 2 Type II-review vil forvente at immutabilitetskravet er uniformt på tværs af alle roller.

---

## Samlet oversigt over påkrævede rettelser inden aktivering

| # | Mandat | Sektion | Kritikalitet | Rettelse |
|---|---|---|---|---|
| 1 | AO | Hard rules §Compliance-grænser | Moderat | Self-certify-forbud udvides til interne beslutninger (ikke kun kundekommunikation) |
| 2 | AP | Hard rules §Least Privilege | Moderat | Immutabilitets-krav til break-glass audit-trail |
| 3 | AP | Ansvarsfordeling + Hand-off | Lav-moderat | Vibeke-reference præciseres med "(project-rolle — ikke aktiveret; security-reviewer fungerer som gate)" |
| 4 | Begge | Hard rules §SOD | Kritisk | Overgangsscenarie-klausul for periode med kun én besat rolle |
| 5 | AP | Hard rules §Security-specifikt | Moderat | Breach-mistanker: direkte eskalering til Dorthe + Mads inden 24 timer (ikke via AO) |

Fund markeret som anbefalinger (ikke blokkerende):

| # | Mandat | Felt | Anbefaling |
|---|---|---|---|
| A | Begge | Hard rules (afslutning) | Eksplicit self-modifikations-forbud tilføjes |

---

## Positive fund

Disse punkter er korrekt implementeret og fortjener eksplicit bekræftelse:

- **Compliance-koordinering vs. -vurdering**: Distinktionen er skarp og konsekvent i hele AO-mandatet. Dette var det sværeste punkt at formulere præcist — Laila har ramt det.
- **DPO-uafhængighed**: AO-mandatets formulering af at Dorthes vurdering er bindende uanset roadmap-præferencer er korrekt. Rapporteringslinjerne fra AO til Mads/Dorthe vs. til Stefan/FGD er korrekt adskilt.
- **Protected escalation-kanalen**: Formulering og placering er operationelt klar.
- **DPA-trigger i AP**: De otte konkrete notifikationsscenarier fra Dorthes §2.2 er inkorporeret. Dette er specificitet på det rette niveau.
- **HSM-isolation**: AP-mandatets "du må aldrig tilgå YubiHSM direkte" og key-ceremony-krav er korrekt og konsistent med Mads' §3.1.
- **Camilla-executor-model**: Korrekt i alle relevante sektioner og anti-patterns.
- **SOD-eksplicithed**: Formuleringen er tydelig, gentaget på rette steder, og eskaleringsvejen (til Laila) er korrekt.

---

## Næste skridt

1. **Laila** indarbejder de fem rettelser fra tabellen ovenfor.
2. **Mads re-check** af de rettede sektioner (kan gøres hurtigt — fem afgrænsede punkter).
3. **Dorthe 4-eyes** på de reviderede AO/AP-mandater bekræfter at GDPR-sektionerne er intakte efter rettelserne.
4. **Stefan** samler Mads + Dorthes konfirmation og præsenterer for FGD med GO/NO-GO-anbefaling.
5. **FGD** giver eksplicit GO.
6. **Camilla** committer begge mandatfiler til `.claude/agents/`.
7. **Trine** noterer rolle-aktivering i audit-trail (når Trine er aktiveret).

---

*Mads (CISO) — 2026-05-08*
