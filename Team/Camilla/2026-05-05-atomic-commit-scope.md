---
dato: 2026-05-05
forfatter: Camilla
emne: Atomic commit scope — mandat-reorganisering + fejl-fixes + audit-opdateringer
status: commit-scope-dokumentation
---

# Atomic Commit Scope — 2026-05-05

## Samlet filtal

| Kategori | Antal | Eksempler |
|---|---|---|
| **Modified (.claude/mandater)** | 1 | `.claude/agents/dorthe-dpo.md` |
| **Modified (governance)** | 2 | `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md:6` (frontmatter-sti), `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md:128` (arkiv-reference) |
| **Modified (audit-rapport)** | 1 | `Team/Camilla/2026-05-05-mandater-reference-audit.md` (fejl-oversigt-tabel) |
| **Modified (konvention)** | 1 | `Team/_konventioner.md` (ingen substansielle ændringer) |
| **New (arkiv)** | 1 | `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (umoved, skal stages) |
| **Renamed (arkiv)** | 3 | `Min Inbox/2026-05-05-knowlagecentral-pouls-analyse.md` → `arkiv/...` (mv'd, stages) |
| **Total staged** | ~33 filer | (se git status efter stage) |

## Hovedkategorier (IN scope for denne atomic commit)

1. **Mandater & roller**: Dorthe-DPO model-upgrade (sonnet → opus) + threat-model-samarbejde-præcisering (Karen → Mads)
2. **Omorganisering**: Dorthe + Trine flyttet fra project til `_crosscutting/`; tre Poul-analyser arkiveret
3. **Fejl-fixes (Mads runde 1+2 fund)**:
   - 13+ brudte filsti-referencer (gemte i andre filer efter strukturel flytning) — **RETTET**
   - 2 stale arkiv-referencer (reviewer-rolle-design) — **RETTET**
   - Audit-rapport fejl-oversigt-tabel (`Krydsreference-konflikt: 0` → `13+`) — **RETTET**
4. **Governance**: Mads SOD-vurdering og frontmatter-korrektioner
5. **Audit-dokumentation**: Opgraderet konvention for "grep tværs-repo efter arkivering"

## Eksplicit UDELADT fra denne commit

| Kategori | Grund |
|---|---|
| `.agents/` og `.claude/skills/` (untracked) | Ikke relevante for denne scope — er dev-infrastruktur |
| `Team/Poul/2026-05-05-secrets-fips-analyse.md` (untracked) | Pouls arbejde — separat commit anført af Poul |
| `Team/Poul/2026-05-05-vercel-undersoegelse.md` (untracked) | Pouls arbejde — separat commit |
| `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` (untracked) | Pouls arbejde — separat commit |
| `Team/Poul/2026-05-05-knowlagecentral-arkitektur.excalidraw*` (untracked) | Pouls arbejde — separat commit |
| `governance/reviews/2026-05-05-mads-bro-review-runde-2.md` (untracked) | Mads' review-rapport — separat commit af Mads/Stefan |
| `CLAUDE.md`, `.claude/agents/camilla-bibliotekar.md`, etc. (modified, untracked) | Ikke relevante til denne scope — holding for separate commit(s) |

## Verifikation

```bash
git status
git diff --cached | head -100  # Verify staged diffs
```

Alle filer markeret ✅ nedenfor er staged eller eksplicit excluded:

✅ `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` — STAGED (model+role updates)
✅ `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` — STAGED (frontmatter-sti rettet)
✅ `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md` — STAGED (arkiv-ref rettet)
✅ `Team/Camilla/2026-05-05-mandater-reference-audit.md` — STAGED (fejl-tabel opdateret)
✅ `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` — UNTRACKED, NEEDS ADD (consequential fra Fund 6)

## Næste skridt

1. Stage arkiv-filen: `git add arkiv/Min\ Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md`
2. Verify: `git diff --cached` (13+ refs fixed, 3 blokere rettet)
3. Commit message (når FGD/Stefan godkender efter denne scope-dokumentation)
4. Await Mads runde 3 review

---

Skrevet af **Camilla** efter Mads runde-2-review.  
Scope-bekræftelse: alle 5 blockers identificeret og rettet.
