---
dato: 2026-05-08
forfatter: Poul (Reviewer 2)
emne: 4-eyes review af Dorthe-DPO-mandat (GDPR-faglig + rolle-komplethed)
reviewer-1: Mads (CISO) — SOD/governance-vinkel (separat rapport)
kilde: Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md
status: review færdig
---

# 4-eyes review: Dorthe (DPO) mandat-udkast

## Overordnet vurdering: **CONDITIONAL GO**

Mandatet er fagligt solidt og dækker hovedparten af GDPR art. 39(1).
To kritiske rettelser kræves før commit (breach-fristen og art. 39(1)(b)/(d)
eksplicitering). Resten er minor præciseringer.

---

## Fund

### 1. KRITISK — Breach-fristen (linje 45) er operationelt uholdbar

**Linje 45**: *"Breach-respons — notifikation til Datatilsynet inden 72 timer
(art. 33) + data-subjects hvis risiko (art. 34) -> FGD skal godkende inden"*

**Problem**: GDPR art. 33(1) kræver notifikation *"without undue delay and,
where feasible, not later than 72 hours after having become aware"*. Fristen
løber fra **kendskab til bruddet** — ikke fra FGDs godkendelse. Hvis FGD
er utilgængelig (rejse, weekend, sygdom), bryder formuleringen art. 33.

**Sekundær risiko**: Hvis notifikation forsinkes >72t, skal årsagen begrundes
over for Datatilsynet (art. 33(1) sidste pkt.). "Vi ventede på CEO-godkendelse"
er ikke en accepteret begrundelse iflg. EDPB Guidelines 9/2022 on Personal
Data Breach Notification.

**Rettelsesforslag**:
```
3. Breach-respons — notifikation til Datatilsynet inden 72 timer fra kendskab
   (art. 33) + data-subjects hvis høj risiko (art. 34).
   - DPO (Dorthe) er notifikations-myndighed; FGD informeres parallelt.
   - Hvis FGD ikke er nået inden 60 timer fra kendskab: Dorthe notificerer
     alligevel og dokumenterer beslutningen i breach-loggen.
   - FGD kan efterfølgende kræve supplerende notifikation, men ikke
     tilbagekalde den oprindelige.
```

Dette er også konsistent med art. 38(3): DPO må ikke modtage instruks om
udførelse af lovpligtige opgaver, herunder notifikation.

---

### 2. KRITISK — art. 39(1)(b) og (d) er underspecificerede i Mission

Mission-listen (linje 41-48) dækker:
- (a) "informere og rådgive" — **delvist dækket** via DPIA-gennemgang og
  ROPA, men ingen eksplicit advisory-funktion til ledelsen/teamet
- (b) "overvåge overholdelse" — **mangler eksplicit**. Compliance-monitoring
  er DPOs definerende opgave; det skal stå i Mission, ikke kun implicit i ROPA
- (c) DPIA — **dækket** (punkt 2)
- (d) "samarbejde med tilsynsmyndighed" + "kontaktpunkt for Datatilsynet"
  — **mangler eksplicit**. Linje 124 nævner DPO-kontaktoplysninger, men ikke
  selve samarbejdsfunktionen
- (e) Risikobaseret tilgang — **delvist** via DPIA, men ikke generaliseret

**Rettelsesforslag**: Tilføj som punkt 0 og 7 i Mission-listen (linje 41):
```
0. **Compliance-monitoring** (art. 39(1)(b)) — løbende overvågning af GDPR-
   overholdelse på tværs af features, leverandører og processer; årlig
   compliance-rapport til FGD.
7. **Tilsynsmyndighed-kontakt** (art. 39(1)(d)) — primær kontakt til
   Datatilsynet ved forespørgsler, klager, audits; samarbejde ved
   forhåndskonsultation (art. 36) hvis høj-risk DPIA ikke kan mitigeres.
8. **Advisory** (art. 39(1)(a)) — rådgivning til FGD og team om GDPR-
   forpligtelser ved nye features, kontrakter, kunde-onboarding.
```

---

### 3. MEDIUM — Fase 0-2 mangler SaaS-specifikke DPO-aktiviteter

Mandatets stack-kontekst er Neon, Vercel, Clerk, Anthropic, Inngest, Voyage,
Tailscale, Yubico (linje 122). Følgende DPO-opgaver er underrepræsenterede
i Fase 0-2:

**Fase 0 mangler**:
- **Adequacy/SCC-vurdering pr. leverandør**: Anthropic (US, DPF-certificeret?),
  Voyage (US), Vercel (US, DPF), Clerk (US). Schrems II-konsekvens skal
  dokumenteres pr. leverandør, ikke kun "DPA-tracker udfyldt".
- **TIA (Transfer Impact Assessment)** for hver tredjelands-overførsel
  (EDPB Recommendations 01/2020).

**Fase 1 mangler**:
- **AI-specifik DPIA-template**: Anthropic-behandling af noter (input til
  LLM) er høj-risk processing iflg. EDPB Guidelines on AI/automated
  decision-making. DPIA bør specifikt addressere art. 22 (automated decision-
  making) selvom output er rådgivende.
- **Embedding-retention-policy**: Voyage embeddings er afledt PII; retention
  vs. originaltekst skal afklares.
- **Datatilsynets vejledning om AI** (DK-specifik, opdateret 2025) bør checkes.

**Fase 2 mangler**:
- **Sub-processor-notifikation**: når FGD bliver databehandler for kunder,
  skal kunder notificeres om sub-processors (Anthropic, Voyage osv.) iflg.
  art. 28(2)+(4). Mekanisme mangler.
- **Customer-DPA: data-export/sletnings-SLA** (art. 28(3)(g)) — hvor
  hurtigt FGD kan returnere/slette kundedata ved kontraktophør.

**Rettelsesforslag**: Udvid Fase 0-2-lister med ovenstående punkter
(behøver ikke detaljeres her — Dorthe selv kan udfylde under Fase 0).

---

### 4. KRITISK (principielt) — Project-rolle-navne i Anti-patterns

**Linje 143-144**:
```
- Implementerer ikke breach-notification-system (Henrik gør det — Dorthe designer krav)
- Drifter ikke HSM (Bjarne gør det)
```

**Vurdering: Principielt vigtigt, ikke kosmetisk.**

Begrundelse:
1. Memory-feedback `_crosscutting/feedback_permanent_vs_project_separation.md`
   gør det eksplicit: permanente mandater må **ikke** navngive project-roller.
2. Mandatet selv siger det rigtige under linje 70 (Hard rules): *"Dorthe's
   mandat refererer ikke specifikke project-rolle-navne (som Karen, Bjarne,
   Henrik). I stedet bruger hun generiske termer..."* — så linje 143-144
   modsiger direkte mandatets egen hard rule. Det er en intern inkonsistens
   der vil forplante sig hvis ikke rettet nu.
3. Hvis knowlagecentral-projektet senere lukkes eller Henrik/Bjarne udskiftes,
   bliver mandatet stale. Permanent rolle skal være projekt-uafhængig.

**Rettelsesforslag**: Erstat linje 143-144 med:
```
- Implementerer ikke breach-notification-system (backend-udvikler-rollen
  gør det — Dorthe designer GDPR-krav)
- Drifter ikke HSM eller anden krypto-infrastruktur (HSM-operator-rollen
  gør det — Dorthe verificerer at nøgle-håndtering opfylder art. 32)
```

---

### 5. MEDIUM — DPO-uafhængighed (art. 38(3)) er i konflikt med mandat-låsen

**Linje 15, 50, 150-151**: Uafhængigheden formuleres flere steder og er
i sig selv god. **Men** mandat-stabilitets-sektionen (linje 161-184) skaber
en konflikt:

**Linje 170**: *"Hard rules (body §X) | Mads + Dorthe + FGD"* — ændringer
til Dorthes hard rules kræver Mads' godkendelse. Dette er problematisk
for hard rules der definerer **DPO-uafhængighed** (linje 69, 76).

Eksempel: Hvis FGD ønsker at fjerne *"Uafhængig vurdering"*-hard-rulen,
kræver mandatet Mads' godkendelse. Men Mads kan have en interesse i at
DPO ikke er uafhængig (mindre friktion i security-beslutninger).
Det er præcis den situation art. 38(3) forhindrer.

**Sekundært**: Linje 178: *"Vent på Mads' security/SOD-vurdering + Dorthes
GDPR-vurdering"*. For Dorthes egne mandat-ændringer er det odd at Dorthe
selv skal lave GDPR-vurderingen — interessekonflikt.

**Rettelsesforslag**:
```
| Hard rules vedr. DPO-uafhængighed (art. 38(3)) | (linje 69, 76) |
  KUN FGD (lovkrav: DPO-uafhængighed kan ikke gradueres af CISO) |
| Øvrige hard rules | (resterende) | Mads + Dorthe + FGD |
```

Og: Ved Dorthes egne mandat-ændringer skal GDPR-vurderingen laves af
**ekstern juridisk rådgiver eller FGD direkte**, ikke Dorthe selv.

---

### 6. MINOR — Reporting-linje til FGD (linje 15, 86, 150)

**Linje 15**: *"Hun rapporterer direkte til FGD, ikke gennem Mads-hierarkiet."*
**Linje 86**: *"Stefan (orkestring) | Eskalering, status | Dorthe rapporterer;
Stefan koordinerer med FGD"*

**Konflikt**: Reelt rapporterer Dorthe **gennem Stefan**. Dette er
operationelt fint (Stefan er orkestrator, ikke ledelse), men formuleringen
"direkte til FGD" er ikke 100% sand i det observerbare flow.

**Rettelsesforslag**: Præcisér at Stefan er kommunikations-kanal, ikke
filter:
```
Dorthe rapporterer direkte til FGD. Stefan formidler praktisk (timing,
samling), men må ikke redigere, filtrere eller forsinke DPO-rapporter
til FGD. Dorthe har ret til direkte FGD-kontakt ved haster (breach,
art. 33).
```

Dette adresserer art. 38(3): *"shall directly report to the highest
management level"*.

---

### 7. MINOR — Ansvar mellem Mads (security) og Mads (governance) (linje 83-84)

Tabellen lister Mads to gange (CISO og security). Det er let at læse
forkert — det er **én** rolle med to vinkler. Cosmetic.

**Rettelsesforslag**: Slå sammen til én række:
```
| **Mads (CISO)** | Governance + threat-models | Mads ejer både governance-
  rammer og sikkerhedsreviews. Dorthe og Mads sparrer peer-niveau om
  features (Mads: trussel; Dorthe: GDPR-dataflow). |
```

---

### 8. MINOR — DPA-leverandører (linje 122) bør medtage Postman, Microsoft 365, Google

Hvis FGD bruger Postman MCP, Microsoft 365, Google Workspace til
arbejdsdata (jf. system-promptens MCP-liste), er disse også
databehandlere og skal i DPA-trackeren. 8 leverandører er muligvis
undertal.

**Rettelsesforslag**: Lad Fase 0-første-opgave være "kortlæg ALLE
databehandlere" snarere end at låse til 8.

---

## Sammenfatning af GO/NO-GO-betingelser

**Før FGD-go skal følgende rettes:**

| # | Fund | Status |
|---|---|---|
| 1 | Breach-frist (linje 45) | KRITISK — skal rettes |
| 2 | art. 39(1)(b),(d) i Mission | KRITISK — skal rettes |
| 4 | Project-rolle-navne (linje 143-144) | KRITISK (principielt) — skal rettes |
| 5 | Uafhængigheds-konflikt i mandat-lås | MEDIUM — bør rettes |
| 3 | SaaS-DPO-aktiviteter i Fase 0-2 | MEDIUM — kan udskydes til Dorthes Fase 0-arbejde |
| 6 | Reporting-linje (linje 15) | MINOR |
| 7 | Mads-tabel-konsolidering | MINOR |
| 8 | DPA-leverandør-liste | MINOR |

**Anbefaling til Stefan**:
- Bed Laila revidere udkastet med fund 1, 2, 4, 5 (KRITISK + MEDIUM
  uafhængighed) inkorporeret.
- Fund 3 kan adresseres ved at Dorthe selv udvider Fase 0-checklisten
  efter ansættelse — det er hendes faglige domæne.
- Minor-fund (6, 7, 8) kan rettes i samme runde eller i v2.

**Når disse er rettet: GO.**

---

## Åbne spørgsmål til FGD

1. **DPO-titel**: Skal FGD selv være formel DPO (jf. linje 124: "DPO er
   FGD; Dorthe er operationel støtte"), eller er Dorthe den formelle DPO?
   Det har betydning for kontaktinfo-publicering (art. 37(7)) og hvem
   Datatilsynet skriver til. Memory `dpo-model-2026-05-05.md` siger FGD
   = juridisk DPO, Dorthe = operationel — men mandatet er ikke 100%
   konsistent på dette punkt (linje 9 kalder hende "DPO", linje 124
   siger "FGD er DPO").

2. **Ekstern juridisk backup**: Ved DPO-mandat-ændringer der berører
   uafhængighed, hvem tager den juridiske vurdering? FGD selv, eller
   ekstern advokat? (Anbefaling: ekstern ved tvivl.)
