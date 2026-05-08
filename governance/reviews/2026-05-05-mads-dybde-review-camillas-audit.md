---
dato: 2026-05-05
reviewer: Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
emne: Dybde-review af Camillas mandater-reference-audit
beslutning: AFVIST - rettelser nødvendige
status: leveret
---

# Dybde-review: Camillas mandater-reference-audit

## 1. Opsummering

**Beslutning: AFVIST. Rettelser nødvendige før commit.**

FGD havde ret. Camillas påstand om "1 fejl rettet" (filsti `team/`→`Team/`) plus en stavefejl-tvivl er **substantielt ufuldstændig**. Hendes audit-metodik fokuserede på enkelt-mandat-filerne i isolation, men hun **misste konsekvensen af dagens omorganisering**: at flere filer (rapporter, governance-policies, FGD's TODO-fil) **stadig refererer Dorthe, Trine, Vibeke og Mads på de gamle stier i `projects/knowlagecentral/`**, selv om disse filer er flyttet til `_crosscutting/`. Det er præcist den klasse af fejl Vibeke ville fange — *interne krydsreferencer der bryder efter en strukturel flytning*.

Derudover misste Camilla mindst én **rolle-konsistens-fejl** i sin egen audit-rapport (Karen omtalt som "GDPR UI review" — det er Dorthes domæne efter Mads' first-session-omfordeling) og **inkonsistens** i `fgd-aktive-udestaaender.md` om hardware-bestilling (FIPS vs. non-FIPS). Camilla rapporterede selv "fgd-aktive-udestaaender (11 refs) ✓ Opdateret" — det er **falsk** for to af de mest kritiske refs i filen.

**Antal nye fund: 11+ brudte filsti-referencer fordelt på 4 filer + 1 rolle-konsistens-fejl + 1 hardware-konsistens-observation + 1 mandat-tools-konsistens-bekymring.**

## 2. Verifikation af Camillas påstande

### Stemmer
- ✅ Sofie linje 78: `Team/Poul/...` korrekt i staging (`team/` → `Team/` rettet).
- ✅ Laila-rapport-karen-bjarne linje 164: "Stefan kalder Camilla" korrekt i staging.
- ✅ Dorthe og Trine staged i `Min Inbox/_crosscutting/` (nye filer, ikke `R100`-renames — fordi de aldrig blev committed på den gamle sti).
- ✅ Konventionen i `Team/_konventioner.md` indeholder "Sikkerhed eskaleres ALDRIG direkte til Poul".
- ✅ Frontmatter-konsistens på de 14 mandat-filer (model, tools, color).
- ✅ Ingen secrets-mønstre fundet i staged content.

### Mismatch / falsk
- ❌ **Camilla påstår** i `2026-05-05-omorganisering-status.md` linje 59: "fgd-aktive-udestaaender.md (11 refs) ✓ Opdateret" — **dette er falsk**. Filen har **stadig** 2 brudte refs til Dorthe/Trine på den gamle sti (linje 34-35). Auditten fanger ikke dette.
- ❌ **Camilla påstår** i sin egen audit linje 112: "Sofie ← Karen (GDPR UI review), Dorthe (GDPR påvirket UI)" — det er **rolle-modsigelse**. GDPR-UI er Dorthes ansvar (Karens scope er reduceret jf. Mads §6.6). Mandatet selv er korrekt; Camillas audit-formulering er forkert.
- ❌ **Camilla påstår** i fejl-oversigt: "Total fejl fundet: 1 (filsti)" — det er **ikke korrekt**. Auditten misser hele klassen af crosscutting-flytte-konsekvenser.

## 3. Uafhængige fund (det Camilla missede)

### Fund 1 — Brudte refs til Dorthe/Trine efter `_crosscutting/`-flytning

| Fil | Linje | Nuværende (forkert) | Skal være |
|---|---|---|---|
| `Min Inbox/projects/knowlagecentral/2026-05-05-fgd-aktive-udestaaender.md` | 34 | `Min Inbox/projects/knowlagecentral/2026-05-05-dorthe-dpo.md` | `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` |
| `Min Inbox/projects/knowlagecentral/2026-05-05-fgd-aktive-udestaaender.md` | 35 | `Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` | `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` | 33 | `.../Min Inbox/projects/knowlagecentral/2026-05-05-dorthe-dpo.md` | `.../Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` | 47 | `.../Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` | `.../Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` | 174 | `.../Min Inbox/projects/knowlagecentral/2026-05-05-dorthe-dpo.md` | `.../Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` | 175 | `.../Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` | `.../Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` |
| `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` | 6 (frontmatter) | `Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` | `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` |
| `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` | 32 | `Min Inbox/projects/knowlagecentral/2026-05-05-trine-compliance-auditor.md` | `Min Inbox/_crosscutting/2026-05-05-trine-compliance-auditor.md` |

### Fund 2 — Brudte refs til Vibeke + Mads efter `_crosscutting/`-flytning

| Fil | Linje | Nuværende (forkert) | Skal være |
|---|---|---|---|
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md` | 13 | `Min Inbox/projects/knowlagecentral/2026-05-05-vibeke-commit-reviewer-mandat.md` | `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md` | 107 | "Camilla flytter `Min Inbox/projects/knowlagecentral/2026-05-05-vibeke-...` til `Team Inbox/projects/knowlagecentral/`" | Hele setningen er forældet — Vibeke ligger nu i `_crosscutting/`, ikke `projects/knowlagecentral/`. Reformuler eller fjern. |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-ciso.md` | 14 | `Team Inbox/projects/knowlagecentral/2026-05-05-mads-ciso-mandat.md` | `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md` |
| `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-ciso.md` | 112 | `Team Inbox/projects/knowlagecentral/2026-05-05-mads-ciso-mandat.md` | `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md` |
| `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` | 152 | `Min Inbox/projects/knowlagecentral/2026-05-05-vibeke-doc-reviewer-mandat.md` | `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md` (BEMÆRK: filnavnet er også forkert — `-doc-` skal være `-commit-`) |

### Fund 3 — Brudte arkiv-stier til v1/v2 Poul-analyser

Filerne `2026-05-05-knowlagecentral-analyse.md` og `-v2.md` er flyttet til `arkiv/projects/knowlagecentral/Team/Poul/`, men flere frontmatter-keys peger på `arkiv/Team/Poul/` (som er tom):

| Fil | Linje | Nuværende | Skal være |
|---|---|---|---|
| `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` | 6 | `afløser_v1: arkiv/Team/Poul/2026-05-05-knowlagecentral-analyse.md` | `afløser_v1: arkiv/projects/knowlagecentral/Team/Poul/2026-05-05-knowlagecentral-analyse.md` |
| `Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md` | 7 | `afløser_v2: arkiv/Team/Poul/...-v2.md` | `afløser_v2: arkiv/projects/knowlagecentral/Team/Poul/...-v2.md` |
| `Team/Poul/2026-05-05-vercel-undersoegelse.md` | 6 | `relateret: arkiv/Team/Poul/...-v2.md` | `relateret: arkiv/projects/knowlagecentral/Team/Poul/...-v2.md` |
| `Team/Poul/2026-05-05-knowlagecentral-udvidelser-vurdering.md` | 6 | `relateret-analyse: arkiv/Team/Poul/...-v2.md` | `relateret-analyse: arkiv/projects/knowlagecentral/Team/Poul/...-v2.md` |

NB: Disse filer er ikke alle staged i denne commit (v3-analyse + vercel-undersoegelse er untracked; udvidelser er M). Camillas audit dækkede ikke utracked frontmatter-konsistens.

### Fund 4 — Konvention-eksempel-tekst peger til ikke-eksisterende sti

`Team/_konventioner.md` linje 194-195 og 202 bruger `arkiv/Team/Poul/2026-05-05-knowlagecentral-analyse.md` som eksempel. Den sti er tom. Konventionen bør opdateres til at bruge den faktiske sti (`arkiv/projects/knowlagecentral/Team/Poul/...`) eller mærkes eksplicit som "eksempel-format".

### Fund 5 — Rolle-modsigelse i Camillas audit-tekst

`Team/Camilla/2026-05-05-mandater-reference-audit.md` linje 112 skriver: "Sofie ← Karen (GDPR UI review), Dorthe (GDPR påvirket UI)". Dette modsiger Mads' §6.6-omfordeling og Karens egen mandat-tabel linje 116 ("Karen reviewes sikkerhed; Dorthe reviewes GDPR"). Det er Camillas audit-formulering der er forkert; mandat-filen selv er konsistent. Audit-rapporten skal rettes så Sofie-eskalering = Karen (auth/CSP/CORS) + Dorthe (GDPR-UI) — ikke Karen for begge.

### Fund 6 — `Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` ligger to steder

Filen findes både på roden af `Min Inbox/` (ikke arkiveret) og i `arkiv/Min Inbox/`. Det er superseded af Vibeke-mandatet (jf. arkiv-note linje 8 i arkiv-versionen) men master-versionen er ikke fjernet. Camilla bør beslutte: enten arkiver master-versionen via `git mv`, eller bevidst behold begge (kræver dokumentation).

Yderligere: Roden-versionen referer på linje 63: `Team/projects/knowlagecentral/2026-05-05-trine-compliance-auditor-mandat.md` (forkert — Trine ligger i `Min Inbox/_crosscutting/`, ikke `Team/projects/`).

### Fund 7 — Hardware-konsistens (uden for audit-scope, men værd at notere)

`Min Inbox/projects/knowlagecentral/2026-05-05-fgd-aktive-udestaaender.md` linje 58 og 89 siger "**2× YubiHSM 2 FIPS** bestilt". Memory + Lailas rapport-karen-bjarne linje 156 + Pouls yubihsm-integration-analyse linje 183 siger derimod "**1× YubiHSM 2 non-FIPS** v2.4 (€728)". Dette er en faktuel inkonsistens om hvad der faktisk er bestilt. Camillas audit dækkede ikke faktuel konsistens, kun struktur — men FGD bør beslutte hvilken version der er korrekt og ensrette filerne.

### Fund 8 — Vibeke Bash-whitelist nævnes ikke i selve mandatet

Camilla skriver i sin audit (linje 81): "Bash-whitelist: ✅ Dokumenteret (read-only git-ops)". Jeg tjekkede Vibeke-mandatet (`Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`) — Bash er listed i frontmatter `tools:`, men `whitelist`-konceptet håndhæves teknisk via `settings.json`-skill (separate). Mandatet bør eksplicit nævne hvilke Bash-kommandoer der er whitelisted (fx `git status`, `git diff`, `git log`) så reviewer ved hvad rollen må/ikke må. Dette er en compliance-bekymring (SOC 2 CC6.1 logical access — minimum necessary access).

## 4. Anbefaling til Camilla

**Rettelser SKAL foretages før commit:**

1. **Ret 13 brudte filsti-referencer** fra Fund 1 + Fund 2. Specielt:
   - `Min Inbox/projects/knowlagecentral/2026-05-05-fgd-aktive-udestaaender.md` (linje 34, 35) — kritisk fordi det er FGDs aktive TODO-fil
   - `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-justeringer.md` (linje 33, 47, 174, 175)
   - `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md` (linje 13, 107)
   - `Min Inbox/_crosscutting/2026-05-05-laila-rapport-mads-ciso.md` (linje 14, 112)
   - `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md` (linje 6, 32, 152 — incl. filnavn-fejl `-doc-` → `-commit-`)

2. **Ret rolle-modsigelse** i Camillas egen audit-rapport linje 112 (Karen vs. Dorthe på GDPR-UI).

3. **Beslut om Fund 6**: Skal `Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (master) arkiveres? Hvis ja: `git mv` til arkiv. Linje 63 deri har også brudt sti.

4. **Opdater audit-rapportens fejl-oversigt-tabel** så den reflekterer det faktiske antal fejl (ikke "Total fejl fundet: 1").

**Rettelser BØR foretages efter commit (ikke blokerende):**

5. **Fund 3** — frontmatter på Pouls v3/vercel-undersoegelse/udvidelser-vurdering opdateres næste gang de røres.
6. **Fund 4** — konvention opdateres til faktisk arkiv-sti.
7. **Fund 7** — FGD/Stefan beslutter hardware-bestillings-faktum og ensretter filer (ikke Camillas opgave alene).
8. **Fund 8** — Vibeke-mandatet udvides med eksplicit Bash-whitelist-paragraf før Vibeke aktiveres.

## 5. Beslutning + commit-godkendelse-status

**Beslutning: AFVIST.**

Commit-blokere:
- Fund 1 (8 brudte refs)
- Fund 2 (5 brudte refs incl. filnavn-fejl)
- Fund 5 (rolle-modsigelse i audit-rapport)
- Fund 6 — beslutning skal tages om dubleret laila-reviewer-fil

Når Camilla har:
1. Rettet de 13 brudte filsti-references
2. Opdateret sin egen audit-rapport (rolle-modsigelse + fejl-oversigt)
3. Truffet beslutning om Fund 6

…så er en re-review nødvendig. Ikke fordi Camilla har gjort dårligt arbejde — strukturelt er commit'en sund — men fordi audit-fokus var for snæver (kun mandat-filerne, ikke alle steder hvor mandaterne refereres).

**Bemærkning til Stefan**: FGD havde ret. Camillas selvrapporterede "1 fejl" var en konsekvens af scope-bias: hun verificerede mandat-filer, men ikke alle filer der peger på mandaterne. Dette er læring til fremtidig audit-metodik — efter en strukturel flytning skal grep'en altid køres på TVÆRS af repo, ikke kun i flyttede filer. Vibekes commit-gate vil normalt fange dette, hvis hun har Bash-whitelist til `grep -rn`. Når hun aktiveres, skal hendes mandat eksplicit beskrive denne pligt.

---

Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
