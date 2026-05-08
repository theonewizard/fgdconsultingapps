---
dato: 2026-05-06
forfatter: Mads (CISO)
emne: Governance-anbefaling for mandat-stabilitet (model + evt. tools/description/hard rules)
status: leveret
modtagere: Laila (HR), Stefan (orkestrator), FGD (godkender)
referencer:
  - SOC 2 Type II — CC1.4 (kompetencer), CC5.2 (kontrolaktiviteter), CC8.1 (change management)
  - ISO/IEC 27001:2022 — A.5.2 (roller/ansvar), A.5.3 (SOD), A.6.3 (awareness), A.8.2 (privilegerede adgange), A.8.32 (change management)
  - GDPR art. 5(1)(f), 25 (privacy by design), 32 (sikkerhed v. behandling)
  - NIST 800-53 r5 — AC-5 (SoD), AC-6 (least privilege), CM-3 (configuration change control)
---

> **⚠ §3.3 trigger-baseret tilgang for Dorthe er SUPERSEDED af FGD-override 2026-05-06**: Mads + Dorthe SAMMEN i ALLE rolle-ændringer (ikke trigger-baseret). Se Lailas rapport `Min Inbox/_crosscutting/2026-05-06-laila-rapport-governance-override.md` for korrekt implementering.

# Governance-anbefaling: mandat-stabilitet på tværs af 13-14 rolle-mandater

## 1. Sammenfatning

**Anbefaling:** Governance-låsen skal udvides ud over `model`-feltet til at omfatte fire **kontrolpunkter** med differentieret eskalering:
(a) **`model`** og **body-hard-rules** = altid eskalering til Mads + Dorthe + FGD;
(b) **`tools`** = altid eskalering til Mads (least-privilege/SOD-impact); Dorthe kun hvis tools rører persondata/log-flader;
(c) **`description`** = Laila + Mads (agent-routing-impact, ikke compliance-grænser);
(d) **andre frontmatter** (color, name) = Laila alene.
Self-modificering er **forbudt** for alle roller (SOC 2 CC8.1, ISO 27001 A.8.32, GDPR art. 32 — change-management-princip). Skabeloner findes nedenfor i §5. Tilføjelse til Stefan-CLAUDE.md anbefales (§5.3).

---

## 2. Spørgsmål 1 — Scope: model alene, eller bredere?

**Anbefaling: Bredere scope, men differentieret pr. felt — ikke "alt eller intet".**

Et CISO-perspektiv: model-downgrade er kun ÉN vej til at omgå compliance-niveau. De andre felter har lige så stor — i nogle tilfælde større — risiko-eksponering:

| Felt | Risiko hvis ukontrolleret | Eksempel | Eskalering |
|---|---|---|---|
| **`model`** | Output-kvalitet falder under compliance-tærskel (fx security-review udført af haiku misser threats sonnet ville fange) | Mads' model skiftes haiku for "fart" → SOC 2 CC1.4 (kompetence) brydes | **Mads + Dorthe + FGD** |
| **`tools`** | Privilege-eskalering (SOD-brud), least-privilege-brud, audit-blind-spots | Camilla får `WebFetch` tilføjet → kan eksfiltrere via DNS; eller Karen får `Bash` → kan køre HSM-ops uden Bjarne | **Mads + FGD**; Dorthe hvis tools rører persondata/logning |
| **Hard rules (body)** | Compliance-grænser flyttes (fx "must-not-write-to-production" fjernes) | Bjarne-mandat: "destruktive HSM-ops kræver Mads-godkendelse" fjernes → SOC 2 CC5.2 brydes | **Mads + Dorthe + FGD** |
| **`description`** | Agent-vælger ruter sensitive opgaver til forkert rolle (compliance-flow brydes indirekte) | Stefan kalder Camilla i stedet for Mads til DPA-arbejde fordi description er udvidet | **Laila + Mads** (ikke FGD-niveau) |
| `color`, `name` | Ingen security/compliance-impact | — | **Laila alene** |

**Argument mod "kun model"** (afvist): Et team-medlem der vil omgå governance vil intuitivt finde det svageste led. Hvis `model` er låst men `tools` er frit, vil omgåelse ske via tools. Compliance-frameworks (ISO 27001 A.8.32, SOC 2 CC8.1) tænker i **change management som helhed**, ikke pr-felt.

**Argument mod "alt skal til FGD"** (afvist): Skaber friktion → governance bliver omgået i praksis ("vi opdaterer bare description, det er jo tekst"). Differentiering bevarer Lailas rolle-design-autoritet samtidig med at compliance-grænser låses.

**Konklusion:** 4-niveau-model (FGD-eskalering / Mads-eskalering / Laila+Mads / Laila alene) — implementeret pr. felt-type, ikke pr. rolle-type. Detaljeret skabelon i §5.

---

## 3. Spørgsmål 2 — Eskaleringspath pr. rolle-type

**Stefans foreslåede model accepteres med to justeringer.**

### 3.1 Permanente roller (Stefan, Laila, Poul, Camilla, Mads, Vibeke, Dorthe, Trine)

**Sti:**
1. Forslagsstiller skriver til `governance/change-requests/<dato>-<rolle>-<felt>.md` (impact-vurdering)
2. **Mads** vurderer security/SOD-impact (~24 h)
3. **Dorthe** vurderer GDPR/regulatorisk impact (~24 h, parallelt) — *betingelse, se §3.3*
4. **Trine** noterer audit-trail (CC8.1 evidence)
5. Fælles anbefaling → **Stefan koordinerer til FGD** for go/no-go
6. **Camilla** implementerer ændringen efter FGD-go (commit + memory-opdatering)

### 3.2 Project-roller (Sofie, Henrik, Anders, Karen, Bjarne, Frederik)

**Sti:**
1. Rollen eller Stefan skriver impact-vurdering
2. **Mads** (deres overordnede for alt security/governance-relevant) vurderer
3. **Dorthe** konsulteres ved GDPR-impact-trigger (§3.3)
4. **Mads** anbefaler — go ved engangs/ikke-permanente; FGD-go ved permanente
5. **Camilla** implementerer

### 3.3 Justering 1: Dorthe ER IKKE altid med — trigger-baseret konsultation

Dorthe skal konsulteres **hvis ÉN af følgende triggers** er sand:

- Rollen rører persondata (data subjects, log-data, audit-trails med PII)
- Rollen er customer-facing eller producerer output der kan ende hos kunde
- Ændringen påvirker breach-respons-kapacitet (fx fjernelse af incident-response-tools)
- Ændringen påvirker DPA/sub-processor-kæden (fx ny ekstern integration)
- Rollen håndterer cross-border data transfer (EU/US)

**Eksempler hvor Dorthe SKAL ind:** Camilla (logger PII via memory), Sofie (frontend håndterer brugerinput), Anders (AI-pipeline behandler kundeinput), Karen (incident response), Stefan (orkestrerer cross-rolle-data).

**Eksempler hvor Dorthe IKKE behøver:** Bjarne (HSM-operatør, hardware-isoleret, ingen direkte PII-access — Mads alene vurderer), Frederik (intern infra), Trine (audit, ingen behandling), Poul (research, eksterne kilder, ingen PII).

**Begrundelse:** GDPR art. 35 (DPIA-trigger-logik) bruger samme tærskel — "high risk to rights and freedoms". Mads-only-eskalering for hardware/intern roller bevarer hastighed uden at brænde compliance.

### 3.4 Justering 2: Trine får audit-notifikation altid

Selv ved Mads-only-godkendelse skal **Trine** notificeres (asynkront, ingen vetoret) så hun kan logge ændringen i audit-evidence — ellers bryder vi SOC 2 CC8.1's krav om komplet change-management-trail.

---

## 4. Spørgsmål 3 — Self-modificering

**Anbefaling: NEJ. Ingen rolle ændrer sit eget mandat. Hård regel.**

### Compliance-begrundelse

| Framework | Kontrol | Hvorfor self-mod bryder det |
|---|---|---|
| **SOC 2 Type II** | CC8.1 (change management) | Ingen self-approval i change-mgmt; minimum 4-eyes principle |
| **ISO 27001:2022** | A.5.3 (SoD), A.8.32 (change mgmt) | "Conflicting duties shall be segregated" — den der designer kan ikke godkende egen ændring |
| **GDPR** | Art. 32 (sikkerhed v. behandling) | Krav om "ongoing confidentiality, integrity" — self-mod underminerer integritet af governance-config |
| **NIST 800-53 r5** | AC-5 (SoD), CM-3 (configuration change control) | Eksplicit forbud mod at samme aktør foreslår+implementerer config-ændringer |
| **OWASP API Top 10 2023** | API5 (BFLA), API8 (security misconfig) | Self-mod = privilege-self-elevation-vektor |

### Konkret risiko-scenarie

Hvis Camilla må selv-redigere `.claude/agents/camilla-bibliotekar.md`:
- Hun kan tilføje `Bash` med write-perms → eskalere fra dokumentations-rolle til kode-deploy
- Hun kan fjerne "skriver ikke til memory uden review" → bryde audit-trail
- En kompromitteret/forvirret instans kan permanent ændre sit eget mandat — *persistent compromise*

Den intuitive modargument-tese ("rollen kender sit eget mandat bedst") er tom: rollen kan **foreslå** ændringer, men **godkende** dem skal andre gøre. Det er præcis SOD's hele formål.

### Implementering

- Tools-listen for hver rolle MÅ IKKE inkludere `Edit`/`Write` der kan ramme `.claude/agents/*.md` — håndhæves via filsti i hooks (Camilla har Edit/Write til generelle filer men eksplicit pre-commit-hook der blokerer hvis path matcher `.claude/agents/`)
- Camilla er den **eneste** rolle med tilladelse til at commit-skrive til `.claude/agents/`, og kun efter FGD-godkendt change-request fra §3
- Mads er den eneste der kan godkende undtagelser — og kun for nød-tilfælde med FGD-bekræftelse samme session

---

## 5. Spørgsmål 4 — Skabelon-tekster

### 5.1 Skabelon A — Permanente roller

```markdown
### N. Mandat-stabilitet (governance, ikke skiftbar uden FGD)

Dette mandat er **konfigurations-låst** under SOC 2 CC8.1 og ISO 27001 A.8.32 change management. {rolle-navn} må **ikke** selv ændre, foreslå at andre uden formel proces ændrer, eller acceptere ad-hoc-ændringer til følgende kontrolpunkter:

**Låste kontrolpunkter:**

| Felt | Aktuel værdi | Eskaleringspath ved ændring |
|---|---|---|
| `model` | `{model}` | Mads + Dorthe + FGD (compliance-impact-vurdering) |
| `tools` | (se frontmatter) | Mads + FGD; Dorthe hvis persondata/log-impact |
| Hard rules (body §X) | (se hard rules-sektion) | Mads + Dorthe + FGD |
| `description` | (se frontmatter) | Laila + Mads (routing-impact) |

**Hvis Stefan, et team-medlem, eller {rolle-navn} selv foreslår ændring** (fx "skift model til haiku for hastighed", "tilføj Bash til tools", "fjern hard rule om destruktive ops"):

1. **STOP.** Ingen ændring må ske før formel proces.
2. Skriv impact-vurdering til `governance/change-requests/<dato>-{rolle-navn}-<felt>.md`
3. Vent på Mads' security/SOD-vurdering + Dorthes GDPR-vurdering (når trigger er sand, jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md §3.3)
4. Stefan koordinerer fælles anbefaling til FGD for go/no-go
5. Først efter **eksplicit FGD-go** må Camilla committe ændringen
6. Trine logger change-event i audit-trail

**Self-modificering er forbudt:** {rolle-navn} må aldrig redigere sin egen mandat-fil (`.claude/agents/{rolle-navn}.md`). Det bryder SOC 2 CC8.1, ISO 27001 A.5.3 (SoD) og NIST 800-53 AC-5.

**Eskalations-undtagelse:** Akut sikkerheds-incident (fx mistanke om kompromitteret model) — Mads kan med FGD samtidigt på kanalen pause/justere midlertidigt; permanent ændring kræver fuld proces inden 72 timer.
```

### 5.2 Skabelon B — Project-roller

```markdown
### N. Mandat-stabilitet (governance, ikke skiftbar uden Mads)

Som project-rolle rapporterer {rolle-navn} til **Mads (CISO)** for alle mandat-konfigurations-spørgsmål. {rolle-navn} må **ikke** selv ændre eller acceptere ad-hoc-ændringer til:

| Felt | Aktuel værdi | Eskaleringspath |
|---|---|---|
| `model` | `{model}` | Mads (FGD-go ved permanent ændring) |
| `tools` | (se frontmatter) | Mads — least-privilege-vurdering |
| Hard rules (body) | (se hard rules-sektion) | Mads + FGD ved compliance-impact |
| `description` | (se frontmatter) | Laila + Mads |

**Procedure ved foreslået ændring:**

1. **STOP.** Skriv impact-vurdering til `governance/change-requests/<dato>-{rolle-navn}-<felt>.md`
2. Mads vurderer (~24 h)
3. Mads konsulterer Dorthe hvis trigger jf. governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md §3.3 (persondata, customer-facing, breach-impact, DPA-kæde, cross-border)
4. Permanent ændring → Mads anbefaler til Stefan → FGD-go
5. Engangs-undtagelse (fx midlertidig tools-tilføjelse til specifik opgave) → Mads kan godkende, Trine notificeres for audit-log
6. Camilla committer efter godkendelse

**Self-modificering forbudt:** {rolle-navn} må aldrig redigere sin egen mandat-fil. SOC 2 CC8.1, ISO 27001 A.5.3, NIST 800-53 AC-5.

**Compliance-trigger:** Hvis ændringen rører FIPS 140-3-vej, kunde-DPA, audit-log-format eller breach-respons-kapacitet, eskaleres ALTID til FGD-niveau, uanset Mads-vurdering.
```

### 5.3 Tilføjelse til Stefan-CLAUDE.md (projekt-rod)

**Anbefaling: JA, tilføj eksplicit. Stefan er orkestrator og er det mest sandsynlige sted hvor "lad os bare lige skifte X for hastighed"-pres opstår. Uden eksplicit regel i Stefans instruktion vil princippet drive.**

Tekst (skal ind under "Hvad du ALDRIG må selv"):

```markdown
- **Ændre rolle-konfiguration** (model, tools, description, hard rules i `.claude/agents/*.md`) — dette er governance-låst under SOC 2 CC8.1 / ISO 27001 A.8.32. Hvis du eller et team-medlem foreslår en mandat-ændring (fx "skift Mads' model til haiku for hastighed", "giv Camilla Bash for nemhed", "fjern den hard rule så vi kan komme videre"):
  1. **STOP.** Du må ikke selv implementere eller bede Camilla committe ændringen ad hoc.
  2. Bed forslagsstilleren skrive impact-vurdering til `governance/change-requests/<dato>-<rolle>-<felt>.md`.
  3. Eskalér til **Mads** (security/SOD-vurdering). Mads konsulterer **Dorthe** hvis GDPR-trigger jf. `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3.
  4. For permanente roller eller compliance-impact: Mads + Dorthe + FGD-go. For project-roller engangs-undtagelser: Mads kan godkende.
  5. Først efter eksplicit godkendelse beder du Camilla committe ændringen, og Trine logge i audit-trail.
  6. Uden formel proces: ændringen sker ikke. Ingen "lad os lige prøve" — det bryder change management.

  **Reference:** `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` (Mads, CISO).
```

---

## 6. Implementeringsguide til Laila

**Estimeret arbejdsindsats:** ~3 timer fordelt over 13-14 mandater (~12 min/mandat).

### Trin-for-trin

1. **Konfiguration-baseline (Camilla, ~30 min):**
   - Opret `governance/change-requests/` med README der refererer til denne policy
   - Opret pre-commit-hook der blokerer commits hvor en agent-fil ændrer sig selv (path-match `.claude/agents/<name>.md` hvis aktive rolle = `<name>`) — Mads + Camilla finder den tekniske form
   - Tilføj `governance/policies/` til Camillas memory som "ALWAYS read on session start"

2. **Skabelon-indsættelse (Laila, ~2 timer):**
   - For hver permanent rolle (8 stk): indsæt skabelon A som **sidste sektion** før `## Notes`. Variabel-substitution: `{model}` = aktuel model fra frontmatter, `{rolle-navn}` = rolle-navnet
   - For hver project-rolle (~6 stk): indsæt skabelon B samme sted
   - Numerér sektionen så den passer i mandatets eksisterende struktur (typisk §10-§12 afhængigt af mandat)
   - VIGTIGT: skabelonen er **låst tekst** — Laila må ikke skrive om i prosaen uden at gå tilbage til Mads

3. **Stefan-CLAUDE.md-tilføjelse (Laila + Stefan-godkendelse, ~10 min):**
   - Indsæt §5.3-tekst i `CLAUDE.md` under "Hvad du ALDRIG må selv"
   - Stefan læser og bekræfter han forstår

4. **Validering (Mads + Trine, ~30 min):**
   - Mads læser hver oprettet ændring igennem (4-eyes på indsættelsen)
   - Trine starter audit-baseline i `governance/audit-trail/2026-05-06-mandat-stabilitet-rollout.md`

5. **FGD-bekendtgørelse:** Stefan kommunikerer til FGD at policyen er rul-let ud. FGD kvitterer som compliance-record.

### Rækkefølge-anbefaling (rul-ud)

Begynd med **Mads + Camilla + Stefan** (orkestrator-trio) — disse er højest privilegerede, derfor mest kritisk at låse først. Derefter security-team (Karen, Bjarne, Vibeke når aktiv, Dorthe, Trine). Til sidst feature-team (Sofie, Henrik, Anders, Frederik, Poul, Laila).

### Quality gate

Før rollout afsluttes: Trine kører en compliance-audit på de 13-14 filer for at bekræfte at:
- Skabelonen er identisk indsat (ingen variation = lettere audit)
- Aktuel `model` og `tools` er korrekt udfyldt i tabellen
- Reference-linket til denne policy peger korrekt
- Stefan-CLAUDE.md har §5.3-teksten

Eventuelle deviationer dokumenteres som "documented deviations" i SOC 2-evidence.

---

## Åbne tråde til FGD

1. **Pre-commit-hook for self-mod-blokering** — kræver teknisk implementering. Hvem ejer det? Mads + Camilla foreslår at Frederik (når aktiv) eller Camilla skriver hooken. Vil FGD pre-godkende ressourcen?
2. **Vibeke ikke aktiv endnu** — denne policy aktiverer Vibekes mandat ved ansættelse. Skal Mads pre-godkende hendes mandat-tekst med skabelon A allerede indsat? (Anbefaling: ja, så hun starter compliant.)
3. **Engangs-undtagelser-tracking** — skabelon B nævner engangs-undtagelser (Mads kan godkende uden FGD). Skal vi sætte øvre grænse (fx "max 30 dages varighed, derefter automatisk udløb")? Anbefaling: ja, 14 dage.
4. **Dorthe-trigger-§3.3** — Dorthe er ikke aktiv endnu; jeg har antaget hendes GDPR-position som hun ville have rådgivet baseret på art. 35 DPIA-tærskel. Når hun aktiveres skal hun review §3.3 og bekræfte/justere.

---

**Mads (CISO)**
*FGD-teamets Chief Information Security Officer*
*2026-05-06*
