---
dato: 2026-05-09
forfatter: Trine (Compliance Auditor)
emne: Skill-assessment — installerede skills vurderet mod audit-kerneopgaver
status: klar til Stefan / anbefaling til Mads
---

# Skill-assessment: Trine — 2026-05-09

Vurdering af 5 installerede skills mod Trines kerneopgaver: ISO 27001:2022 + SOC 2 Type II — control-mapping, evidens-indsamling, audit-forberedelse, månedsrapporter.

## Vurderingstabel

| Skill | Teknisk indhold | Fit til audit-kerneopgave | Rolle-konflikt / SOD-note | Anbefaling |
|---|---|---|---|---|
| `context7` | Henter aktuel library/framework-dokumentation via MCP | Allerede Must-use i Trines mandat. Bruges ved opdateringer til ISO 27001:2022, SOC 2 TSC-revisioner og COSO. | Ingen konflikt | Allerede adopteret — ingen ændring |
| `deep-research` | Multi-source webresearch via firecrawl/exa MCP; producerer citerede rapporter | Stærk fit: ISO 27001-kontrol-fortolkninger, SOC 2 TSC-updates, COSO-revisioner kræver flerkilde-syntese med kildehenvisninger — præcis det deep-research leverer. | Ingen konflikt; research er Trines eget domæne (verifikation kræver forståelse af standarderne) | Tilfojes som Should-use |
| `security-review` | Next.js/Supabase-developer-checklist: input-validering, SQL-injection, JWT, RLS, CSRF, rate-limiting m.m. | Begrænset direkte fit. Indholdet er en implementerings-checkliste til developer-roller — ikke en audit-rubrik. Nyttig som referenceramme for hvilken evidens Trine bor forvente fra backend-/frontend-udviklere, men Trine anvender den ikke til at reviewe kode. | SOD-risiko: adoption som eget arbejdsredskab trækker Trine mod developer-review-territoriet (security-reviewer/Karen's ansvarsomrade). Bryder auditor-uafhankighed. | Situational — kun som referenceramme for evidens-forventninger; aldrig som kode-review-vaerktoj. Ingen tilfoejelse til mandat-skills |
| `yaml` | `@json-render/yaml` — streaming YAML-wire-format til UI-rendering med AI SDK (yaml-spec/yaml-edit fences, TransformStream) | Ingen fit. Dette er et frontend-renderings-bibliotek, ikke et generelt YAML-dokumentationsvaerktoj. Evidens-indsamling i YAML-format er en dokumentations-konvention Trine kan adoptere uafhaengigt af dette skill. | Ingen SOD-konflikt, men heller ingen audit-relevans | Afvist — tilhorer frontend-udvikler-rollen |
| `agent-browser` | Browser-automation CLI (CDP/Chrome); sessioner, screenshots, formularudfyldning, web-scraping | Smal fit: kunne bruges til at browse offentlige standarder (ISO/NIST/AICPA). Risiko: evidens-indsamling via live browser-session kan betyde Trine genererer evidens i stedet for at verificere produceret evidens — bryder auditor-uafhangighed. | SOD-overlap med Frederik (pen-test-koordinator), der ejer agent-browser per Mads' rolle-tabel. | Situational — kun til opslag i offentlige standarddatabaser; aldrig til penetrationstest eller aktiv evidens-generering. Anbefales afklaret med Mads og Frederik for klar snitflade |

## Nettoanbefaling

**Tilfoej til Should-use:** `deep-research` — eneste skill med klar, konfliktfri fit til Trines kerneopgaver.

**Behold som Must-use:** `context7` — allerede i mandatet, ingen aendring nodvendig.

**Afvis til mandatoptagelse:** `yaml` (forkert domane), `security-review` (SOD-risiko ved adoption som eget vaerktoj), `agent-browser` (snitflade til Frederik uafklaret; auditor-uafhangighedsrisiko).

## Procesbemaerkning

Enhver aendring af Trines mandats Skills-sektion (fx tilfoejelse af `deep-research` som Should-use) udloser change-request-flowet under Trines eget mandat §Mandat-stabilitet og Mads' governance-policy. Denne assessment er en anbefaling til Stefan (videre til Mads + FGD) — Trine aendrer ikke sit eget mandat. Camilla committter forst efter eksplicit FGD-go med skriftlige vurderinger fra Mads og Dorthe.
