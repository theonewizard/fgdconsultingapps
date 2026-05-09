---
dato: 2026-05-09
forfatter: Camilla
emne: Skill-assessment — vurdering af 6 installerede skills mod kerneopgave
status: intern (ikke commit-trigger)
trigger: Stefan-bestilling, FGD-initiativ
---

# Skill-assessment — Camillas kernekompetencer

## Sammenfatning

Af de 6 nævnte skills er **3 direkte styrkende** for mit arbejde
(`neon-postgres`, `context7`, `coding-standards`), **1 betinget relevant**
(`python-patterns`), og **2 har misvisende navne** og er reelt
ikke-relevante for min rolle (`yaml`, `devtools` — begge er
`@json-render/*`-specifikke, ikke generelle).

Verifikation: jeg har læst frontmatter + intro for hvert skill i
`/Users/fgd/.claude/skills/<skill>/SKILL.md`. Konklusioner herunder
er primær-bevis-baserede, ikke navne-gætte.

## Per-skill vurdering

### 1. `yaml` — IKKE relevant (navnet er misvisende)

**Hvad skillen faktisk er** (frontmatter, l. 2-3):
> "YAML wire format for json-render with streaming parser, prompt
> generation, and AI SDK transform. Use when working with
> @json-render/yaml, YAML-based spec streaming, yaml-spec/yaml-edit
> fences, or YAML prompt generation."

Skillen handler om `@json-render/yaml` (LLM-spec streaming format),
**ikke** generel YAML-syntax, CI/CD-config, frontmatter-validering
eller GitHub Actions. For mit arbejde med `.claude/agents/*.md`
frontmatter, AsyncAPI 3.x, OpenAPI 3.1, Spectral-linting og GitHub
Actions/CI er denne skill **ikke** brugbar.

**Vurdering**: Tilføjes IKKE til mit standardværktøj. For ægte YAML-arbejde
bruger jeg generel viden + Context7 til version-tjek af YAML-relaterede
værktøjer (Spectral, yaml-lint, GitHub Actions schema).

### 2. `neon-postgres` — STYRKER min DB-ekspertise direkte

**Hvad skillen faktisk er** (frontmatter):
> "Guides and best practices for working with Neon Serverless Postgres.
> Covers getting started, local development, choosing a connection
> method, Neon features, authentication, PostgREST-style data API,
> Neon CLI, and Neon's Platform API/SDKs."

Pointers til officiel `neon.com/docs/*.md`-content (kan hentes som
markdown). Dækker connection-methods, branching, Neon CLI, MCP-server,
Admin API.

**Vurdering**: TILFØJES til mit standardværktøj. Komplementerer mine
Neon-MCP-kald (`describe_table_schema`, `prepare_database_migration`,
`run_sql`) med authoritativ doc-reference. Særligt værdifuldt når jeg:
- vælger connection-driver til migrations vs. runtime
- planlægger branch-baserede migrations-tests
- skal forklare Neon-arkitektur til Stefan/database-design-rolle
- verificerer feature-tilgængelighed (RLS, auth, Data API) før
  migrations-plan

**Bemærk**: Erstatter ikke MCP-værktøjerne — supplerer dem med kontekst.

### 3. `context7` — STYRKER min version-verificering

**Hvad skillen faktisk er** (frontmatter):
> "Use when the user asks about any library, framework, SDK, API, CLI
> tool, or cloud service ... Prefer over web search or training-data
> recall, even for familiar libraries."

Workflow: `resolve-library-id` → `query-docs` med fokuseret topic.

**Vurdering**: TILFØJES (er reelt allerede i brug). Direkte aligned med
memory-feedback `feedback_version_verification.md`: alle analyser skal
verificere SDK/framework-versioner. Bruges i:
- Drizzle-ORM migrations-syntax (versions-skift mellem 0.30.x og 0.40.x)
- Spectral-rule-syntax for OpenAPI-linting
- Neon serverless-driver release-notes
- Postgres feature-flags (RLS, JSONB, generated columns)

### 4. `python-patterns` — BETINGET relevant

**Hvad skillen faktisk er** (frontmatter, oversat fra japansk):
> "Pythonic-idiomer, PEP 8-standarder, type hints, best practices til
> at bygge robuste, effektive og vedligeholdbare Python-applikationer."

Indhold er på japansk men dækker generelle Python-mønstre (EAFP, PEP 8,
type hints, list-comprehensions).

**Vurdering**: NEJ — ikke til mit standardværktøj. Min rolle skriver
ikke produktions-Python; jeg laver migrations-SQL, git-operationer og
markdown-dokumentation. Hvis jeg en dag skal lave et migrations-script
i Python (fx data-backfill), aktiverer jeg den ad-hoc. Sproget (japansk
indhold) er en mindre praktisk barriere men ikke blokerende.

### 5. `coding-standards` — STYRKER min review-kompetence

**Hvad skillen faktisk er** (frontmatter):
> "Baseline cross-project coding conventions for naming, readability,
> immutability, and code-quality review."

KISS, DRY, YAGNI, naming, immutability — generel kvalitets-baseline.

**Vurdering**: TILFØJES. Jeg er executor af git-commits og review-input
til Vibeke. At kunne pege på en delt baseline af coding-conventions
hjælper mig vurdere om en commit er pænt struktureret før staging
(navngivning, file-layout, redundans). Erstatter ikke domain-roller
(database-design-rolle ejer SQL-konventioner; security-rolle ejer
RLS-policies), men giver en fælles top-level reference.

### 6. `devtools` — IKKE relevant (navnet er misvisende)

**Hvad skillen faktisk er** (frontmatter):
> "Drop-in inspector panel for any json-render app. Use when the user
> wants to debug a generative UI, inspect the spec tree ... for
> @json-render/devtools."

Det er en json-render-specifik React/Vue/Svelte/Solid devtools-komponent
— **ikke** generelle udviklerværktøjer (browser-devtools, Postman, jq,
osv.).

**Vurdering**: Tilføjes IKKE. Ingen overlap med min rolle.

## Anbefaling — mit standardværktøj efter denne assessment

| Skill | Status | Brug |
|---|---|---|
| `neon-postgres` | TILFØJ | Authoritativ doc-reference til DB-arbejde |
| `context7` | BIBEHOLD | Version-verificering (allerede i brug) |
| `coding-standards` | TILFØJ | Review-baseline før staging |
| `python-patterns` | AD-HOC | Kun ved Python-scripts |
| `yaml` | AFVIS | Misvisende navn — `@json-render`-specifik |
| `devtools` | AFVIS | Misvisende navn — `@json-render`-specifik |

## Eskalering til Stefan

To navngivne skills (`yaml`, `devtools`) **lyder** generiske men er
reelt `@json-render/*`-specifikke. Hvis FGD/teamet planlægger at lade
roller aktivere disse på basis af navnet alene, vil det give forkerte
forventninger. Forslag: skill-inventaret (`2026-05-05-skill-inventar.md`)
udvides med kolonnen "faktisk scope" hvor navne er misvisende. Det er
en discoverability-policy-beslutning — Mads bør se det.

## Verifikations-spor

Primær-bevis hentet fra:
- `/Users/fgd/.claude/skills/yaml/SKILL.md` (l. 1-5)
- `/Users/fgd/.claude/skills/neon-postgres/SKILL.md` (l. 1-12)
- `/Users/fgd/.claude/skills/context7/SKILL.md` (l. 1-15)
- `/Users/fgd/.claude/skills/python-patterns/SKILL.md` (l. 1-10)
- `/Users/fgd/.claude/skills/coding-standards/SKILL.md` (l. 1-15)
- `/Users/fgd/.claude/skills/devtools/SKILL.md` (l. 1-10)

Ingen grep-audit nødvendig (ingen claims om "alle X fixet"); dette er
en vurderings-rapport, ikke en audit.
