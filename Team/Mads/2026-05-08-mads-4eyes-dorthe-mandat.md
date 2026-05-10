---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: 4-eyes review — Dorthe DPO mandat-udkast
status: CONDITIONAL GO
relateret: Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md
reviewet-af: Mads (CISO) — governance, SOD, compliance-perspektiv
---

# 4-eyes Review: Dorthe DPO mandat-udkast

**Samlet vurdering: CONDITIONAL GO**

Mandatudkastet er solidt og Lailas co-design med Mads' governance-rammer er tydeligt. Dorthe-rollen er nødvendig og designet er i overensstemmelse med GDPR art. 37-39. Tre punkter skal rettes inden commit — to er kritiske (blokkerende), et er kritisk-adjacent. Tre punkter er minor og kan rettes i samme PR.

---

## Fund 1 — KRITISK (blokkerende): Breach-notifikation blokeres af FGD-godkendelseskrav

**Linje 45, Mission punkt 3:**
> "Breach-respons — notifikation til Datatilsynet inden 72 timer (art. 33) + data-subjects hvis risiko (art. 34) -> FGD skal godkende inden"

**Problem:**
GDPR art. 33 kræver notifikation "uden unødigt ophold og om muligt senest 72 timer" fra *kendskab* til brud — ikke fra FGD's godkendelse. Formuleringen "FGD skal godkende inden" skaber en compliance-risiko: hvis FGD er utilgængelig (ferie, sygdom, tidsforskel, teknisk fejl) og Dorthe afventer godkendelse, kan 72-timer-fristen overskrides. Det er en GDPR-overtrædelse, der i sig selv kan medfører bøde (art. 83(4)).

Hertil kommer at GDPR art. 38(3) beskytter DPO'ens funktionelle uafhængighed — et krav om forudgående ledergodkendelse til en notifikation, der er et lovkrav, underminerer denne uafhængighed og kan blive vurderet negativt af Datatilsynet ved inspektion.

**Korrekt governance-model:**
Dorthe skal have kompetence til at notificere Datatilsynet *selvstændigt* inden for 72 timer. FGD underrettes *parallelt* — ikke som forudsætning. For notifikation til data-subjects (art. 34) er tidsrammen "uden unødigt ophold" (ingen fast grænse), men samme princip gælder.

**Rettelsesforslag — linje 45:**
```
Breach-respons — notifikation til Datatilsynet inden 72 timer (art. 33) + data-subjects
hvis risiko (art. 34). Dorthe notificerer selvstændigt ved at 72-timer-fristen nærmer
sig; FGD orienteres parallelt (ikke som forudsætning). Hvis tid tillader (>24 timer
tilbage) indhentes FGD-accept inden notifikation; under tidspres handler Dorthe og
dokumenterer beslutningsgrundlaget.
```

Tilsvarende skal linje 63 (Arbejdsgang punkt 3, breach-bullet) præciseres. Nuværende tekst: "Breach: implementér notifikationsprocedure i samarbejde med Mads/security-reviewer" er OK, men bør tilføje: "Dorthe er decision-maker på Datatilsynets notifikation — Mads er sparring, ikke godkender."

---

## Fund 2 — KRITISK (blokkerende): Project-rolle-navne i Anti-patterns

**Linje 143-144:**
```
- Implementerer ikke breach-notification-system (Henrik gør det — Dorthe designer krav)
- Drifter ikke HSM (Bjarne gør det)
```

**Problem:**
Governance-reglen om permanent vs. project-rolle separation (memory: `_crosscutting/feedback_permanent_vs_project_separation.md`) er eksplicit: permanente mandater må ikke navngive project-roller direkte. "Henrik" og "Bjarne" er project-roller. Mandatudkastet har allerede en korrekt formulering i Hard rules (linje 70-71), som bruger generiske termer.

Disse to linjer er inkonsistente med resten af mandatets egen governance-linje og bryder cross-cutting-reglen.

**Ironi:** Mandatets Hard rules (linje 70-71) beskriver eksplicit projekt-rolle-uafhængighed — og Anti-patterns-sektionen bryder det i samme dokument.

**Rettelsesforslag — erstat linje 143-144:**
```
- Implementerer ikke breach-notification-system (backend-/frontend-udvikler gør det — Dorthe designer GDPR-krav)
- Drifter ikke HSM (HSM-operatør gør det)
```

---

## Fund 3 — KRITISK-ADJACENT: Mandat-stabilitet — Dorthe kan ikke vurdere sin egen mandatændring (SOD)

**Linje 169:**
```
| `model` | `opus` | Mads + Dorthe + FGD (compliance-impact-vurdering) |
```
**Linje 171:**
```
| Hard rules (body §X) | (se hard rules-sektion) | Mads + Dorthe + FGD |
```
**Linje 178:**
```
3. Vent på Mads' security/SOD-vurdering + Dorthes GDPR-vurdering (uanset trigger; ...)
```

**Problem:**
Separation of Duties (SOD) kræver at en rolle ikke kan vurdere ændringer til sit eget mandat. Dorthe som "Dorthe GDPR-vurdering" i sin egen mandatændring-eskaleringspath er et SOD-brud. Det svarer til at CFO godkender sin egn lønjustering.

Parallelt: I Mads' eget mandat (linje 160-163) er eskaleringspath for Mads' mandat "Mads + Dorthe + FGD" — Mads er her med fordi Mads er den kompetente til at vurdere security-impact af sin egen mandatændring. Det er en lidt anden situation: Mads er assessor, ikke subject. Men for Dorthe: Dorthe er subject — hun kan ikke simultaneously være uafhængig assessor.

**Rettelsesforslag — linje 169, 171:**
```
| `model` | `opus` | Mads + Poul (eller Trine når aktiv) + FGD |
| Hard rules (body §X) | (se hard rules-sektion) | Mads + Poul (eller Trine når aktiv) + FGD |
```

Linje 178: Erstat "Dorthes GDPR-vurdering" med "Mads' security/SOD-vurdering + uafhængig GDPR-vurdering (Trine eller Poul)".

**Note:** Dette er ikke et blokerende compliance-krav som Fund 1 og 2 — men det er en SOD-svaghed der vil fremstå i en SOC 2 Type II-audit som en kontrolsvaghed. Anbefales rettet i samme PR.

---

## Fund 4 — MINOR: Camilla-formuleringen er for bred

**Linje 87:**
> "Camilla hjaelper Dorte med alt dokumentation"

**Problem:**
Formuleringen "alt dokumentation" er for bred og uklar om ansvarsfordelingen. Camilla er executor/administrator; Dorthe er content-owner. Formuleringen kan give Camilla et uklart mandat og flytte ansvar over på Camilla for indhold Dorthe ejer.

Der er desuden en stavefejl: "Dorte" (mangler h).

**Rettelsesforslag:**
```
Camilla (bibliotekar) | DPA-dokumentation, memory-vedligehold | Dorthe skriver ROPA/DPA-indhold;
Camilla administrerer git, formatterer og vedligeholder links i memory. Dorthe er content-owner,
Camilla er executor.
```

---

## Fund 5 — MINOR: WebSearch/WebFetch — acceptabelt, men bør begrundes

**Frontmatter linje (tools):**
```
tools: Read, Write, Edit, Glob, Grep, WebSearch, WebFetch, Agent, TodoWrite
```

**Observation:**
WebSearch og WebFetch er i Dorthes tools-liste. Stefan (orkestrerings-rolle) har ikke disse. For en DPO er web-adgang legitimt begrundet: EDPB-afgørelser opdateres løbende, Datatilsynets vejledninger publiceres online, DPA-standard-klausuler (SCCs) publiceres af EU-Kommissionen. Det er anderledes end en udvikler-rolle eller en administrativ rolle.

**Anbefaling:**
Tilføj en sætning i Stack-kompetencer eller Arbejdsgang der begrunder web-adgang eksplicit — fx "WebSearch/WebFetch anvendes til: EDPB-afgørelser, nationale DPA-vejledninger, EU-Kommissionens SCC-opdateringer, Datatilsynets offentliggjorte afgørelser." Dette gør tool-brugen auditbar og forhindrer scope-creep.

Dette er ikke et SOD-brud og ikke blokkerende — men anbefales tilføjet.

---

## Fund 6 — MINOR: Manglende eksplicit art. 38(3) uafhængighedserklæring i Hard rules

**Observation:**
Uafhængighed af Mads er nævnt i Mission og Hand-off, men Hard rules har ingen eksplicit regel som beskytter Dorthes art. 38(3)-status operationelt. Det kunne blive et spørgsmål i en DPA-inspektion: "Kan DPO'en dokumentere at hun ikke modtager instruktioner om udøvelse af sine opgaver?"

**Rettelsesforslag — tilføj til Hard rules (linje 67ff.):**
```
- **GDPR art. 38(3) uafhængighed**: Dorthe modtager ikke instruktioner fra Mads, Stefan eller andre
  team-roller om udøvelse af sine DPO-opgaver. Uenigheder om GDPR-vurderinger eskaleres direkte
  til FGD — ikke via Mads-hierarkiet.
```

---

## Opsummering

| Fund | Kategori | Blokkerende | Handling |
|---|---|---|---|
| 1. Breach-notifikation + FGD-godkendelse | Kritisk | Ja | Ret linje 45 + 63 |
| 2. Henrik/Bjarne i Anti-patterns | Kritisk | Ja | Ret linje 143-144 |
| 3. Dorthe i sin egen eskaleringspath (SOD) | Kritisk-adjacent | Nej, men bør rettes | Ret linje 169, 171, 178 |
| 4. Camilla-formulering + stavefejl | Minor | Nej | Ret linje 87 |
| 5. WebSearch/WebFetch begrundelse mangler | Minor | Nej | Tilføj begrundelse |
| 6. Art. 38(3) eksplicit i Hard rules | Minor | Nej | Tilføj regel |

**Konklusion:** Ret fund 1 og 2 (begge blokkerende) samt fund 3 (SOD-svaghed) inden commit. Fund 4-6 kan rettes i samme omgang. Mandatets grundlæggende design er godt — uafhængighedsprincippet er korrekt implementeret i arkitektur og flow; der er bare to konkrete anti-patterns og et proceduralt hul ved breach-tidspres der skal tættes.

Mads anbefaler **CONDITIONAL GO** til FGD: godkend med betingelse om at ovenstående 6 fund er adresseret i den endelige version Camilla committer.
