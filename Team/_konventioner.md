# Team-konventioner

Disse konventioner gælder for alle roller i FGD-teamet (Stefan + alle .claude/agents/).

## Sprog

- **Dansk** i al kommunikation med FGD og i rolle-mandater (`.claude/agents/*.md` body, README'er, inbox-resuméer).
- **Engelsk** i tekniske felter: frontmatter-keys, file paths, tool-navne, kode, commit-beskeder *til kode-projekter* (men commit-beskeder i dette repo er på dansk — det er en intern arbejdsmappe).

## Filnavngivning

- Små bogstaver, bindestreg som separator: `videopipeline-analyse.md`.
- Datoer i ISO-format: `2026-05-05`.
- Inbox-filer prefixes altid med dato: `2026-05-05-emne.md`.
- Agent-filer: `<fornavn>-<rolle-kort>.md` (fx `laila-hr.md`, `mette-tester.md`).

## Mapper

```
/                                          ← projektrod
├── CLAUDE.md                               ← Stefans instrukser (loades automatisk)
├── .claude/agents/                         ← rolle-definitioner (committet, project-uafhængig)
├── Min Inbox/
│   ├── _crosscutting/                      ← permanente rolle-leverancer + crosscutting-rapporter
│   ├── projects/
│   │   └── vidensapp/                      ← project-specifik inbox (teamet → FGD pr. projekt)
│   └── .gitkeep
├── Team Inbox/
│   ├── _crosscutting/                      ← godkendte permanente rolle-mandater
│   ├── projects/
│   │   └── vidensapp/                      ← project-specifik inbox (FGD → teamet pr. projekt)
│   └── .gitkeep
├── Team/                                   ← rollernes arbejdsmapper
│   ├── Camilla/                            ← fast rolle (project-uafhængig)
│   ├── Laila/                              ← fast rolle (project-uafhængig)
│   ├── Mads/                               ← fast rolle (project-uafhængig)
│   ├── Poul/                               ← fast rolle (project-uafhængig, crosscutting analyser)
│   ├── projects/
│   │   └── vidensapp/                      ← project-specifikke arbejdsmapper
│   │       ├── Sofie/                      ← project-rolle: frontend
│   │       ├── Henrik/                     ← project-rolle: backend
│   │       ├── Anders/                     ← project-rolle: AI-pipeline
│   │       ├── Karen/                      ← project-rolle: security architect
│   │       ├── Bjarne/                     ← project-rolle: HSM-operatør
│   │       ├── Dorthe/                     ← project-rolle: DPO
│   │       ├── Trine/                      ← project-rolle: compliance-auditor
│   │       └── Frederik/                   ← project-rolle: pen-test-koordinator
│   └── _konventioner.md                    ← denne fil
├── governance/                             ← Mads + Poul: governance-policies, compliance, threat-models, design-docs (project-uafhængig)
└── arkiv/                                  ← arkiverede dokumenter (spejler hoved-katalog)
    ├── _crosscutting/                      ← arkiverede crosscutting-dokumenter
    ├── Min Inbox/
    └── projects/
        └── vidensapp/
            ├── Min Inbox/
            └── Team/Poul/
```

### Project-niveau

**Aktive projekter**: `projects/<projektname>/` (fx `projects/vidensapp/`)

**Faste roller** (Stefan, Laila, Poul, Camilla, Mads) forbliver på rod-niveau under `Team/` uden project-scope — de tjener på tværs af alle projekter.

**Project-roller** (Sofie, Henrik, Anders, Karen, Bjarne, Dorthe, Trine, Frederik) får hver sin mappe under `Team/projects/<projekt>/<rolle>/` når de er aktive for projektet.

## Commit-gate — obligatorisk review

**Hard rule**: Camilla må IKKE kalde `git commit` før dokumentations-reviewer har godkendt.

Workflow:

1. Camilla forberedier arbejdet: læser opgave, laver ændringer, stager med `git add <filer>` (OK uden godkendelse).
2. Camilla noterer kort til reviewer: "Klart til review — [kort beskrivelse af ændringer]"
3. Reviewer læser ændringerne (`git diff --cached` eller modified-filer), verificerer at indhold matcher forventet og rapporteret.
4. Reviewer godkender skriftligt: "Commit godkendt" (via Agent-besked eller note i `governance/reviews/`).
5. Camilla kommitter kun efter godkendelse.
6. Push kræver separat FGD-godkendelse.

**Sanktion**: Hvis Camilla kommitter uden reviewer-godkendelse, er det instruks-overtrædelse — dokumenteres i feedback-memory, eskaleres til Stefan.

Reviewer-rolle: Se `.claude/agents/<navn>-reviewer.md` for hvem og autoritet.

## Inbox-flow

### Team Inbox/ (FGD → teamet)

- FGD lægger filer her: opgaver, info, vedhæftede dokumenter.
- Stefan læser ved sessionstart og ved hver besked.
- Når en opgave er overtaget af en rolle: Stefan kvitterer ved at flytte/arkivere filen — eller noterer i `Min Inbox/<dato>-status.md` hvilken rolle der arbejder på det.

### Min Inbox/ (teamet → FGD)

- Færdige resultater, udkast til rolle-ansættelser, status-rapporter, spørgsmål til FGD.
- Filnavn: `<dato>-<emne>.md`.
- Hver fil starter med kort dansk TL;DR og afsluttes med "Næste skridt"-sektion (hvad FGD eller teamet skal gøre).

### Team/<rolle>/ (intern arbejdsmappe)

- Kladder, scratch-noter, pågående arbejde, midlertidige output.
- Ingen forventet stabilitet — roller må gerne rydde op selv.
- Større analyse-rapporter (Pouls fund, Camillas indekser) gemmes her med dato-prefix til fremtidig reference.

## Hand-off-mønstre

| Situation | Flow |
|---|---|
| Ny opgave fra FGD | Team Inbox/ → Stefan → relevant rolle → Min Inbox/ |
| Uklart hvilken rolle | Stefan → Poul (analyse) → Stefan vælger |
| Mangler kompetence | Stefan → Poul (gap-analyse) → Laila (rolle-design) → Min Inbox/ til FGD-godkendelse → Camilla (commit) |
| Ny info om FGD/projektet | Hvilken rolle der lærer det → Stefan → Camilla (memory-opdatering) |
| Compliance-vurdering / governance-policy | Hvem som helst → Stefan → Mads |
| System-niveau threat-model | Mads + Poul (peer-sparring) |
| Pen-test / audit-forberedelse | Mads ejer; involverer Karen ved feature-niveau |
| Dokument/skema/git | Hvem som helst → Camilla |
| Ekstern research | Hvem som helst → Poul |

## Ansvar — kort version

| Rolle | Skriver til |
|---|---|
| Stefan | `Min Inbox/`, `Team/<rolle>/` (kun korte koordinerings-noter) |
| Laila | `Min Inbox/<dato>-ny-rolle-*.md`, `Team/Laila/` |
| Poul | Agent-rapporter retur, `Team/Poul/<dato>-*.md`, `governance/design-docs/` (hvis governance-påvirkning) |
| Mads | `Team/Mads/`, `governance/` (alle subfolders) |
| Camilla | Alt — inkl. `.claude/agents/*` (efter godkendelse), git, memory, DB |

Stefan, Laila og Poul **skriver aldrig** til `memory/`-mappen eller laver git-commits — det er Camillas område. Mads skriver ikke til `memory/` eller `.claude/agents/` direkte.

## Eskalerings-paths

**Korrekt eskalerings-kæde for sikkerhed og compliance**:

- **Security / compliance-spørgsmål** → Mads (CISO). Mads kalder Poul ved behov for dyb research.
- **Arkitektur / behovsanalyse / research** → Poul direkte
- **Hardware / HSM-operationer** → Bjarne (rapporterer til Mads)
- **Code-review med sikkerhedspåvirkning (threat-modeling, crypto, auth, RLS)** → Karen (rapporterer til Mads)
- **GDPR-spørgsmål** → Dorthe (DPO, når aktiveret) eller Mads indtil Dorthe ansættes
- **Audit-evidens / SOC 2 / ISO 27001** → Trine (når aktiveret), rapporterer uafhængigt
- **Dokument-vedligehold / git / memory** → Camilla
- **Commit-godkendelse** → Vibeke (når aktiveret), Mads midlertidig bro-reviewer
- **Rolle-design / HR-spørgsmål** → Laila
- **Penetrations-test** → Frederik (når aktiveret)

**Vigtig regel**: Sikkerhed eskaleres **ALDRIG direkte til Poul** — altid via Mads. Poul kontaktes af Mads når research-dybde eller compliance-arkæologi er nødvendig.

Stefan orkestrerer og delegerer til ovenstående mønstre.

## Navne-pulje (til fremtidige ansættelser)

Sigende danske fornavne — Laila vælger frit fra denne (eller andre danske navne), så længe der ikke er konflikt:

> Anders, Mette, Jens, Sofie, Henrik, Karen, Lars, Ida, Bo, Pia, Søren, Birgitte, Niels, Anne, Kim, Lone, Per, Helle, Mads, Trine, Ole, Marianne, Frederik, Louise, Tobias, Charlotte.

## Datoer

I dag: brug systemets dato-information. I memory og inbox-filer: altid absolutte datoer (ikke "i går", "næste uge").

## Dokument-vedligehold — levende vs. arkiverede

Dokumenter i denne workspace falder i to kategorier:

### Levende dokumenter

- **Status**: `leveret` (analyse/rapport der stadig er master) eller `aktiv` (løbende arbejde)
- **Ansvar**: opdateres når underliggende beslutninger eller fakta ændrer sig
- **Trigger**: når FGD træffer ny beslutning eller facts ændres, beder Stefan eksplicit Camilla opdatere dokumentet + alle afhængige dokumenter
- **Eksempler**:
  - `Team/Poul/2026-05-05-vidensapp-analyse-v3.md` (master arkitektur)
  - `Min Inbox/projects/vidensapp/2026-05-05-fgd-aktive-udestaaender.md` (FGD's TODO)
  - `Team/_konventioner.md` (denne fil)
  - `.claude/agents/*.md` (rolle-mandater)
  - Memory-filer i `~/.claude/projects/.../memory/`

### Arkiverede dokumenter

- **Status**: `arkiveret`
- **Frontmatter-felter**: tilføj `arkiveret-af:`, `arkiveret-dato: <YYYY-MM-DD>`, `afløst-af: <sti-til-ny-version>`
- **Banner**: tilføj øverst i body efter frontmatter: `> **⚠ Dette dokument er arkiveret pr. <dato>. Master er [<titel>](<link>). Vedligeholdes ikke.**`
- **Placering**: Filen flyttes til `arkiv/<spejl-sti>/<filnavn>` med `git mv` (bevarer historik). Mappe-struktur under `arkiv/` spejler hoved-katalog (fx `arkiv/Team/Poul/`, `arkiv/Min Inbox/`).
- **Relative links**: Opdateres så de peger på korrekt sted efter flytning. Typisk `../../../Team/Poul/...` eller `../../Team/Poul/...` afhængig af dybde.
- **Vedligehold**: IKKE — de bevares som historik
- **Procedure**: 
  1. Ny version kommer (eller dokument bliver obsolet).
  2. Camilla markerer den gamle fil som arkiveret: `status: arkiveret`, `afløst-af:`, `arkiveret-dato:`, `arkiveret-fra:` (oprindelig sti).
  3. Tilføjer banner øverst i body.
  4. Opdaterer alle relative links i filen.
  5. Flytter fil til `arkiv/` med `git mv`.
  6. Finder andre aktive filer der refererer til den gamle sti og opdaterer dem til `arkiv/...`.
- **Excalidraw-filer**: når en ny PDF eller diagram-master udgivelse sker, markeres Excalidraw som `outdated` (ikke arkiveret) med companion-fil `.NOTE.md` der dokumenterer at den ikke er aktuel — PDF er master pr. dato. Forbliver in-place.

### Eksempel: arkivering med flytning til arkiv/

Fil oprindelig på: `Team/Poul/2026-05-05-vidensapp-analyse.md` (v1)
Flyttes til: `arkiv/Team/Poul/2026-05-05-vidensapp-analyse.md`

```yaml
---
status: arkiveret
afløst-af: Team/Poul/2026-05-05-vidensapp-analyse-v3.md
arkiveret-dato: 2026-05-05
arkiveret-fra: Team/Poul/2026-05-05-vidensapp-analyse.md
---

> **⚠ Dette dokument er arkiveret pr. 2026-05-05. Master er [v3-analysen](../../../Team/Poul/2026-05-05-vidensapp-analyse-v3.md). Vedligeholdes ikke.**

# Analyse...
```

### Ansvar

- **Stefan**: stiller Camilla spørgsmål når han ser uoverensstemmelser. Markerer aldrig selv som arkiveret.
- **Camilla**: holder indeks og markerer arkivering. Sikrer pil-sammenhænge mellem versioner.
- **Poul/Laila/Andre**: skriver kun nye dokumenter; flagrer til Stefan hvis de ser at noget bør opdateres eller arkiveres.

## Memory-organisation — permanent AI-hukommelse

Memory-mappen ligger **udenfor** repo-projektet: `~/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/`

Loades **automatisk** af Claude Code system-prompt hver session. Skal være velorganiseret og vedligeholdt:

```
memory/
├── MEMORY.md                          ← index (Markdown, auto-loaded)
├── _crosscutting/                     ← FGD-team-konventioner, feedback, permanent-rolle-info
│   ├── feedback_version_verification.md
│   ├── feedback_agent_placering.md
│   ├── stefan-rolle-godkendelses-flow.md
│   ├── dokument-vedligeholds-governance.md
│   └── (roller som Mads som gælder hele organisationen)
└── projects/
    └── vidensapp/                     ← vidensapp-projekt-specifik info
        ├── vidensapp-projekt-2026-05-05.md
        ├── hsm-fips-140-produktionsmål-2026-05-05.md
        └── (andre vidensapp-filer)
```

### Filstruktur

Hver memory-fil har frontmatter:

```yaml
---
name: {{kort navn}}
description: {{én linje — bruges til relevans-vurdering}}
type: user | feedback | project | reference
project: crosscutting | vidensapp | andet-projekt-navn
---
```

- **`type: feedback`** — FGD-korrektioner og bekræftede arbejdsmønstre. Inkludér **Why** og **How to apply**.
- **`type: project`** — vidensapp-projektet: beslutninger, deadlines, motivationer.
- **`type: reference`** — pointers til eksterne systemer (Linear, Slack, dashboards).
- **`type: user`** — info om FGD selv (rolle, præferencer).
- **`project: crosscutting`** — gælder alle projekter.
- **`project: vidensapp`** — kun vidensapp-kontekst.

### Organisering

- **Crosscutting** = info der gælder hele FGD-teamet (feedback, konventioner, permanente roller, governance-principper).
- **Project-specifik** = kun relevant for ét projekt (vidensapp arkitektur, kunde-vertikaler, roller som Karen/Bjarne).

### Vedligehold

- **Camilla** ejer memory — hun er **eneste skribent** til `memory/`-mappen.
- Stefan/Poul/Laila/Mads fortæller Camilla hvad der skal huskes; hun formulerer og indekserer.
- Dublet-tjek: hvis en ny memory-fil minder om en eksisterende, opdatér den gamle i stedet for at duplikere.
- **MEMORY.md** grupperes efter crosscutting + project-sektioner med relative stier.
