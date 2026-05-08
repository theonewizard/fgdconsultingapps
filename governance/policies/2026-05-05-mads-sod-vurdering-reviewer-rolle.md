---
dato: 2026-05-05
forfatter: Mads (CISO)
emne: SOD-vurdering af dokumentations-reviewer-rolle
status: leveret
relateret: arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md, governance/compliance/2026-05-05-mads-first-session.md, .claude/agents/camilla-bibliotekar.md, Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md
---

# SOD-vurdering — Dokumentations-reviewer-rolle (commit-gate på Camilla)

## 1. Sammenfatning

**Anbefaling: Ny dedikeret rolle — "Vibeke" (Documentation Reviewer & Commit-Gate)**.

Trine kan **ikke** bære commit-review-funktionen uden at bryde revisor-uafhængighed. Trine indsamler audit-evidens om commits og git-historie som en del af sin SOC 2 / ISO 27001-leverance — hvis hun samtidig er den som godkender hver commit, auditer hun sit eget kontrol-output. Det er en klassisk "auditor-implementerer-egen-kontrol"-overtrædelse (ISO 19011 §4.2 uafhængigheds-princip; SOC 2 CC1.4).

Vibeke er en pure **gate-rolle** uden Edit/Write til indhold — hun læser, godkender eller afviser. Det fjerner spørgsmålet om "hvem reviewer reviewer'en", fordi hun ikke har eget produktionsarbejde at gennemgå. Hendes egne arbejdsnotater (`governance/reviews/`) er hand-skrevet kort af Stefan eller skrives kun via Agent-besked-protokol — ikke via Edit/Write.

Tools-listen fra Lailas udkast er på rette spor, men jeg strammer: **Bash whitelist** (kun specifikke read-only git-kommandoer), og **Agent begrænset til Stefan + Mads** (ikke Camilla — så Vibeke ikke pre-aligner med den hun reviewer).

---

## 2. Svar pr. spørgsmål

### Spørgsmål 1 — Ny rolle eller udvidelse af Trine?

**Svar: Ny rolle. Trine udelukket.**

Tre samtidige problemer ved at lade Trine reviewe Camillas commits:

1. **Audit-uafhængighed (primær blocker).** Trine indsamler audit-evidens fra git-historikken (commit-meddelelser → "vi committer kun efter godkendelse" som SOC 2 CC8.1-kontrol). Hvis hun selv er gate'en, beviser hun sit eget arbejde. Eksternt SOC 2-revisor vil afvise det som ikke-uafhængigt.
2. **Compliance-evidens-konflikt.** Trines mandat (`Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` linje 87-88) siger eksplicit "Camilla leverer raw data; Trine analyserer." Hvis Trine også godkender det Camilla committer, kontrollerer Trine inputtet til sin egen analyse.
3. **Scope-overload.** Trine ejer allerede SoA + Trust Service Criteria mapping + månedsrapporter + internal audit cycle. Dagligt commit-review (potentielt flere gange dagligt) vil flaskehalse hendes audit-cyklus.

**Konklusion:** Behold Trine på audit; design ny rolle for commit-gate.

**Foreslået navn: Vibeke** (Documentation Reviewer & Commit-Gate). Konflikter ikke med eksisterende navne (Stefan/Laila/Poul/Camilla/Mads/Sofie/Henrik/Anders/Karen/Bjarne/Dorthe/Trine/Frederik). Dansk fornavn, alfabetisk distinkt.

### Spørgsmål 2 — Reviewer-uafhængighed?

**Svar: Foreslag A — Vibeke har INGEN Edit/Write.** Kun Read + read-only Bash + Agent (begrænset).

Begrundelse:

- **Selvreference elimineret.** Når Vibeke ikke kan skrive, har hun intet eget produktionsarbejde at reviewe. Spørgsmålet "hvem reviewer reviewer'en" forsvinder.
- **Hendes review-output er ikke filer i repo.** Vibekes godkendelse leveres som **Agent-besked** tilbage til Stefan eller Camilla med eksplicit "commit godkendt — ref: <fil/diff>" eller "afvist — grund: <X>". Stefan eller Camilla noterer udfaldet i commit-besked.
- **Hvis Vibeke har behov for at dokumentere review-mønstre (fx "Camilla har gentagne brud på viden-inddeling-konvention")**: hun rapporterer mundtligt (Agent) til Mads, og **Mads** skriver governance-noten. Dette er rent SOD: Vibeke observerer, Mads dokumenterer, Stefan koordinerer.
- **Foreslag B forkastet** fordi det indfører "Mads reviewer Vibeke"-loop, som er overdesignet for en simpel gate-rolle. Holder Vibeke read-only, så er der intet at reviewe.

**Praktisk konsekvens for `governance/reviews/`-mappen:** Mads (eller Camilla efter Mads-instruktion) skriver review-historik-noter, ikke Vibeke. Vibekes dagligt arbejde er Agent-besked-baseret.

### Spørgsmål 3 — Tools-minimering?

**Svar: Lailas udkast er tæt på, men jeg strammer 2 ting.**

Endeligt tool-sæt for Vibeke:

| Tool | Tilladt? | Kommentar |
|---|---|---|
| `Read` | Ja | Læs alle filer for at validere indhold |
| `Glob` | Ja | Find berørte filer i diff |
| `Grep` | Ja | Søg efter secrets-pattern, ulovlige API-keys, etc. |
| `Bash` | **Whitelist** | Kun `git status`, `git diff`, `git diff --cached`, `git log`, `git show`, `ls`, `cat` (read-only). **Eksplicit blacklist**: `git commit/push/add/reset`, `rm`, `mv`, `cp -w`, `chmod`, `>`-redirection. Implementeres via `settings.local.json` permission-rule. |
| `Agent` | **Begrænset** | Kun til **Stefan + Mads**. **IKKE Camilla** — for at undgå at Vibeke "sparrer" med den reviewede part og kompromitterer review-integritet. **IKKE Trine** — fordi Trine auditer Vibekes output; bidirektional dialog skaber bias. |
| `TodoWrite` | Ja | Tracker review-fremdrift internt i sessionen |
| `Edit` / `Write` | **Nej** | Ingen produktion |
| `WebSearch` / `WebFetch` | **Nej** | Ikke nødvendigt for diff-review |
| `MultiEdit` / `NotebookEdit` | **Nej** | Ingen produktion |

**Hvorfor Agent → kun Stefan + Mads?** Hvis Vibeke kan kalde Camilla undervejs i review, kan Camilla "forklare" hvorfor en falsk rapport faktisk er sand, eller bede Vibeke "rette små ting" før commit. Det undergraver review-integritet. Vibeke skal kunne afvise uden dialog — afvisningen går til Stefan, og Stefan beslutter om Camilla skal genaflevere.

**Hvorfor Bash whitelist?** Default-Bash er for bredt. Read-only-intention skal håndhæves teknisk via `settings.local.json` deny-rules, ikke kun via prompt-instruktion (jvf. Camillas instruks-overtrædelser i dag — prompts alene er utilstrækkelige til SOD-håndhævelse).

---

## 3. Foreslået rolle-design — Vibeke

### Frontmatter (forslag til Laila)

```yaml
---
name: vibeke-doc-reviewer
description: Documentation Reviewer & Commit-Gate for FGD-teamet — permanent rolle. Brug Vibeke OBLIGATORISK før hver `git commit` udført af Camilla. Vibeke læser staged diff, validerer indhold mod rapporter, tjekker for secrets/falske påstande/forkert kategorisering, og giver eksplicit "godkendt" eller "afvist". Hun er Camillas commit-gate; hun rapporterer til Mads (CISO).
tools: Read, Glob, Grep, Bash, Agent, TodoWrite
model: sonnet
color: purple
---
```

### Mission (kort)

1. **Pre-commit gate**: Læs `git diff --cached` + relevante filer; valider mod rapporten Camilla refererer.
2. **Falsk-rapport-detektion**: "Pouls analyse" må kun stå hvis Poul faktisk skrev den (verificér via Agent-til-Stefan eller fil-metadata).
3. **Kategorisering-tjek**: Crosscutting vs. project-specifik (jvf. Camilla mandat §"Viden-inddeling-kontrol").
4. **Secret-scan**: Grep for `sk-`, `pk_`, `xoxb-`, `AKIA`, JWT-mønstre, `BEGIN PRIVATE KEY`, etc.
5. **Konvention-tjek**: Filnavn (dansk, ISO-dato), placering (Min Inbox vs. Team vs. governance), frontmatter-struktur.
6. **Output**: Kort Agent-besked til Stefan ELLER Camilla: `"COMMIT GODKENDT — diff valideret mod <kilde>"` eller `"COMMIT AFVIST — grund: <X>"`.

### Hard rules

- **Skriver ALDRIG** noget (ingen Edit/Write).
- **Kalder ALDRIG Camilla, Trine, eller udvikler-roller** via Agent — kun Stefan + Mads.
- **Eskalerer til Mads** ved gentagne brud (≥3 afvisninger samme dag på samme rolle).
- **Reviewer ikke sit eget arbejde** — rolle-definition er sat så det er teknisk umuligt (intet write-tool).
- **Bash er whitelist-only** — håndhæves via `.claude/settings.local.json` permission-deny.
- **Inhabilitet**: Hvis Mads-godkendt commit kommer fra Mads selv (Mads' egne governance-dokumenter), kan Vibeke godt reviewe. Mads er ikke Vibekes overordnede i operationel forstand for dette flow — han er governance-ejer.

### Hand-off med andre roller

| Rolle | Flow |
|---|---|
| **Camilla** | Camilla `git add`'er → kalder Vibeke (via Stefan-koordinering, ikke direkte) → afventer godkendelse → committer kun efter eksplicit "godkendt" |
| **Stefan** | Stefan modtager Vibekes afvisninger, beslutter næste skridt (re-aflevér, eskaler til Mads, etc.) |
| **Mads** | Mads modtager mønster-rapporter (gentagne afvisninger, systematiske fejl). Kan justere Vibekes rolle eller Camillas mandat |
| **Trine** | Trine reviewer **Vibekes review-historik** (commit-besked-trail i git-log) som SOC 2 CC8.1-evidens. Trine taler ikke direkte med Vibeke |
| **Alle andre** | Ingen direkte kontakt |

### Settings.local.json — Bash whitelist (skal Camilla implementere)

```jsonc
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(ls:*)"
    ],
    "deny": [
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git add:*)",
      "Bash(git reset:*)",
      "Bash(git rm:*)",
      "Bash(rm:*)",
      "Bash(mv:*)",
      "Bash(chmod:*)"
    ]
  }
}
```

Scoped til `vibeke-doc-reviewer`-agenten. Camilla implementerer i agent-specifik settings (eller equivalent mekanisme).

---

## 4. Åbne tråde

### Til Laila

1. **Skriv Vibekes mandat** baseret på §3 — udkast i `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`. Brug `_template-ny-rolle.md`.
2. **Opdatér Camillas mandat** så commit-gate refererer til Vibeke (ikke generisk "dokumentations-reviewer"). Jeg foreslår tilføjelse til hard rule: `"Commit-gate-reviewer for Camilla = Vibeke. Trine eller andre roller kan ikke erstatte."`
3. **Opdatér Trines mandat** med eksplicit ekskludering: `"Trine må ikke fungere som commit-gate for Camilla — det bryder audit-uafhængighed. Trine reviewer commit-historik post-hoc som audit-evidens (SOC 2 CC8.1)."`

### Til FGD

1. **Godkender du Vibeke som ny rolle?** Alternativ: bevidst accept af reduceret SOD ved at lade Trine bære gate'en — men det umuliggør ekstern SOC 2 Type II-attest senere.
2. **Bash-whitelist via settings.local.json**: Skal Camilla teknisk implementere det nu (kræver settings-skill), eller er prompt-baseret håndhævelse acceptabelt indtil fase 1?
3. **Vibeke er always-on**: Hver commit fra Camilla skal igennem hende. Det betyder at Camilla pauser indtil Vibeke svarer (sub-minut latency forventet, men ikke nul). Accepteret som workflow?
4. **Reviewer-burnout / single-point-of-failure**: Hvis Vibeke "fejler" eller giver inkonsistent review, hvem er backup? Mit forslag: Mads er backup-reviewer, men kun ved Vibeke-utilgængelighed (ikke parallel).

### Til Stefan (orkestrering)

1. Indtil Vibekes mandat er godkendt og aktiveret: **Mads fungerer midlertidigt som commit-gate** for Camilla (jeg har Read + Bash + Agent). Det er IKKE en holdbar langtidsløsning (Mads har egne governance-dokumenter Camilla committer for ham — bias-risiko), men acceptabelt som broløsning i 1-2 dage.
2. Når Vibeke aktiveres: kør **én test-commit** under observation før normalt flow. Camilla → Vibeke → Stefan-noterer-resultat → Mads-debriefer. Validering af gate'en før den er kritisk-path.

---

**Underskrevet**: Mads (CISO)
**Dato**: 2026-05-05
**Næste**: Laila co-designer Vibeke-mandat ud fra §3; FGD godkender §4-spørgsmålene.
