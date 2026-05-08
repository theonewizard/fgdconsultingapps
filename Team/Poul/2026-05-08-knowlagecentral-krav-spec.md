---
dato: 2026-05-08
forfatter: Poul
emne: Knowlagecentral - krav-specifikation
status: udkast
dokumenttype: levende-dokument
relateret_arkitektur: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
relateret_brief: Team Inbox/projects/knowlagecentral/2026-05-05-knowlagecentral-evernote-style.md
versions-verificeret: 2026-05-08 (Expo SDK 55, Capacitor 8.3.1, Next.js 16.2, React 19.2)
---

# Knowlagecentral — Krav-specifikation

> Dette er et levende dokument. Krav tilføjes/justeres efterhånden som FGD og teamet afklarer scope. Ændringer logges i §10 (revisions-log).

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
- Krav: native scroll-performance, system-tastatur-integration, share-extension (modtag tekst fra andre apps), push-notifikationer.

### 3.3 Native Android

- Selvstændig app i Google Play Store, optimeret til touch.
- Samme krav som iOS: native scroll, system-tastatur, share-intent, push-notifikationer.

### 3.4 Hvad "native" betyder i denne kontekst — anbefaling

To realistiske strategier (FGD vælger; se §9 åbne spørgsmål):

| Strategi | Beskrivelse | Konsekvens |
|---|---|---|
| **A: Cross-platform via Expo / React Native** | Én React Native-kodebase deler logik og UI-primitives med web-koden hvor muligt; kompilerer til ægte native iOS + Android binaries. **Verificeret 2026-05-08: Expo SDK 55** (React Native 0.83, New Architecture only). | Reel native performance; én skill-set (React) for hele teamet; deler `packages/zod-contracts` med web. ~3-6 ugers ekstra dev-tid for v1. Sofie kan udvide rolle eller Laila ansætter mobil-specialist. |
| **B: Hybrid via Capacitor** | Web-koden pakkes ind i en native shell (WKWebView/WebView) der eksponerer device-API'er. **Verificeret 2026-05-08: Capacitor 8.3.1** (kræver Node 22+, Xcode 26+ for iOS). | Hurtigere til markedet (~1-2 uger ekstra); samme kodebase som web. Kompromis: scroll-/keyboard-feel er web-niveau, ikke native; nogle App Store-reviewers afviser tynde wrappers. |

**Pouls anbefaling**: Strategi A (Expo / React Native) hvis FGD prioriterer brugeroplevelse i de regulerede vertikaler, hvor mobil-feel signalerer professionalisme. Strategi B (Capacitor) hvis FGD vil have appen i Store hurtigst muligt og accepterer hybrid-feel. PWA alene (uden native shell) er fravalgt jf. det nye scope.

**Begge strategier deler kodebase med web** via turborepo `packages/zod-contracts` og `packages/ai` — API-kontrakter er konsistente på tværs af platforme.

---

## 4. Funktionelle krav

### 4.1 Note-håndtering

| ID | Krav | Prioritet |
|---|---|---|
| FN-01 | Bruger kan oprette ny note manuelt i et valgt notebook | MVP |
| FN-02 | Bruger kan redigere note (markdown-editor, debounced auto-save) | MVP |
| FN-03 | Bruger kan slette note (soft-delete med 30-dages restore-vindue) | MVP |
| FN-04 | System gemmer version-snapshot ved 30s inaktivitet ELLER 200 char akkumuleret diff | MVP |
| FN-05 | Bruger kan se versions-historik, diffe to versioner og rulle tilbage | Fase 2 |
| FN-06 | Bruger kan tilføje/fjerne tags pr. note (manuelle + AI-foreslåede markeret separat) | MVP |
| FN-07 | Bruger kan flytte note mellem notebooks | MVP |
| FN-08 | Note understøtter markdown-syntax (overskrifter, lister, links, kode-blokke, tabeller) | MVP |
| FN-09 | Note kan have vedhæftninger (billeder, PDF) | Fase 3 |
| FN-10 | Rich-text-editor (TipTap eller lignende) i stedet for/ovenpå markdown | Åbent (§9) |

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
| FN-21 | Evernote `.enex`-import (parser konverterer til markdown + tags) | Fase 2 |
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

**Pouls anbefaling**: I MVP (fase 1) leveres FN-40 + FN-41 (kun søgeboks-mode, samme backend som §4.5.1). FN-42 + FN-43 (AI-svar med RAG) leveres i fase 2 sammen med hybrid søgning, fordi RAG-kvalitet er stærkt afhængig af pgvector-similarity.

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
| NFR-17 | FIPS 140-3 produktionsmål for HSM (YubiHSM 2 FIPS, fase 1.5+); non-FIPS i MVP |
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

### 5.5 Offline-support

| ID | Krav |
|---|---|
| NFR-29 | **Web/PWA**: Serwist offline-cache for noter besøgt seneste 7 dage (fase 2) |
| NFR-30 | **Native iOS/Android**: Lokal SQLite-cache for alle noter brugeren har adgang til; sync ved online (fase 2) |
| NFR-31 | Offline-redigering med last-write-wins conflict resolution; CRDT (Y.js) overvejes fase 3 |
| NFR-32 | UI viser tydeligt offline-status + ulæste ændringer |

### 5.6 Observability

| ID | Krav |
|---|---|
| NFR-33 | OpenTelemetry traces propageres via W3C Trace Context (`traceparent` header) |
| NFR-34 | Sentry til errors (web + native) |
| NFR-35 | Vercel Speed Insights til Web Vitals (web) |
| NFR-36 | Strukturerede logs (JSON); ingen PII i logs |
| NFR-37 | Health-endpoints `/livez` + `/readyz` (unauthenticated, minimal info) |

### 5.7 Kost-loft

| ID | Krav |
|---|---|
| NFR-38 | AI-budget < 500 DKK/md (~$70/md), fordelt på prod ($50) / test ($15) / dev ($5) workspace |
| NFR-39 | Infrastructure < $50/md (Vercel Pro $20 + Speed Insights $10 + Neon free + Clerk free + Inngest free) |
| NFR-40 | Native app distribution: Apple Developer Program $99/år + Google Play $25 engangs |

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
- Custom kunde-branding pr. organisation (logo, farver) — kun FGD-branding i v1
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
| §3 Platform-scope | v3 §3.1 (komponent-oversigt) | Native iOS/Android er **nyt** scope; v3 dækker kun web/PWA |
| §4.1-4.2 Note + notebook | v3 §4 (datamodel: `app.notes`, `app.notebooks`) | 1:1 dækket |
| §4.3 Import | v3 §8 (import-pipeline) | 1:1 dækket |
| §4.4 AI-berigelse | v3 §6 (AI-berignings-pipeline) | 1:1 dækket |
| §4.5.1 Standard søgning | v3 §7 (søgning) | 1:1 dækket |
| §4.5.2 Hurtig-søgnings-prompt | **ikke i v3** | **Ny feature** — kræver design-tilføjelse til v3 |
| §4.6 Deling | v3 §5.3 (magic-link-flow) | 1:1 dækket |
| §4.7 Organisation | v3 §4 (`app.organizations`) + §10 (fase 2) | 1:1 dækket |
| §4.8 Eksport | v3 §10 fase 2 | Krav-niveau ikke detaljeret i v3 — udvides ved fase 2 design |
| §5.1-5.2 Sikkerhed/compliance | v3 §5 + Mads' governance-dokumenter | 1:1 dækket |
| §5.5 Offline native | **ikke i v3** | **Ny feature** — kræver design-tilføjelse til v3 |

**Konsekvens**: Når dette krav-dokument er godkendt af FGD, skal v3-analysen udvides med:
1. Native platform-arkitektur (Strategi A eller B per §3.4)
2. Hurtig-søgnings-prompt UI + RAG-pipeline (FN-40 til FN-45)
3. Native offline-cache + sync-strategi (NFR-30, NFR-31)

Stefan/Camilla bør tracke disse som "v4-tilføjelser" i `Team/Poul/`.

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
| **Expo SDK** (hvis Strategi A) | **55** (RN 0.83) | WebSearch 2026-05-08 | New Architecture only; SDK 55 udgivet feb 2026 |
| **Capacitor** (hvis Strategi B) | **8.3.1** | WebSearch 2026-05-08 | Kræver Node 22+, Xcode 26+ |
| Tailwind CSS | v4 | v3-analyse | |
| shadcn/ui | latest | v3-analyse | |

Alle versioner re-verificeres ved fase-overgange (MVP→fase 1.5, fase 1.5→fase 2).

---

## 9. Åbne spørgsmål til FGD

Følgende kræver afklaring før implementering kan starte. Stefan koordinerer svar.

### 9.1 Native platform-strategi

**Spørgsmål**: Strategi A (Expo / React Native, ægte native) eller Strategi B (Capacitor, hybrid web-shell)?

**Konsekvens for tidsplan**:
- Strategi A: ~3-6 ugers ekstra dev-tid for v1. Kræver ny rolle (mobil-specialist) eller udvidelse af Sofie's mandat.
- Strategi B: ~1-2 ugers ekstra dev-tid. Sofie kan dække alene.

**Pouls anbefaling**: Strategi A for de regulerede vertikaler — mobil-feel signalerer professionalisme. Men hvis time-to-first-customer er kritisk, er Strategi B forsvarlig.

### 9.2 Hurtig-søgnings-prompt — AI-niveau i MVP

**Spørgsmål**: Skal hurtig-søgnings-prompten allerede i MVP have AI-svar-mode (FN-42 + FN-43), eller er ren søgeboks (FN-40 + FN-41) nok til v1?

**Konsekvens**:
- MVP med kun søgeboks: leveres på fase 1-tidsplan.
- MVP med AI-svar: kræver hybrid søgning (pgvector + RRF) fremrykkes fra fase 2 til fase 1 → +1-2 uger.

**Pouls anbefaling**: Ren søgeboks i MVP; AI-svar-mode i fase 2. Begrundelse: AI-svar uden god retrieval (kun FTS) bliver lavkvalitet og skaber dårlig første-indtryk.

### 9.3 Indlejret rich-text editor

**Spørgsmål**: Markdown-only editor i MVP, eller TipTap-baseret rich-text fra fase 2?

(Også åbent i v3-analyse §13, gentages her for fuldstændighed.)

### 9.4 Native offline-mængde

**Spørgsmål**: I native apps — skal *alle* brugerens noter cache'es lokalt, eller kun seneste 7 dages besøgte (som web/PWA)?

**Konsekvens**: Fuld cache giver bedre offline-oplevelse men øger initial sync-tid + lokal disk-forbrug. For konsulenter i mødelokaler uden wifi er fuld cache attraktivt.

**Pouls anbefaling**: Konfigurérbar pr. bruger; default = "alle notebooks jeg er ejer af + seneste 30 dage besøgte delte notebooks".

### 9.5 Push-notifikationer

**Spørgsmål**: Skal native apps have push-notifikationer ved AI-berigelse-færdig, ved invite-modtaget, ved kunde-kommentar?

**Konsekvens**: Kræver Apple Push Notification service (APNs) + Firebase Cloud Messaging (FCM) integration; ny rolle eller udvidelse af Anders/Henrik. ~1 uges ekstra dev-tid.

(Også åbent i v3-analyse §13.)

### 9.6 App Store-strategi

**Spørgsmål**: Skal apps udgives under FGD Consulting-brand som "Knowlagecentral", eller white-label så hver kunde kan re-brande?

**Konsekvens**: Single-brand er trivielt. White-label kræver enterprise distribution (Apple Business Manager) eller separate App Store-listings pr. kunde — markant fordyrende.

**Pouls anbefaling**: FGD Consulting single-brand i v1; white-label fase 4+ hvis kunde-vertikalen kræver det.

### 9.7 Kunde-betalt FIPS-tier

**Spørgsmål**: Aktiverer fase 2 FIPS 140-3 HSM automatisk for alle kunder, eller kun kunder der eksplicit kræver det (og betaler)?

**Konsekvens**: FIPS-tier koster €1.187,50 × 2 enheder + Hetzner colocation. Hvis ikke alle kunder betaler, mindsker det runway-pres.

(Krydser med Mads' compliance-roadmap; afklaring sker i samspil mellem Mads og FGD.)

---

## 10. Revisions-log

| Dato | Forfatter | Ændring |
|---|---|---|
| 2026-05-08 | Poul | Initial udkast. Mission statement (§1) ordret fra FGD-godkendelse. Nyt scope inkorporeret: native iOS/Android (§3), hurtig-søgnings-prompt (§4.5.2), Evernote-funktionalitets-paritet (§4.1-4.4). Versions-verificeret Expo SDK 55, Capacitor 8.3.1. |

---

## 11. Næste skridt

1. **FGD review**: Stefan præsenterer dokumentet; FGD svarer på §9 åbne spørgsmål.
2. **Status flyttes til `vedtaget`** når §9 er afklaret.
3. **Laila** designer mobil-rolle (hvis Strategi A) eller udvider Sofie's mandat (hvis Strategi B).
4. **Poul** udvider v3-arkitektur til v4 med:
   - Native platform-arkitektur
   - Hurtig-søgnings-prompt + RAG-pipeline
   - Native offline-cache + sync-strategi
5. **Camilla** opdaterer turborepo-struktur med `apps/mobile/` (hvis Strategi A) eller `apps/web` Capacitor-config (hvis Strategi B).
6. **Henrik** udvider OpenAPI-contract med hurtig-søgnings-prompt-endpoint.
