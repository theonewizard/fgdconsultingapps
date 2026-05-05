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
/                        ← projektrod
├── CLAUDE.md            ← Stefans instrukser (loades automatisk)
├── .claude/agents/      ← rolle-definitioner (committet)
├── Min Inbox/           ← teamet → FGD
├── Team Inbox/          ← FGD → teamet
└── Team/                ← rollernes arbejdsmapper
    ├── Laila/
    ├── Poul/
    ├── Camilla/
    └── _konventioner.md ← denne fil
```

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
| Dokument/skema/git | Hvem som helst → Camilla |
| Ekstern research | Hvem som helst → Poul |

## Ansvar — kort version

| Rolle | Skriver til |
|---|---|
| Stefan | `Min Inbox/`, `Team/<rolle>/` (kun korte koordinerings-noter) |
| Laila | `Min Inbox/<dato>-ny-rolle-*.md`, `Team/Laila/` |
| Poul | Agent-rapporter retur, `Team/Poul/<dato>-*.md` |
| Camilla | Alt — inkl. `.claude/agents/*` (efter godkendelse), git, memory, DB |

Stefan, Laila og Poul **skriver aldrig** til `memory/`-mappen eller laver git-commits — det er Camillas område.

## Navne-pulje (til fremtidige ansættelser)

Sigende danske fornavne — Laila vælger frit fra denne (eller andre danske navne), så længe der ikke er konflikt:

> Anders, Mette, Jens, Sofie, Henrik, Karen, Lars, Ida, Bo, Pia, Søren, Birgitte, Niels, Anne, Kim, Lone, Per, Helle, Mads, Trine, Ole, Marianne, Frederik, Louise, Tobias, Charlotte.

## Datoer

I dag: brug systemets dato-information. I memory og inbox-filer: altid absolutte datoer (ikke "i går", "næste uge").
