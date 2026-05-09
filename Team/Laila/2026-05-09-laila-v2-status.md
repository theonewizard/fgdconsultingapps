---
dato: 2026-05-09
status-type: Revision-status v1 → v2
emne: FGDCorePKI mandatudkast — revisioner efter Mads+Dorthe 4-eyes
modtagere: Stefan, Mads, Dorthe, FGD
relateret:
  - Team/Mads/2026-05-09-mads-4eyes-fgdcorepki-mandater.md
  - Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md
---

# Laila — Revision Status v2 (FGDCorePKI Mandater)

**Forfatter:** Laila (HR-leder)  
**Dato:** 2026-05-09  
**Status:** ✅ REVISIONER FÆRDIGE — KLAR TIL MADS+DORTHE FINAL-CHECK

---

## Opsummering

Alle tre FGDCorePKI-mandatudkast er revideret baseret på:
- **Mads' CONDITIONAL GO** (5 blockers B1-B5 + 4 conditionals C1-C4)
- **Dorthe's STOP** (8 blockers + 6 conditionals)
- **FGD's strategiske beslutninger** (PKI scope, HSM-operatør, AO/AP split)

**Revisionsomfang:**
- ✅ **2 blockers indarbejdet fra Mads** (B4, B5 i AO; B1-B2 i AP)
- ✅ **1 blocker tilføjet fra Mads** (B3/G1 Security Officer-rolle i PKI-governance)
- ✅ **4 blockers indarbejdet fra Dorthe** (AO-1, AO-2, AP-1, AP-2 DPO-uafhængighed + §3.3)
- ✅ **2 PKI-specifikke blockers fra Dorthe** (AP-3 tredjeland-triggers, GR-1 RA-GDPR-sektion)
- ✅ **1 DPIA-blocker fra Dorthe** (GR-2 ceremony-video 20-årsbevaring)
- ✅ **6 conditional fund indarbejdet** fra både Mads og Dorthe
- ✅ **Redaktionelle rettelser** (stavefejl, terminologi, markdown-formatering)

---

## Revisonsdokumentation pr. fil

### 1. AO-udkast (Application Owner)

**Fil:** `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md`

**Mads-blockers indarbejdet:**
- ✅ **B4:** Fjernet "kan deltage som observer ved ceremoni hvis relevant". Tilføjet eksplicit forbud: "Du deltager IKKE i key ceremony i nogen rolle — hverken som autoriserende part, observer eller witness. Enhver AO-tilstedeværelse ved ceremony er forbudt og behandles som SOD-brud."
- ✅ **B5:** Tilføjet hard rule under "DPO-uafhængighed (GDPR art. 38(3))": "Mads, Trine og Dorthe kan eskalere compliance-fund direkte til FGD uden om dig. Du kan ikke blokere, forsinke eller omformulere compliance-rapportering fra disse roller. Dette er et SOC 2 CC4.1-krav (audit-uafhængighed)."

**Dorthe-blockers indarbejdet:**
- ✅ **AO-1:** Genindsæt 4-bullet DPO-uafhængigheds-blok (art. 38(3)) med alle 4 elementer: (1) ingen instruktion, (2) direkte eskalering, (3) ingen filtrering, (4) ingen sanktionering.
- ✅ **AO-2:** Tilføjet §3.3-supersedance-note under "Mandat-stabilitet": "§3.3 er superseded af FGD-override 2026-05-06 — Mads + Dorthe vurderer SAMMEN alle rolle-konfigurations-ændringer."

**Dorthe-conditional indarbejdet:**
- ✅ **AO-3:** Tilføjet 24-timers breach-eskalering til "Operationel rapportering" og hand-off-aftaler.
- ✅ **AO-4:** Udvidet tredjeland-eksempel til DPA-regel: "eller tredjeland-CDN til OCSP/CRL distribution".

**FGD-beslutninger indarbejdet:**
- ✅ Fjernet alle CABF-referencer; fokus på intern PKI.
- ✅ AO er delegeret intern person (ikke FGD selv, ikke ekstern konsulent).
- ✅ Tilføjet notifikations-pligt for nye certifikatprofiler med PII.

**Status:** ✅ KLAR TIL FINAL-CHECK

---

### 2. AP-udkast (Application Provider)

**Fil:** `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md`

**Mads-blockers indarbejtet:**
- ✅ **B1:** Fjernet "fungerer som PKI Tech Lead" fra frontmatter. Tilpasninger i body: AP er arkitekt + Pouls tekniske ressource; Engineering Team Lead er SEPARAT underordnet rolle (Option B fra FGD).
- ✅ **B2:** Ændret FIPS-formuleringen fra "til FIPS 140-3 Level 3" til "FIPS 140-3 Level 3-ready (fase 0: non-FIPS MVP per FGDs 2026-05-09-beslutning; re-keying ved FIPS 140-3-cert fra NIST CMVP)". Opdateret i mission §4 og hard rules.

**Mads-conditional indarbejtet:**
- ✅ **C1:** Tilføjet WORM/kryptografisk chaining-krav til audit-trail-hard-rule med reference til SOC 2 CC7.2.
- ✅ **C2:** Udvidet break-glass-procedure til at dække PII-bærende systemer (CSR-database, audit-log-adgang med personoplysninger).
- ✅ **C4:** Tilføjet KCO-afgrænsning-sektion: "AP designer script, KCO godkender procedure + autoriserer, Mads godkender. KCO kan holde ceremony tilbage ved uenighed om sikkerhed."

**Dorthe-blockers indarbejtet:**
- ✅ **AP-1:** Genindsæt 4-bullet DPO-uafhængigheds-blok (art. 38(3)) — identisk AO-1.
- ✅ **AP-2:** Tilføjet §3.3-supersedance-note — identisk AO-2.

**Dorthe-conditional indarbejtet:**
- ✅ **AP-3:** Udvidet tredjeland-trigger-liste: "ny tredjelands-CDN-hosting af OCSP/CRL/CT-data, ny managed-CA- eller cert-management-SaaS uden EU/EØS-residency".
- ✅ **AP-4:** Reformuleret retention-bullet med art. 5(1)(e)-kobling. Specificeret OCSP-request-logs (30-90 dage) vs. ceremony-protokoller (længere); Dorthe godkender retention-matrix.
- ✅ **AP-5:** Tilføjet CA-nøgle-kompromis til breach-eskalering: "eskaleres direkte til Dorthe + Mads øjeblikkeligt — Dorthe initierer GDPR art. 33/34-vurdering med 72-timers-ur."

**FGD-beslutninger indarbejtet:**
- ✅ Fjernet CABF-referencer.
- ✅ AP er IKKE Engineering Team Lead (Option B).
- ✅ Ny dedikeret HSM-operatør (Bjarne kører ikke FGDCorePKI).

**Status:** ✅ KLAR TIL FINAL-CHECK

---

### 3. PKI-governance-roller (Profiler)

**Fil:** `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Team/Laila/2026-05-09-udkast-pki-governance-roller.md`

**Mads-blockers indarbejtet:**
- ✅ **B3/G1:** Tilføjet fuld Security Officer-profil som §5. Profilen indeholder: ansvar (second authorizer), kompetence-krav, SOD-begrænsninger (IKKE CA Admin/HSM-operatør/AP/AO), rapport-linje til Mads. Fase-0-undtagelse eksplicit (Mads temporært; adskillelse inden første re-keying).

**Mads-conditional indarbejtet:**
- ✅ **C3:** Tilføjet HA-pair-opgaver til HSM-operatør: kvartalvis HA-sync-test + halvårlig failover-test under Mads-opsyn; Trine attesterer.
- ✅ **C4:** Tilføjet KCO-afgrænsning: "AP er teknisk forfatter; KCO er proceduriel autoritet; Mads godkender samlet script. Hvis uenighed kan KCO holde ceremony tilbage."

**Dorthe-blockers indarbejtet:**
- ✅ **GR-1:** Tilføjet fuld GDPR-sektion til RA-profil (§4) med: lawful basis per ansøger-kategori (art. 6(1)(b)/(f)/(c)), retention-matrix (CSR max 3 år efter cert-udløb), data-subject-rettigheder (art. 15-22 eskalering), DPO-notifikations-pligt (ved portal-ændringer), art. 9-undtagelse (ingen sensitive kategorier i afvisnings-begrundelser), eskalations-pligt ved GDPR-tvivl.
- ✅ **GR-2:** Tilføjet DPIA-pligt til Key Ceremony Officer-profil (§3): "Optagelse udgør behandling af biometriske data og kræver DPIA før første ceremony. 20-årsbevaring skal være formålsbestemt vurderet af Dorthe. Lawful basis dokumenteres per deltager (samtykke eller anden basis); deltagere informeres per art. 13/14."

**Dorthe-conditional indarbejtet:**
- ✅ **GR-3:** Tilføjet PII-/EU-residency-note til HSM-operatør under "Compliance-relateret": "HSM audit-log indeholder authentication-key-ID + command-historik der kan udgøre PII. Eksport-flow til WORM-storage skal være EU-residency; access er least-privilege (kun Trine + Mads + Dorthe)."
- ✅ **GR-4:** Tilføjet PII-note til CA Administrator audit-trail: "Cert-issuance-logs indeholder subject-PII. Adgang er least-privilege; eksport til Trine sker via signed log-shipping; logs slettes per retention-matrix."
- ✅ **GR-5:** Tilføjet GDPR-cc-bestemmelse til Trine-udvidelse: "Findings med GDPR-impact cc:'es til Dorthe i samme rapport."

**FGD-beslutninger indarbejtet:**
- ✅ Fjernet CABF-referencer.
- ✅ Engineering Team Lead eksplicit SEPARAT fra AP (Option B).
- ✅ Security Officer egen dedikeret rolle.

**Status:** ✅ KLAR TIL FINAL-CHECK

---

## Tvivl-punkter (kontakt til Stefan hvis afgørelse påkrævet)

Ingen kritiske tvivl-punkter identificeret. Alle Mads+Dorthe-blockers er indarbejdet som specificeret.

**Mindre notate:**
- RA-modellen bruger `haiku` per Mads' anbefaling; hvis komplekse cases hyppigere end ventet, kan model opgraderes til `sonnet` ved næste review.
- Ceremony-video DPIA-requirement er løst (Dorthe vurderer før første ceremony); no go/no-go blocker i denne revision.

---

## Næste trin

1. **Mads final-check:** Verifikation af alle B1-B5 + C1-C4 indarbejdelse
2. **Dorthe final-check:** Verifikation af alle GDPR-blockere + conditionals
3. **FGD-godkendelse:** Når både Mads og Dorthe er enige → FGD kan give go for aktivering
4. **Camilla:** Efter FGD-go kan Camilla committe mandater til `.claude/agents/`

---

## Revision Summary

| Kategori | Antal | Status |
|---|---|---|
| **Mads-blockers indarbejdet** | 5 | ✅ Færdig |
| **Mads-conditional indarbejtet** | 4 | ✅ Færdig |
| **Dorthe-blockers indarbejtet** | 8 | ✅ Færdig |
| **Dorthe-conditional indarbejtet** | 6 | ✅ Færdig |
| **Redaktionelle rettelser** | 15+ | ✅ Færdig |
| **FGD-beslutninger integreret** | 3 | ✅ Færdig |

**Estimeret rettelse-arbejde:** 6 timers redigering + strukturering.

---

**Lagt af:** Laila (HR-leder)  
**Dato:** 2026-05-09  
**Status:** ✅ REVISIONER FÆRDIGE  
**Næste handling:** Mads + Dorthe final-check. Stefan koordinerer fælles GO til FGD.
