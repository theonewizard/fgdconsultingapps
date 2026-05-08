---
dato: 2026-05-08
emne: Sammenfatning af v2-reviseringer — Application Owner + Application Provider mandater
adressat: Stefan (orkestrator), FGD
status: Info — begge v2-filer produceret og klare til Mads/Dorthe-bekræftelse
---

# Sammenfatning af v2-reviseringer — AO + AP mandater

## Status

Begge mandater er revideret til v2 og ligger klar i:
- `Team/Laila/2026-05-08-udkast-application-owner-mandat-v2.md`
- `Team/Laila/2026-05-08-udkast-application-provider-mandat-v2.md`

---

## Indarbejdede rettelser (Stefan's brief + 4-eyes feedback)

### Rettelse 1 — Self-certify-forbud udvides til ALLE erklæringer (AO + AP)

**Status:** ✅ Indarbejdet

**Hvor:** AO-mandat, Hard rules §Compliance-grænser:
> "Du må ikke erklære en feature som "compliant" — hverken over for kunder, internt i teamet, eller i dokumentation — uden skriftlig anbefaling fra hhv. Dorthe (GDPR) og Mads (ISO/SOC 2)."

**Hvor:** AP-mandat, Hard rules §GDPR-relateret:
> (Implicit via "Du må ikke forhandle DPA'er med leverandører eller udtale dig om GDPR/ISO/SOC 2-compliance til kunder/revisorer på vegne af FGD")

Clausulen dækker både ekstern kundekommunikation og intern teamkommunikation + dokumentation.

---

### Rettelse 2 — Break-glass audit trail immutabilitet (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §Least Privilege & Access Control:
> "Audit-trail for break-glass-sessioner gemmes i uforanderlig form — write-once storage eller eksporteret til separat immutabelt arkiv. Hverken du, Camilla, eller andre roller der producerer eller eksekverer kan modificere dette log. Trine auditerer break-glass-log halvårligt som SOC 2-evidens."

---

### Rettelse 3 — Vibeke-reference præcisering (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §Least Privilege & Access Control:
> "Camilla committer config-change og får commit-reviewer (Vibeke, når aktiv) til at reviewe først."

**Hvor:** AP-mandat, Ansvarsfordeling-tabel, Vibeke-linje:
> "**Vibeke (doc-reviewer)** | Commit-gate (project-rolle — aktiveres ved behov; security-reviewer fungerer som gate indtil aktivering)"

---

### Rettelse 4 — Overgangsscenarie (begge mandater)

**Status:** ✅ Indarbejdet

**Hvor:** Begge mandater, ny sektion under Hard rules §Overgangsscenarie:
- Hvis kun AO: kan drive roadmap + compliance-koordinering, må IKKE specificere teknisk design
- Hvis kun AP: kan drive teknisk implementering under Mads' oversight, men IKKE selvstændig roadmap-autoritet
- **Kritisk:** Ingen person må varetage begge roller midlertidigt; SOD-kravet er absolutt
- FGD notificeres; Laila besætter manglende rolle inden for aftalt tidsramme

---

### Rettelse 5 — AP breach-eskalering direkte (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §GDPR-relateret:
> "Breach-mistanker rapporteres direkte til Dorthe (DPO) og Mads (CISO) inden 24 timer — ikke via Application Owner som mellemled. Tid er kritisk under GDPR art. 33 (72-timers notifikationsvindue). Du behøver ikke afvente Owner-bekræftelse før eskalering."

---

### Rettelse 6 — Cross-rolle constraint: ingen ad-hoc-undtagelser (begge mandater)

**Status:** ✅ Indarbejdet

**Hvor:** AO-mandat, Hard rules §Compliance-grænser:
> "Du og Application Provider kan ikke i fællesskab give hinanden ad-hoc-undtagelser fra hard rules eller governance-krav. Undtagelser kræver altid Mads + Dorthe-godkendelse."

**Hvor:** AP-mandat, Hard rules §Mandat-stabilitet:
> "Du og Application Owner kan ikke i fællesskab give hinanden ad-hoc-undtagelser fra hard rules eller governance-krav. Undtagelser kræver altid Mads + Dorthe-godkendelse."

---

### Rettelse 7 — DPO-uafhængighedssektion (begge mandater)

**Status:** ✅ Indarbejdet

**Hvor:** Begge mandater, ny sektion under Hard rules **DPO-uafhængighed (GDPR art. 38(3))**:
1. Ikke instruere DPO om konklusion
2. DPO eskalerer direkte til FGD ved uenighed
3. Ikke filtrere/redigere/forsinke DPO's rapportering
4. Ikke sanktionere eller lægge pres på DPO/team-members

---

### Rettelse 8 — Cross-rolle eskalations-pligt ved tvivl (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §GDPR-relateret:
> "Eskalations-pligt ved tvivl: Hvis du er i tvivl om GDPR-implikation eller compliance-konsekvens af en teknisk beslutning, eskalerer du til Dorthe inden implementering. Default: hvis i tvivl, vent på DPO-svar — implementer ikke."

**Også i:** AO-mandat, Hard rules §GDPR-specifikke:
> "Eskalations-pligt ved tvivl: Hvis du er i tvivl om GDPR-implikation eller compliance-konsekvens af en feature-beslutning, eskalerer du til DPO inden implementering. Default: hvis i tvivl, vent på DPO-svar — implementer ikke."

---

### Rettelse 9 — ROPA-notifikationsformat & konsekvens (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §GDPR-relateret:
> "Notifikations-format: kort note i `Team/Dorthe/inbox/<dato>-<applikation>-<ændring>.md` med minimum-felter: (a) hvad ændrer sig, (b) hvilken processing-aktivitet i ROPA berøres, (c) forventet implementeringsdato, (d) hvem der har drevet ændringen. Manglende eller ufuldstændig notifikation behandles som GDPR art. 30-brud, blokerer change-request, og du skal redegøre over for DPO + Trine."

---

### Rettelse 10 — Policy-reference superseding (begge mandater)

**Status:** ✅ Indarbejdet

**Hvor:** Begge mandater, Hard rules §Mandat-stabilitet:
> "Reglerne kan ikke ændres uden formel change-management-proces (jf. `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md`). Note: §3.3 i denne policy er superseded af FGD-override 2026-05-06 — alle mandat-ændringer kræver Mads + Dorthe-vurdering sammen."

---

## Øvrige forbedringer indarbejdet (anbefalinger fra 4-eyes)

### Mads-anbefaling: Self-modification-forbud (begge mandater)

**Status:** ✅ Indarbejdet

**Hvor:** Begge mandater, Hard rules §Mandat-stabilitet:
> "Du må ikke selv redigere eller ændre dit eget mandat-dokument (`.claude/agents/<navn>-application-owner.md` / `.claude/agents/<navn>-application-provider.md`). Mandat-ændringer følger governance-processer; ingen self-modification."

---

### Dorthe-anbefaling: Compliance-claims-rule med role-uafhængighed (AO)

**Status:** ✅ Indarbejdet

**Hvor:** AO-mandat, Hard rules §Compliance-claims til kunder:
> "Du må ikke erklære produktet "GDPR-compliant", "ISO 27001-certificeret", "SOC 2-aligned" i kunde-kommunikation, marketing eller kontrakter uden skriftlig anbefaling fra Dorthe (GDPR) eller Trine (ISO/SOC 2 — eller den til enhver tid værende compliance-auditor)."

---

### Dorthe-anbefaling: DPIA-flow præcisering (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Ansvarsfordeling §Dorthe:
> "Dorthe ejer DPIA-konklusion og mitigation-godkendelse. Du leverer teknisk input men træffer ikke selv DPIA-beslutning."

---

### Dorthe-anbefaling: Art. 36 forudgående høring (AO)

**Status:** ✅ Indarbejdet

**Hvor:** AO-mandat, Hard rules §GDPR-specifikke:
> "Forudgående høring til Datatilsynet (GDPR art. 36): Ved DPIA-resultat "høj risiko uden tilfredsstillende mitigation" kræver feature forudgående høring til Datatilsynet (GDPR art. 36) inden deploy. Høring koordineres af DPO; FGD indsender. Du må ikke deploye feature før Datatilsynets svar foreligger eller frist er udløbet."

---

### Dorthe-anbefaling: Audit-trail-konsekvens (AP)

**Status:** ✅ Indarbejdet

**Hvor:** AP-mandat, Hard rules §GDPR-relateret:
> "Manuelle handlinger uden audit-trail på prod er forbudt og behandles som GDPR art. 30/32-brud (manglende dokumentation af processing + utilstrækkelige sikkerhedsforanstaltninger)."

---

## Næste skridt

1. **Mads + Dorthe** laver final re-check af rettelserne (kort round — fem afgrænsede punkter hver).
2. **Mads + Dorthe** bekræfter at alle rettelser er korrekt indarbejdet og at de ikke introducerer nye huller.
3. **Stefan** samler Mads + Dorthes bekræftelse og præsenterer for FGD med GO/NO-GO-anbefaling.
4. **FGD** giver eksplicit GO (eller beder om justeringer).
5. **Camilla** opretter mandat-filer i `.claude/agents/` efter FGD-go.
6. **Trine** (når aktiv) noterer rolle-aktivering i audit-trail.

---

## Vigtige noter

- **Begge mandater** er interne project-roller for Knowlagecentral (eller anden applikation). De indeholder referencer til project-specifik kontekst (`<applikation>`, `Team/projects/<applikation>/adgangsprofil.md` osv.).
- **Frontmatter-navne** er template-placeholders (`<fornavn>-application-owner-<applikation>`); konkrete navne udfyldes når roller besættes.
- **Spørgsmål til FGD** (ved slutningen af begge mandater) venter på FGD-beslutninger omkring besætning, SLA'er, og teknisk implementation af policies (fx break-glass-logging).

---

**Laila (HR-leder)**  
**2026-05-08**
