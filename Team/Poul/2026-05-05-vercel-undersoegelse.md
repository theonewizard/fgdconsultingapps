---
dato: 2026-05-05
forfatter: Poul
emne: Vercel + Vercel-Neon-integration undersoegelse - justeringer til v2
status: leveret
relateret: arkiv/Team/Poul/2026-05-05-knowlagecentral-analyse-v2.md
relateret: Team/Poul/2026-05-05-knowlagecentral-udvidelser-vurdering.md
---

# Vercel-undersøgelse — justeringer til v2

## Sammendrag

3 vigtige opdagelser:

1. **Vercel anbefaler aktivt MIGRATION FRA Edge Runtime TIL Node.js på Fluid Compute** (officiel warning på `/docs/functions/runtimes/edge`). Default-on for nye projekter siden 2025-04-23.
2. **Vercel-Native Neon Integration auto-provisionerer ÉT Neon-projekt med branches pr. preview-deploy** — IKKE 3 separate projekter. v2-strategien er stadig mulig, men den naturlige integrations-pattern er 1 projekt + branches med automatisk PR-isolation.
3. **Vercel Pro = 1 custom environment per projekt** — rækker præcis til "test", men ikke yderligere "staging".

**Holder v2?** Stort set ja, med justeringer: (a) Edge → Node.js, (b) overvej 1-Neon-projekt vs. 3-projekter tradeoff, (c) Vercel AI SDK v6 over rå Anthropic SDK.

**Kan rolle-udkast godkendes som de er?** Nej, alle 3 har behov for justeringer.

## 0. Tool-status (vigtig sandhed)

Vercel MCP var IKKE tilgængelig i mine agent-tools på trods af FGD's intention om at koble den. Jeg inspicerede `/Users/fgd/.claude/plugins/installed_plugins.json` og fandt kun: superpowers, context7, code-review, andrej-karpathy-skills, code-simplifier. Ingen Vercel-plugins installeret.

Vercel-plugins findes i marketplacet (`vercel@claude-plugins-official` + `vercel-best-practices@claude-plugins-official`) men er ikke aktiveret.

**Action**: FGD køre `/plugin install vercel@claude-plugins-official` så fremtidige agent-kald har MCP-adgang.

Jeg faldt tilbage på Vercel docs via WebFetch — anbefalinger nedenfor er produkt/featurelandskab, ikke konto-specifikke.

## 1. Vercel-konto-status (kunne ikke verificeres)

Uden Vercel MCP kan jeg ikke se: plan, team-scope, eksisterende projekter, env vars, region-præferencer.

**Antagelse for resten af rapporten**: FGD er på (eller går på) Pro plan, $20/user/md, personal eller solo-team scope.

## 2. Vercel ↔ Neon Native Integration — faktisk virkemåde

Verificeret via `https://neon.com/docs/guides/vercel-native-integration`.

### Auto-provisionering
- Integrationen opretter Neon-konto + projekt automatisk.
- Eksisterende Neon-brugere får ny org: `Vercel: <team-name>`.
- **Billing flyttes til Vercel** — du faktureres af Vercel for Neon-forbruget.

### Per-PR branch-isolation
- Automatisk, men kræver opt-in toggle ("Required → Preview").
- Når aktiv: Neon opretter `preview/<git-branch>` automatisk pr. Vercel preview-deploy.
- Branches slettes automatisk når deploy fjernes (cleanup op til 6 mdr).
- Resultat: **gratis PR-DB-isolation** uden custom kode.

### Env vars auto-injiceret
- `DATABASE_URL` (pooled), `DATABASE_URL_UNPOOLED`, `PGHOST`, `PGUSER`, `PGDATABASE`, `PGPASSWORD`, legacy `POSTGRES_*`.
- Hvis Neon Auth aktiveret: `NEON_AUTH_BASE_URL`, `VITE_NEON_AUTH_URL`.

### Multi-projekt-strategi (kritisk for v2)
Ja, muligt — opret "additional databases" indenfor samme Vercel-integration. Hver bliver separat Neon-projekt.

**Tradeoff**:

| Strategi | Pro | Con |
|---|---|---|
| **1 projekt + branches (anbefalet)** | Auto-PR-isolation; én billing-stream; nemmere setup | Mindre stærk fortroligheds-isolation; shared compute-pool |
| **3 separate projekter (v2 oprindeligt)** | Fuld isolation; separate billing-views; separate regions muligt | Mister automatisk preview-branching; manuel cleanup; mere kompleks env-mapping |

**Anbefaling**: Start med 1 Neon-projekt + branches. Splitting er envejs-migrering der er nem at lave senere.

### Custom Vercel-environments (Pro)
Pro plan giver 1 custom environment per projekt. Rækker til "test".

For at koble custom-env til Neon-branch: sæt `DATABASE_URL` env var manuelt for custom env. Integration giver URL pr. branch i Neon-dashboardet.

### Pris
- Vercel Pro Functions: Active CPU-baseret, $0.128/time aktiv CPU.
- Neon-kvoter: branches under integration tæller mod Neon-projektets compute hours og storage. Free tier: 10 branches max → vi skal sandsynligvis op på Neon Launch ($19/md+) for produktion.

## 3. Anbefalinger: anvend / ikke-anvend andre Vercel-features

### Vercel AI SDK v6 — JA (Anders bør bruge)
- Stable, v6, open-source.
- Anthropic native via `@ai-sdk/anthropic` provider.
- Built-in: streaming, telemetry, error-handling/retry, language model middleware.
- Pris: gratis SDK; AI Gateway-feature frivillig.
- Begrundelse for skift fra rå Anthropic SDK: telemetry går automatisk i Vercel observability; provider-agnostisk (1-line ændring til Gemini/GPT senere); Vercel-supporteret.

### Vercel KV — IKKE relevant
Udfaset som eget produkt i 2026. KV-storage skal nu provisioneres via Marketplace (Upstash, Redis Cloud). Ikke i MVP.

### Vercel Blob — kan være relevant senere
$0.023/GB. Use case: attachments/billeder i noter. Ikke i MVP. Dokumentér som mulighed i fase 3+.

### Vercel Cron — JA til simple jobs
Pro: 100 cron jobs, minimum interval 1 min. HTTP GET trigger til route handler.

Sammenlignet med Inngest: ingen retry, ingen DLQ, ingen step-functions. Kun fire-and-forget.

Use cases der passer: nightly cleanup af expired invites, magic-links, weekly stats.

**Anbefaling**: Inngest til AI-pipeline (step-funktioner). Vercel Cron til daglig housekeeping.

### Vercel Edge Config — NEJ til AI-toggle
- Læser <1ms, writes "seconds".
- For `ai_enrichment_enabled`: vi har det som DB-felt på `notebooks`/`organizations` (per-tenant). DB er bedre når det er per-bruger/per-organisation.
- Brug Edge Config kun hvis vi senere får globale feature flags.

### Vercel Functions vs. Edge Runtime — UNDGÅ Edge

Officiel Vercel-warning på `/docs/functions/runtimes/edge`:

> "We recommend migrating from edge to Node.js for improved performance and reliability. Both runtimes run on Fluid compute with Active CPU pricing."

- Edge har ingen klar performance-fordel længere efter Fluid Compute (default fra 2025-04-23).
- Edge har stadig restriktioner: 25 sek til første response, 2 MB code limit, ingen native Node-libs.
- For vores stack (RSC + Server Actions + Neon HTTP-driver + Anthropic SDK): Node.js Functions med Fluid Compute er det rigtige valg.

**Sofie og Henrik bør IKKE bruge** `export const runtime = 'edge'`. Default Node.js + Fluid Compute (auto-on for nye projekter).

### Vercel Analytics + Speed Insights — JA delvist
- Web Analytics: $3/100k events efter free tier. Privacy-friendly visitor-tracking. Ikke observability.
- Speed Insights: $10/projekt/md + $0.65/10k events. Web Vitals (LCP, INP, CLS).
- Vs. Sentry: komplementært, ikke erstatning.
- **Anbefaling**: Speed Insights JA ($10/md). Web Analytics vent. Sentry beholdes til error tracking.

### Custom domains (`docs.fgdconsulting.se`)
Process for subdomain:
1. Tilføj domæne i Vercel Project Settings → Domains.
2. Vercel viser DNS-record.
3. CNAME hos FGD's DNS-provider: name=`docs`, value=`cname.vercel-dns.com`.
4. Vercel verificerer + udsteder SSL automatisk.

**Action til FGD**: Hvilken DNS-provider hostes `fgdconsulting.se` på?

## 4. Justeringer til v2-arkitekturen

| v2 antagelse | Faktisk situation | Justering |
|---|---|---|
| 3 separate Neon-projekter | Vercel-Integration pusher mod 1 projekt + auto-branches | **Skift til 1 Neon-projekt + 3 named branches + auto-preview-branches**. Fortrolighed kan splittes senere. |
| `DATABASE_URL` mappes manuelt | Auto-injiceret | **Brug integrations-flow** ikke manuel secret. |
| Direct Anthropic SDK | AI SDK v6 har Anthropic native + telemetry | **Anders skifter til AI SDK v6** |
| (implicit) Edge runtime ok | Vercel anbefaler aktivt mod Edge | **Eksplicit Node.js + Fluid Compute** |
| Custom env for "test" | Pro = 1 custom env | OK — dokumenteret at vi har brugt vores eneste |
| Inngest til alle bg-jobs | Vercel Cron dækker simple periodiske gratis | **Tilføj Vercel Cron til daglig housekeeping** |
| Sentry alene | Vercel Speed Insights komplementerer | **Tilføj Speed Insights** ($10/md) |
| Frankfurt på 3 projekter | Med 1 projekt: Frankfurt valgt ved oprettelse | **Stadig Frankfurt for prod-branch**. Branches arver region. |

## 5. Konkrete ændringer Laila skal lave i rolle-udkast

### Sofie (frontend) — JUSTERINGER

- Tilføj: "Brug Node.js runtime som default. Skriv aldrig `export const runtime = 'edge'` — Vercel anbefaler aktivt mod Edge i 2026."
- Tilføj: "Speed Insights setup ($10/md) — installér `@vercel/speed-insights/next`, monter i `app/layout.tsx`."
- Tilføj: "Custom domain `docs.fgdconsulting.se` via CNAME til `cname.vercel-dns.com`."
- Note: "Vi bruger Vercel-Native Neon-integration — Sofie behøver ikke opsætte DB-secrets manuelt."

### Henrik (backend/Neon) — STØRSTE JUSTERINGER

**Justering #1**: Erstat "Etablér 3 Neon-projekter" med:
> "Etablér 1 Neon-projekt via Vercel Native Integration (Marketplace → Neon). Region: Frankfurt. Aktivér 'Required → Preview' toggle. Opret named branches: `prod` (long-lived, scale-to-zero OFF), `test` (long-lived, scale-to-zero ON), `dev` (long-lived, scale-to-zero ON). Map til Vercel-environment via integrations-UI."

**Justering #2**: Secrets-mapping ny tabel:

| Secret | Production | Preview (PR) | Custom env 'test' | Development |
|---|---|---|---|---|
| `DATABASE_URL` | Auto (prod-branch) | Auto (preview/<branch>) | Manuel (test-branch URL) | Auto (dev-branch) |
| `CLERK_SECRET_KEY` | Prod instance | Dev instance | Dev instance | Dev instance |
| `ANTHROPIC_API_KEY` | Prod workspace | Dev workspace | Test workspace | Dev workspace |
| `INNGEST_EVENT_KEY` | Prod env | Dev env | Test env | Dev env |
| `VOYAGE_API_KEY` | Prod account | Prod account | Prod account | Prod account |

**Justering #3**: Tilføj Vercel Cron jobs (`vercel.json` `crons`):
- `0 3 * * *` GET `/api/cron/expire-invites` — slet expired invites.
- `0 4 * * *` GET `/api/cron/archive-old-versions` — komprimér/slet versions >12 mdr.
- Cron-routes validerer `vercel-cron/1.0` user-agent + intern signing-token.

**Justering #4**: Hard rule: alle Route Handlers og Server Actions kører Node.js runtime.

**Justering #5**: Tilføj `/api/cron/*` til OpenAPI-kontrakt.

### Anders (AI-pipeline) — JUSTERINGER

**Justering #1**: Brug Vercel AI SDK v6 (`ai` + `@ai-sdk/anthropic`) frem for rå `@anthropic-ai/sdk`. Voyage-3.5 embeddings via direkte SDK.

**Justering #2**: 3 Anthropic workspaces fastholdes uafhængigt af Neon-strategi.

**Justering #3**: Inngest + Vercel Cron koeksisterer — Inngest til AI-berigelse, Cron til cleanup.

**Justering #4**: Inngest webhook-endpoint `/api/inngest` kører Node.js runtime.

**Justering #5**: Anders evaluerer senere AI Gateway (frivillig) — ikke MVP-blocker.

## Åbne spørgsmål til FGD

1. **Vercel-plugin installation** — kør `/plugin install vercel@claude-plugins-official`?
2. **Neon-strategi** — 1 projekt + branches (anbefalet) eller 3 projekter?
3. **DNS-provider** for `fgdconsulting.se`?
4. **Vercel team-scope** — personal eller team?
5. **AI Gateway** — Anders evaluerer senere?

## Genbrug

- `vercel@claude-plugins-official` + `vercel-best-practices@claude-plugins-official` plugins identificeret — installer for fremtidige MCP-adgang.
- `context7@claude-plugins-official` installeret — Sofie/Henrik/Anders skal bruge til version-verificering.

## Kilder

- Vercel Native Neon Integration: https://neon.com/docs/guides/vercel-native-integration
- Vercel Environments: https://vercel.com/docs/deployments/environments
- Vercel Edge Runtime warning: https://vercel.com/docs/functions/runtimes/edge
- Fluid Compute: https://vercel.com/docs/fluid-compute
- Vercel Storage: https://vercel.com/docs/storage
- Vercel Cron Jobs: https://vercel.com/docs/cron-jobs
- Vercel Edge Config: https://vercel.com/docs/edge-config
- Vercel Pro pricing: https://vercel.com/pricing
- Vercel Domains: https://vercel.com/docs/domains
- AI SDK v6: https://ai-sdk.dev/docs
