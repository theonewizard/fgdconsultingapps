# Stefan-mandat: Formelt governance-dokument

**Laila's rapport til FGD**, 2026-05-08

---

## Hvad er skrevet

Et formelt mandat-dokument for Stefan (FGD's AI-organisator) som parallel til eksisterende mandater for Laila, Poul, Camilla, Mads, og fremtidigt Dorthe + Trine.

**Fil**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-08-stefan-organisator-mandat.md`

Formål: **Single source of truth** for Stefans rolle-definition, tools, model, governance-regler, og mandat-stabilitet — samme struktur som alle andre permanente roller.

---

## Nøgle-valg

### 1. Model: `opus`

Stefan kører på `claude-opus-4-7` (FGDs valg af session-model). Frontmatter specificerer `opus` så orchestrator-tooling ved at Stefan er high-reasoning-capacity.

**Grund**: Stefan udfører orkestrering, afvejning af behov, parallelt-koordinering. Det kræver nuancering. `haiku` ville være for svag til kompleks delegering; `sonnet` ville være overkill men acceptabelt. `opus` matcher Camilla + Poul (andre høj-kapacitets-roller).

### 2. Tools: `Read, Glob, Grep, Bash, TodoWrite, Agent`

- **Read** — læs alle filer (Team Inbox, memory, mandater, analyser, kontekst)
- **Glob** — søg efter filpatterns (`*.md`, `projects/*/`, osv.)
- **Grep** — søg efter indhold (role-overlap, references, status)
- **Bash** — **kun** `git status`, `git log`, `ls` (read-only git; Camilla ejer commits)
- **TodoWrite** — tracke fremdrift, dependencies, prioriteter
- **Agent** — delegér til Laila, Poul, Camilla, Mads, Dorthe, Trine, project-roller

**IKKE inkluderet:**
- **Write, Edit** — Stefan må ikke skrive produktionskode eller dokumentation selv
- **WebSearch, WebFetch** — Poul gør research
- **Bash (write)** — Camilla ejer git + database; Stefan kun `git status`/`git log`

### 3. Governance-struktur: Identisk med Mads + Camilla + Poul + Laila

**Mandat-stabilitet-sektion** omfatter:
- **Låste kontrolpunkter**: model, tools, hard rules, description — alle kræver formel ændring
- **Eskalerings-proces**: impact-vurdering → Mads + Dorthe vurderer → Stefan koordinerer til FGD → kun FGD kan godkende
- **Self-modification forbud**: Stefan kan ikke ændre sin egen `.claude/agents/stefan-organisator.md` (SOC 2 CC8.1, ISO 27001 A.5.3 SoD, NIST 800-53 AC-5)
- **Eskalerings-undtagelse**: Akut incident (orkestrering-deadlock, kompromitteret rolle) — Mads + FGD kan midlertidigt justere; permanent ændring kræver fuld proces inden 72 timer

**Dette sikrer**, at Stefan selv ikke kan ændre sit mandat ad-hoc, og at alle ændringer går gennem governance-proces som alle andre permanente roller.

---

## Konsistens med eksisterende mandater

| Element | Mads | Camilla | Poul | Laila | Stefan |
|---|---|---|---|---|---|
| Frontmatter | ja | ja | ja | ja | **ja** |
| Mission-sektion | ja | ja | ja | ja | **ja** |
| Delegerings-hierarki | ja (via hard rules) | ja (table) | ja (table) | ja (table) | **ja** |
| Hard rules | ja | ja | ja | ja | **ja** |
| Ansvarsfordeling-tabel | ja | ja | ja | ja | **ja** |
| Skills-sektion | ja | ja | ja | ja | **ja** |
| Ansvarsområder pr. fase | ja | ja | ja | ja | **ja** |
| Anti-patterns | ja | ja | ja | ja | **ja** |
| Hand-off-aftaler | ja | ja | ja | ja | **ja** |
| Hilsen-skabelon | ja | ja | ja | ja | **ja** |
| **Mandat-stabilitet** (med låste kontrolpunkter, self-modification forbud, eskalerings-proces) | **ja** | **ja** | **ja** | **ja** | **ja** |

Stefan-mandatet følger **identisk struktur** som de andre permanente roller — ingen ad-hoc-undtagelser.

---

## Vigtige governance-detaljer i mandatet

1. **§"Hvad Stefan ALDRIG må selv"** — explicit forbud mod:
   - Ændre rolle-konfiguration
   - Skrive til memory (Camilla ejer)
   - Committe (Camilla ejer git)
   - Analysere kilder selv (Poul gør det)
   - Designe roller (Laila gør det)

2. **§"Standard-workflow"** — Step-by-step flow som FGD kan reference når han sender opgaver:
   - Læs Team Inbox
   - Afklar behov (evt. Poul)
   - Tjek eksisterende roller (overlap?)
   - Design eller hverv (Laila hvis ny)
   - Delegér (Agent)
   - Saml resultater (Min Inbox/)
   - Rapport til FGD

3. **§"Delegerings-hierarki"** — tabel som Stefan selv refererer når han vurderer "hvem skal jeg kalde?"

4. **§"Governance-integration"** — requirement at Mads har vurderet compliance-impact før Stefan præsenterer feature til FGD.

5. **§"Mandat-stabilitet"** — **KRITISK**: Stefan selv **kan ikke** ændre modellen fra `opus` til `haiku`, fjerne `Agent` fra tools, eller droppe hard rules uden fuldt governance-flow. Det sikrer at Stefan ikke kan påvirke sin egen rolle-konfiguration.

---

## Hvad FGD skal godkende

Før Camilla kan flytte dette til `.claude/agents/stefan-organisator.md` og committe, skal FGD godkende:

1. **Model-valg**: `opus` er korrekt, eller ønskes `sonnet`/`haiku`?
2. **Tools-valg**: Skal Stefan have **Write** eller **Edit** hvis han selv skal skrive status-noter? (Nuværende design: Stefan skriver kun korte koordinerings-noter i `Min Inbox/` + `Team/`, ikke hele dokumenter → **ikke** nødvendig)
3. **Governance-struktur**: Accepterer FGD at Stefan ikke kan ændre sit eget mandat? (Ja, det er intenderet for SOC 2 CC8.1/ISO 27001 A.5.3 SoD)
4. **Mandat-indhold**: Er der vigtige Stefan-opgaver/regler der mangler i §"Standard-workflow" eller §"Delegerings-hierarki"?

---

## Næste skridt

1. **FGD godkender** (eller justerer) mandatet.
2. **Camilla** forflytter filen fra `Min Inbox/_crosscutting/` til `.claude/agents/stefan-organisator.md` og committer med reference til FGD's godkendelse.
3. **Stefan** blir derefter officialiseret som permanent rolle med formelt mandat parallelt til Laila, Poul, Camilla, Mads.
4. (Fremtidigt: samme format når Dorthe + Trine + eventuelle andre permanente roller aktiveres)

---

**Laila**  
2026-05-08
