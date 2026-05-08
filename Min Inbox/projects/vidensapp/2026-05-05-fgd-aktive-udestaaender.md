---
dato-oprettet: 2026-05-05
sidst-opdateret: 2026-05-05 (sen aften — Mads CISO aktiveret)
status: aktiv
type: levende-dokument
ejer: Stefan (vedligeholder)
beskrivelse: Levende oversigt over FGD's aktive udeståender. Stefan opdaterer ved hver session-skift og når status ændres. Punkter krydses af eller fjernes når de er løst.
---

# FGD's aktive udeståender

> **Dette er et levende dokument**. Stefan opdaterer det løbende. Krydsede ting `[x]` venter ikke længere; ting med `[ ]` venter på dig.

## Faste roller pr. 2026-05-05

5 permanente roller: **Stefan** (orkestring), **Laila** (HR), **Poul** (analyse, peer m. Mads), **Camilla** (DB/git/memory), **Mads** (CISO, peer m. Poul).

## A. Mandat-godkendelser

### Godkendt + aktiveret

- [x] **Mads — CISO** (permanent rolle, sonnet) — aktiveret 2026-05-05

### Project-mandater — afventer din godkendelse (opdaterede + nye)

5 project-mandater (opdateret med Mads' SOD-justeringer + skills):
- [ ] [Sofie — frontend](Min Inbox/projects/vidensapp/2026-05-05-sofie-frontend-mandat.md) (model: sonnet)
- [ ] [Henrik — backend / Neon](Min Inbox/projects/vidensapp/2026-05-05-henrik-backend-mandat.md) (model: sonnet)
- [ ] [Anders — AI-pipeline](Min Inbox/projects/vidensapp/2026-05-05-anders-ai-pipeline-mandat.md) (model: sonnet)
- [ ] [Karen — security architect (scope reduceret, GDPR flyttet til Dorthe)](Min Inbox/projects/vidensapp/2026-05-05-karen-security.md) (model: sonnet, deltids-rådgiver, rapporterer til Mads)
- [ ] [Bjarne — HSM-operatør](Min Inbox/projects/vidensapp/2026-05-05-bjarne-hsm-operator.md) (model: haiku, rapporterer til Mads)

3 nye rolle-udkast (efter Mads' first-session anbefaling):
- [ ] [Dorthe — DPO permanent](Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md) (rapporterer til FGD direkte ift. GDPR art. 38(3))
- [ ] [Trine — compliance-auditor permanent](Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md) (ISO 27001 + SOC 2)
- [ ] [Frederik — pen-test-koordinator project](Min Inbox/projects/vidensapp/2026-05-05-frederik-pentest.md)

Reference-rapporter (læses sammen med mandaterne):

- [Lailas rapport runde 1 (Sofie/Henrik/Anders)](Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-3-roller.md)
- [Lailas rapport runde 2 (Karen + Bjarne)](Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-karen-bjarne.md)
- [Lailas rapport Mads CISO](Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-mads-ciso.md)

## B. Konto-oprettelser (du gør selv)

Disse blocker Camillas Fase 0 setup-arbejde.

- [ ] **Vercel Pro** aktiveret på personal account (hvis ikke allerede). Husk DPA i dashboard.
- [ ] **Cloudflare-konto** for `fgdconsulting.se` (planlagt flytning). CNAME for `docs` → `cname.vercel-dns.com`, proxy "DNS only".
- [ ] **Tailscale-konto** + tilføj Vercel-egress + HSM-host-server til samme tailnet.
- [ ] **Anthropic 3 workspaces** (`vidensapp-prod`, `vidensapp-test`, `vidensapp-dev`) med spend-caps $50/$15/$5. **ZDR aktiveret på prod-workspace** før første kunde-data.
- [ ] **Clerk Free-konto** + 2 instances (dev + prod). JWT-template "neon" konfigureres af Henrik.
- [ ] **Sentry-projekt** (free tier OK). Camilla opretter hvis du delegerer.
- [ ] **Inngest cloud-konto** + 3 environments. Camilla opretter hvis du delegerer.

## C. Hardware (er bestilt — afventer levering)

- [x] 2× YubiHSM 2 FIPS bestilt 2026-05-05 (Layer 2: online + offline)
- [ ] **Modtag hardware** (5-10 dages leveringstid). Bjarne unboxer + verificerer serial mod Yubico-kvittering.
- [ ] **HSM-host server** (Linux + UPS) klar i FGD-kontor inden hardware ankommer.
- [ ] **Fysisk lokation** for offline HSM besluttet (bankboks vs. låst skab vs. fire-proof safe).

## D. DPA-signering (compliance-blokker for kunde-onboarding)

Du skal signere databehandleraftaler med alle leverandører før første kunde-data:

- [ ] Neon (via console eller sales)
- [ ] Vercel (DPA i dashboard, kræver Pro)
- [ ] Clerk (på signup)
- [ ] Anthropic (DPA + ZDR-anmodning på prod-workspace)
- [ ] Inngest
- [ ] Voyage AI (request via support)
- [ ] Yubico (efter hardware-levering)
- [ ] Tailscale (efter konto-oprettelse)

Camilla tracker i `sys.dataprocessing_agreements`-tabel når DB er oppe.

## E. Designvalg du skal beslutte (ikke akut, men inden Fase 2)

Pouls åbne v3-spørgsmål:

- [ ] **Subteam-feature**: aktivér `app.organizations` + magic-link-invite UI allerede i Fase 1, eller venter til Fase 2 som planlagt? (Datamodel er klar i begge tilfælde.)
- [ ] **Editor**: markdown-only i MVP, eller TipTap-baseret rich-text fra Fase 2? Påvirker Sofies komponent-stack.
- [ ] **Mobile push-notifications**: ønskes der senere (Fase 3) notifikation når AI-berigelse er færdig på mobil?
- [ ] **Customer-facing branding**: skal kunder kunne se custom logo pr. organisation, eller ren FGD-branding i v1?

## F. Sikkerhed / FIPS strategiske spørgsmål

- [x] **Hardware bestilt**: 2× YubiHSM 2 FIPS i Layer 2 (online + offline). Cert-risiko (FIPS 140-2 v2.2 udløb 2. maj 2026) accepteret. Vi udvikler mod FIPS 140-3 cert-anvendelse i produktion (forventet Q2 2026).
- [ ] **Clerk Enterprise BYOK**: Skal vi i fase 3+ flytte JWT-signing til vores HSM (kræver Clerk Enterprise — se sales)? Eller forblive software-side hos Clerk? — **Mads vurderer**.
- [ ] **Formel system-audit**: Hvis første regulereret kunde kræver formel SOC 2 / ISO 27001 / FIPS-attest, igangsættes det først når kontrakt dækker omkostning ($30-100k engangs, 6-12 mdr proces) — **Mads ejer roadmap**.

## G. Mads' first-session leveret (2026-05-05)

Mads' rapport ligger i `governance/compliance/2026-05-05-mads-first-session.md`.

Hovedfund:
- 5 SOD-issues identificeret (deploy-gate, Camilla commit-flow, Sofie/Henrik/Anders reviewer-rolle)
- 3 nye roller anbefalet (Dorthe/Trine/Frederik) — godkendt af FGD
- 4 nye skills anbefalet installeret (agent-browser, e2e-testing, security-review, skill-comply)
- ~55 ikke-vurderede skills — kræver senere CISO-led revurdering
- 5 compliance-teams starter parallelt (godkendt af FGD)
- DPO-model: FGD juridisk + Dorthe operationel (godkendt af FGD)

- [ ] **Laila implementerer Mads' justeringer** (i gang) — 9 mandat-opdateringer + 3 nye rolle-udkast
- [ ] **Camilla verificerer 4 nye skills** (i gang) — installation-status
- [ ] **Stefan-CLAUDE.md justeringer** (afventer Lailas flag) — Mads har anbefalinger Camilla implementerer
- [ ] **Pen-test-vendor**: Improsec (DK) som førstegangs — kontakt for sales-pitch når Frederik er aktiv
- [ ] **CISO-led revurdering** af v3 + alle skills (Mads + Poul peer-runde) — afventer 5 nye/opdaterede mandater er godkendt

## H. Lavt prioriteret (kan vente)

- [ ] **Excalidraw-fil opdatering**: `Team/Poul/2026-05-05-vidensapp-arkitektur.excalidraw` mangler de nye komponenter (Tailscale, HSM online/offline, Mads-rolle). Mermaid-PDF + drawio er master pr. 2026-05-05.

## Næste skridt

Højeste prioritet: **Godkend de 5 project-mandater (sektion A)** så Camilla kan committe alt samlet, og Mads' første session kan starte. Derefter kan B/C/D køre parallelt med Mads' compliance-team-design (sektion G).

Sig til når noget skal opdateres her — Stefan vedligeholder filen.
