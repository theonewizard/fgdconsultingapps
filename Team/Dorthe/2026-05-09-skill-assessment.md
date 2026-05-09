---
forfatter: Dorthe (DPO)
dato: 2026-05-09
emne: Skill-assessment — relevans for GDPR-compliance-arbejde
type: vurdering
til: Stefan -> FGD
---

# Skill-assessment: nye skills imod DPO-kerneopgaver

Vurdering af `deep-research`, `context7`, `yaml` og `agent-browser` imod mine kerneopgaver: ROPA, DPIA, breach-respons, data-subject-rettigheder, DPA-negotiation, customer-DPA-templates.

## Bemærkning om verifikation

`context7` er bekræftet tilgængelig som MCP-server (jf. session-instruks). De øvrige tre fremgår ikke af `skills/skills/`-mappen i repo'et som SKILL.md-pakker; jeg vurderer på basis af brief'en og antager at de er tilgængelige som annonceret. Hvis nogen ikke er aktiveret, skal Camilla bekræfte og jeg justerer.

## 1. `deep-research` — STÆRKT JA, must-use

**Relevans: Høj.** Mit arbejde er research-tungt på områder hvor systematik og kildeporing er retsligt påkrævet:

- **Lovgivnings-research**: GDPR art. 1-99 + national gennemførelse (databeskyttelsesloven), EDPB-guidelines (1/2024 om legitimate interests, 2/2023 om art. 6(1)(f), m.fl.), Datatilsynets vejledninger, CJEU-domme (Schrems II, Bindende afgørelser).
- **DPA-analyse**: systematisk sammenligning af leverandør-DPA'er (Neon, Vercel, Clerk, Anthropic, Inngest, Voyage, Tailscale, Yubico) imod SCC 2021/914 + krav i art. 28(3).
- **Adequacy-decisions**: tracking af tredjelands-status (US Data Privacy Framework, UK adequacy, m.fl.) — kræver struktureret kildeporing.
- **DPIA-evidens**: art. 35 kræver dokumenteret risk-assessment med kildereferencer.

**Anvendelse**: alle DPIA'er, alle DPA-reviews, breach-impact-vurderinger.

**Forbehold**: GDPR-research kræver primær-kilde (eur-lex.europa.eu, edpb.europa.eu, datatilsynet.dk) — ikke sammendrag. Skill skal kunne pege til officielle URLs, ikke kun sekundærkilder.

## 2. `context7` — JA, must-use (eksisterende dependency)

**Relevans: Mellem-høj.** Allerede listet i mit nuværende must-use. Bekræftet stadig relevant.

- **SDK-version-tracking**: Anthropic SDK, Voyage SDK, Clerk SDK — versionsskift kan påvirke data-flow (fx nye telemetri-felter, nye retention-defaults). Påvirker ROPA + DPIA.
- **DPA-version-tracking**: SCC-versioner, leverandør-T&C'er (mindre godt dækket — primær web-kilde foretrækkes).

**Begrænsning**: context7 er primært for tekniske SDK'er/frameworks, ikke retskilder. Bruger den hvor det giver mening (SDK-versioner), ikke til EDPB/Datatilsynet.

## 3. `agent-browser` — BETINGET JA, situational

**Relevans: Mellem.** Kan teoretisk hente Datatilsynets sider og EU-retsakter direkte, men:

- **edpb.europa.eu, datatilsynet.dk, eur-lex.europa.eu** — alle er offentlige, statiske HTML/PDF. WebFetch er ofte tilstrækkeligt; agent-browser tilfører værdi når sider er JS-renderede eller bag søge-formularer (fx Datatilsynets afgørelses-database).
- **Leverandør-DPA-portaler**: nogle leverandører (Vercel, Clerk) har DPA bag login eller dynamiske sider — her er agent-browser nyttigt.
- **Customer-DPA-research**: konkurrent-analyse (hvordan tilbyder andre AI-vendors DPA'er til kunder) — typisk bag click-throughs.

**Anvendelse**: når WebFetch ikke kan opløse JS-rendering eller søge-flows. Ikke standard, kun ved konkret behov.

**Vigtig forbehold (uafhængighed)**: agent-browser bruges også af Frederik (pen-test). Min brug er udelukkende til research, aldrig til probing — adskillelse mellem DPO-research og pen-test-aktivitet skal være tydelig i logs.

## 4. `yaml` — NEJ, ikke relevant for mig

**Relevans: Lav.** YAML-håndtering er udvikler/devops-arbejde. Mit output er prosa-dokumenter (ROPA-tabeller i markdown, DPIA-rapporter, breach-notifikationer). Frontmatter i mine notater håndteres allerede via Write/Edit. Hvis ROPA en dag eksporteres som struktureret YAML/JSON for tooling, så kunne det blive relevant — men det er Camillas/backend-udviklerens domæne, ikke mit.

## Anbefaling — opdateret skill-konfiguration

Tilføj til **must-use** (anvendes ved hver opgave i kategorien):

- `deep-research` — DPIA, DPA-analyse, lovgivnings-research, breach-impact-vurdering
- `context7` — uændret (SDK-version-tracking)
- `superpowers:writing-plans` — uændret
- `web-design-guidelines` — uændret

Tilføj til **situational** (kun ved konkret trigger):

- `agent-browser` — når WebFetch ikke kan opløse JS-renderede sider eller søge-flows på Datatilsynet/EDPB/leverandør-portaler. Logges separat fra Frederiks pen-test-brug.

Ingen ændringer foreslået for `yaml`.

## Mandat-impact

Tilføjelse af `deep-research` og `agent-browser` til mit standardværktøj er en **tools-ændring** og falder under mandat-stabilitet (SOC 2 CC8.1 / ISO 27001 A.8.32). Jeg foreslår derfor:

1. Stefan eskalerer fælles til Mads (security/SOD) + Poul eller Trine (4-eyes for DPO-mandatændring — jeg reviewer ikke selv).
2. Mads vurderer om `agent-browser` introducerer SOD-konflikt (Frederiks pen-test vs. min research).
3. FGD godkender før Camilla committer ændring til `.claude/agents/dorthe.md`.

Indtil formel proces er gennemført, bruger jeg `deep-research` og `agent-browser` ad-hoc når Stefan eksplicit beder mig om det i en opgave — ikke som default.

## Sammenfatning

| Skill | Verdict | Kategori |
|---|---|---|
| `deep-research` | Stærkt ja | Must-use (kræver mandat-opdatering) |
| `context7` | Ja | Must-use (uændret) |
| `agent-browser` | Betinget ja | Situational (kræver mandat-opdatering + SOD-vurdering) |
| `yaml` | Nej | Ikke relevant |

— Dorthe
