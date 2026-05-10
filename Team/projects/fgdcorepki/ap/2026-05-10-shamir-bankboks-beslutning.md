---
dato: 2026-05-10
fra: Application Provider (FGDCorePKI)
projekt: FGDCorePKI
type: beslutningsnote
emne: Shamir 5. share — depot-beslutning: bankboks
udløsende-beslutning: FGD-beslutning 2026-05-10 — 5. Shamir-share gemmes i bankboks (ikke ekstern advokat/notar)
reference-arkitektur: Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md
reference-dpia1-svar: Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md (§4 F9.2 key-hierarki ASCII-diagram)
status: beslutningsnote — input til CP/CPS og ceremony-procedure-dokumentation
---

# Shamir 5. share — depot-beslutning: bankboks

## 1. FGDs beslutning

**Beslutning truffet 2026-05-10**: Den 5. share af 3-of-5 Shamir Secret Sharing-splittet for ceremony-arkiv-key (CAK) deponeres i **bankboks** — ikke hos ekstern advokat eller notar som alternativt overvejet.

**Baggrund**: DPIA-1 svaret §4 F9.2 ASCII-diagram listede tentativt de 5 share-depoter:

1. KCO (HQ-safe)
2. CA Admin (HQ-safe-2)
3. HSM-operator (off-site safe)
4. Security Officer (bank-deposit)
5. **Ekstern juridisk depot** ← afløst af denne beslutning

FGDs beslutning er at den 5. (tidligere foreslået ekstern advokat/notar) i stedet placeres i bankboks. Konkret bankboks-tilknytning og opbevarings-form per shares er beskrevet i §3 nedenfor.

---

## 2. Tekniske implikationer for ceremony-procedure

### 2.1 Adgang til bankboks under recovery-ceremoni

Recovery-ceremonien kræver minimum 3 af 5 shares samles på air-gapped recovery-host (jf. DPIA-1 svar §4 F9.2 recovery-procedure step 1). Bankboks-share er én af de 5 — og er muligvis nødvendig hvis 3 af de andre 4 ikke kan samles.

**Adgangs-model**:

| Aspekt | Spec |
|---|---|
| **Hvem har adgangsret til bankboks** | **Mindst to navngivne personer** (dual-control); typisk Security Officer + KCO eller Security Officer + designeret stedfortræder. **Ingen enkelt-person må kunne åbne bankboksen alene.** |
| **Authentication til banken** | Banken's egne KYC-procedurer + bankens egne dual-signature-krav på boks-aftalen. Banken er ikke en del af FGDs Trust Anchor — banken giver fysisk adgang, ikke kryptografisk capability. |
| **Hvornår må bankboks-share udtages** | Kun under formel recovery-ceremoni godkendt af KCO + Security Officer + Mads (CISO); audit-logged før udtagning. Ad-hoc udtag forbudt. |
| **Genindsættelse af share** | Efter recovery-ceremoni-afslutning genindsættes share i bankboks under samme dual-control. Hvis recovery udløste CAK-rotation, destrueres gammel share og ny share genereres + indsættes (jf. F9-supplement §2.3 generation-skift-procedure). |
| **Kontroversielle scenarier** | Bankboks-konto-indehaver må ikke være enkelt-person; aftalen tegnes på FGD juridisk-enhed eller på dual-signatar-aftale — aldrig på personlig konto. |

### 2.2 Dual-control krav

**[AP-teknisk]** Dual-control gælder på flere lag:

1. **Bankens egen dual-signatur**: Aftalen med banken kræver to autoriserede underskrifter for åbning af boksen. Dette er **bankens** kontrol (uden for FGDs governance) men styrker SOD.
2. **FGDs interne dual-control**: Selv om kun én person teknisk kunne åbne (hvis bank-aftalen tillod det), kræver FGDs ceremony-procedure to navngivne personer fysisk tilstede ved bankboks-åbning.
3. **Pre-godkendelse**: Skriftlig godkendelse fra KCO + Mads (CISO) før bankboks-besøg foretages. Begrundelsen logges i ceremony-protokol og audit-trail.
4. **Vidne-attestation**: De to personer der åbner boksen attesterer hver især (signed) at sharen blev udtaget intakt og uden tegn på tampering. Tamper-evident kuvert eller seglet container anbefales (jf. §3 opbevarings-form).

**[Kræver Mads]** at godkende den endelige dual-control-procedure, herunder hvilke konkrete personer (rolle-niveau) der har bankboks-adgangsret.

**[Kræver KCO]** at indarbejde bankboks-procedure i ceremony-script for recovery-ceremonier, så det ikke håndteres ad hoc under tidspres.

### 2.3 Tids-konsekvenser for recovery-ceremoni

Bankens åbningstider og lokations-afstand påvirker recovery-ceremoni-planlægning:

| Faktor | Implikation |
|---|---|
| Bankens åbningstid | Recovery-ceremoni må planlægges inden for bankåbningstid, eller bankboks-share-tilgang prioriteres tidligt i recovery-vinduet. Akut recovery uden for åbningstid kræver alternative 3 shares. |
| Geografisk afstand | Hvis bankboks-lokation er langt fra recovery-host-lokation, planlægges hentning som separat dual-control-rejse — ikke "drop by". Audit-logget transport-protokol. |
| Transport-sikkerhed | Tamper-evident container under transport; share må ikke separeres fra dual-control-eskorte fra bankboks til recovery-host. |

**[AP-anbefaling]**: Recovery-ceremoni planlægges normalt så de 3 nemmest tilgængelige shares (KCO HQ-safe + CA Admin HQ-safe-2 + HSM-operator off-site safe) udgør threshold 3-of-5. Bankboks-share + Security Officer-share er **redundans** for tilfælde hvor en eller flere af de første 3 ikke kan tilgås (fysisk skade, kompromittering, manglende person).

### 2.4 Påvirkning af DPIA-1 §4 F9.2 ASCII-diagram

ASCII-diagrammet i hoveddokumentet skal opdateres ved næste revision af arkitektur-dokumentet:

```
Tidligere (DPIA-1 svar §4 F9.2):
  5. Ekstern juridisk depot

Nu (efter FGD-beslutning 2026-05-10):
  5. Bankboks (geografisk adskilt fra de øvrige 4 shares)
```

**[AP-teknisk]** AP bestiller opdatering af arkitektur-dokumentets §3.5 og DPIA-1-svar-diagram via standard change-management-flow (Camilla committer; commit-reviewer reviewer). Ingen ad-hoc edit.

---

## 3. Anbefaling til CP/CPS-dokumentation af depot-procedure

CP/CPS (Certificate Policy / Certification Practice Statement) er PKI-styringsdokument der formelt beskriver ceremony-procedurer og key-management. Bankboks-depot-proceduren skal CP/CPS-dokumenteres for audit-egnethed (SOC 2 CC6.1, ISO 27001 A.5.17 / A.8.24 / A.8.27).

**[AP-anbefaling]** CP/CPS-dokumentation skal indeholde:

### 3.1 Depot-spec

- **Location-type**: Bankboks i etableret bank-institution i EU (jf. L1 EU-only-residency)
- **Bank-juridisk identitet**: Konkret bank navngives i CP/CPS (ikke i offentligt-publiceret CP, men i internt CPS-bilag — sikkerhedsklassificeret)
- **Aftale-form**: Skriftlig boks-aftale tegnet på FGDs juridiske enhed (eller dual-signatar-aftale), aldrig på personlig konto
- **Geografisk lokation**: Skal være geografisk adskilt fra de øvrige 4 share-depoter (jf. §4 nedenfor)
- **Bank-dual-signature**: Aftalen kræver to autoriserede underskrifter for åbning

### 3.2 Adgangs-procedure

- **Pre-conditions for udtag**: Formel recovery-ceremoni-godkendelse fra KCO + Security Officer + Mads (CISO); skriftlig begrundelse arkiveret i audit-trail
- **Dual-control-deltagere**: To navngivne personer (rolle-niveau specificeret i CPS-bilag); ingen enkelt-person må have udtag-kapacitet
- **Tamper-evidence**: Share opbevares i tamper-evident kuvert eller seglet container; seglet inspiceres og dokumenteres ved hver åbning
- **Transport-protokol**: Direkte transport bankboks → recovery-host under fortsat dual-control-eskorte; audit-logget med start-/sluttider
- **Genindsættelse**: Efter ceremony-afslutning genindsættes share i ny tamper-evident kuvert; segl-ID logges
- **Ekstraordinær udtag-forbud**: Bankboks-share må ikke udtages til ad-hoc-formål, "test", "tjek", eller andet uden for formel recovery-ceremoni

### 3.3 Generation-skift-procedure

Når CAK-rotation udløser ny Shamir-split (jf. F9-supplement §2.3, ved 7-årig cadence):

1. Ny share-1...5 genereres på air-gapped host under formel ceremony
2. Bankboks-share udskiftes som del af samme ceremony eller som separat planlagt dual-control-tur
3. Gammel bankboks-share destrueres på recovery-host (ikke i banken — destruktionsceremoni må følge audit-procedure)
4. Ny bankboks-share placeres i ny tamper-evident kuvert; segl-ID logges; bankboks-aftale-protokol opdateres

### 3.4 Audit-evidens

- **Trine-audit halvårligt**: Trine verificerer at bankboks-aftalen er gyldig, at adgangsret-listen er aktuel, og at evt. udtag i perioden er fuldt audit-loggede. Trine besøger ikke selv bankboksen — verifikation sker via aftale-dokumenter, audit-log-eksport, og share-segl-ID-tracking.
- **SOC 2 CC6.1 evidens**: Bank-aftale-kopi (sikkerhedsklassificeret), adgangsret-liste, audit-log-eksport for alle udtag i perioden
- **ISO 27001 A.5.17 evidens**: Authentication-information-håndtering — share er authentication-information; depot-procedure er control-evidens
- **ISO 27001 A.8.24 evidens**: Use of cryptography — Shamir-share-management er del af crypto-key-management-praksis

### 3.5 Rest-risici (CP/CPS skal eksplicit nævne)

- **Bank-konkurs / ophør**: Hvis banken ophører, skal share evakueres og placeres i alternativ depot — formel governance-proces (Mads + FGD)
- **Bank-aftale-ændring**: Hvis banken unilateralt ændrer adgangs-betingelser (fx fjerner dual-signature-krav), reviewes aftalen ved næste halvårlige Trine-audit
- **Force-majeure**: Hvis bankboks-share ikke kan tilgås (fysisk skade, lockdown), tillader 3-of-5 threshold at recovery alligevel gennemføres med de øvrige 4 shares — bankboks-share er redundans, ikke single-point-of-failure

---

## 4. Opbevaring: Bankboks geografisk adskilt fra de øvrige 4 shares

**[AP-krav]** Bankboks-lokation skal være geografisk adskilt fra de fire øvrige share-depot-lokationer (KCO HQ-safe, CA Admin HQ-safe-2, HSM-operator off-site safe, Security Officer-depot — sidstnævnte er muligvis allerede en bankboks; jf. §4.1 nedenfor).

### 4.1 Klargøring af mulig overlap med Security Officer-depot

DPIA-1 svaret §4 F9.2 ASCII-diagram listede tentativt **share 4 = Security Officer (bank-deposit)**. Med FGD-beslutningen om at **share 5 nu også er bankboks**, skal det afklares:

- **Er det samme bankboks?** **Nej — det må det ikke være.** To shares i samme fysiske bankboks eliminerer redundans og skaber single-point-of-failure mod fysisk hændelse på lokationen.
- **Er det samme bank?** **Bør undgås.** Hvis begge shares er hos samme bank, deler de samme bank-konkurs-risiko og samme jurisdiktions-eksponering. **[AP-anbefaling]** To separate banker, ideelt to forskellige byer eller regioner.
- **[Kræver FGD]** Endelig afklaring af om share-4 (Security Officer) faktisk er bankboks, og om bank-institutionen er den samme som share-5. Hvis sammenfald: re-allokering af enten share-4 eller share-5 til alternativ depot-form (fx HSM-operator off-site safe-overlap-tjek, eller helt tredje lokation).

### 4.2 Geografisk adskillelse-spec

**[AP-anbefaling]** Konkret geografisk adskillelses-krav for samtlige 5 shares:

| Aspekt | Krav |
|---|---|
| **Minimum afstand** | Mindst 50 km mellem ethvert par af share-lokationer (eller alternativt: forskellige byer + forskellige strøm-/forsynings-grids) |
| **Forskellige bygninger** | Ingen to shares må være i samme fysiske bygning |
| **Forskellige adgangs-systemer** | KCO HQ-safe og CA Admin HQ-safe-2 er begge i HQ — disse må placeres i to separate sikrede rum med separate adgangs-systemer (PACS-zoner). De øvrige 3 shares (HSM-operator off-site, Security Officer-depot, bankboks) er på 3 separate lokationer i 3 forskellige byer eller regioner. |
| **Brand-/oversvømmelses-/jordskælvs-zone-forskellighed** | Ingen to shares må være eksponeret for samme single point of failure (samme datacenter-tier-event, samme storm-zone osv.) |

**[Kræver Mads]** at godkende endelig geografisk adskillelse-spec; AP's krav er minimum-baseline.

### 4.3 Audit-konsekvens af geografisk adskillelse

Trine's halvårlige audit skal verificere geografisk adskillelse — typisk via lokations-attest fra hver share-holder + opdateret share-depot-register. **[AP-anbefaling]** Share-depot-registret holdes i to versioner:

- **Detaljeret version**: Konkrete lokationer, bank-institution, kontakt-personer — sikkerhedsklassificeret, kun KCO + Mads + FGD-tilgang
- **Audit-evidens-version**: Bekræftelse af at de 5 shares er på 5 geografisk adskilte lokationer + adgangs-procedure, uden at afsløre konkrete adresser — Trine + ekstern auditor-tilgang

---

## 5. Åbne afhængigheder fra denne beslutning

| # | Input | Ejer | Påvirker | Frist |
|---|---|---|---|---|
| BD1 | Konkret bank-institution + lokation for bankboks | FGD + Security Officer | CP/CPS-bilag (sikkerhedsklassificeret) | Inden første ceremony |
| BD2 | Bekræftelse af om share-4 (Security Officer) og share-5 (bankboks) er forskellige institutioner og forskellige lokationer | FGD | F9.2 share-allokering finaliseres | Pre-ceremony |
| BD3 | Mads-godkendelse af geografisk adskillelse-spec (min. 50 km eller alternativ baseline) | Mads | Geografisk-distribution-proceduren | Pre-ceremony |
| BD4 | KCO indarbejder bankboks-procedure i ceremony-script for recovery-ceremoni | KCO | Ceremony-script + tør-kørsel | Pre-ceremony |
| BD5 | Camilla committer ASCII-diagram-opdatering i arkitektur-dokumentet (§3.5 og §9-relateret diagram) | AP designer; Camilla committer | Arkitektur-dokumentation up-to-date | Inden næste arkitektur-revision |
| BD6 | Trine-input om audit-evidence-format for share-depot-register | Trine | CP/CPS §3.4 audit-evidens-detaljer | Inden første halvårlige audit |

---

## 6. Sammenfatning

FGDs beslutning om bankboks som 5. share-depot er teknisk forsvarlig under følgende forudsætninger:

1. **Bankboks-lokation er geografisk adskilt** fra share-4 og de øvrige 3 share-depot-lokationer
2. **Dual-control gælder på flere lag**: bankens egen dual-signatur + FGDs interne dual-control + skriftlig pre-godkendelse fra KCO + Mads
3. **Share opbevares i tamper-evident kuvert** med segl-ID-tracking
4. **CP/CPS dokumenterer fuldt** depot-procedure, generation-skift, audit-evidens, rest-risici (jf. §3 ovenfor)
5. **Bankboks-aftalen** tegnes på FGDs juridiske enhed eller som dual-signatar-aftale — aldrig personlig konto

**[AP-anbefaling primær]**: Beslutningen kan eksekveres som beskrevet, betinget af lukning af afhængighederne BD1-BD6 inden første ceremony. AP designer opdateringerne; Mads + Dorthe vurderer compliance-impact; Camilla committer ændringerne via change-management-flow; KCO opdaterer ceremony-script.

---

**Forfatter**: Application Provider (FGDCorePKI)
**Dato**: 2026-05-10
**Status**: Beslutningsnote — input til CP/CPS, ceremony-script, og arkitektur-dokumentations-opdatering
**Distribution**: Application Owner, Mads (CISO), Dorthe (DPO awareness), Stefan; KCO (når aktiv); Camilla (forberedelse til arkitektur-doc-edit-commit); ekstern Trine (audit-readiness-input)
**Næste milestone**: Lukning af BD1-BD6; opdatering af arkitektur-dokumentets §3.5 og DPIA-1-svar-diagram via change-management-flow
