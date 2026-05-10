---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: 4-eyes review — Vibeke commit-reviewer mandat
status: CONDITIONAL GO
relateret: Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md
reviewet-af: Mads (CISO) — SOD, governance, compliance, Bash-whitelist
---

# 4-eyes Review: Vibeke commit-reviewer mandat

**Samlet vurdering: CONDITIONAL GO**

Vibeke-mandatets kernedesign er korrekt: read-only, SOD-løst by-design, én autoriseret reviewer. Det er det rigtige kontrol-princip. To fund er blokkerende og skal rettes inden aktivering. Fire fund er minor og bør rettes i samme omgang. Mandatets Bash-whitelist og mandat-stabilitet-sektion er solide.

---

## Fund 1 — KRITISK (blokkerende): Utilgængelighed — pause-regel er underspecificeret

**Mandat §Hard rules 4 + §Noter til implementering:**

> "Hvis du ikke svarer, pauses commits — der er ingen backup-reviewer (Mads fungerer kun midlertidigt som bro under initialisering)."
> "Pause-regel: Hvis Vibeke ikke kan nås i >X minutter (evt. concurrent-timeout), pauses alle Camilla-commits. Stefan noteres."

**Problem:**
"X minutter" er uspecificeret. Dette er ikke en redaktionel sletning — det er et aktivt hul i en kontrol der skal fungere automatisk. Uden konkret tærskel kan Camilla eller Stefan fortolke "utilgængelighed" forskelligt: 5 minutter? 30? En dag? Tærsklen er operationelt kritisk fordi:

1. For kort timeout → Camilla blokeres ved normale ventetider (Vibeke er agent med naturlig latency).
2. For lang timeout → kontrollen er illusorisk; commits kan akkumulere og vente på en reviewer der aldrig svarer.
3. SOC 2 CC8.1 kræver at kontroller er *dokumenterede og målbare* — "X minutter" er ikke auditbar.

**Rettelsesforslag — §Hard rules 4 og §Noter til implementering:**

Definér konkret tærskel, f.eks.:

```
Pause-regel: Hvis Vibeke ikke svarer inden 15 minutter efter Camillas notifikation,
pauses alle Camilla-commits. Stefan noteres. Commits genoptages IKKE uden Vibekes
eksplicitte godkendelse — heller ikke ved Stefans initiativ alene.
```

Tærsklen (15 min er et forslag) skal besluttes og dokumenteres af FGD/Stefan. Selve tallet er ikke Mads' beslutning — men det skal ikke stå som "X".

**Sekundært:** Mandatet siger "Mads fungerer kun midlertidigt som bro under initialisering" — men specificerer ikke hvornår initialiseringsperioden slutter og hvad der erstatter Mads-broen. Tilføj eksplicit: "Mads-broen ophører når Vibekes første godkendte test-commit er logget (jf. §Noter til implementering, test-commit-punkt). Fra da er pause den eneste fallback."

---

## Fund 2 — KRITISK (blokkerende): Agent-begrænsning — "kun Stefan + Mads" men §Hard rules 3 forbyder Mads-eskalering til Trine

**Mandat §Hard rules 3:**

> Du kalder **kun**:
> - **Stefan** — eskalering ved tvivl, policy-spørgsmål, eller mønster-problemer
> - **Mads** — security-implications eller gentagne brud

**§Hand-off:**

> **Trine** | Ingen direkte | Trine reviewer din review-historik (git-log + commit-beskeder) post-hoc som SOC 2 CC8.1-evidens. Bias-risiko hvis I taler direkte.

**Problem:**
Designet er korrekt (Trine-isolation for audit-uafhængighed). Men mandatets §Hard rules 3 siger Vibeke eskalerer gentagne brud til Mads. Hvad sker der hvis Mads ikke er tilgængelig og Vibeke ser et kritisk sikkerhedsmønster (f.eks. secrets i 3 commits fra Camilla)? Mandatet siger implicit: vent. Men der er ingen fallback-sti for Mads-utilgængelighed ud over Stefan — og Stefan er orkestrerings-rolle, ikke security-kompetent nok til at afgøre om et mønster er kritisk. 

Dette skaber en kontrol-blindzone: ved simultant Mads- og Vibeke-utilgængelighed (fx ved session-restart) kan Camilla akkumulere commit-kø uden nogen der eskalerer sikkerheds-mønsteret.

**Rettelsesforslag — §Hard rules 3, tilføj:**

```
Ved Mads-utilgængelighed og kritisk sikkerhedsmønster (≥3 brud samme mønster
eller secret-lækage): eskalér til Stefan med label "KRITISK — Mads utilgængelig".
Stefan holder commits i kø og notificerer FGD direkte. Ingen commits frigives
uden Mads eller FGD-godkendelse.
```

Dette er ikke et SOD-brud — Stefan er orkestrering, ikke reviewer. Stefan videresender ikke godkendelse; han holder pausen og notificerer FGD.

---

## Fund 3 — MINOR: Bash-whitelist — `cat` bør specificeres yderligere

**Mandat §Hard rules 2:**

> Tilladt: `git status`, `git diff`, `git diff --cached`, `git log`, `git show`, `ls`, `cat`

**Observation:**
`cat` er uspecificeret. I sig selv er `cat` read-only — men `cat > fil` (med redirection) er skrivende. Whitelisten siger "enhver skrivende shell-kommando" er forbudt, og mandatet siger systemet skal blokere det teknisk. Problemet er at `cat` alene uden redirection er uskadeligt, men `cat file1 file2 > output.txt` er en omgåelse.

Hvis settings.local.json kun whitelister kommando-*navne* (og ikke argumentmønstre), kan redirection bruges alligevel afhængigt af Claude Code's shell-sandboxing.

**Rettelsesforslag:**

Tilføj forbudt-liste: "`>` og `>>` shell-redirection er forbudt uanset kommando" — og verificer at settings.local.json-implementeringen blokerer shell-redirection i Bash-tool, ikke kun kommando-navne. Camilla bør bekræfte ved test-commit at `cat fil.txt > ny.txt` bliver blokeret.

Dette er teknisk minor — Claude Code's Bash-tool håndterer shell-sandboxing — men det bør ekspliciteres i mandatet så implementeringen er auditbar.

---

## Fund 4 — MINOR: SOD — Vibeke er korrekt isoleret, men §Mission siger "peer-niveau med Mads"

**Mandat §Mission:**

> Du er **Vibeke** [...] permanent rolle (peer-niveau med Mads, ikke underordnet).

**Observation:**
"Peer-niveau med Mads" er governance-korrekt for uafhængighed fra Camilla og commit-rettigheder. Men det skaber en implicit forventning at Vibeke og Mads har sammenlignelig myndighed i f.eks. policy-beslutninger. Det har de ikke — Vibeke's scope er commit-gate, ikke governance-design.

Dette er ikke et SOD-brud, men det kan skabe uklarheder hvis Vibeke i fremtiden forsøger at eskalere *forbi* Mads til FGD direkte (ikke tilladt ifølge Agent-begrænsning og flow).

**Rettelsesforslag:** Præcisér: "peer-niveau med Mads hvad angår reviewer-uafhængighed og commit-gate-autoritet — ikke hvad angår governance-scope eller policy-beslutninger."

---

## Fund 5 — MINOR: §4-eyes integration — mandatet nævner ikke det nye 4-eyes governance-flow eksplicit

**Observation:**
Vibeke-mandatets §Hard rules 4 beskriver commit-gate med Vibeke som enereviewer. Men mandatet refererer ikke til det 4-eyes governance-flow som er dokumenteret i Stefans CLAUDE.md og Mads' mandat — nemlig at Mads + Dorthe begge vurderer ALLE rolle-konfigurationsændringer.

For Vibekes eget mandat er eskaleringspath korrekt (§Mandat-stabilitet, linje 157-161: Mads + Dorthe + FGD). Det er i orden.

**Det der mangler** er en reference til hvad 4-eyes *betyder for Vibeke's rolle i commit-gaten*: Vibeke er *implementeringen* af 4-eyes for dokumentations-commits. Mandatet bør eksplicit sige dette, så SOC 2-auditors forstår rollens funktion i kontrol-rammen:

**Rettelsesforslag — §Mission, tilføj:**

```
Vibeke er implementeringen af 4-eyes princippet for dokumentations- og
governance-commits: Camilla skriver, Vibeke godkender. Ingen commit må
passere med én part alene.
```

---

## Fund 6 — MINOR: Projekt-rolle-navne — mandatet er rent (ingen brud)

**Vurdering:**
Vibeke-mandatets Agent-begrænsning nævner konkrete navne: "Camilla", "Trine", "Karen", "Bjarne", "Sofie", "Henrik", "Anders", "Dorthe", "Frederik", "Poul", "Laila".

**Analyse:**
Dette er et edge-case i governance-reglen om permanent vs. project-rolle separation. Mads' og Dorthes mandater må ikke navngive project-roller i funktionelle beskrivelsesbokse (Anti-patterns, Kompetencer). Men Vibeke-mandatets §Hard rules 3 er en Agent-whitelist — den er *operationel kontrol*, ikke funktionel beskrivelse. Det er forsvarligt at navngive hvem Vibeke MÅ og IKKE MÅ kalde, fordi whitelisten er præcis og ikke kan erstattes af generiske kategorier (du kan ikke sige "kald ikke <project-roller>" — Vibeke skal vide præcist hvem de er for at håndhæve whitelisten).

**Konklusion:** Ingen brud. Navnene er her brugt som access-control-liste, ikke som rollebeskrivelse. Kontrol-formålet er legitimt.

---

## Opsummering

| Fund | Kategori | Blokkerende | Handling |
|---|---|---|---|
| 1. Pause-tærskel "X minutter" uspecificeret | Kritisk | Ja | Konkretisér tærskel + Mads-bro-ophørsdefinition |
| 2. Mads-utilgængelighed ved kritisk mønster | Kritisk | Ja | Tilføj Stefan-fallback-sti med FGD-eskalering |
| 3. `cat` + redirection underspecificeret | Minor | Nej | Tilføj redirection-forbud + bekræft settings.local.json |
| 4. "Peer-niveau med Mads" kan misfortolkes | Minor | Nej | Præcisér scope for peer-status |
| 5. 4-eyes integration ikke eksplicit navngivet | Minor | Nej | Tilføj 4-eyes formulering i §Mission |
| 6. Projekt-rolle-navne i whitelist | Ingen brud | N/A | Ingen handling nødvendig |

**Konklusion:** Mandatets kernekontrol (read-only, SOD by-design, ingen backup-reviewer-delegation) er arkitektonisk korrekt og implementerer SOC 2 CC8.1 korrekt i princippet. Fund 1 og 2 er operationelle huller der gør kontrollen ikke-auditbar og potentielt ikke-fungerende i edge-cases. De skal rettes inden Vibeke aktiveres.

Mads anbefaler **CONDITIONAL GO** til FGD: godkend med betingelse om at fund 1 og 2 rettes (og gerne 3-5 i samme omgang) inden Camilla committer Vibeke-mandatet og aktiverer rollen.
