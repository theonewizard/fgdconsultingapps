---
dato: 2026-05-05
forfatter: Poul
emne: Vidensapp - v2 (versions-tjekket, FGD-svar indarbejdet, miljø-separation)
status: leveret
afløser: Team/Poul/2026-05-05-vidensapp-analyse.md (v1)
relateret-brief: Team Inbox/2026-05-05-vidensapp-evernote-style.md
relateret-svar: Team Inbox/2026-05-05 - Svar på Pouls spørgsmål
---

# Vidensapp v2 — Analyse

Brief: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team Inbox/2026-05-05-vidensapp-evernote-style.md`
Standarder: `/Users/fgd/.claude/CLAUDE.md` (API & Web Service Development Standards)
Konventioner: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/_konventioner.md`

## Ændringer fra v1 → v2

**Datum verificering: 2026-05-05** — alle framework-versioner valideret mod aktuelle sources.

- **Stack-versioner verificeret**: Next.js 16.2 (ikke 15), Clerk 7.2, React 19.2, Drizzle 0.45.2, pgvector 0.8.2, Tailwind v4, voyage-3.5 (ikke voyage-3), Sonnet 4.6 (ikke 4.5), Haiku 4.5 (bekræftet), Inngest v4.

- **Datamodel udvidet**: nye tabeller `app.organizations` (tenant isolation), `app.organization_members`, `app.organization_invites`. Tabel `notebooks` får `organization_id` (nullable) for støtte til subteams på kunde-side. Felt `ai_enrichment_enabled BOOLEAN DEFAULT true` på `notebooks` for GDPR opt-out per notebook. RLS-policies udvidet til at dække både direkte notebook-membership OG organisation-membership.

- **Sikkerhedsmodel udvidet**: Databehandleraftale-tabel pr. leverandør (Neon, Vercel, Clerk, Anthropic, Inngest, Voyage AI). Encryption-at-rest dokumentation. Magic-link invite-flow: FGD opretter kunder manuelt via admin-panel (ikke self-signup i MVP). Audit-log pr. organisation (ikke blot per notebook). Anthropic Zero Data Retention policy aktiveret på prod-workspace.

- **AI-budget-kalibrering**: 500 DKK/md (~70 USD) rækker til 23.000+ noter/md på basis af Haiku 4.5 batch-pris — rigeligt headroom for v1. Per-workspace spend-caps: prod $50, test $15, dev $5. Inngest concurrency-limit skaleret til 3 parallelle jobs per organisation (ikke globalt).

- **Versions-historik debouncing**: 30 sekunder inaktivitet ELLER 200 karakters akkumuleret diff før snapshot gemmes. LZ4-komprimering estimerer ~150 MB/år pr. 10k noter.

- **Domæne**: alle eksempler bruger `docs.fgdconsulting.se` (ikke `app.fgdconsulting.se`) — kan køres bag corporate proxy/VPN.

- **Import-pipeline raffinering**: txt-noter har ingen mappe-struktur i filsystemet → alle landet i "Inbox"-notebook efter import + post-import organiserings-flow hvor AI foreslår notebook-tildeling pr. note.

- **Ny hovedsektion "Miljø-strategi"**: 3 separate Neon-projekter (dev, test, prod — ikke kun branches). Vercel custom environments (kræver Pro plan). Clerk dev+prod instances (test deler dev). Anthropic 3 separate workspaces. Inngest 3 environments. Secrets-mapping dokumenteret pr. miljø. Promotion-pipeline med manuel approval før prod-pushes. DB-migration sikkerhedsregler klare (downtime-plan, ROLLBACK-test).

- **Scope-justering**: 5–7 uger total (var 4–6 uger).

---

## 1. Behovs-analyse

*(Identisk med v1; gentages for kontekst)*

### Aktører

| Aktør | Beskrivelse |
|---|---|
| **FGD (ejer)** | Opretter notebooks og noter, deler notebooks med kunder, raffinerer AI-output, søger på tværs |
| **Kunde (ekstern bruger)** | Inviteret til ét eller flere notebooks; kan være medlem af organisation; har rolle: læser, redaktør eller admin; ser kun det de har adgang til |
| **System (AI-worker)** | Berigelsesjobs trigget ved import/ændring; producerer titel, resumé, tags, link-forslag, embeddings |

### Primære brugerflows

1. **Opret note manuelt** — FGD åbner notebook, opretter note, skriver indhold, gemmer (auto-save). Berigelses-job triggres async ved første save og ved efterfølgende ændringer over en tærskel.
2. **Importér eksisterende txt-noter** — FGD vælger mappe / drag-and-drop. Hver fil → ny note i "Inbox"-notebook (status: "imported"). Berigelses-jobs spawnes batch-vis. AI foreslår notebook-tildeling. UI viser fremdrift.
3. **AI-raffinering** — FGD åbner note, ser AI-foreslået titel/resumé/tags som "udkast"; accepterer, redigerer eller forkaster pr. felt. Manuel "kør berignelse igen" knap pr. note.
4. **Søg** — global søgning på tværs af alle notebooks brugeren har adgang til (inkl. organisation-medlemskab). Filtrér på notebook, tags, dato. Resultater viser snippet med højlyst match.
5. **Del notebook med kunde** — FGD inviterer kundes email, vælger rolle (viewer/editor/admin); kunden får magic-link, logger ind, ses kun delte notebooks + evtl. organisation.
6. **Mobil read-mostly** — primær use case fra mobil er "slå op" + evt. quick-note. Fuld redigering desktop-først.
7. **Subteam-organisering (fase 2)** — Kunde kan oprette interne teams inden for sin organisation; tildele notebooks til teams; administrere team-medlemskab.

### Use cases (udvidet)

- UC-01 Opret notebook (ejer eller organisation-admin)
- UC-02 Opret/rediger/slet note
- UC-03 Importér txt-batch → Inbox-notebook
- UC-04 Auto-berigelse trigges (opt-out pr. notebook)
- UC-05 Manuel berigelse + acceptér forslag
- UC-06 Global søgning (FTS + senere semantisk)
- UC-07 Del notebook + administrér roller
- UC-08 Versioner/diff på note (rollback, debouncing)
- UC-09 Eksportér notebook (markdown-zip)
- UC-10 Audit log for ændringer på delte noter + organisationer
- UC-11 [Fase 2] Opret organisation + invitér members
- UC-12 [Fase 2] Subteam-styring inden for organisation

### Ikke-funktionelle krav

| Krav | Mål |
|---|---|
| **Sikkerhed** | OWASP API Top 10 compliance; BOLA umuligt pga. RLS + API-tjek; alle writes idempotente; secrets i Vercel env / Neon secret; databehandleraftaler dokumenterede |
| **Performance** | P95 < 300 ms for liste/søg op til 50k noter; cold-start tolerable (<1s) |
| **Mobil** | Responsivt UI, PWA installerbar, basal offline-cache for læsning af noter besøgt seneste 7 dage |
| **Tilgængelighed** | 99.5% (Vercel + Neon SLA) |
| **Observability** | OpenTelemetry traces, struktur-logs uden PII, fejl-tracking (Sentry) |
| **GDPR** | Opt-out AI per notebook; data-export pr. organisation; slet-konto flow; databehandleraftaler |
| **Kost-loft v1** | < 50 USD/md eller 500 DKK/md |

---

## 2. Foreslået arkitektur (version 2.0)

### Komponenter

```
                        ┌─────────────────┐
                        │ Browser / PWA   │  Next.js 16 App Router
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
                  │   Clerk     │ │ Inngest v4           │
                  │ (auth, JWT  │ │ (event bus +         │
                  │  templates) │ │  step functions)     │
                  └──────┬──────┘ └──────┬───────────────┘
                         │               │ HTTPS callback til
                         │ JWT i request │ /api/inngest
                         ▼               ▼
            ┌────────────────────────────────────────────────────────┐
            │ Neon Postgres — 3 projekter (dev, test, prod)          │
            │  - Frankfurt region (eu-central-1)                     │
            │  - pg_session_jwt (RLS Authorize)                      │
            │  - pgvector 0.8.2 (embeddings, HNSW)                   │
            │  - tsvector + GIN (FTS)                                │
            │  - LZ4-compression for version-historik                │
            │  - logical schemas: app, audit, sys                    │
            │  - Prod: scale-to-zero OFF, read-only replicas         │
            │  - Test/Dev: scale-to-zero ON                          │
            └────────────────────────────────────────────────────────┘
                                 ▲
                                 │ enrich-job henter note,
                                 │ skriver enrichment + org-audit
                                 │
                  ┌──────────────┴──────────────────┐
                  │ Anthropic API (3 workspaces)    │
                  │  - Prod: Haiku 4.5 + Sonnet 4.6 │
                  │  - Test: samme (delt credentials)
                  │  - Dev: samme                   │
                  │  - voyage-3.5 embeddings        │
                  │  - Zero Data Retention (prod)   │
                  └─────────────────────────────────┘
```

### Data-flow: import + auto-berigelse

```
1. Bruger uploader 50 txt-filer (multipart) → Route Handler /api/import
2. Route Handler (auth ✓, kvote ✓, org-medlemskab ✓) opretter rows:
   - `notes` (notebook_id → "Inbox", status='imported')
   - `enrichment_jobs` (status='queued', organization_id set)
   - `note_import_suggestions` (foreslåede notebooks pr. note)
   i én transaktion. Returnerer 202.
3. Route Handler emit'er Inngest event 'note/enrichment.requested' pr. note.
4. Inngest worker (Inngest v4 Cloud) trigger /api/inngest endpoint per event:
   step 1: hent note-tekst + organisation.ai_enrichment_enabled flag med RLS-bypass service-role
   step 2: hvis enabled: kald Anthropic Haiku 4.5 → titel + resumé + tags
   step 3: generér embedding (voyage-3.5) + foreslåede notebook-links
   step 4: skriv resultat til `notes` + `note_enrichments`, marker job 'done'
   step 5: log til `audit.events` (organization_id, action='enrichment_completed')
5. Frontend henter fremdrift via Server-Sent Events fra /api/jobs/stream
```

### Data-flow: read (søg)

```
Browser → RSC page /search?q=foo
  → Server Component kalder DB med Clerk JWT (via @neondatabase/serverless HTTP fetch)
  → Postgres RLS validerer JWT, ser kun rows hvor user er medlem af notebook
    ELLER medlem af organisation der ejer notebook
  → Hybrid query: tsvector match + (senere) pgvector cosine, RRF → top 20
  → returnerer JSX med snippets + highlight
```

### Designvalg (med begrundelse)

| Valg | Hvorfor |
|---|---|
| **Next.js 16 (ikke 15)** | Nyere stabilitet, React 19.2, bedre SSR-performance, serverless-friendly |
| **App Router (RSC)** | Native data-fetch på server, mindre JS til klient, korrekt cache-revalidation |
| **Inngest v4** | Step-functions, retries, DLQ, dashboard; free tier 50k/md; ingen Redis |
| **3 Neon-projekter (dev/test/prod)** | Isolering af miljøer, separate RLS-politikker, promotion-safety |
| **Clerk 7.2** | Hurtigst at få i drift (~30 min), native JWT-template "neon", MFA built-in |
| **Voyage-3.5** | Bedre quality end voyage-3, lavere kostgrad end OpenAI, 1024 dim |
| **Sonnet 4.6** | Når FGD manuelt anmoder: kompleks rewrite, struktur-analyse |
| **Haiku 4.5** | Batch-berigelse, høj quality/pris-ratio, 500 DKK/md ≈ 23k+ noter |
| **Magic-link invites** | FGD styrer customer-onboarding manuelt (sikkerhed + compliance); ingen self-signup i v1 |
| **Organisation-tabel** | Støtte til subteams fra dag-1 (designs i v1, feature activates v2) |

---

## 3. Datamodel (v2 — udvidet)

Schema: `app`. Audit i `audit`. Alle PK'er er UUIDv7 (tidssorterbar, opaque).

```sql
-- USERS (fra Clerk webhook)
CREATE TABLE app.users (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  clerk_user_id   TEXT UNIQUE NOT NULL,
  email           CITEXT UNIQUE NOT NULL,
  display_name    TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX users_clerk_idx ON app.users (clerk_user_id);

-- ORGANISATIONS (ny i v2) — tenant isolation
CREATE TABLE app.organizations (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_id        UUID NOT NULL REFERENCES app.users(id) ON DELETE RESTRICT,
  name            TEXT NOT NULL,
  ai_enrichment_enabled BOOLEAN NOT NULL DEFAULT true,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX orgs_owner_idx ON app.organizations (owner_id);

-- ORGANISATION MEMBERS (ny i v2)
CREATE TABLE app.organization_members (
  organization_id UUID NOT NULL REFERENCES app.organizations(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES app.users(id) ON DELETE CASCADE,
  role            TEXT NOT NULL DEFAULT 'member', -- 'owner', 'admin', 'member'
  invited_by      UUID REFERENCES app.users(id),
  invited_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_at     TIMESTAMPTZ,
  PRIMARY KEY (organization_id, user_id)
);
CREATE INDEX orgm_user_idx ON app.organization_members (user_id);

-- ORGANISATION INVITES (ny i v2) — magic-link
CREATE TABLE app.organization_invites (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  organization_id UUID NOT NULL REFERENCES app.organizations(id) ON DELETE CASCADE,
  email           CITEXT NOT NULL,
  role            TEXT NOT NULL DEFAULT 'member',
  token_hash      TEXT NOT NULL UNIQUE,
  expires_at      TIMESTAMPTZ NOT NULL DEFAULT now() + '7 days',
  invited_by      UUID NOT NULL REFERENCES app.users(id),
  invited_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  consumed_at     TIMESTAMPTZ
);
CREATE INDEX orginv_email_idx ON app.organization_invites (email) WHERE consumed_at IS NULL;

-- NOTEBOOKS (updated v2) — now optional organization_id
CREATE TABLE app.notebooks (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_id        UUID NOT NULL REFERENCES app.users(id) ON DELETE RESTRICT,
  organization_id UUID REFERENCES app.organizations(id) ON DELETE CASCADE,
  name            TEXT NOT NULL,
  description     TEXT,
  ai_enrichment_enabled BOOLEAN NOT NULL DEFAULT true,
  archived_at     TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX notebooks_owner_idx ON app.notebooks (owner_id) WHERE archived_at IS NULL;
CREATE INDEX notebooks_org_idx ON app.notebooks (organization_id) WHERE archived_at IS NULL;

CREATE TYPE app.notebook_role AS ENUM ('owner','editor','viewer');

-- NOTEBOOK MEMBERS (unchanged)
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

-- NOTEBOOK INVITES (unchanged)
CREATE TABLE app.notebook_invites (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  notebook_id     UUID NOT NULL REFERENCES app.notebooks(id) ON DELETE CASCADE,
  email           CITEXT NOT NULL,
  role            app.notebook_role NOT NULL,
  token_hash      TEXT NOT NULL UNIQUE,
  expires_at      TIMESTAMPTZ NOT NULL DEFAULT now() + '7 days',
  invited_by      UUID NOT NULL REFERENCES app.users(id),
  invited_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  consumed_at     TIMESTAMPTZ
);
CREATE INDEX invites_email_idx ON app.notebook_invites (email) WHERE consumed_at IS NULL;

-- NOTES (updated v2) — explicit ai_enrichment_enabled
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
  ai_enrichment_enabled BOOLEAN NOT NULL DEFAULT true,
  status          TEXT NOT NULL DEFAULT 'draft',
  source          TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX notes_notebook_idx ON app.notes (notebook_id, updated_at DESC);
CREATE INDEX notes_search_idx ON app.notes USING GIN (body_search);

-- NOTE VERSIONS (v2 — LZ4 komprimering)
CREATE TABLE app.note_versions (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  body_md_compressed BYTEA NOT NULL, -- LZ4 compressed
  title           TEXT,
  edited_by       UUID NOT NULL REFERENCES app.users(id),
  edited_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  debounce_count  INT NOT NULL DEFAULT 1
);
CREATE INDEX nv_note_time_idx ON app.note_versions (note_id, edited_at DESC);

-- TAGS
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

-- NOTE ENRICHMENTS (unchanged)
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

-- NOTE EMBEDDINGS (v2 — pgvector 0.8.2, HNSW)
CREATE TABLE app.note_embeddings (
  note_id         UUID PRIMARY KEY REFERENCES app.notes(id) ON DELETE CASCADE,
  model           TEXT NOT NULL DEFAULT 'voyage-3.5',
  dim             INT  NOT NULL DEFAULT 1024,
  embedding       vector(1024) NOT NULL,
  generated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX note_emb_hnsw_idx ON app.note_embeddings
  USING hnsw (embedding vector_cosine_ops) WITH (m=16, ef_construction=64);

-- JOBS
CREATE TYPE app.job_status AS ENUM ('queued','running','done','failed','cancelled');

CREATE TABLE app.enrichment_jobs (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  note_id         UUID NOT NULL REFERENCES app.notes(id) ON DELETE CASCADE,
  organization_id UUID REFERENCES app.organizations(id) ON DELETE SET NULL,
  inngest_run_id  TEXT,
  status          app.job_status NOT NULL DEFAULT 'queued',
  attempts        INT NOT NULL DEFAULT 0,
  error           TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  finished_at     TIMESTAMPTZ
);
CREATE INDEX ej_note_idx ON app.enrichment_jobs (note_id, created_at DESC);
CREATE INDEX ej_status_idx ON app.enrichment_jobs (status) WHERE status IN ('queued','running');
CREATE INDEX ej_org_idx ON app.enrichment_jobs (organization_id) WHERE status IN ('queued','running');

-- AUDIT (updated v2) — per organisation
CREATE TABLE audit.events (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  organization_id UUID,
  actor_user_id   UUID,
  action          TEXT NOT NULL,
  resource_type   TEXT NOT NULL,
  resource_id     UUID NOT NULL,
  payload         JSONB,
  trace_id        TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX audit_org_idx ON audit.events (organization_id, created_at DESC);
CREATE INDEX audit_resource_idx ON audit.events (resource_type, resource_id, created_at DESC);

-- DATA PROCESSING AGREEMENTS (ny i v2)
CREATE TABLE app.dataprocessing_agreements (
  id              UUID PRIMARY KEY DEFAULT uuidv7(),
  vendor_name     TEXT NOT NULL,
  version         TEXT NOT NULL DEFAULT '1.0',
  signed_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at      TIMESTAMPTZ,
  content         TEXT NOT NULL,
  signed_by       UUID NOT NULL REFERENCES app.users(id)
);

-- RLS POLICIES (v2)
ALTER TABLE app.notes ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.notebooks ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.notebook_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.note_enrichments ENABLE ROW LEVEL SECURITY;

-- Notes: visible if user is notebook member OR org member
CREATE POLICY notes_select_v2 ON app.notes FOR SELECT
  USING (
    notebook_id IN (
      SELECT nm.notebook_id FROM app.notebook_members nm
      WHERE nm.user_id = (SELECT id FROM app.users WHERE clerk_user_id = auth.user_id())
    )
    OR
    notebook_id IN (
      SELECT n.notebook_id FROM app.notes n
      WHERE n.organization_id IN (
        SELECT om.organization_id FROM app.organization_members om
        WHERE om.user_id = (SELECT id FROM app.users WHERE clerk_user_id = auth.user_id())
      )
    )
  );

-- Notebooks: visible if owner or notebook member or org member
CREATE POLICY notebooks_select_v2 ON app.notebooks FOR SELECT
  USING (
    owner_id = (SELECT id FROM app.users WHERE clerk_user_id = auth.user_id())
    OR
    id IN (
      SELECT nm.notebook_id FROM app.notebook_members nm
      WHERE nm.user_id = (SELECT id FROM app.users WHERE clerk_user_id = auth.user_id())
    )
    OR
    organization_id IN (
      SELECT om.organization_id FROM app.organization_members om
      WHERE om.user_id = (SELECT id FROM app.users WHERE clerk_user_id = auth.user_id())
    )
  );
```

---

## 4. Sikkerhedsmodel (v2 — udvidet)

### Auth-arkitektur: Clerk 7.2 + Neon RLS Authorize

- Clerk JWT Template "neon": `alg=RS256`, `iss`, `aud`, `exp`, `nbf` valideres af `pg_session_jwt`.
- Kort-levede access tokens (~60 sek).
- Refresh-tokens roteret.

### Authorization: Defense-in-depth

1. **API-lag** (Next.js Server Actions / Route Handlers): Medlemskab tjekkes eksplicit.
2. **RLS** (Postgres): Hvis lag 1 fejler, RLS blokerer (jf. politikkerne ovenfor).
3. **Property-level** (BOPLA): Zod `.strict()` afviser ukendte felter.

### OWASP API Top 10 (v2 status)

| Risiko | Mitigation v2 |
|---|---|
| API1 BOLA | RLS + per-resource API-check; org-isolation |
| API2 Broken Auth | Clerk 7.2 + JWT RS256 pinning |
| API3 BOPLA | Zod strict validering |
| API4 Unrestricted Resource | Vercel WAF + Inngest concurrency 3/org + note-quota |
| API5 Broken Function-Level Auth | Rolle-rank check på alle mutating endpoints |
| API6 Unrestricted Sensitive Business Flow | Rate-limit /api/import + org-level budgets |
| API7 SSRF | Aldrig fetch URL fra note-body server-side |
| API8 Misconfig | Prod ≠ test ≠ dev; secrets via Vercel env/Neon secret |
| API9 Improper Inventory | OpenAPI 3.1 contract i repo; CI lint (Spectral) |
| API10 Unsafe Consumption | Anthropic-respons valideres mod Zod før DB-write |

### Data-behandling & compliance

- **Databehandleraftaler**: tabel `dataprocessing_agreements` — Neon, Vercel, Clerk, Anthropic, Inngest, Voyage.
- **GDPR**: opt-out AI per notebook; data-export pr. organisation; slet-konto flow.
- **Antropic Zero Data Retention**: aktiveret på prod-workspace.
- **Audit**: alle ændringer logges med `organization_id` + `actor_user_id`.

---

## 5. AI-berignings-pipeline (v2)

### Berigelser

| Berigelse | Model | Kostgrad | Værdi | Opt-out? |
|---|---|---|---|---|
| Titel-forslag | Haiku 4.5 | Lav | Høj | Per notebook |
| Resumé (2-3 sætninger) | Haiku 4.5 | Lav | Høj | Per notebook |
| Tags (3-7 stk) | Haiku 4.5 | Lav | Høj | Per notebook |
| Foreslåede links | Haiku 4.5 + similarity search | Mellem | Mellem | Per notebook |
| Embedding | voyage-3.5 | Lav | Lav v1, høj senere | Per notebook |
| Komplekse opgaver (rewrite, struktur) | Sonnet 4.6 | Høj | Lav v1 (manuel) | Altid manuel |

### Inngest v4 orchestration

- Concurrency per organisation: max 3 parallelle jobs.
- Dødt-bogstavs-kø efter 3 fejlforsøg.
- Dashboard-visibilitet.

### Budget-kalibrering

- **Haiku 4.5**: ~0,000026 USD/note (titel+resumé+tags).
- **23.000+ noter/md** på 70 USD budget.
- **Voyage-3.5 embeddings**: ~0,002 USD/1M tokens.
- **Sonnet 4.6** (manuel): ~10x kostgrad, sparsom brug.

**Per-workspace budgets (spend caps)**:
- Prod: $50/md
- Test: $15/md
- Dev: $5/md

---

## 6. Søgning (v2)

### MVP: Postgres FTS (tsvector + GIN)

Under 100k noter: `tsvector` + `ts_rank_cd()` er tilstrækkelig.

Når korpus vokser: tilføj pgvector + HNSW og flette via RRF (Reciprocal Rank Fusion).

### Hybrid-query-skitse (fase 2)

```sql
WITH fts AS (
  SELECT id, ts_rank_cd(body_search, plainto_tsquery('simple', $1)) AS s,
         row_number() OVER (ORDER BY ts_rank_cd(...) DESC) AS r
  FROM app.notes
  WHERE body_search @@ plainto_tsquery('simple', $1)
    AND notebook_id IN (SELECT ... )  -- RLS check
  LIMIT 50
),
vec AS (
  SELECT n.id, 1 - (e.embedding <=> $2::vector) AS s,
         row_number() OVER (ORDER BY e.embedding <=> $2::vector) AS r
  FROM app.notes n JOIN app.note_embeddings e ON n.id = e.note_id
  WHERE n.notebook_id IN (SELECT ... )  -- RLS check
  LIMIT 50
)
SELECT id, COALESCE(1.0/(60+f.r),0) + COALESCE(1.0/(60+v.r),0) AS rrf
FROM fts f FULL OUTER JOIN vec v USING (id)
ORDER BY rrf DESC LIMIT 20;
```

---

## 7. Import-pipeline (v2)

### Flow: drop-zone UI

1. FGD åbner `/import`-side.
2. Drag-and-drop mappe (HTML5 `webkitdirectory`).
3. Klient batcher i 20-fil-chunks, POST'er multipart til `/api/import` med `Idempotency-Key`.
4. Server opretter `notes` i "Inbox"-notebook + `enrichment_jobs` i én transaktion.
5. AI foreslår notebook-tildeling baseret på titel + resumé.
6. UI viser fremdrift via SSE `/api/jobs/stream`.

### Post-import organisation

Når berigelse er færdig: UI viser "Foreslåede notebooks" per note, FGD accepterer eller ignorerer forslag.

---

## 8. Miljø-strategi: dev → test → prod

**Neon-projekter** (ikke branches):
- **Prod**: Scale-to-zero OFF, read-only replicas enabled, daily backups, custom domain support.
- **Test**: Scale-to-zero ON, manual testing branch, weekly backups.
- **Dev**: Scale-to-zero ON, developer branches, på-demand.

**Vercel**:
- **Prod**: Pro plan, custom domain `docs.fgdconsulting.se`, auto-deploy på main-push.
- **Test**: Custom environment (Pro), auto-deploy på test-branch.
- **Dev**: Hobby plan, pull-request previews.

**Clerk**:
- **Prod instance** + **Dev instance** (test deler dev).

**Anthropic**:
- **Prod workspace**: Zero Data Retention enabled, $50/md spend cap.
- **Test workspace**: samme credentials som prod (begrænset spend cap $15/md).
- **Dev workspace**: samme credentials (spend cap $5/md).

**Inngest**:
- **Prod environment**, **Test environment**, **Dev environment**.

**Secrets-mapping**:

| Secret | Prod | Test | Dev |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | Prod workspace | Test workspace | Dev workspace |
| `CLERK_SECRET_KEY` | Prod instance | Prod instance (test=dev) | Dev instance |
| `DATABASE_URL` | Prod Neon project | Test Neon project | Dev Neon project |
| `INNGEST_EVENT_KEY` | Prod env | Test env | Dev env |
| `VOYAGE_API_KEY` | Prod account | Prod account | Prod account |

**Promotion-pipeline**:
1. Dev branch → review/test.
2. Kald Camilla: "promoter migrations til test".
3. Manual approval fra FGD: "godkendt, gå til prod".
4. Camilla runner migrations på prod, deploys på main.

---

## 9. Implementeringsplan (5–7 uger)

### Fase 0 — Forberedelse (1 uge)

- Lailas rolle-design + ansættelse (Frontend Dev, Backend/Neon, AI Pipeline).
- Camilla: 3 Neon-projekter + Vercel Pro + Clerk + Anthropic workspaces.
- Backend-rolle: repo-skelet, Drizzle migrations, OpenAPI stub.
- Inngest cloud-setup.

### Fase 1 — MVP (2–3 uger): "FGD bruger alene"

- Auth (Clerk login/signup).
- Notebooks: CRUD.
- Notes: CRUD, markdown-editor.
- Import: drop-zone + CLI (landet i "Inbox").
- AI auto-berigelse: Haiku 4.5 batch (titel, resumé, tags) via Inngest v4.
- Manuel raffinering: acceptér/rediger/forkast pr. felt.
- Søgning: FTS via tsvector.
- Mobil-responsivt (Tailwind v4 + shadcn).
- OpenAPI 3.1 contract.
- RFC 9457 fejl + idempotency.
- Observability: Vercel OTel + Sentry.

### Fase 2 — v1 delt (2–3 uger): "Med kunder"

- Organisation-tabel aktiveret.
- RLS Authorize aktiveret.
- Notebook sharing + magic-link invite.
- Audit-log.
- Rate-limiting.
- PWA via Serwist.
- Version-historik + LZ4 komprimering.
- Pgvector + HNSW aktiveret.
- Hybrid-søgning (FTS + vector + RRF).
- GDPR: export + slet-konto.

### Fase 3+ — Nice-to-have

- Slash-commands i editor.
- Co-editing (Y.js).
- Evernote/Notion import.
- Public read-only links.
- Sonnet 4.6 premium-features.

---

## 10. Stack-recap (v2)

| Lag | Valg | Version |
|---|---|---|
| Frontend | Next.js + App Router + React Server Components | 16.2 |
| Framework | React | 19.2 |
| Styling | Tailwind CSS | v4 |
| UI Components | shadcn/ui | latest |
| Editor | Markdown (valgfrit rich-text i fase 2) | tbd |
| Auth | Clerk | 7.2 |
| Hosting | Vercel | Pro |
| Database | Neon Postgres | frankfurt, 3 projects |
| ORM | Drizzle | 0.45.2 |
| Vectors | pgvector | 0.8.2 |
| Background jobs | Inngest Cloud | v4 |
| AI (batch) | Anthropic Haiku | 4.5 |
| AI (complex) | Anthropic Sonnet | 4.6 |
| Embeddings | Voyage AI | 3.5 |
| PWA | Serwist | latest |
| Observability | Vercel OTel + Sentry | builtin |
| Testing | Vitest + React Testing Library | latest |
| Linting | ESLint | 9+ |
| Type-checking | TypeScript | 5.6+ |

---

## 11. Åbne spørgsmål til FGD (fra v1, til bekræftelse)

1. **Design-stil** for UI — følger du Linear, Vercel, Stripe, Notion, eller custom? Påvirker shadcn-tema valg + figma-setup.
2. **Vercel Pro plan** — OK til 50 USD/md budget?
3. **Subteam-timing** — feature i v2, eller senere?
4. **Customer-volumen v1** — forventes hvor mange kunder første 6 md?
5. **Tekstmappe-struktur** ved import — skal hierarki mappes til notebooks, eller allt i Inbox + AI-organisering?

---

## Sources & references

- Neon RLS Authorize: https://neon.tech/docs/guides/neon-rls-authorize-clerk
- pgvector 0.8.2: https://neon.com/docs/extensions/pgvector
- Inngest v4: https://www.inngest.com/docs
- Next.js 16: https://nextjs.org/docs
- Clerk 7.2: https://clerk.com/docs
- Voyage AI: https://docs.voyageai.com/
- Anthropic API: https://docs.anthropic.com/
- Vercel Pro: https://vercel.com/pricing
