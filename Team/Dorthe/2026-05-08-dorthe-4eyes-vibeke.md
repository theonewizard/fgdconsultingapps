---
date: 2026-05-08
author: dorthe-dpo
subject: 4-eyes GDPR-review — Vibeke commit-reviewer-mandat
verdict: CONDITIONAL GO
---

# 4-eyes GDPR-review: Vibeke commit-reviewer-mandat

**Dorthe, DPO — uafhængig vurdering fra GDPR-vinkel**
**Grundlag:** `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`

---

## Konklusion

**CONDITIONAL GO**

Mandatet er operationelt forsvarligt fra et GDPR-perspektiv, men tre betingelser skal opfyldes inden aktivering.

---

## Fund

### 1. Dataadgang — dataminimering (GDPR art. 5(1)(c))

**Fund:** Vibeke har read-adgang til ALLE staged filer via `Read`, `Glob`, `Grep` og `Bash`-kommandoer (`git diff --cached`, `git show`). Der er ingen begrænsning på hvilke fil-typer eller indhold hun eksponeres for. En commit kan indeholde:

- Personnavne i dokumentation (teammedlemmers navne, kundenavne)
- E-mail-adresser i DPA-tracker eller ROPA
- Potentielt PII i breach-rapporter eller DPIA-dokumenter
- Memory-opdateringer med brugerhistorik

**Vurdering:** Adgang er nødvendig for at reviewe commits korrekt — det er legitimt formål (GDPR art. 6(1)(f): berettiget interesse i integritets-kontrol). Men mandatet specificerer ingen dataminimerings-tilgang til PII-eksponering.

**Betingelse A:** Tilføj instruks til mandatet: Vibeke må ikke gemme, logge eller videreformidle PII hun ser under review ud over hvad der fremgår af commit-godkendelsen (fx "HOLD — filnavn bryder konvention"). Vibeke refererer PII-indhold alene for at validere, ikke for at reproducere det.

---

### 2. Memory-review — GDPR-risici ved persondata i memory (GDPR art. 5, 25)

**Fund:** Mandatet nævner eksplicit at Vibeke validerer "crosscutting-inddeling" og "viden-inddeling-kontrol (Camillas mandat)". Camillas memory-opdateringer kan indeholde personoplysninger om:

- FGD (mail, rolle, præferencer)
- Teammedlemmer (kompetencer, beslutningshistorik)
- Kunder (vertikal, regulatorisk kontekst)

Vibeke reviewes ikke for om hun selv håndterer disse korrekt — kun om filerne er placeret rigtigt.

**Vurdering:** Der er en GDPR art. 25 (Privacy by Design)-risiko: memory-arkitektur er ikke dokumenteret i ROPA endnu (dette er Dorthes eget udestående). Vibeke's review er ikke den udløsende risiko — men memory-review uden ROPA-dækning er en uafdækket behandlingsaktivitet.

**Betingelse B:** ROPA skal opdateres til at inkludere memory-behandling (formål, kategorier, retention, adgang) inden Vibeke aktiveres på memory-commits. Dette er Dorthes ansvar — Camilla eksekverer dokumentationen.

---

### 3. Audit-trail — GDPR art. 5(2) ansvarlighed og SOC 2-bevis

**Fund:** Mandatet beskriver review-proceduren og beslutningstyperne (GODKENDT/AFVIST/TVIVL), men specificerer **ikke** at godkendelser skal logges på en måde der er tilgængeligt for audit. Trine nævnes som reviewer af "review-historik (git-log + commit-beskeder)" post-hoc, men:

- Det er uklart om GODKENDT-beskeder til Camilla gemmes synligt i git-historik
- Afviste commits efterlader muligvis ingen spor i git-log (de committes ikke)

**Vurdering:** GDPR art. 5(2) kræver at FGD som dataansvarlig kan demonstrere compliance. Godkendelseslog er ikke kun SOC 2-krav — det er også dokumentation for at behandlingsaktiviteter er kontrolleret. Et afvist commit med PII-fejl bør logges med hvad der blev fundet og afvist.

**Betingelse C:** Vibeke-mandatet skal præcisere at alle AFVIST-beslutninger med compliance-begrundelse (PII, secrets, GDPR-dokument-placering) skal noteres i `governance/godkendelseslog.md` — ikke blot i Camillas commit-dialog. GODKENDT-beslutninger behøver kun git-log-spor.

---

### 4. DPO-uafhængighed (GDPR art. 38(3))

**Fund:** Vibekes mandat nævner eksplicit at hun **ikke** må kalde Dorthe (listede under "Udvikler-roller" der er forbudt at kalde). Dette er korrekt — Vibeke bør ikke instruere Dorthe om GDPR-vurderinger, og Dorthe bør ikke indgå i review-dialog der kan kompromittere uafhængig vurdering.

**Vurdering:** Designet er korrekt. Vibeke's adgang til Dorthes GDPR-dokumentation (ROPA, DPIA, DPA-tracker) via `Read` er forventet og nødvendig for at validere filplacering og indhold. Dette er ikke en instrukturering af Dorthe — det er kontrol af dokumentation.

**Ingen betingelse — dette er OK.**

---

### 5. Mandat-stabilitet — SOD-integritet ved denne review

**Fund:** Dette 4-eyes review er initieret som del af mandatets godkendelsesproces. Dorthe reviewer Vibeke's mandat, ikke sit eget. SOD-princippet er korrekt overholdt:

- Vibeke reviewer Camillas commits (ikke Vibekes egne, da hun ikke har Write/Edit)
- Mads reviewer security/SOD-aspekter (separat review)
- Dorthe reviewer GDPR-aspekter (dette dokument)
- FGD godkender samlet

Mandatets governance-tabel (§Mandat-stabilitet) indeholder Dorthe korrekt som en af dem der skal konsulteres ved tools/hard-rules-ændringer. Eskaleringsstien er konsistent med `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md`.

**Ingen betingelse — kæden er korrekt.**

---

## Sammenfatning af betingelser

| # | Betingelse | Ansvarlig | Frist |
|---|---|---|---|
| A | Tilføj dataminimerings-instruks til Vibeke-mandatet: PII set under review må ikke reproduceres udover commit-beslutningen | Laila (mandat-redaktør) + FGD-go | Inden aktivering |
| B | ROPA opdateres med memory-behandling (formål, kategorier, retention, adgang) inden Vibeke reviewes memory-commits | Dorthe (indhold) + Camilla (dokumentation) | Inden Vibeke-aktivering på memory-commits |
| C | Tilføj krav: AFVIST-beslutninger med compliance-begrundelse logges i `governance/godkendelseslog.md` | Laila (mandat-redaktør) + FGD-go | Inden aktivering |

---

## Næste skridt

Stefan koordinerer:
1. Mads' parallel security/SOD-review (hvis ikke allerede modtaget)
2. Samlet anbefaling (Mads + Dorthe) præsenteres for FGD
3. FGD giver eksplicit GO med eller uden betingelserne implementeret først
4. Camilla committer opdateret Vibeke-mandat efter FGD-go
5. Trine logger change-event i audit-trail

*Dorthe, DPO — 2026-05-08*
