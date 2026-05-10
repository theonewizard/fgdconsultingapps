---
dato: 2026-05-10
fra: Dorthe (DPO)
til: Application Provider (FGDCorePKI)
emne: DPIA-1 F-spørgsmål — ceremony-recording
projekt: FGDCorePKI
type: dpia-input-anmodning
svarfrist: inden-key-ceremony
---

# DPIA-1 F-spørgsmål til Application Provider — ceremony-recording

## 1. Kontekst og formål

Velkommen ombord, AP. Tak for arkitektur-dokumentet leveret 2026-05-10 (`Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md`) — det indeholder allerede foreløbige tekniske præferencer på F1, F5 og F9 i §9. Denne anmodning beder om **formel og udvidet input** så jeg kan lukke DPIA-1 og levere proportionalitets-konklusion til FGD inden første Tier 1 Root key ceremony.

**Hvad er DPIA-1:** Fuld Data Protection Impact Assessment (GDPR art. 35) for behandlingsaktiviteten "Key ceremony video/lyd-optagelse" under FGDCorePKI Root + Intermediate CA-ceremonier. Screening-konklusionen (`Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ceremony-recording.md`) er at fuld DPIA er obligatorisk på grund af:

- **Art. 9-trigger:** Biometri (ansigt + stemme) anvendt til identifikations-formål (non-repudiation kræver entydig kobling til deltager)
- **Langtidsbevaring:** Per nuværende mandat-tekst minimum 20 år WORM-arkivering — overstiger PKI-baseline (7-10 år)
- **Datakombination:** Recording + ceremony-protokol + HSM-audit-log + smartcard-ID giver komplet identifikations-evidens over år

DPIA-1 skal afsluttes **inden første Tier 1 ceremony** (jf. AP's §7.3 milestone 1 + §7.3 milestone 2). Hvis DPIA ikke er konkluderet, skal første ceremony enten udskydes eller afholdes som tør-kørsel uden optagelse — sidstnævnte opfylder ikke non-repudiation-formålet.

**Hvorfor AP er essentiel input-kilde:** AP ejer den tekniske arkitektur-vurdering bag recording-form, arkiv-lokation, og krypterings-stak. Dorthe ejer GDPR-konklusionen, men kan ikke vurdere proportionalitet eller mitigation-design uden AP's tekniske grundlag.

---

## 2. F-spørgsmål adresseret til AP

Nedenstående tre F-spørgsmål fra screeningen har AP som primær eller co-ejer. AP har allerede leveret foreløbige præferencer i arkitektur-dokumentets §9 — disse anerkendes som udgangspunkt; spørgsmålene herunder beder om **udvidet detalje, alternativ-vurdering, og formaliseret begrundelse**.

### F1 — Recording-form (co-ejet med KCO + Dorthe)

**Screening-spørgsmål:** Skal optagelsen omfatte fuld video (ansigt synligt) eller alternativ form (audio + screen + hænder uden ansigt; ansigtsmasking; kun screen-capture)?

**AP's foreløbige præference (arkitektur §9):** Screen-capture + audio + hænder uden ansigt for almindelige ceremonier; fuld video kun hvis non-repudiation-formålet eksplicit kræver entydig ansigtsidentifikation.

**Formaliseret AP-input ønskes:**

1. **Teknisk forsvarbarhed pr. recording-variant:** For hver af følgende fire varianter — vurder om non-repudiation-evidens-værdien holder, og hvad det betyder for forensisk rekonstruktion ved senere mistanke om ceremony-kompromittering:
   - (a) Fuld video (ansigt + stemme + hænder + skærm)
   - (b) Audio + skærm + hænder uden ansigt (AP's foreløbige præference)
   - (c) Audio + skærm + ansigt-blurret video (post-processed)
   - (d) Kun skærm + audio (ingen krops-/ansigts-video)
2. **Differentiering pr. ceremony-type:** Skal recording-form variere mellem Root key generation (mest kritisk), Intermediate-signing, Tier 1 CRL re-issuance, og Root re-keying? AP's arkitektur-dokument tabel i §1.2 nævner alle fire som JA-recording — er der argumenter for differentiering?
3. **Witness-rolle vs. operatør-rolle:** Skal alle deltagere optages ens, eller er der argumenter for at differentiere mellem KCO/CA Admin/HSM-operatør (aktive handlinger) og witnesses (passive bevidnere)?
4. **Tekniske grænser:** Hvilke recording-varianter er teknisk realistiske at producere under selve ceremonien uden at forstyrre dual-control-procedure? (Skærm-capture er trivielt; multi-kamera + selektiv blurring kræver post-processing-pipeline med yderligere chain-of-custody-implikationer.)

### F5 — Arkiveringslokation (co-ejet med Mads)

**Screening-spørgsmål:** On-prem WORM (EU-residency) eller cloud-immutable (kræver tredjelands-vurdering hvis ikke EU)?

**AP's foreløbige præference (arkitektur §9):** On-prem WORM EU-residency primær; sekundært EU-cloud immutable (Scaleway, Hetzner, Stackit object storage med EU-region-pinning og immutability/Object-Lock); tredjelands-overførsel udelukket per L1.

**Formaliseret AP-input ønskes:**

1. **EU-only-konsistens med L1:** Bekræft at recording-arkiv inherits den samme EU-only-residency som OCSP/CRL-stakken (jf. AP §4.1 + L1). Skal der være en eksplicit LÅST forudsætning på linje med L1 også for ceremony-recording?
2. **Konkret leverandør-shortlist:** Hvilke 2-3 EU-leverandører anbefaler AP konkret for cloud-immutable-storage-optionen (med immutability/Object-Lock + dokumenteret EU-region-pinning + DPA tilgængelig)? Dette skal jeg bruge til DPA-tracker-input.
3. **On-prem WORM-platform:** Hvilken on-prem-platform anbefaler AP (NetApp SnapLock, Dell ECS, Cohesity, dedikeret WORM-tape, eller andet)? Kobling til WORM-arkivet beskrevet i AP §5.2 — er ceremony-recording-arkivet samme fysiske infrastruktur, eller separat?
4. **Geografisk distribution:** Skal der være redundans-kopi i sekundær EU-lokation (geografisk adskilt fra primær), og i så fald — hvordan håndteres synkronisering uden at bryde immutability-garantien?
5. **Sletning ved retention-udløb:** Når retention udløber (afventer F2-konklusion), hvordan eksekveres sletning teknisk? WORM-platforme har ofte "compliance-mode" der ikke tillader sletning før hard-coded retention-grænse — er platformen valgt sådan at retention kan justeres ved DPIA-konklusion uden at kræve platform-skifte?

### F9 — Krypterings-arkitektur (co-ejet med Mads + HSM-operatør)

**Screening-spørgsmål:** Hvilke nøgler, hvor opbevares de, hvordan håndteres key-rotation over 20 år?

**AP's foreløbige præference (arkitektur §9):** AES-256-GCM file-encryption; symmetrisk file-key wrappet med ceremony-arkiv-key (separat Domain på YubiHSM2 eller separat HSM-instance); arkiv-key roteres hvert 5. år; gamle keys revokeres men slettes ikke; 3-of-5 Shamir-split for arkiv-key.

**Formaliseret AP-input ønskes:**

1. **HSM-placering:** Skal ceremony-arkiv-key bo på samme YubiHSM2-fleet som CA-keys (separat Domain), eller på en dedikeret separat HSM-instance? Argumenter for/imod blast-radius-reduktion.
2. **Key-hierarki:** Tegn det fulde nøgle-hierarki for recording-arkivet:
   - File-encryption-key (per recording-fil eller per ceremony?)
   - Wrap-key (ceremony-arkiv-key)
   - Wrap-key-rotation-fane (gamle versioner bevaret hvor og hvordan?)
   - Recovery-procedure (hvis arkiv-key tabes — hvilken Shamir-share-rekonstruktion?)
3. **20-års key-rotation-strategi:** Hvis F2 konkluderer 20-år retention, beskriv to alternativer:
   - (a) "Bevarings-strategi": gamle wrap-keys behold som revokeret men accessible; nye recordings krypteres med nyt wrap-key; over 20 år kan op til 4-5 wrap-key-generationer være levende
   - (b) "Re-encryption-strategi": hvert 5-10 år dekrypteres + re-krypteres alle historiske recordings med nyeste wrap-key; gamle wrap-keys destrueres efter re-encryption verificeret
   - Hvilken anbefaler AP, og hvilke risici (tab-af-adgang vs. tab-af-konfidentialitet) prioriteres?
4. **Cipher-agility:** Hvis AES-256-GCM kompromitteres eller opgraderes til ny cipher (PQC-relevant) inden 20 år udløber, kan arkitekturen håndtere algorithm-skifte? Hvad er omkostnings-estimatet?
5. **Sammenhæng med AP §5.2 break-glass-arkiv:** Er ceremony-recording-arkivet teknisk identisk med break-glass-arkivet (samme platform + samme key-hierarki), eller separat infrastruktur? Argumenter for/imod konsolidering.
6. **Auditor-access (Trine):** Hvordan får Trine read-access til recordings ved kvartalvis review — uden at give nogen enkelt person dekryptering-mulighed udenfor dual-control?

---

## 3. Hvad svarene bruges til

Mine svar bruges til **DPIA-1 mitigation- og proportionalitets-vurdering**:

| Anvendelse | F-spørgsmål | DPIA-konsekvens |
|---|---|---|
| Proportionalitetsvurdering art. 5(1)(c) data-minimering | F1 | Hvis (b) eller (c) opfylder non-repudiation, må fuld video (a) anses uproportional |
| Proportionalitetsvurdering art. 5(1)(e) opbevaringsbegrænsning | F1 + F5 + F9 | Differentieret retention (biometri-tunge dele kortere end protokol-dele) afhænger af om recording-form kan splittes teknisk |
| Risikovurdering for registrerede | F5 + F9 | EU-residency + krypterings-styrke + key-management-kvalitet bestemmer rest-risiko-niveau |
| Mitigation-design | F1 + F5 + F9 | Konkret mitigation-katalog i DPIA-rapport (kapitel 6) |
| Art. 32-vurdering (security of processing) | F5 + F9 | Krypterings-arkitektur skal vurderes "appropriate" jf. risiko |
| Art. 36 trigger-vurdering | Alle tre | Hvis rest-risiko er høj, skal Datatilsynet konsulteres (8-uge tilsyns-svartid + buffer) |
| Lawful basis (art. 9-undtagelse) | F1 + F8 | Hvis F1 reducerer biometri-eksponering, kan art. 9-undtagelse evt. baseres på art. 9(2)(g) (væsentlig samfundsinteresse) hvor art. 9(2)(a) (eksplicit samtykke) ellers ville være primær |

**Konsekvens for FGDCorePKI-projektet:** Uden AP-svar på F1/F5/F9 kan jeg ikke konkludere DPIA-1. Uden DPIA-1-konklusion kan første Tier 1 Root key ceremony ikke afholdes (jf. AP §7.3 milestone 1 + §7.3 milestone 2). DPIA-1 er på den kritiske sti for projektets Fase 0 produktions-go-live.

---

## 4. Svarformat og svarkanal

**Svarkanal:** Skriv svar til `Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md`

**Frontmatter (skabelon):**

```yaml
---
dato: <udfyldes>
fra: Application Provider (FGDCorePKI)
til: Dorthe (DPO)
emne: DPIA-1 F-spørgsmål — AP-svar
projekt: FGDCorePKI
type: dpia-input-svar
reference: Team/projects/fgdcorepki/ap/2026-05-10-dorthe-dpia1-f-spoergsmaal.md
status: udkast | endeligt
---
```

**Body-struktur:**

1. **Kort sammenfatning** (3-5 linjer) — har præferencerne fra arkitektur §9 ændret sig efter dybere analyse?
2. **F1 — Recording-form** — svar på alle 4 underspørgsmål; tabel-form hvor det giver mening
3. **F5 — Arkiveringslokation** — svar på alle 5 underspørgsmål; konkret leverandør-shortlist for cloud-option
4. **F9 — Krypterings-arkitektur** — svar på alle 6 underspørgsmål; ASCII-diagram for nøgle-hierarki
5. **Åbne afhængigheder** — input AP afventer fra Mads/HSM-operatør/Engineering Lead inden endelig svar kan låses
6. **AP-anbefaling pr. F-spørgsmål** — én sætning per F: "AP anbefaler primært variant X med begrundelse Y"

**Detaljerings-niveau:** Teknisk dybde svarende til arkitektur-dokumentets §2 og §5 — Dorthe skal kunne citere AP-vurdering direkte i DPIA-rapporten uden yderligere oversættelse.

**Sprog:** Dansk i body, engelsk for tekniske termer hvor det er konventionen (Object Lock, immutability, wrap-key, Domain, Capabilities, Shamir Secret Sharing).

---

## 5. Svar på alle tre F-spørgsmål er nødvendige

**Alle tre F-spørgsmål (F1, F5, F9) skal besvares før DPIA-1 kan konkluderes.** Delvise svar (f.eks. kun F1 + F5) er ikke tilstrækkelige fordi:

- F1 (recording-form) bestemmer biometri-volumen → bestemmer hvilken art. 9-undtagelse der er proportional
- F5 (arkiv-lokation) bestemmer art. 32-sikkerheds-vurdering og DPA-shortlist
- F9 (kryptering) bestemmer både art. 32-styrke og 20-års-retention-realiserbarhed; uden F9 kan retention-anbefalingen i F2 (Mads-ejet) ikke bero på solid teknisk fundament

**Frist:** "inden-key-ceremony" — konkret deadline kan AO + Mads + Engineering Team Lead fastsætte i fælles roadmap, men jeg har brug for AP-svar **mindst 3 uger før første ceremony** for at kunne:

1. Konkludere DPIA-1 (1-2 uger efter AP-svar)
2. 4-eyes-review af DPIA (KCO + Mads + AP) — 1 uge
3. Eventuel Datatilsyns-konsultation (art. 36) — 8 uger + buffer hvis høj rest-risiko

Hvis AO fastsætter ceremony-dato, send mig en heads-up så jeg kan bagudregne min DPIA-deadline.

**Hvis AP ikke kan svare på et delspørgsmål** (f.eks. fordi det afhænger af Engineering Lead eller HSM-operatør): meld den åbne afhængighed eksplicit i §5 af svar-filen, så jeg kan trække den ind i DPIA's "rest-risiko"-sektion eller eskalere via Stefan.

---

## 6. Sammenhæng med øvrige DPIA-1 F-spørgsmål

Til kontekst — disse F-spørgsmål er adresseret til andre ejere og påvirker ikke AP direkte, men nævnes så AP forstår hele billedet:

| F# | Adresseret til | Ejer-afgørelse |
|---|---|---|
| F2 — 20-år retention vs. WebTrust-baseline | Mads + AO | Tilbageviser til AP for teknisk realiserbarhed (F9 input) |
| F3 — Lawful basis pr. deltagerkategori | Dorthe + AO + HR | Påvirker hvilken art. 6-grund der skal dokumenteres |
| F4 — Art. 9-undtagelse | Dorthe (vurdering) | Afhænger af F1 (biometri-volumen) |
| F6 — Adgangskontrol til arkivet | Mads + Trine | Operationel adgangsstyring; AP leverer teknisk kontrol-mulighed |
| F7 — Data-subject-rettigheder (art. 15/17) | Dorthe | Påvirker information-template og sletning-procedure |
| F8 — Information til deltagere (art. 13/14) | Dorthe + KCO | Skal modtage information før samtykke kan gives |

Dorthes egne F-spørgsmål (F4 + F7) afventer F1 + F9 fra AP samt F2 fra Mads/AO.

---

## 7. Næste skridt

1. AP læser denne anmodning og arkitektur-dokumentets §9 igen
2. AP identificerer interne afhængigheder (Mads, HSM-operatør, Engineering Lead) og koordinerer
3. AP skriver svar til `Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md`
4. Dorthe læser svar; ved uklarheder skrives opfølgningsspørgsmål til samme inbox-fil eller ny fil
5. Dorthe konkluderer DPIA-1 og distribuerer til 4-eyes-review (KCO + Mads + AP)
6. Hvis høj rest-risiko: Dorthe → FGD via Stefan → eventuel Datatilsyns-konsultation

**Spørgsmål om denne anmodning** kan AP stille via Stefan eller direkte tilføje til svar-filen som §6 "Spørgsmål til Dorthe".

---

**Forfatter:** Dorthe (DPO)
**Dato:** 2026-05-10
**Status:** Anmodning afsendt — afventer AP-svar i `Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md`
**Næste milestone:** DPIA-1-konklusion 1-2 uger efter AP-svar lukket
