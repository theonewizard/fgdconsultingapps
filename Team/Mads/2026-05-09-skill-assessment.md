---
date: 2026-05-09
author: Mads (CISO)
status: anbefaling — ingen mandat-ændringer uden governance-proces
scope: permanent rolle, projekt-uafhængig
---

# Skill Assessment — Mads (CISO)

Dato: 2026-05-09. Vurdering baseret på skill-navn og forventet funktion. Konkret SKILL.md-indhold er ikke læst for hver skill — en fuld aktiverings-beslutning kræver det.

## 1. Skills der styrker kerneopgaver

| Skill | Kerneopgave styrket | Konkret nytteværdi |
|---|---|---|
| `security-review` | Compliance-overblik, threat-modeling | Struktureret sikkerhedsgennemgang af PR'er, arkitektur-udkast og leverandør-integrations; supplerer security-reviewers rapporter |
| `deep-research` | Design-fase-sparring (Poul), pen-test management | Dybdegående research på compliance-frameworks (FIPS 140-3 release notes, GDPR-guidance, ISO 27001:2022 annex A updates) uden at afvente Poul; øger min evne til at verificere Pouls analyser |
| `context7` | Compliance-overblik (allerede Must-use) | Bekræfter aktuelle versioner af frameworks og RFC'er — brug ved ethvert compliance-tjek (allerede i mandat) |
| `excalidraw-diagrams` | Design-fase-sparring, threat-modeling | Threat-model-diagrammer (STRIDE attack trees, data-flow-diagrammer til DPIA) — Mads kan tegne compliance-rammen, Poul tegner arkitekturen |
| `yaml` | Pipeline-opsætning | Review af CI/CD-pipeline-konfiguration (GitHub Actions, Dependabot, Semgrep-regler) for compliance-relevante parametre; ingen kodning, men governance-review |
| `mcp` | Design-fase-sparring | Forstå MCP-integrations attack surface ved leverandørvurderinger — relevant for GDPR data-flow og OWASP API Top 10 vurdering |

## 2. Irrelevante skills (ikke Mads' domæne)

| Skill | Begrundelse |
|---|---|
| `python-patterns` | Implementerings-patterns — hører til udvikler-roller, ikke governance |
| `neon-postgres` | Database-drift og schema-design — Camillas domæne |
| `ai-sdk` | AI-implementering — udvikler-roller; Mads vurderer compliance-aspekter af integrations, ikke SDK-brug |
| `devtools` | Browser-devtools til frontend-fejlfinding — ikke governance-relevant |
| `e2e-testing` | Test-implementering — Mads definerer krav (hvornår pen-test, hvad scope), men skriver ikke test-kode |
| `image` | Billedgenerering — ingen åbenbar governance-kobling |
| `coding-standards` | Kode-standarder til implementering — Mads input er compliance-krav til standarderne, ikke standarderne selv |
| `agent-browser` | Web-scraping/automation til research — hører til Poul (research) eller Frederik (pen-test prep) |

## 3. Anbefalinger til mandat-opdatering

**Ingen ændringer gennemføres uden governance-proces** (SOC 2 CC8.1, ISO 27001 A.8.32). Nedenstående er input til impact-vurdering.

### Foreslået Must-use (tilfojes eksisterende liste)

- `security-review` — naturlig forlængelse af eksisterende Must-use-logik; bruges ved enhver compliance-vurdering og threat-model-sparring

### Foreslået Should-use (tilfojes eksisterende liste)

- `deep-research` — relevant ved FIPS/SOC 2/GDPR-opdateringer og pen-test vendor-vurderinger; ikke hver opgave, men systematisk ved nye integrations og audit-forberedelse
- `excalidraw-diagrams` — relevant ved DPIA data-flow og STRIDE threat-model-diagrammer; situationsbestemt men hyppig nok til Should-use

### Forbliver Situational eller udgår

- `yaml` — behov dækkes allerede via Read/Edit ved pipeline-review; ingen formel opgradering nødvendig
- `mcp` — situationsbestemt ved MCP-leverandørvurdering; ikke fast behov

## 4. Governance-næste-skridt

Anbefalingerne ovenfor skal følge fuld governance-proces:

1. Impact-vurdering til `governance/change-requests/<dato>-mads-skills.md`
2. Mads security/SOD-vurdering + Dorthes GDPR-vurdering (parallelt)
3. Fælles anbefaling via Stefan til FGD
4. FGD eksplicit go — derefter Camilla committer mandat-ændring
5. Trine logger change-event i audit-trail
