---
date: 2026-05-08
reviewer: mads-ciso
subject: Final 4-eyes review — Camilla v2-mandat
source: Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md
verdict: GO
---

# Final 4-eyes review: Camilla v2-mandat

## Verificering af tidligere kritiske fund

### Fund 1 — Grep-artefakt-gemning obligatorisk med navngiven filsti

**STATUS: LØST**

§1 Hard rule pkt. 3 specificerer eksplicit:

> "Grep-output **skal** gemmes til: `Team/Camilla/audit-<dato>-grep-result.txt`. Filstien skal fremgå eksplicit i rapporten. Ikke optionelt."

Filsti er navngivet, format er angivet, "ikke optionelt" er eksplicit markeret. Verificerings-checklist (§Verificerings-checklist) gentager kravet: "Grep-output gemmes og inkluderes i rapport? (filsti)". Dobbeltforankret.

---

### Fund 2 — Bash-whitelist: `git reset` splittet, `git stash` forbudt, autoritet = Vibeke

**STATUS: LØST**

§6 Bash whitelist specificerer:

**Tilladt:**
- `git reset HEAD <file>` (unstage enkeltfil) — eksplicit tilladt, enkeltfil-scope

**Forbudt:**
- `git reset --hard`, `git reset --soft` (forbudt helt) — begge varianter eksplicit forbudt
- `git stash` (forbudt helt — kan skjule staged ændringer for Vibeke-review) — forbudt med begrundelse

Autoritet er Vibeke: §5 og §6 refererer begge konsekvent "Vibeke-godkendelse" som commit-gate. Splittet korrekt, stash forbudt, autoritet tydelig.

---

### Fund 3 — Henrik/Karen fjernet

**STATUS: LØST**

Gennemlæsning af hele dokumentet: ingen forekomster af "Henrik" eller "Karen" i mandatteksten. Database-samarbejdsrollen er anonymiseret til "database-design-rolle (project-rolle)" i §Mission og §Samarbejde. Security-rollen er tilsvarende "security-rolle" (§Mission). Navne er fjernet.

---

### Fund 4 — Neon-MCP og memory-arbejdsgang tilføjet

**STATUS: LØST**

§Database-arbejde indeholder eksplicit Neon MCP-flow:

> "Du arbejder med Neon Postgres-skemaer via MCP-værktøjer (`mcp__Neon__*`-værktøjer: list_projects, describe_table_schema, prepare_database_migration, run_sql, osv.)."

Firetrins-flow (Forberedelse → Planlægning → Udfører → Rapport) er specificeret.

§Memory-arbejdsgang er fuldt udbygget med filstruktur, memory-typer, hvad der ikke skrives, og arbejdsgang ved opdatering (5 trin). Begge mangler fra tidligere version er adresseret.

---

## Verificering af tidligere minor fund

### Fund 5 — §7 konsolideret med Mandat-stabilitet

**STATUS: LØST**

§7 Model-stabilitet er en kort, selvstændig sektion:

> "Model-skift følger fuld governance-proces under §Mandat-stabilitet. Ingen model-ændring uden Mads + Dorthe + FGD-godkendelse."

Den uddelegerer til §Mandat-stabilitet i stedet for at gentage content. Konsolideret uden tab af governance-dækning.

---

### Fund 6 — Grep scope-regel tilføjet (repo-rod)

**STATUS: LØST**

§1 Hard rule pkt. 3 specificerer eksplicit:

> "Grep skal køre fra repo-roden (`grep -rn ... .`), ikke på udvalgt subscope"

Reglen er eksplicit og operationel.

---

## Nye fund

### Nyt fund A — Memory GDPR-regel: "rigtige mennesker" vs. AI-roller

**SEVERITY: Minor — acceptabel, men bør noteres**

§Hvad du IKKE skriver i memory indeholder:

> "AI-roller og agenter er ikke omfattet af dette forbud (de er ikke fysiske personer under GDPR)."

Dette er korrekt juridisk (GDPR art. 4(1) definerer "person" som fysisk person). Det er dog en substantiel compliance-afgørelse nedfældet direkte i et operationelt mandat uden eksplicit Dorthe-vurdering. Camilla kan i god tro gemme information om agent-adfærd der potentielt afspejler FGDs instruktioner og arbejdsmønstre — indirekte persondata.

**Anbefaling**: Dorthe bør bekræfte at denne afgørelse er korrekt i konteksten af FGDs samlede memory-arkitektur. Ikke blokerende for GO, men Dorthe-bekræftelse bør indhentes inden Fase 1.

### Nyt fund B — §5 Git-gate refererer kun "Agent-kald" til Vibeke

**SEVERITY: Minor — ingen bloker**

§5 formulerer:

> "Du må **IKKE** `git commit` uden **Vibeke's eksplicit godkendelse** (via Agent-kald)."

"Via Agent-kald" er teknisk korrekt for nuværende workflow, men kræver at Vibeke-rollen altid er tilgængelig via Agent. Hvis Vibeke-rollen ikke er aktiveret i en session, er det uklart om Camilla skal blokere eller eskalere til Stefan. Vibeke-rollens scope er ikke defineret i dette mandat.

**Anbefaling**: Tilføj én linje: "Hvis Vibeke ikke er tilgængelig: eskalér til Stefan, commit sker ikke." Ikke blokerende.

### Nyt fund C — §Mandat-stabilitet: "Hard rules (body §1-7)" refererer eksakt 7 sektioner

**SEVERITY: Minimal — selvdokumenterende**

Tabellen lister "Hard rules (body §1-7)" som låst kontrolpunkt. Mandatet har aktuelt §1-7 under Hard rules. Hvis en ny §8 tilføjes uden at opdatere tabellen, er den teknisk ubeskyttet. Formulering som "Hard rules (body §Hard rules)" ville være mere robust. Ikke blokerende — eksisterende §1-7 er korrekt og change-management-processen er tilstrækkelig til at fange det.

---

## Samlet vurdering

**VERDICT: GO**

Alle 6 tidligere fund (4 kritiske + 2 minor) er korrekt løst. Dokumentet er operationelt og strukturelt sound. Tre nye fund identificeret — alle minor, ingen blokerende.

Prioriterede efterfølgende handlinger (ikke GO-blokerende):

1. **Dorthe-bekræftelse** af memory GDPR-afgørelse (Nyt fund A) — inden Fase 1
2. **Vibeke-fallback** ved manglende tilgængelighed (Nyt fund B) — næste mandat-revision
3. **§Mandat-stabilitet tabel-formulering** (Nyt fund C) — næste mandat-revision

Camilla v2-mandatet er klar til aktivering.

---

*Mads, CISO — 2026-05-08*
