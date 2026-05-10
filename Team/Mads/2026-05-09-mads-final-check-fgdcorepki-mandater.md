---
dato: 2026-05-09
type: final-check
emne: FGDCorePKI AO+AP+PKI-roller v2
vurderer: Mads (CISO)
status: CONDITIONAL GO
input-dokumenter:
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-owner.md (v2)
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md (v2)
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md (v2)
reference-4-eyes: Team/Mads/2026-05-09-mads-4eyes-fgdcorepki-mandater.md
---

# Mads — Final-check: FGDCorePKI mandater v2

## Samlet status: CONDITIONAL GO

Alle fem blockers (B1-B5) er reelt løst og FGDs beslutninger (Option B, intern PKI, ny HSM-operatør, delegeret AO) er konsistent indarbejdet i alle tre dokumenter. Det grundlæggende governance-design er forsvarligt og kan aktiveres.

To betingede rettelser er imidlertid **ikke korrekt implementeret**, på trods af at Lailas v1-til-v2-changelog hævder det modsatte. Det er et problem ikke blot teknisk, men for troværdigheden af Lailas claim-verifikation. Mandaterne må ikke committes og aktiveres af Camilla før disse to rettelser er på plads.

Derudover er der to redaktionelle fejl der ikke er rettet som lovet, og en forældet formulering i PKI-governance-roller der bør ryddes.

---

## Blocker-check (B1-B5)

| Blocker | Status | Note |
|---------|--------|------|
| B1 AP er ikke Tech Lead; ETL er separat rolle fra dag 1 | LOEST | AP-frontmatter beskriver korrekt arkitekt-rollen uden "Tech Lead". AP-body linje 151: "Du er ikke Tech Lead; Engineering Lead rapporterer tekniske fremskridt til dig." PKI-governance §6 har eksplicit "ADVARSEL: Denne rolle er SEPARAT fra Application Provider (Option B fra FGD)." Konsistent i alle tre dokumenter. |
| B2 FIPS-formulering afspejler non-FIPS MVP | LOEST | AP-mission §4 (linje 48): "FIPS 140-3 Level 3-ready (fase 0: non-FIPS MVP per FGDs 2026-05-09-beslutning; re-keying ved FIPS 140-3-cert fra NIST CMVP)." Hard rules linje 110: identisk formulering med tilføjet "Fase 0 er non-FIPS MVP" og Mads' status-tracking-ansvar. Korrekt på begge steder. |
| B3 Security Officer er selvstaendig rolle i PKI-governance-roller | LOEST | PKI-governance §5 (linje 239-283) er fuld profil med ansvar, kompetence-krav, SOD-begrænsninger, rapport-linje (Mads), model-anbefaling (sonnet), dual-control-autoritet, og fase-0-undtagelse (Mads temporaert). Normaliserings-deadline (inden lang-sigts-stabilisering) er dokumenteret. |
| B4 AO-observer-adgang ved key ceremony eksplicit forbudt | LOEST | AO linje 74: "Du deltager IKKE i key ceremony i nogen rolle — hverken som autoriserende part, observer eller witness. Enhver AO-tilstedeværelse ved ceremony er forbudt og behandles som SOD-brud." Anti-patterns linje 117 gentager forbud. |
| B5 Direkte eskalering forbi AO — Mads, Dorthe, Trine har eksplicit ret | LOEST | AO linje 95-96: "Mads, Trine og Dorthe kan eskalere compliance-fund direkte til FGD uden om dig. Du kan ikke blokere, forsinke eller omformulere compliance-rapportering fra disse roller." SOC 2 CC4.1-referece er til stede. |

---

## Betinget-check (C1-C4)

| Rettelse | Status | Note |
|----------|--------|------|
| C1 WORM/kryptografisk chaining i AP audit-trail | LOEST | AP linje 97: "WORM-garanti eller kryptografisk chaining (append-only, tamper-evident) for SOC 2 CC7.2-overhold." Break-glass linje 74 og 113 specificerer WORM og tamper-evident for break-glass-logs. SOC 2 CC7.2 er refereret. |
| C2 Break-glass daekker PII i CSR-data og audit-log | IKKE LOEST — se fund FC2 | Lailas changelog linje 195 hevder "Udvidet break-glass-procedure til at daekkePII-baerende systemer (CSR-database, audit-log-adgang)." Den paagaeldende tekst er ikke til stede i mandatets body. Se detaljeret fund nedenfor. |
| C3 HSM-operatoer-profil inkluderer HA-sync og failover | LOEST | PKI-governance HSM-operatoer §2 specifikke opgaver linje 107-108: "HA-sync: kvartalvis verifikation at wrap-key-replication til sekundaer YubiHSM2 er intakt og kan restore" og "Failover-test: halvaarlig test af failover fra primaer til sekundaer YubiHSM2 under Mads-opsyn; Trine attesterer." |
| C4 KCO er proceduriel autoritet; AP er teknisk procedure-forfatter — begge beskrevet | DELVIST LOEST — se fund FC4 | Kravet er fuldt opfyldt i AP-dokumentet (linje 117 og linje 165). KCO-profilen (§3 linje 158) har derimod kun den generiske formulering "Samarbejde med Application Provider + Mads". Rolleafgraensningen fremgaar ikke af KCO-profilen selv — kun af AP-dokumentet. Se detaljeret fund nedenfor. |

---

## Resterende fund

### FC2 — C2 ikke implementeret i AP-mandatets body (krav til rettelse inden commit)

Lailas changelog (linje 195) hevder:

> "C2: Udvidet break-glass-procedure til at daekke PII-baerende systemer (CSR-database, audit-log-adgang); tilfojet eksplicit note om CSR-data med personoplysninger."

Verificeret mod AP-dokumentets body: AP linje 74 omhandler udelukkende HSM-debugging ("Hvis du har brug for at debugge HSM-integration live"). AP linje 113 beskriver break-glass-procedurens trin for HSM-adgang.

Ingen steder i AP-mandatets body fremgaar det at break-glass-proceduren (eller en analog procedure med tilsvarende krav om begrundelse, Mads-godkendelse, automatisk logning og WORM-garanti) finder anvendelse ved direkte adgang til CSR-databasen, audit-log-laesning eller andre systemer der behandler PII fra Subject-felter.

Min 4-eyes-rettelse var specifik: "PII-baerende systemer (CSR-database, audit-log-laese-adgang) behandles analogt med HSM — skriftlig begrundelse, Mads-godkendelse, automatisk logning, Trine-audit."

Kraevet rettelse: Laila tilfojer i AP hard rules et afsnit (eller udvider break-glass-afsnittet) med ordlyd der eksplicit daekker PII-baerende systemers behandling analogt med HSM-break-glass. Koordineres med Dorthes GDPR-vurdering jf. min 4-eyes.

### FC4 — C4 mangler i KCO-profilen (bor rettes inden commit)

AP-dokumentet opfylder C4 korrekt: linje 117 (hard rules "Key Ceremony-design-koordination"): "Du designer ceremony-script teknisk; Key Ceremony Officer godkender proceduren; Mads godkender det faerdige script inden ceremoni." Linje 165 (hand-off-aftaler): "Du leverer teknisk script-udkast; KCO reviewes for procedureuafhaengighed; Mads godkender samlet script."

KCO-profilen (PKI-governance §3, linje 158) har derimod kun: "Samarbejde med Application Provider + Mads pa full step-by-step script for Root CA key generation; script-review og signering for ceremoni."

Rolleafgraensningen — at AP er teknisk forfatter, KCO er proceduriel autoritet, og Mads godkender — fremgaar ikke af KCO-profilens egen tekst. En auditor der laeskun KCO-profilen ser ikke afgransningen.

Kraevet rettelse: Laila tilfojer i KCO §3 under "Ceremony-script-development": "Application Provider leverer teknisk script-udkast; Key Ceremony Officer er eneste procedurelle autoritet for ceremony-scriptet som helhed; Mads godkender det faerdige script inden ceremoni eksekveres. KCO kan holde ceremony tilbage hvis der er uenighed om procedure-sikkerhed, indtil Mads megler."

### Redaktionelle resterende fejl (bor rettes inden commit)

Foelgende redaktionelle fejl er ikke rettet pa trods af Lailas changelog-pastand:

- AP linje 97: `audit.events` er ikke erstattet med neutral terminologi. Changelogen linje 214 hevder rettelsen er lavet, men `audit.events` staar stadig i mandatets body. Kraevet rettelse: erstat med "PKI audit-log" eller "FGDCorePKI audit-events".
- AP linje 144: "du designes arkitektur/integration" — "designes" er ikke rettet til "designer". Ansvarsfordelingstabellen har stadig fejlformen.
- AP linje 154 og 163: "Du designers ændring" og "Du designes procedure" — "designers" og "designes" er ikke rettet til "designer".
- AP linje 163: "Med CA Administrator: Du designes procedure" — ikke rettet.

### Forældet formulering i PKI-governance (redaktionel anbefaling)

PKI-governance §6 SOD-begrænsninger linje 296: "Engineering Team Lead rapporterer til Application Provider (hvis AP er dedikeret) eller direkte til Mads (hvis ingen AP i fase 1)."

Formuleringen "hvis ingen AP i fase 1" er forældet. FGD har besluttet at AP og ETL er to separate roller fra dag 1 (Option B). Der er ingen "fase 1 uden AP"-scenarie. Formuleringen bor rettes til: "Engineering Team Lead rapporterer til Application Provider (teknisk mentor) og til Mads (compliance-oversight)."

---

## Anbefaling til FGD

**CONDITIONAL GO — klar til aktivering efter to rettelser og redaktionel oprydning.**

De fem blockers fra min 4-eyes er alle løst og FGDs beslutninger er korrekt implementeret. Rolledesignet er forsvarligt.

To rettelser skal pa plads inden Camilla committer:

1. **FC2**: Laila tilfojer break-glass-analog for PII-baerende systemer (CSR-database, audit-log-laesning) i AP hard rules. Koordineres med Dorthes GDPR-vurdering.
2. **FC4**: Laila tilfojer rolleafgraensnings-tekst (AP = teknisk forfatter; KCO = proceduriel autoritet; Mads godkender) i KCO-profilen §3 under "Ceremony-script-development".

Yderligere rettes de redaktionelle fejl der stadig er tilstede i AP-dokumentet (audit.events-terminologi + verbformer).

Nar rettelserne er lavet, behoves ikke ny fuld 4-eyes — Mads godkender med stikprovekontrol af de paagaeldende afsnit, og Stefan kan koordinere FGD-GO.

**Forbehold:** Dorthes GDPR-vurdering af CSR-databehandling og break-glass for PII-systemer er en separat forudsaetning for aktivering (jf. min 4-eyes). Mads koordinerer midlertidigt GDPR-vurderingen efter FGD-override 2026-05-06 indtil Dorthe er fuldt aktiveret.

---

*Vurderet af: Mads (CISO)*
*Dato: 2026-05-09*
*Naeste handling: Laila retter FC2 + FC4 + redaktionelle fejl. Stefan koordinerer Dorthes GDPR-vurdering parallelt. Mads stikprovekontrol af de rettede afsnit. Herefter Stefan → FGD-GO → Camilla committer.*
*Trine logger dette review-event i audit-trail ved aktivering.*
