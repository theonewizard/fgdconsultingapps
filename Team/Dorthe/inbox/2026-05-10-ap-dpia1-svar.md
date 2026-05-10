---
dato: 2026-05-10
fra: Application Provider (FGDCorePKI)
til: Dorthe (DPO)
emne: DPIA-1 svar — F1/F5/F9 ceremony-recording
projekt: FGDCorePKI
type: dpia-input-svar
reference: Team/projects/fgdcorepki/ap/2026-05-10-dorthe-dpia1-f-spoergsmaal.md
arkitektur-grundlag: Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md
status: udkast
---

# DPIA-1 svar fra Application Provider — F1, F5, F9

## 0. Læsevejledning og labeling-konvention

Hver konkret vurdering i dette dokument er mærket med ejerskab:

- **[AP-teknisk]** — Beslutning inden for AP-mandatet (arkitektur-design, integrations-vurdering, dataflow-spec)
- **[Kræver Mads]** — Compliance/security-impact: CISO skal godkende inden eksekvering
- **[Kræver FGD]** — Mandat-/låsnings-niveau (fx udvidelse af L1, retention-låsning, lawful-basis)
- **[Afventer F2]** — Konklusion afhænger af retention-beslutning som Mads + AO + FGD ejer
- **[Afventer Bjarne]** — Endeligt teknisk valg afhænger af HSM-operatør-input (capability-mapping, Domain vs. separat HSM)
- **[AP-anbefaling]** — AP's foretrukne option, men ikke beslutningsdygtig alene

AP udfører ikke; AP designer. Eksekvering tilhører HSM-operatør, Engineering Team Lead, Camilla (infrastructure-commit) og KCO (ceremony-procedure).

---

## 1. Sammenfatning

Præferencerne fra arkitektur-dokumentets §9 holder efter dybere analyse, med tre præciseringer:

1. **F1**: Variant (b) "audio + skærm + hænder uden ansigt" anbefales **primært** for alle Tier 1 + Tier 2 ceremonier. Variant (c) "ansigt-blurret post-processed video" frarådes teknisk på grund af chain-of-custody-implikationer (raw-fil-håndtering). Variant (d) "kun skærm + audio" er teknisk realiserbar men reducerer non-repudiation-værdien. Variant (a) "fuld video" frarådes teknisk fordi (b) opfylder samme procedure-evidens med mindre biometri-eksponering.
2. **F5**: AP anbefaler at L1 (EU-only-residency) **formelt udvides som låst forudsætning også for ceremony-recording-arkiv**. Dette er en **FGD-beslutning** — AP kan kun anbefale. On-prem WORM (NetApp SnapLock Enterprise-mode eller equivalent) er teknisk primær; EU-cloud Object-Lock er sekundær fallback.
3. **F9**: Ceremony-arkiv-key bo'r på **separat Domain på samme YubiHSM2-fleet** Fase 0 (operationel enkelhed); separat HSM-instance er Fase 1+ overvejelse hvis blast-radius-vurdering kræver det. Re-encryption-strategi (variant b) anbefales **betinget af F2 lander på 20 år**; hvis F2 lander på 7-10 år, er bevarings-strategi (variant a) tilstrækkelig.

Alle 15 underspørgsmål besvares nedenfor. §5 lister åbne afhængigheder. §6 indeholder AP-anbefaling per F (én sætning).

---

## 2. F1 — Recording-form (4 underspørgsmål)

### F1.1 — Teknisk forsvarbarhed pr. recording-variant

| Variant | Indhold | Non-repudiation-evidens-værdi | Forensisk rekonstruktions-værdi | Chain-of-custody-kompleksitet | AP-vurdering |
|---|---|---|---|---|---|
| **(a) Fuld video** | Ansigt + stemme + hænder + skærm | Maksimal (entydig ansigts-ID + handlings-evidens) | Maksimal | Lav (single stream, single arkiv-fil per kilde) | **[AP-teknisk]** Teknisk realiserbar, men leverer marginalt mere evidens end (b) i ceremony-kontekst hvor deltager-identitet allerede er etableret pre-ceremony via smartcard + ID-check + fysisk PACS-log + signed paper-protokol. Marginal-værdien af ansigts-video balancerer ikke biometri-eksponering. **Frarådes.** |
| **(b) Audio + skærm + hænder uden ansigt** | Stemme + skærm-capture + hænder-cam | Høj — stemme-identifikation + handlings-attribution | Høj — alle kommandoer + skærm-output + fysiske handlinger på HSM/transfer-medie synlige | Lav-medium (3 streams: audio, screen-capture, hands-cam) | **[AP-anbefaling primær]** Stemme er stadig art. 9 biometri, men volumen er reduceret. Hænder-cam dokumenterer fysiske handlinger (USB-isætning, smartcard-tap, knappe-tryk på safe) uden ansigts-eksponering. |
| **(c) Audio + skærm + ansigt-blurret video** | Som (a) men post-processed face-blur | Høj på papir — men chain-of-custody er svækket: raw-video skal enten bevares (defeat formålet) eller destrueres under ceremony-procedure (skaber "ceremoni-i-ceremoni" med ny dual-control + ny audit-trail). Blur-pipeline er blød software (kan reverseres, kan fejle, kan introducere bias). | Medium-høj | **Høj** — post-processing-pipeline kræver ekstra hardened workstation, dual-control under blurring, tamper-evident hash-sammenligning før-/efter-blur, dokumenteret raw-destruktion. | **[AP-teknisk] Frarådes.** Post-processing introducerer ny angrebsoverflade og forstyrrer non-repudiation-kæden mere end variant (b) eliminerer biometri. |
| **(d) Kun skærm + audio** | Skærm-capture + stemme | Medium — handlinger på skærm dokumenteres, men fysiske handlinger uden for skærm (HSM-USB-tilslutning, smartcard-håndtering, transfer-medie-isætning) udokumenterede. Stemme-identifikation bevarer art. 9-status. | Medium — gaps i fysisk-handling-dokumentation gør det vanskeligere at modbevise senere mistanke om fx HSM-substitution mid-ceremony. | Meget lav (2 streams) | **[AP-teknisk]** Realiserbar, men non-repudiation-værdien for fysiske handlinger forsvinder. Hvis F2 lander kort retention (7-10 år) og biometri-volumen er primær bekymring, er (d) en valid mitigation; hvis 20 år retention og høj non-repudiation-krav, er (b) bedre balance. |

**Konklusion F1.1**: AP's tekniske forsvarbarhed-rangering: **(b) > (d) > (a) > (c)**. Variant (c) frarådes uanset proportionalitets-konklusion på grund af chain-of-custody-svækkelse. Proportionalitets-konklusion (om biometri-mængden i (b) eller (d) er proportional) **ejer Dorthe** — AP's input er kun teknisk evidens-værdi. **[Kræver Dorthe]**

### F1.2 — Differentiering pr. ceremony-type

Arkitektur-dokumentets §1.2 tabel klassificerede alle fire ceremony-typer som JA-recording. Efter dybere analyse:

| Ceremony-type | Kritikalitet | Recording-anbefaling | Begrundelse |
|---|---|---|---|
| Root key generation | Maksimal — engangs-event for 10-20 år validity-cyklus | **(b)** fuld scope | Højeste non-repudiation-krav; senere mistanke om kompromittering kan kun afvises med stærk evidens |
| Intermediate-CA-signing | Høj — påvirker hele cert-hierarkiet | **(b)** fuld scope | Som ovenfor; sjælden event så ekstra evidens er proportionalt |
| Tier 1 CRL re-issuance | Medium — rutinemæssig + offline-aktivering | **(b)** fuld scope eller **(d)** reduceret scope | **[AP-teknisk]** AP foreslår **(d)** for rutinemæssig CRL re-issuance hvor ingen ny CA-key skabes — non-repudiation-kravet er lavere fordi handlingen ikke skaber nye trust-anchors. **[Kræver Mads + Dorthe]** at vurdere om differentiering er proportional eller skaber procedure-kompleksitet der overstiger gevinsten. |
| Root re-keying (PQC, kompromittering) | Maksimal — undtagelses-event | **(a)** fuld video, hvis kompromittering-vurderet eller ekstern audit | **[AP-teknisk]** Eneste case hvor variant (a) kan være proportional: ekstraordinær event hvor maksimal evidens kan være påkrævet for senere juridisk eller regulatory review. **[Kræver FGD per ceremony]** — pre-decision pr. specifik event. |

**Konklusion F1.2**: **[AP-anbefaling]** Differentiering anbefales: rutine-CRL kan reduceres til (d); ekstraordinær re-keying kan eskaleres til (a) per FGD-beslutning. Standard-baseline for Root + Intermediate-signing er (b). Endelig differentierings-skema kræver **[Kræver Mads]** + **[Kræver Dorthe]** fælles vurdering.

### F1.3 — Witness-rolle vs. operatør-rolle

| Rolle | Aktiv handling | Recording-anbefaling | Begrundelse |
|---|---|---|---|
| KCO | Procedure-styring, dual-control-aktivering | Audio + hænder | Stemme-evidens + handlings-evidens nødvendig |
| CA Administrator | Smartcard-tap, HSM-aktivering | Audio + hænder | Som ovenfor |
| HSM-operatør (Bjarne) | USB-tilslutning, HSM-physical-handling | Audio + hænder | Som ovenfor |
| Security Officer | 2nd-auth, observation af dual-control | Audio (stemme-attestation) | Mindre fysisk handling; stemme-attestation tilstrækkelig |
| Witnesses (passive bevidnere) | Passiv observation, signing af paper-protokol | Audio (stemme-attestation ved start/slut + signed paper-protokol) eller **kun signed paper-protokol** | **[AP-teknisk]** Witnesses udfører ingen handling der kræver kontinuerlig recording. Stemme-attestation ved ceremony-start ("Jeg, [navn], witness, bekræfter at jeg observerer") + signed paper er tilstrækkelig non-repudiation. Reducerer biometri-volumen yderligere. |

**[AP-anbefaling]**: Differentiering anbefales — operatør-roller får fuld (b)-recording; witness-roller får audio-attestation kun ved start/slut + paper-signing. **[Kræver Dorthe]** at vurdere om witness-attestations-audio er nødvendig eller om signed paper-protokol alene er proportionalt. **[Kræver KCO]** at bekræfte at differentiering ikke svækker procedure-integritet.

### F1.4 — Tekniske grænser

| Recording-setup | Teknisk realiserbarhed under ceremony | Forstyrrelse af dual-control | Post-processing-behov |
|---|---|---|---|
| Skærm-capture | Trivielt — OS-niveau screen-recorder (OBS, ffmpeg eller HSM-host's egen recorder) skrives til read-only-skrivbar local disk + signeret efter ceremony | Ingen | Ingen — raw output er final |
| Single audio-mic (alle deltagere) | Trivielt — single USB-mic, single audio-track | Ingen | Ingen |
| Multi-channel audio (per-deltager-mic) | Medium — kræver mixer-setup; per-deltager mic-kanal forenkler stemme-identifikation | Lav | Optional speaker-diarization (post) — IKKE påkrævet for non-repudiation |
| Single hands-cam (top-down på arbejdsbord + HSM) | Lav-medium — fast kamera-position, ingen operatør-styret pan/zoom | Ingen (kamera er fixet på forhånd) | Ingen |
| Multi-camera selektiv blurring (variant c) | **Høj** — kræver post-processing-pipeline med dual-control under blurring; ny chain-of-custody-procedure for raw-fil-destruktion; software-blur-bias-risk | Ingen under selve ceremonien, men **introducerer ny ceremoni-i-ceremoni** for blurring-eksekvering | **Høj** — separat hardened workstation, hash-evidens, dual-witness ved blur-eksekvering, raw-destruktions-procedure |

**Konklusion F1.4**: **[AP-teknisk]** Variant (b)-setup (audio + screen-capture + single hands-cam) er realiserbart **uden** post-processing og **uden** at forstyrre dual-control-procedure. Setup-konfiguration skal pre-testes i tør-kørsel før første rigtige ceremony — dette koordineres med **[KCO]** og **[Engineering Team Lead]**. Variant (c) frarådes alene på grund af post-processing-kompleksiteten.

---

## 3. F5 — Arkiveringslokation (5 underspørgsmål)

### F5.1 — EU-only-konsistens med L1

**[AP-anbefaling]** Ja — AP anbefaler at L1's EU-only-residency formelt udvides til at gælde også ceremony-recording-arkivet. Begrundelser:

1. **Logisk konsistens**: Ceremony-recording er audit-/non-repudiation-evidens for samme PKI som L1 beskytter. Inkonsistens ville være designsmæl: certs fra et EU-only-PKI dokumenteret i tredjelands-arkiv.
2. **Art. 9-data**: Recording indeholder biometri (art. 9). Tredjelands-overførsel af art. 9-data har strengere krav (art. 49) og er praktisk udelukket post-Schrems II uden ekstra TIA + supplementary measures.
3. **CLOUD Act-eksponering**: Tredjelands-leverandør med US-relation eksponerer art. 9-data for ekstraterritorial adgang.
4. **DPA-byrde**: EU-only-leverandører har simplere DPA-skabeloner; ingen SCC + TIA-overhead.

**[Kræver FGD]** — formel udvidelse af L1 som "låst forudsætning også for ceremony-recording-arkiv" er en mandat-/låsnings-niveau-beslutning. AP kan ikke selv låse en forudsætning. **AP-anmodning til Dorthe**: bring denne anbefaling med i DPIA-1-rapportens kapitel om proportionalitet, så FGD kan låse i samme beslutning som DPIA-konklusionen.

### F5.2 — Konkret EU-leverandør-shortlist for cloud-immutable-storage

**[AP-teknisk-fit forslag]** Følgende EU-leverandører er teknisk relevante som cloud-immutable-storage-option for ceremony-recording-arkivet. Listen er **teknisk-fit-forslag**; **DPA-vurdering og leverandør-godkendelse ejer Dorthe** — AP forhandler ikke DPA'er.

| # | Leverandør | EU-region | Immutability-mekanisme | Teknisk-fit-vurdering | DPA-vurdering ejer |
|---|---|---|---|---|---|
| 1 | **Scaleway** (Frankrig) | FR-PAR, FR-AMS (NL via region-pinning) | S3-compatible Object Lock (governance + compliance mode) | Stærk: native S3 Object Lock; FR-juridisk grundlag; ingen CLOUD Act-eksponering | Dorthe |
| 2 | **OVHcloud** (Frankrig) | FR-GRA, FR-RBX, DE-FRA, PL-WAW | S3-compatible Object Lock på Object Storage; Cold Archive-tier | Stærk: bredt EU-region-valg; FR-juridisk grundlag; veldokumenteret EU-residency | Dorthe |
| 3 | **Stackit** (Tyskland — Schwarz Group) | DE-FRA, DE-MUC | S3-compatible Object Lock | God: tysk juridisk grundlag; relativt ny på markedet, men EU-souverænitet er core-positionering | Dorthe |
| 4 | **Hetzner Storage Box** + **Object Storage (beta/GA-status verificeres)** | DE-FSN, DE-NUE, FI-HEL | Object Lock-status skal Context7-verificeres inden valg | Medium: Hetzner er stærk på pris/EU-residency, men Object Lock-modenhed er senere til markedet end Scaleway/OVH | Dorthe + AP teknisk verifikation |
| 5 | **IONOS** (Tyskland) | DE-FRA, DE-BER | S3-compatible Object Lock | God: tysk juridisk grundlag; IONOS Cloud har stigende enterprise-feature-modenhed | Dorthe |

**[AP-teknisk]** AP's primær-anbefaling for cloud-option: **Scaleway** eller **OVHcloud** — begge har modent S3 Object Lock, dokumenteret EU-residency og DPA-tilgængelighed. Sekundær: **Stackit** eller **IONOS** for tysk juridisk præference.

**[Kræver Dorthe]** DPA-tilstrækkelighed, sub-processor-kæde-vurdering, evt. TIA hvis sub-processor-kæden eksponerer for tredjeland.

**[Kræver Mads]** Konkret leverandør-valg for security-postprofil (encryption-at-rest, mTLS for API, IAM-model, audit-log-eksport).

**Note om verifikation**: AP har **ikke** kørt Context7-verificering på Object Lock-feature-status for hver leverandør i dette dokument — faktisk tilgængelighed på dato pr. valgs-tidspunkt skal verificeres af AP + Engineering Team Lead inden konfiguration.

### F5.3 — On-prem WORM-platform

**[AP-teknisk]** AP's anbefaling for on-prem WORM-platform afhænger af konsolidering vs. separation (jf. F9.5 nedenfor):

| Platform | Type | Modes | AP-fit-vurdering |
|---|---|---|---|
| **NetApp SnapLock** | Filer-niveau WORM (NAS) | **Compliance** (hard-locked, kan ikke modificeres af admin) eller **Enterprise** (admin-revocable) | Bredt anvendt for compliance-arkiv; god fit hvis on-prem NetApp-stack allerede findes |
| **Dell ECS** | Object storage med retention | Native object-immutability | God for hybrid object/file workloads |
| **Cohesity SmartFiles** | Konsolideret backup + arkiv | Object Lock + retention-policies | God hvis bredere backup-konsolidering ønskes |
| **Dedikeret WORM-tape (LTO med WORM-medie)** | Offline tape | Hardware-WORM (kan ikke overskrives fysisk) | Stærk for længste retention (20+ år); kræver tape-library + restore-procedure | 

**[AP-anbefaling]** For Fase 0: **NetApp SnapLock i Enterprise-mode** for ceremony-recording-arkivet. Begrundelse:

- Enterprise-mode tillader retention-justering hvis DPIA-1-konklusion eller senere DPIA-revisitation reducerer retention (fx fra 20 år til 10 år)
- Compliance-mode låser retention til hard-coded grænse — hvis F2 senere reduceres, sidder vi fast med data vi ifølge GDPR art. 5(1)(e) ikke længere må opbevare
- **[Kræver Mads]** at vurdere om Enterprise-mode-svækkelse (admin kan teknisk revoke) er acceptabel risk vs. Compliance-mode's regulatory-hard-lock. Mitigation: dual-control + audit-log på alle admin-operationer på SnapLock-volume + Trine-review halvårligt.

**Sammenhæng med arkitektur §5.2 break-glass-arkiv**: Break-glass-arkiv-platformen er beskrevet som "S3 Object Lock (compliance-mode)" eller "NetApp SnapLock". Konsolidering vs. separation behandles i F9.5 (separat key-management-spørgsmål). **[AP-anbefaling]** for Fase 0: **konsolidér på samme fysiske platform** (NetApp SnapLock Enterprise-mode), men **separate volumes med separate retention-policies og separate access-control-lister** — så blast-radius reduceres uden at duplikere infrastruktur. **[Kræver Mads]** endelig vurdering.

### F5.4 — Geografisk distribution (redundans-kopi)

**[AP-teknisk]** AP anbefaler **redundans-kopi i sekundær EU-lokation** geografisk adskilt fra primær. Konkret design:

```
Primær arkiv (EU-region 1):
  - NetApp SnapLock Enterprise-mode volume
  - Live recording-fil skrives her ved ceremony-afslutning

Sekundær arkiv (EU-region 2, geografisk adskilt — fx FR + DE):
  - NetApp SnapLock Enterprise-mode volume eller EU-cloud Object Lock
  - Synkroniseret via SnapMirror (NetApp-native) eller scriptet integrity-checked rsync over mTLS

Synkroniserings-model:
  - Primær arkiv låser fil med Object Lock / SnapLock retention SAMTIDIG med at sekundær modtager kopi
  - Sekundær låser med samme retention-stempel
  - Begge sider har samme cryptographic hash chain-entry; tampering på én side detekteres ved hash-sammenligning under audit
  - Sletning ved retention-udløb: koordineret via centraliseret retention-controller; begge sider sletter samtidig (audit-logged)
```

**Synkroniserings-uden-at-bryde-immutability**: Synkronisering må ikke ændre eksisterende immutable filer. Mekanismer:

1. **Append-only**: Nye recordings appendes; gamle modificeres aldrig
2. **Hash-verificering**: Før hver sync verificeres at primær-fil ikke er ændret (Merkle-tree-rod sammenlignes)
3. **Atomic-commit**: Sekundær låser fil kun efter fuld modtagelse + hash-verificering; partial transfer rolles tilbage

**[Kræver Mads]** at godkende synkroniserings-model inkl. mTLS-cert-management mellem EU-regioner og adgangskontrol til synkroniserings-controller.

### F5.5 — Sletning ved retention-udløb (compliance-mode-fælde)

**[AP-teknisk]** Dette er en **kritisk fælde**: WORM-platforme i "compliance-mode" tillader ofte ikke sletning før hard-coded retention-grænse. Hvis F2-konklusionen senere ændrer retention (fx revisitation efter 5 år konkluderer at 10 år er proportionalt i stedet for oprindeligt valgte 20), sidder vi fast med data vi GDPR-art.-5(1)(e)-mæssigt ikke længere må opbevare.

**AP-anbefaling for platform-valg**:

1. **Vælg Enterprise-mode (eller equivalent admin-revocable)** for ceremony-recording-arkivet — ikke Compliance-mode
2. **Mitigation for tamper-resistance-svækkelse**:
   - Dual-control på alle SnapLock-admin-operationer (to separate authentication-keys, ingen kan revoke alene)
   - Alle admin-operationer audit-logges til separat immutable break-glass-log (jf. arkitektur §5.2)
   - Trine auditerer SnapLock-admin-aktivitet halvårligt (SOC 2 CC7.2-evidens)
   - Retention-justering kræver formel governance-proces: Mads + Dorthe + FGD-godkendelse, dokumenteret i `governance/change-requests/`
3. **For break-glass-arkivet** (separat use-case): **Compliance-mode** kan være acceptabelt fordi break-glass-logs har faste 7-års retention der matcher SOC 2-baseline og ikke skal kunne reduceres baseret på senere DPIA

**[Kræver Mads]** at vurdere Enterprise vs. Compliance-mode trade-off for ceremony-recording specifikt. Dette er en security-vs.-GDPR-fleksibilitet trade-off der ligger inden for CISO-mandatet.

**Sletning-eksekverings-procedure (når retention udløber)**:

1. Centraliseret retention-controller identificerer fil med udløbet retention
2. Dual-control aktivering af sletnings-operation (to separate operatører)
3. Sletning eksekveres simultant på primær + sekundær arkiv
4. Sletnings-event logges i break-glass-log (uforanderligt arkiv)
5. Trine verificerer halvårligt at sletninger er gennemført korrekt og at ingen "zombie-data" forbliver

---

## 4. F9 — Krypterings-arkitektur (6 underspørgsmål)

### F9.1 — HSM-placering: separat Domain vs. dedikeret HSM-instance

**[AP-anbefaling for Fase 0]**: **Separat Domain på samme YubiHSM2-fleet** (specifikt: dedikeret Domain "CeremonyArchive" på Tier 2 HSM-par eller på et tredje YubiHSM2 dedikeret til arkiv-funktionen).

**Argumenter for separat Domain på samme fleet**:

- Operationel enkelhed: samme HSM-management-procedurer (audit-eksport, firmware-update, backup)
- YubiHSM2 Domain-isolation er teknisk solid: authentication-key med Domain "Root" kan ikke tilgå nøgler i Domain "CeremonyArchive" selvom de fysisk er på samme device (jf. arkitektur §2.2)
- HSM-slot-budget: 256 object slots per HSM, ceremony-arkiv-key + Shamir-shares-evidens fylder marginalt
- Lavere CapEx Fase 0

**Argumenter for dedikeret separat HSM-instance**:

- Blast-radius: hvis YubiHSM2 firmware-bug eller fysisk kompromittering rammer fleet, er ceremony-arkiv-key på separat HSM uberørt
- Cleaner separation-of-duties: HSM-operatør kan have forskellige authentication-keys for CA-fleet vs. arkiv-HSM
- Bedre fit hvis senere arkitektur-vurdering kræver formel "evidence-vault" separation

**[AP-anbefaling sluttelig]**: For Fase 0 = separat Domain (lavere kompleksitet). For Fase 1+ = revisitér efter første års drift; hvis blast-radius-vurdering skalerer (fx flere PKI-instanser), eskalér til separat HSM-instance.

**[Kræver Mads]** endelig blast-radius-vurdering. **[Afventer Bjarne]** Bjarnes input om Domain-konfiguration vs. fleet-management-overhead. AP-anbefalingen kan ændres baseret på HSM-operatør-praksis.

### F9.2 — Key-hierarki (ASCII-diagram)

```
                 Ceremony Recording File (raw stream output)
                              |
                              | (encrypt symmetrisk per fil)
                              v
                  +-------------------------+
                  | Recording File on disk  |
                  |   - AES-256-GCM         |
                  |   - per-file unique     |
                  |     nonce (96-bit)      |
                  |   - per-file unique DEK |
                  +-------------------------+
                              |
                              | DEK (Data Encryption Key, 256-bit)
                              | wrappet med...
                              v
                  +-------------------------+
                  | KEK = Ceremony-Arkiv    |
                  |       Key (CAK)         |
                  |   - AES-256             |
                  |   - YubiHSM2 Domain     |
                  |     "CeremonyArchive"   |
                  |   - Capabilities:       |
                  |     wrap-data,          |
                  |     unwrap-data         |
                  |   - Authentication via  |
                  |     dual-control auth-  |
                  |     key                 |
                  +-------------------------+
                              |
              +---------------+---------------+
              |                               |
              | (gen 1: 2026-2031)            | (gen 2: 2031-2036, etc.)
              v                               v
   +-------------------+           +-------------------+
   | CAK-gen-1         |           | CAK-gen-2         |
   | (5-aars vindue)   |           | (5-aars vindue)   |
   | Status: rotated   |           | Status: active    |
   |        (revoked   |           |                   |
   |         men ikke  |           |                   |
   |         slettet)  |           |                   |
   +-------------------+           +-------------------+
              |                               |
              | (recovery — alle CAK-gen      |
              |  beskyttes på samme måde)     |
              v                               v
                  +-------------------------+
                  | Shamir Secret Sharing   |
                  |   - 3-of-5 split per    |
                  |     CAK-generation      |
                  |   - shares fordelt:     |
                  |     1. KCO (HQ-safe)    |
                  |     2. CA Admin         |
                  |        (HQ-safe-2)      |
                  |     3. HSM-operator     |
                  |        (off-site safe)  |
                  |     4. Security Officer |
                  |        (bank-deposit)   |
                  |     5. Ekstern juridisk |
                  |        depot            |
                  +-------------------------+
                              |
                              | Recovery-procedure: minimum
                              | 3 shares samles på
                              | air-gapped recovery-host
                              | under dual-control ceremony;
                              | CAK rekonstrueres in-memory;
                              | DEK unwrappes; recording-fil
                              | dekrypteres til ephemeral
                              | viewing session
                              v
                       (read-only viewing)
```

**Per-fil eller per-ceremony DEK?** **[AP-teknisk]**: Per-fil DEK. Begrundelse: granularitet for selektiv decryption (Trine-audit kan behøve at se én specifik recording uden at unwrappe alle); lavere blast-radius hvis enkel DEK kompromitteres.

**Wrap-key-rotation-fane (gamle versioner)**: Gamle CAK-generationer bevares på HSM med status "rotated" (capability `wrap-data` fjernet, `unwrap-data` bevaret) så historiske recordings stadig kan dekrypteres. Shamir-shares for gamle generationer arkiveres med samme procedure som aktive — ingen destruktion før alle recordings krypteret med den generation er slettet (retention-udløb).

**Recovery-procedure**:

1. Mindst 3 Shamir-shares samles fysisk på air-gapped recovery-host
2. CAK rekonstrueres in-memory (aldrig persisteret)
3. DEK unwrappes via HSM (kræver dual-control authentication-key for "auth-archive-recovery" — separat fra daglig drift-key)
4. Recording-fil dekrypteres til ephemeral viewing session (RAM only, no disk write)
5. Session auto-logs til immutable break-glass-log
6. Session auto-terminates efter max 4 timer
7. Air-gapped host wipes RAM ved session-slut

### F9.3 — 20-års key-rotation-strategi: bevarings-strategi (a) vs. re-encryption-strategi (b)

**[Afventer F2]** Endelig anbefaling afhænger af om F2 lander på 7-10 år eller 20 år.

**Variant (a) — Bevarings-strategi**:
- Gamle CAK-generationer bevares som "rotated" (revoked men accessible)
- Nye recordings krypteres med nyeste CAK
- Over 20 år: op til 4-5 CAK-generationer "live" (5-års rotation)

**Risici**:
- **Tab-af-konfidentialitet**: Ældre CAK-generationer er ældre algorithm-baseret; hvis AES-256-GCM viser sig svagt eller kompromitteret, er gamle recordings fortsat krypteret med svag CAK
- Større angrebsoverflade (flere generationer, flere Shamir-share-sæt vedligeholdes)
- Fortsat brug af 5-15-årige Shamir-share-fysiske-arkiver med tilhørende fysisk-sikkerheds-vedligehold

**Fordele**:
- Operational simplicity: ingen re-encryption-ceremonier
- Lavere ceremony-overhead over 20-årshorisont

**Variant (b) — Re-encryption-strategi**:
- Hvert 5-10 år dekrypteres + re-krypteres alle historiske recordings med nyeste CAK
- Gamle CAK destrueres efter re-encryption verificeret (hash-sammenligning + integrity-validation)

**Risici**:
- **Tab-af-adgang**: Hvis re-encryption-ceremoni fejler midt-event, kan vi miste adgang til alle historiske recordings
- **Chain-of-custody-svækkelse**: Plaintext recording berører re-encryption-host under processen — ny risiko for kompromittering eller tampering
- Stor procedural-omkostning (storskala-ceremoni hvert 5-10 år; kræver dual-control + audit-evidens på alle filer)
- Cipher-skifte mid-ceremony håndteres godt — kan også opgradere algorithm (PQC) som del af re-encryption

**Fordele**:
- Konfidentialitet altid baseret på nyeste cipher
- Færre live-CAK-generationer (typisk 1-2 i stedet for 4-5)
- Cipher-agility indbygget (jf. F9.4)

**[AP-anbefaling betinget på F2]**:

| F2 lander på | AP-anbefaling | Begrundelse |
|---|---|---|
| **7-10 år** | **Variant (a) — Bevarings-strategi** | Kun 1-2 CAK-generationer live; bevarings-overhead er lav; re-encryption-risiko ikke berettiget |
| **20 år** | **Variant (b) — Re-encryption-strategi**, med re-encryption-cyklus hvert 7-10 år | 4-5 generationer er ikke længere håndterbart; konfidentialitet over 20 år kræver cipher-agility som re-encryption tilbyder |
| **Differentieret retention** (fx biometri 7 år, protokol 20 år) | Hybrid — variant (a) for kort retention, variant (b) for lang | Ægte separation hvis recording teknisk kan splittes — afhænger af recording-form |

**[Kræver Mads]** endelig krypto-strategi-godkendelse uanset variant. **[Kræver FGD]** hvis variant (b) vælges, fordi det er en multigeneration-procedural commitment.

### F9.4 — Cipher-agility (algorithm-skifte inden 20 år)

**[AP-teknisk]** Arkitekturen kan håndtere algorithm-skifte:

- **DEK-niveau**: Per-fil DEK gør det muligt at re-encrypt fil-by-fil med ny algorithm uden at touch'e KEK-hierarkiet
- **KEK-niveau (CAK)**: AES-256-GCM kan udskiftes med ChaCha20-Poly1305 eller PQC-symmetric-equivalent når nødvendigt; HSM understøtter cipher-agility som firmware-feature
- **Variant (b) re-encryption-strategi**: Naturlig point at indbygge algorithm-skifte; hver re-encryption-cyklus kan opgradere cipher

**Omkostnings-estimat for algorithm-skifte**:

| Scenarie | Omkostning |
|---|---|
| Variant (a) + algorithm-skifte | **Høj** — kræver re-encryption af alle historiske data ved skifte (de-facto en variant-(b)-ceremoni alligevel) |
| Variant (b) + algorithm-skifte | **Marginal merkost** — algorithm-skifte foretages i næste planlagte re-encryption-ceremoni (5-10 års-cyklus) |

**Konklusion**: Variant (b) giver bedre cipher-agility-readiness uden ekstra ceremoni-overhead. Hvis variant (a) vælges (kort retention), accepteres at algorithm-skifte vil koste ekstra ceremoni. **[Kræver Mads]** at vurdere PQC-roadmap-koblingen til ceremony-arkiv.

### F9.5 — Sammenhæng med break-glass-arkiv (konsolidering vs. separation)

**[AP-teknisk]** Sammenhæng-vurdering:

| Aspekt | Konsolidering (samme platform + samme key-hierarki) | Separation (separat platform + separat key-hierarki) |
|---|---|---|
| **Operational complexity** | Lavere — samme HSM-Domain, samme audit-tooling, samme backup-procedure | Højere — to parallelle stacks |
| **Blast-radius** | Større — kompromittering af shared platform/key rammer både ceremony-recording + break-glass-evidens | Mindre — separation isolerer compromise |
| **Access-control populations** | Overlap er problematisk — break-glass-operatører ≠ ceremony-auditorer; konsolideret access bløder mandate-grænser | Klarere SOD — break-glass-mandater skilles fra ceremony-evidens-mandater |
| **Audit-tooling** | Trine bruger ét set tools; lavere træningsbehov | Trine bruger to sets; højere overhead men også klarere rapport-separation |
| **CapEx + OpEx** | Lavere | Højere |
| **Compliance-vurdering** | SOC 2 + ISO 27001 acceptable hvis access-control er stærk | SOC 2 + ISO 27001 foretrækker separation hvor blast-radius er problematisk |

**[AP-anbefaling]**: **Hybrid — samme fysiske platform (NetApp SnapLock), separate volumes med separate access-control + separate key-hierarki**:

- **Fælles platform**: Samme NetApp SnapLock-stack (CapEx-effektivitet)
- **Separate volumes**: `vol-ceremony-recording` (Enterprise-mode, 7-20-års retention afh. F2) vs. `vol-break-glass-evidence` (Compliance-mode, 7-års fast retention)
- **Separate keys**: Egen CAK for ceremony-recording (Domain "CeremonyArchive" på HSM); egen separat KEK for break-glass-arkiv (Domain "BreakGlass" på HSM)
- **Separate authentication-keys**: `auth-archive-ceremony` ≠ `auth-archive-breakglass`; ingen rolle har begge defaults
- **Fælles infrastruktur-audit men separate compliance-audit-rapporter**

**[Kræver Mads]** endelig vurdering. AP's anbefaling balancerer operational og blast-radius-overvejelser; hvis Mads vurderer at blast-radius er overordnet, kan separat platform overvejes som Fase 1+.

### F9.6 — Auditor-access (Trine) — read-only uden solo-decrypt

**[AP-teknisk]** Design for auditor-viewing-session:

```
Trine ønsker review af recording R fra ceremony C
                    |
                    v
1. Trine indsender audit-request til governance-system
   (begrundelse + scope; auto-logged)
                    |
                    v
2. HSM-operatør (Bjarne) initierer dual-witness-session:
   - Trine + Bjarne tilstede fysisk eller via mTLS-attesteret
     remote-session
   - Hardened workstation i secure-room (samme grade som
     ceremony-rum); booted fra read-only media
                    |
                    v
3. Bjarne authentikerer mod HSM med "auth-archive-viewing"
   (separat authentication-key; capabilities: unwrap-data
   only, no export)
                    |
                    v
4. Recording-DEK unwrappes via HSM into ephemeral session
   memory (RAM only)
                    |
                    v
5. Recording-fil dekrypteres på workstation til viewing
   buffer (RAM only — disk-write blokeret af kernel-
   policy + OS-level audit)
                    |
                    v
6. Trine reviewer i viewing-window
   (samtale logs/transcript er audit-evidens; recording
   selv eksporteres ikke)
                    |
                    v
7. Session auto-terminates efter max 4 timer eller ved
   manual-end; RAM wipes; ingen plaintext på disk
                    |
                    v
8. Hele sessionen audit-logges til separat immutable
   break-glass-log (samme arkitektur som break-glass-
   sessioner, jf. AP-mandat: "Audit-trail for break-
   glass-sessioner gemmes i uforanderlig form")
                    |
                    v
9. Trine's audit-rapport refererer til session-ID;
   recording selv re-shippes aldrig
```

**Nøgleegenskaber**:

- **Trine kan ikke solo-decrypt**: HSM-operatør (Bjarne) skal authentikere `auth-archive-viewing` for at unwrap DEK
- **HSM-operatør kan ikke solo-decrypt**: Authentication-key alene giver kun unwrap-capability i en session; recording-fil-adgang kræver fysisk præsens på workstation hvor Trine er attesteret
- **Plaintext berører aldrig disk**: Disk-write blokeret af kernel-policy + OS-level audit
- **Session-evidence**: Auto-logged break-glass-log med start/slut-timestamps, deltagere, recording-ID, audit-begrundelse
- **Tids-begrænsning**: Max 4 timer matcher AP-mandatets break-glass-procedure-spec

**Bemærk**: Dette er **ikke** break-glass per se — det er kvartalvis audit. Men den **bruger samme audit-trail-infrastruktur** som break-glass for konsistens og fordi auditor-access til art. 9-data har høj compliance-relevans (Dorthe-input vigtig).

**[Kræver Mads]** at godkende design + capabilities-mapping for `auth-archive-viewing`. **[Afventer Bjarne]** for konkret HSM-operator-procedure-script. **[Kræver Trine-input]** at viewing-format er tilstrækkeligt for hendes audit-formål uden eksport.

---

## 5. Åbne afhængigheder

AP afventer følgende inputs inden endelig svar kan låses:

| # | Input | Ejer | Hvilke F-spørgsmål påvirkes | Frist |
|---|---|---|---|---|
| AD1 | F2 retention-konklusion (7-10 år vs. 20 år vs. differentieret) | Mads + AO + FGD | F9.3 (key-rotation-strategi); F9.4 (cipher-agility-omkostning); F1.2 (recording-form per ceremony-type — proportionalitet) | Inden DPIA-1-konklusion |
| AD2 | O7 datacenter-tier + safe-grade for Tier 1 | Mads + FGD | F5.3 (on-prem WORM-platform — fysisk lokation/sikring) | Inden ceremony-script låses |
| AD3 | O11 SIEM-platform-valg | Mads + Engineering Lead | F5 audit-integration; F9.6 viewing-session-logging | Inden første ceremony |
| AD4 | Mads' godkendelse af 5-årig CAK-rotation-cadence | Mads | F9.2 (key-hierarki); F9.3 (rotation-strategi) | Pre-ceremony |
| AD5 | HSM-operatør (Bjarne) input på Domain vs. separat HSM-instance for ceremony-arkiv | Bjarne + Mads | F9.1; F9.5 | Pre-ceremony |
| AD6 | FGD-formel udvidelse af L1 til at gælde ceremony-recording-arkiv | FGD via Stefan | F5.1; F5.2 (vendor-shortlist scope) | DPIA-1-konklusion er passende moment |
| AD7 | Engineering Team Lead bekræftelse af recording-setup-realiserbarhed (variant b: audio + screen + hands-cam) i tør-kørsel | Engineering Lead + KCO | F1.4 | Pre-ceremony tør-kørsel |
| AD8 | Trine-input på audit-viewing-format-tilstrækkelighed | Trine | F9.6 | Inden første audit-cyklus |
| AD9 | Context7-verificering af Object Lock-feature-status hos Hetzner Object Storage | AP + Engineering Lead | F5.2 | Inden cloud-leverandør-valg |
| AD10 | Mads vurdering Enterprise-mode vs. Compliance-mode for ceremony-recording-arkiv | Mads | F5.5 | Inden platform-valg-låses |

Manglende inputs blokerer ikke DPIA-1-konklusion på AP's vegne — Dorthe kan trække åbne afhængigheder ind i DPIA-rapportens "rest-risiko"-sektion. Men flere af dem (AD1, AD4, AD5, AD10) bør ideelt lukkes inden DPIA-1 distribueres til 4-eyes-review.

---

## 6. AP-anbefaling pr. F-spørgsmål (én sætning hver)

**F1**: AP anbefaler primært variant (b) "audio + screen-capture + hænder uden ansigt" for alle Tier 1 og Tier 2 ceremonier, med differentiering for rutinemæssige Tier 1 CRL re-issuance (variant d) og ekstraordinær Root re-keying (variant a per FGD-beslutning); variant (c) frarådes på grund af post-processing chain-of-custody-svækkelse.

**F5**: AP anbefaler EU-only-residency formelt udvidet som låst forudsætning (FGD-beslutning), med on-prem NetApp SnapLock Enterprise-mode som primær platform og Scaleway eller OVHcloud Object Lock som sekundær EU-cloud-fallback; konsolideret fysisk platform med separate volumes for ceremony-recording vs. break-glass-evidens.

**F9**: AP anbefaler per-fil AES-256-GCM DEK wrappet med ceremony-arkiv-key (CAK) på separat YubiHSM2 Domain "CeremonyArchive" med 3-of-5 Shamir Secret Sharing for CAK-recovery; 5-årig CAK-rotation; bevarings-strategi (a) hvis F2 lander 7-10 år, re-encryption-strategi (b) hvis F2 lander 20 år; auditor-viewing-session med dual-witness + RAM-only decryption + break-glass-log-integration sikrer at hverken Trine eller HSM-operatør kan solo-decrypt.

---

## 7. Spørgsmål til Dorthe

1. **F1.2 differentiering**: Er Dorthe enig i at differentieret recording-scope per ceremony-type er proportionalt, eller foretrækker DPO ensartet baseline (samme variant for alle)?
2. **F1.3 witness-rolle**: Er stemme-attestation ved ceremony-start + signed paper-protokol tilstrækkeligt for witnesses, eller kræver art. 9-vurderingen kontinuerlig audio?
3. **F5.1 L1-udvidelse**: Vil Dorthe formelt anbefale i DPIA-1-rapporten at FGD låser L1's EU-only også for ceremony-recording-arkivet?
4. **F9.6 audit-viewing**: Er den foreslåede viewing-session-arkitektur tilstrækkelig som art. 32-mitigation, eller skal yderligere kontroller tilføjes (fx Mads-godkendelse per audit-session, ikke kun governance-request)?
5. **Differentieret retention**: Hvis F2 lander differentieret (biometri kortere end protokol), hvor teknisk skarp skal opdelingen være — kan vi acceptere at samme recording-fil indeholder begge dele med samme retention, eller skal recording teknisk splittes ved capture-tid?

---

**Forfatter**: Application Provider (FGDCorePKI)
**Dato**: 2026-05-10
**Status**: Udkast — distribueret til Dorthe (DPO); kopi til Application Owner og Mads (CISO) for awareness; afventer Dorthe-feedback eller DPIA-1-konklusion
**Næste milestone**: DPIA-1-konklusion 1-2 uger efter dette svar; AP står til rådighed for opfølgningsspørgsmål eller dyb-dyk på enhver underdel
