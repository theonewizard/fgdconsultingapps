---
indkommet: 2026-05-05
fra: FGD
status: under-analyse
ejer: Poul (analyse) -> TBD (arkitektur+build)
---

# Knowlagecentral i Evernote-stil — opgavebrief

## FGD's ord

> Vi skal have opsat en videns applikation som kan håndtere noter som i Evernote.
> Mange af mine noter er i txt-format og meget basale, så der vil være en del
> kommentarer og research der skal gøres for at få dem gjort sigende.
> De skal organiseres i notebøger som er relaterede til emneområder.
> Det kan f.eks. være ud fra kunder.
> Til formålet skal data lagres i Neon, og der skal bygges en frontend hvor jeg
> kan søge de enkelte noter frem fra både en computer og mobil enhed.

## Bekræftede rammer (afklaret med FGD 2026-05-05)

| Rammepunkt | Valg |
|---|---|
| **Brugere** | Dig (FGD) + delte notebooks til kunder (multi-user, RBAC, per-notebook authorization) |
| **AI-berigelse** | Auto-berigelse ved import + manuel raffinering ovenpå |
| **Volumen v1** | Under 100 noter initialt — men datamodellen skal kunne skalere |
| **Frontend-stack** | Next.js / React (PWA-vinkel er relevant for desktop+mobil i én kodebase) |
| **Backend-DB** | Neon Postgres |

## Implicitte krav (udledt)

- **Sikkerhed**: dette er en multi-user app med ekstern adgang → fuld OWASP API
  Top 10-compliance gælder (BOLA er kritisk pga. delte notebooks pr. kunde).
  Se `/Users/fgd/.claude/CLAUDE.md` for FGD's API-standarder.
- **Søgning**: skal virke på computer og mobil → server-side fuldtekst (Postgres
  `tsvector` eller pgvector) + responsivt UI.
- **Import-pipeline**: txt-filer skal kunne uploades / pege på en mappe og
  blive indekseret + beriget automatisk.
- **AI-berigelse v1** (forslag): titel-forslag, resumé, tags, foreslåede
  notebook-tilknytninger, links mellem relaterede noter.

## Hvad Poul skal levere (se Pouls scratch i Team/Poul/)

1. Behovs-analyse: brugerflows, use cases, ikke-funktionelle krav.
2. Foreslået arkitektur (Next.js + Neon + AI-pipeline) — med diagram.
3. Datamodel-skitse (notes, notebooks, users, sharing, enrichment-jobs).
4. Sikkerhedsmodel (auth, authz pr. notebook, sharing-regler).
5. AI-berignings-pipeline (model-valg, batching, manuel raffinerings-UI-flow).
6. Identifikation af manglende roller — hvad skal Laila ansætte?
7. Faseret implementeringsplan (MVP -> v1 -> nice-to-have).

## Næste skridt efter Pouls analyse

- Stefan vurderer leverancen.
- Laila ansætter nødvendige roller (sandsynligvis: en frontend-udvikler,
  en backend/Neon-specialist, evt. en AI-pipeline-designer).
- Camilla opretter Neon-projekt + git-struktur når arkitekturen er godkendt.

## Tillæg 2026-05-05 — krav fra FGD

### Miljø-separation: dev → test → prod (kritisk)

FGD vil have stabil tre-trins miljø-opdeling så han kan prøve nye idéer af
uden at påvirke produktionens tilgængelighed. Kravet gælder hele stakken:

- **Vercel**: production + stabilt test/staging (ud over ad-hoc previews) + lokal dev.
- **Neon**: separate branches (eller projekter) for prod / test / dev. Copy-on-write
  branching skal udnyttes så test-data altid kan genskabes fra prod-snapshot uden
  at kompromittere prod-data (PII-redaktion ved promotion?).
- **Clerk**: separate dev/prod instances (Clerk har det indbygget). Test-miljø
  skal beslutte: deler det Clerk dev-instance, eller har det sin egen?
- **Inngest**: env-keys pr. miljø.
- **Anthropic**: separate API-keys pr. miljø med spend-cap pr. key.
- **Domæner**: `app.fgdconsulting.se` (prod), `test.app.fgdconsulting.se`
  (eller subdomain efter FGD's valg), `localhost` (dev).

Pouls opdaterede analyse skal eksplicit dække:

- Hvordan promotion sker (dev → test → prod): manuel godkendelse? Trunk-based?
- Hvordan secrets håndteres pr. miljø (Vercel env-scoping).
- Hvordan DB-migrations ruller ud sikkert (test først, så prod, med rollback-plan).
- CI-pipeline der gater promotion (lint + typecheck + tests + Spectral skal være
  green før test-deploy; manuel approval før prod-deploy).

### Versions-aktualitet

Husk Pouls opdaterede mandat: alle frameworks/SDK'er skal Context7-verificeres.
Den oprindelige analyse anbefalede "Next.js 15" — sandsynligvis 16 nu.
Re-validér samtlige version-valg.
