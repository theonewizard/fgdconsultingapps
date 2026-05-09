---
dato: 2026-05-09
type: 4-eyes-vurdering
emne: FGDCorePKI AO+AP+PKI-roller mandatudkast
vurderer: Mads (CISO)
status: CONDITIONAL GO
input-dokumenter:
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md
  - Team/Mads/2026-05-09-pki-org-krav.md
  - Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md
  - Team/Poul/2026-05-09-fgdcorepki-org-struktur.md
  - governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md
---

# Mads — 4-eyes vurdering: FGDCorePKI mandater

## Samlet vurdering

**CONDITIONAL GO**

Laila har leveret solide udkast der afspejler Mads' PKI-krav (2026-05-09) og de eksisterende governance-rammer korrekt. Grundstrukturen i alle tre dokumenter er forsvarlig: SOD-adskillelsesprincippet er gennemgående, compliance-grænser er eksplicitte, og mandat-stabilitetsklausulen er medtaget.

Der er imidlertid tre blockers og fem betingede fund der skal rettes inden aktivering. Mandaterne må ikke aktiveres — og FGD bør ikke give GO — før disse rettelser er indarbejdet af Laila og dokumenterne er revurderet.

Dorthe skal parallelt vurdere GDPR-implikationerne af CSR-databehandling, Subject-felter i audit-logs og eventuel PII i certifikatmetadata. Denne vurdering er en forudsætning for FGD-GO og er ikke en del af nærværende dokument.

---

## AO-udkast

### Generel vurdering

AO-udkastet er det stærkeste af de tre. SOD-forbuddet mod AO × AP er klart og utvetydigt formuleret i frontmatter, mission, hard rules og anti-patterns. Forbuddet mod direkte CA-drift, HSM-adgang og key ceremony-deltagelse er korrekt. Mandat-stabilitetsklausulen er medtaget og refererer korrekt til governance-policyen.

### Fund

**AO-F1: FIPS-terminologi (acceptabel)**
AO-draft formulerer "FIPS-re-keying-plan-ejerskab" og nævner "når YubiHSM2 FIPS-variant er tilgængelig". Dette er korrekt og afspejler FGDs 2026-05-09-beslutning (Non-FIPS MVP nu, re-keying ved FIPS 140-3-cert). Ingen rettelse kræves.

**AO-F2: Intern stavefejl i anti-patterns (redaktionel)**
Anti-pattern "Har uhovedede roadmap-power" er en stavefejl — sandsynligvis "uhindret" eller "ubegrænset". Rettes ved revision, men er ikke en governance-blocker.

**AO-F3: AO-deltagelse i key ceremony som observer (præcisering krævet)**
Mandatteksten tillader AO at "deltage som observer ved ceremoni hvis relevant" (jf. hard rules, afsnit om key ceremony). Dette er i strid med Pouls org-struktur §5.1, der eksplicit lister "deltage i key ceremony (heller ikke som witness)" som forbudt for AO. Observer-status skaber compliance-uklarhed: en AO der er til stede ved key ceremony, kan påvirke cermoniens formelle uafhængighed og skaber en linje i auditorers øjne der er unødvendig. Rettelse: fjern "kan deltage som observer"-formulering; tilføj eksplicit forbud analog til Pouls §5.1.

**AO-F4: Compliance-selvstændighed — eskalering forbi AO (styrkes)**
Ansvarsfordelings-tabellen angiver at Mads leverer compliance-scorecard til AO som videreformidler til Stefan. Det er korrekt i normaltilfældet. Men det bør ekspliciteres — analogt til Pouls org-struktur §2.2 — at Mads, Trine og Dorthe alle har ret til at eskalere direkte til FGD uden om AO, og at AO ikke kan filtrere eller forsinke compliance-rapportering. Dette er et SOC 2 CC4.1-krav (audit-uafhængighed). Rettelse: tilføj hard rule om "AO kan ikke filtrere compliance-eskalering — Mads, Trine og Dorthe eskalerer direkte til FGD ved fund".

### Samlet for AO

Tre fund: ét præciseringskrav (F3) og ét styrkelseskrav (F4) er obligatoriske; F2 er redaktionel. Ingen stopblocker, men F3 og F4 skal rettes inden aktivering.

---

## AP-udkast

### Generel vurdering

AP-udkastet er teknisk velunderbygget og viser god forståelse for PKI-arkitekturrollen. Break-glass-proceduren er detaljeret og korrekt. Mandat-stabilitetsklausulen er medtaget. GDPR-notifikationsforpligtelserne er stærke og operationelt præcise.

Der er imidlertid to blockers der begge handler om rolledefinitionskonsistens.

### Fund

**AP-B1: AP = PKI Tech Lead-konflikt (BLOCKER)**

AP-frontmatter description formulerer AP som "fungerer som PKI Tech Lead for engineering-teamet". AP-body formulerer "Du er Tech Lead; Engineering Lead rapporterer tekniske blokerere til dig."

PKI-governance-roller-udkastet §5 definerer Engineering Team Lead / PKI Tech Lead som en selvstændig rolle der rapporterer til Application Provider (eller Mads). AP er altså AP's direkte underordnede i PKI-governance-udkastet, men AP og Tech Lead er identiske i AP-mandatudkastet.

Pouls org-struktur §4 SOD-matrix markerer AP × Tech Lead som OK‡ ("best practice separat; kombination kræver undtagelse"). Kombinationen er ikke forbudt, men kræver eksplicit formel undtagelse med normaliserings-deadline og Trine-attestation, analogt til de fire andre tidsbegrænsede undtagelser i Pouls §3.2.

Valget er ét af to:

- **Option A**: AP og Tech Lead er samme person i fase 0. Kræver eksplicit dokumenteret SOD-undtagelse i `governance/sod-undtagelser/`; normaliserings-deadline inden første eksterne kunde; Trine attesterer kvartalsvis. AP-mandatteksten skal opdateres til at afspejle dette som en tidsbegrænset kombination, ikke som APens permanente rolle.
- **Option B**: AP og Tech Lead er separate roller fra dag 1. PKI-governance-roller-udkastet er da korrekt, og AP-mandatteksten skal rettes til at beskrive Engineering Lead som en separat underordnet rolle.

FGD skal tage stilling til dette valg. Laila retter i overensstemmelse med FGDs beslutning.

**AP-B2: FIPS-status-uoverensstemmelse (BLOCKER)**

AP-mandatets mission §4 formulerer: "Du designer YubiHSM2-konfiguration til FIPS 140-3 Level 3, algoritme-valg (ECDSA P-384 eller RSA 4096), wrap-key-management, og planegger teknisk migration fra FIPS 140-2 (når relevant)."

Dette er i uoverensstemmelse med:

1. FGDs 2026-05-09-beslutning: Non-FIPS MVP nu (non-FIPS YubiHSM2 €728), med re-keying-plan ved FIPS 140-3-cert — ikke fra FIPS 140-2-drift.
2. Pouls vinkel 5 (blokerende fund): YubiHSM2 FIPS 140-2 cert (#3916) udløb 2026-05-02; FIPS 140-3 cert er endnu ikke granted af NIST CMVP. Der er i dag ingen gyldig FIPS-validering på YubiHSM2.
3. Mads' compliance-note øverst i krav-dokumentet: "At bygge en ny PKI-infrastruktur på en standard der udløber i 4 måneder er ikke forsvarligt."

AP-mandatets nuværende formulering skaber et compliance-problem: det beskriver FIPS 140-3 Level 3 som nuværende driftsmål, men FGD har besluttet Non-FIPS MVP med planlagt fremtidig migration. AP-mandatet skal reflektere den faktiske situation: fase 0 er non-FIPS (dokumenteret risiko-accept per Mads' krav-dok); AP designer til FIPS 140-3-readiness, men aktuel drift er non-FIPS afventende CMVP-certificering.

Rettelse: Erstat "til FIPS 140-3 Level 3" med "FIPS 140-3 Level 3-ready (fase 0: non-FIPS MVP per FGDs 2026-05-09-beslutning; re-keying ved FIPS 140-3-cert)". Tilsvarende opdatering i hard rules afsnit om FIPS-compliance-design.

**AP-F3: Audit-trail forankring i SOC 2 (betinget)**

AP-mandatets audit-trail-forpligtelse (hard rules, GDPR-relateret) er solidt forankret i GDPR art. 30/32. SOC 2 Type II kræver imidlertid yderligere: specifikt at audit-logs er WORM-immutable eller kryptografisk chained, og at de kan tilgås af ekstern revisor. Disse krav er nævnt i Mads' krav-dok §4.5, men mangler i AP-mandatets formulering af audit-trail-forpligtelsen.

Rettelse: Tilføj reference til WORM-storage/kryptografisk chained log-krav og ekstern revisor-adgang (SOC 2 CC7.2). Dette er ikke en blocker, men er nødvendigt for at AP's audit-trail-forpligtelse er tilstrækkelig som SOC 2 Type II-evidens.

**AP-F4: Break-glass dækker ikke PII i CSR/audit-events (betinget)**

AP-mandatets break-glass-procedure er korrekt for HSM-troubleshooting. Den dækker ikke eksplicit scenarier hvor AP tilgår produktions-audit-events eller CSR-queue-data der indeholder personoplysninger (Subject-felter med navne, email-adresser, organisationsidentifikation). Break-glass-proceduren bør gælde for enhver direkte prod-adgang til systemer der behandler PII.

Rettelse: Udvid break-glass-afsnittet eller tilføj en separat formulering om at PII-bærende systemer (CSR-database, audit-log-læseadgang) behandles analogt med HSM — skriftlig begrundelse, Mads-godkendelse, automatilogning, Trine-audit. (Dette er Dorthes domæne at endeligt formulere.)

**AP-F5: Typos og terminologi-uoverensstemmelse (redaktionel)**

Følgende redaktionelle fejl bør rettes:
- "Designes" → "Designer" (flere steder i hard rules og anti-patterns)
- "Met Dorthe" og "Met Camilla" → "Med Dorthe" og "Med Camilla" (hand-off-aftaler)
- "rappotere" → "rapportere"
- "planegger" → "planlægger"
- "TVivl" → "tvivl"
- "audit.events" — dette er Knowlagecentral-specifikk tabelnavngivning; for FGDCorePKI bør terminologien være neutral ("PKI audit-log" eller "FGDCorePKI audit-events") for at undgå kryds-applikations-forvirring.
- "Lagd af" → "Lagt af"

### Samlet for AP

To blockers (B1, B2) kræver rettelse og i B1's tilfælde en FGD-beslutning. Tre betingede fund (F3, F4, F5) rettes parallelt. Mandatet er ikke aktiveringsklart i nuværende form.

---

## PKI-governance-roller

### Generel vurdering

PKI-governance-roller-udkastet er det mest komplekse dokument og fungerer godt som profil-oversigt. SOD-begrænsningerne for hver rolle er korrekt specificerede og alignede med Mads' SOD-matrix (krav-dok §5). Rolledokumentet er ikke et fuldt mandat men en profil-spec til brug for Lailas efterfølgende mandat-design.

### Fund

**PKI-G1: Security Officer-rollen mangler (BLOCKER)**

Mads' krav-dok §1.7 definerer Security Officer som en af de syv core-funktioner i PKI-governance-strukturen. Pouls org-struktur §1 lister Security Officer som separat project-rolle. PKI-governance-roller-udkastet nævner Security Officer i indledningen som "kan varetages af Mads eller dedikeret rolle (i opstartsfase = Mads)" — men designer ikke rollen. Dokumentet designer kun 5 roller ud af 7 (CA Administrator, HSM-operatør, Key Ceremony Officer, RA, Engineering Team Lead) plus Trine-udvidelse. Security Officer mangler en profil.

Security Officer er ikke en ubetydelig rolle: den er second authorizer ved Root CA-aktivering, CA-revokering, HSM factory reset, wrap-key rotation og emergency CA shutdown. Without to-mands-princippet operationaliseret i Security Officer-rollen, er dual-control for de mest kritiske PKI-operationer udokumenteret.

Rettelse: Tilføj Security Officer-profil (analog til de fem øvrige profiler). Profilen skal indeholde: ansvar, kompetence-krav, SOD-begrænsninger (navnlig: må IKKE kombineres med HSM-operatør ved samme operation; funktionelt uafhængig af CA Administrator), rapport-linje (Mads), og fase-0-undtagelse (Mads varetager midlertidigt, normaliserings-deadline inden 1. eksterne kunde).

**PKI-G2: HSM-operatør-profil dækker ikke HA-pair-ansvar (betinget)**

FGDs 2026-05-09-beslutning om Tier 2 HA-pair (2× YubiHSM2, wrap-key replication) er ikke reflekteret i HSM-operatør-profilen. Profilen beskriver single-HSM-operation. Med HA-pair-beslutning bliver HSM-operatøren ansvarlig for wrap-key-replication-procedure, sync-test og failover-verifikation.

Rettelse: Tilføj HA-sync-relaterede opgaver til HSM-operatør-profilens "Specifikke opgaver": "(e) HA-sync: kvartalvis verifikation at wrap-key-replication til sekundær YubiHSM2 er intakt og kan restore; (f) Failover-test: halvårlig test af failover fra primær til sekundær YubiHSM2 under Mads-opsyn; Trine attesterer."

**PKI-G3: KCO — AP som ceremony-script-medforfatter (præcisering)**

KCO-profilen angiver at ceremony-script-development sker i "Samarbejde med Application Provider + Mads". Dette er korrekt proceduremæssigt, men bør præcisere afgrænsningen: AP er teknisk forfatter af script-trin; KCO er autoritet for proceduren som helhed; Mads godkender det færdige script. AP autoriserer ikke ceremony-eksekvering — det er KCOs og Security Officers domæne. Denne afgrænsning er vigtig for at AP ikke ubevidst antager en autoriseringsrolle den ikke må have (jf. AP hard rule om key ceremonies).

Rettelse: Tilføj præcisering i KCO-profilen: "Application Provider leverer teknisk script-udkast; Key Ceremony Officer er eneste procedurelle autoritet; Mads godkender samlet script inden ceremoni."

**PKI-G4: RA-model-anbefaling haiku — overvej begrænsning (betinget)**

RA-profilen anbefaler `haiku`-model med begrundelse "regelbaseret godkendelse; CSR-struktur-validering er routine; komplekse ident-cases eskaleres." Dette er rimeligt for rutinetilfælde, men RA-rollen indeholder policy-fortolkning og identitetsverifikation der ved kanttilfælde kræver nuanceret vurdering. Haiku er acceptabelt hvis eskalerings-pligten er operationelt skarp (komplekse cases sendes til CA Administrator + Mads inden 1 time). Denne skarphed bør fremgå eksplicit af det endelige RA-mandat.

Dette er ikke en blocker i profil-dokumentet, men Laila bør notere det i forbindelse med mandat-design.

**PKI-G5: "Lagd af" staveform (redaktionel)**
"Lagd af" → "Lagt af" (alle tre dokumenter).

### Samlet for PKI-governance-roller

Én blocker (G1: Security Officer mangler). To betingede fund (G2, G3) kræver inddatering inden FGD-GO. G4 og G5 er redaktionelle noter til Laila.

---

## Krævede rettelser inden aktivering

Nedenstående liste er struktureret i prioritetsrækkefølge. "Inden aktivering" betyder: inden Camilla committer mandaterne og FGD giver GO.

### Blockers (skal rettes inden FGD-GO)

**B1 — AP-mandatets AP=Tech Lead-konflikt**
- Ansvarlig: Laila (rettelse) + FGD (beslutning om option A vs. B)
- Specifik handling: FGD beslutter om AP og Tech Lead kombineres i fase 0 (undtagelse med normaliserings-deadline) eller separeres fra dag 1. Laila retter AP-mandatteksten og PKI-governance-roller-profilen konsistent.
- Hvis option A: opret `governance/sod-undtagelser/2026-05-09-ap-tech-lead-fase0.md` med normaliserings-deadline og Trine-attestationsplan.
- Reference: Pouls org-struktur §4 SOD-matrix fodnote ‡; SOC 2 CC6.1

**B2 — AP-mandatets FIPS-status-uoverensstemmelse**
- Ansvarlig: Laila (rettelse baseret på dette review)
- Specifik handling: Erstat "til FIPS 140-3 Level 3" i mission §4 med "FIPS 140-3 Level 3-ready (fase 0: non-FIPS MVP per FGDs 2026-05-09-beslutning; re-keying ved FIPS 140-3-cert fra NIST CMVP)". Tilsvarende rettelse i hard rules, FIPS-compliance-design-afsnit.
- Reference: Mads krav-dok §0 (compliance-note); Pouls vinkel 5 (blokerende fund); FGDs 2026-05-09-beslutning (Pouls org-struktur frontmatter)

**B3 — PKI-governance-roller mangler Security Officer-profil**
- Ansvarlig: Laila (tilføjer profil)
- Specifik handling: Tilføj Security Officer-profil som §7 i PKI-governance-roller-dokumentet. Profilen skal følge same format som de øvrige fem profiler og inkludere fase-0-undtagelse (Mads midlertidigt), normaliserings-deadline, SOD-begrænsninger (navnlig: ikke HSM-operatør ved samme operation), og rapport-linje (Mads).
- Reference: Mads krav-dok §1.7; Pouls org-struktur §1 og §3.2

**B4 — AO-mandatets observer-tilladelse ved key ceremony**
- Ansvarlig: Laila (rettelse)
- Specifik handling: Fjern "kan deltage som observer ved ceremoni hvis relevant" fra AO hard rules. Tilføj i stedet eksplicit: "Du deltager IKKE i key ceremony i nogen rolle — hverken som autoriserende part, observer eller witness. Enhver AO-tilstedeværelse ved ceremony er forbudt og behandles som SOD-brud."
- Reference: Pouls org-struktur §5.1 (AO forbud); Mads krav-dok §1.1 SOD-krav

**B5 — AO-mandatets compliance-eskalerings-selvstændighed**
- Ansvarlig: Laila (tilføjer hard rule)
- Specifik handling: Tilføj eksplicit hard rule: "Mads, Trine og Dorthe kan eskalere compliance-fund direkte til FGD uden om Application Owner. AO kan ikke filtrere, forsinke eller omformulere compliance-rapportering fra disse roller. Dette er et SOC 2 CC4.1-krav (audit-uafhængighed)."
- Reference: Pouls org-struktur §2.2 (AO eskalations-sti); SOC 2 CC4.1; ISO 27001 A.5.3

### Betingede rettelser (rettes sideløbende med blockers, inden FGD-GO)

**C1 — AP audit-trail SOC 2-forankring**
- Tilføj reference til WORM-storage/kryptografisk chained log og ekstern revisor-adgangskrav i AP's audit-trail-hard-rule (SOC 2 CC7.2).

**C2 — AP break-glass for PII-bærende systemer**
- Udvid break-glass til at gælde CSR-database og audit-log-data med PII. Koordineres med Dorthes GDPR-vurdering.

**C3 — HSM-operatør HA-pair-opgaver**
- Tilføj HA-sync og failover-test som specifikke opgaver i HSM-operatør-profilen.

**C4 — KCO ceremony-script-afgrænsning**
- Tilføj præcisering: AP er teknisk forfatter; KCO er procedureel autoritet; Mads godkender.

### Redaktionelle rettelser (rettes inden commit)

- AP-udkast: "Designes" → "Designer" (flere forekomster)
- AP-udkast: "Met" → "Med" (hand-off-aftaler, tre forekomster)
- AP-udkast: "rappotere" → "rapportere"
- AP-udkast: "planegger" → "planlægger"
- AP-udkast: "TVivl" → "tvivl"
- AP-udkast: "audit.events" → "FGDCorePKI audit-events" eller neutral PKI-terminologi
- AO-udkast: "uhovedede" → "ubegrænset" eller "uhindret"
- Alle tre udkast: "Lagd af" → "Lagt af"

---

## Governance-noter

### SOD-absolute forbud — bekræftet til stede i alle tre dokumenter

Samtlige fire absolutte SOD-forbud (CA Administrator × PKI Auditor, CA Administrator × RA, PKI Auditor × enhver operationel rolle, AO × CA Administrator) er korrekt angivet i alle relevante profiler og mandater. Disse er ikke i fare.

### Mandat-stabilitet — bekræftet

Alle tre dokumenter indeholder korrekt reference til `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` og forbud mod self-modification. Dette er tilfredsstillende.

### FIPS 140-3 CMVP-status — løbende overvågning påkrævet

YubiHSM2 FIPS 140-2 cert (#3916) udløb 2026-05-02. FIPS 140-3 cert er endnu ikke granted af NIST CMVP per 2026-05-09. FGDs MVP-beslutning (non-FIPS YubiHSM2) er korrekt reflekteret i AO-mandatet men IKKE i AP-mandatet (blocker B2 ovenfor). Stefan og Mads tjekker CMVP active list ugentligt; re-keying-event planlægges fra dag 1 som planlagt fremtidig handling, ikke som evt. fremtidig handling.

### Dorthe-koordinering kræves

Dorthe skal vurdere:
1. GDPR-implikationer af CSR-databehandling (Subject-felter med personoplysninger)
2. Om audit-log-kategorier i PKI-governance-roller-dokumentet kræver DPIA-opdatering
3. Break-glass-udvidelse til PII-systemer (C2 ovenfor)

Mads koordinerer midlertidigt GDPR-vurderingen efter FGD-override 2026-05-06 indtil Dorthe er fuldt aktiveret.

---

*Vurderet af: Mads (CISO)*
*Dato: 2026-05-09*
*Næste handling: Laila retter per blockers B1-B5 + betingede C1-C4. FGD beslutter option A/B for B1 (AP=Tech Lead-valg). Dorthe vurderer GDPR-impact parallelt. Stefan koordinerer samlet scorecard til FGD-GO.*
*Trine logger dette review-event i audit-trail ved aktivering.*
