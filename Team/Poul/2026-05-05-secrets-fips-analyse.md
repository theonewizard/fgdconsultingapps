---
dato: 2026-05-05
forfatter: Poul
emne: Secret-handling, FIPS 140-2 Level 3 vurdering, behov for security-rolle (Karen)
status: leveret
relateret: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
---

# Secret-handling & FIPS 140-2 — Analyse for Knowlagecentral

## 1. Topline

FGD's bekymring om "API-keys i database" er **teknisk begrundet, men løst rigtigt i v3-design**. API-keys lever i Vercel environment variables (KMS-backed under overfladen); kun sha256-hashes af invite-tokens er gemt i Postgres, som er korrekt praksis.

**FIPS 140-2 Level 3-certificering er praktisk umulig for SaaS-stakken uden hardware security module (HSM)**. Omkostning: +$3–5k/måned + 3–6 måneders arbejde. Mere pressant: FIPS 140-2-standardens acceptperiode endte 1. apr 2022; CMVP sunset alle certs 21. sept 2026.

**Realistisk mål**: "FIPS-aware best practices" ved at kombinere Vercel Vault (KMS-backed) + Infisical ($18–50/md, FIPS 140-3-compliant) + AWS KMS envelope-encryption for PII (~$1/md per 10k operationer). Total: ~$50–100/md ekstra end baseline. Parallelt: Ansæt **Karen**, deltids security architect (rådgiver, ikke prod-kode), til threat-modellering + secure code review i alle faser.

---

## 2. Spor A — Secret-inventar: Hvor hver secret lever

| Secret | Værdi-niveau | Primær lagring | Håndtering | Status i v3 |
|---|---|---|---|---|
| `ANTHROPIC_API_KEY` | høj | Vercel Vault (KMS-backed native integration) | Automatisk env-injection pr. miljø; rotér halvårligt | ✓ korrekt |
| `VOYAGE_API_KEY` | høj | Vercel Vault | Automatisk env-injection pr. miljø | ✓ korrekt |
| `CLERK_SECRET_KEY` | høj | Vercel Vault | Automatisk env-injection pr. miljø | ✓ korrekt |
| `INNGEST_EVENT_KEY` + `INNGEST_SIGNING_KEY` | høj | Vercel Vault | Automatisk env-injection pr. miljø | ✓ korrekt |
| `SENTRY_DSN` | medium | Vercel Vault | Automatisk env-injection (offentlig DSN-del acceptabel) | ✓ korrekt |
| `CRON_SECRET` | høj | Vercel Vault | Hardkode og match i Vercel Cron signed requests | ✓ korrekt |
| `DATABASE_URL`, `DATABASE_URL_UNPOOLED`, `PGHOST`, `PGUSER`, `PGPASSWORD` | høj | Neon Native Integration auto-injection | Neon-project-level secrets, ikke manuelt i Vercel UI | ✓ korrekt |
| Clerk JWT private key | høj | Clerk-managed via Clerk console | Private key forbliver hos Clerk; ingen lager-sted i app | ✓ korrekt |
| Magic-link invite tokens | medium | Postgres `notebook_invites.token_hash`, `organization_invites.token_hash` | sha256-hash kun; raw token i email-link engangs (single-use + 7-days expiry) | ✓ korrekt |
| **Hvad der LIGGER CLEARTEXT i Postgres** (OK): | | | | |
| Note-body (markdown) | low-medium | Postgres with Neon AES-256 at-rest + table-level RLS | Per-user notebook-adgang via RLS | ✓ acceptable |
| AI-summaries | low | Postgres with AES-256 + RLS | Kun tilgængelig via RLS-filtrering | ✓ acceptable |
| Embeddings | low | Postgres pgvector + RLS | Bare tal; ingen PII | ✓ acceptable |
| Audit-payload | medium | Postgres JSON + RLS | Indeholder handling + user-ID, ikke secrets | ✓ acceptable |
| Kunde-emails | **PII** | Postgres cleartext! | Indsamlet ved organization-invite; NOT encrypted app-layer | ⚠️ skal adresseres |

---

## 3. Spor B — FIPS 140-2 Level 3 i detaljer

### 3.1 De fire niveauer (kortversjon)

| Niveau | Definition | Typisk hardware | Relevans for SaaS |
|---|---|---|---|
| **1** | Software-only OK; minimal modstands mod manipulation | Almindeligt CPU + OS | AWS KMS-backed Vercel-vault |
| **2** | Tamper-evident; fysiske låse/segmenter | Crypto-modul på kort | Begræns til højtillidsmiljøer (gov, finans) |
| **3** | Tamper-resistant; kryptografisk identitets-authentificering + trusted I/O path | Fysisk HSM med røveliste | AWS CloudHSM, Thales Luna, SoftHSM (test-only) |
| **4** | Fuld fysisk envelope + sensor-array; detektér og ødelæg sensitive materiale ved intrusion | Specialiseret sikkerhedskredsløb | Finans-sektoren; Microsoft Azure Dedicated HSM |

**Kritisk**: Software-baserede secret-managers (Hashicorp Vault, Infisical, 1Password, AWS Secrets Manager) kan MAKSIMALT opnå Level 2-certificering. Level 3+ kræver fysisk hardware.

### 3.2 SaaS-stack-status

| Leverandør | Offentlig FIPS-cert | Niveau | Kommentar |
|---|---|---|---|
| **Vercel** | Ingen offentlig cert | — | Bruger AWS-underliggende KMS (Level 3-cert) men publicerer ikke sig selv |
| **Neon Postgres** | Ingen offentlig cert | — | Neon Infrastructure bruger AWS — dermed implicit KMS-protegeret |
| **Clerk** | Ingen offentlig cert | — | Har SOC 2 Type II; secrets-håndtering unclear |
| **Anthropic** | Ingen offentlig cert | — | API-keys i standard HTTPS; anbefalede "store i env, ikke i kode" |
| **Inngest** | Ingen offentlig cert | — | Webhook-signing via HMAC-SHA256; secrets i Vercel integration |
| **Voyage AI** | Ingen offentlig cert | — | Svageste compliance-profil: eneste doc er privacy policy + SSL |
| **AWS KMS** | ✓ Level 3 certified | 3 | Under-the-hood for Vercel Vault; bedste praksis for sensitive data |
| **Infisical** | ✓ FIPS 140-3 compliant (in planning) | 3 | Self-hosted option; SOC 2 Type II i cloud |

**Konklusion**: Ingen fra SaaS-stakken har offentlig FIPS 140-2/3-certificering. Vercel + Neon bruger AWS KMS (Level 3) under overfladen, men det er ikke transparent.

### 3.3 FIPS 140-2 sunset

- **CMVP decision (1. apr 2022)**: Stop accept af nye FIPS 140-2-valideringer.
- **Sunset date (21. sept 2026)**: Alle eksisterende 140-2 validerings går til "Historical List"; producenter skal migrere til FIPS 140-3.
- **Implikation for FGD**: Hvis en kunde _kræver_ FIPS 140-2-cert i dag, skal du være klar på at migrere til 140-3 inden sept 2026 — eller acceptere at det bliver "legacy compliance" status.

---

## 4. Spor C — Realistisk sikkerhedsniveau for knowlagecentralen

### 4.1 Trusselsmodel

- **Målgruppe**: 3–5 små-til-mellemstore virksomheder (SMV'er) i Danmark/Sverige, 100–200 noter samlet.
- **Data-klassifikation**: Ikke fra regulerede brancher (Poul: bekræft hvis kundevertikalen indeholder finans/sundhed/offentlig).
- **Antagelse**: Standardiserede sikkerhedskrav (GDPR + "rimelig sikkerhed"), ikke specifik statlig/bankstyret compliance.

### 4.2 Anbefaling: "FIPS-aware best practices"

**I stedet for fuld Level 3-certificering**, implementér et lag-based approach:

#### Lag 1: Vercel Vault KMS-backed (baseline)
- Alle runtime-secrets (`ANTHROPIC_API_KEY`, `CLERK_SECRET_KEY`, etc.)
- Automatisk rotation pr. miljø
- Håndteret af Vercel Native Integration
- **Øverste omkostning**: $0 (included i Vercel Pro)

#### Lag 2: Infisical for højtværdi-secrets (fase 1 eller 2)
- Selv-hostet eller cloud-instans
- FIPS 140-3-roadmap;SOC 2 Type II i dag
- Håndter API-key-rotation + audit-log + team-access-kontrol
- **Omkostning**: $18–50/md (self-hosted; cloud-variantはyet dearer)
- **Opsætning**: 1–2 dage for Henrik + Camilla

#### Lag 3: AWS KMS envelope-encryption for PII (fase 2)
- Kunde-emails + andre PII-felter: optionelt krypter på app-layer før DB-insert
- Master key i AWS KMS (Level 3-cert); data-key roteret pr. record
- **Omkostning**: ~$1/md per 10k operations
- **Kompleksitet**: 1–2 dage; Camilla + Henrik designerer Drizzle-hooks

#### Lag 4: Periodisk threat-modeling (fase 0+1+2)
- Karen faciliterer "STRIDE"-øvelse med Henrik + Anders
- Identificér nye risici pr. feature-fase
- Opdatér DPA-oversigt pr. leverandør

### 4.3 Alternativer fravalgt

| Alternativ | Pris | Kompleksitet | Hvorfor fravalgt |
|---|---|---|---|
| HashiCorp Vault Enterprise + HSM | $2–5k/md + hardware | 4–6 mdr | Massive overengineering for SMV-scope |
| AWS CloudHSM (ét HSM) | $870/md + data-transfer | 2–3 mdr | Overkill uden eksisterende AWS-adoption |
| Thales Luna HSM (on-prem) | $10–20k engangspris + $5k/år | 1–2 mdr setup | Kræver fysisk datacenter-plads |
| In-house "secret rotation" CLI | $0 | Vedvarende maintenance | Fejlprone; Infisical gør det bedre |

---

## 5. Spor D — Karen: Ny security architect-rolle

### 5.1 Rolledefinition

**Navn**: Karen (alternativer: Mette, Helle, Tina — afventer Lailas designer)

**Type**: Deltids-rådgiver (ikke fuld dev; ingen prod-kode)

**Model**: **Sonnet** (claude-sonnet-4-6). Kan eskalere til Poul ved dyb research/compliance-analyse.

**FTE-estimat**: 0,25–0,5 FTE spread over 5–7 uger (Fase 0 [threat-model] → Fase 1 [code-review] → Fase 2 [pre-launch pentest management]).

### 5.2 Kompetencer (kernekrav)

- Cryptography fundamentals (symmetric/asymmetric, hashing, signing)
- FIPS 140-2/3 + CMVP-landskab
- OAuth 2.1 + OpenID Connect (deep dive på JWT-validering, refresh-rotation)
- OWASP API Top 10 (dybde på BOLA, BOPLA, rate-limiting, SSRF)
- Threat-modelling (STRIDE, risk-matrix)
- Secure code review (API, crypto, data-flow)
- Databehandleraftaler (DPA) + GDPR-konsekvenser
- Secret-management (rotation, versioning, audit)

### 5.3 Ansvarsområder pr. fase

| Fase | Opgaver | Leverandel |
|---|---|---|
| **Fase 0** (uge 1) | Threat-model STRIDE-øvelse (Alice — FGD, Bob — kunde, Eve — attacker). Secret-strategi + DPA-tabel. Anbefal: Infisical y/n, KMS envelope-encryption y/n. | Threat-model-dokument + decision-matris |
| **Fase 1** (uge 2–3) | Secure code review Henrik's Route Handlers + RLS-policies. JWT validering-spot-check. Zod .strict() coverage. Inngest step-function audit. Idempotency + RFC 9457 error-leak check. | Code-review notat pr. område (handoff til Henrik for fixes) |
| **Fase 2** (uge 4–5) | Pre-launch pentest management + DPA-forfald-check. Help Camilla kontrahere eksternt pentest-firma (ønsket: 3–5 dage limited-scope test på staging). Verificér Anthropic ZDR-flag. | Pentest report + remediation-prioritering |

### 5.4 Tools (least-privilege + constraints)

**Allow**:
- `Read` (all files)
- `Glob`, `Grep` (grep over codebase for patterns)
- `WebSearch`, `WebFetch` (research OWASP, FIPS, threat-intel)
- `Bash` (read-only: `git status`, `git log`, `grep`, `wc`, o.l. — **IKKE edit/rm**)
- `Agent` (delegate to Camilla + Poul for deep analysis, DB work, git)
- `TodoWrite` (track security checklist progress)

**Deny**:
- `Write`, `Edit` til produktionskode eller migrations
- `Write`, `Edit` til noget uden Stefan/FGD-godkendelse
- Skrive direkte til `memory/` (Camilla ejer det)

**Arbejdsmappe**: `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/Karen/` (Camilla opretter når Laila designer rollen).

### 5.5 Samspil med Henrik + Anders

- **Henrik** (backend): Karen reviewerer hans Route Handlers, RLS, JWT-integrationskode.
- **Anders** (AI-pipeline): Karen tjekker Inngest-jobs for auth-bypass, spend-cap-mekanismer, prompt-injection-risici.
- **FGD** (ejer): Karen advisorer på DPA-negotiation + pentest-timing.
- **Camilla** (DB/infra): Karen advisorer på Infisical-opsætning + KMS-envelope-encryption design.

---

## 6. Pouls fem anbefalinger til Stefan (håndsrækning til FGD)

1. **Spørg FGD konkret**: "Hvad observerede du der vakte bekymring omkring secrets i Postgres?" (Læringspunkt: så han en gammel version af designdokumentet, eller har han nye data?)

2. **Afvis Level 3-certificering udtrykkeligt**: "FIPS 140-2 Level 3 koster $3–5k/md + 3–6 mdr; FIPS 140-2 sunset 21. sept 2026. Ikke givet i SMV-scope."

3. **Foreslå best-practices-tilgang**: "Vi implementerer FIPS-aware best practices for $50–100/md ekstra — Infisical + KMS-envelope for PII. Startter fase 2, efter at fase 1 er stabil."

4. **Ansæt Karen i Fase 0**: "Threat-modelling + secret-strategi foruden før fase 1 dev starter. Sikrer vi rammer fejl tidligt, ikke på prod."

5. **Opdater memory med beslutning**: "Hvis FGD bekræfter best-practices-vej, skriver Camilla det ned som referencepunkt for fremtidt security-arbejde."

---

## 7. Fem åbne spørgsmål til FGD

1. **Konkret observation**: Hvad præcis var det i secret-håndteringen der bekymrede dig? (Gammel kladde, specifikt felt, eller mere generel bekymring?)

2. **Kunde-vertikal**: Vil dine første kunder være fra regulerede brancher (finans, sundhed, offentlig)? (Påvirker compliance-ambitonsniveau.)

3. **Budget for sikkerhed**: Er +$50–100/md i ekstra infra-omkostninger acceptable for FIPS-aware best practices?

4. **ISO 27001 / SOC 2-roadmap**: Skal knowlagecentralen tilstræbe ISO 27001-certificering eller "blot" SOC 2 Type II på sigt?

5. **Karens omfang**: Deltids-rådgiver (1–2 timer/uge) eller mere fuldt sideløbende med dev-teamet (8–10 timer/uge)?

---

## 8. Sources & Referencer

1. FIPS 140-2 (NIST) — https://csrc.nist.gov/projects/cryptographic-module-validation-program/fips-140-2
2. FIPS 140-3 (NIST) — https://csrc.nist.gov/publications/detail/fips/140-3/final
3. CMVP Historical List (FIPS 140-2 sunset) — https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules
4. Vercel Vault (Native Integration) — https://vercel.com/docs/storage/vercel-postgres/vault
5. Neon Native Integration — https://vercel.com/integrations/neon
6. Infisical Docs — https://infisical.com/docs
7. Infisical Pricing — https://infisical.com/pricing
8. AWS KMS Pricing — https://aws.amazon.com/kms/pricing/
9. Clerk Security — https://clerk.com/docs/security
10. Anthropic API Security — https://docs.anthropic.com/en/api/api-keys
11. OAuth 2.1 (RFC 6749 refresh) — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1
12. OpenID Connect Spec — https://openid.net/specs/openid-connect-core-1_0.html
13. OWASP API Security Top 10 (2023) — https://owasp.org/www-project-api-security/
14. STRIDE Threat Modeling — https://www.microsoft.com/en-us/security/blog/2007/09/11/stride-chart/
15. GDPR & Encryption — https://ec.europa.eu/info/law/law-topic/data-protection_en
16. AWS CloudHSM Pricing — https://aws.amazon.com/cloudhsm/pricing/
17. HashiCorp Vault Enterprise — https://www.hashicorp.com/products/vault

---

**Status**: Analyse leveret; afventer FGD-godkendelse af best-practices-vej og Karen-ansættelse.
