---
dato: 2026-05-05
forfatter: Mads (CISO)
emne: First-session leverance — governance-rammer, compliance-teams, skill-gap, rolle-gap, mandat-review, pen-test-vendors
status: leveret
relateret: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md, Team/Camilla/2026-05-05-skill-inventar.md, Team/Poul/2026-05-05-secrets-fips-analyse.md, Team/Poul/2026-05-05-yubihsm-integration-analyse.md, Team/Poul/2026-05-05-vercel-undersoegelse.md
---

# Mads' First-Session Leverance — Konsolideret Governance-vurdering

## 1. Sammendrag

FGD's knowlagecentral har et solidt teknisk fundament (v3-arkitektur er gennemtænkt; Layer 2 HSM er state-of-the-art for SMV-segmentet), men **governance-rammen er endnu ikke skrevet**. De 5 project-mandater i `Min Inbox/` er kompetent designet af Laila, men adresserer ikke systematisk de fire compliance-spor (GDPR, ISO 27001:2022, SOC 2 Type II, FIPS 140-3) som FGD har truffet beslutning om at adressere parallelt. SOD-overhold er delvist indbygget (Karen vs. Bjarne, Mads vs. Karen), men ikke konsekvent — Henrik kan i dag mergde sin egen kode, og Anders ejer både prompt-bibliotek og audit-log-emit fra Inngest.

**Største 5 fund**:

1. **Manglende deploy-gate som SOD-håndhæver**: Ingen mandat kræver tvungen reviewer ≠ author på PRs til `apps/web/`, migrations, eller HSM-bridge. Dette er SOC 2 CC8.1 og ISO 27001 A.8.32-blocker.
2. **Karen er overbelastet**: Hendes mandat dækker threat-modeling, secure code-review, FIPS-roadmap, DPA-review, key-ceremony-design, pen-test-koordination OG GDPR/ISO/SOC 2-controls. Dette er 2-3 fuldtidsroller pakket i én deltidsrolle. Risiko: ingen af sporene færdiggøres til audit-niveau.
3. **GDPR-rolle (DPO) mangler**: Vi har ingen ansvarlig for ROPA, DPIA, data-subject-rettigheder (GDPR art. 30, 35, 12-22), breach-notification (72-timer art. 33). Karen kan ikke være DPO når hun samtidig er security-architect — det er rolleblanding.
4. **Audit-trail-strategi er ikke designet**: Henrik logger til `audit.events`, men ingen har defineret immutability (append-only, hash-chain), retention (GDPR vs. SOC 2 modstrider hinanden), eller ekstern arkivering. Dette er ISO 27001 A.8.15 og SOC 2 CC7.2-blocker.
5. **Skill-inventar har ~55 ikke-vurderede skills**: Adskillige sikkerhedsrelevante skills (`security-review`, `agent-browser`, `e2e-testing`, `skill-comply`) er installeret men ikke knyttet til mandater — risiko for skygge-anvendelse uden governance.

**Anbefalede nye roller**: 3 (Dorthe — DPO; Trine — Compliance-auditor; Frederik — Pen-test-koordinator). De 5 udkast-mandater anbefales godkendt med justeringer, IKKE som de står.

**Status pr. compliance-standard**:

| Standard | Status | Blocker |
|---|---|---|
| **GDPR** | Rød — ingen DPO, ROPA mangler, DPA-tracker er på Karen | DPO-rolle skal designes (Dorthe) |
| **ISO 27001:2022** | Gul — Annex A controls ikke mappet | Compliance-auditor (Trine) |
| **SOC 2 Type II** | Gul — Trust Service Criteria ikke mappet, evidence-pipeline mangler | Compliance-auditor (Trine) |
| **FIPS 140-3** | Grøn (på vej) — Karen + Bjarne ejer det; afventer cert Q2 2026 | Cert-status hos Yubico |

**Vigtigste åbne spørgsmål til FGD**: (1) Antal nye roller — godkender du 3 (Dorthe/Trine/Frederik) eller foretrækker du udvidelse af eksisterende (men det rammer SOD)? (2) Pen-test-vendor — accept af "vendor-rotation hvert 2. år" som SOC 2 best-practice? (3) Audit-log-retention — GDPR ønsker minimum, SOC 2 ønsker 1 år, ISO 27001 ønsker 3 år; vi vælger 3 år som baseline?

---

## 2. Governance-rammer

Disse syv principper er det filter Mads bruger til alle øvrige sektioner. De skal vedtages før de 5 project-mandater aktiveres.

### 2.1 Separation of Duties (SOD) — eksplicit

**Definition for FGD**: Udvikler ≠ reviewer ≠ approver. Dette er SOC 2 CC8.1, ISO 27001 A.5.3, A.8.32. Min fortolkning af FGDs SOD-beslutning fra dag i dag:

- En udvikler (Sofie/Henrik/Anders) kan **aldrig** merge sin egen PR til `main`.
- Karen reviewer security-impacted PRs; **kan ikke selv være author** på prod-kode (allerede i hendes mandat — bevares).
- Bjarne eksekverer HSM-ops; **kan ikke selv designe runbooks** (allerede i mandat — bevares).
- Mads (jeg) kan **ikke selv merge** governance-policies — FGD eller Stefan godkender.
- Camilla committer på vegne af andre roller; må **ikke skrive selvstændigt** indhold til policies/threat-models/runbooks.

**Implikation**: Vi skal have minimum 2 backend-udviklere senest når Henrik bliver flaskehals (fase 2), eller en eksplicit "buddy reviewer" fra Karen + en anden FGD-godkendt reviewer på Henriks PRs.

### 2.2 Compliance integreret design (ikke add-on)

Hver feature, hver integration, hver migration skal **før implementering** scores på fire compliance-akser:

| Akse | Spørgsmål | Vurderes af |
|---|---|---|
| GDPR | Tilfører feature ny PII? Ny processing-purpose? Ny third-party data-flow? | Dorthe (DPO) |
| ISO 27001:2022 | Hvilke Annex A controls påvirkes? (typisk A.5/8/14) | Trine (compliance-auditor) |
| SOC 2 Type II | Hvilken Trust Service Criterion påvirkes? (CC, A, C, P, PI) | Trine |
| FIPS 140-3 | Bruger feature crypto? Hvilket modul? Hvilket key-niveau? | Karen + Mads |

**Form**: Lille tjek-skabelon `governance/compliance/feature-scorecard-template.md` (jeg leverer i fase 0). Henrik/Sofie/Anders udfylder ved hver væsentlig PR; Trine/Dorthe gennemgår.

### 2.3 Defense-in-depth: API + RLS + property-level + HSM

Ingen feature må stå på ét sikkerhedsbarriere:

1. **Lag 1 — API**: Clerk JWT alg-pinned (RS256), `iss/aud/exp/nbf/sub` valideret, eksplicit `organization_members` tjek.
2. **Lag 2 — RLS**: Neon `pg_session_jwt` per row.
3. **Lag 3 — property/field**: Zod `.strict()` reject ukendte felter, BOPLA-mitigering.
4. **Lag 4 — HSM**: signing/encryption-ops via YubiHSM bridge for high-value (invites fase 1.5; field-level fase 2).

Dette er allerede i v3 — jeg gør det eksplicit governance-krav: hvis en feature mangler ét lag, kommer den ikke i prod uden Mads-undtagelse + dokumentation.

### 2.4 Audit-trail: immutable + append-only

Hver mutation til `app.*` tabeller skriver til `audit.events`. Krav (skal designes detaljeret af Karen + Trine):

- **Append-only** via DB-grant (`REVOKE UPDATE, DELETE ON audit.events FROM app_user`).
- **Hash-chain** mellem rækker (forrige row's `event_hash` indgår i denne row's payload — tamper-evidens).
- **Retention**: 3 år (lex SOC 2/ISO/GDPR-konflikt; vi vælger længste, dokumenterer i ROPA).
- **Ekstern arkivering**: Månedlig export til S3 Glacier eller tilsvarende WORM-storage (fase 2).
- **Read-access**: Kun Trine + Mads + Stefan har query-adgang til `audit.events` på prod.

### 2.5 Threat-model pr. ny feature/integration

**Hard rule**: Før en ny feature går i implementering, skal Karen levere mini-STRIDE (max 1 side) og Mads godkende. Eksempler hvor det udløses:

- Ny third-party leverandør (fx Inngest-replacement, ny AI-provider)
- Ny PII-felt
- Nyt write-endpoint
- Ny crypto-operation
- Ændring i auth-flow

For FGD's regulerede vertikal er dette et kunde-tillidskrav, ikke kun internt.

### 2.6 Data-flow tracking for GDPR (ROPA)

ROPA (Record of Processing Activities, GDPR art. 30) skal være levende dokument. Jeg foreslår tabel-struktur i `governance/dpa/ropa.md`:

| Processing-aktivitet | Formål | Data-kategorier | Data-subjects | Modtagere | Tredjelands-overførsel | Retention | Sikkerhedsforanstaltninger |
|---|---|---|---|---|---|---|---|

Dorthe ejer denne ved aktivering. Indtil Dorthe ansat: Mads vedligeholder midlertidigt.

### 2.7 Encryption: at-rest, in-transit, key-management

- **At-rest**: Neon AES-256 (Postgres-niveau) + field-level encryption på PII-felter (kunde-emails) via HSM-baseret envelope (fase 2). Karen designer.
- **In-transit**: TLS 1.3 mandatory; Vercel platform tvinger det. Tailscale = WireGuard mTLS.
- **Key-management**: YubiHSM Layer 2 (online + offline) med wrap-key-ceremony (Karen + Bjarne, 4-eyes). Halvårlig rotation. Cert-target: FIPS 140-3 Q2 2026.

### 2.8 Løbende teknologi-vurdering (ikke engangs-runde)

FGD har eksplicit krævet at governance-policy sikrer kontinuerlig teknologi-vurdering. Jeg foreslår:

- **Kvartalsvis**: Mads + Poul peer-runde over installerede skills + dependencies. Output: rapport i `governance/policies/quarterly-review-<dato>.md`.
- **Ved hver minor version-bump**: Henrik/Sofie/Anders kører `context7` + Trine kører compliance-impact-tjek.
- **Ved hver major version-bump**: Threat-model fra Karen + Mads-godkendelse før merge.
- **Vendor-DPA-review**: Halvårligt, eller ved DPA-version-update (Dorthe).

---

## 3. Hold-sammensætning — 5 parallelle compliance-teams

Bemærk: FGD har truffet beslutning om **4 parallelle compliance-teams**. Jeg tilføjer Pen-test som dedikeret 5. team (kører tværs over de 4 og er pre-launch-blocker for SOC 2 Type II), så vi får 5 teams i alt.

### Team 1 — GDPR

**Lead**: Dorthe (DPO, ny rolle)
**Deltagere**: Karen (security-perspektiv), Henrik (data-flow i kode), Anders (AI-dataflow til Anthropic/Voyage), Camilla (DPA-tracker, ROPA-DB)
**Skills**: `context7` (GDPR-update tracking), `superpowers:writing-plans`, `deep-research`
**Leverancer pr. fase**:
- Fase 0: ROPA v1, DPA-tracker udfyldt for alle 8 leverandører, DPO-kontaktoplysninger publiceret
- Fase 1: DPIA for AI-berigelse + magic-link, breach-respons-runbook (72-timer)
- Fase 2: Customer-DPA-template, data-subject-request flow (export/delete), SCCs-tjek for tredjelands-overførsel

### Team 2 — ISO 27001:2022

**Lead**: Trine (compliance-auditor, ny rolle)
**Deltagere**: Mads (governance-policies), Karen (Annex A.8 controls), Henrik (A.8.25 secure development), Camilla (A.8.13 backup), Bjarne (A.8.24 cryptography)
**Skills**: `superpowers:writing-plans`, `context7` (ISO 27001:2022 updates), `code-review`
**Leverancer pr. fase**:
- Fase 0: Statement of Applicability (SoA) — hvilke af 93 Annex A controls er i scope
- Fase 1: Control-implementering pr. Annex A — dokumenteret + testet
- Fase 2: Internal audit cycle 1 (Trine reviewer evidens; Mads underskriver)

### Team 3 — SOC 2 Type II

**Lead**: Trine (compliance-auditor — samme person som ISO 27001 indtil scope vokser)
**Deltagere**: Mads, Karen, Henrik, Camilla, Anders
**Skills**: `superpowers:writing-plans`, `code-review`, `security-review`
**Leverancer pr. fase**:
- Fase 0: Trust Service Criteria mapping (vi går efter Security + Confidentiality + Availability i v1; Privacy + Processing Integrity overvejes fase 3)
- Fase 1: Evidence-pipeline (audit-log → S3 Glacier; CI-attest; access-review-cadence)
- Fase 2: Type II observation period start (typisk 6+ måneder før revisor); månedlig evidence-review

### Team 4 — FIPS 140-3

**Lead**: Karen (security-architect)
**Deltagere**: Mads (godkender), Bjarne (eksekverer), Henrik (bridge-service kode), Trine (audit-evidens)
**Skills**: `context7` (FIPS 140-3 + Yubico cert-status), `superpowers:systematic-debugging`
**Leverancer pr. fase**:
- Fase 0: FIPS 140-3 roadmap-dokument (afhængighed af Yubico cert Q2 2026)
- Fase 1.5: HSM live på non-FIPS hardware; design + dokumentation matcher FIPS 140-3 cert-niveau
- Fase 2+: Cert-validering migration når Yubico FIPS-enheder ankommer; dokumentation opdateret til formelt niveau

### Team 5 — Pen-test (cross-cutting, SOC 2-blocker)

**Lead**: Frederik (pen-test-koordinator, ny rolle)
**Deltagere**: Mads (RoE-godkender), Karen (scope + remediation-prioritering), Henrik (fix-implementering), Trine (audit-evidens)
**Skills**: `agent-browser` (vurdér internt før vendor-test for at tilbyde "før-pen-test-tjek"), `e2e-testing`, `security-review`
**Leverancer pr. fase**:
- Fase 0: Vendor-shortliste, RoE-template, scope-definition (jf. §7)
- Fase 1: Internt pre-pen-test (Frederik kører `agent-browser` + `security-review` + `e2e-testing` for tidlige fund)
- Fase 2: Eksternt pen-test (3-5 dage limited-scope på staging), remediation-tracking, attest-rapport

---

## 4. Skill-gap-analyse pr. team

### 4.1 Pr. team — installerede skills, huller, overlap

| Team | Dækket af installerede skills | Hul | Anbefalet handling |
|---|---|---|---|
| **GDPR** | `context7`, `deep-research`, `superpowers:writing-plans` | Ingen GDPR-specifik skill | OK — DPO arbejder primært med dokumenter, ikke kode. Tilstrækkeligt. |
| **ISO 27001** | `context7`, `code-review`, `superpowers:writing-plans` | Ingen ISO-specifik skill | OK — `skill-comply` evalueres som potentielt audit-evidence-værktøj (§4.2). |
| **SOC 2** | `code-review`, `security-review`, `context7` | Ingen continuous-monitoring skill | Vurdér om `skill-comply` kan automatisere "agent-følger-regel" attest |
| **FIPS 140-3** | `context7`, `superpowers:systematic-debugging` | Ingen FIPS-specifik skill | OK — Yubico-cert-tracking via `context7` + WebFetch er tilstrækkeligt |
| **Pen-test** | `agent-browser`, `e2e-testing`, `security-review`, `superpowers:systematic-debugging` | Ingen DAST/SAST skill | Vurdér om der findes Semgrep/Trivy/Gitleaks-skills i Vercel-plugin (§4.2) |

### 4.2 Specielle fokus-skills (FGDs forespørgsel)

| Skill | Vurdering | Anbefaling |
|---|---|---|
| **`vercel:auth`** | Clerk via Marketplace-integration. Kan simplificere env-vars og DPA-flow (Vercel = sub-processor for Clerk). | **Installér + brug** ved fase 0 setup. Henriks mandat justeres så Clerk-integration sker via `vercel:auth`. SOC 2-fordel: en mindre direkte sub-processor at tracke (Vercel håndterer det). |
| **`vercel:next-forge`** | Monorepo-framework alternativ til vores manuelle Turborepo. **Ikke anbefalet skifte** for v1 — vi har allerede besluttet Turborepo og monorepo-struktur er låst i Henriks mandat. Skift = scope-creep. | **Skip** for v1. Re-evaluér ved fase 3. |
| **`vercel:workflow`** | Vercel Workflow DevKit konkurrerer med Inngest. **Karen skal threat-modelle** før vi overvejer skifte. Inngest har retry/DLQ/concurrency-control der er audit-relevante for SOC 2 CC7.1. | **Skip** for v1. Ander's mandat fastholder Inngest. Re-evaluér fase 2 hvis Inngest cost/lock-in bliver problem. |
| **`vercel:ai-gateway`** | Model-routing + cost-tracking + rate-limiting via Vercel. **Kunne være DPA-fordel** (Vercel = sub-processor for Anthropic, vi tracker færre direkte DPA'er). Cost-tracking er SOC 2 CC7.2-evidens for kapacitets-management. | **Vurdér i fase 1** (ikke MVP-blocker). Anders får opgaven: kør spike i fase 1.5 om migration giver compliance-fordel. |
| **`vercel:runtime-cache`** | Server-side cache med tag-invalidering. **Compliance-neutral**; performance-optimering. | OK at bruge når Sofie/Henrik vurderer det. Ingen governance-impact. |
| **`agent-browser`** | Browser-automation. **Stærk pen-test-værdi** for Frederik (pre-pen-test-scan på staging). Også DAST-pre-tjek. | **Installér og knyt til Frederiks mandat**. Begrænset til staging-miljø; aldrig prod uden eksplicit Mads-godkendelse. |
| **`e2e-testing`** | Playwright. **Ja, standard-test-stack**. Compliance-fordel: e2e-tests = SOC 2 CC7.1-evidens for "controls operate as designed". | **Installér og gør standard**. Sofies + Henriks mandater udvides så e2e-tests skal eksistere før merge for `apps/web/` features. |
| **`security-review`** | Audit-skill med checkliste-baseret review. **Ja, deploy-gate**. | **Installér som CI-step**. Kører automatisk på PRs der ændrer auth/crypto/RLS-paths. Karen reviewer output, men automatik fanger lavt-hængende frugt. |
| **`skill-comply`** | Auto-genererer scenarier i 3 strictness-levels og måler om agents følger reglerne. **Stærk SOC 2 CC1.4-evidens** — beviser at policies følges, ikke bare eksisterer. | **Installér og kør månedligt** mod alle 10 agents. Trine ejer rapporter. Dette er et governance-game-changer. |

### 4.3 Duplikat-konsolidering (fra Camillas inventar)

5-7 skills har multiple kilder (`ai-sdk`, `next-cache-components`, `react-best-practices`, `shadcn`, `neon-postgres`). **Min anbefaling**: Vælg Vercel-plugin-versionen som primær for de skills der findes der (vedligeholdes af platform-ejer), og fjern eller skjul `.agents/skills`-duplikatet for at undgå version-skifter.

**Beslutning skal ske i Camillas regi** efter Mads/Poul-runden — ikke i denne leverance.

### 4.4 Skills til at fjerne fra inventar

Følgende skills er **irrelevante** for knowlagecentral og bør marker `not-applicable` (de bør ikke fjernes — kan være relevante for FGD's andre projekter):

- `python-patterns`, `python-pep8-review`, `python-testing` (ikke vores stack)
- `springboot-*` (ikke vores stack)
- `react-three-fiber`, `remotion`, `remotion-best-practices`, `ucp`, `video-checksum-pipeline` (ikke vores feature-set)
- `shadcn-svelte`, `svelte`, `vue`, `solid` (vi bruger React)

Camilla noterer i skill-inventar.

---

## 5. Rolle-gap-analyse — 3 nye roller anbefales

### 5.1 Rolle 1 — Dorthe (DPO / GDPR-lead)

| Felt | Værdi |
|---|---|
| **Navn** | Dorthe |
| **Type** | Permanent rolle (GDPR-overhold er ikke projekt-bundet) |
| **Ansvar** | DPO efter GDPR art. 37-39: ROPA, DPIA, breach-respons (72-timer), data-subject-rettigheder, DPA-negotiation, kunde-faqende GDPR-kommunikation |
| **Model** | sonnet (compliance-arbejde kræver dyb ræsonnement, ikke opus-niveau) |
| **Tools** | Read, Write, Edit (i `governance/dpa/` + `Team/Dorthe/`), Glob, Grep, WebSearch, WebFetch, Agent, TodoWrite |
| **Forhold til eksisterende** | Co-design med Mads (lige som Karen). DPO er **uafhængig** af security-architect — det er GDPR art. 38(3): DPO "skal ikke modtage instruktioner om udførelsen". Karen kan ikke være DPO. |
| **SOD-overhold** | Ja — Dorthe reviewer Henriks data-modeller fra GDPR-vinkel; Karen reviewer fra security-vinkel. Ingen rolleblanding. |

**Hvorfor ikke udvide Karen**: GDPR DPO-rolle er lovreguleret uafhængig (selvom DPO ikke er obligatorisk for FGD's størrelse, er det best-practice for regulerede vertikaler). Sammenblanding med security-architect er en SOC 2 + GDPR-konflikt.

### 5.2 Rolle 2 — Trine (Compliance-auditor / ISO 27001 + SOC 2-lead)

| Felt | Værdi |
|---|---|
| **Navn** | Trine |
| **Type** | Permanent rolle (ISO + SOC 2 er løbende cycle, ikke projekt) |
| **Ansvar** | SoA-vedligehold, control-mapping, evidens-indsamling, internal audit cycle, vendor-DPA-tracking (med Dorthe), audit-log-review (månedligt) |
| **Model** | sonnet |
| **Tools** | Read, Write, Edit (i `governance/compliance/`), Glob, Grep, Bash (read-only — `git log`, evidence-extraction), Agent, TodoWrite |
| **Forhold til eksisterende** | Modtager input fra Karen (security-controls), Henrik (dev-controls), Camilla (DB-controls), Bjarne (HSM-controls). Rapporterer til Mads. |
| **SOD-overhold** | Ja — Trine auditerer; Mads godkender; FGD signerer eksternt. Trine merger ikke kode. |

**Hvorfor ikke udvide Mads**: Mads ejer governance-policies + threat-modeling-sparring + sec-team-sammensætning. Auditing kræver dedikeret tid + uafhængig blik på Mads' egne governance-policies. Internal-auditor-uafhængighed er ISO 19011 best-practice.

### 5.3 Rolle 3 — Frederik (Pen-test-koordinator)

| Felt | Værdi |
|---|---|
| **Navn** | Frederik |
| **Type** | Project-rolle (aktiv før hver pen-test-cyklus, dvale imellem). FTE: 0,2 i pre-launch-vinduer. |
| **Ansvar** | Vendor-shortliste-vedligehold, RoE-skrivning, scope-definition, internt pre-pen-test (med `agent-browser` + `security-review` + `e2e-testing`), remediation-tracking |
| **Model** | sonnet |
| **Tools** | Read, Write, Edit (i `governance/pen-test/`), Glob, Grep, Bash, WebSearch, WebFetch, Agent, TodoWrite — **ikke** Edit på prod-kode |
| **Forhold til eksisterende** | Modtager fund fra eksternt vendor + interne værktøjer. Eskalerer til Karen + Mads. Henrik fixer. |
| **SOD-overhold** | Ja — Frederik koordinerer + scanner; Henrik fixer; Karen verificerer. |

**Hvorfor ikke udvide Karen**: Karen designer threat-models + secure-code-review. Pen-test-koordinering har sin egen disciplin (RoE-jura, vendor-management, remediation-tracking) der kræver dedikeret fokus før eksterne vendors koster $30-100k.

### 5.4 Eksisterende roller der dækker resten

- **Karen** dækker fortsat threat-modeling, secure-code-review, FIPS-roadmap, key-ceremony-design, og forbliver lead på FIPS-team.
- **Bjarne** dækker fortsat HSM-eksekvering uændret.
- **Mads** dækker governance + cross-team koordination + alle 5 teams' overordnede direction.

### 5.5 Roller jeg har overvejet og fravalgt

- **Privacy-engineer**: Dækkes af Henrik (data-modeller) + Dorthe (GDPR-controls) + Karen (encryption). Ny rolle vil overlappe.
- **DevSecOps**: Dækkes af Camilla (CI/CD-pipeline) + Mads (deploy-gates governance) + Karen (SAST/DAST review). Ny rolle vil overlappe.
- **Forensics-specialist**: Reaktiv rolle. Etableres først hvis vi har breach. Til den tid: ekstern vendor.

---

## 6. Mandat-review — 10 mandater

### 6.1 Stefan (orkestrator)

- **Følger governance**: Ja
- **SOD**: Ja — Stefan delegerer, udfører ikke produktionsarbejde
- **Justering**: Tilføj eksplicit: "Ved feature-godkendelser med compliance-impact: kald Mads + Trine før FGD-godkendelse." Praktisk: Stefan sikrer at compliance-scorecard er udfyldt før han spørger FGD om GO.

### 6.2 Laila (HR)

- **Følger governance**: Ja
- **SOD**: Ja — co-design med Mads for security-roller (allerede i mandat)
- **Justering**: Udvid co-design-aftale til også at gælde **compliance-roller** (Dorthe, Trine, Frederik) — ikke kun security.

### 6.3 Poul (analytiker)

- **Følger governance**: Ja
- **SOD**: Ja — peer-relation til Mads (allerede i mandat)
- **Justering**: Tilføj: "Når Poul leverer arkitektur-analyser, lægger han kopi i `governance/design-docs/` så Trine/Dorthe kan auditere baggrund." Allerede delvist i konventioner — gør eksplicit.

### 6.4 Camilla (bibliotekar / DB / git / memory)

- **Følger governance**: Delvis
- **SOD**: ⚠ Konflikt — Camilla committer på vegne af alle og kører migrations. Hun har **teknisk** evne til at indføre uautoriserede ændringer.
- **Justering**: 
  - **Hard rule tilføjes**: "Camilla må aldrig commit uden referenceret godkendelse — enten en `Min Inbox/<dato>-<emne>.md` med FGD-signoff, eller en eksplicit Stefan-besked. Hver commit-besked skal pege til godkendelses-kilde."
  - **Hard rule tilføjes**: "Migrations til prod kræver **Henrik (designer) + Camilla (executor) + Mads-godkendelse hvis crypto/RLS/audit-log påvirkes**."
  - **Hard rule tilføjes**: "Memory-opdateringer der vedrører compliance/security skal cc:'es til Mads."

### 6.5 Mads (CISO — mig selv)

- **Følger governance**: Selvfølgelig
- **SOD**: Ja
- **Justering**: Tilføj eksplicit: "Mads ejer ikke audit-evidens-indsamling — det er Trines opgave. Mads godkender SoA, men Trine indsamler evidens og rapporterer." Sikrer Trines uafhængighed.

### 6.6 Sofie (frontend) — UDKAST

- **Følger governance**: Delvis
- **SOD**: ⚠ Mangler eksplicit reviewer-krav
- **Justering**: 
  - Tilføj **Hard rule**: "Hver PR til `apps/web/` kræver mindst én reviewer der ikke er author. For PRs der ændrer auth-flow/CSP/CORS/cookie-håndtering: Karen er obligatorisk reviewer."
  - Tilføj **Hard rule**: "e2e-tests (`e2e-testing` skill) skal eksistere for nye user-flows før merge."
  - Tilføj **Hard rule**: "Speed Insights + Sentry må ikke disables uden Mads-godkendelse (audit-trail-krav, SOC 2 CC7.2)."
  - Tilføj samspil med Dorthe: "GDPR-impacted UI (cookie-banner, consent-flow, data-export-UI) reviewes af Dorthe før merge."

### 6.7 Henrik (backend) — UDKAST

- **Følger governance**: Delvis
- **SOD**: ⚠ Stor konflikt — Henrik designer schema, kører migrations-design, ejer RLS-policies, og kan teknisk merge sin egen kode hvis ingen reviewer er obligatorisk.
- **Justering**: 
  - Tilføj **Hard rule** (kritisk): "Migrations + RLS-policies merges aldrig uden 2 reviewers: Karen (security) + Mads-godkendelse for alt audit/crypto/auth-relateret. Camilla executor først efter Mads-godkendelse."
  - Tilføj **Hard rule**: "OpenAPI-ændringer der ændrer auth-scheme, error-format, eller PII-felter reviewes af Dorthe + Karen."
  - Tilføj **Hard rule**: "Audit-log struktur designes i samarbejde med Trine; immutability + hash-chain + retention er governance-baseret krav (jf. §2.4)."
  - Tilføj **Hard rule**: "Idempotency-cache, rate-limiting, RFC 9457 errors er obligatoriske ved hver write-endpoint — `security-review` skill kører i CI som gate."

### 6.8 Anders (AI-pipeline) — UDKAST

- **Følger governance**: Delvis
- **SOD**: ⚠ Konflikt — Anders ejer både prompt-bibliotek (kode) og audit-log-emit fra Inngest. Han kan teknisk skjule prompt-injection-fund.
- **Justering**: 
  - Tilføj **Hard rule**: "Prompt-bibliotek-ændringer reviewes af Karen (prompt-injection-vinkel) + Trine (audit-evidens for AI-decisions)."
  - Tilføj **Hard rule**: "Anders ejer **emission** af AI-relaterede audit-events; Henrik ejer audit-tabel-skemaet; Trine ejer review af logs. Adskillelse er obligatorisk."
  - Tilføj **Hard rule**: "ZDR-verifikation er Mads-godkendt før første kunde-data — ikke en sidste-minut-tjek."
  - Tilføj **Hard rule**: "Spend-cap-ændringer kræver Mads-godkendelse (governance: capacity-management er SOC 2 A.1.1)."

### 6.9 Karen (security architect) — UDKAST

- **Følger governance**: Ja
- **SOD**: Ja
- **Justering**: 
  - **Reduktion** af scope: GDPR DPO-arbejde fjernes (Dorthe overtager). DPA-tracker delegeres til Dorthe. Karen forbliver lead på FIPS-team og security-architect.
  - Tilføj eksplicit: "Karen rapporterer til Mads (allerede underliggende, men gør eksplicit)."
  - Tilføj: "Karen co-reviewer med Trine på SOC 2 CC6.1-CC6.8 (Logical/Physical Access Controls)."

### 6.10 Bjarne (HSM-operatør) — UDKAST

- **Følger governance**: Ja
- **SOD**: Ja — eksplicit 4-eyes med Karen
- **Justering**: 
  - Tilføj: "Bjarne logger hver HSM-ops til `governance/runbooks/hsm-execution-log.md` (immutable append-only) — Trine reviewer månedligt for SOC 2-evidens."
  - Tilføj: "Quarterly DR-test rapporteres til Trine + Mads for audit-evidens, ikke kun Karen."

---

## 7. Pen-test-vendor-shortliste (DK/EU)

Frederik vedligeholder denne. Mit udgangspunkt 2026-05-05 — verificeres via WebSearch ved hver pen-test-cyklus:

### Top-tier (DK-baseret, regulerede-brancher-erfaring)

1. **Improsec** (København) — Anbefalet for SMV-finansiel/sundhed. CREST-medlem, dansk-talende, kender NIS2/GDPR-kontekst. Cost-niveau: ~80-150k DKK for 5-dages limited-scope.
2. **CSIS Security Group** (København) — Større firma, bredere kompetencer, cyber-incident-response også. Bedre ved kompleks scope. Cost-niveau: ~120-200k DKK.
3. **Dubex Conscia** (Søborg) — Etableret, samarbejder med banker/forsikring. Bredt scope (network + app + cloud).

### Tier-2 (DK / Norden)

4. **Trifork Security** (Aarhus) — Mindre firma, cloud-pen-test specialiseret. Bedre cost for narrow web-app-scope.
5. **Truesec** (Stockholm + København) — Norden-bredt, regulerede-brancher-fokus, CREST.

### EU-bredt (sammenligning + uafhængighed)

6. **NCC Group EMEA** (UK + EU offices) — Stor international, dyr men respekteret af big-4-revisorer. Cost: ~$30-80k USD.
7. **F-Secure Consulting / WithSecure** (FI + EU) — Stærke på app-pen-test, Mac-løsninger, cloud-native.

### Anbefaling for MVP-launch (fase 2)

**Førstegangs-vendor**: Improsec (DK-baseret, dansk kontekst, cost-effektiv, regulerede-brancher-erfaring).

**Andengangs-vendor (12-18 mdr efter første)**: NCC Group eller Truesec — vendor-rotation er SOC 2 best-practice + skifter "blinde vinkler".

**Frederik leverer**: RoE-template + scope-definition i fase 0; vendor-final-selection i fase 1; faktisk pen-test fase 2.

---

## 8. Åbne spørgsmål til FGD

1. **Antal nye roller** [BESVARET 2026-05-05]: Godkender du 3 nye permanente/project-roller (Dorthe DPO, Trine compliance-auditor, Frederik pen-test-koordinator)? Eller foretrækker du sammenlægning (men det rammer SOD og GDPR-uafhængighed)?
2. **Audit-log-retention** [BESVARET 2026-05-05]: Vi accepterer 3 år som baseline (overholder ISO + SOC 2; GDPR-tilladt så længe ROPA dokumenterer hvorfor)?
3. **Pen-test-cyklus** [BESVARET 2026-05-05]: Pre-launch + halvårligt + ved major release? Eller er årligt nok i fase 2?
4. **`skill-comply` som månedlig governance-test** [BESVARET 2026-05-05]: Accepterer du at Trine kører den månedligt mod alle agents og rapporterer compliance-rate til Mads + dig?
5. **DPO-publicering** [BESVARET 2026-05-05]: GDPR art. 37(7) kræver at DPO-kontakt offentliggøres. Skal Dorthe (en AI-rolle) være den publicerede DPO, eller skal du selv være registreret DPO med Dorthe som assist? (Praktisk: dig som DPO + Dorthe som operationel støtte er den juridisk renest model.)
6. **Vendor-rotation pen-test** [BESVARET 2026-05-05]: Accept af "skift vendor mindst hvert 2. år"?
7. **Compliance-team kick-off** [BESVARET 2026-05-05]: Skal alle 5 teams starte parallelt nu, eller staggered (GDPR + FIPS først, ISO + SOC 2 + Pen-test efter MVP-launch)?

---

## 9. FGDs svar på åbne spørgsmål (2026-05-05)

> Modtaget fra FGD via Stefan, 2026-05-05. Tre af de syv åbne spørgsmål i §8 er nu adresseret. De fire resterende spørgsmål er listet til sidst.

### Spørgsmål 1 — Antal nye roller (§8.1)

**Mit oprindelige spørgsmål** (§8.1): "Antal nye roller: Godkender du 3 nye permanente/project-roller (Dorthe DPO, Trine compliance-auditor, Frederik pen-test-koordinator)? Eller foretrækker du sammenlægning (men det rammer SOD og GDPR-uafhængighed)?"

**FGDs svar** (ordret):
> "Godkender 3 nye roller, du laver rolle-beskrivelserne til det og samarbejder med Laila"

**Fortolkning + næste skridt**:

FGD godkender de 3 anbefalede roller fra §5: **Dorthe** (DPO/GDPR-lead), **Trine** (Compliance-auditor for ISO 27001 + SOC 2), **Frederik** (Pen-test-koordinator). FGD giver mig (Mads) mandat til at skrive rolle-beskrivelserne i samarbejde med Laila — dvs. jeg leverer indhold (kompetencer, ansvar, hard rules, hand-off-aftaler, model/tools-anbefaling), Laila leverer format (frontmatter, struktur, dansk-tonen, mandat-konvention).

**Konkret handling**:

1. Jeg udarbejder substans-udkast for hver af de 3 roller i `Team/Mads/2026-05-05-rolle-substans-dorthe.md`, `...-trine.md`, `...-frederik.md` (kompetencer, ansvarsområder, faser, SOD-overhold, hand-off-aftaler, anti-patterns).
2. Jeg sender substans-udkastene videre til Laila via Stefan med opdrag om mandat-formatering iht. konventioner i `Team/_konventioner.md`.
3. Laila producerer endelige `.claude/agents/dorthe-dpo.md`, `trine-compliance-auditor.md`, `frederik-pen-test-koordinator.md`.
4. Camilla committer mandaterne efter FGD signoff.
5. SOD-overhold verificeres pr. rolle (Dorthe uafhængig af Karen jf. GDPR art. 38(3); Trine uafhængig af Mads' egne policies jf. ISO 19011; Frederik adskilt fra Henrik som fixer).

**Bemærkning**: FGD har ikke specificeret model-valg eller aktiverings-tidspunkt — jeg foreslår sonnet for alle tre (jf. §5.1-5.3) og staggered aktivering (Dorthe + Trine fase 0; Frederik når pen-test-cyklus rammer fase 2).

### Spørgsmål 2 — Pen-test-vendor-rotation + 2 pen-test-teams (§8.6 + §8.3)

**Mit oprindelige spørgsmål** (§8.6): "Vendor-rotation pen-test: Accept af 'skift vendor mindst hvert 2. år'?"

**FGDs svar** (ordret):
> "Vi går med 2 års rotation, i udviklingen nu skal vi bare have 2 teams under dig der laver pentest"

**Fortolkning + næste skridt**:

To beslutninger i ét svar:

**Del A — Vendor-rotation**: FGD accepterer **2 års rotation-cyklus** for eksterne pen-test-vendors (jf. §7 anbefalede leverandør-shortliste). Dette er SOC 2 best-practice og minimerer "blinde vinkler". Førstegangs-vendor (Improsec, DK) bruges i fase 2; andengangs-vendor (NCC Group eller Truesec) skiftes ind ved cyklus 2 (24 mdr efter første pen-test).

**Del B — 2 pen-test-teams under mig i udvikling**: FGD ønsker **2 separate pen-test-teams** under min (CISO) ansvar i udviklings-fasen. Min fortolkning: dette er **interne** teams (ikke eksterne vendors) der kører kontinuerlig adversarial-tænkning mod knowlagecentralen før ekstern pen-test. To-team-strukturen muliggør:

- **Rotation-baseret SOD**: Team A dækker en feature-cyklus; Team B dækker næste. Forhindrer "fortroligheds-blindhed" hvor samme team altid ser samme kode.
- **Red team / Blue team-split**: Team A (red) angriber; Team B (blue) defender + verificerer remediation.
- **Cross-validation**: Findings fra Team A skal verificeres af Team B før remediation lukkes.

**Konkret handling**:

1. Jeg udarbejder design for de **2 interne pen-test-teams** i `Team/Mads/2026-05-05-pen-test-teams-design.md`:
   - Team-sammensætning (hvilke roller indgår; sandsynligvis varianter af Frederik + agent-browser + security-review + e2e-testing skill-pakke pr. team)
   - Rotation-policy (per kvartal eller per feature-cyklus)
   - Red/Blue-split eller alternativ struktur (åbent for FGD-input)
   - Eskalation-flow (begge teams rapporterer til Mads)
2. Frederik (når aktiveret) bliver lead for **begge** teams (single-coordinator-model) ELLER vi deler i Frederik-A og Frederik-B som søsterroller. Beslutning ligger hos FGD efter mit design.
3. Vendor-rotation-policy formaliseres i `governance/policies/pen-test-vendor-rotation.md` med 2-års-cyklus, vendor-shortliste-vedligehold, og kriterier for vendor-skift (minimum hvert 2. år; oftere ved kvalitetsbrist).
4. Frederiks rolle-beskrivelse (jf. spørgsmål 1) udvides så han ejer **begge** interne teams' koordination + ekstern vendor-rotation.

**Uklarhed jeg flagger**: FGDs sætning "i udviklingen nu skal vi bare have 2 teams under dig der laver pentest" kan tolkes som (a) udskydelse af ekstern vendor til efter MVP, eller (b) parallel: 2 interne teams kører kontinuerligt + ekstern vendor i fase 2 launch. Jeg går med fortolkning (b) da SOC 2 Type II forudsætter ekstern pen-test pre-launch — men beder Stefan bekræfte med FGD.

### Spørgsmål 3 — Audit-log-retention 3 år + Dorthe dokumenterer i GDPR-politik (§8.2)

**Mit oprindelige spørgsmål** (§8.2): "Audit-log-retention: Vi accepterer 3 år som baseline (overholder ISO + SOC 2; GDPR-tilladt så længe ROPA dokumenterer hvorfor)?"

**FGDs svar** (ordret):
> "Ja 3 år er det vi går med, og sikre dig Dorthe dokumenterer det i vores GDPR-politik samt hvorfor vi har 3 år."

**Fortolkning + næste skridt**:

FGD bekræfter **3 års audit-log-retention** som baseline (jf. §2.4). Dette er længste blandt de tre rammeværker (ISO 27001 anbefaler 3 år; SOC 2 typisk 1 år; GDPR kræver minimum-retention med dokumenteret formål). Dorthe får eksplicit ansvar for at dokumentere både **valget** og **begrundelsen** i FGDs GDPR-politik (sandsynligvis lokeret i `governance/dpa/gdpr-politik.md` når Dorthe aktiveres).

**Begrundelse for 3 år (skal dokumenteres af Dorthe)**:

1. **ISO 27001:2022 A.8.15 (Logging)**: Anbefaler retention der understøtter incident-investigation; 3 år dækker typisk forældelses-frister + audit-cycle-overhead.
2. **SOC 2 Type II CC7.2 + CC7.3**: Audit-trail som monitoring-evidens; 1 år som minimum, men 3 år understøtter Type II observation period (12+ mdr) + revisorens look-back.
3. **GDPR art. 5(1)(e) "storage limitation"**: Personoplysninger må ikke opbevares længere end nødvendigt til formål. 3 år begrundes med:
   - Forensics-behov ved breach (gennemsnitlig breach-detection: 200+ dage iht. IBM Cost of Data Breach 2024)
   - Civil-retlige forældelses-frister (DK: typisk 3 år for forbrugerkrav)
   - Audit-cyclus-krav (SOC 2 Type II look-back; ISO 27001 surveillance-audit hvert 12. md, recertificering hvert 36. md)
4. **Rammekonflikt-håndtering**: Når GDPR ønsker minimum og ISO/SOC 2 ønsker længere, dokumenterer ROPA art. 30 hvorfor længere periode er nødvendig — dette er GDPR-compliant.

**Konkret handling**:

1. **Henrik + Trine**: Designer audit-log-tabel (`audit.events`) med 3-års-retention indbygget (auto-purge efter 3 år + 1 dag fra `event_at`). Krav fra §2.4: append-only, hash-chain, ekstern arkivering (S3 Glacier WORM) månedligt.
2. **Camilla**: Migration til prod kræver Mads + Henrik + Karen-godkendelse (jf. §6.7 hard rule om migrations + crypto/RLS/audit-log).
3. **Dorthe** (når aktiveret): Dokumenterer i `governance/dpa/gdpr-politik.md`:
   - Retention-periode: 3 år
   - Begrundelse (de 4 punkter ovenfor)
   - Auto-purge-mekanisme + verifikation
   - Eksterne arkiverings-detaljer
   - Data-subject-rettigheder ift. audit-trail (art. 17 right-to-erasure undtagelse: legal obligation + legitimate interest)
4. **Trine** (når aktiveret): Audit-log-retention-policy kobles til SoA (ISO 27001 A.8.15) + Trust Service Criteria mapping (SOC 2 CC7.2).
5. **ROPA**: Dorthe tilføjer audit-log som processing-aktivitet med 3-års retention og dokumenteret formål (security-incident-investigation, compliance-audit-evidens, kunde-tillid).

### Spørgsmål 4 — Pen-test-cyklus halvårligt (§8.3)

**Mit oprindelige spørgsmål** (§8.3): "Pen-test-cyklus: Pre-launch + halvårligt + ved major release? Eller er årligt nok i fase 2?"

**FGDs svar** (ordret):
> "Halvårligt — mere hyppig regression-fangst end årligt"

**Fortolkning + næste skridt**:

FGD vælger **halvårlig pen-test-cyklus** (semi-annual) som standard, frem for årlig. Dette er strikser end markedsstandard for SMV (årlig er normen) og afspejler regulerede-vertikal-kontekst hvor regression-fangst-frekvens er kritisk. Cyklus-design:

- **2 cyklusser pr. år pr. team**: Hvert af de 2 interne pen-test-teams kører 2 fuldstændige pen-test-cyklusser pr. år.
- **Total 4 pen-tests om året**: 2 teams × 2 cyklusser = 4 distinkte pen-test-runs årligt, fordelt på året (cirka kvartalsvis hvis vi forskyder teams 90 dage).
- **Forslag til cadence** (Mads' design — afventer FGD-godkendelse efter Frederik aktiveres):
  - Q1: Team A — fuld scope (web + API + RLS + HSM-bridge)
  - Q2: Team B — fuld scope (uafhængig blik; cross-validate Team A's findings)
  - Q3: Team A — delta-scope (kun ændringer siden Q1 + regression på fund)
  - Q4: Team B — delta-scope + pre-launch-spike hvis major release
- **Major release-spike**: Ekstra ad-hoc pen-test ved major releases (uafhængig af kvartals-cyklus).

**Konsekvens for Frederik (når aktiveret)**:

- Frederik koordinerer 4 distinkte pen-test-cyklusser pr. år (mod oprindelig forventning på 2).
- Ressource-forbrug: ~2 ugers fokus pr. cyklus × 4 = 8 uger pr. år for Frederik (FTE 0,15-0,2 årligt).
- Reporting-cadence: Frederik leverer halvårlig konsolideret rapport (Q2 + Q4) til Mads + Trine + FGD med findings-trend-analyse.

**Konkret handling**:

1. Jeg opdaterer `Team/Mads/2026-05-05-pen-test-teams-design.md` (jf. svar #2) med halvårlig-cadence + Q-fordeling + delta vs. fuld-scope-strategi.
2. Frederiks rolle-substans-udkast (jf. svar #1) udvides så halvårlig-cyklus + 4 årlige runs + cross-team-validation er eksplicit i hans ansvarsområder.
3. Trines rolle-substans-udkast (jf. svar #1) tilføjer "modtager halvårlig pen-test-evidens fra Frederik som SOC 2 CC4.1 + ISO 27001 A.8.29-evidens".

### Spørgsmål 5 — `skill-comply` månedligt (§8.4)

**Mit oprindelige spørgsmål** (§8.4): "Accepterer du at Trine kører den månedligt mod alle agents og rapporterer compliance-rate til Mads + dig?"

**FGDs svar** (ordret):
> "Ja, månedligt — Trine kører"

**Fortolkning + næste skridt**:

FGD godkender **månedlig `skill-comply`-eksekvering** mod alle 10+ agent-mandater (Stefan, Laila, Poul, Camilla, Mads, Sofie, Henrik, Anders, Karen, Bjarne + Dorthe, Trine, Frederik når de aktiveres = 13 agents potentielt). Trine ejer kørslen, rapporteringen, og remediation-tracking. Dette er en **SOC 2 CC1.4-game-changer** — vi har ikke kun policies skrevet ned, vi måler kontinuerligt om agents faktisk følger dem.

**Operationel design**:

- **Frekvens**: Månedligt (12 cyklusser/år).
- **Scope pr. cyklus**: Alle aktive agents × 3 strictness-levels (loose/normal/strict prompts) = 13 agents × 3 levels = 39 scenarie-runs pr. måned.
- **Token-budget (estimat)**: Hver scenarie-run ≈ 5-15k input tokens + 1-3k output tokens. 39 runs × 15k ≈ 585k tokens/måned i input + ~80k output. Med Sonnet pricing (3 USD/Mtok input, 15 USD/Mtok output): ~3 USD/md = ~36 USD/år. Trivielt for governance-værdi.
- **Rapporterings-format**: Trine producerer månedlig rapport `governance/compliance/skill-comply-<YYYY-MM>.md` med:
  - Compliance-rate pr. agent (% af scenarier hvor agent fulgte alle hard rules)
  - Trend-graf måned-over-måned
  - Findings-prioritering (rolle-mandat-svaghed vs. agent-tolknings-fejl vs. skill-konflikt)
  - Anbefalet remediation (ofte: mandat-justering via Laila + Mads co-design)
- **Eskalation**: Compliance-rate < 85% pr. agent = automatisk Mads-review + Stefan-eskalation.

**Konkret handling**:

1. **Trine** (når aktiveret): Får eksplicit ansvarsområde "månedlig `skill-comply`-eksekvering" i sit rolle-substans-udkast (jf. svar #1).
2. **Mads**: Etablerer baseline-måling i fase 0 (først kørsel uge 1-2 efter Trine aktiveres). Sætter compliance-rate-thresholds (foreløbig 85% green / 70-85% yellow / <70% red).
3. **Anthropic-token-budget**: Approxime 50 USD/år allokeres til governance-pipeline (skill-comply + dependency-cycle-tjek). Optages i Camillas tracking når relevant.
4. **Reporting-flow**: Trine → Mads (review + sign-off) → Stefan → FGD (månedlig). Stefan inkluderer rapporten i månedligt status-resumé til FGD.
5. **Skill-comply selv**: Skal verificeres op mod aktuel version via context7 ved første kørsel (jf. version-verifikations-governance i memory).

### Spørgsmål 6 — DPO-publicering: FGD juridisk + Dorthe operationel (§8.5)

**Mit oprindelige spørgsmål** (§8.5): "Skal Dorthe (en AI-rolle) være den publicerede DPO, eller skal du selv være registreret DPO med Dorthe som assist?"

**FGDs svar** (ordret):
> "FGD juridisk DPO + Dorthe operationel støtte — FGD registreret hos Datatilsynet; Dorthe leverer dagligt arbejde"

**Fortolkning + næste skridt**:

FGD vælger den juridisk reneste model (som anbefalet): **FGD personligt** registreres som juridisk DPO hos **Datatilsynet** (DK's tilsynsmyndighed); **Dorthe** leverer det operationelle dag-til-dag-arbejde under FGDs tilsyn. Dette løser flere udfordringer:

- **GDPR art. 37(7) compliance**: Datatilsynet og data-subjects har en navngiven juridisk person at kontakte (FGD), ikke en AI-rolle (Dorthe).
- **GDPR art. 38(3) DPO-uafhængighed**: FGD som DPO modtager ikke instruktioner om DPO-udførelse fra "data controller" (sig selv som virksomhedsejer) — fungerer juridisk fordi FGD er **selvstændig konsulent** med klar rolle-adskillelse i sin egen virksomhed.
- **GDPR art. 38(6) konflikter med andre roller**: FGD som DPO + virksomhedsejer er en grænse-case, men acceptabel for SMV-konsulent uden direkte conflict-of-interest med andre roller.
- **Operationel skalering**: Dorthe leverer ROPA-vedligehold, DPIA-skrivning, breach-respons-koordination, kunde-DPA-svar — det daglige DPO-arbejde der ellers kunne overstige FGDs kapacitet.

**Konkret handling**:

1. **Dorthe** (når aktiveret) får task som første-leverance: forberede registrering hos Datatilsynet for FGD som DPO. Konkret indhold:
   - Udfylde "Anmeldelse af databeskyttelsesrådgiver" via [datatilsynet.dk](https://www.datatilsynet.dk) (online-formular).
   - FGDs kontakt-oplysninger som DPO (navngivet, e-mail, telefon).
   - Beskrivelse af DPO-funktion + ressourcer (Dorthe som AI-assist + budget + uafhængighed).
   - FGD signerer + indsender; Dorthe assisterer med dokumentation.
2. **Dorthe** producerer intern dokumentation `governance/dpa/dpo-rollefordeling.md` der beskriver:
   - FGDs juridiske DPO-rolle (registreret myndighed-kontakt, ansvarlig for GDPR art. 37-39).
   - Dorthes operationelle DPO-støtte-rolle (omfang, leverancer, eskalations-flow).
   - Konflikt-håndtering: hvis FGD som virksomhedsejer træffer beslutning der konflikter med GDPR-overhold, eskalerer Dorthe til Stefan + advokat-konsultation.
   - Dokumentation over uafhængighed: ressourcer, kommunikations-flow, beskyttelse mod afskedigelse (n/a for FGD som ejer).
3. **Camilla** opdaterer FGDs DPA-tracker med "DPO-kontakt: <FGD@fgdconsulting.se>" når registrering er gennemført.
4. **Kunde-DPA-template** (fase 2-leverance fra Dorthe): Inkluderer DPO-kontakt-info som <FGD@fgdconsulting.se> med Dorthe som CC for operationelle henvendelser.
5. **Webside-publicering**: GDPR art. 37(7) kræver offentlig publicering. Dorthe leverer udkast til `knowlagecentral.dk/privacy` eller tilsvarende med DPO-kontakt-blok.

**Vigtigt for SOD**: Dette skaber rolle-blanding (FGD som ejer + DPO), men er accepteret praksis for konsulent-SMV. Trine bør i fase 1 audit-evaluere om denne kombi forbliver acceptabel når kundebase vokser; ved >50 kunder bør ekstern DPO overvejes.

### Spørgsmål 7 — Parallel kick-off af alle 4 compliance-teams (§8.7)

**Mit oprindelige spørgsmål** (§8.7): "Skal alle 5 teams starte parallelt nu, eller staggered (GDPR + FIPS først, ISO + SOC 2 + Pen-test efter MVP-launch)?"

**FGDs svar** (ordret):
> "Alle 4 parallel start — GDPR + ISO 27001 + SOC 2 + FIPS 140-3 samtidigt (selvom kun interne pen-test og ikke formel SOC 2-cert i Fase 2)"

**Fortolkning + næste skridt**:

FGD vælger **parallel kick-off af alle 4 compliance-spor** (GDPR, ISO 27001, SOC 2, FIPS 140-3). Pen-test-team (oprindeligt mit team #5) er ikke et compliance-spor i sig selv, men en aktivitet der understøtter SOC 2 + ISO 27001 — Frederik aktiveres parallelt da han ejer pen-test-koordination tværs over alle 4 spor. **Vigtig nuance**: SOC 2 kører som "design-aware" spor, ikke formel certificering i fase 2 (jf. ny SOC 2-roadmap-justering nedenfor).

**Resource-plan**:

- **Kick-off-dato**: Uge 1 (denne uge) — afventer kun at Laila producerer de 3 nye mandater (Dorthe, Trine, Frederik) baseret på mit substans-udkast.
- **Forudsætninger** (skal være på plads før kick-off):
  - Alle 3 nye rolle-mandater designet (Mads + Laila co-design, FGD signoff)
  - Camilla committer mandater i `.claude/agents/`
  - Governance-skabeloner på plads: SoD-policy, code-review-flow, deploy-gates, feature-scorecard-template (Mads leverer uge 1)
- **Parallel team-aktivering** (uge 2):

| Team | Lead | Første-leverance (uge 2-3) | Kritisk afhængighed |
| --- | --- | --- | --- |
| GDPR | Dorthe | ROPA v1 + DPA-tracker overtagelse fra Karen + Datatilsynet-registrering for FGD | DPO-rolle godkendt |
| ISO 27001 | Trine | Statement of Applicability v1 (93 Annex A controls scoped) | SoA-template + Mads' input |
| SOC 2 (design-aware) | Trine | Trust Service Criteria mapping v1 (Security + Confidentiality + Availability) | Trines tid; **ikke** ekstern auditor i fase 2 |
| FIPS 140-3 | Karen | FIPS 140-3 roadmap v1 + Yubico cert-status-tracking | Karens reduceret scope (Dorthe overtager DPA) |
| Pen-test (cross-cutting) | Frederik | RoE-template + scope-definition + 2-team-design | Frederik aktiveret + 2-team-design fra Mads |

- **Resource-konflikt-risiko**: Trine leder både ISO 27001 + SOC 2-spor. Ved volumen-overskridelse aktiveres "ISO først, SOC 2 efter ISO v1" som fallback. Mads monitorerer ugentligt.
- **Karen-overbelastning**: Allerede mitigeret ved at Dorthe overtager DPA-arbejdet. Karen forbliver lead på FIPS + security-architect generelt.
- **Mads' rolle**: Cross-team koordination + ugentlig stand-up med alle 4 leads (Dorthe, Trine, Karen, Frederik) i fase 0 (uge 1-3) — derefter månedlig.

**Konkret handling**:

1. **Mads** (denne uge): Substans-udkast for Dorthe, Trine, Frederik (jf. svar #1) — leverer i `Team/Mads/`.
2. **Laila** (uge 1-2): Mandat-formatering iht. konventioner.
3. **FGD** (uge 2): Signoff på alle 3 mandater i én runde.
4. **Camilla** (uge 2): Committer mandater + Mads' governance-skabeloner.
5. **Mads** (uge 1-2): Leverer governance-skabeloner: `governance/policies/sod-policy.md`, `code-review-flow.md`, `deploy-gates.md`, `governance/compliance/feature-scorecard-template.md`, `governance/policies/pen-test-vendor-rotation.md`.
6. **Mads + Camilla** (uge 1-2): Audit-log-design-spec (immutable + hash-chain + 3-års retention) til Henrik.
7. **Uge 2-3**: Alle 4 teams kick-off parallel (Dorthe, Trine, Karen, Frederik leverer første-leverancer fra tabel ovenfor).
8. **Uge 4+**: Mads' månedlig cross-team status-rapport til Stefan + FGD.

### SOC 2-roadmap-justering (afledt af FGDs beslutning om kun 2 interne pen-test-teams)

FGDs beslutning under svar #2 om **kun 2 interne pen-test-teams** uden ekstern vendor i fase 1+2 har en direkte konsekvens for SOC 2 Type II-vejen som skal eksplicit dokumenteres:

**Ændringen**:

- Original anbefaling (§3 Team 5 + §7): Ekstern pen-test-vendor (Improsec) som pre-launch-blocker for SOC 2 Type II i fase 2.
- Ny beslutning: 2 interne pen-test-teams under Mads/Frederik. **Ingen ekstern vendor** i fase 1+2.
- Konsekvens: **SOC 2 Type II formel certificering ikke realistisk i fase 2**. SOC 2 Type II audit kræver ekstern uafhængig pen-test som del af CC4.1 (Monitoring Activities) + CC7.1 (System Operations) evidens-kæde. Auditor (Big-4 eller specialist som A-LIGN, Coalfire, Schellman) accepterer ikke kun-internt pen-test.

**Hvad vi opretholder**:

1. **"SOC 2-aware design"**: Vi designer arkitekturen + governance-rammer + audit-trail + control-implementeringer som om vi går efter SOC 2 Type II. Dette betyder:
   - Trust Service Criteria mapping fortsætter (Trine, fase 0)
   - Evidence-pipeline bygges (audit-log → S3 Glacier; CI-attest; access-review-cadence)
   - Control-implementering pr. CC1-CC9 + relevante A/C-criteria
   - Internal audit cycle 1 i fase 2 (Trine)
2. **`skill-comply`-evidens**: Månedlig kørsel (jf. svar #5) skaber CC1.4-evidens (policies-followed) der ER unik for FGD og styrker SOC 2-narrativet.
3. **Halvårlig pen-test (intern)**: 4 årlige interne pen-tests (jf. svar #4) skaber CC4.1-evidens, men ikke som ekstern uafhængig validering.

**Hvad vi udskyder**:

1. **Formel SOC 2 Type II auditor-engagement**: Udskudt til efter fase 2.
2. **Ekstern pen-test (Improsec/NCC Group/Truesec)**: Udskudt indtil enten:
   - **Trigger A**: Kunde-kontrakt eksplicit kræver SOC 2 Type II-rapport (typisk enterprise-kunder eller regulerede vertikaler hvor kunden selv skal SOC 2-attesere deres sub-processors)
   - **Trigger B**: FGD beslutter at SOC 2 Type II er strategisk salgs-værktøj (typisk når 3-5+ enterprise-kunder i pipeline)
   - **Trigger C**: Investor / akquirer kræver SOC 2 Type II som M&A-due-diligence
3. **SOC 2 Type II observation period**: Kan ikke starte før ekstern pen-test gennemført. Typisk 12+ måneder fra trigger til attest.

**Aktivering ved trigger**:

Når én af triggers (A/B/C) udløses, har FGD en hurtig vej til SOC 2 Type II:

- Vi har allerede design + controls + audit-trail + evidence-pipeline
- Vi mangler "kun" ekstern pen-test (~80-150k DKK) + auditor-engagement (~150-300k DKK afhængigt af scope) + 12 mdr observation
- Total time-to-attest fra trigger: ~14-15 mdr (3 mdr forberedelse + 12 mdr observation)
- Total cost-to-attest: ~250-450k DKK

**Dokumentation**:

1. Trine producerer i fase 0: `governance/compliance/soc2-roadmap.md` med:
   - "SOC 2-aware design" scope (hvad vi gør nu)
   - Trigger-definitions (A/B/C)
   - Aktiverings-playbook (steps når trigger udløses)
   - Dependencies på Frederik (ekstern pen-test koordination) og budget-estimater
2. **FGDs salgs-position**: Indtil formel cert: kommunikér som "SOC 2-aligned architecture, formal audit available on customer request". Dette er ærligt og signalerer modenhed.

**Hvad ændrer sig IKKE**:

- ISO 27001:2022 fortsætter som planlagt (formel cert mulig uden ekstern pen-test, dog er pen-test stærk evidens for A.8.29)
- GDPR fortsætter som planlagt (DPO + ROPA + DPIA er kerne)
- FIPS 140-3 fortsætter som planlagt (Yubico cert Q2 2026)
- Halvårlig intern pen-test fortsætter (kvalitets-værdi uafhængigt af SOC 2)

---

---

## 10. Foreslået rækkefølge for næste skridt

### Uge 1 (denne uge)

1. FGD svarer på de 7 åbne spørgsmål (§8) — dette låser scope.
2. **Stefan** beder Laila + Mads co-designe Dorthe, Trine, Frederik mandater (3 udkast i `Min Inbox/`).
3. **Mads** opretter governance-skabeloner: `governance/policies/sod-policy.md`, `governance/policies/code-review-flow.md`, `governance/policies/deploy-gates.md`, `governance/compliance/feature-scorecard-template.md`.
4. **Mads + Camilla** skitserer audit-log-design (immutable + hash-chain + retention) som spec til Henrik.

### Uge 2

5. FGD godkender de **8 mandater** (5 oprindelige + 3 nye + 0 — Mads er allerede aktiv) i én runde.
6. **Camilla** committer alle mandater + governance-skabeloner.
7. **Trine** (når aktiveret) påbegynder SoA + Trust Service Criteria mapping.
8. **Dorthe** (når aktiveret) påbegynder ROPA + DPA-tracker overtagelse fra Karen.

### Uge 3+

9. Compliance-team kick-off (parallelt eller staggered iht. §8 spørgsmål 7).
10. **Frederik** (når aktiveret) skriver RoE-template + scope-definition.
11. **Karen** afgiver GDPR DPA-tracker til Dorthe; fortsætter på FIPS-team.
12. **Mads** facilitere første kvartalsvise teknologi-vurderings-runde (jf. §2.8).

### Hardware-spor (parallelt)

- HSM-hardware ankommer ~2026-05-15 (allerede bestilt).
- Bjarne unboxer + verificerer.
- Karen + Bjarne eksekver key-ceremony fase 1.5.
- FIPS-team aktiveres når cert-status hos Yubico bliver sigtet til Q2 2026 milestone.

---

**Underskrevet**: Mads (CISO)
**Dato**: 2026-05-05
**Næste status-rapport**: Efter FGD-svar på §8-spørgsmål.
