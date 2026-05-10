---
dato: 2026-05-09
type: re-check
emne: FC2+FC4 stikprøve — FGDCorePKI v2.1
vurderer: Mads (CISO)
status: GO
input-dokumenter:
  - Team/Laila/2026-05-09-udkast-fgdcorepki-application-provider.md (v2.1)
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md (v2.1)
reference-final-check: Team/Mads/2026-05-09-mads-final-check-fgdcorepki-mandater.md
---

# Mads — Re-check: FC2+FC4 stikprøve (v2.1)

## FC2 — Break-glass PII: GO

AP-dokumentet indeholder nu et dedikeret afsnit "Break-glass for PII-bærende systemer" under Hard rules (linje 78-84). Afsnittet dækker præcis de systemer jeg krævede:

- CSR-database / certifikat-ansøger-data (X.509 Subject-felter med navn, e-mail, organisation) — eksplicit nævnt
- Audit-logs med PII (issuance-logs med Subject CN/SAN, authentication-historik med operatør-ID) — eksplicit nævnt

Alle fire procedurelle krav er til stede:
- Skriftlig begrundelse — ja
- CISO-godkendelse (Mads) — ja
- Automatisk audit-trail i WORM-format — ja
- Maksimal session 4 timer — ja

Trine auditerer break-glass-logs halvårligt er eksplicit nævnt. Derudover er der tilføjet et femte krav om Dorthe-notifikation inden 24 timer for GDPR art. 32-vurdering — dette rækker ud over mit minimumskrav og er i overensstemmelse med Dorthes GDPR-perspektiv.

FC2 er fuldt implementeret.

## FC4 — KCO-afgrænsning: GO

KCO-profilen (PKI-governance-roller §3, Specifikke opgaver, Ceremony-script-development) indeholder nu et dedikeret rolleafgrænsnings-afsnit (linje 158-160). Det specificerer eksplicit:

- AP leverer teknisk script-udkast (procedurerne for HSM-kommandoer, nøgleparametre, validerings-checkpoints) — ja
- KCO godkender proceduren som helhed for ceremoniuel sikkerhed og autoritet (kan ikke delegeres) — ja, KCO er proceduriel autoritet
- Mads godkender det færdige script inden ceremoni eksekveres — ja
- KCO kan holde ceremony tilbage ved uenighed om procedure-sikkerhed — ja
- Mads megler — ja

Afgrænsningen fremgår nu direkte af KCO-profilen selv. En auditor der læser kun KCO-profilen vil se den fulde rollefordeling. Dette opfylder mit krav om at afgrænsningen ikke blot var synlig i AP-dokumentet.

FC4 er fuldt implementeret.

## Bonus-check

**`audit.events`-reference:** Ingen forekomster af `audit.events` er til stede i AP-mandatets body. Terminologien er neutraliseret konsistent til "PKI audit-log" og "PKI audit-log med WORM-garanti". Changelogen linje 222 dokumenterer rettelsen korrekt.

**ETL "fase 1"-scenariet:** PKI-governance-roller §6 SOD-begrænsninger er opdateret. Formuleringen "Engineering Team Lead rapporterer til Application Provider (hvis AP er dedikeret) eller direkte til Mads (hvis ingen AP i fase 1)" er fjernet. Den er erstattet med "Engineering Team Lead rapporterer til Application Provider (teknisk mentor) og til Mads (compliance-oversight)." Rapport-linje-tabellen er opdateret tilsvarende. Det forældede fase-1-scenarie eksisterer ikke længere i dokumentet.

Begge bonus-rettelser er implementeret korrekt.

## Samlet: GO

Alle krav fra CONDITIONAL GO er opfyldt i v2.1:

- FC2 (break-glass for PII-bærende systemer): implementeret med korrekt scope og alle fire procedurelle krav plus Dorthe-notifikation
- FC4 (KCO-afgrænsning synlig i KCO-profilen): implementeret med fuld rollefordeling AP/KCO/Mads og KCO's ret til at holde ceremony tilbage
- Bonus: `audit.events`-terminologi neutraliseret; ETL fase-1-formulering fjernet

Mandaterne kan committes og aktiveres af Camilla efter FGD-GO.

**Forbehold videreføres fra final-check:** Dorthes GDPR-vurdering af CSR-databehandling og break-glass for PII-systemer er en separat forudsætning for aktivering. Mads koordinerer midlertidigt GDPR-vurderingen efter FGD-override 2026-05-06 indtil Dorthe er fuldt aktiveret. Stefan koordinerer FGD-GO.

**Næste handling:** Stefan præsenterer samlet GO til FGD. FGD giver eksplicit GO. Camilla committer alle tre mandater (AO + AP + PKI-governance-roller) i samme commit med reference til dette re-check-dokument. Trine logger commit-event i audit-trail.

---

*Vurderet af: Mads (CISO)*  
*Dato: 2026-05-09*
