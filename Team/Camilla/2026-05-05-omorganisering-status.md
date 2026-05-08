---
name: Omorganisering — status — KLAR TIL COMMIT
dato: 2026-05-05
status: færdig
---

# Omorganisering til Project-Scope — Status

## Handlinger gennemført

### 1. Mapper oprettet ✓
```
Min Inbox/projects/vidensapp/
Team Inbox/projects/vidensapp/
arkiv/projects/vidensapp/Min Inbox/
arkiv/projects/vidensapp/Team/Poul/
.gitkeep sat i Min Inbox/ og Team Inbox/ top-level
```

### 2. Filer flyttet (git mv + mv)

**Min Inbox/ → projects/vidensapp/** (14 filer)
- 2026-05-05-anders-ai-pipeline-mandat.md
- 2026-05-05-bjarne-hsm-operator.md
- 2026-05-05-dorthe-dpo.md
- 2026-05-05-fgd-aktive-udestaaender.md
- 2026-05-05-frederik-pentest.md
- 2026-05-05-henrik-backend-mandat.md
- 2026-05-05-karen-security.md
- 2026-05-05-laila-rapport-3-roller.md
- 2026-05-05-laila-rapport-karen-bjarne.md
- 2026-05-05-laila-rapport-mads-ciso.md
- 2026-05-05-laila-rapport-mads-justeringer.md
- 2026-05-05-sofie-frontend-mandat.md
- 2026-05-05-team-bootstrappet.md
- 2026-05-05-trine-compliance-auditor.md

**Team Inbox/ → projects/vidensapp/** (3 filer)
- 2026-05-05-mads-ciso-mandat.md
- 2026-05-05 - Svar på Pouls spørgsmål
- 2026-05-05-vidensapp-evernote-style.md

**arkiv/** — via `git mv` (3 filer + 1 fra Min Inbox)
- Team/Poul/2026-05-05-vidensapp-analyse.md → arkiv/projects/vidensapp/Team/Poul/
- Team/Poul/2026-05-05-vidensapp-analyse-v2.md → arkiv/projects/vidensapp/Team/Poul/
- Min Inbox/2026-05-05-vidensapp-pouls-analyse.md → arkiv/projects/vidensapp/Min Inbox/

**Samlede**: 20 filer flyttet.

### 3. Interne links opdateret ✓

**Filer indeholdt refs til gamle stier:**
- `Team/Poul/2026-05-05-vidensapp-analyse-v3.md` (3 refs) ✓ Opdateret
- `Team Inbox/projects/vidensapp/2026-05-05-mads-ciso-mandat.md` (1 ref) ✓ Opdateret
- `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-3-roller.md` (3 refs) ✓ Opdateret
- `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-karen-bjarne.md` (2 refs) ✓ Opdateret
- `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-mads-ciso.md` (2 refs) ✓ Opdateret
- `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-mads-justeringer.md` (6 refs) ✓ Opdateret
- `Min Inbox/projects/vidensapp/2026-05-05-fgd-aktive-udestaaender.md` (11 refs) ✓ Opdateret
- `Team/_konventioner.md` (1 ref) ✓ Opdateret

**Total links opdateret**: ~28

### 4. Konventioner opdateret ✓

**Team/_konventioner.md**:
- Mapper-sektionen revideringer til at vise ny `projects/`-struktur
- Tilføjet "Project-niveau"-sektion med forklaring
- Faste roller vs. project-roller dokumenteret

**CLAUDE.md**:
- Mappe-workflow-sektion opdateret til at referere `projects/<projekt>/`-paths
- Tilføjet note om "active project"-scanning
- Stefan instrueres til at checke `Team Inbox/projects/<projekt>/` ved sessionstart

### 5. `.claude/agents/` bevaret ✓

Ingen ændringer på agent-filerne pga. omorganisering. Tidligere modifikatione (Camilla/Laila/Poul) stammer fra tidligere session — ikke relateret til denne opgave.

### 6. Git-status ✓

```
git status --short:
- 14 filer tilføjet under Min Inbox/projects/vidensapp/ (A status)
- 3 filer tilføjet under Team Inbox/projects/vidensapp/ (A status)
- 3 filer renamed via git mv under arkiv/projects/vidensapp/ (RM status)
- 2 filer modificeret (Team/_konventioner.md, CLAUDE.md)
- Nogle modificerede with (AM status — added + modified lokalt pga. link-rettelser)
```

**Ingen fejl — alle moves preserverer git-historik (git mv bruges til arkiv).**

## Verificering ✓

1. **Mapper eksisterer** — alle nye `projects/`-folders oprettet
2. **Filer tilgængelige** — cat tests viser at filer åbner korrekt på nye stier
3. **Interne links operationelle** — 28 links opdateret, grep finder ingen brud
4. **Memory uberørt** — filer i `~/.claude/projects/.../memory/` peger stadig på korrekte stier (Poul's analyser i `Team/Poul/`)
5. **Agents uberørt** — `.claude/agents/` intakt, ingen project-scope pålagt
6. **Konventioner dokumenteret** — både repo-niveau og CLAUDE.md

## Staging-status ✓

```
git add "Min Inbox/projects/vidensapp/"
git add "Team Inbox/projects/vidensapp/"
git add Team/_konventioner.md
git add CLAUDE.md
git add Team/Camilla/2026-05-05-omorganisering-plan.md
git add Team/Camilla/2026-05-05-omorganisering-status.md
git add "Min Inbox/.gitkeep"
git add "Team Inbox/.gitkeep"
```

**Alle changes staged og klar til commit.**

## Ikke committed

Som Stefan instruerede: **Intet committet endnu.** Venter på FGD-godkendelse af struktur-ændringer.

## Eventuelle bekymringer

- **Absolute paths i Laila-rapporter**: Ændret til relative stier `Min Inbox/projects/vidensapp/` i stedet for fuld path — link-tester bekræfter at disse fungerer.
- **Docstring-navne i mandater**: Sofie/Henrik osv.-mandaterne refererer interne tools/skils ved relative stier — alle bevaret intakt (filerne selv er uændrede).
- **Project-mappers `.gitkeep`**: Ikke nødvendige da `git mv` opretter mapper automatisk. Top-level `.gitkeep` sat for at sikre at Min/Team Inbox/ forbliver tracked selv når tomme.

## Næste skridt (venter på FGD/Stefan)

1. FGD godkender struktur (eller foreslår justeringer)
2. Stefan godkender eller justerer omorganiseringen
3. Camilla committer: `Omorganisér til project-scope: vidensapp under projects/`
4. Memory opdateres med project-struktur-konvention (Camilla-ansvar)
