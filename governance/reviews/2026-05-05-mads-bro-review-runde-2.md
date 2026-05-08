---
dato: 2026-05-05
reviewer: Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
emne: Re-review (runde 2) af Camillas rettelser
beslutning: AFVIST
status: leveret
---

# Re-review (runde 2): Camillas rettelser

## 1. Opsummering

**Beslutning: AFVIST.** Camilla har rettet flertallet af de 13 brudte filsti-references, men har **misset 2 stale refs**, **ikke opdateret sin egen fejl-oversigt-tabel**, og **har uncommittede working-tree-ændringer i Dorthes mandat** der skal afklares før commit.

Hovedfremskridt: Fund 1+2 (de 13 brudte filsti-refs) er stort set rettet (~11 af 13). Fund 5 (rolle-modsigelse Karen vs. Dorthe på GDPR-UI) er **korrekt rettet** i audit-rapport linje 114. Fund 6 (dubleret laila-reviewer-fil) er løst — kun arkiv-versionen findes nu. Fund 7 (hardware-konsistens) er **korrekt håndteret** — aktive dokumenter siger "2× YubiHSM 2 FIPS"; Pouls historiske analyser bevarer "non-FIPS €728"-referencer som forventet.

Men der er **3 substantielle commit-blocker** tilbage.

## 2. Verifikation pr. spor

### Spor 1 — 13 brudte filsti-refs (Fund 1 + Fund 2)

| Fil/Linje | Fund 1+2 | Status |
|---|---|---|
| `fgd-aktive-udestaaender.md:34, 35` | dorthe/trine projects→_crosscutting | RETTET (linje 34-35 peger nu på `_crosscutting/`) |
| `laila-rapport-mads-justeringer.md:33, 47, 174, 175` | dorthe/trine projects→_crosscutting | RETTET |
| `laila-rapport-vibeke.md:13` | vibeke projects→_crosscutting | RETTET |
| `laila-rapport-vibeke.md:107` | forældet sætning | RETTET (sætningen er reformuleret/fjernet) |
| `laila-rapport-mads-ciso.md:14, 112` | mads-ciso projects→_crosscutting | RETTET (peger nu på `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md`) |
| `mads-sod-vurdering-reviewer-rolle.md:32` | trine projects→_crosscutting | RETTET |
| `mads-sod-vurdering-reviewer-rolle.md:152` | vibeke projects→_crosscutting + filnavn | RETTET (`-commit-` korrekt; sti `_crosscutting/`) |
| **`mads-sod-vurdering-reviewer-rolle.md:6` (frontmatter)** | laila-reviewer-fil flyttet til arkiv | **IKKE RETTET** — peger stadig på `Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (filen findes kun i `arkiv/Min Inbox/`) |

Hits fra `grep -rn 'projects/knowlagecentral/2026-05-05-(dorthe|trine|vibeke|mads-ciso)' --exclude-dir=arkiv` i ikke-arkiverede filer er nu kun i (a) review/audit-rapporterne selv som beskriver fejlene, og (b) `Team/Camilla/2026-05-05-eskalerings-audit-og-omorganisering.md` der dokumenterer flytningen i tabel-form. Begge er korrekte — de **beskriver** flytningen. Ingen aktive `[link](sti)`-referencer på de gamle stier.

`vibeke-doc-reviewer` findes stadig i `mads-sod-vurdering-reviewer-rolle.md:82` — men det er inde i et historisk frontmatter-forslag-blok (Mads' oprindelige forslag til Laila), så det er acceptabelt som historisk. Ikke en blocker.

### Spor 2 — Fund 5 rolle-modsigelse

Audit-rapport linje 114: nu korrekt **"Sofie ← Karen (auth/CSP/CORS), Dorthe (GDPR-UI)"**. RETTET.

### Spor 3 — Fund 6 dubleret fil

`Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (rod-versionen) er **væk** fra arbejdstreet. Kun `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` eksisterer. RETTET.

**Bemærkning**: Filen er *untracked* i `arkiv/Min Inbox/` (ikke endnu staged). Camilla skal `git add` den, ellers er ændringen ikke i commit'en.

### Spor 4 — Fund 7 hardware-konsistens

`fgd-aktive-udestaaender.md` linje 58, 89: nu "2× YubiHSM 2 FIPS bestilt" → matcher `laila-rapport-karen-bjarne.md:156` ("2× YubiHSM 2 FIPS"). **Konsistent.**

Pouls historiske analyser (`Team/Poul/2026-05-05-yubihsm-integration-analyse.md`, `knowlagecentral-analyse-v3.md`) bevarer korrekt "non-FIPS €728"-referencer som historisk dokumentation. **Korrekt håndteret.** RETTET.

### Spor 5 — Audit-rapportens fejl-oversigt ærlig?

**Executive Summary (linje 12-25)**: Korrekt opdateret. Anerkender 13+ brudte refs, scope-bias, læring om grep-tværs-repo. ✓

**Fejl-oversigt-tabel (linje 285-293)**: **IKKE OPDATERET.** Tabellen siger stadig:

| Fejl-type | Antal |
|---|---|
| Filsti-konvention (team/ vs Team/) | 1 |
| Stavefejl | 1 |
| Krydsreference-konflikt | **0** |

Det er **falsk** efter rettelsen. Tabellen burde reflektere at 13+ krydsreference-fejl blev fundet og rettet. Dette er præcist samme inkonsistens-mønster der gjorde at runde 1 blev afvist — Camilla rapporterer ærligt i én del af dokumentet og falsk i en anden. **COMMIT-BLOCKER.**

### Spor 6 — Uafhængig kontrol-grep

**Nyt fund 1**: `Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md:128` — **brudt sti**:
```
Lailas tidligere reviewer-design-udkast (`Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md`) arkiveres som superseded.
```
Filen findes nu kun i `arkiv/Min Inbox/...`. Stien peger på en ikke-eksisterende rod-version. Skal opdateres til `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` eller eksplicit "(arkiveret)".

**Nyt fund 2**: `governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md:6` (frontmatter) — **brudt sti** (samme problem):
```
relateret: Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md, ...
```

Begge fund er konsekvenser af samme arkivering — Camilla flyttede filen til `arkiv/` men opdaterede ikke de to filer der stadig peger på rod-stien. **Læring: arkiv-flytninger kræver samme grep-tværs-repo som strukturelle flytninger.**

### Spor 7 — git status / staged diff

Staged: 33 filer (23 nye + 5 renames + Konventioner M + 4 .gitkeep) — størrelses-niveau matcher Camillas rapport.

**Nyt fund 3 (alvorligt)**: `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` har **uncommittede working-tree-ændringer** der ikke er staged:

```
-Model-valg: sonnet
+Model-valg: opus
-Uafhængig vurdering...ikke fra sikkerheds-perspektiv (Karen gør det).
+Uafhængig vurdering...ikke fra sikkerheds-perspektiv.
-Samarbejder med Karen ved threat-models...
+Samarbejder med Mads ved threat-models...
```

Dette er substantielle ændringer — model-skift fra sonnet til opus, fjernet reference til Karen i hard rules, ændret peer-rolle fra Karen til Mads. Camilla har ikke nævnt disse ændringer i sin rapport. **Hvis hun committer nu, går staged version (sonnet/Karen) i Git, mens working tree (opus/Mads) afviger.**

Yderligere working-tree-ændringer (ikke staged):
- `.claude/agents/camilla-bibliotekar.md` (M)
- `.claude/agents/laila-hr.md` (M)
- `.claude/agents/poul-analytiker.md` (M)
- `CLAUDE.md` (M)
- `Team/Laila/README.md` (M)
- `Team/Poul/2026-05-05-knowlagecentral-udvidelser-vurdering.md` (M)
- 3 arkiverede filer (M)
- `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` (M)

Plus en lang liste untracked filer (governance/, Team/Poul/secrets-fips, yubihsm, vercel, knowlagecentral-analyse-v3 m.fl.).

Camilla skal **eksplicit beslutte** for hver af disse: skal de med i denne commit, holdes uden for, eller separates til efterfølgende commit? Hendes rapport siger ikke noget om det.

## 3. Manglende rettelser (commit-blocker)

1. **Audit-rapport linje 285-293 (fejl-oversigt-tabel)** — opdater tabellen så den reflekterer 13+ krydsreference-fejl rettet (i stedet for "Krydsreference-konflikt: 0").

2. **`governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md:6`** — frontmatter `relateret:`-key skal opdateres fra `Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` til `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (eller fjernes hvis ikke længere relevant).

3. **`Min Inbox/_crosscutting/2026-05-05-laila-rapport-vibeke.md:128`** — opdater arkiv-note til den faktiske arkiv-sti.

4. **Working-tree-divergens i `Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md`** — afklar med Stefan/FGD om Dorthes model skal være sonnet eller opus, og om peer-rolle skal være Karen eller Mads. Stage den korrekte version. Som det er nu vil staged != working.

5. **Untracked + uncommittede filer**: Camillas commit-rapport skal eksplicit liste hvad der er IN scope for denne atomic commit og hvad der holdes til senere. Specielt:
   - `arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md` (untracked) — denne **skal** med, da den er konsekvensen af Fund 6-rettelsen.
   - `governance/reviews/` (untracked, mit eget arbejde) — separat commit.
   - `Team/Poul/secrets-fips-analyse.md`, `yubihsm-integration-analyse.md`, `knowlagecentral-analyse-v3.md` osv. (untracked) — Pouls arbejde, sandsynligvis separat commit.

## 4. Nye fund i runde 2

- **Fund N1**: `laila-rapport-vibeke.md:128` — brudt arkiv-sti (rod-version)
- **Fund N2**: `mads-sod-vurdering-reviewer-rolle.md:6` (frontmatter) — brudt arkiv-sti (rod-version)
- **Fund N3** (alvorlig): Working-tree-divergens i `dorthe-dpo.md` med substantielle indholdsmæssige ændringer (model + peer-rolle) som ikke er staged og ikke er nævnt i rapport

Fund N1 + N2 er præcis samme klasse fejl som runde 1: en strukturel flytning (denne gang arkivering, ikke `_crosscutting/`-flytning), der efterlader stale refs i andre filer. Camilla missede igen at køre `grep -rn` tværs-repo efter flytningen.

## 5. Beslutning

**AFVIST.** 5 rettelser nødvendige før commit:

1. Opdater fejl-oversigt-tabel i audit-rapport (Spor 5)
2. Ret `mads-sod-vurdering-reviewer-rolle.md:6` frontmatter (Fund N2)
3. Ret `laila-rapport-vibeke.md:128` arkiv-note (Fund N1)
4. Afklar dorthe-dpo.md staged vs working tree (Fund N3) — model + peer-rolle skal med eller uden ind i staging, men skal være konsistent
5. Eksplicit dokumentér scope for atomic commit (hvad er in/out)

Når disse 5 er på plads + `git add arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md`: **runde 3 review.**

**Bemærkning til Stefan**: Camilla har gjort ægte fremskridt på de 13 brudte refs og på rolle-modsigelsen. Men hun har gentaget samme metode-fejl: rettet det Mads pegede på, uden at køre uafhængig grep-tværs-repo for at fange den klasse af fejl der er blottet. Dette mønster er præcist hvorfor Vibeke er nødvendig — en uafhængig commit-gate der hver gang kører grep-tværs-repo for stale refs efter enhver flytning eller arkivering. Indtil Vibeke er live, anbefaler jeg at Camilla **eksplicit dokumenterer i hver commit-rapport** hvilke greps hun har kørt og med hvilke patterns.

Yderligere observation: Working-tree-divergens i `dorthe-dpo.md` er en **NY KLASSE** af fejl jeg ikke fangede i runde 1, fordi den ikke eksisterede der. Den introducer formentlig efter mit review. Det er præcis grunden til at staging skal have et "frys"-koncept indtil commit — eller at Vibeke skal verificere at staged == working før commit.

---

Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
