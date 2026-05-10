---
date: 2026-05-08
author: dorthe-dpo
subject: Final 4-eyes GDPR-review — Vibeke commit-reviewer-mandat (efter rettelser)
verdict: GO
---

# Final GDPR-review: Vibeke commit-reviewer-mandat

**Dorthe, DPO — uafhængig slutvurdering**
**Reference:** `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`
**Første review:** `Team/Dorthe/2026-05-08-dorthe-4eyes-vibeke.md` (CONDITIONAL GO, 3 betingelser)

---

## Samlet konklusion

**GO**

Alle tre GDPR-betingelser fra første review er korrekt implementeret i mandatet. Ingen nye fund der blokerer aktivering.

---

## Verifikation af betingelser

### Betingelse A — PII-dataminimering (GDPR art. 5(1)(c))

**Status: LØST**

Mandat linje 82 (review-procedure trin 5) indeholder nu:

> "PII-dataminimering: Personnavne, e-mailadresser, telefonnumre eller andre kontaktoplysninger på rigtige mennesker der ses i staged filer må ikke reproduceres, citeres eller gemmes i nogen form ud over selve commit-beslutningen (GO/NO-GO + årsag). Vibeke noterer kun beslutningen — ikke PII-indholdet. Hvis PII-handling påkræver dokumentation, sendes rapport direkte til Mads uden PII-visning. Dette sikrer GDPR art. 5(2) (dataminimering)."

Formuleringen dækker art. 5(1)(c) korrekt: Vibeke har adgang til PII som nødvendig del af review-opgaven (legitimt formål, art. 6(1)(f)), men er eksplicit forbudt at reproducere, gemme eller videreformidle det. Proportionalitetskravet er opfyldt.

---

### Betingelse C — AFVIST-beslutninger logges (GDPR art. 5(2) ansvarlighed)

**Status: LØST**

Mandat linje 91 indeholder nu:

> "Loggingskrav ved AFVIST: Når du afviser en commit med compliance-begrundelse (PII-datalækage, secrets, forkert projekt-inddeling, GDPR-dokument-placering eller anden governance-overtrædelse), skal afvisningen logges i `governance/godkendelseslog.md` med: dato, commit-beskrivelse/ref, afvisningsårsag (kategori + specifik grund), og dit navn (Vibeke). Dette kræves af GDPR art. 5(2) (ansvarlighed) og SOC 2 CC8.1 audit-spor."

Ansvarligheds-kæden er komplet: GODKENDT-beslutninger efterlader spor via git-log (Camilla committer med reference), AFVIST-beslutninger med compliance-begrundelse logges eksplicit i `governance/godkendelseslog.md`. GDPR art. 5(2) er dækket.

---

### Betingelse B — ROPA-dækning af memory-behandling (GDPR art. 5, 25)

**Status: DELVIST LØST — ikke bloker for aktivering**

Dette er Dorthes eget ROPA-udestående, ikke et mandat-rettelsespunkt for Vibeke. Mandatet specificerer korrekt at Vibeke validerer fil-placering og indhold, men ikke at hun selv er ansvarlig for ROPA-dokumentation.

Betingelsen er operationelt løst ved at Vibeke-mandatet begrænser hendes rolle til read-only review — hun behandler ikke persondata, hun kontrollerer om dokumenter er korrekt placeret. Memory-behandlingen som databehandlingsaktivitet skal dokumenteres i ROPA inden memory-commits reviewes. Dette er Dorthes opgave og vil indgå i ROPA-sprint (Fase 0).

**Konklusion:** Vibeke kan aktiveres på alle commit-typer undtagen memory-commits. Aktivering på memory-commits afventer ROPA-opdatering fra Dorthe.

---

## Nye fund

**Ingen fund der blokerer GO.**

Et enkelt observationspunkt til Mads' orientering (ikke bloker):

**Observation — FGD-utilgængelighed ved pause (linje 94):** Mandatets § "Håndtering af utilgængelighed" specificerer at Stefan eskalerer direkte til FGD ved 15-minutters pause. Det forudsætter at FGD altid er tilgængelig. Ved FGDs fravær (ferie, sygdom) er der ingen fallback-DPO eller -reviewer nævnt. Dette er et operationelt hul — ikke et GDPR-brud i mandatteksten, men det bør adresseres i en drifts-procedure. Anbefaling: Stefan noterer i Min Inbox hvem der kan kontaktes hvis FGD ikke svarer inden 30 minutter.

---

## Sammenfatning

| Betingelse | Grundlag | Status |
|---|---|---|
| A — PII-dataminimering tilføjet | GDPR art. 5(1)(c) | LØST |
| C — AFVIST-beslutninger logges | GDPR art. 5(2) | LØST |
| B — ROPA-dækning memory | GDPR art. 5, 25 | DELVIST — Dorthes udestående, ikke bloker |

**Verdict: GO**

Vibeke-mandatet er klar til FGD-godkendelse og aktivering (med undtagelse af memory-commits indtil ROPA er opdateret).

*Dorthe, DPO — 2026-05-08*
