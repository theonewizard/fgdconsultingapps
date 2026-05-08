---
dato: 2026-05-05
fra: Laila
til: FGD
emne: Tre nye roller — Sofie, Henrik, Anders — udkast til godkendelse
status: klar til FGD-review og godkendelse
---

# Vidensapp-projekt: 3 nye roller — klar til launch

## Resumé

Baseret på Pouls konsoliderede v3-analyse (`Team/Poul/2026-05-05-vidensapp-analyse-v3.md`) og arkitektur-diagram har jeg designet tre specialiseret roller til vidensapp-projektet:

1. **Sofie** — Frontend-udvikler (Next.js, RSC, design-system, PWA)
2. **Henrik** — Backend-udvikler (Vercel Functions, Drizzle, Neon RLS, OpenAPI 3.1)
3. **Anders** — AI-pipeline-designer (Inngest, Anthropic, Voyage, prompt-management)

Alle tre roller følger eksisterende Stefan-team-konventioner: dansk kommunikation, least-privilege tools, OWASP API Top 10 + RFC-standarder, og tydelige hand-off-aftaler med hinanden og med Camilla/Poul/Stefan.

## Rolle-oversigt

| Rolle | Model | Primært fokus | Hard rules (vigtigste) | Klargøring |
|---|---|---|---|---|
| **Sofie** | sonnet | Frontend: Browser, RSC, design-system, accessibility | Aldrig Edge runtime; OpenAPI-genereret klient; WCAG 2.2 AA non-negotiable | Next.js 16.2, React 19.2, Tailwind v4, shadcn/ui |
| **Henrik** | sonnet | Backend: API, Drizzle, RLS, Clerk, OpenAPI contract | Alle routes Node.js; OpenAPI single source of truth; idempotency + RFC 9457 | Drizzle 0.45.2, Neon + RLS, Clerk 7.2, Vercel Functions |
| **Anders** | sonnet | AI-pipeline: Inngest, Anthropic, Voyage, prompts, budgets | Inngest Node.js; Zod-validering før DB-write; note-body lækker aldrig til events; spend-caps enforced | Inngest v4, AI SDK v6, Anthropic spend-caps, prompt-bibliotek |

**Valg af model**: Alle tre får **sonnet** da:
- Frontend-arbejde (Sofie) kræver god context-håndtering for komplekst React/RSC-ræsonnement.
- Backend-API-design (Henrik) kræver nuanceret ræsonnement omkring sikkerhed, RLS-policies, migrations.
- Prompt-engineering (Anders) kræver dybt ræsonnement om prompt-kvalitet, error-paths, budget-trade-offs.

Haiku ville være for let for disse opgaver; Opus er overkill da Poul (opus) allerede håndterer dyb research.

## Hand-off-matrice (der findes også i Excalidraw-diagram fra Poul)

```
Sofie (Frontend)
  ↓ sender OpenAPI-kontrakt til ↓
Henrik (Backend)
  ↓ sender Inngest webhook-format til ↓
Anders (AI-pipeline)
  ↓ rapporterer job-status via API til ↓
Sofie (UI refreshes jobs)

Alle tre:
  ↓ rapporterer dagligt til → Stefan
  ↓ koordinerer deploy-seq med → Camilla
  ↓ eskalerer sikkerhed-tvivl til → Mads (CISO)
```

## Arbejdsgang ved projekt-start (Fase 0 — forberedelse)

1. **Camilla sætter infra op** (Vercel projekt, Neon, Clerk, Anthropic-workspaces, git-repo).
2. **Henrik designes OpenAPI 3.1** contract (baseret på Pouls datacflow i v3 §3.3).
3. **Sofie prototyper UI** baseret på Henrik's OpenAPI.
4. **Anders designes Inngest-flow** baseret på Henrik's webhook-endpoint.
5. **Alle tre leverer til Stefan** ved hver milepæl (component-audit, API-lint, budget-plan).

Estimeret forberedelsestid: 1 uge. MVP-build: 2-3 uger efter.

## Filer du skal godkende

3 mandat-filer ligger i `Min Inbox/`:

- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/projects/vidensapp/2026-05-05-sofie-frontend-mandat.md`
- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/projects/vidensapp/2026-05-05-henrik-backend-mandat.md`
- `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/projects/vidensapp/2026-05-05-anders-ai-pipeline-mandat.md`

## Spørgsmål til dig

1. **Model-valg**: synes du sonnet er rigtig for alle tre, eller vil du justere (fx Henrik → haiku hvis primarily implementation, ikke design)?
2. **Tool-liste**: jeg har givet alle `Bash` (nødvendig til dependency-check, build-debug, git-status). OK?
3. **Fase-timing**: skal vi ansætte alle tre nu og lade dem forberede i parallel, eller vil du vente til efter Camilla's infra-setup?

## Næste skridt

**Godkendelse (du)**:
1. Læs de 3 mandater i `Min Inbox/`.
2. Hvis OK: skriv en kort besked "Godkendt" — jeg sender dem til Camilla for commit.
3. Hvis ændringer: red direkte i filerne eller send mig kommentarer.

**Efter godkendelse (mig + Camilla)**:
1. Jeg beder Camilla flytte filerne fra `Min Inbox/` til `.claude/agents/` og committe.
2. Camilla updater `Team/_konventioner.md` hvis nye navne skal dokumenteres (de bliver allerede listet i puljen).
3. Stefan får besked om at de tre roller er aktive.

**Camilla's infra-setup** starter parallelt — hun koordinerer med dig når accounts og secrets skal konfigureres.

---

**Klar til at gå videre når du siger ja.** Tak.
