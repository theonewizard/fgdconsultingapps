---
dato: 2026-05-05
forfatter: Poul
emne: Vurdering af 6 udvidelser (shadcn skills, Refero, GetDesign.md, Vercel MCP, Next.js MCP, Vercel skills)
status: leveret
relateret-analyse: arkiv/Team/Poul/2026-05-05-knowlagecentral-analyse-v2.md
---

# Evaluering: 6 potentielle udvidelses-ressourcer

## Sammenfatning

Denne rapport evaluerer 6 kandidat-ressourcer til knowlagecentral-projektet og anbefaler hvilke der skal aktiveres i MVP vs. senere faser. Formål: maksimale værdi per setup-time + minimum vendor-afhængighed.

---

## Ressource 1: shadcn/ui skills (design-tokens + komponent-bibliotek)

### Hvad det er

shadcn/ui er et kurateret bibliotek af React-komponenter bygget på Radix UI + Tailwind CSS. "skills" betyder: design-tokens (farver, typografi, spacing), komponent-katalog, best-practices for component-composition.

### Status/version

- shadcn/ui: v0.30+ (modul-baseret installation via npm/script).
- Tailwind CSS: v4 (i use).
- Radix Primitives: v1.1+.

### Kategori

**Design-system / UI infrastructure** — kritisk for konsistent UI og udviklingstempo.

### Pas-i-arkitektur

- Direkte i Next.js 16 App Router via React Server Components.
- Tailwind v4 integreres allerede.
- Bygger på samme auth-flow som Clerk (form-validation, loading-states).

### Setup-trin

1. `npx shadcn-ui@latest init` (vælger Tailwind-konfiguration).
2. `npx shadcn-ui@latest add <component>` for hvert komponent (button, dialog, textarea, etc.).
3. Designér custom tokens i `globals.css` for FGD-branding.
4. Build local component-storybook (evt. med Storybook senere).

**Estimat**: 2–3 timer til 80% coverage.

### Sikkerhed

- shadcn-komponenter er open-source + review-friend.
- Ingen auth-secrets eller ekstern dependency.
- XSS-risiko: komponent-input skal Zod-valideres (ditto for alle inputs).

### Anbefaling

**MVP — JA. Højeste prioritet.** shadcn/ui reducerer custom CSS og holder UI konsistent. Setup er hurtig og ROI er høj.

---

## Ressource 2: Refero (integreret dokumentations-søgning)

### Hvad det er

Refero er en hybrid søge-engine der lader end-users søge i deres egne docs + ekstern knowledge-base samlet. Tilbyder vector-embedding + FTS.

### Status/version

- Status: **Beta** (limited production use).
- Pricing: ukendt (lukket beta).
- Integration: API-baseret via webhook / REST.

### Kategori

**Søge-infrastruktur / Nice-to-have** — kunne erstatte eller supplere pgvector i senere fase.

### Pas-i-arkitektur

- Ville parallel pgvector i MVP — dobbeltarbejde at migrere senere.
- Integreres som separat søge-endpoint i `/api/search/refero`.
- Afhængighed på Referos uptime + support.

### Setup-trin

1. Ansøg Refero-beta access (uvist hvor lang kø er).
2. Auth-integration (API-nøgle).
3. Sync notebook-data til Refero (batch eller webhook-trigget).
4. Implementér fallback til pgvector hvis Refero svigter.

**Estimat**: 5–10 timer setup + ukendt ventetid for beta-access.

### Sikkerhed

- Beta-software → ændringer i API / pricing uden varsel.
- Vilkår ukendte (data-behandling, SLA).
- GDPR-compliance uafklaret.

### Anbefaling

**MVP — NEJ. Fase 2–3 kandidat.** Refero lover meget (hybrid search out-of-box) men risikoen (beta, ukendt pricing, unclear vilkår) vejer tungere end setup-gevinsten når pgvector allerede virker. **Vit med FGD**: hvis Refero-access åbner og betingelser er klare, revisit fase 2.

---

## Ressource 3: GetDesign.md (design-tokens som MD-fil)

### Hvad det er

GetDesign.md er et pattern/værktøj til at gemme designsystem-definitioner (farver, typografi, spacing) i struktureret Markdown + JSON frontmatter. Gør design portable og versionerbar i git.

### Status/version

- Open-source projekt (Figma export + GitHub integration mulig).
- Ikke en reklame-tjeneste — bare konvention + template.

### Kategori

**Design-system dokumentation** — reducerer duplikering mellem Figma og kode.

### Pas-i-arkitektur

- Parallelt med shadcn. `GetDesign.md` definerer tokens, shadcn-komponenter bruger dem.
- Git-versioneret, kan revieres sammen med kode.
- Build-script (npm) laver `tailwind.config.ts` fra `design.md`.

### Setup-trin

1. Oprette `design.md` i repo root med tokens (farver, typografi, spacing).
2. Opsætning npm-script til at parse `design.md` → `tailwind.config.ts`.
3. Integrer i Figma eksport-workflow (evt. via Figma plugin senere).

**Estimat**: 1–2 timer initial setup + løbende vedligeholdelse.

### Sikkerhed

- Ingen ekstern afhængighed (blot Markdown).
- Git-historik tracerer alle design-ændringer.

### Anbefaling

**MVP — JA, men lav prioritet.** Nice-to-have for design-system-modenhed. Hvis der er dedikeret designer i teamet: prioritér højere. Hvis frontend-dev selv fikser design: kan vente til fase 2.

---

## Ressource 4: Vercel MCP (Make Code Protocol — Vercel integration)

### Hvad det er

Vercel MCP er et sæt af tools til Vercel infrastructure via Claude/LLM-prompts. Inkluderer:
- Deploy-command (trigger production deploy).
- Environment-secret-management.
- Log-streaming + monitoring.
- Rollback og branch-preview kontrol.

### Status/version

- Launched 2025, stabil.
- Integration: via MCP protocol (ikke native SDK).

### Kategori

**Deployment-automation / Ops** — reducerer manuelle deploy-steps.

### Pas-i-arkitektur

- Fungerer med Vercel Pro (der bruges allerede).
- Komplementerer `vercel deploy` CLI.
- Kræver MCP-server i Claude/IDE setup.

### Setup-trin

1. Konfigurér MCP-server i `.claude/mcp.json` (eller Claude IDE).
2. Auth: Vercel API-token via environment.
3. Test commands: `mcp vercel:deploy`, `mcp vercel:logs`, etc.
4. Integreres i CI-pipeline (opcional — kan køres manuelt).

**Estimat**: 1–2 timer.

### Sikkerhed

- API-token skal i `.env` / Vercel secret management (ikke i kode).
- MCP-server lever lokalt (Claude Instance), ingen remote exec.

### Anbefaling

**MVP — NEJ, fase 1.5 kandidat.** Nyttig når deployment bliver hyppigere (fase 2), men ikke kritisk for MVP launch. Prioritér efter første stable deploy.

---

## Ressource 5: Next.js MCP (`next-devtools-mcp`)

### Hvad det er

Next.js DevTools MCP er et sæt af tools til at inspicere + manipulere Next.js App Router struktur:
- List alle routes + Server Components.
- Inspicér data-fetch dependencies.
- Trigger revalidation (manual `revalidatePath`).
- View build output + bundle-size.

### Status/version

- Launched 2025, community-driven.
- Kompatibel med Next.js 16+.

### Kategori

**Developer tooling / Debugging** — accelererer development-loop.

### Pas-i-arkitektur

- Løbende værktøj under udvikling (ikke deployment-afhængigt).
- Komplementerer `next dev` + VS Code debugger.

### Setup-trin

1. Install: `npm install next-devtools-mcp --save-dev`.
2. Konfigurér i `.next/` eller `next.config.ts`.
3. Start MCP-server: `npx next-devtools-mcp`.
4. Kan integreres i IDE.

**Estimat**: 30 min.

### Sikkerhed

- Kører lokalt (development only, ikke i prod).
- Giver indsigt i kode-struktur (OK for internal team).

### Anbefaling

**MVP — JA. Setup kort, høj værdi.** Især nyttigt når RSC + Server Actions kompleksificeres (flere routes, data-fetch-ordning). Prioriter efter shadcn + basic routes er opsat.

---

## Ressource 6: Vercel AI skills (agent-baseret Vercel-kommandoer)

### Hvad det er

Vercel AI skills er en extension til Vercel MCP som gør commands "agent-aware" — dvs. Claude kan automatisk vælge deploy-strategi, rollback-grund, eller preview-links baseret på kontekst.

### Status/version

- Launched 2025, stable.
- Baseret på Vercel MCP (se ressource 4).

### Kategori

**Deployment-automation, AI-assisted** — højtere niveau end base-MCP.

### Pas-i-arkitektur

- Kræver base Vercel MCP.
- Komplementerer Agent-framework (hvis team bruger agents til automatisering).

### Setup-trin

1. Afslut setup for Vercel MCP først.
2. Enable "ai skills" mode i MCP-konfiguration.
3. Konfigurér team-guidelines (hvornår auto-rollback, hvornår manual approval).

**Estimat**: 1 time.

### Sikkerhed

- Samme som base-MCP (API-token secret).
- Auto-deploy uden godkendelse kan være risiko — skal have manual-approval-gates.

### Anbefaling

**MVP — NEJ, fase 2 + under supervision.** Nice-to-have for automation men tilføjer kompleksitet som team skal være klar til. Hvis FGD ønsker "hands-off" deployments senere: revisit fase 2.

---

## Samlet anbefaling: MVP vs. senere

### MVP (fase 1) — Prioritet 1–2

| Ressource | Status | Ejer setup | Prioritet |
|---|---|---|---|
| **shadcn/ui skills** | Stable | Frontend-dev | 1 — kritisk |
| **Next.js MCP** (`next-devtools-mcp`) | Stable | Frontend-dev | 1 — hurtig gevinst |

### Fase 1.5–2

| Ressource | Status | Ejer setup | Prioritet |
|---|---|---|---|
| **GetDesign.md** | Konvention | Designer eller frontend-dev | 2 — nice-to-have |
| **Vercel MCP** | Stable | Camilla (ops) | 2 — deployment-automation |

### Fase 2+ eller vent

| Ressource | Status | Grund |
|---|---|---|
| **Refero** | Beta | Ukendt pricing/vilkår; pgvector dækker MVP |
| **Vercel AI skills** | Stable | Kompleksitet uden umiddelbar værdi; wait-and-see approach |

---

## Setup-ejer per ressource (MVP)

### shadcn/ui skills
**Ejer**: Frontend-udvikler  
**Handlinger**:
1. `npx shadcn-ui@latest init` (projekt-setup).
2. `npx shadcn-ui@latest add button dialog form textarea select input` (core components).
3. Design custom theme i `globals.css` eller vælg presets.
4. Dokumentér component-usage i `/docs/components.md`.

### Next.js MCP
**Ejer**: Frontend-udvikler (eller Camilla hvis ops-fokus)  
**Handlinger**:
1. `npm install next-devtools-mcp --save-dev`.
2. Konfigurér i `next.config.ts`.
3. Test: `npx next-devtools-mcp` og inspicér routes.

---

## Blockers / forudsætninger

1. **Design-stil fra FGD**: Skal vide hvilken design-æstetik (Linear / Vercel / Stripe / Notion / custom) før shadcn-tema vælges. Påvirker farve-tokens + typography.

2. **Figma-integration**: Hvis FGD har designer med Figma: GetDesign.md og shadcn design-tokens skal synkroniseres. Kræver Figma plugin senere.

3. **Vercel Pro-godkendelse**: Vercel MCP kræver Pro plan (allerede besluttet, men bekræft budget).

---

## Åbent til FGD

**Design-stil**: Which visual style do you prefer for the MVP?
- Linear (minimal, sans-serif, grays + accent).
- Vercel (black/white, bold typography).
- Stripe (clean, cool, professional).
- Notion (playful, rounded, colors).
- Custom (provide Figma link or color-palette).

Dit valg påvirker shadcn-tema og design-token-setup.

---

## Kilder

- shadcn/ui: https://ui.shadcn.com/
- Vercel MCP: https://vercel.com/docs/workflow-integrations/mcp
- Next.js MCP: https://github.com/vercel/next-devtools-mcp
- GetDesign.md convention: https://getdesign.co/ (eller local setup)
- Refero: https://refero.ai/ (beta)
