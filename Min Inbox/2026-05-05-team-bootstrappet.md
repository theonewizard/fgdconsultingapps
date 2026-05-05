# Team-bootstrappet — Stefan, Laila, Poul, Camilla er på plads

**Fra:** Stefan
**Til:** FGD
**Dato:** 2026-05-05

## TL;DR

Teamet er sat op. Næste session vil jeg (Stefan) hilse som AI-organisator, og du kan begynde at lægge opgaver i `Team Inbox/`.

## Hvad blev gjort

| Fil | Formål |
|---|---|
| `CLAUDE.md` | Stefans rolle, pragmatisk delegerings-regel, mappe-workflow |
| `.claude/agents/laila-hr.md` | **Laila** — AI HR (haiku) |
| `.claude/agents/poul-analytiker.md` | **Poul** — senior analyse, multi-track (opus) |
| `.claude/agents/camilla-bibliotekar.md` | **Camilla** — docs, git, DB, memory (haiku) |
| `.claude/agents/_template-ny-rolle.md` | Skabelon Laila bruger til nye ansættelser |
| `Team/_konventioner.md` | Navngivning, hand-offs, sprog, ansvar |
| `Team/{Laila,Poul,Camilla}/README.md` | Arbejdsmapper |
| `.gitignore` + git-init | Repo'et er nu under version-kontrol |

Initial commit: `ceb4d70` ("Bootstrap: Stefan-team — AI-organisator + Laila, Poul, Camilla").

## Vigtigt at vide

- **Subagents loades ved sessionstart.** Laila, Poul og Camilla kan først kaldes i en ny Claude-session i denne mappe. I den session kan jeg uddelegere normalt.
- **Stefans grænser** (pragmatisk valg): Jeg må læse alt og skrive korte koordinerings-noter. Al produktion (kode, docs, analyser, memory, git ud over status) uddelegeres altid.
- **Camilla skriver memory** — ikke Stefan, ikke Laila, ikke Poul. Memory ligger i `~/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/` og loades automatisk i hver session.

## Næste skridt — første opgave til Camilla i næste session

Når du starter en ny session i denne mappe, beder jeg Camilla skrive de første memory-entries (jeg har klargjort indholdet allerede):

1. **project:** FGD's delegerings-først arbejdsmodel.
2. **project:** Faste roller (Laila/Poul/Camilla) + at fremtidige hires får sigende danske navne.
3. **user:** FGD's profil (ejer FGDConsulting, dansk, foretrækker rolle-baseret AI-organisation).
4. **feedback:** Kun Camilla skriver til memory.
5. **reference:** Auto-memory-systemets format og placering.

Planen ligger gemt i `/Users/fgd/.claude/plans/hi-stefan-du-er-rippling-tide.md` hvis du vil genbesøge den.

## Forslag til test af opsætningen

I næste session, prøv en af:

- **"Læg en testopgave i Team Inbox/"** — fx en fil med "undersøg hvilke MCP-servers der er tilgængelige i denne mappe og deres formål". Stefan delegerer til Poul.
- **"Ansæt en rolle der kan hjælpe med video-checksum-pipeline"** — Stefan → Poul (analyse) → Laila (rolle-design) → udkast i Min Inbox/.
- **"Camilla, hvad husker du om mig?"** — Camilla læser memory og rapporterer.
