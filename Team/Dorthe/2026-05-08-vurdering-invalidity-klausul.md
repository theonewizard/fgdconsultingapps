---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: GDPR-vurdering — invalidity-klausul + Camilla Hard Rule 8
change-request: governance/change-requests/2026-05-08-alle-mandater-invalidity-klausul.md
status: leveret
modtagere: Stefan, FGD, Mads
---

# GDPR/Persondata-vurdering: Invalidity-klausul + Camilla Hard Rule 8

## Sammenfatning

| Ændring | Konklusion | Betingelse |
|---|---|---|
| **Ændring 1** — Invalidity-klausul (alle 8 mandater + CLAUDE.md) | **CONDITIONAL GO** | Se betingelse 1a og 1b nedenfor |
| **Ændring 2** — Camilla Hard Rule 8 (README-vedligehold) | **GO** | Ingen betingelser |

---

## Ændring 1 — Invalidity-klausul

### Baggrund og formål

Ændringen tilføjer en eksplicit ugyldighedsregel i trin 5 på tværs af alle 8 mandater og CLAUDE.md: FGD's godkendelse af en mandat-ændring er ugyldig, og Camilla må ikke handle på den, medmindre Mads' **og** Dorthes skriftlige vurderinger begge foreligger i `governance/change-requests/`.

### GDPR art. 38(3) — DPO-uafhængighed

**Vurdering: Klausulen styrker DPO-uafhængigheden — med én vigtig forbehold.**

GDPR art. 38(3) kræver at den dataansvarlige sikrer at DPO'en ikke modtager instrukser om udførelse af sine opgaver, og at DPO'en rapporterer direkte til den øverste ledelse (her: FGD). Uafhængighed er ikke blot beskyttelse mod instrukser — det indebærer også at DPO'ens funktion er reel og ikke illusorisk.

**Styrkende aspekter:**

1. Klausulen gør det strukturelt umuligt at omgå Dorthes GDPR-vurdering ved blot at indhente FGD's go uden at Dorthe har været involveret. Dette er en konkret operationalisering af art. 38(3): DPO'en er garanteret at blive hørt, ikke blot nominelt.
2. Klausulen forhindrer at governance-ændringer der potentielt påvirker persondata-behandling (fx mandat-ændringer til roller der håndterer PII, breach-respons, audit-logs) gennemføres uden GDPR-vurdering. Dette er i tråd med EDPB's vejledning om DPO-involvering i risiko-relaterede beslutninger (EDPB Guidelines 07/2020).
3. At Dorthes vurdering er en **materiel forudsætning** — ikke blot høring — giver den faktisk retsvirkning. Det er bedre for art. 38(3) end en procedure der blot noterer Dorthes synspunkt.

**Forbehold — betingelse 1a:**

Klausulen må ikke fortolkes som at Dorthe kan **blokere** mandat-ændringer ud over en rimelig vurderingsperiode, eller at Dorthes vurdering er endeligt bindende over FGD's. GDPR art. 38(3) forbyder at DPO'en fratages uafhængighed — men det er en beskyttelse for DPO'en, ikke en vetorettighed. FGD som dataansvarlig har det juridiske ansvar og den endelige beslutningskompetence.

**Anbefalet formulering** (betingelse 1a): Klausulens trin 5 bør suppleres med en maksimal svarfrist for Dorthes vurdering — jeg foreslår **48 timer** for ordinære ændringer og **8 timer** ved nød-scenarie. Hvis fristen overskrides uden begrundelse, kan Stefan eskalere direkte til FGD med notat om at DPO-vurdering er forsinket. FGD kan herefter godkende på trods, men det dokumenteres eksplicit i change-request-filen som "godkendt uden DPO-vurdering pga. fristoverskridelse". Dette bevarer accountability uden at gøre governance-processen blokerende.

### GDPR art. 33 — Breach-notification (72-timers-krav)

**Vurdering: Potentiel konflikt — skal adresseres eksplicit.**

GDPR art. 33 kræver at den dataansvarlige (FGD) notificerer Datatilsynet inden 72 timer efter at have fået kendskab til et databrud. Dorthe's mandat §3 og hard rule om DPO-uafhængighed specificerer allerede at Dorthe **notificerer selvstændigt inden fristen; FGD orienteres parallelt; FGD-godkendelse er ikke en forudsætning for notifikation.**

**Problemstilling:**

Invalidity-klausulen omhandler mandat-ændringer, ikke breach-respons. De to flows er adskilte og bør forblive det. Der er dog én situation hvor de kan kollidere: et databrud opdages under eller umiddelbart efter et scenarie hvor et mandat er under ændring (fx Dorthes egne tools eller model er under revision). I dette tilfælde er spørgsmålet: er invalidity-klausulen til hinder for at Dorthe handler?

**Analyse:**

Klausulen låser Camilla fra at **committe mandat-ændringer** uden begge vurderinger. Den låser ikke Dorthe fra at udøve sine eksisterende GDPR-funktioner. Dorthes breach-respons-kapacitet er knyttet til hendes nuværende mandat — ikke til eventuelle mandat-ændringer i gang. En ændring af Dorthes mandat der er under review ændrer ikke Dorthes eksisterende pligter.

**Men:** Change-request-dokumentet specificerer at invalidity-klausulen indsættes i **alle 8 mandater + CLAUDE.md**. Hvis klausulen fejlagtigt fortolkes som at enhver FGD-beslutning kræver Dorthes forudgående godkendelse, kan det sætte FGD i en situation under breach-respons hvor governance-procedurens frister interfererer med 72-timers-fristen.

**Anbefalet formulering** (betingelse 1b): Invalidity-klausulen skal indeholde en eksplicit undtagelsesklausul for nød-scenarie-ændringer under aktiv breach-respons eller security-incident. Foreslået tilføjelse til trin 5 i alle mandater:

> "**Undtagelse — nødsituation**: Ved aktiv databreach (GDPR art. 33-scenarie) eller akut security-incident, kan Mads og FGD midlertidigt justere et mandat uden forudgående Dorthe-vurdering. Dorthe notificeres parallelt. Permanent ændring kræver fuld proces inden 72 timer efter incident-lukning."

Denne undtagelse svarer til den der allerede er i Camilla-mandatets eskalerings-undtagelse og er dermed konsistent med eksisterende governance-ramme.

### DPA/ROPA-forpligtelse

**Vurdering: Ingen ny ROPA-forpligtelse introduceres.**

Invalidity-klausulen ændrer ikke hvilke persondata der behandles, af hvem, med hvilke formål, eller under hvilke retsgrundlag. Klausulen er en procedureregelom hvem der skal godkende konfigurationsændringer. Den introducerer ikke ny databehandling, ny kategori af PII, ny leverandør, eller ny overførselsmekanisme.

Eksisterende ROPA-posten for governance-aktiviteter (behandlingsaktivitet: "AI-agent-konfiguration og change-management") behøver ikke opdateres som følge af denne ændring alene. Tilsvarende udløses DPIA-krav ikke, da ændringen ikke udgør "high risk processing" i art. 35's forstand.

### GDPR-risici ved Mads+Dorthe-vurdering som obligatorisk forudsætning

**Vurdering: Lav risiko, men ét punkt kræver opmærksomhed.**

At begge vurderinger er obligatorisk forudsætning er GDPR-neutralt i sig selv — klausulen tilføjer procedurekrav, ikke nye databehandlinger. Den eneste GDPR-relevant risiko er den beskrevet under art. 33 ovenfor (nødsituation-kollision), som håndteres af betingelse 1b.

Et supplerende punkt: klausulen specificerer at vurderinger foreligger i `governance/change-requests/`. Disse filer kan i sjældne tilfælde indeholde personhenvisninger (fx "Poul vurderede at Sofie's mandat..."). Da AI-roller ikke er fysiske personer under GDPR, er dette ikke en persondata-problematik. Hvis `governance/change-requests/`-filer imidlertid begynder at indeholde oplysninger om reelle persons adfærd (FGD, konsulenter, kunder), bør ROPA opdateres. Dette er et åbent monitorerings-punkt — ikke en bloker for GO.

### Konklusion Ændring 1

**CONDITIONAL GO** — under følgende betingelser:

1. **Betingelse 1a**: Indsæt maksimal svarfrist for Dorthes vurdering (anbefalet 48 timer ordinær, 8 timer nød) med eskalerings-procedure ved fristoverskridelse. Dorthe's vurdering er obligatorisk men ikke veto-ubegrænset.

2. **Betingelse 1b**: Indsæt eksplicit nødsituations-undtagelse i trin 5 (parallel med Camilla-mandatets eksisterende undtagelse) for aktivt breach/incident-scenarie.

Disse betingelser er tekniske justeringer af formuleringen, ikke principielle invendinger mod klausulens formål. Klausulens intention — at sikre reel DPO-involvering og Mads' security-vurdering inden mandat-ændringer — er GDPR-kompatibel og styrker faktisk DPO-uafhængighedens substantielle indhold.

---

## Ændring 2 — Camilla Hard Rule 8 (README-vedligehold)

### Persondata i README.md

**Vurdering: README.md behandler ikke persondata i GDPR's forstand.**

Jeg har gennemlæst README.md (aktuel tilstand, 2026-05-08). Indholdet er:

- Team-tabel med AI-rollenavne (Stefan, Laila, Poul, Camilla, Mads, Dorthe, Trine, Vibeke) og funktionsbeskrivelser
- Mappestruktur-oversigt
- Workflow-beskrivelse
- Governance-opsummering
- Aktivt projekt-reference

AI-rollenavne er ikke persondata. GDPR art. 4(1) definerer persondata som oplysninger der kan henføres til en **identificeret eller identificerbar fysisk person**. Fiktive AI-roller er ikke fysiske personer, og deres navne er ikke persondata uanset at de er menneskelignende navne.

README.md indeholder ikke:
- Brugeroplysninger (e-mailadresser, navne på kunder eller konsulenter)
- FGD's personoplysninger udover den overordnede rolle som dataansvarlig (som er offentligt tilgængelig information)
- Nogen form for PII om tredjeparter

**Konklusion**: Hard Rule 8 introducerer ingen persondata-behandling og udløser ingen DPIA-forpligtelse, ingen ROPA-opdatering og ingen DPA-overvejelse.

### Operational GDPR-vurdering

Hard Rule 8 er en vedligeholds-regel der sikrer at et offentligt eller internt repository-dokument afspejler faktisk tilstand. Set fra et GDPR-perspektiv:

- **Privacy by design (art. 25)**: Ingen relevans — README indeholder ingen persondata.
- **Data minimization (art. 5(1)(c))**: Ikke relevant — ingen persondata er involveret.
- **Accountability (art. 5(2))**: Neutral — god governance-hygiejne som ikke introducerer compliance-risici.

### Konklusion Ændring 2

**GO** — ingen betingelser.

Hard Rule 8 er GDPR-neutral. Den introducerer ingen persondata-behandling, ingen ny dataflow, og ingen compliance-risici. Den er positivt organisatorisk (god dokumentations-hygiejne) og negativ GDPR-risiko er nul.

---

## Dorthe-perspektiv på den samlede pakke

Begge ændringer er foreneligt med Dorthes DPO-funktion og GDPR-compliance. Den vigtigste observation fra mit perspektiv er at invalidity-klausulen giver DPO-funktionen reel operationel styrke, hvilket er i tråd med EDPB's vejledning om at DPO-involvering ikke må være pro forma. De to betingelser jeg stiller er defensive justeringer der forhindrer at en ellers god governance-mekanisme kan overbelastes i nødsituationer.

Jeg anbefaler at Stefan og Mads koordinerer de to betingelser inden implementering, og at Camilla-ændringsprocessen afventer den samlede change-request-godkendelse.

---

**Dorthe (DPO)**
*FGD-teamets Data Protection Officer*
*2026-05-08*

Dokumentet er Dorthes uafhængige GDPR-vurdering og afgivet uden instrukser fra Mads (CISO) eller Stefan, i overensstemmelse med GDPR art. 38(3).
