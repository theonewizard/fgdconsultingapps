---
dato: 2026-05-05
fra: Stefan
emne: Vidensapp i Evernote-stil — Pouls analyse + spørgsmål til dig
status: afventer-fgd-svar
---

# Til FGD — Pouls analyse er klar

Poul har leveret en grundig behovs-analyse + arkitekturforslag for vidensappen.
Camilla persisterer den fulde tekst i `Team/Poul/2026-05-05-vidensapp-analyse.md`.

## Pouls hovedanbefaling (kort)

**Stack**: Next.js 15 App Router (PWA via Serwist) + Neon Postgres med pgvector
+ Clerk auth + Neon RLS Authorize + Inngest til AI-pipeline + Anthropic Haiku 4.5
(batch-berigelse) + Sonnet 4.5 (manuel rewrite). Hostet på Vercel + Neon i Frankfurt.
Drizzle som ORM. OpenAPI 3.1-kontrakt + RFC 9457 errors + idempotency fra dag 1.

**Sikkerhed**: defense-in-depth mod BOLA — både API-lag tjekker membership *og*
Postgres RLS som sidste forsvarslinje. Opfylder dine globale API-standarder.

**Søgning**: Postgres tsvector i MVP, udvides til hybrid (FTS + pgvector via RRF)
i fase 2 når der er volumen at retfærdiggøre det.

**Scope**: Medium. MVP (du alene) ~2-3 uger. v1 (delt med kunder) yderligere ~2-3 uger.

## Roller Laila skal ansætte (3 stk)

1. **Frontend-udvikler** (Next.js 15 + Serwist)
2. **Backend/Neon-specialist** (RLS, Drizzle, OpenAPI)
3. **AI-pipeline-designer** (Inngest + Anthropic + prompts)

Eksisterende team (Camilla, Poul, mig) dækker DB-opsætning, ad-hoc research og koordination.

## 10 spørgsmål Poul beder dig om at svare på før vi går i gang

| # | Spørgsmål | Hvorfor det betyder noget |
|---|---|---|
| 1 | Hvor mange txt-noter har du, og er de struktureret i mapper? | Mappe-struktur kan mappes 1-1 til notebooks |
| 2 | Hvor mange kunder forventes i de første 6 mdr? | Påvirker plan-valg på Vercel/Neon/Clerk |
| 3 | Skal kunder kunne self-signup via invite-link, eller opretter du dem manuelt? | Påvirker invite-flow design |
| 4 | Brug for "subteam" pr. kunde (3 personer hos samme kunde)? | Afgør om vi skal have organisations-koncept i v1 |
| 5 | Domæne: `app.fgdconsulting.se` ok, eller noget andet? | Påvirker Clerk + CORS allow-list |
| 6 | AI-budget pr. måned (cap)? | Sætter loft i Anthropic-konsol + Inngest concurrency |
| 7 | Skal kunder kunne opt-out af AI-berigelse pr. notebook? (GDPR) | Påvirker datamodel (flag på notebooks) |
| 8 | Region: Frankfurt ok? (Neon har ikke nordisk endnu) | Lavest latens fra DK/SE er Frankfurt |
| 9 | Eksisterende OpenAI/Anthropic-konti vi skal bruge, eller opretter Camilla nye? | Konto- og fakturerings-spor |
| 10 | Versions-historik: alle ændringer eller kun manuelle snapshots? | Påvirker storage-volumen i `note_versions` |

## Mine anbefalinger til næste skridt

1. **Du svarer på de 10 spørgsmål** (helt fint hvis nogle er "vælg du, Poul").
2. **Du godkender stack-retningen** (eller siger "skift X til Y").
3. Derefter sætter jeg **Laila** i gang med at designe og oprette de 3 roller.
4. Parallelt kan **Camilla** oprette Neon-projekt + Vercel + Clerk-skelet, så de
   nye roller kan starte i et kørende miljø.

## Hvor du finder detaljerne

- Pouls fulde analyse (datamodel-skitse, RLS-policies, SQL for hybrid-søg,
  Inngest-flow, faseret plan): `Team/Poul/2026-05-05-vidensapp-analyse.md`
- Original opgavebrief: `Team Inbox/2026-05-05-vidensapp-evernote-style.md`

— Stefan
