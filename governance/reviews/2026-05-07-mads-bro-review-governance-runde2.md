---
dato: 2026-05-07
reviewer: Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
emne: Runde 2 verifikation af Lailas governance-rettelser efter FGD-override
beslutning: AFVIST
status: leveret
---

# Runde 2 bro-review — governance-override implementering

## Sammenfatning

**Beslutning:** AFVIST. 6 af 7 spor verificeret OK; Spor 3 (Stefan-CLAUDE.md) bryder FGD-override 2026-05-06 ved at videreføre trigger-baseret Dorthe-konsultation som primær eskaleringspath i selve Stefans hovedinstruktion. Banneret i Mads' superseded policy retter ikke at Stefans operative tekst replikerer den superseded regel.

## Verifikations-resultat pr. spor

### Spor 1 — De 4 knowlagecentral-mandater (Sofie/Henrik/Anders/Karen): GODKENDT

- Alle 4 har strengen `(ikke trigger-baseret)` i FGD-override-konteksten (Sofie:106, Henrik:108, Anders:103, Karen:175). Mit grep-mønster fangede ordet, men ALTID i den korrekte negations-form ("ALLE ændringer (ikke trigger-baseret)") — dvs. det er bekræftelse på rettelse, ikke restkode.
- Alle 4 har sektion "## Mandat-stabilitet (governance, ikke skiftbar uden FGD)".
- Eskaleringsformulering: "eskalér til Mads (CISO) og Dorthe (DPO). Begge skal vurdere ALLE ændringer".

### Spor 2 — Bjarne + Frederik governance-section: GODKENDT

- Bjarne:165 og Frederik:158 har "## Mandat-stabilitet"-sektion.
- Model-felt verificeret: Bjarne `model: haiku` (linje 39), Frederik `model: sonnet` (linje 33). Korrekt jf. rolle-design.
- Self-modificering-forbud + Camilla-commit-eksklusiv tilstede.

### Spor 3 — Stefan-CLAUDE.md governance-regel: AFVIST

- Sektion findes (linje 34-42), self-modificering-procedure tilstede.
- **FEJL:** Linje 37: "Eskalér til **Mads** (security/SOD-vurdering). Mads konsulterer **Dorthe** hvis GDPR-trigger jf. `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3."
- Dette gengiver netop den **trigger-baserede konsultation** FGD eksplicit overrode 2026-05-06. §3.3 er markeret SUPERSEDED i policy-filen, men Stefan-CLAUDE.md (Stefans operative master-instruks) replikerer reglen ordret som primær eskaleringspath.
- Lailas egen runde2-rapport (linje 109) bekræfter formuleringen: "Eskalér til Mads + Dorthe (ved GDPR-trigger jf. §3.3)" — også her trigger-baseret.
- **Effekt:** Stefan vil i praksis følge CLAUDE.md, ikke de 6 mandater. Inkonsistens mellem master-orchestrator-regel og rolle-mandater.

### Spor 4 — Mads' policy superseded-banner: GODKENDT

- governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md linje 14: superseded-banner med eksplicit "FGD-override 2026-05-06" og "Mads + Dorthe SAMMEN i ALLE rolle-ændringer (ikke trigger-baseret)".

### Spor 5 — Tværgående trigger-rester: GODKENDT med forbehold

- Hits i: Mads' superseded policy (forventet), de 6 mandaters negations-formuleringer (forventet), Lailas rapporter (forventet), gammelt review (historik). Ingen ægte rest-kode i mandater.

### Spor 6 — Self-modificering forbud i 14 mandater: GODKENDT

- Alle 14 aktive mandater har 1 hit på "Self-modificering". `_template-ny-rolle.md` har 0 hits, men er skabelon (ikke krav).

### Spor 7 — git status: GODKENDT med note

- 41 staged + 23 modificeret + 18 untracked = 69 ændringer total. Forventet "~52+ staged" ikke nået, men runde 2-rettelser ligger som "MM" (modified efter staging) — Camilla skal re-stage før commit. Ikke compliance-blocker.

## Mangler-liste til Laila (skal rettes før godkendelse)

**1. Spor 3 — Stefan-CLAUDE.md linje 37:**

Aktuel tekst:
> "Eskalér til **Mads** (security/SOD-vurdering). Mads konsulterer **Dorthe** hvis GDPR-trigger jf. `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` §3.3."

Skal erstattes af (eksempel):
> "Eskalér til **Mads + Dorthe SAMMEN** (FGD-override 2026-05-06). Begge vurderer ALLE rolle-konfigurations-ændringer for compliance-impact uafhængigt af trigger: Mads = security/SOD/SOC 2/ISO 27001; Dorthe = GDPR/data-håndtering. Reference: `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` (§3.3 superseded af FGD-override; se policy-banner)."

**2. Linje 38** ("For permanente roller eller compliance-impact: Mads + Dorthe + FGD-go. For project-roller engangs-undtagelser: Mads kan godkende.") skal også gennemgås — FGD-override siger "Mads + Dorthe SAMMEN i ALLE rolle-ændringer", hvilket potentielt udelukker "Mads alene godkender project-roller". Laila skal afklare med FGD om project-rolle-undtagelsen er FGD-godkendt eller skal fjernes.

## Compliance-vurdering

Den aktuelle inkonsistens mellem Stefan-CLAUDE.md og rolle-mandaterne er en **change-management-svaghed** under SOC 2 CC8.1 og ISO 27001 A.8.32: ændrings-policy skal være konsistent på tværs af artefakter. Audit-trail vil vise modstridende eskaleringspaths, hvilket Trine vil flagge ved første compliance-pas.

## Anbefaling til Stefan

- **AFVIS commit.** Bed Laila rette Stefan-CLAUDE.md linje 37-38 (FGD-override-formulering, ikke trigger-baseret), og afklare project-rolle-undtagelsen med FGD inden runde 3.
- Når rettet: kort runde 3-verifikation kun på Spor 3.
- Camilla må IKKE committe før Spor 3 er ren.

---

**Mads (CISO, bro-reviewer)**  
*Permanent rolle — bro-reviewer indtil Vibeke aktiveres*  
2026-05-07
