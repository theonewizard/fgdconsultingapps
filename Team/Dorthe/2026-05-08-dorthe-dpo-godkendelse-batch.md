---
dato: 2026-05-08
forfatter: Dorthe (DPO)
emne: DPO-godkendelse — batch-review af tre reviderede mandater
mandater-reviewet:
  - Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md
  - Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md
  - Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md (kodeblok)
grundlag: 4-eyes GDPR-fund fra 2026-05-08 (Team/Dorthe/2026-05-08-dorthe-4eyes-vibeke.md og 2026-05-08-dorthe-4eyes-trine.md)
---

# DPO-godkendelse — Batch-review af tre mandater

**Dorthe, DPO | 2026-05-08**

Jeg bekræfter hermed GDPR-vurdering af tre reviderede mandater. Scope: kun GDPR-aspekter, ikke governance eller SOD.

---

## Mandat 1 — Camilla v2

**Fil:** `Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md`

### Checks udført

**Project-rolle-navne (Henrik/Karen fjernet?):**
Grep på `Henrik`, `Karen`, `Bjarne`, `Anders`, `Sofie` returnerer 0 hits. Bekræftet: ingen project-rolle-navne forekommer i mandatteksten.

**Memory-governance (ingen persondata om rigtige mennesker):**
Mandatets `## Hvad du IKKE skriver i memory`-sektion indeholder ikke et eksplicit forbud mod at gemme persondata om rigtige mennesker (fx kundenavne, mailadresser). Sektionen lister "kode-konventioner, arkitektur, filstier, git-historik, bug-fixes, indhold der allerede findes i CLAUDE.md, midlertidig session-state" — men taler ikke om persondata eller GDPR-minimeringsregel.

### Vurdering

**CONDITIONAL GO**

Camilla v2 løser de strukturelle governance-krav (ingen project-rolle-navne) og opfylder Mads' SOD-betingelser. Fra et GDPR-perspektiv er mandatet acceptabelt til aktivering, men med én udestående betingelse:

**Resterende fund — Betingelse M1:**
`## Hvad du IKKE skriver i memory`-sektionen bør eksplicit inkludere: *"Persondata om rigtige mennesker (fx kundenavne, e-mailadresser, persontilknyttede data) skrives ikke i memory medmindre FGD eksplicit har givet tilladelse, og Dorthe har bekræftet ROPA-dækning."* Dette er ikke en blocker for aktivering, men skal indarbejdes inden Camilla starter memory-opdateringer der vedrører kundedata. Frist: inden første kundedata accepteres (fase 2).

---

## Mandat 2 — Vibeke

**Fil:** `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`

### Checks udført

**PII-dataminimerings-regel (Betingelse A fra 4-eyes-review):**
Bekræftet tilføjet. Linje 82: *"PII-dataminimering: Personnavne, e-mailadresser, telefonnumre eller andre kontaktoplysninger på rigtige mennesker der ses i staged filer må ikke reproduceres, citeres eller gemmes i nogen form ud over selve commit-beslutningen (GO/NO-GO + årsag). [...] Dette sikrer GDPR art. 5(2) (dataminimering)."*

**AFVIST-beslutninger logget jf. art. 5(2) (Betingelse C fra 4-eyes-review):**
Bekræftet tilføjet. Linje 91: *"Loggingskrav ved AFVIST: Når du afviser en commit med compliance-begrundelse (PII-datalækage, secrets, forkert projekt-inddeling, GDPR-dokument-placering eller anden governance-overtrædelse), skal afvisningen logges i `governance/godkendelseslog.md` med: dato, commit-beskrivelse/ref, afvisningsårsag (kategori + specifik grund), og dit navn (Vibeke). Dette kræves af GDPR art. 5(2) (ansvarlighed) og SOC 2 CC8.1 audit-spor."*

### Vurdering

**GO**

Begge GDPR-betingelser fra 4-eyes-reviewet (Betingelse A og C) er korrekt indarbejdet. Formuleringerne er præcise og operationelt anvendelige — Vibeke kan implementere dem.

**Udestående Betingelse B (ROPA for memory-behandling):** Betingelse B fra 4-eyes-reviewet var Dorthes eget ansvar (ROPA opdateres med memory-behandling). Dette er et udestående Dorthe-arbejdsopgave — ikke et mandat-krav til Vibeke. Vibeke-mandatet er ikke afventende på dette. Dorthe noterer: Vibeke bør ikke reviewe memory-commits med kundedata inden ROPA-opdatering er foretaget — dette kommunikeres til Stefan som driftsregel, ikke mandatændring.

---

## Mandat 3 — Trine

**Fil:** `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` (kodeblok)

### Checks udført

**GDPR-grænseflade for evidence-logs (Krav 1 fra 4-eyes-review):**
Bekræftet tilføjet som hard rule. Linje 77: *"GDPR-grænseflade for evidence-indsamling: Trine indsamler audit-logs der kan indeholde persondata. Retsgrundlag: legitim interesse (GDPR art. 6(1)(f)) — compliance-forpligtelse. Retention-begrænsning: audit-evidence slettes/anonymiseres efter SOC 2/ISO 27001 krav (typisk 3–7 år); Dorthe fastsætter præcis retention. Dataminimering: Trine indsamler kun det minimum der er nødvendigt for at dokumentere kontrol-effektiviteten. Koordinering: Trine koordinerer med Dorthe (DPO) når nye evidence-kategorier tilføjes (ROPA-opdatering)."*

Retsgrundlag, retention og dataminimering er alle adresseret. Dorthe-koordinering ved nye evidence-kategorier er korrekt operationaliseret.

**Breach-eskalering til Dorthe (Krav 2 fra 4-eyes-review):**
Bekræftet tilføjet som hard rule. Linje 78: *"Breach-eskalering: Hvis Trine under evidence-indsamling eller audit opdager indikation på databrud eller sikkerhedshændelse, eskalerer hun omgående og parallelt til Mads (CISO) og Dorthe (DPO). GDPR art. 33's 72-timers-frist løber fra kendskab — Trine må ikke vente på Mads' bekræftelse før Dorthe orienteres."*

Formuleringen om parallel eskalering (ikke sekventiel) og den eksplicitte reference til 72-timers-fristen fra art. 33 er præcis og korrekt.

**"Karen" i Anti-patterns (Krav 3 fra 4-eyes-review):**
Bekræftet rettet. Anti-patterns-sektionen (linje 153) skriver nu: *"Trine auditer; security-reviewer adviserer på sikkerhed"* — ingen konkrete project-rolle-navne forekommer.

### Vurdering

**GO**

Alle tre GDPR-krav fra 4-eyes-reviewet er korrekt og præcist indarbejdet. GDPR-grænsefladen for evidence-logs er operationelt stærk — Dorthe noterer særligt at formuleringen om "Dorthe fastsætter præcis retention" er hensigtsmæssig: det giver fleksibilitet til at justere retention-krav efterhånden som SOC 2-perioderne er defineret, uden at mandatet skal genrevideres.

---

## Samlet DPO-status

| Mandat | Verdict | Resterende |
|---|---|---|
| Camilla v2 | **CONDITIONAL GO** | Betingelse M1: tilføj PII-forbud i memory-sektion (inden fase 2 kundedata) |
| Vibeke | **GO** | Ingen mandatkrav. Dorthes eget udestående: ROPA for memory-behandling inden memory-commits med kundedata |
| Trine | **GO** | Ingen |

Vibeke og Trine er GDPR-clearede til aktivering. Camilla v2 er cleareret med betingelse M1 som fase 2-krav.

Stefan koordinerer videre med Mads for samlet anbefaling til FGD.

*Dorthe — DPO, 2026-05-08*
