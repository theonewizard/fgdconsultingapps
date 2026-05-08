---
dato: 2026-05-08
forfatter: Poul
emne: Knowlagecentral - krav-specifikation
status: vedtaget
dokumenttype: levende-dokument
relateret_arkitektur: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
relateret_brief: Team Inbox/projects/knowlagecentral/2026-05-05-knowlagecentral-evernote-style.md
versions-verificeret: 2026-05-08 (Expo SDK 55, Capacitor 8.3.1, Next.js 16.2, React 19.2)
---

# Knowlagecentral — Krav-specifikation

> Dette er et levende dokument. Krav tilføjes/justeres efterhånden som FGD og teamet afklarer scope. Ændringer logges i §10 (revisions-log).
>
> **Status 2026-05-08**: Alle 7 åbne spørgsmål afklaret af FGD. Dokumentet er nu **vedtaget** og udgør grundlag for v4-arkitektur og rolle-design.

---

## 1. Mission Statement

> Knowlagecentral giver konsulenter og deres kunder et sikkert, AI-beriget noteværktøj — fra råt tekstindhold til struktureret viden, bygget til de brancher der ikke kan gå på kompromis med sikkerhed.

---

## 2. Produkt-overblik

Knowlagecentral er en multi-tenant videns-platform i Evernote-stil, hvor en konsulent (FGD og senere andre konsulenter) opbygger sin egen videns-base af noter organiseret i notebooks, og kontrolleret kan dele udvalgte notebooks med kunder via magic-link-invites med rolle-baseret adgang.

Råt tekstindhold (txt-batches, manuelle noter, importer fra Evernote) løftes automatisk til struktureret viden gennem AI-berigelse: titel-forslag, resumé, tags, foreslåede notebook-tilknytninger og embeddings til semantisk søgning. Brugeren accepter, redigerer eller afviser hvert AI-forslag pr. felt — AI'en er en assistent, ikke en autoritet.

Platformen er bygget til regulerede vertikaler (finans, sundhed, gov-rådgivning) hvor compliance ikke er et add-on: data-residency i EU (Frankfurt), defense-in-depth-sikkerhed (Clerk JWT + Neon RLS + API-lag-checks + Zod strict), opt-out af AI pr. notebook (GDPR), Anthropic Zero Data Retention, FIPS 140-3 HSM som produktionsmål, og audit-log på alle skrivninger.

Hele teknik-arkitekturen er detaljeret i `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md`. Dette dokument fokuserer på *hvad* produktet skal kunne — ikke *hvordan* det er bygget.

---

## 3. Platform-scope

Knowlagecentral skal være tilgængelig på følgende platforme i v1:

### 3.1 Web (primær)

- **Responsiv web-app**: fungerer på desktop-browsere (primær redigerings-flade) og mobil-browsere (læs + quick-note).
- **PWA-installerbar**: brugeren kan tilføje appen til home screen på iOS/Android/desktop og få en native-lignende oplevelse uden app-store. Offline-cache for noter besøgt seneste 7 dage (fase 2 via Serwist).
- **Domæne**: `docs.fgdconsulting.se` (Cloudflare CNAME, "DNS only", Vercel Pro hosting).

### 3.2 Native iOS (iPad + iPhone)

- Selvstændig app i App Store, optimeret til touch-interaktion.
- Krav: native scroll-performance, system-tastatur-integration, share-extension (modtag tekst fra andre apps).
- Push-notifikationer fravalgt i v1 (se §9.5).

### 3.3 Native Android

- Selvstændig app i Google Play Store, optimeret til touch.
- Samme krav som iOS: native scroll, system-tastatur, share-intent.
- Push-notifikationer fravalgt i v1 (se §9.5).

### 3.4 Hvad "native" betyder i denne kontekst — VALGT STRATEGI

**FGD-beslutning 2026-05-08: Strategi A — Expo / React Native (ægte native).**

| Strategi | Beskrivelse | Status |
|---|---|---|
| **A: Cross-platform via Expo / React Native** ✅ VALGT | Én React Native-kodebase deler logik og UI-primitives med web-koden hvor muligt; kompilerer til ægte native iOS + Android binaries. **Verificeret 2026-05-08: Expo SDK 55** (React Native 0.83, New Architecture only). | Reel native performance; én skill-set (React) for hele teamet; deler `packages/zod-contracts` med web. ~3-6 ugers ekstra dev-tid for v1. Kræver mobil-specialist (ny rolle via Laila). |
| B: Hybrid via Capacitor | Web-koden pakkes ind i en native shell. | **Fravalgt**: kompromis på scroll-/keyboard-feel og App Store-review-risiko vurderedes uacceptabel for de regulerede vertikaler hvor mobil-feel signalerer professionalisme. |

**Konsekvens af Strategi A**:
- Turborepo udvides med `apps/mobile/` (Expo).
- Kodedeling med web via `packages/zod-contracts` og `packages/ai`.
- Laila skal designe en mobil-specialist-rolle (eller udvide Sofie's mandat).
- Henrik's API-contract skal verificeres som platform-agnostisk (allerede tilfældet via OpenAPI).

---

## 4. Funktionelle krav

### 4.1 Note-håndtering

| ID | Krav | Prioritet |
|---|---|---|
| FN-01 | Bruger kan oprette ny note manuelt i et valgt notebook | MVP |
| FN-02 | Bruger kan redigere note (TipTap rich-text-editor, debounced auto-save) | MVP |
| FN-03 | Bruger kan slette note (soft-delete med 30-dages restore-vindue) | MVP |
| FN-04 | System gemmer version-snapshot ved 30s inaktivitet ELLER 200 char akkumuleret diff | MVP |
| FN-05 | Bruger kan se versions-historik, diffe to versioner og rulle tilbage | Fase 2 |
| FN-06 | Bruger kan tilføje/fjerne tags pr. note (manuelle + AI-foreslåede markeret separat) | MVP |
| FN-07 | Bruger kan flytte note mellem notebooks | MVP |
| FN-08 | **TipTap-baseret rich-text editor med markdown-input-support** (overskrifter, lister, links, kode-blokke, tabeller; markdown-shortcuts og paste-konvertering understøttes) | MVP |
| FN-09 | Note kan have vedhæftninger (billeder, PDF) | Fase 3 |

> **Note**: FN-10 (separat åbent rich-text-spørgsmål) er fjernet — fusioneret ind i FN-08 efter FGD-beslutning 2026-05-08. TipTap er den valgte editor; markdown er ikke længere en selvstændig editor-mode men input-support i TipTap.

### 4.2 Notebook-organisering

| ID | Krav | Prioritet |
|---|---|---|
| FN-11 | Bruger kan oprette/omdøbe/arkivere notebook | MVP |
| FN-12 | Hver bruger har én default "Inbox"-notebook (kan ikke slettes; alle imports lander her) | MVP |
| FN-13 | Notebook har navn, beskrivelse, opt-in/opt-out af AI-berigelse | MVP |
| FN-14 | Notebook kan være privat (kun owner) eller delt (medlemmer + roller) | MVP |
| FN-15 | Notebook kan tilhøre en organisation (multi-tenant subteam-feature) | Fase 2 |
| FN-16 | Bruger kan se liste over alle notebooks de har adgang til | MVP |

### 4.3 Import

| ID | Krav | Prioritet |
|---|---|---|
| FN-17 | Bruger kan drag-and-drop en mappe med txt-filer (HTML5 `webkitdirectory`) | MVP |
| FN-18 | Alle importerede noter lander i Inbox-notebook med `status='imported'` | MVP |
| FN-19 | Import udløser auto-berigelse på hver note (medmindre opt-out) | MVP |
| FN-20 | CLI-fallback: `pnpm dlx fgd-import <dir>` til store batches | MVP |
| FN-21 | Evernote `.enex`-import (parser konverterer til TipTap-JSON + tags) | Fase 2 |
| FN-22 | UI viser fremdrift pr. fil under import (queued / processing / done / failed) | MVP |
| FN-23 | Import er idempotent (`Idempotency-Key` header forhindrer duplikater ved retry) | MVP |

### 4.4 AI-berigelse

| ID | Krav | Prioritet |
|---|---|---|
| FN-24 | AI foreslår titel for hver ny/ændret note (Haiku 4.5) | MVP |
| FN-25 | AI foreslår resumé (2-3 sætninger) | MVP |
| FN-26 | AI foreslår 3-7 tags | MVP |
| FN-27 | AI foreslår notebook-tilknytning (fra brugerens eksisterende notebooks) | MVP |
| FN-28 | AI foreslår relaterede noter (similarity via embeddings) | Fase 2 |
| FN-29 | Bruger kan accept / edit / reject hvert AI-forslag pr. felt | MVP |
| FN-30 | Bruger kan trigge "kør berigelse igen" manuelt | MVP |
| FN-31 | Bruger kan vælge Sonnet 4.6 til "premium-raffinering" (struktur-rewrite) | Fase 3 |
| FN-32 | Opt-out af AI-berigelse pr. notebook OG pr. note | MVP |
| FN-33 | AI-forslag streames token-for-token til UI (Streamdown + ai-elements) | MVP |

### 4.5 Søgning

#### 4.5.1 Standard søgning (eksisterende)

| ID | Krav | Prioritet |
|---|---|---|
| FN-34 | Global søgefelt søger på tværs af alle notebooks brugeren har adgang til | MVP |
| FN-35 | Søgning bruger Postgres FTS (`tsvector` + `ts_rank_cd`) i MVP | MVP |
| FN-36 | Hybrid søgning (FTS + pgvector + RRF) i fase 2 | Fase 2 |
| FN-37 | Filtre: notebook, tag, dato-interval, kun mine egne / kun delte | MVP |
| FN-38 | Resultat viser snippet med highlight + notebook-tilknytning | MVP |
| FN-39 | RLS sikrer at brugeren kun får hits fra notebooks de har adgang til | MVP |

#### 4.5.2 Hurtig-søgnings-prompt (NY feature)

**Definition**: En altid-tilgængelig command-bar (ligesom Cmd-K / Ctrl-K i moderne web-apps) hvor brugeren skriver et naturligt-sprog-spørgsmål eller en kort søgning, og systemet svarer enten med:

- **Direkte søgeresultater** (hvis input ligner en søgning: enkeltord, query-streng), eller
- **AI-genereret svar** (hvis input ligner et spørgsmål: "hvad sagde kunde X om budget i marts?" → AI læser top-N relevante noter og syntetiserer svar med kilde-referencer).

**Adskillelse fra standard søgning** (FN-34 ff.):

| | Standard søgning | Hurtig-søgnings-prompt |
|---|---|---|
| Aktivering | Dedikeret søge-side (`/search`) | Globalt keyboard shortcut (Cmd/Ctrl-K) overalt i appen |
| Input | Query-string + filtre | Naturligt sprog ELLER query |
| Output | Liste af notes med snippets | Enten note-liste ELLER AI-svar med kilde-citater |
| Latency-krav | < 300ms P95 | < 800ms til første token (streaming) |
| Backend | Postgres FTS (MVP) / hybrid (fase 2) | RAG: hybrid-søg → top-K noter → Haiku 4.5 syntese |

| ID | Krav | Prioritet |
|---|---|---|
| FN-40 | Globalt keyboard shortcut (Cmd/Ctrl-K) åbner hurtig-søgnings-prompt overalt | MVP |
| FN-41 | Prompt kan bruges som ren søgeboks (returnerer note-liste) | MVP |
| FN-42 | Prompt kan bruges som AI-spørgsmål (returnerer AI-svar med kilde-citater) | Fase 2 (kræver hybrid søgning) |
| FN-43 | AI-svar viser hvilke noter blev brugt som kontekst (klikbare links) | Fase 2 |
| FN-44 | Recent searches gemmes pr. bruger (lokalt + server-side) | Fase 2 |
| FN-45 | Prompt understøtter slash-commands (fx `/notebook:X søgeord` for at filtrere) | Fase 3 |

> **MVP-scope (FGD-beslutning 2026-05-08)**: MVP leverer **kun FN-40 + FN-41** (søgeboks-mode, samme backend som §4.5.1). FN-42 + FN-43 (AI-svar med RAG) leveres i **fase 2** sammen med hybrid søgning, fordi RAG-kvalitet er stærkt afhængig af pgvector-similarity.

### 4.6 Deling med kunder (magic-link, RBAC)

| ID | Krav | Prioritet |
|---|---|---|
| FN-46 | Owner kan invitere ekstern bruger til notebook via email + rolle | MVP |
| FN-47 | Roller: `viewer` (read), `editor` (read+write), `admin` (read+write+invite) | MVP |
| FN-48 | Invite genererer magic-link (`token = randomBytes(32).hex`, hash gemt) | MVP |
| FN-49 | Invite udløber efter 7 dage (single-use, `consumed_at` markeres) | MVP |
| FN-50 | Modtager logger ind via Clerk; system tilføjer dem til `notebook_members` | MVP |
| FN-51 | Owner kan se alle medlemmer + ændre rolle + fjerne medlem | MVP |
| FN-52 | Audit-log entry for hver invite-genereret, accepteret, fjernet | MVP |

### 4.7 Subteam / organisation (fase 2)

| ID | Krav | Prioritet |
|---|---|---|
| FN-53 | Owner kan oprette en organisation med navn + beskrivelse | Fase 2 |
| FN-54 | Owner kan invitere medlemmer til organisation (ikke per-notebook) | Fase 2 |
| FN-55 | Notebook kan tilknyttes organisation; alle medlemmer får adgang via `organization_members` | Fase 2 |
| FN-56 | Organisation-admin kan administrere medlemmer + notebooks | Fase 2 |
| FN-57 | Audit-log pr. organisation (separate views) | Fase 2 |

### 4.8 Eksport

| ID | Krav | Prioritet |
|---|---|---|
| FN-58 | Bruger kan eksportere notebook som zip med markdown-filer + metadata | MVP |
| FN-59 | Eksport indeholder originalt indhold + AI-berigelser (titel, resumé, tags) | MVP |
| FN-60 | Eksport pr. organisation (alle deres notebooks) — GDPR data-export | Fase 2 |
| FN-61 | Slet-konto flow: bruger kan exportere alt + slette permanent (cascade) | Fase 2 |

---

## 5. Ikke-funktionelle krav

### 5.1 Sikkerhed

| ID | Krav |
|---|---|
| NFR-01 | OWASP API Security Top 10 (2023) compliance — checklist på hver release |
| NFR-02 | TLS 1.3 in-transit; AES-256 at-rest (Neon default) |
| NFR-03 | Clerk JWT med `alg=RS256` (alg-pinned, never `none`); claims `iss/aud/exp/nbf/sub` valideret |
| NFR-04 | Defense-in-depth: API-lag-check + Neon RLS + Zod `.strict()` (tre-lags BOLA/BOPLA-mitigation) |
| NFR-05 | Idempotency-Key obligatorisk på alle non-GET writes |
| NFR-06 | Rate-limiting på alle public endpoints; `RateLimit-*` headers |
| NFR-07 | RFC 9457 Problem Details for HTTP errors |
| NFR-08 | Magic-link-invites: token kun gemt som SHA-256 hash; expiry + single-use |
| NFR-09 | Audit-log på alle skrivninger (`actor_user_id`, `organization_id`, `trace_id`) |
| NFR-10 | Secrets aldrig i kode/logs/error-responses; via Vercel env (auto-injection fra Native Integration) |
| NFR-11 | CORS: allow-list pr. origin (ingen `*` med credentials) |

### 5.2 Compliance

| ID | Krav |
|---|---|
| NFR-12 | Data-residency EU (Frankfurt eu-central-1) — alle Neon-branches arver region |
| NFR-13 | GDPR opt-out af AI-berigelse pr. notebook OG pr. note |
| NFR-14 | GDPR data-export pr. organisation + slet-konto flow |
| NFR-15 | Anthropic Zero Data Retention (ZDR) aktiveret på prod-workspace før første kunde-data |
| NFR-16 | DPA'er signeret + tabel-tracket pr. leverandør (Neon, Vercel, Clerk, Anthropic, Inngest, Voyage, Yubico) |
| NFR-17 | **FIPS 140-3 HSM er produktionsmål for ALLE kunder** (ikke kun dem der betaler). YubiHSM 2 FIPS, fase 1.5+; non-FIPS i MVP. **Ingen FIPS-tier-differentiering** — FIPS er native i produktet (FGD-beslutning 2026-05-08). |
| NFR-18 | SOC 2 Type II — design-niveau fra dag 1; certificering fase 3+ (ejes af Mads + Trine) |
| NFR-19 | ISO 27001:2022 — design-niveau fra dag 1; certificering fase 3+ (ejes af Mads + Trine) |
| NFR-20 | DPO-funktion: FGD juridisk DPO; Dorthe operationel support (ROPA, DPIA, breach-respons) |

### 5.3 Performance

| ID | Krav |
|---|---|
| NFR-21 | P95 < 300 ms for liste/søg op til 50.000 noter (web + native) |
| NFR-22 | P95 < 800 ms til første token i hurtig-søgnings-prompt AI-mode |
| NFR-23 | Cold-start tolerable (<1s) på Vercel Fluid Compute (Node.js runtime) |
| NFR-24 | Native app cold-start < 2s |
| NFR-25 | Web Vitals: LCP < 2.5s, INP < 200ms, CLS < 0.1 |

### 5.4 Tilgængelighed

| ID | Krav |
|---|---|
| NFR-26 | 99.5% oppetid (Vercel + Neon SLA) |
| NFR-27 | WCAG 2.2 AA på alle UI-komponenter (web + native hvor muligt) |
| NFR-28 | Responsivt web-UI fra 320px (iPhone SE) til 4K |

### 5.5 Offline-support og push

| ID | Krav |
|---|---|
| NFR-29 | **Push-notifikationer**: fravalgt i v1 (FGD-beslutning 2026-05-08). Ingen APNs/FCM-integration i MVP eller fase 2. Revurderes fase 3+ hvis kunde-feedback tilsiger det. |
| NFR-30 | **Native iOS/Android offline-cache**: kun noter brugeren faktisk har besøgt; cache slettes fra device efter **1 dag**. Data on-device krypteres (AES-256, nøgle i iOS Keychain / Android Keystore). Data in-flight krypteres (TLS 1.3). |
| NFR-30b | **On-device kryptering er obligatorisk**: app afviser at cache data hvis enhedens krypterede storage (Keychain / Keystore) ikke er tilgængelig — ingen plaintext-fallback under nogen omstændigheder. App viser tydelig fejlmeddelelse og fortsætter i online-only-mode. |
| NFR-31 | **Web/PWA**: Serwist offline-cache for noter besøgt seneste 7 dage (fase 2). Web/PWA bruger browserens egen storage-kryptering; on-device-kryptering på samme niveau som native gælder ikke for web. |
| NFR-32 | Offline-redigering med last-write-wins conflict resolution; CRDT (Y.js) overvejes fase 3 |
| NFR-33 | UI viser tydeligt offline-status + ulæste ændringer |

### 5.6 Observability

| ID | Krav |
|---|---|
| NFR-34 | OpenTelemetry traces propageres via W3C Trace Context (`traceparent` header) |
| NFR-35 | Sentry til errors (web + native) |
| NFR-36 | Vercel Speed Insights til Web Vitals (web) |
| NFR-37 | Strukturerede logs (JSON); ingen PII i logs |
| NFR-38 | Health-endpoints `/livez` + `/readyz` (unauthenticated, minimal info) |

### 5.7 Kost-loft og distribution

| ID | Krav |
|---|---|
| NFR-39 | AI-budget < 500 DKK/md (~$70/md), fordelt på prod ($50) / test ($15) / dev ($5) workspace |
| NFR-40 | **App Store-strategi: single-brand distribution under "Knowlagecentral" / FGD Consulting** (FGD-beslutning 2026-05-08). Én listing pr. platform (Apple App Store + Google Play). Ingen white-label / kunde-rebranding i v1. Apple Developer Program $99/år + Google Play $25 engangs. |
| NFR-41 | Infrastructure < $50/md (Vercel Pro $20 + Speed Insights $10 + Neon free + Clerk free + Inngest free) |

---

## 6. Out of scope (v1)

Følgende er **eksplicit ikke** med i v1 (MVP + fase 2). Kan evalueres til fase 3+:

- Real-time co-editing (Y.js / collaborative cursors)
- Public read-only links (ikke-autentificerede shares)
- Vedhæftninger ud over markdown-tekst (billeder, PDF, lyd, video)
- Slash-commands i editor (`/today`, `/template:X`)
- Notion-import (kun Evernote-import er planlagt fase 2)
- AI-genererede dashboards / analytics på tværs af noter
- Knowlagecentral som MCP-server (separat behovs-analyse — Poul leverer senere)
- Multi-language UI (kun dansk + engelsk i MVP)
- Custom kunde-branding pr. organisation (logo, farver) — single-brand i v1 (NFR-40)
- White-label App Store-distribution (revurderes fase 4+)
- Push-notifikationer (NFR-29) — revurderes fase 3+
- E2E-encryption (zero-knowledge) — vurderes mod Mads' threat-model fase 3+
- Apple Watch / Wear OS apps
- Desktop-native apps (macOS / Windows / Linux) — PWA dækker desktop-mobil-installation
- Browser-extension (Chrome/Firefox clip-to-Knowlagecentral)
- Webhook-API til kunder (kunde får notifikationer ved opdateringer)

---

## 7. Mapping mellem v3-arkitektur og krav-spec

For at sikre sporbarhed mellem teknisk arkitektur (`Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md`) og dette krav-dokument:

| Krav-blok | v3-sektion | Note |
|---|---|---|
| §3 Platform-scope | v3 §3.1 (komponent-oversigt) | Native iOS/Android (Strategi A, Expo) er **nyt** scope; v3 dækker kun web/PWA |
| §4.1-4.2 Note + notebook | v3 §4 (datamodel: `app.notes`, `app.notebooks`) | TipTap rich-text er nyt valg; data-format opdateres til TipTap-JSON |
| §4.3 Import | v3 §8 (import-pipeline) | 1:1 dækket; `.enex`-parser skal output TipTap-JSON |
| §4.4 AI-berigelse | v3 §6 (AI-berignings-pipeline) | 1:1 dækket |
| §4.5.1 Standard søgning | v3 §7 (søgning) | 1:1 dækket |
| §4.5.2 Hurtig-søgnings-prompt | **ikke i v3** | **Ny feature** — kræver design-tilføjelse til v3 (kun FN-40+FN-41 i MVP) |
| §4.6 Deling | v3 §5.3 (magic-link-flow) | 1:1 dækket |
| §4.7 Organisation | v3 §4 (`app.organizations`) + §10 (fase 2) | 1:1 dækket |
| §4.8 Eksport | v3 §10 fase 2 | Krav-niveau ikke detaljeret i v3 — udvides ved fase 2 design |
| §5.1-5.2 Sikkerhed/compliance | v3 §5 + Mads' governance-dokumenter | NFR-17 ændret: FIPS native for alle kunder |
| §5.5 Offline native | **ikke i v3** | **Ny feature** — kræver design-tilføjelse til v3 (NFR-30/30b: kun besøgte, 1-dags TTL, AES-256 + Keychain/Keystore) |

**Konsekvens**: Når dette krav-dokument er godkendt af FGD (status nu **vedtaget**), skal v3-analysen udvides til v4 med:
1. Native platform-arkitektur (Expo / React Native, Strategi A)
2. TipTap rich-text editor + datamodel (TipTap-JSON i stedet for plain markdown)
3. Hurtig-søgnings-prompt UI (FN-40 + FN-41 i MVP; FN-42-FN-45 fase 2+)
4. Native offline-cache med 1-dags TTL + AES-256 + Keychain/Keystore (NFR-30/30b)
5. FIPS native HSM-arkitektur for alle kunder (NFR-17) — Mads opdaterer HSM-arkitektur

---

## 8. Versions-verifikation (per 2026-05-08)

| Komponent | Anbefalet version | Verificeret via | Note |
|---|---|---|---|
| Next.js | 16.2 | v3-analyse + Context7 (2026-05-05) | Cache Components stable |
| React | 19.2 | v3-analyse + Context7 | Server Actions GA |
| Clerk | 7.2 | v3-analyse | JWT-template "neon" |
| Neon | n/a (managed) | v3-analyse | Frankfurt + pg_session_jwt |
| Vercel AI SDK | v6 | v3-analyse | streamText + generateObject |
| Anthropic Haiku | 4.5 (`claude-haiku-4-5-20251001`) | v3-analyse | Batch-berigelse |
| Anthropic Sonnet | 4.6 | v3-analyse | Manuel raffinering |
| Voyage embeddings | voyage-3.5 (1024d) | v3-analyse | |
| Inngest | v4 | v3-analyse | Step-functions |
| Drizzle ORM | 0.45.2 | v3-analyse | Schema + migrations |
| **Expo SDK** (Strategi A — VALGT) | **55** (RN 0.83) | WebSearch 2026-05-08 | New Architecture only; SDK 55 udgivet feb 2026 |
| **TipTap** | seneste 2.x (verificeres i v4) | v3 §13 + FGD-beslutning 2026-05-08 | Rich-text editor; markdown-input via tiptap-markdown extension |
| Tailwind CSS | v4 | v3-analyse | |
| shadcn/ui | latest | v3-analyse | |

Alle versioner re-verificeres ved fase-overgange (MVP→fase 1.5, fase 1.5→fase 2). TipTap-version verificeres eksplicit af Poul i v4-arkitektur.

---

## 9. Afklarede spørgsmål

Alle 7 åbne spørgsmål er afklaret af FGD 2026-05-08. Sektionen er bevaret som beslutningslog.

### 9.1 Native platform-strategi — AFKLARET 2026-05-08

**Beslutning**: **Strategi A — Expo / React Native (ægte native)**.

**Begrundelse**: Mobil-feel signalerer professionalisme i de regulerede vertikaler; reel native performance prioriteres over time-to-market.

**Konsekvens**: ~3-6 ugers ekstra dev-tid for v1; kræver mobil-specialist-rolle (Laila designer).

### 9.2 Hurtig-søgnings-prompt — AI-niveau i MVP — AFKLARET 2026-05-08

**Beslutning**: **Kun søgeboks (FN-40 + FN-41) i MVP. AI-svar (FN-42 + FN-43) i fase 2.**

**Begrundelse**: AI-svar uden god retrieval (kun FTS) bliver lavkvalitet og skaber dårlig første-indtryk. Hybrid søgning (pgvector + RRF) er forudsætning for AI-mode.

### 9.3 Indlejret rich-text editor — AFKLARET 2026-05-08

**Beslutning**: **TipTap rich-text editor med markdown-input-support** fra MVP.

**Begrundelse**: Bedre brugeroplevelse end markdown-only; TipTap dækker både rich-text og markdown-shortcuts/paste i samme editor. FN-08 og FN-10 fusioneret.

### 9.4 Native offline-mængde — AFKLARET 2026-05-08

**Beslutning**: **Kun besøgte noter; slettes fra device efter 1 dag. Data on-device OG in-flight skal krypteres** (AES-256 + iOS Keychain / Android Keystore + TLS 1.3). On-device kryptering obligatorisk — ingen plaintext-fallback.

**Begrundelse**: Konservativ data-minimering for regulerede vertikaler; reducerer eksponering ved tabt/stjålet enhed; aligner med GDPR data-minimering og kommende SOC 2 / ISO 27001-controls.

**Konsekvens**: NFR-30 omskrevet; NFR-30b tilføjet (obligatorisk encryption-storage-check ved app-start).

### 9.5 Push-notifikationer — AFKLARET 2026-05-08

**Beslutning**: **Nej, ikke i v1.** Revurderes fase 3+.

**Begrundelse**: Reducerer scope og leverandør-eksponering (APNs/FCM = ekstra DPA'er, ekstra data-flow). Brugere får notifikationer i app-UI i stedet.

### 9.6 App Store-strategi — AFKLARET 2026-05-08

**Beslutning**: **Single-brand "Knowlagecentral" / FGD Consulting**. Én listing pr. platform.

**Begrundelse**: White-label kræver enterprise distribution + separate listings pr. kunde — markant fordyrende. Single-brand er trivielt og signalerer FGD's afsenderskab.

### 9.7 Kunde-betalt FIPS-tier — AFKLARET 2026-05-08

**Beslutning**: **FIPS native — alle kunder får FIPS, ikke kun dem der betaler.**

**Begrundelse**: FGD-strategisk valg: FIPS er en del af produkt-DNA'et, ikke en tier. Dette eliminerer SOD-kompleksitet (ingen split-tier-kontrol), styrker værditilbud i regulerede vertikaler, og forenkler Mads' compliance-roadmap. Konsekvens: HSM-omkostninger absorberes som fast cost; Mads opdaterer HSM-arkitektur til "FIPS native fra fase 1.5".

**Konsekvens**: NFR-17 omskrevet; Mads' compliance-roadmap revideres; fase 1.5 budget skal afspejle YubiHSM 2 FIPS som obligatorisk for alle kunder.

---

## 10. Revisions-log

| Dato | Forfatter | Ændring |
|---|---|---|
| 2026-05-08 | Poul | Initial udkast. Mission statement (§1) ordret fra FGD-godkendelse. Nyt scope inkorporeret: native iOS/Android (§3), hurtig-søgnings-prompt (§4.5.2), Evernote-funktionalitets-paritet (§4.1-4.4). Versions-verificeret Expo SDK 55, Capacitor 8.3.1. |
| 2026-05-08 | FGD + Poul | Alle 7 åbne spørgsmål afklaret. Status: udkast → vedtaget. Strategi A (Expo) valgt; TipTap rich-text valgt; MVP-søg kun søgeboks; native offline = besøgte noter / 1-dags TTL / AES-256 + Keychain/Keystore + obligatorisk encrypted storage; push fravalgt v1; single-brand App Store; FIPS native for alle kunder. |

---

## 11. Næste skridt

Følgende skridt udløses af vedtagelsen 2026-05-08. Stefan koordinerer; ejer-rolle markeret.

1. **Laila — mobil-specialist-rolle (Strategi A)**:
   Designer ny rolle (eller udvider Sofie) med Expo SDK 55 / React Native 0.83 / iOS-Keychain / Android-Keystore-kompetencer. Skal også dække AES-256 on-device-kryptering og obligatorisk encrypted-storage-check (NFR-30b). Leverance: rolle-mandat i `.claude/agents/` udkast → governance-godkendelse.

2. **Henrik — API-contract-udvidelse (TipTap)**:
   Udvider OpenAPI-contract med:
   - TipTap-JSON som primær note-content-format (afløser plain markdown)
   - Hurtig-søgnings-prompt-endpoint (FN-40 + FN-41 til MVP)
   - Search-payload schema der ikke binder sig til AI-mode endnu (FN-42-43 reserveres som fase 2-version-bump)
   - `.enex`-import-endpoint output-mapping til TipTap-JSON (fase 2)

3. **Mads — HSM-arkitektur-opdatering (FIPS native)**:
   Reviderer HSM-arkitektur til "FIPS 140-3 obligatorisk for alle kunder fra fase 1.5". Opdaterer:
   - `governance/policies/` HSM-policy (fjerner FIPS-tier-differentiering)
   - Compliance-roadmap (SOC 2 / ISO 27001 — FIPS bliver core-control, ikke optional)
   - Budget-impact for fase 1.5 (YubiHSM 2 FIPS × 2 + Hetzner colocation som fast cost)
   - Threat-model: én HSM-arkitektur for alle kunder forenkler trust boundary

4. **Poul — v4-arkitektur-udvidelse**:
   Udvider v3-analyse til v4 med:
   - Native platform-arkitektur (Expo / React Native, Strategi A)
   - TipTap-JSON datamodel + migration fra evt. tidligere markdown-felter
   - Hurtig-søgnings-prompt UI + (fase 2) RAG-pipeline
   - Native offline-cache + sync-strategi (NFR-30 + NFR-30b: 1-dags TTL, AES-256, Keychain/Keystore, obligatorisk encrypted storage)
   - Verifikation af TipTap-version (Context7) før v4 godkendes
   - Krydsreference til Mads' opdaterede HSM-arkitektur (FIPS native)

5. **Camilla — turborepo-struktur**:
   Når Lailas mobil-rolle er godkendt og Pouls v4 er udkast: opretter `apps/mobile/` (Expo SDK 55) i turborepo, deler `packages/zod-contracts` og `packages/ai`. Opdaterer `memory/projects/knowlagecentral/` med vedtaget krav-spec.

6. **Stefan — orkestrering**:
   - Sikrer at krav-spec'ens vedtagelse logges i `Min Inbox/projects/knowlagecentral/`
   - Koordinerer rækkefølge: Laila + Mads kan starte parallelt; Henrik venter på Pouls v4-skitse for TipTap-datamodel; Camilla venter på Lailas rolle-godkendelse + Pouls v4.
   - Eskalerer NFR-17 (FIPS native) til Mads + Dorthe for samlet compliance-impact-vurdering før implementering starter.

