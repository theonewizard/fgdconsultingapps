---
dato: 2026-05-08
fra: Laila (HR-leder)
til: Stefan (orkestrator) → FGD
emne: Udkast Application Owner + Application Provider mandater — klar til review
status: UDKAST til FGD-godkendelse
---

# Application Owner + Application Provider Rolle-mandater — Udkastene ligger

**Laila her.** Baseret på Mads' SOD-vurdering + Dorthes GDPR-vurdering har jeg designet to nye projekt-rolle-mandater:

1. **Application Owner** — produkt-ejer pr. applikation, roadmap, team-ansvar, compliance-trigger
2. **Application Provider** — teknisk applikations-ejer, arkitektur, integrationer, audit-trail

Begge udkast ligger i `Team/Laila/` og er klar til FGD-review:

- `Team/Laila/2026-05-08-udkast-application-owner-mandat.md`
- `Team/Laila/2026-05-08-udkast-application-provider-mandat.md`

---

## Hvad jeg gjorde

**Application Owner-mandatet** inkorporerer:
- Mads' "compliance-koordinering" (Owner trigger, ikke vurderer)
- Dorthes GDPR-grænser (ikke juridisk data controller, notificerer DPO før tredjeland, DPIA-screening, kvartalsvis ROPA-attest)
- Ansvarsfordeling med Mads/Dorthe/Provider/Stefan
- 8 GDPR-hard-rules + compliance-drift-ansvar

**Application Provider-mandatet** inkorporerer:
- Mads' least-privilege-krav pr. integration
- Mads' code-review-gate (security-reviewer på alle PR'er)
- Mads' change-management-flow (Provider designer, Camilla implementerer)
- Dorthes dataflow-notifikations-pligt (5 hverdage)
- Dorthes forbud mod selv at godkende leverandører (kræver DPA-vurdering)
- DPA-trigger-liste, DPIA-input-pligt, audit-trail-krav
- HSM-operatør-delegation (Provider designer, ikke executerer)

Begge roller har frontmatter med forslag til model (`sonnet`), farver (blå/cyan for at adskille), tools-sæt.

---

## Vigtige afsnit

**Application Owner — critical hard rules:**

```
- Compliance-vurdering er IKKE dit ansvar (Mads/Dorthe ejer)
- Du må ikke godkende tredjelandsoverførsler uden DPO-notifikation
- DPA påkrævet før ny leverandør i prod
- Team-members kan eskalere compliance direkte til Mads (protected channel)
- Du rapporterer til FGD via Stefan, IKKE direkte til Mads
```

**Application Provider — critical hard rules:**

```
- Least privilege pr. integration; adgangsprofil dokumenteret
- Alle PRs reviewes af security-reviewer; compliance-relevant flagges
- Ingen infrastruktur-ændringer uden Camilla som executor
- Dataflow-notifikation til Dorthe inden 5 hverdage (GDPR art. 30-pligt)
- Kan ikke selv godkende ny leverandør (kræver DPA + Mads)
- HSM aldrig tilgået direkte (goes through operator)
```

---

## Næste skridt til FGD

1. **Læs begge udkast** — fokuser på hard rules for at sikre at grænser virker praktiske
2. **Mads + Dorthe 4-eyes review** — de skal verificere at deres vurderinger er korrekt indarbejdet
3. **Spørgsmål til svar** — begge mandater har åbne spørgsmål i slutningen omkring besætning, livscyklus, break-glass-procedure
4. **GO / ændringer** — når FGD godkender, beder jeg Camilla committe og oprette `Team/<applikation>/application-owner/` + `Team/<applikation>/application-provider/` undermapper

---

## Åbne punkter jeg markerer

**Application Owner-mandat:**
- Hvem skal være Owner for Knowlagecentral?
- Kan Owner + Provider være samme person (SOD-implikation)?
- Eskalerings-flow hvis Owner/Dorthe uenige om compliance-niveau?

**Application Provider-mandat:**
- Hvem skal være Provider for Knowlagecentral?
- Hvornår aktiveres dedikeret HSM-operator?
- Hvordan implementeres break-glass-procedure teknisk?

---

**Status:** Udkastene ligger klar. Stefan, over til dig at koordinere med FGD, Mads, og Dorthe.

*Laila — 2026-05-08*

