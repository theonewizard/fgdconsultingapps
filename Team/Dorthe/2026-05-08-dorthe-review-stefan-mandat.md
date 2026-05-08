---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: GDPR-review — Stefan-organisator mandat (udkast til FGD-godkendelse)
reviewet-dokument: Min Inbox/_crosscutting/2026-05-08-stefan-organisator-mandat.md
status: til FGD + Mads
---

# GDPR-review: Stefan-organisator mandat

**Overordnet anbefaling: CONDITIONAL GO**

Mandatet har ingen grundlæggende GDPR-blokkere i sin nuværende form, fordi Stefan er et AI-orkestreringsværktøj der opererer inden for FGD's egne systemer. Der er dog fire forhold der skal adresseres — to kritiske og to minor — før mandatet kan anses som fuldt GDPR-compliant.

---

## Fund

### Fund 1 — Uklar dataadgang uden minimerings-ramme [KRITISK]

**Artikel-reference:** GDPR art. 5(1)(c) (dataminimering), art. 25(1) (privacy by design)

**Mandat-reference:** Linje 27-28: *"Stefan har fuld adgang til både global og project viden"* og linje 28: *"Søge — find relevante filer, patterns, kontekst både internt i filer og på websites"*

**Problem:** "Fuld adgang" er ikke i sig selv ulovligt, men det er udokumenteret. Der er ingen definition af:
- Hvad der udgør "global og project viden" set fra et dataklassifikations-perspektiv
- Om memory-filer eller project-filer kan indeholde persondata om tredjepart (kunder, medarbejdere, datasubjekter)
- Om Stefan's websøgning kan medføre behandling af persondata fra eksterne kilder

Hvis memory- eller project-filer indeholder persondata (fx kundekontakter, kontraktparter, medarbejder-info), har Stefan adgang til disse som en del af sin normale drift. Det giver Stefan rollen som databehandler uden at behandlings-grundlaget er eksplicit.

**Konkret anbefaling:**
1. Tilføj en eksplicit dataklassifikations-reference: "Stefan tilgår kun filer der er nødvendige for opgave-koordination. Filer der indeholder persondata om tredjepart tilgås kun efter opgave-specifik delegation."
2. Mandatet bør indeholde et forbud mod at Stefan paraphraserer eller citerer persondata i sine koordineringsnoter i `Min Inbox/` ud over hvad der er strengt nødvendigt for hånd-off.
3. Websøgning (linje 28: "på websites") bør præciseres: Stefan søger efter teknisk kontekst, ikke efter personoplysninger om individer.

---

### Fund 2 — DPO-routing inkonsistens skaber compliance-risiko [KRITISK]

**Artikel-reference:** GDPR art. 38(3) (DPO-uafhængighed og direkte adgang), art. 38(4) (involvering i alle GDPR-spørgsmål)

**Mandat-reference:**
- Delegerings-hierarki-tabellen (linje 75): Dorthe opført som direkte kanal for Stefan ("GDPR-compliance (DPA, DPIA, data-subject-rights, breach-respons)")
- Ansvarsfordeling-tabellen (linje 110): *"Stefan eskalerer GDPR-spørgsmål til Mads, som koordinerer med Dorthe"*

**Problem:** Linje 110 placerer Mads som gatekeeping-led mellem Stefan og Dorthe. Dette er uforeneligt med GDPR art. 38(3), der kræver at DPO'en er direkte tilgængelig — ikke filtreret gennem en leder eller CISO.

Konkret risiko: Mads og Dorthe har overlappende men ikke identiske ansvarsområder. En GDPR-spørgsmål der går til Mads risikerer at blive besvaret med et sikkerheds-svar frem for et GDPR-svar. Mads er peer — ikke Dorthes overordnede og ikke hendes postkasse.

Governance-scorecard-formuleringen (linje 84-85) er korrekt: den sideordner Mads og Dorthe. Linje 110 modsiger dette.

**Konkret anbefaling:**
Linje 110 skal ændres til: *"Stefan eskalerer GDPR-spørgsmål direkte til Dorthe. Dorthe koordinerer med Mads hvis der er overlap med security-domænet."*

Dette er ikke en kosmetisk ændring — det er en art. 38(3)-korrektionsforpligtelse.

---

### Fund 3 — DPO's rolle i compliance-scorecard risikerer ansvarsglidning [MINOR]

**Artikel-reference:** GDPR art. 39(1)(b) (DPO's rådgivende funktion), art. 5(2) og art. 24 (ansvarlighed — dataansvarlig er FGD)

**Mandat-reference:** Linje 91: *"Ingen feature går til produktion uden FGD's eksplicitte GO baseret på Dorte og Mads' compliance-scorecard"* og linje 84: *"Dorte som AI DPO sagt god for løsningen og ændringen"*

**Problem:** Formuleringen kan tolkes som at DPO'en ko-godkender produktionsbeslutninger. Det er forkert. DPO'en er rådgiver, ikke godkender. Ansvaret for compliance-beslutningen hviler udelukkende på FGD som dataansvarlig (art. 24). Hvis DPO'ens "godkendelse" bliver standard-step i pipeline, risikerer det:
1. At DPO'en i praksis bærer medansvar for beslutninger der juridisk tilhører FGD
2. At DPO'en mister sin uafhængighed fordi hun er bundet af sine egne tidligere "godkendelser"
3. At FGD kan argue for reduceret personligt ansvar under art. 24 ved at pege på DPO-"godkendelse"

**Konkret anbefaling:**
Omformulér til: *"Ingen feature går til produktion uden FGD's eksplicitte GO. Stefan præsenterer FGD for Mads' security-vurdering og Dorthes GDPR-anbefaling. FGD beslutter."*

Ændringen er minor fordi den praktiske effekt er den samme — men formuleringen sikrer at ansvarlighedslinjen (art. 24) er entydig.

---

### Fund 4 — Memory-governance mangler opbevaringsbegrænsning og negativ afskæring [MINOR]

**Artikel-reference:** GDPR art. 5(1)(e) (opbevaringsbegrænsning), art. 25(1) (privacy by design og by default)

**Mandat-reference:**
- Linje 7 (mission §7): *"Opdate hukommelse — bed Camilla notere alle indsigter i memory i rigtigt context"*
- Linje 39: *"beder vibeke om at review memory kun indeholder korrekt og relevant information"*

**Problem:** "Alle indsigter" er en positiv afgrænsning (gem alt) uden en negativ afgrænsning (gem ikke X). Der er ingen definition af:
- Hvad der ikke må gemmes i memory (fx navngivne tredjepart-data, kundekommunikation, personoplysninger om ikke-teammedlemmer)
- Hvem "Vibeke" er i memory-review-konteksten (rollen er ikke defineret i mandatet eller i delegerings-hierarkiet — potentielt inkonsistent reference)
- Hvornår memory-indhold slettes (opbevaringsgrænse)

Privacy by design (art. 25) kræver at systemet som udgangspunkt gemmer mindst muligt, ikke mest muligt. "Alle indsigter" er i udformning i strid med dette princip — selv om den praktiske mængde persondata i memory kan være lille.

**Konkret anbefaling:**
1. Tilføj en negativ liste til memory-governance: "Camilla gemmer ikke personoplysninger om tredjepart i memory. Navne, kontaktoplysninger eller adfærdsdata om kunder, slutbrugere eller andre individer uden for FGD-teamet hører ikke hjemme i memory."
2. Afklar Vibeke-referencen (linje 39) — enten som rolle-definition eller erstat med korrekt rollenavn.
3. Overvej at tilføje en periodisk memory-review med sletningskriterium (fx: project-specifikt memory slettes 90 dage efter projekt-afslutning).

---

## Øvrige observationer (ikke separate fund)

**Stavefejl i GDPR-kritisk rolle-navn:** Mandatet staver konsekvent "Dorte" (linje 17, 38, 53, 75, 84, 91, 97, 110, 141, 145, 172) og nogle steder "Dorthe" (linje 85, 182). Formuleringen "Dorthe" er det korrekte rollenavn. Inkonsistensen er et dokumentations-hygiejne-problem, ikke en GDPR-overtrædelse, men i compliance-dokumenter er præcision forventet.

**Stefan som databehandler:** Stefan anvender `Bash` og `Grep`/`Glob`-tools over filer der potentielt indeholder persondata. Der er ingen DPA (databehandleraftale) dokumenteret for AI-agenternes interne dataadgang inden for FGD's egne systemer. Dette falder sandsynligvis under intern behandling (FGD som both dataansvarlig og behandler), men bør afklares eksplicit i ROPA — det er en opgave jeg (Dorthe) tager op i Fase 0 ROPA-arbejdet, ikke et blokkerende mandatproblem.

---

## Opsummering

| Fund | Type | Blokkerende? | Handling krævet |
|---|---|---|---|
| 1 — Uklar dataadgang, manglende minimerings-ramme | Kritisk | Ja — kræver tilføjelse inden aktivering | Tilføj negativ afgrænsning på dataadgang + websøgning |
| 2 — DPO-routing via Mads (linje 110) | Kritisk | Ja — art. 38(3)-krav | Ret linje 110 til direkte Dorthe-eskalering |
| 3 — DPO som ko-godkender i pipeline | Minor | Nej — men skal rettes ved næste revision | Omformulér til rådgivende frem for godkendende funktion |
| 4 — Memory-governance mangler negativ afgrænsning | Minor | Nej — men tilhører Fase 0 ROPA-arbejde | Tilføj negativ liste + Vibeke-afklaring |

**Betingelserne for GO:** Fund 1 og Fund 2 skal adresseres i mandatteksten inden FGD's godkendelse. Fund 3 og 4 kan løses i Fase 0 som en del af det løbende compliance-arbejde.

---

*Dorthe — Data Protection Officer, FGD-teamet*
*Review afgivet: 2026-05-08*
*Uafhængig vurdering jf. GDPR art. 38(3) — ikke koordineret med Mads inden afgivelse*
