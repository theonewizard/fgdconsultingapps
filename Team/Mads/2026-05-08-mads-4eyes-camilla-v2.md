---
title: "4-eyes review: Camilla bibliotekar v2-mandat"
date: 2026-05-08
reviewer: Mads (CISO)
subject: Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md
verdict: CONDITIONAL GO
---

# 4-eyes review — Camilla bibliotekar v2-mandat

**Reviewer:** Mads (CISO)  
**Dato:** 2026-05-08  
**Mandat-version:** v2 (fil: `Min Inbox/_crosscutting/2026-05-06-camilla-bibliotekar-mandat-v2.md`)  
**Grundlag:** SOC 2 CC8.1, ISO 27001 A.5.3, NIST 800-53 AC-5, CC7.2

---

## Samlet vurdering: CONDITIONAL GO

Mandatet er markant bedre end v1. Strukturen er korrekt: scope-first, grep-som-evidens, no-false-claims. Der er tre punkter der skal adresseres inden aktivering — to minor, ét kritisk.

---

## Fund

### KRITISK

**Fund 1 — §3 No-false-claims: definitionen af "verificeret" er for løs under SOC 2 CC7.2**

`§3` kræver grep-output som bevis, men specificerer ikke:
- hvilken grep-syntaks der er tilstrækkelig (enkelt pattern vs. full-scope multi-pattern)
- at grep-outputtet skal gemmes til en navngiven artefakt-fil der kan refereres i audit-trail
- hvad "0 hits" teknisk betyder (fx om grep returnerer exit-code 1 = no match vs. fejl)

Under SOC 2 Type II er det ikke nok at vise 0 hits i en rapport — **auditoren forventer en *reproducerbar* artefakt-sti** (fil i repo eller staging-område) der kan fremvises ved evidence-request. `§3` punkt 3 siger "gem output til fil (fx `Team/Camilla/audit-grep-result.txt`)" — men "fx" gør det optionalt.

**Rettelsesforslag:**
Gør artefakt-gemning obligatorisk: "Gem grep-output som `Team/Camilla/<dato>-<opgave>-grep.txt`. Filstien *skal* inkluderes i rapport som reference. Uden denne sti er rapporten ugyldig som audit-evidens."

---

### MINOR

**Fund 2 — §6 Bash-whitelist: `git reset` er tilladt uden scope-begrænsning**

Whitelist tillader `git reset` uden kvalifikation. `git reset --hard` og `git reset HEAD~N` er destruktive operationer der kan slette commits og staged changes. `git reset HEAD <file>` (unstage) er ufarlig.

En kompromitteret eller fejlende Camilla-session der kører `git reset --hard` kan slette ukritisk eller kritisk evidens. Under SOC 2 CC6.1 (logical access) og ISO 27001 A.8.20 (network controls — tilsvarende logisk rettighed) er dette en reel risiko.

**Rettelsesforslag:**
Opdel whitelist:
```
Tilladt (read/unstage): git reset HEAD <file>
Forbudt uden Stefan-godkendelse: git reset --hard, git reset --soft, git reset HEAD~N
```

**Fund 3 — §4 Memory-feedback-overhold: ingen operationel trigger**

`§4` kræver at Camilla "læser feedback-files FØR audit-opgave", men specificerer ikke *hvad* der definerer en audit-opgave (scope-analyse, bulk-operation, claim-rapportering er nævnt i §1 — men §4 refererer ikke eksplicit til §1's kriterier).

Risiko: Camilla vurderer selv om en opgave er "audit" — og kan undgå memory-læsning ved at kalde det noget andet.

**Rettelsesforslag:**
Knytte §4-trigger direkte til §1's opgaveliste: "§4 aktiveres automatisk når en opgave falder under §1's definitioner (scope-analyse, bulk-operation, claim-rapportering). Ingen vurdering fra Camilla om 'er dette en audit-opgave' — §1-listen er exhaustive."

---

## Vurdering af de 7 reviewpunkter

### 1. Verificerings-procedure (§1) — tilstrækkelig som SOC 2-kontrol?

**Delvist.** Proceduren er korrekt designet (scope-first, grep-som-evidens, loop til 0 hits). Svagheden er at grep-output'et ikke er *obligatorisk* lagret som navngiven artefakt (se Fund 1). Med den rettelse: JA, tilstrækkelig som SOC 2 CC7.2 detective control.

### 2. Git-whitelist (§6) — korrekt afgrænset?

**Nej — `git reset` er for bredt.** Se Fund 2. Resten af whitelisten er acceptabel: `git status/diff/log/show/add` er ufarlige; `git commit/push/rebase/merge/rm` er korrekt forbudt.

Ét manglende punkt: `git stash` er hverken tilladt eller forbudt. Bør eksplicit placeres som "forbudt uden godkendelse" (stash kan skjule staged changes for reviewer).

### 3. No-false-claims (§3) — stærke nok kontrolpunkter?

**Stærke, men ikke audit-tight.** Kontrolpunkterne er de rigtige (re-grep, loop til 0 hits, ingen "FIXET" uden bevis). Svaghed er den ikke-obligatoriske artefakt-gemning (Fund 1). Med rettelsen: tilstrækkelig.

### 4. Memory-feedback-overhold (§4) — operationelt specificeret?

**Utilstrækkeligt.** Kravet er til stede, men triggeren er for løs. Se Fund 3. Kræver eksplicit binding til §1's opgave-definition.

### 5. Mandat-stabilitet (governance-sektionen) — korrekt?

**JA.** Governance-sektionen refererer korrekt SOC 2 CC8.1, ISO 27001 A.5.3, NIST 800-53 AC-5. Eskalerings-sti (Mads + Dorthe → Stefan → FGD) er korrekt. Self-modificerings-forbud er eksplicit. 72-timers undtagelses-klausul er proportional.

### 6. SOD: Camilla ejer git + memory — SOD-risiko?

**Ja, der er en reel SOD-risiko, men den er acceptabelt mitigeret — med ét forbehold.**

Risikoen: én rolle kan skrive memory (påvirke fremtidigt AI-beslutnings-grundlag) OG committe til git (gøre det permanent). Det er i princippet en enkelt aktør der kan manipulere både operationel hukommelse og audit-trail.

Mitigering i mandatet:
- Vibeke-gate på commits (Camilla kan ikke committe alene)
- Mads cc-krav på compliance/security memory-opdateringer
- Self-modificerings-forbud

**Udestående gap**: Mandatet specificerer ikke hvem der *reviewer memory-opdateringer* (ikke kun compliance-memory, men generelt). Vibeke-gate gælder git-commits — men memory kan teknisk skrives og staged uden Vibeke-review, og Vibeke reviewes ikke som memory-governance-rolle. Forslag: Mads bør cc'es på alle `memory/`-writes (ikke kun compliance-relaterede), eller Vibeke's gate udvides til også at inkludere memory-ændringer specifikt. Dette er ikke blokkerende for CONDITIONAL GO, men bør adresseres i v3.

### 7. Projekt-rolle-navne i mandatet?

**NEJ — og det er korrekt.** Mandatet nævner ikke project-roller ved navn (ingen "Karen", "Bjarne", "Henrik", "Frederik" i adfærdsregler). Eneste rollenavne er permanente (Vibeke, Mads, Stefan, Poul) og er i samarbejdssektionen, ikke i hard rules. Det er i overensstemmelse med `feedback_permanent_vs_project_separation.md`.

---

## Opsummering af handlinger for godkendelse

| # | Type | Handling | Ejer |
|---|---|---|---|
| 1 | KRITISK | Gør artefakt-gemning obligatorisk i §3 (navngiven fil, sti i rapport) | Laila/Camilla |
| 2 | MINOR | Split `git reset` i whitelist: `HEAD <file>` tilladt, `--hard/--soft/HEAD~N` forbudt | Laila/Camilla |
| 3 | MINOR | Tilføj `git stash` til forbudt-liste i §6 | Laila/Camilla |
| 4 | MINOR | Bind §4-trigger eksplicit til §1's opgave-kriterier | Laila/Camilla |
| 5 | ANBEFALING (v3) | Udvid memory-review-scope: Mads cc på alle memory-writes, ikke kun compliance | Stefan/Mads |

---

## Konklusion

Mandatet er solidt designet og adresserer de kendte fejlmønstre fra runde 1-5 korrekt. CONDITIONAL GO: Fund 1 (kritisk) skal rettes inden aktivering. Fund 2-4 (minor) bør rettes i samme omgang. Fund 5 (anbefaling) kan vente til v3.

Mads anbefaler at Laila udsteder v2.1 med rettelserne, hvorefter Camilla-mandatet kan aktiveres efter FGD-go.

---

*Mads (CISO) — FGD-teamet, 2026-05-08*
