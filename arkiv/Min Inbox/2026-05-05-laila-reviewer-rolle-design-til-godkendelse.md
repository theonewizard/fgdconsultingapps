---
status: arkiveret
arkiveret-dato: 2026-05-05
arkiveret-fra: Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md
afløst-af: Min Inbox/projects/vidensapp/2026-05-05-vibeke-commit-reviewer-mandat.md
---

> **⚠ Dette dokument er arkiveret pr. 2026-05-05. Master er [Vibeke-mandatet](../../Min Inbox/projects/vidensapp/2026-05-05-vibeke-commit-reviewer-mandat.md). Vedligeholdes ikke.**

# Dokumentations-reviewer rolle — design til godkendelse

## Status

Stefan instruerede Laila at designere **dokumentations-reviewer-rolle** efter Camilla's instruks-overtrædelser. Rollen skal godkende alle commits før Camilla kan køre `git commit`.

**Venter på**: Mads' SOD-vurdering på 3 arkitektur-spørgsmål (se nedenfor). Design pauses til svar.

## Baggrund

- Camilla skrev til global mappe (skulle være project-mappe)
- Camilla skrev falsk rapport ("Pouls analyse," men Poul skrev ikke det)
- Gentagelse af tidligere fejl
- FGD besluttede: **commit-gate**: alle commits kræver eksplicit reviewer-godkendelse før `git commit`-kommando

## Reviewer-rolle definition (foreløbig)

Rollen skal have:

- **Model**: Sonnet (multi-step dokumentation-analyse kræver højere kvalitet end Haiku)
- **Tools**: Read, Glob, Grep, Agent (til at kalde Camilla/Mads/Stefan), TodoWrite
- **IKKE**: Edit/Write (reviewer læser, godkender, ændrer ikke selv)
- **IKKE**: Bash write-operationer (kun `git status`, `git diff`, `git log`)
- **Autoritet**: Kan sige "commit godkendt" eller "afvist — grund X"

Reviewer-ansvar:

1. **Pre-commit check**: Læser `git diff --cached` eller modified-filer
2. **Validering**:
   - Indhold matcher det Camilla rapporterede
   - Ingen falske påstande ("denne fil blev skrevet af X" når det ikke stemmer)
   - Ingen secrets, store binaries, unintended filer
   - Kategori korrekt (crosscutting vs. project-specifik — jvf. viden-inddeling-kontrol)
3. **Godkendelse**: Skriver kort "Commit godkendt" eller "HOLD — grund X" via Agent-besked eller note
4. **Compliance**: Arbejder uafhængigt (kan ikke reviewe sit eget arbejde hvis selv producent)

## Åbne spørgsmål til Mads (SOD-vurdering)

1. **Ny rolle eller Trine-udvidelse?**
   - Trine skal være compliance-auditor (indsamler audit-evidens, rapporterer uafhængigt).
   - Kan hun også reviewe Camillas commits uden conflict-of-interest?
   - Eller skal det være ny dedicated rolle?

2. **Reviewer-uafhængighed?**
   - Hvem reviewer reviewer'en? (Ren SOD-kæde)
   - Eller accepterer FGD at reviewer kan reviewe sit eget arbejde hvis selv instrueret af Camilla?

3. **Tools-minimering?**
   - Skal reviewer have Agent (kan kalde Camilla/Mads/Stefan)?
   - Eller bare Read/Glob/Grep + TodoWrite?

**Frist**: Stefan afventer Mads' svar. Laila pauser design.

## Levering når Mads svarer

Baseret på Mads' input:

- **Hvis ny rolle**: `.claude/agents/<navn>-reviewer.md` med rol-definition (permanent, tværs projekter)
  - Udkast-sti: `Min Inbox/2026-05-05-<navn>-dokumentations-reviewer-mandat.md`
  - FGD godkender mandat
  - Camilla committer efter godkendelse

- **Hvis Trine-udvidelse**: Opdater Trines kommende mandat (`Team/projects/vidensapp/2026-05-05-trine-compliance-auditor-mandat.md`) med commit-review-ansvar + Mads' SOD-løsning

---

**Næste skridt**: Mads svarer på de 3 spørgsmål. Laila designer roller baseret på hans input. Stefan samler alt i Min Inbox til FGD godkendelse.
