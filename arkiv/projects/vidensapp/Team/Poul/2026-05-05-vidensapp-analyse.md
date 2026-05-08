---
dato: 2026-05-05
forfatter: Poul
emne: Vidensapp i Evernote-stil - behovs-analyse + arkitekturforslag
status: leveret
relateret-brief: Team Inbox/2026-05-05-vidensapp-evernote-style.md
relateret-resume: Min Inbox/2026-05-05-vidensapp-pouls-analyse.md
---

# Analyse: Vidensapp i Evernote-stil

Brief: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team Inbox/2026-05-05-vidensapp-evernote-style.md`
Standarder: `/Users/fgd/.claude/CLAUDE.md` (API & Web Service Development Standards)
Konventioner: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/_konventioner.md`

## Spørgsmål

1. Hvilken arkitektur, datamodel og sikkerhedsmodel matcher en Next.js + Neon-baseret vidensapp med multi-user sharing?
2. Hvor i stakken hører AI-berigelsen hjemme, og hvilken model+orchestrator skal vi vælge?
3. Hvilke roller mangler teamet for at gå fra analyse til kørende MVP?
4. Hvad er den mindste leverance der løser FGD's problem (MVP) og hvad er en sund vej til delt v1?

## Resultat (TL;DR)

Anbefalet stack: **Next.js 15 App Router (PWA via Serwist) + Neon Postgres med pgvector + Neon RLS Authorize + Clerk + Inngest + Anthropic-modeller (Haiku 4.5 til berignings-batch, Sonnet 4.5 til komplekse opgaver) deployet på Vercel**. Authorization defense-in-depth: API-laget tjekker per-resource membership, Postgres RLS via `pg_session_jwt` + Clerk JWT er sidste forsvarslinje mod BOLA. Søgning: hybrid (Postgres FTS `tsvector` + pgvector via RRF) — start med FTS i MVP, tilføj pgvector som "nice-to-have" når der er volumen at retfærdiggøre det. Berigelsesjobs kører som Inngest step-functions for retries, dødt-bogstavs-kø og observability — undgå Vercels 60s/300s-timeouts. Scope: **medium** (4-6 ugers fokuseret arbejde til v1). Roller der skal ansættes: **frontend-udvikler (Next.js/PWA)**, **backend/Neon-specialist (DB, RLS, API-kontrakter)**, **AI-pipeline-designer**.

---

## 1. Behovs-analyse

### Aktører

| Aktør | Beskrivelse |
|---|---|
| **FGD (ejer)** | Opretter notebooks og noter, deler notebooks med kunder, raffinerer AI-output, søger på tværs |
| **Kunde (ekstern bruger)** | Inviteret til ét eller flere notebooks; har rolle: læser eller redaktør; ser kun det de har adgang til |
| **System (AI-worker)** | Berigelsesjobs trigget ved import/ændring; producerer titel, resumé, tags, link-forslag, embeddings |

### Primære brugerflows

1. **Opret note manuelt** — FGD åbner notebook, opretter note, skriver indhold, gemmer (auto-save). Berigelses-job triggres async ved første save og ved efterfølgende ændringer over en tærskel.
2. **Importér eksisterende txt-noter** — FGD vælger mappe / drag-and-drop / CLI-script. Hver fil → ny note (status: "imported"). Berigelses-jobs spawnes batch-vis. UI viser fremdrift.
3. **AI-raffinering** — FGD åbner note, ser AI-foreslået titel/resumé/tags som "udkast"; accepterer, redigerer eller forkaster pr. felt. Manuel "kør berignelse igen" knap pr. note.
4. **Søg** — global søgning på tværs af alle notebooks brugeren har adgang til. Filtrér på notebook, tags, dato. Resultater viser snippet med højlyst match.
5. **Del notebook med kunde** — FGD inviterer kundes email, vælger rolle (viewer/editor); kunden får email-link, opretter konto/logger ind, ser kun delte notebooks.
6. **Mobil read-mostly** — primær use case fra mobil er "slå op" + læg evt. en quick-note. Fuld redigering må gerne være desktop-først.

### Use cases (kort liste)

- UC-01 Opret notebook (ejer)
- UC-02 Opret/rediger/slet note
- UC-03 Importér txt-batch
- UC-04 Auto-berigelse trigges
- UC-05 Manuel berigelse + acceptér forslag
- UC-06 Global søgning (FTS + senere semantisk)
- UC-07 Del notebook + administrér roller
- UC-08 Versioner/diff på note (rollback)
- UC-09 Eksportér notebook (markdown-zip)
- UC-10 Audit log for ændringer på delte noter

### Ikke-funktionelle krav

| Krav | Mål |
|---|---|
| **Sikkerhed** | OWASP API Top 10 compliance; BOLA umuligt pga. RLS + API-tjek; alle writes idempotente; secrets i Vercel env / Neon secret |
| **Performance** | P95 < 300 ms for liste/søg op til 10k noter; cold-start tolerable (<1s) — Neon scale-to-zero accepteres for v1 |
| **Mobil** | Responsivt UI, PWA installerbar, basal offline-cache for læsning af noter besøgt seneste 7 dage |
| **Tilgængelighed** | 99.5% (Vercel + Neon SLA dækker dette i praksis for intern brug) |
| **Observability** | OpenTelemetry traces, struktur-logs uden PII, fejl-tracking (Sentry) |
| **Skalering** | Datamodel + queries skal ikke skulle omskrives til 10k noter / 100 brugere |
| **Kost-loft v1** | < 50 USD/md (Vercel Hobby/Pro + Neon Launch + Clerk free + Inngest free + Anthropic API) |

---

## 2. Foreslået arkitektur

### Komponenter

```
                        ┌─────────────────┐
                        │ Browser / PWA   │  Next.js 15 App Router
                        │ (desktop+mobil) │  + Serwist (offline cache)
                        └────────┬────────┘
                                 │ HTTPS, JWT i cookie/Auth header
                                 ▼
                  ┌────────────────────────────┐
                  │ Vercel Edge / Node runtime │
                  │  - React Server Components │
                  │  - Server Actions (writes) │
                  │  - Route Handlers (API)    │
                  │  - Middleware (auth gate)  │
                  └──────┬──────────┬──────────┘
            authn JWT    │          │  trigger event
                         │          ▼
                  ┌──────┴──────┐ ┌──────────────────────┐
                  │   Clerk     │ │ Inngest Cloud        │
                  │ (auth, JWT  │ │ (event bus +         │
                  │  templates) │ │  step functions)     │
                  └──────┬──────┘ └──────┬───────────────┘
                         │               │ HTTPS callback til
                         │ JWT i request │ /api/inngest
                         ▼               ▼
            ┌────────────────────────────────────────────┐
            │ Neon Postgres (eu-central-1 eller eu-west) │
            │  - pg_session_jwt (RLS Authorize)          │
            │  - pgvector (embeddings)                   │
            │  - tsvector + GIN (FTS)                    │
            │  - logical schemas: app, audit             │
            │  - branches: prod / preview-* / dev        │
            └────────────────────────────────────────────┘
                                 ▲
                                 │ enrich-job henter note,
                                 │ skriver enrichment-felter
                                 │
                  ┌──────────────┴──────────┐
                  │ Anthropic API           │
                  │  - Haiku 4.5 (batch)    │
                  │  - Sonnet 4.5 (kompleks)│
                  │  - voyage-3 embeddings  │
                  │    (eller OpenAI text-  │
                  │    embedding-3-small)   │
                  └─────────────────────────┘
```

### Data-flow: import + auto-berigelse

```
1. Bruger uploader 50 txt-filer (multipart) → Route Handler /api/import
2. Route Handler (auth ✓, kvote ✓) opretter rows i `notes` (status='imported')
   og `enrichment_jobs` (status='queued') i én transaktion. Returnerer 202.
3. Route Handler emit'er Inngest event 'note/enrichment.requested' pr. note.
4. Inngest worker (i Inngest Cloud) trigger /api/inngest endpoint per event:
   step 1: hent note-tekst med RLS-bypass service-role
   step 2: kald Anthropic Haiku 4.5 → titel + resumé + tags
   step 3: generér embedding (voyage-3) → pgvector
   step 4: forsøg notebook-link forslag (similarity-search top-3)
   step 5: skriv resultat til `notes` + `note_enrichments`, marker job 'done'
5. Frontend henter fremdrift via Server-Sent Events fra /api/jobs/stream
   (eller pull hvert 3. sek hvis SSE er overkill).
```

### Data-flow: read (søg)

```
Browser → RSC page /search?q=foo
  → Server Component kalder DB med Clerk JWT (via @neondatabase/serverless
    HTTP fetch + Authorization header)
  → Postgres RLS validerer JWT, ser kun rows hvor user er medlem
  → Hybrid query: tsvector match + (senere) pgvector cosine,
    flettet med Reciprocal Rank Fusion (RRF) → top 20
  → returnerer JSX med snippets + highlight
```

### Designvalg (med begrundelse)

| Valg | Hvorfor |
|---|---|
| **App Router (RSC)** | Native data-fetch på server, mindre JS til klient, korrekt cache-revalidation pr. note efter Server Action |
| **Server Actions til writes**, **Route Handlers til import + Inngest callback + webhooks** | Clean separation; Server Actions giver gratis revalidation, Route Handlers håndterer non-human callers |
| **Inngest** (frem for QStash eller egen BullMQ) | AI-pipeline med flere trin → step-functions, automatiske retries, dødt-bogstavs-kø, dashboard. Free tier (50k step-runs/md) dækker langt over v1-volumen. Ingen Redis at drifte |
| **Vercel + Neon (samme region: Frankfurt)** | Lav latency mellem compute og DB; begge har solid free/Pro tier; Vercel deployment integration med Neon branches via Neon-Vercel integration |
| **Clerk + Neon RLS Authorize** | Clerk er hurtigste auth at få i drift på Next.js (~30 min), Neons egen `pg_session_jwt` er bygget specifikt til denne kombo, RLS giver defense-in-depth mod BOLA |
| **Serwist** (PWA) | next-pwa er ikke-vedligeholdt; Serwist er den aktive arvtager og fungerer med App Router |

---

## 3. Datamodel-skitse

Schema: `app`. Audit-tabeller i schema `audit`. Alle PK'er er UUIDv7 (tids-sorterbar, opaque udadtil — opfylder API-standardens "never expose database primary keys directly").

```sql
-- Brugere kommer fra Clerk, men vi spejler en minimal record
-- via webhook for at kunne joine.
CREATE TABLE app.users (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  clerk_user_id   TEXT UNIQUE NOT NULL,
  email           CITEXT UNIQUE NOT NULL,
  display_name    TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX users_clerk_idx ON app.users (clerk_user_id);

CREATE TABLE app.notebooks (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_id        UUID NOT NULL REFERENCES app.users(id) ON DELETE RESTRICT,
  name            TEXT NOT NULL,
  description     TEXT,
  customer_label  TEXT,
  archived_at     TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX notebooks_owner_idx ON app.notebooks (owner_id) WHERE archived_at IS NULL;

CREATE TYPE app.notebook_role AS ENUM ('owner','editor','viewer');

CREATE TABLE app.notebook_members (
  notebook_id     UUID NOT NULL REFERENCES app.notebooks(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES app.users(id) ON DELETE CASCADE,
  role            app.notebook_role NOT NULL,
  invited_by      UUID REFERENCES app.users(id),
  invited_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_at     TIMESTAMPTZ,
  PRIMARY KEY (notebook_id, user_id)
);
CREATE INDEX nbm_user_idx ON app.notebook_members (user_id);

CREATE TABLE app.notebook_invites (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  notebook_id     UUID NOT NULL REFERENCES app.notebooks(id) ON DELETE CASCADE,
  email           CITEXT NOT NULL,
  role            app.notebook_role NOT NULL,
  token_hash      TEXT NOT NULL,
  expires_at      TIMESTAMPTZ NOT NULL,
  invited_by      UUID NOT NULL REFERENCES app.users(id),
  invited_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  consumed_at     TIMESTAMPTZ
);
CREATE INDEX invites_email_idx ON app.notebook_invites (email) WHERE consumed_at IS NULL;

CREATE TABLE app.notes (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  notebook_id     UUID NOT NULL REFERENCES app.notebooks(id) ON DELETE CASCADE,
  created_by      UUID NOT NULL REFERENCES app.users(id),
  title           TEXT,
  body_md         TEXT NOT NULL DEFAULT '',
  body_search     TSVECTOR GENERATED ALWAYS AS
                    (to_tsvector('simple',
                       coalesce(title,'') || ' ' || coalesce(body_md,'')))
                    STORED,
  status          TEXT NOT NULL DEFAULT 'draft',
  source          TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX notes_notebook_idx ON app.notes (notebook_id, updated_at DESC);
CREATE INDEX notes_search_idx ON app.notes USING GIN (body_search);

CREATE TABLE app.note_versions (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  body_md         TEXT NOT NULL,
  title           TEXT,
  edited_by       UUID NOT NULL REFERENCES app.users(id),
  edited_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX nv_note_time_idx ON app.note_versions (note_id, edited_at DESC);

CREATE TABLE app.tags (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_id        UUID NOT NULL REFERENCES app.users(id),
  name            CITEXT NOT NULL,
  UNIQUE (owner_id, name)
);

CREATE TYPE app.tag_source AS ENUM ('user','ai_suggested','ai_accepted');

CREATE TABLE app.note_tags (
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  tag_id          UUID NOT NULL REFERENCES app.tags(id) ON DELETE CASCADE,
  source          app.tag_source NOT NULL DEFAULT 'user',
  PRIMARY KEY (note_id, tag_id)
);

CREATE TABLE app.note_enrichments (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  model           TEXT NOT NULL,
  prompt_version  TEXT NOT NULL,
  suggested_title TEXT,
  summary         TEXT,
  suggested_tags  TEXT[],
  related_note_ids UUID[],
  raw_response    JSONB,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_by     UUID REFERENCES app.users(id),
  accepted_at     TIMESTAMPTZ
);
CREATE INDEX enr_note_idx ON app.note_enrichments (note_id, created_at DESC);

CREATE TABLE app.note_embeddings (
  note_id         UUID PRIMARY KEY REFERENCES app.notes(id) ON DELETE CASCADE,
  model           TEXT NOT NULL,
  dim             INT  NOT NULL,
  embedding       vector(1024) NOT NULL,
  generated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX note_emb_hnsw_idx ON app.note_embeddings
  USING hnsw (embedding vector_cosine_ops);

CREATE TYPE app.job_status AS ENUM ('queued','running','done','failed','cancelled');

CREATE TABLE app.enrichment_jobs (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  inngest_run_id  TEXT,
  status          app.job_status NOT NULL DEFAULT 'queued',
  attempts        INT NOT NULL DEFAULT 0,
  error           TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  finished_at     TIMESTAMPTZ
);
CREATE INDEX ej_note_idx ON app.enrichment_jobs (note_id, created_at DESC);
CREATE INDEX ej_status_idx ON app.enrichment_jobs (status) WHERE status IN ('queued','running');

CREATE TABLE audit.events (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  actor_user_id   UUID,
  action          TEXT NOT NULL,
  resource_type   TEXT NOT NULL,
  resource_id     UUID NOT NULL,
  payload         JSONB,
  trace_id        TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX audit_resource_idx ON audit.events (resource_type, resource_id, created_at DESC);
```

### Pgvector-placering

- **Hvor**: separat `note_embeddings`-tabel (1-1 til `notes`).
- **Hvorfor separat**: kan udskifte embedding-model uden at røre notes-rækker; null-able adoption (ikke alle noter har embedding endnu); CRUD på note triggrer ikke unødig vector-index update.
- **Index**: HNSW med `vector_cosine_ops`. Gode startparametre: `m=16, ef_construction=64`.
- **Dim**: 1024 (voyage-3) eller 1536 (OpenAI text-embedding-3-small). Anbefaling: voyage-3 for kvalitet/pris.

---

## 4. Sikkerhedsmodel

### Auth-valg: **Clerk + Neon RLS Authorize**

| Kriterium | Auth.js v5 | Clerk | Neon Auth (Stack-baseret) |
|---|---|---|---|
| Setup-tid (App Router) | Mellem (config-tung) | Hurtigst (~30 min) | Hurtig, tæt på Neon |
| Native Neon RLS-integration | Ingen — du skal selv generere JWT | **Officiel Neon-guide** | Officielt understøttet |
| MFA / passkeys / social out-of-the-box | Skal selv hooke | Indbygget | Indbygget |
| Pris v1 (få brugere) | Gratis | Gratis under 10k MAU | Gratis (beta) |
| Vendor-lock | Lav | Mellem | Høj (Neon-eksklusiv) |
| Modenhed | Auth.js team flytter til Better Auth (sept 2025) | Modent, A-runde | Beta-pakker |

**Anbefaling**: **Clerk**. Eneste reelle ulempe er vendor-lock, men compensated af modenhed, hastighed og en officiel Neon RLS-guide.

### Token-strategi

- Clerk udsteder kort-levede session JWTs (~60 sek), automatisk refreshet.
- Konfigurér Clerk JWT Template "neon" så claim-strukturen matcher hvad `pg_session_jwt` forventer.
- `alg`: pinned i Clerk (RS256). `iss`/`aud`/`exp`/`nbf` valideres af `pg_session_jwt` mod JWKS-URL fra Clerk.
- For mTLS/DPoP: ikke nødvendigt v1.

### Authorization: defense-in-depth

**Lag 1 — API/Server Actions**: Inden hver write tjekkes membership eksplicit.

**Lag 2 — Postgres RLS**: hvis lag 1 svigter, RLS blokerer:

```sql
ALTER TABLE app.notes ENABLE ROW LEVEL SECURITY;

CREATE POLICY notes_select ON app.notes FOR SELECT
  USING (
    notebook_id IN (
      SELECT nm.notebook_id FROM app.notebook_members nm
      WHERE nm.user_id = (
        SELECT id FROM app.users WHERE clerk_user_id = auth.user_id()
      )
    )
  );
```

**Lag 3 — Property-level (BOPLA)**: Schema-validér requests med Zod og `.strict()` for at afvise ukendte felter.

### OWASP API Top 10 — status pr. design

| Risiko | Mitigation |
|---|---|
| API1 BOLA | RLS + per-resource API-check |
| API2 Broken Auth | Clerk + JWT pinning |
| API3 BOPLA | Zod strict + server-side felt-vask |
| API4 Unrestricted Resource | Vercel WAF + Inngest concurrency-limit + per-bruger note-quota |
| API5 Broken Function-Level Auth | Rolle-rank check på alle mutating endpoints |
| API6 Unrestricted Sensitive Business Flow | Rate-limit på /api/import |
| API7 SSRF | Aldrig fetch URL fra note-body server-side i v1 |
| API8 Misconfig | Vercel preview ≠ prod; secrets via Vercel env; CORS allow-list |
| API9 Improper Inventory | OpenAPI-kontrakt for /api/* committet i repo; CI lint med Spectral |
| API10 Unsafe Consumption | Anthropic-respons valideres mod Zod-schema før den skrives til DB |

### Errors

Alle API-fejl returneres som `application/problem+json` (RFC 9457).

### Idempotency

`POST /api/notes`, `POST /api/import`, `POST /api/notebooks/:id/invite` accepterer header `Idempotency-Key` (UUID).

### Observability

OpenTelemetry SDK i Next.js. Tracer: hver request → spans for DB-queries og Anthropic-kald. W3C `traceparent` propageres. Logs uden PII.

### CORS

Eksplicit allow-list: kun `https://app.fgdconsulting.se` (+ `http://localhost:3000` i dev).

---

## 5. AI-berignings-pipeline

### Berigelser ved import / ændring

| Berigelse | Model | Kostgrad | Værdi |
|---|---|---|---|
| Titel-forslag | Haiku 4.5 | Lav | Høj |
| Resumé (2-3 sætninger) | Haiku 4.5 | Lav | Høj |
| Tags (3-7 stk) | Haiku 4.5 | Lav | Høj |
| Foreslåede notebook-links | Haiku 4.5 + similarity-search top-5 | Mellem | Mellem |
| Embedding | voyage-3 | Lav | Lav v1, høj senere |
| "Relaterede noter" | Beregnet ud fra embedding | Lav | Mellem |
| Komplekse opgaver (rewrite, struktur) | Sonnet 4.5 | Mellem | Lav v1 (kun manuel) |

### Model-valg (konkret)

- **Haiku 4.5**: Default for batch-berigelse.
- **Sonnet 4.5**: Manuel rewrite/outline/inkonsistens-tjek.
- **Embeddings**: voyage-3 (1024 dim).

### Orchestrator: **Inngest**

- Step-functions: hvert step idempotent og retry-bart.
- Concurrency-limit pr. bruger: max 5 parallelle jobs.
- Dødt-bogstavs-kø: jobs der fejler 3 gange → marker `failed`.

### Job-livscyklus

```
queued → running → done
                 → failed (retries opbrugt)
                 → cancelled (bruger redigerede note → ny job spawned)
```

### Manuel raffinerings-UI flow

1. Bruger åbner note. Hvis `note_enrichments` har ikke-accepterede forslag, vis dem som "udkast" badges på felter.
2. Pr. felt: knapperne **Acceptér** / **Rediger** / **Forkast**.
3. Knap **"Kør AI igen"** med valg: full enrichment / kun tags / kun resumé.
4. Knap **"Forklar dette afsnit" / "Foreslå relaterede noter"** → Sonnet, vises i sidebar.

---

## 6. Søgning

### Anbefaling: **Hybrid (FTS i MVP, pgvector tilføjet senere) med RRF**

- **MVP under 100 noter**: FTS via `tsvector` + GIN-index er nok.
- **Når korpuset vokser**: tilføj pgvector + HNSW.
- **Hybrid via RRF**: Reciprocal Rank Fusion for at flette FTS + vector.

### SQL-skitse for hybrid query (v1.x)

```sql
WITH fts AS (
  SELECT id, ts_rank_cd(body_search, plainto_tsquery('simple', $1)) AS s,
         row_number() OVER (ORDER BY ts_rank_cd(body_search, plainto_tsquery('simple', $1)) DESC) AS r
  FROM app.notes
  WHERE body_search @@ plainto_tsquery('simple', $1)
  LIMIT 50
),
vec AS (
  SELECT n.id,
         1 - (e.embedding <=> $2::vector) AS s,
         row_number() OVER (ORDER BY e.embedding <=> $2::vector) AS r
  FROM app.notes n JOIN app.note_embeddings e ON e.note_id = n.id
  ORDER BY e.embedding <=> $2::vector
  LIMIT 50
)
SELECT id, COALESCE(1.0/(60+f.r),0) + COALESCE(1.0/(60+v.r),0) AS rrf
FROM fts f FULL OUTER JOIN vec v USING (id)
ORDER BY rrf DESC LIMIT 20;
```

---

## 7. Import-pipeline til de eksisterende txt-noter

### Anbefalet flow: **UI-upload (drop-zone) som primær, CLI som fallback**

**UI-upload**: Næste-app side `/import` med drag-and-drop af mappe (HTML5 `webkitdirectory`). Klient batcher i chunks á 20, POST'er multipart til `/api/import` med `Idempotency-Key` pr. batch.

Server (Route Handler):
1. Validér filtype (`.txt`, `.md`), max 1 MB pr. fil, max 100 filer pr. request.
2. Validér target notebook membership (≥ editor).
3. Insert i én transaktion: rows i `notes` (status='imported') + `enrichment_jobs` (status='queued').
4. Emit Inngest event `note/enrichment.requested` pr. note.
5. Return 202 Accepted med `import_id` + array af note-IDs.

UI poller `/api/import/:id/status` og viser fremdrift.

**CLI-script** (til de første massive imports): Lille Node-script i `scripts/import-txt.ts`. Læser mappe rekursivt, kalder samme `/api/import` endpoint med personal access token (Clerk machine-to-machine). Genbruger samme pipeline.

### Filnavn → note-mapping

- Default: filnavn (uden extension) bliver initial titel.
- Mappe-struktur kan mappes til notebook: `kunde-x/note1.txt` → notebook "kunde-x".

---

## 8. Manglende roller — Lailas to-do

### Rolle 1: **Frontend-udvikler (Next.js + PWA)**

- **Mandat**: Bygge Next.js 15 App Router-app, RSC + Server Actions, design-system (Tailwind + shadcn/ui), responsivt UI, Serwist PWA-setup.
- **Skills**: Next.js 15 App Router, React 19, TypeScript, Tailwind, Serwist, Clerk SDK.
- **Outputs**: kørende UI med alle CRUD-flows + import + søg + share-dialog.

### Rolle 2: **Backend/Neon-specialist**

- **Mandat**: Designe og migrere DB-skema (Drizzle), opsætte Neon RLS Authorize med Clerk JWT-template, OpenAPI 3.1-kontrakt, idempotency-håndtering.
- **Skills**: Postgres, RLS, Neon CLI, Drizzle, OpenAPI 3.1, Zod, OWASP API.
- **Outputs**: migrations-mappe, openapi.yaml, alle API-endpoints implementeret med tests.

### Rolle 3: **AI-pipeline-designer**

- **Mandat**: Designe Inngest step-functions, prompt-engineering for titel/resumé/tags (versionering i `prompt_version`), embedding-strategi, evaluerings-script.
- **Skills**: Inngest, Anthropic SDK, prompt design, embedding-modeller, simpel evals.
- **Outputs**: `lib/ai/enrich.ts` + Inngest functions + prompt-bibliotek + eval-script.

### Eksisterende roller dækker

- **Camilla**: kører migrations, opretter Neon-projekt + branches, owner af `memory/`.
- **Poul**: ad-hoc research.
- **Stefan**: orkestrerer + kvitterer.

### Genbrugs-muligheder (skills/MCP)

- **neon-postgres skill**: backend-rollen skal bruge denne ved opsætning.
- **context7 skill / MCP**: alle udviklere bruger til docs-opslag.
- **excalidraw-diagrams skill**: når arkitektur stabiliserer, generér officielt diagram.

---

## 9. Faseret implementeringsplan

### Fase 0 — Forberedelse

| Leverance | Ejer |
|---|---|
| Lailas rolle-design + ansættelse af 3 nye roller | Laila |
| Neon-projekt oprettet, dev/preview/main branches | Camilla |
| Vercel-projekt oprettet, koblet til Neon-integration | Camilla |
| Clerk-projekt oprettet, JWT-template "neon" konfigureret | Backend-rolle |
| Repo-skelet + CI (lint, typecheck, Spectral) | Backend-rolle |

### Fase 1 — MVP (2-3 uger): "FGD bruger appen alene"

- Auth (Clerk login/signup)
- Notebooks: list / opret / omdøb / arkivér
- Notes: list / opret / rediger / slet (markdown-editor)
- Import: drop-zone + CLI for de eksisterende txt-noter
- AI auto-berigelse: titel + resumé + tags via Inngest + Haiku 4.5
- Manuel raffinering: acceptér/rediger/forkast pr. felt
- Søgning: Postgres FTS (tsvector)
- Mobil-responsivt UI (uden PWA endnu)
- OpenAPI-kontrakt + RFC 9457 fejl + idempotency på import
- Basal observability (Vercel/OTel + Sentry)

### Fase 2 — v1 delt med kunder (2-3 uger)

- RLS Authorize aktiveret
- Notebook sharing: invite via email, roller (owner/editor/viewer), accept-flow
- Audit-log på delte ressourcer
- Rate-limiting på public endpoints
- PWA via Serwist: installerbar, offline-cache
- Versions-historik på noter
- "Relaterede noter" via embedding similarity (pgvector aktiveres her)
- Hybrid-søg (FTS + vector + RRF)
- GDPR: data-export pr. notebook, slet konto-flow

### Fase 3 — Nice-to-have

- Slash-commands i editor
- Real-time co-editing (Y.js + WebSocket)
- Integration: import fra Evernote/Notion-export
- Public read-only share-links
- Mobil push-notifikationer
- Sonnet 4.5 til "rewrite/struktur" som premium-action
- Per-customer custom domain

### Konkret stack-recap

| Lag | Valg |
|---|---|
| Hosting | Vercel (Pro når sharing aktiveres for SLA) |
| DB | Neon Postgres (Launch plan så scale-to-zero kan slås fra på prod-branch) |
| Auth | Clerk |
| ORM | Drizzle |
| Background jobs | Inngest Cloud |
| AI | Anthropic Haiku 4.5 (batch) + Sonnet 4.5 (manuel) |
| Embeddings | voyage-3 (1024 dim) |
| Search | Postgres tsvector + pgvector (HNSW), hybrid via RRF i fase 2 |
| Cache | Next.js indbyggede data cache + `revalidatePath` |
| PWA | Serwist (fase 2) |
| Observability | Vercel OTel-export + Sentry |
| CI | GitHub Actions: typecheck, eslint, Spectral, drizzle-kit check |

---

## 10. Åbne spørgsmål til FGD

1. Eksisterende txt-noter — hvor mange og hvordan struktureret? Mappe-hierarki = notebook-struktur?
2. Kunde-volumen v1: hvor mange kunder forventes i de første 6 måneder?
3. Kunde-onboarding: skal kunder kunne self-signup via invite-link, eller skal FGD oprette dem manuelt?
4. Notebook-deling — er der behov for "subteam" på kunde-side?
5. Domæne: `app.fgdconsulting.se` ok, eller separat?
6. AI-budget: cap per måned?
7. GDPR/privatliv: skal note-indhold ekspliciteret behandles af AI fra day-one, eller skal kunder kunne opt-out pr. notebook?
8. Region: Frankfurt (eu-central-1) ok?
9. Eksisterende OpenAI/Anthropic-konti vi skal bruge, eller opretter Camilla nye?
10. Versions-historik — alle ændringer eller bare "manuelt gemte snapshots"?

---

## Sources

- Neon Auth: https://neon.com/docs/ai/skills/neon-postgres/references/neon-auth.md
- pgvector i Neon Postgres: https://neon.com/docs/extensions/pgvector.md
- Neon Connection Pooling: https://neon.com/docs/connect/connection-pooling.md
- Neon RLS Authorize + Clerk: https://neon.tech/docs/guides/neon-rls-authorize-clerk
- Introducing Neon RLS: https://neon.com/blog/introducing-neon-authorize
- pg_session_jwt extension: https://github.com/neondatabase/pg_session_jwt
- Next.js Server Actions vs Route Handlers: https://dev.to/whoffagents/nextjs-15-server-actions-vs-route-handlers-when-to-use-each-i-got-this-wrong-for-3-months-49hm
- Inngest vs Trigger vs Vercel Cron: https://www.hashbuilds.com/articles/next-js-background-jobs-inngest-vs-trigger-dev-vs-vercel-cron
- Hybrid search med pgvector + tsvector: https://www.tigerdata.com/blog/postgresql-hybrid-search-using-pgvector-and-cohere
- Bedste auth library for Next.js 2026: https://blog.logrocket.com/best-auth-library-nextjs-2026/
- Clerk + Neon RLS demo: https://github.com/neondatabase/clerk-nextjs-neon-rls
- Next.js 16 PWA med Serwist: https://blog.logrocket.com/nextjs-16-pwa-offline-support/
