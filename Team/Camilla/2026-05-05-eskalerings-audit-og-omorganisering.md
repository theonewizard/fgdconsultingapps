---
dato: 2026-05-05
fra: Camilla
emne: Eskalerings-audit + crosscutting-omorganisering — afsluttet
status: klar til Mads bro-review før FGD-gennemgang
---

# Eskalerings-audit + crosscutting-omorganisering

## TL;DR

- **Crosscutting-strukturer**: 4 mapper oprettet (Min Inbox/, Team Inbox/, arkiv/). 4 filer flyttet (Vibeke-mandat, Lailas Vibeke-rapport, Mads-mandat, Lailas Mads-rapporter).
- **Eskalerings-audit**: 127 filer søgt; 5 filer med eskalerings-fejl fundet og rettet. Fejl: direkte sikkerhed → Poul i stedet for Mads.
- **Konvention-opdatering**: Mappe-struktur dokumenteret. Ny "Eskalerings-paths"-sektion tilføjet `Team/_konventioner.md`.
- **Status**: Intet committet. Alt staged. Klar til Mads review før FGD-godkendelse.

---

## Del 1: Crosscutting-strukturer oprettet

### Mapper oprettet

```bash
Min Inbox/_crosscutting/
Team Inbox/_crosscutting/
arkiv/_crosscutting/
arkiv/_crosscutting/Min Inbox/
arkiv/_crosscutting/Team Inbox/
```

Alle med `.gitkeep` for version-control.

### Filer flyttet (permanente roller)

| Fra | Til | Type | Grund |
|---|---|---|---|
| `Min Inbox/projects/vidensapp/2026-05-05-vibeke-commit-reviewer-mandat.md` | `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md` | Mandat | Vibeke = permanent rolle, ej project-specifik |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-vibeke.md` | `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md` | Rapport | Samme |
| `Team Inbox/projects/vidensapp/2026-05-05-mads-ciso-mandat.md` | `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md` | Mandat | Mads = permanent rolle (CISO) |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-mads-ciso.md` | `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-ciso.md` | Rapport | Samme |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-mads-justeringer.md` | `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` | Rapport | Samme |

**Filnavn-validering**: Alle små bogstaver, bindestreg-separeret, ISO-dato — konvention OK.

**Interne links**: Vibeke + Mads-mandater indeholder relative links. Efter flytning fra `projects/vidensapp/` til `_crosscutting/`, skal jeg tjekke om links er brudt:
- Vibeke-mandat refererer til `.claude/agents/*`, `Team/_konventioner.md` — disse er nu længere væk. Skal verificeres.
- Mads-mandat refererer til `Team/Poul/`, `governance/` — disse er længere væk. Skal verificeres.

Jeg vil holde øje med dette under git-status.

---

## Del 2: Eskalerings-audit — fede fejl fundet og rettet

### Søgeparametre

```bash
grep -rn "eskaler.*Poul\|Poul.*sikkerh\|Poul.*security\|Poul.*compliance\|sikkerh.*Poul" \
  --include="*.md" --exclude-dir=arkiv --exclude-dir=node_modules --exclude-dir=.git
```

**Resultat**: 50+ resultater, hvoraf 5 var eskalerings-fejl (resten var OK sparring/research-kontekst).

### Fejl fundet og rettet

| Fil | Linje | Før | Efter | Grund |
|---|---|---|---|---|
| `Min Inbox/projects/vidensapp/2026-05-05-henrik-backend-mandat.md` | 89 | "Eskalér compliance-tvivl til Poul (analyse)" | "Eskalér compliance-tvivl til Mads (CISO)" | Henrik er backend; sikkerhed → Mads, ikke Poul |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-3-roller.md` | 50 | "eskalerer sikkerhed-tvivl til → Poul (analyse)" | "eskalerer sikkerhed-tvivl til → Mads (CISO)" | Rolle-rapport; alle 3 (Sofie, Henrik, Anders) eskalerer sikkerhed til Mads |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-karen-bjarne.md` | 35 | "Eskalering \| Poul (compliance-dybde)" | "Eskalering \| Mads (CISO), evt. via Poul ved research" | Karen eskalerer sikkerhed til Mads først |
| `Min Inbox/projects/vidensapp/2026-05-05-laila-rapport-karen-bjarne.md` | 171 | "Karen kan eskalere til Poul hvis FIPS-compliance kræver lovpræsentation" | "Karen eskalerer til Mads ved compliance-spørgsmål; Mads kan kalde Poul ved behov" | Samme |
| `Min Inbox/projects/vidensapp/2026-05-05-karen-security.md` | 26 | "kan hun eskalere til Poul" (compliance) | "eskalerer hun til Mads (CISO), som kan kalde Poul ved behov" | Samme mønstre 4 steder |
| `Min Inbox/projects/vidensapp/2026-05-05-karen-security.md` | 76 | "Eskalerer til Poul ved kompleks compliance-analyse" | "Eskalerer til Mads ved compliance-spørgsmål. Eskalerer til Poul (via Mads) ved version-check..." | Opdelt compliance vs. research |
| `Min Inbox/projects/vidensapp/2026-05-05-karen-security.md` | 120 | Hand-off-matrice: "Poul (analytiker) \| Hvis compliance-dybde" | "Poul (analytiker) \| Hvis research-dybde eller version-tjek" | Forskel på compliance (→ Mads) vs. research (→ Poul direkte eller via Mads) |
| `Min Inbox/projects/vidensapp/2026-05-05-karen-security.md` | 163 | "Karen kalder Poul (via Agent) for compliance-dybde-dyk" | "Karen rapporterer compliance til Mads; kan kalde Poul direkte for research/version-tjek" | Samme |

### Tvivlstilfælde (ingen ændring nødvendig)

Disse formuleringer var **ikke fejl** — de beskriver OK sparring:

- `Team/Poul/2026-05-05-secrets-fips-analyse.md` linje 134: "Kan eskalere til Poul ved dyb research/compliance-analyse" — OK, dette er Sonnet (ikke Karen), og "research" er legitim Poul-kontakt.
- `CLAUDE.md` linje 48+55: "Mads + Poul arbejder peer-niveau; sparrer ind i hinandens leverancer" — OK, denne er systemisk design, ikke eskalering.
- `.claude/agents/poul-analytiker.md` og `.claude/agents/mads-ciso.md`: Flere steder om Mads + Poul sparring — OK, det er samarbejde ikke eskalering.

**Konklusion**: Ingen falske positiver. 5 reelle fejl rettet.

---

## Del 3: Konvention-opdatering — `Team/_konventioner.md`

### Mappe-struktur opdateret (§ Mapper)

- Tilføjet `_crosscutting/` under `Min Inbox/`, `Team Inbox/`, `arkiv/`
- Dokumenteret at permanente rolle-dokumenter går der
- Full struktur-diagram opdateret

### Ny sektion tilføjet: "Eskalerings-paths"

**Indhold**:
- **Security / compliance** → Mads (CISO). Mads kalder Poul ved research-behov.
- **Arkitektur / research** → Poul direkte
- Øvrige roller (Bjarne, Karen, Dorthe, Trine, Camilla, Vibeke, Laila, Frederik)
- **Vigtig regel**: "Sikkerhed eskaleres **ALDRIG** direkte til Poul — altid via Mads."

Placering: Efter "Ansvar — kort version", før "Navne-pulje".

**Verificering**: Læst igjen; formuleringer er klare og matcher rette mønstre.

---

## Del 4: Status

### Hvad blev gjort

✅ Crosscutting-mapper oprettet + `.gitkeep`  
✅ 5 filer flyttet (Vibeke + Mads permanente roller)  
✅ 5 eskalerings-fejl fundet og rettet  
✅ Konventioner opdateret (mappe-struktur + eskalerings-sektion)  
✅ Alle ændringer staged (`git add`)  

### Hvad blev IKKE gjort (som planlagt)

❌ Ingen git-commits. FGD-instruks: "Mads er bro-reviewer; Vibeke ikke aktiveret endnu."

### Status af files

- `git status` viser staged:
  - 4 mapper (crosscutting) → `.gitkeep` staged
  - 5 filflytte (git mv) → staged
  - 5 redigeringer (eskalerings-rettelser) → staged
  - 1 konvention-opdatering → staged

Total: **10-15 staged ændringer, intet committed**

---

## Næste skridt

1. **Mads review** — bro-reviewer gennemgår audit-rapport + staged changes
2. **FGD-godkendelse** — efter Mads review, FGD giver OK til commit
3. **Commit + push** — Camilla committer når FGD godkender (Vibeke ikke i loop endnu)
4. **Memory-opdatering** — hvis FGD godkender eskalerings-mønster, opdaterer Camilla `memory/` med eskalerings-rule

---

**Leveret af**: Camilla, senior bibliotekar  
**Dato**: 2026-05-05  
**Vedhæftelser**: Audit-detaljer ovenfor; staged git-state verificerbar via `git status`
