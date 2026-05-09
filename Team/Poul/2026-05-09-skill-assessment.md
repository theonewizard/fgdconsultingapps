---
forfatter: Poul (analytiker)
dato: 2026-05-09
emne: Skill-assessment for kerneopgaver
status: udkast til Stefan/FGD
---

# Skill-assessment — Poul (analytiker)

## Sammendrag

Af de 12 installerede skills styrker **5 direkte** mine kerneopgaver (research, framework-analyse, arkitektur, peer-review). **3 er situationelle** (når specifikt trigger). **4 er irrelevante** for min rolle (frontend/Python/MCP-rendering — hører under udviklerteamet). Jeg anbefaler at gøre `context7`, `deep-research`, `excalidraw-diagrams` og `security-review` til mine **standard-skills**, og dokumentere det i mit mandat.

## Vurdering pr. skill

| Skill | Relevans for mine kerneopgaver | Anbefaling |
|---|---|---|
| `context7` | **Kritisk** — version-verificering (Next.js, Drizzle, Neon, AI SDK, Clerk). Allerede must-use i mit mandat. | **Standard** (allerede listet) |
| `deep-research` | **Høj** — strukturerer research-metodik: 3-5 sub-spørgsmål, multi-source firecrawl/exa, citerede rapporter. Matcher præcis min arbejdsgang for vendor-vurdering og framework-research. **MCP-krav: firecrawl ELLER exa** — skal verificeres at en af dem er konfigureret før jeg kan bruge skillen produktivt. | **Standard** (med MCP-forbehold) |
| `excalidraw-diagrams` | **Høj for arkitektur-review** — JSON-format med rectangle/arrow/text. Jeg har allerede leveret `2026-05-05-knowlagecentral-arkitektur.excalidraw`; skill formaliserer mønstret (boundElements, containerId, focus/gap på arrows). Forbedrer reproducerbarhed. | **Standard for arkitektur-leverancer** (allerede listet som situational — opgrader) |
| `security-review` | **Medium-høj for peer-sparring med Mads** — OWASP-checklist + auth/secrets/input-validation. Matcher min "OWASP API Top 10"-kompetence. Mads ejer det formelle compliance-arbejde, men jeg bruger den når jeg laver arkitektur-pre-review før Mads får dossieret. | **Standard ved arkitektur-review med sikkerheds-impact** |
| `agent-browser` | **Lav-medium** — primært Frederiks pen-test-værktøj (jf. `pen-test-vendor-2026-05-05.md`). Jeg kan bruge det til vendor-screening (browse Yubico/Improsec-sites), men WebFetch dækker mit behov. | **Situational** (overlader til Frederik) |
| `coding-standards` | **Lav** — baseline frontend/backend-konventioner. Hører under udviklerteamet. Jeg refererer kun, hvis arkitektur-review berører kode-niveau. | Skip |
| `yaml` | **Lav** — `@json-render/yaml` wire-format, ikke generel YAML. Irrelevant for mine analyser. | Skip |
| `python-patterns` | **Ingen** — Knowlagecentral-stack er Node.js/TypeScript. | Skip |
| `ai-sdk` | **Lav-medium** — relevant når jeg framework-vurderer AI SDK v6 (jf. memory: `vercel-neon-strategi-2026-05-05.md`), men context7 dækker dokumentations-opslag bedre. | Situational |
| `mcp` | **Ingen** — `@json-render/mcp` (UI-rendering for MCP), ikke MCP-protokol-research. | Skip |
| `image` | **Ingen for billed-analyse** — det er en **JSON→SVG/PNG-renderer (Satori-baseret)**, ikke et image-analyse-værktøj. **Svar på Stefans spørgsmål 4: Nej, jeg kan ikke analysere PKI-hierarki-billeder via denne skill.** Til billede-analyse bruger jeg Read-tool direkte (multimodal) på `.png`/`.jpg`/`.pdf`. | Skip — misforståelse afklaret |
| `devtools` | **Ingen** — runtime-inspector for json-render-apps. Udvikler-værktøj. | Skip |

## Specifikke svar på Stefans spørgsmål

**(1) Hvilke skills styrker konkret?** `context7`, `deep-research`, `excalidraw-diagrams`, `security-review`.

**(2) `deep-research` — strukturerer research-metodik?** Ja. Skillen kodificerer det jeg allerede gør ad-hoc: dekomponer i 3-5 sub-spørgsmål, multi-source-search, deep-read 3-5 nøgle-kilder, syntese med citater + confidence-rating. Forbedrer mest min reproducerbarhed og leverance-format. **Forudsætning**: firecrawl eller exa MCP skal være tilgængelig — ellers reducerer skillen til metodik-skabelon (hvilket alligevel er værdifuldt).

**(3) `excalidraw-diagrams` — bedre arkitektur-diagrammer?** Ja. Mit eksisterende `.excalidraw`-output er manuelt; skillen formaliserer bindings (`boundElements`, `containerId`, arrow `startBinding`/`endBinding`) så diagrammer er stabile under flytning. Anbefaler den som standard ved alle arkitektur-leverancer.

**(4) `image` — analysere PKI-hierarki-billeder?** **Nej** — `image`-skill er en JSON-til-billede-renderer. Til billed-analyse bruger jeg Read-tool direkte (multimodal Claude læser `.png`/`.jpg` indholdsmæssigt). **Ingen ny skill nødvendig**.

**(5) Anbefalede standard-skills i mit mandat:**

Tilføj til "Must-use" i mandatet:
- `superpowers:deep-research` (research-strukturering — kræver firecrawl/exa MCP)
- `superpowers:security-review` (når arkitektur-review har sikkerheds-impact, før Mads-handoff)

Opgrader fra "Situational" til "Should-use":
- `excalidraw-diagrams` (alle arkitektur-leverancer, ikke kun komplekse)

Behold som "Must-use":
- `context7` (uændret — kritisk for version-verificering)

## Mandat-ændring

Tilføjelse af must-use/should-use skills til mit mandat (`.claude/agents/poul-analytiker.md`) er **ikke** låst kontrolpunkt (model/tools/hard-rules/description). Skill-listen er beskrivende dokumentation — kan opdateres som almindelig ændring efter Stefans/FGD's accept. **Ingen change-request-proces nødvendig.**

Hvis FGD vil acceptere ovenstående, beder jeg Stefan koordinere opdatering af mandat-fil via Camilla.

## Risici / forbehold

- `deep-research` afhænger af firecrawl/exa MCP. Hvis ingen er konfigureret, falder skillen tilbage til ren metodik-skabelon. **Action**: bed Stefan/Camilla verificere MCP-konfiguration.
- `security-review`-skill er generisk OWASP-checklist; den **erstatter ikke** Mads' formelle threat-modeling eller Trines compliance-audit. Jeg bruger den som **pre-review** før hand-off, ikke som final security-vurdering.

## Kilder

- `/Users/fgd/.claude/skills/deep-research/SKILL.md`
- `/Users/fgd/.claude/skills/excalidraw-diagrams/SKILL.md`
- `/Users/fgd/.claude/skills/image/SKILL.md` (afklaret som renderer, ikke analyzer)
- `/Users/fgd/.claude/skills/context7/SKILL.md`
- `/Users/fgd/.claude/skills/security-review/SKILL.md`
- Mit eget mandat: `.claude/agents/poul-analytiker.md`
