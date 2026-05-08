# Laila-rapport: Stefan-mandat v2 — Rettelser gennemført

**Dato**: 2026-05-08  
**Fil**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-08-stefan-organisator-mandat.md`

---

## Sammenfatning

Begge compliance-drevne rettelser (FGD + Mads + Dorthe review) er gennemført. Ingen andre ændringer lavet.

---

## Rettelse 1 — Webadgang begrænses til skills/plugins/roller

**Dataminimering (GDPR art. 5(1)(c))**: Stefan får webadgang KUN via `find-skills`-skill til at opdage nye skills, plugins og roller. Al anden research går gennem Poul.

### Præcise linjer ændret:

1. **Linje 28** ("Hvad Stefan MÅ selv"):  
   - Før: `- **Søge** find relevante filer, patterns, kontekst både internt i filer og på websites`
   - Efter: `- **Søge** (\`Glob\`, \`Grep\`) — find relevante filer, patterns og kontekst internt i repo. Webadgang er **kun** tilladt via \`find-skills\`-skill til at opdage nye skills, plugins og roller — ikke til general research (det er Pouls domæne).`

2. **Linje 42** ("Hvad Stefan ALDRIG må selv"):  
   - Før: `- **Hente eksterne kilder** — Stefan bruger Poul til deep research.`
   - Efter: `- **Hente eksterne kilder til research** — Poul gør WebSearch/WebFetch for research og kildedyk. Stefan bruger kun webadgang via \`find-skills\`-skill til at opdage nye skills/plugins/roller.`

3. **Linje 125** ("Skills — Should-use"):  
   - Før: `- \`find-skills\` — overlap-tjek med eksisterende roller før Laila designerer`
   - Efter: `- \`find-skills\` — opdage nye skills/plugins/roller (eneste tilladt webadgang for Stefan); overlap-tjek med eksisterende roller før Laila designerer`

---

## Rettelse 2 — DPO-routing compliant med GDPR art. 38(3)

**DPO-uafhængighed**: Dorthe eskaleres direkte for GDPR-spørgsmål. Ikke via Mads (bryder art. 38(3)). FGD beslutter på baggrund af både Mads' security-scorecard OG Dorthes GDPR-anbefaling.

### Præcise linjer ændret:

4. **Linje 110** ("Ansvarsfordeling med andre roller" — Dorthe-rækken):  
   - Før: `| **Dorthe** | Stefan eskalerer GDPR-spørgsmål til Mads, som koordinerer med Dorthe. |`
   - Efter: `| **Dorthe** | Stefan eskalerer GDPR-spørgsmål direkte til Dorthe (jf. GDPR art. 38(3)). Dorthe er uafhængig DPO — ikke en del af Mads' hierarki. |`

5. **Linje 172** ("Hand-off-aftaler" — Mads-linjen):  
   - Før: `- **Med Mads**: Stefan beder Mads og Dorte vurdere compliance før feature-go; Mads leverer scorecard; Stefan samler til FGD.`
   - Efter: `- **Med Mads**: Stefan beder Mads vurdere security-compliance (FIPS/SOC2/ISO27001) før feature-go; Mads leverer security-scorecard til Stefan.`

6. **Linje 172-173** ("Hand-off-aftaler" — Ny linje efter Mads):  
   - Tilføjet: `- **Med Dorthe**: Stefan eskalerer GDPR-spørgsmål direkte til Dorthe (jf. GDPR art. 38(3)). Dorthe rådgiver; FGD beslutter på baggrund af Dorthes anbefaling. Dorthe er ikke en del af Mads' compliance-hierarki.`

7. **Linje 91** ("Governance-integration"):  
   - Før: `Ingen feature går til produktion uden FGD's eksplicitte GO baseret på Dorte og Mads' compliance-scorecard.`
   - Efter: `Ingen feature går til produktion uden FGD's eksplicitte GO. FGD beslutter på baggrund af Mads' security-scorecard og Dorthes GDPR-anbefaling — DPO rådgiver, dataansvarlig (FGD) beslutter.`

---

## Verifikation

- ✓ Søge-linjen i "MÅ" sektionen nævner nu kun `find-skills` til webadgang
- ✓ "ALDRIG"-linjen om eksterne kilder er opdateret
- ✓ `find-skills` i Skills-sektionen har præcisering
- ✓ Ansvarsfordeling-Dorthe-rækken siger "direkte" og nævner art. 38(3)
- ✓ Hand-off har separat Dorthe-linje
- ✓ Governance-integration linje siger "FGD beslutter på baggrund af ... Dorthes anbefaling"

---

## v3 — Minor fixes (GDPR art. 5(1)(c) dataminimering)

**FGD-godkendelse**: 2026-05-08 — to precision-rettelser vedr. persondata i memory.

### Rettelse 3 — Hard rule 6: Forbud mod persondata om rigtige mennesker

**Baggrund**: GDPR art. 5(1)(c) + art. 25 kræver dataminimering. Memory må ikke indeholde persondata om rigtige mennesker. Præcisering: AI-agenter/roller er ikke fysiske personer under GDPR.

**Præcis linje ændret:**

1. **Linje 100** ("Hard rules" — Hukommelse-disciplin):  
   - Før: `6. **Hukommelse-disciplin** — Alle opgaver skal I beskrivelsen sikre at der indgår at Camilla er dokumentations ansvarlig og at ændringer uden for Team/[ agent navn] er nogo. Er der tale om opgave resultater eller kode til andre agenter er det Camilla der flytter det til korrekt destination.`
   - Efter: `6. **Hukommelse-disciplin** — [...samme intro...]. Memory må **aldrig** indeholde persondata om rigtige mennesker (navne, kontaktoplysninger, adfærd, identifikationsoplysninger om tredjepart). AI-roller og agenter er ikke omfattet af dette forbud.`

### Rettelse 4 — Memory-bullet: Stefan-ansvar for persondata-udelukkelse

**Baggrund**: Præcisering af Stefans proceduralt ansvar når han instruerer Camilla om memory-opdateringer.

**Præcis linje ændret:**

1. **Linje 39** ("Hvad Stefan ALDRIG må selv" — Memory-bullet):  
   - Tilføjet til slutningen: `Stefan sikrer at instruktioner til Camilla om memory-opdateringer aldrig inkluderer persondata om rigtige mennesker.`

---

---

## v4 — 4-Eyes Princip + Godkendelseslog (FGD-tilføjelse 2026-05-08)

**FGD-godkendelse**: 2026-05-08 — 4-eyes-princip formaliseres med obligatorisk godkendelseslog og Stefans eskaleringsansvar.

### Rettelse 5 — Poul-række i delegerings-hierarki opdateret

**Linje 72** ("Delegerings-hierarki" — Poul-række):
- Før: `| **Poul** | Behovs-analyse, research, arkitektur-vurdering, vendor-selection, version-tjek | Senior analytiker — research og behovs-kortlægning. Flere analyser parallelt tilladt. Implementer 4-eyes princippet |`
- Efter: `| **Poul** | Behovs-analyse, research, arkitektur-vurdering, vendor-selection, version-tjek | Senior analytiker — research og behovs-kortlægning. Flere analyser parallelt tilladt. Pouls analyser der danner grundlag for væsentlige ændringer peer-reviewes af Mads (teknisk/compliance) eller Dorthe (GDPR) jf. §4-Eyes Princip. |`

### Rettelse 6 — "Godkendelseslog"-undsektion tilføjet til §4-Eyes Princip

**Indsættelse efter "Hvem er de 2 autoriserede reviewere?"-tabellen (før "Stefans ansvar"):**

Ny undsektion `### Godkendelseslog — Camillas ansvar, Stefans pligt` med:
- Definition: Camilla fører løbende log over alle store beslutninger under 4-eyes
- Stefans ansvar: be Camilla opdatere loggen efter hver godkendelse (dato, beslutning, reviewer 1, reviewer 2, FGDs GO)
- Eskalering: Hvis loggen ikke opdateres → Stefan gør Mads og Dorthe opmærksomme straks
- Logfil-lokation: `governance/godkendelseslog.md`

### Rettelse 7 — "Stefans ansvar under 4-eyes" opdateret

**Præcisering af Stefans proceduralt ansvar:**
- Tilføjet: `Stefan **beder Camilla** logge godkendelsen umiddelbart efter FGDs GO.`
- Tilføjet: `Hvis en reviewer er utilgængelig: Stefan noterer dette eksplicit og beder FGD beslutte om at vente eller acceptere single review som undtagelse — undtagelsen logges.`

### Verifikation v4

- ✓ Poul-række nævner nu peer-review af Mads/Dorthe per 4-Eyes Princip
- ✓ Ny "Godkendelseslog"-undsektion forklarer Camillas rolle og Stefans eskalerings-ansvar
- ✓ "Stefans ansvar"-undsektion nævner logging og undtagelse-logging
- ✓ Hard rule 7 (eksisterende, linje 101) refererer korrekt til §4-Eyes Princip
- ✓ Alle ændringer sikrer at godkendelser er tracerbare og compliance-revisable

---

## Bemærkninger

- **Rettelser 1-4 (v2-v3)** — allerede gennemført og i memory.
- **Rettelser 5-7 (v4)** — tilføjet som en følge af FGDs 4-eyes-direktiv fra 2026-05-08.
- **Filstatus**: Ready for FGD-godkendelse + Camilla-commit.
- **Arkitektur**: 4-eyes-princippet er nu integreret i Stefans kernel-ansvar via Poul-rækken, Godkendelseslog-undsektion, og Hard rule 7. Logging bliver enforceable via Camillas ansvar og Stefans eskalerings-tilsyn.
