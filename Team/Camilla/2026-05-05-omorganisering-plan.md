---
name: Omorganisering til project-scope
description: Flytningsplan for strukturering af Multi-Project KDB
status: plan
---

# Omorganisering til project-scope — plan

## Strategi

**Faste roller** (Stefan/Laila/Poul/Camilla/Mads) bevares **uden project-scope**. De tjener på tværs.

**Project-specifikke roller** (Sofie/Henrik/Anders/Karen/Bjarne/Dorthe/Trine/Frederik) får hver sitt folder under `Team/projects/<projekt>/<rolle>/`.

**Inbox + arkiv** får `projects/<projekt>/` subfoldere.

## Fil-oversigt

**Min Inbox/ (14 aktive filer)**:
- 7 rolle-mandater (Anders, Bjarne, Dorthe, Frederik, Henrik, Karen, Sofie)
- 4 Laila-rapporter (3-roller, Karen-Bjarne, Mads, Mads-justeringer)
- 2 status-filer (team-bootstrappet, fgd-aktive-udestaaender)
→ Alle → `Min Inbox/projects/knowlagecentral/`

**Team Inbox/ (3 filer)**:
- 1 Mads CISO-mandat
- 1 Poul svar på spørgsmål
- 1 knowlagecentral-evernote-style
→ Alle → `Team Inbox/projects/knowlagecentral/`

**Team/Poul/ (9 knowlagecentral-relevante filer)**:
- 6 knowlagecentral-analyse + arkitektur (v3 master + PDF + Excalidraw + NOTE)
- 2 andre-sikkerhed-analyser (secrets-FIPS, YubiHSM)
- 1 vercel-undersøgelse
- +1 udvidelser-vurdering

**Beslutning for Pouls output**: **BEVARES i Team/Poul/** (fordi Poul er fast rolle, crosscutting). Filnavnene får ikke project-tag — git-historik + CLAUDE.md vil dokumentere at de udspringer af knowlagecentral-behovet.

**Team/Camilla/, Team/Laila/, Team/Mads/** — bevares uden ændring (faste roller).

**arkiv/ (3 filer)**:
- v1, v2 knowlagecentral-analyser fra Poul
- v1 knowlagecentral-analyse fra Min Inbox
→ Flytter til `arkiv/projects/knowlagecentral/` med samme sti-spejling

## Flytnings-liste

```
Min Inbox/*2026-05-05*.md (14 filer) 
  ├─ 2026-05-05-anders-ai-pipeline-mandat.md
  ├─ 2026-05-05-bjarne-hsm-operator.md
  ├─ 2026-05-05-dorthe-dpo.md
  ├─ 2026-05-05-fgd-aktive-udestaaender.md
  ├─ 2026-05-05-frederik-pentest.md
  ├─ 2026-05-05-henrik-backend-mandat.md
  ├─ 2026-05-05-karen-security.md
  ├─ 2026-05-05-laila-rapport-*.md (4 filer)
  ├─ 2026-05-05-sofie-frontend-mandat.md
  └─ 2026-05-05-team-bootstrappet.md
  → Min Inbox/projects/knowlagecentral/

Team Inbox/*2026-05-05*.md (3 filer)
  ├─ 2026-05-05-mads-ciso-mandat.md
  ├─ 2026-05-05 - Svar på Pouls spørgsmål
  └─ 2026-05-05-knowlagecentral-evernote-style.md
  → Team Inbox/projects/knowlagecentral/

arkiv/Min Inbox/2026-05-05-knowlagecentral-pouls-analyse.md
  → arkiv/projects/knowlagecentral/Min Inbox/

arkiv/Team/Poul/2026-05-05-knowlagecentral-analyse-*.md (2 filer)
  → arkiv/projects/knowlagecentral/Team/Poul/
```

**Knowlagecentral-filer i Team/Poul/** forbliver på plads (fast rolle-mappe).

## Interne links der skal opdateres

**Primære kilder**:
1. `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` — hovedanalyse, refs til andre filer
2. Alle 7 rolle-mandater — refs til analyse-filer, arkitektur-PDF
3. `.claude/agents/poul-analytiker.md` — refs til output-filer (evt.)
4. `Team/_konventioner.md` — "Mapper"-sektion skal reflektere ny struktur
5. `CLAUDE.md` — mappe-workflow skal referere `projects/<projekt>/`-path'er

**Grep-resultat**: ~15-20 interne paths kræver update.

## Mapper der skal oprettes

```
mkdir -p "Min Inbox/projects/knowlagecentral"
mkdir -p "Team Inbox/projects/knowlagecentral"
mkdir -p "arkiv/projects/knowlagecentral/Min Inbox"
mkdir -p "arkiv/projects/knowlagecentral/Team/Poul"
```

Derefter `.gitkeep` i nye top-level inboxer hvis de bliver tomme.

## Risici & mitigering

| Risiko | Handling |
|---|---|
| Broken relative links | Identificér alle `[text](../../../...)` patterns, test med `cat` efter flytning |
| Git-historik går tabt | Bruger `git mv` (ikke `cp` + `git rm`) — historik bevares |
| Inline-embeds (PDFs i markdown) | PDF-links skal opdateres (test refs fra Min Inbox-mandater til `Team/Poul/*.pdf`) |
| Stefan-CLAUDE.md outdated | Opdater mappe-workflow-sektion eksplicit |
| Memory-filer peger på gamle paths | Camilla opdaterer memory efter flytning |

## Rækkefølge

1. Opret mapper
2. `git mv` alle Min Inbox-filer
3. `git mv` alle Team Inbox-filer
4. `git mv` alle arkiv-filer
5. Grep + manuelt fix af interne links i:
   - v3-analyse
   - 7 rolle-mandater
   - Team/_konventioner.md
   - CLAUDE.md
6. Verificér `.claude/agents/` uberørt
7. `git status` — alle moves skal være renames (`RM`)

## Håndtering af .gitkeep

- `Min Inbox/.gitkeep` + `Team Inbox/.gitkeep` bevares på top-niveau (tomme inboxer efter flytning skal have .gitkeep)
- Nye `projects/`-folders får deres egne `.gitkeep` hvis de skal committed (typisk nej — mappestrukturen oprettes af move'erne)

## Estimat

- **Filer flyttet**: ~20 (14 Min Inbox + 3 Team Inbox + 3 arkiv)
- **Mapper oprettet**: 4
- **Interne links opdateret**: ~20
- **Ingen binærer eller secrets flyttet**

**Ressource**: ~45 minutter fra git-mv til link-verifikation.
