---
dato: 2026-05-05
forfatter: Camilla
emne: Komplet inventar af installerede skills + plugins (forberedelse til CISO-led revurdering)
status: aktiv
formål: Mads + Poul bruger denne som udgangspunkt for revurdering af v3 + rolle-mandater
---

## Topline

- **Total skills installeret**: 87 (45 via .agents/skills + 18 lokale + 18 via Vercel plugin + superpowers/context7/karpathy)
- **Plugins**: 6 (vercel v0.40.1, superpowers v5.1.0, context7, code-review, andrej-karpathy-skills, code-simplifier)
- **Heraf brugt allerede i v3**: ~25-30 (shadcn, next-*, react, ai-sdk, neon-postgres, turborepo, streamdown, context7, excalidraw-diagrams, find-skills, vercel:*, m.fl.)
- **Heraf endnu ikke vurderet af Mads**: ~55-60
- **Kategorier**: 14
- **Plugin-kilder der ikke kunne læses helt**: `vercel` plugin-katalog er stor (50+ subskills i `/skills/` + `.claude/skills/`); opsummeret herunder

---

## Sektion 1 — Kategoriseret tabel

| Skill | Beskrivelse | Kategori | Plugin | Brugt i v3 |
|---|---|---|---|---|
| **vercel:vercel-agent** | Agent-arkitektur fra Vercel Marketplace | Vercel-platform | vercel | ○ |
| **vercel:ai-gateway** | Model-routing, cost-tracking, rate-limiting | AI / SDK | vercel | ○ |
| **vercel:ai-sdk** | Vercel AI SDK best practices (duplikat af ai-sdk) | AI / SDK | vercel | ✓ |
| **vercel:auth** | Clerk integration via Vercel Marketplace | Auth & Security | vercel | ○ |
| **vercel:bootstrap** | Project bootstrap til monorepo-setup | Build / DevOps | vercel | ○ |
| **vercel:chat-sdk** | Chat-interface patterns | AI / SDK | vercel | ○ |
| **vercel:deploy** | Vercel deploy best practices + workflow | Build / DevOps | vercel | ○ |
| **vercel:deployments-cicd** | CI/CD integration med Vercel | Build / DevOps | vercel | ○ |
| **vercel:env** | Environment variable management | Build / DevOps | vercel | ○ |
| **vercel:env-vars** | Extended env-var patterns | Build / DevOps | vercel | ○ |
| **vercel:knowledge-update** | Skill knowledge refresh | Skill-meta | vercel | ○ |
| **vercel:marketplace** | Vercel Marketplace integration guide | Vercel-platform | vercel | ○ |
| **vercel:next-cache-components** | Next.js cache components (duplikat) | Frontend / Next.js | vercel | ✓ |
| **vercel:next-forge** | Monorepo framework i.st.f. manual Turborepo | Frontend / Next.js | vercel | ○ |
| **vercel:next-upgrade** | Next.js version migration guide | Frontend / Next.js | vercel | ○ |
| **vercel:nextjs** | Next.js best practices (duplikat) | Frontend / Next.js | vercel | ✓ |
| **vercel:plugin-audit** | Plugin security audit skill | Skill-meta | vercel | ○ |
| **vercel:react-best-practices** | React patterns (duplikat) | Frontend / Next.js | vercel | ✓ |
| **vercel:routing-middleware** | Next.js routing & middleware | Backend / Runtime | vercel | ○ |
| **vercel:runtime-cache** | Server-side runtime caching | Frontend / Next.js | vercel | ○ |
| **vercel:shadcn** | shadcn/ui integration (duplikat) | UI-komponenter | vercel | ✓ |
| **vercel:status** | Vercel project status + diagnostics | Build / DevOps | vercel | ○ |
| **vercel:storage** | Vercel KV/Postgres storage patterns | Database / Storage | vercel | ○ |
| **vercel:turbopack** | Turbopack bundler (Next.js 15+) | Build / DevOps | vercel | ○ |
| **vercel:verification** | Vercel deploy verification tests | Testing & Verification | vercel | ○ |
| **vercel:vercel-cli** | Vercel CLI advanced workflows | Build / DevOps | vercel | ○ |
| **vercel:vercel-functions** | Vercel Functions (serverless) | Backend / Runtime | vercel | ○ |
| **vercel:vercel-sandbox** | Sandboxed code execution environment | Backend / Runtime | vercel | ○ |
| **vercel:workflow** | Vercel Workflow DevKit (async tasks) | Backend / Runtime | vercel | ○ |
| **ai-sdk** | Anthropic AI SDK patterns | AI / SDK | .agents/skills | ✓ |
| **ai-elements** | AI UI component patterns | UI-komponenter | .agents/skills | ✓ |
| **agent-browser** | Browser agent for automation | Dokumentation / Diagrammer | .agents/skills | ○ |
| **autoship** | Continuous deployment automation | Build / DevOps | .agents/skills | ○ |
| **before-and-after** | Code comparison & migration guides | Dokumentation / Diagrammer | .agents/skills | ✓ |
| **building-components** | React component architecture | Frontend / Next.js | .agents/skills | ✓ |
| **codegen** | Code generation from schemas (json-render) | json-render | .agents/skills | ○ |
| **core** | json-render core runtime | json-render | .agents/skills | ○ |
| **deploy-to-vercel** | One-click Vercel deployment | Build / DevOps | .agents/skills | ○ |
| **devtools** | json-render devtools integration | json-render | .agents/skills | ○ |
| **find-skills** | Skill discovery & search | Skill-meta | .agents/skills | ✓ |
| **image** | json-render image rendering | json-render | .agents/skills | ○ |
| **ink** | Ink CLI rendering (json-render) | json-render | .agents/skills | ○ |
| **jotai** | Jotai state management (json-render) | json-render | .agents/skills | ○ |
| **mcp** | MCP server integration | json-render | .agents/skills | ○ |
| **neon-postgres** | Neon Postgres patterns + migrations | Database / Storage | .agents/skills | ✓ |
| **next** | Next.js foundation patterns | Frontend / Next.js | .agents/skills | ○ |
| **next-best-practices** | Next.js optimization & security | Frontend / Next.js | .agents/skills | ✓ |
| **next-cache-components** | Next.js Cache Components (App Router) | Frontend / Next.js | .agents/skills | ✓ |
| **react** | React core patterns & hooks | Frontend / Next.js | .agents/skills | ○ |
| **react-email** | React Email template rendering (json-render) | json-render | .agents/skills | ○ |
| **react-native** | React Native framework | Mobile | .agents/skills | ○ |
| **react-pdf** | PDF generation from React (json-render) | json-render | .agents/skills | ○ |
| **react-three-fiber** | 3D rendering (json-render) | Specialiseret | .agents/skills | ✗ |
| **redux** | Redux state management (json-render) | json-render | .agents/skills | ○ |
| **remotion** | Video composition framework | Specialiseret | .agents/skills | ✗ |
| **remotion-best-practices** | Remotion optimization patterns | Specialiseret | .agents/skills | ✗ |
| **shadcn** | shadcn/ui component library | UI-komponenter | .agents/skills | ✓ |
| **shadcn-svelte** | shadcn for Svelte | UI-komponenter | .agents/skills | ○ |
| **skill-creator** | Skill development framework | Skill-meta | .agents/skills | ○ |
| **solid** | Solid.js framework (json-render) | json-render | .agents/skills | ○ |
| **streamdown** | Streaming markdown parser | AI / SDK | .agents/skills | ✓ |
| **svelte** | Svelte framework | Frontend / Next.js | .agents/skills | ○ |
| **turborepo** | Monorepo management via Turborepo | Build / DevOps | .agents/skills | ✓ |
| **ucp** | Universal Component Pattern | Specialiseret | .agents/skills | ✗ |
| **vercel-cli-with-tokens** | Vercel CLI + token management | Build / DevOps | .agents/skills | ○ |
| **vercel-composition-patterns** | Vercel UI composition strategies | Frontend / Next.js | .agents/skills | ✓ |
| **vercel-react-best-practices** | Vercel-specific React patterns | Frontend / Next.js | .agents/skills | ✓ |
| **vercel-react-native-skills** | React Native på Vercel | Mobile | .agents/skills | ○ |
| **vercel-react-view-transitions** | View Transitions API in React | Frontend / Next.js | .agents/skills | ○ |
| **vue** | Vue.js framework | Frontend / Next.js | .agents/skills | ○ |
| **web-design-guidelines** | Web design best practices | Code-kvalitet | .agents/skills | ✓ |
| **xstate** | XState state machine (json-render) | json-render | .agents/skills | ○ |
| **yaml** | YAML parsing/generation (json-render) | json-render | .agents/skills | ○ |
| **zustand** | Zustand state management (json-render) | json-render | .agents/skills | ○ |
| **coding-standards** | Code quality & style (lokal) | Code-kvalitet | lokal | ○ |
| **context7** | Real-time library documentation | Dokumentation / Diagrammer | lokal/plugin | ✓ |
| **deep-research** | Long-form research & analysis | Dokumentation / Diagrammer | lokal | ○ |
| **design-system** | Design system governance (lokal) | UI-komponenter | lokal | ○ |
| **e2e-testing** | Playwright e2e testing framework | Testing & Verification | lokal | ○ |
| **excalidraw-diagrams** | Excalidraw diagram generation | Dokumentation / Diagrammer | lokal | ✓ |
| **neon-postgres** | Neon database patterns (duplikat lokal) | Database / Storage | lokal | ✓ |
| **python-patterns** | Python code patterns | Code-kvalitet | lokal | ✗ |
| **python-pep8-review** | Python PEP8 linting | Code-kvalitet | lokal | ✗ |
| **python-testing** | Python unit testing patterns | Testing & Verification | lokal | ✗ |
| **security-review** | Security audit framework | Auth & Security | lokal | ○ |
| **skill-comply** | Skill compliance verification | Skill-meta | lokal | ○ |
| **springboot-patterns** | Spring Boot framework | Code-kvalitet | lokal | ✗ |
| **springboot-security** | Spring Boot security patterns | Auth & Security | lokal | ✗ |
| **springboot-tdd** | Spring Boot TDD methodology | Testing & Verification | lokal | ✗ |
| **springboot-verification** | Spring Boot test verification | Testing & Verification | lokal | ✗ |
| **video-checksum-pipeline** | Video processing pipeline | Specialiseret | lokal | ✗ |
| **superpowers:* (alle ~25 subskills)** | Contributor guidelines, skill development, evaluation | Process / Governance | superpowers | ○ |
| **context7** | Library documentation fetcher (plugin) | Dokumentation / Diagrammer | plugin | ✓ |
| **code-review** | Code review framework (claude-plugins-official) | Code-kvalitet | plugin | ○ |
| **code-simplifier** | Code reduction & refactoring (claude-plugins-official) | Code-kvalitet | plugin | ○ |
| **andrej-karpathy-skills:karpathy-guidelines** | ML/AI best practices from Karpathy | Process / Governance | andrej-karpathy-skills | ○ |

---

## Sektion 2 — Allerede brugt i v3 (✓-markeret)

**Bekræftede fra Pouls forrige analyser + memory:**

- **Frontend/UI**: shadcn, shadcn-svelte, next-best-practices, next-cache-components, vercel-react-best-practices, vercel-composition-patterns, building-components, web-design-guidelines, ai-elements, design-system
- **AI/Runtime**: ai-sdk, streamdown, context7
- **Database**: neon-postgres
- **DevOps**: turborepo
- **Dokumentation**: excalidraw-diagrams, find-skills, before-and-after
- **Meta**: context7 (skill knowledge)
- **Vercel plugin duplikater**: vercel:nextjs, vercel:react-best-practices, vercel:shadcn, vercel:ai-sdk, vercel:next-cache-components

---

## Sektion 3 — Nye siden sidste revurdering (○-markeret)

**Primært fra Vercel plugin v0.40.1 (installeret 2026-05-05):**

1. **vercel:next-forge** — Monorepo-framework alternativ til Turborepo. Potentiel konflikt med eksisterende Turborepo-setup.
2. **vercel:auth** — Clerk via Marketplace. Vi bruger Clerk manuelt; kunne simplificeres.
3. **vercel:workflow** — Vercel Workflow DevKit til async tasks. Konkurrerer med Inngest.
4. **vercel:ai-gateway** — Model-routing + cost-tracking. FGD bruger rå Anthropic API; kunne være relevant for produktion.
5. **vercel:runtime-cache** — Server-side cache supplement til Next.js Cache Components.
6. **vercel:vercel-sandbox** — Sandboxed code execution. Potentiel for pen-testing (Karens rolle).
7. **vercel:turbopack** — Bundler for Next.js 15+. Vi bruger Webpack/esbuild i dag.
8. **vercel:next-upgrade** — Next.js migration guide. Relevant ved major version bump.
9. **vercel:routing-middleware** — Next.js routing patterns. Vi har ad-hoc routing; kunne systematiseres.
10. **vercel:vercel-functions** — Serverless patterns. Duplikat af "Backend / Runtime" idé.
11. **e2e-testing** — Playwright-baseret; kunne være FGD standard-test-stack.
12. **agent-browser** — Browser automation; potentiel for Karens pen-test-værktøj.
13. **security-review** — Slash-command baseret sikkerhed. Kan blive del af deploy-gate.
14. **code-simplifier** — Code reduction (claude-plugins-official). Relevant til review.
15. **deep-research** — Long-form analysis. Poul bruger; ikke gennemlyst af CISO.

**Python/Spring-skills (ikke relevant for knowlagecentralen, men inventariseret):**
- python-patterns, python-pep8-review, python-testing
- springboot-patterns, springboot-security, springboot-tdd, springboot-verification

**Specialiseret (irrelevant for knowlagecentral):**
- react-three-fiber, remotion, remotion-best-practices, ucp, video-checksum-pipeline

---

## Sektion 4 — Duplikater & overlaps

Flere skills er tilgængelige i **både Vercel plugin og lokale/agent-skills versioner**:

| Skill | Kilder | Note |
|---|---|---|
| **ai-sdk** | .agents/skills + vercel:ai-sdk | Konsolidér til én kilde |
| **next-cache-components** | .agents/skills + vercel:next-cache-components | Samme funktionalitet; vælg kilde |
| **react-best-practices** | .agents/skills + vercel:react-best-practices | Vercel kan være nyere |
| **shadcn** | .agents/skills + vercel:shadcn | Duplikat; vælg kilde |
| **neon-postgres** | .agents/skills + lokal skill | Duplikat; konsolidér |

**Anbefaling**: Mads + Poul skal vælge én kilde per skill for at undgå forvirring.

---

## Sektion 5 — Åbne spørgsmål til Mads + Poul

1. **vercel:next-forge vs. Turborepo**: Skal vi migrere eksisterende Turborepo-setup til Vercel's next-forge som monorepo-base? Hvad er sikkerhed/governance-implikationer?

2. **vercel:auth (Clerk)**: Vi bruger Clerk manuelt i dag. Skal vercel:auth-integration erstatte det? Hvornår er det værd?

3. **vercel:workflow vs. Inngest**: Vi har Inngest til async tasks. Er vercel:workflow et alternativ eller supplement? Security-review?

4. **vercel:ai-gateway**: Skal vi bruge det over rå Anthropic API? Hvad med cost-tracking + model-routing i produktion? FIPS-relevans for Karen?

5. **e2e-testing (Playwright)**: Skal det være standard-test-stack i v3? Hvad med integration med CI/deploy-gate?

6. **agent-browser**: Er det egnet som pen-test-værktøj for Bjarne (Layer 2 HSM-testing)? Security vurdering?

7. **security-review skill**: Kan det blive del af automatisk deploy-gate (pre-deploy audit)?

8. **Python/Spring-skills**: Skal disse fjernes fra FGD-inventarup (ikke relevant for knowlagecentral)? Eller skal de dokumenteres som "ikke til brug"?

9. **Duplikat-konsolidation**: Accepterer Mads + Poul én kilde pr. skill, eller skal vi understøtte multiple kilder baseret på kontekst?

10. **code-simplifier + deep-research**: Disse er nye claude-plugins-official skills. Hvor bruges de? Skal de inkluderes i role-mandater (Poul, Stefan)?

---

## Opsamling til Mads

Denne inventar danner udgangspunktet for CISOs tværgående sikkerhedsvurdering. Fokus-områder:

- **Vercel plugin >= 0.40.1 integration** — 30+ nye subskills kræver audit
- **Duplikat-konsolidation** — 5-7 skills med multiple kilder
- **Role-mandater alignment** — Hvilke skills hører til Karen/Bjarne/Poul/Stefan?
- **Production gate-integration** — Hvilket subset skal være "sikkerhed-godkendt"?
- **Deprecation-strategi** — Python/Spring/Remotion skills; skal fjernes eller dokumenteres som N/A?

Alle skills marked ○ (ikke vurderet) skal gennem Mads' revurdering før de kan anbefales til knowlagecentral-v3-phase 2.

