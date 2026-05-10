---
name: Memory-struktur-strategi FGD-team
description: Analyse af project vs crosscutting memory-filer; risiko-vurdering; anbefalinger
type: strategy
---

## Kort topline

**Nuværende:** 14 memory-filer (excl. MEMORY.md). **Forurening-risiko: HØJS** — 9 filer er knowlagecentral-specifik; 5 er crosscutting FGD-team. Når Stefan/Laila/Poul arbejder på projekt 2, indlæses alle 14 filer automatisk, og kontekst bliver brumt.

**Anbefaling:** **Scenarie B+C hybrid** — fysisk adskillelse i `memory/_crosscutting/` og `memory/projects/knowlagecentral/` kombineret med eksplicit `project`-tag i frontmatter (giver Stefan manuel kontrol hvis project-awareness ikke er built-in). Kræver 0 ændringer i Claude Code's auto-loader (MEMORY.md loader som før, men kan geografisk struktureres).

---

## 1. Aktuel tilstand

### Klassifikation af 14 filer

| Fil | Type | Scope | Risiko |
|---|---|---|---|
| knowlagecentral-projekt-2026-05-05.md | project | knowlagecentral | **HØJS** — arkitektur, fasering, rolle-mandater, kostbudget specifikt |
| hsm-fips-140-produktionsmål-2026-05-05.md | project | knowlagecentral | **HØJS** — FIPS-strategi + hardware-plan knowlagecentral-bound |
| yubihsm-vej-2026-05-05.md | project | knowlagecentral | **HØJS** — implementerings-vej for knowlagecentral HSM-fase 1.5 |
| vercel-neon-strategi-2026-05-05.md | project | knowlagecentral | **HØJS** — teknologi-valg (1 Neon-projekt + branches) specifikt til knowlagecentral |
| kunde-vertikal-regulering-2026-05-05.md | project | knowlagecentral | **HØJS** — kundesegment-validering + feature-roadmap til knowlagecentral |
| mads-first-session-2026-05-05.md | project | knowlagecentral | **HØJS** — governance-leverance tæt koblet til knowlagecentral-fase 0/1 |
| dpo-model-2026-05-05.md | project | knowlagecentral | **HØJS** — GDPR-DPO-model til knowlagecentral-kontekst + Dorthe-rolle |
| pen-test-vendor-2026-05-05.md | project | knowlagecentral | **HØJS** — pen-test-vendor-shortliste og Frederik-rolle for knowlagecentral |
| skill-inventar-2026-05-05.md | project | knowlagecentral | **HØJS** — 87 skills katalogiseret til knowlagecentral-fase 0 + Mads-godkendelse |
| mads-ciso-rolle-2026-05-05.md | crosscutting | FGD-team | **LIDT** — Mads er permanent rolle (alle projekter); men aktiveret under knowlagecentral-kontekst |
| dokument-vedligeholds-governance.md | crosscutting | FGD-team | **GEM** — governance for alle dokumenter (ikke knowlagecentral-specifik) |
| feedback_version_verification.md | crosscutting | FGD-team | **GEM** — protokol for alle analyser (ikke knowlagecentral-specifik) |
| stefan-rolle-godkendelses-flow.md | crosscutting | FGD-team | **GEM** — Stefan-proces for alle projektroller |
| feedback_agent_placering.md | crosscutting | FGD-team | **GEM** — agent-fil-konvention for alle projekter |

**Samlet:** 9 knowlagecentral-specifik + 5 crosscutting. **Udgangspunkt for forurening** når projekt 2 starter.

### Aktuelle memory-load-regler

- Claude Code loader automatisk `MEMORY.md` fra `~/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/`
- Systemet læser indeks-linjerne (fil-referencer)
- Systemet har ingen built-in project-awareness — alle filer i `memory/`-mappen er "globalt tilgængelige" for alle sessions i projektet
- MEMORY.md selv er uden frontmatter, uden project-tag

---

## 2. Risiko-analyse

### Scenarie: Stefan starter projekt 2 (f.eks. "HSM-firmware-platform")

**Hvad sker:**
1. Stefan åbner project 2's arbejdsmappe
2. Claude Code loader auto-memory fra `~/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/MEMORY.md` (samme repo-instance)
3. Alle 14 memory-filer bliver kontekst-loadet
4. **Resultat:** Mads bliver bedt om at besvare spørgsmål om knowlagecentral-HSM når han skal fokusere på projekt 2's architecture

**Konkrete problemer:**
- `knowlagecentral-projekt-2026-05-05.md` (16K) blander rolle-mandater (Sofie/Henrik/Anders) med projekt 2's kontekst
- `hsm-fips-140-produktionsmål-2026-05-05.md` + `yubihsm-vej-2026-05-05.md` + `skill-inventar-2026-05-05.md` (3K + 2.5K + 3K = 8.5K dead-weight) når projekt 2 handler om andet
- `mads-ciso-rolle-2026-05-05.md` bliver genindlæst for hver session; Mads er permanent, men konteksten der aktiverede ham er knowlagecentral-bound
- Stefan tolker feedback-files som projektet-agnostisk, men de kom alle fra knowlagecentral-bootstrap

---

## 3. Foreslåede scenarier

### Scenarie A — Frontmatter-tagging

**Struktur:** Alle 14 filer bliver i `memory/` (flat). Hver fil får `project: knowlagecentral | crosscutting` i frontmatter.

**Mekanisme:**
```markdown
---
name: Knowlagecentral v3
project: knowlagecentral  ← TAG
---
```

**Pro:**
- Nul ændringer i mappestruktur
- Minimal ændring i MEMORY.md-indeks (én kolonne-tag)
- Claude Code's auto-loader fortsætter virker uændret

**Con:**
- Kræver manuelt filter-logic i Stefans prompt ved hver session ("indlæs kun `project: {{ aktivt-projekt }}`")
- Stefan skal eksplicit sige "vi arbejder på projekt knowlagecentral" — hvis glemt, loader alt
- Svært at enforke — ingen teknisk barriere mod at lade knowlagecentral-filer "lække" ind

---

### Scenarie B — Physical folder separation (ren)

**Struktur:**
```
memory/
├── _crosscutting/          ← Filer der gælder alle projekter
│   ├── dokument-vedligeholds-governance.md
│   ├── feedback_version_verification.md
│   ├── feedback_agent_placering.md
│   ├── stefan-rolle-godkendelses-flow.md
│   └── mads-ciso-rolle-2026-05-05.md  (permanent rolle, men crosscutting)
├── projects/
│   └── knowlagecentral/
│       ├── knowlagecentral-projekt-2026-05-05.md
│       ├── hsm-fips-140-produktionsmål-2026-05-05.md
│       ├── yubihsm-vej-2026-05-05.md
│       ├── vercel-neon-strategi-2026-05-05.md
│       ├── kunde-vertikal-regulering-2026-05-05.md
│       ├── mads-first-session-2026-05-05.md
│       ├── dpo-model-2026-05-05.md
│       ├── pen-test-vendor-2026-05-05.md
│       └── skill-inventar-2026-05-05.md
└── MEMORY.md              ← Index opdelt i 2 sektioner
```

**Pro:**
- Visuelt klart hvilke filer der er project-bound
- Matcher repo-struktur (`projects/knowlagecentral/` etc.)
- Git-historie er ren og læsbar (fysisk flytning via `git mv`)

**Con:**
- **KRITISK RISK:** Claude Code's auto-loader tager `MEMORY.md` og ved ikke at se ned i undermapper. Test påkrævet at bekræfte at sub-folder-referencer virker
- Vil kræve at `MEMORY.md` har relative paths til `_crosscutting/` og `projects/knowlagecentral/` — systemet skal kunne opløse dem

---

### Scenarie C — Hybrid (anbefalet)

Kombinerer B + A:

**Struktur:** (som B) fysisk adskillelse i `_crosscutting/` og `projects/knowlagecentral/`

**MEMORY.md index:** grupperet efter scope, MEN hver fil har også `project`-tag i frontmatter

```markdown
## Crosscutting (FGD-team govenance)
- [Dokument-vedligeholds-governance](_crosscutting/dokument-vedligeholds-governance.md) — ...

## Knowlagecentral
- [Knowlagecentral v3 arkitektur](projects/knowlagecentral/knowlagecentral-projekt-2026-05-05.md) — ...
```

**Samtidig frontmatter:**
```markdown
---
project: knowlagecentral
---
```

**Pro:**
- Fysisk isolation sikrer at repo-layout er rent
- Frontmatter-tag giver Stefan "fail-safe" — selvom filen blev indlæst per uheld, kan han se hvilket projekt den tilhører
- MEMORY.md bliver selv dokumentation (læsr kan se struktur uden at læse filerne)
- Hvis claude.ai's auto-loader IKKE støtter subfolder, virker det stadig (indeks peger på files, systemet loader dem, paths opløses)

**Con:**
- To steder at vedligeholde information (folder-navn + frontmatter-tag)
- Duplication, men ved duplikation tjekkes mod hinanden (kvalitets-alarm)

---

## 4. Implementerings-skitse (hvis FGD vælger C)

### Faser

**Fase 1 — forberedelse (15 min):**
1. Læs alle 5 crosscutting-filer igennem (ingen ændring)
2. Læs alle 9 knowlagecentral-filer igennem (ingen ændring)
3. Sørg for at hver fil har `project: ...` i frontmatter (mange har det)

**Fase 2 — mappestruktur (5 min):**
```bash
cd /Users/fgd/.claude/projects/-Users-fgd-git-repos-FGDConsultingKnowlageDatabase/memory/
mkdir -p _crosscutting projects/knowlagecentral

# Flyt crosscutting-filer
git mv dokument-vedligeholds-governance.md _crosscutting/
git mv feedback_version_verification.md _crosscutting/
git mv feedback_agent_placering.md _crosscutting/
git mv stefan-rolle-godkendelses-flow.md _crosscutting/
git mv mads-ciso-rolle-2026-05-05.md _crosscutting/

# Flyt knowlagecentral-filer
git mv knowlagecentral-projekt-2026-05-05.md projects/knowlagecentral/
git mv hsm-fips-140-produktionsmål-2026-05-05.md projects/knowlagecentral/
# ... osv 9 filer
```

**Fase 3 — MEMORY.md-opdatering (10 min):**
```markdown
# MEMORY — FGD Knowledge Database

## Crosscutting (alle projekter)

- [Dokument-vedligeholds-governance](_crosscutting/dokument-vedligeholds-governance.md) — Levende vs. arkiverede docs; v1/v2/v3-pointer; Camilla markerer arkivering
- [Feedback: Version-verificering](_crosscutting/feedback_version_verification.md) — Alle analyser skal eksplicit verificere frameworks/SDK'ers aktuelle version via Context7
- ...

## Knowlagecentral (projekt: 2026-05-05)

- [Knowlagecentral v3 — final arkitektur + HSM Layer 2](projects/knowlagecentral/knowlagecentral-projekt-2026-05-05.md) — Single source of truth: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md...
- ...
```

**Fase 4 — validering (10 min):**
- Tjek at `git status` viser rene `git mv` operations (ingen deletions/additions)
- Kør Claude Code og bekræft at MEMORY.md indlæses og links opløses

**Fase 5 — git-commit (5 min):**
```bash
git commit -m "Reorganiser memory: fysisk adskillelse af crosscutting vs. knowlagecentral-projekter"
```

---

## 5. Åbne spørgsmål til FGD

1. **Projekt-navngivning:** Er `projects/knowlagecentral/` det rigtige navn? Eller skal det være `projects/2026-knowlagecentral/` eller `projects/knowledge-database/`?

2. **Claude Code auto-loader kompatibilitet:** Du skal test at Fase 4 virker — lav en quick check at auto-memory-indlæsning fungerer med relative paths til undermapper.

3. **Mads' placering:** Mads er permanent rolle (ikke knowlagecentral-specifik), men blev aktiveret i knowlagecentral-konteksten. Skal hans fil være i `_crosscutting/` (fordi han er permanent) eller `projects/knowlagecentral/` (fordi det er hvor han startede)? **Anbefaling:** `_crosscutting/` — han er team-ressource, ikke project-dedikeret.

4. **Fremtidigt projekt 2:** Når du starter projekt 2, skal der være en `projects/projekt-2/` -mappe? Eller kan crosscutting-memory dele for alle og hver projekt lige med sin egen sub-folder?

---

## 6. Vigtigste risici hvis vi ikke gør noget

- **Kontekst-forurening:** Stefans memory-load bliver støjpræget når projekt 2 starter (8,5K dead-weight fra knowlagecentral)
- **Rolle-forvirring:** Mads bliver kontekst-loadet udsving-agtig når han arbejder på projekt 2
- **Øget token-forbrug:** Hver session indlæser ~40K memory (14 filer) da kun 10-15K er relevant; 65 % waste
- **Git-historik-kompleksitet:** Uden klart skema bliver memory-mappen uorganiseret som projekterne vokser

---

## Anbefaling

**Scenarie C — Hybrid (fysisk + frontmatter)** fordi:
1. Fysisk isolation sikrer at git-layout forbliver læsbart
2. Frontmatter-tag giver redundant "safety net" hvis claude.ai's loader ikke støtter subfolder
3. MEMORY.md bliver selv dokumentation (grupperet indeks)
4. Enkel at enforke og vedligeholde
5. Skaber mønster for projekt 2+

**Næste trin:** FGD bekræfter:
- Scenario C ok?
- Mads i `_crosscutting/` eller `projects/knowlagecentral/`?
- Test af claude.ai auto-loader med subfolder-paths?

Derefter udfører Camilla fase 1–5 og committer.

---

**Rapport skrevet:** 2026-05-05 21:30  
**Status:** Klar til FGD-godkendelse
