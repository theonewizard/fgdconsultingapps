---
dato: 2026-05-09
forfatter: Poul (senior analytiker)
emne: FGDCorePKI — three-tier PKI-arkitektur, 10-vinkler-analyse
bestilt-af: Stefan (på vegne af FGD)
input: Team/Mads/2026-05-09-pki-org-krav.md
status: UDKAST — input til Mads, Laila, Stefan, FGD
metode: research-baseret med live-verificering af RFC/NIST/CABF/Yubico-status (2026-05-09)
---

# FGDCorePKI — arkitektur-analyse fra 10 vinkler

## Sammendrag (TL;DR)

Three-tier-arkitekturen som Mads har skitseret er **konceptuelt sund** og følger industri-best-practice for høj-tillids PKI (Root offline, intermediate, issuing). Analysen finder dog **3 blokerende fund**, **6 fund der kræver afklaring**, og **1 vinkel der er acceptabel som beskrevet**. De vigtigste blocker-issues er:

1. **YubiHSM2's FIPS 140-2 certifikat udløb 2. maj 2026 (7 dage siden)**; FIPS 140-3 cert er endnu ikke granted af NIST CMVP. "FIPS 140-3 fra dag 1" som compliance-mål kan **ikke verificeres i dag** — det er en faktisk blocker, ikke en formuleringssag.
2. **PQC-blindhed for 15-20 års Root CA**: en Root genereret 2026 med RSA-4096 eller ECDSA P-384 vil overleve den realistiske safe-horizon for klassisk asymmetrisk krypto. NIST har finaliseret ML-DSA (FIPS 204) i aug 2024. Mads' design adresserer ikke crypto-agility.
3. **Tier 2 shared YubiHSM2 SPoF** (Mads har selv flagget dette) er ikke afklaret — uden HA-pair eller dokumenteret RTO/RPO kan arkitekturen ikke godkendes.

Derudover er der vigtige gaps som Mads' dokument ikke har dækket: **Name Constraints** (RFC 5280 §4.2.1.10), **pathLenConstraint**, **time-source-integritet for offline Root**, **CABF SC-081v3 leaf-validity-trajektorie** (allerede 200 dage fra 2026-03-15 for public TLS), **Certificate Transparency** (RFC 9162) hvis ekstern tillid skal være mulig, og **YubiHSM2's hardware-grænser** (256 object slots, ~7 RSA-4096-sign/sek).

---

## Vinkel 1 — Physical Security

### Arkitektur set fra denne vinkel

Tier 1 (offline Root CA) er den fysiske trust anchor. Dens kompromittering er nukleart scenario. YubiHSM2 i Level 3 har tamper-evident og tamper-response (zeroization), men hardware-extraction-angreb mod en fysisk enhed der ligger i en svag location er stadig den primære angrebsvektor mod FIPS 140-3 Level 3. Tier 2/3 er online — de står i et server-rack og er udsat for både fysisk og logisk insider-trussel.

### Fund og risici

- **Mads' dokument specificerer ikke datacenter-tier-niveau** (Uptime Institute Tier III/IV) eller fysisk safe-klasse for Tier 1 (RC3/RC4 eller EN 1143-1)
- **Air-gap-definition er uklar**: er det: (a) ingen netværk overhovedet, (b) lokalt LAN uden internet, eller (c) "logical air gap" (host firewall blokerer alt)? Forskellen er kritisk for Root CA
- **Tomandsregel for fysisk safe-adgang er ikke beskrevet** — kun for HSM-operationer. To-mands-regel skal også gælde safe-locks (split-knowledge: ingen enkeltperson kender begge lås-koder eller har begge nøgler)
- **Tamper-evidence på YubiHSM2-emballage** mellem ceremonier: hvilke seals (numbered tamper-evident bags), hvilken chain-of-custody-protokol mellem ceremonier, hvor opbevares enhederne (separate physical safes fra wrap-key-backup)?
- **Bjarne's eksisterende Knowlagecentral-HSM**: er FGDCorePKI-HSM samme fysiske lokation eller separat? Hvis samme: fælles physical compromise = blast-radius på begge systemer

### Anbefalinger

1. **Datacenter-krav fastsættes**: Tier III minimum (99.982% uptime, N+1 redundancy). For Tier 1 (offline Root): SOC 2 Type II-certificeret datacenter eller dedikeret FGD-lokation med tilsvarende fysisk kontrol; safe minimum **EN 1143-1 Grade III** eller UL TL-15 (Underwriter Laboratories tool/torch resistance 15 min)
2. **Air-gap definition formaliseres** i CP/CPS §5: "Tier 1 Root CA host opererer på dedikeret hardware uden NIC, uden trådløs, uden Bluetooth; ceremonier afholdes i Faraday-skærmet rum (TEMPEST hvis truslen er statslig aktør)"
3. **Tomandsregel for safe-adgang**: dual-lås på safe (two physical keys held by separate persons, eller PIN+key-split). Adgang logges fysisk med dato/tid/personer/begrundelse (papirjournal i safe + elektronisk PACS)
4. **Chain-of-custody-protokol**: numbered tamper-evident bags (Tyvek-style), seal-numre logges ved hver bevægelse, witnesses signerer custody-skift. NIST SP 800-152 (CKMS) §6.4 reference

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- Datacenter-tier + safe-grade specificeres i CP/CPS inden ceremony 1
- Air-gap formaliseret som "no-network host + Faraday option for high-threat scenarios"
- Tomandsregel udvides til fysisk adgang, ikke kun logisk

---

## Vinkel 2 — Key Management Lifecycle

### Arkitektur set fra denne vinkel

Mads' §3.1, §3.4, §3.5 dækker generation, backup og rotation. Lifecycle skal mappe til **NIST SP 800-57 Part 1 Rev. 5** §5.6 (key states: pre-activation → active → suspended/deactivated → compromised → destroyed) og **NIST SP 800-152** for hele CKMS.

### Fund og risici

- **Ingen explicit destruction procedure**: Rev 5 §8.3 kræver crypto erase eller fysisk destruktion af nøgle-bærende media. YubiHSM2 factory reset er nævnt i SOD-matrix men ikke som lifecycle-trin
- **Wrap-key er en kritisk sub-key der ikke har egen lifecycle**: Mads' §3.4 nævner "AES-256 wrap-key genereret under initial setup" — men wrap-key skal selv have rotation, escrow-strategy, og kompromitterings-respons. En kompromitteret wrap-key betyder kompromitterede backups
- **Authentication keys (yhsm2-auth)** har ingen rotation-cycle defineret. YubiHSM2's authentication keys er den logiske gateway — uden rotation er stale credentials et insider-risk
- **Crypto-agility er fraværende**: design-dokumentet låser RSA-4096 / ECDSA P-384. Når PQC migration kommer (NIST 2035-mål), kræver det **dual-CA-certifikat-strategi** eller fuld re-key-ceremony for alle tiers. Dette er ikke planlagt
- **No M-of-N split for wrap-key**: hvis wrap-key opbevares ét sted (selv encrypted), er det enkelt-punkts-kompromis. Shamir Secret Sharing (3-of-5 fx) på wrap-key giver samme dual-control-ejendom som m-of-n smartcards
- **Key archival vs. destruction**: skal historiske CA private keys arkiveres for verifikations-formål (gamle signaturer skal kunne verificeres)? Hvis ja: archive-procedure mangler. Hvis nej (anbefalet for Root signing keys): explicit destroy-procedure mangler

### Anbefalinger

1. **Komplet lifecycle-tabel** tilføjes CP/CPS §6 baseret på NIST SP 800-57 Pt 1 Rev 5 §5.6: pre-activation, active, deactivated, compromised, destroyed — for hver key class (Root signing, ICA signing, Issuing CA signing, OCSP responder signing, wrap-key, authentication-key, log-signing-key)
2. **Wrap-key M-of-N split** via Shamir Secret Sharing 3-of-5; shares opbevares geografisk distribueret. Wrap-key rotation hver 5. år eller ved personell-skift
3. **Authentication-key rotation policy**: hver 12. måned + ved personell-skift (off-boarding cleared inden for 24 timer per SOC 2 CC6.3)
4. **Crypto-agility dokumenteres**: planlagt re-keying-event ved NIST PQC-migration; certifikat-profiler designed til at understøtte hybrid signaturer (klassisk + ML-DSA) når X.509-extensions for hybrid er stabiliseret (IETF LAMPS WG track)
5. **Destruction procedure** i CP/CPS §6: HSM zeroize-kommando + verification + witness-signature + physical destruction (drill + shred) for retirede HSM'er

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- Komplet NIST SP 800-57 Pt1 Rev5-aligned lifecycle-tabel i CP/CPS
- Wrap-key Shamir 3-of-5 split + rotation hver 5. år
- Crypto-agility plan (PQC) eksplicit i policy-dokument

---

## Vinkel 3 — Certificate Lifecycle + Revocation

### Arkitektur set fra denne vinkel

Mads' §3.2 + §3.3 + tabellen i §3.2 (Root 15-20 år, Intermediate 10 år, Issuing 5 år, leaf max 1 år).

### Fund og risici

- **Leaf max 1 år er forældet for public TLS**: CABF Ballot SC-081v3 (vedtaget april 2025) reducerer max-leaf-validity progressivt:
  - **Fra 2026-03-15: max 200 dage** (allerede gældende — vi er over)
  - Fra 2027-03-15: max 100 dage
  - Fra 2029-03-15: max 47 dage
  
  Hvis FGDCorePKI nogensinde skal udstede public-trust TLS, skal leaf-policy alignes med disse trinvise reduktioner. For private/intern brug er 1 år acceptabelt teknisk, men automation skal designed til at understøtte 47-dages-cyklus uden refactoring (build for the future)
- **Root 15-20 år er for langt for crypto-agility**: med PQC på horisonten anbefales **maks 10-12 år** for nye Root CAs udstedt 2026, så re-keying-event kan ske inden ML-DSA-migration. Mozilla Root Store Policy (lokal industri-benchmark) accepterer op til 25 år men "should be" 15 år
- **Intermediate 10 år + Issuing 5 år overlap** kan skabe forvirrede chain-validations hvis ICA udløber midt i en Issuing-CA's levetid. Skal være matematisk konsistent: Issuing CA validity ≤ remaining ICA validity ved sub-CA udstedelse
- **CRL-strategi**: §3.3 "minimum én gang dagligt" er acceptabelt MVP, men ved mass-revocation event vokser CRL hurtigt. **CRL-størrelses-grænse**: hvis >5 MB bør delta-CRL eller partitioned CRL bruges (RFC 5280 §5)
- **OCSP-responder med delegated signing key (anbefalet)**: ikke specificeret om OCSP-key er på YubiHSM2 (begrænser slots, performance-impact) eller software-key med kort levetid (RFC 6960 §4.2.2.2.1 tillader). Performance-impact: hver OCSP-respons koster ~73 ms ECDSA P-256 på YubiHSM2 — ved bare 100 req/sek er HSM mættet
- **OCSP-stapling og must-staple**: for at undgå OCSP-server som SPoF/DoS-mål skal certifikater udstedes med **TLS Feature extension status_request (must-staple, RFC 7633)**. Dette er ikke nævnt i Mads' design
- **Certificate Transparency (RFC 9162)**: kun nævnt i Stefans prompt, ikke i Mads' dokument. **Hvis FGDCorePKI nogensinde skal være public-trust** → CT-logging er obligatorisk (Chrome, Apple, Mozilla policies). Hvis privat-only → ingen CT-krav, men dette skal være eksplicit FGD-beslutning
- **CAA records (RFC 8659)**: hvis FGDCorePKI udsteder for offentlige domæner, skal de domæner have CAA-record der whitelister FGDCorePKI. Ikke nævnt i Mads' dokument

### Anbefalinger

1. **Validity-tabel justeres**:
   - Root: 12 år (PQC-headroom)
   - Intermediate: 8 år
   - Issuing: 4 år
   - Leaf private/intern: 1 år (90 dage anbefalet for ACME-friendly automation)
   - Leaf public-trust (hvis aktuelt): følger CABF SC-081v3-trajektorie automatisk (200/100/47-dage)
2. **OCSP-responder dedikeret nøgle med 1-måneds-validity** + auto-renewal; software-key med kort levetid for at undgå HSM-flaskehals; OCSP-stapling med must-staple på alle leaf-certifikater
3. **Delta-CRL implementeres fra dag 1** for at håndtere mass-revocation-event uden CRL-blowup
4. **CT-strategy beslutning**: Mads + FGD afgør om FGDCorePKI nogensinde skal være public-trust; hvis ja → CT-logging-integration designed nu (RFC 9162, brug Sectigo/Google/Cloudflare logs eller eget tiled log per RFC 9162)
5. **CAA-record-policy**: tilføjes RA-validation (RFC 8659 mandatory check inden issuance for public domains)

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- Validity periods justeres for PQC-headroom; CABF-trajektorie indarbejdes hvis public-trust
- OCSP delegated-key + must-staple som standard; delta-CRL fra dag 1
- CT vs. private-only beslutning kræver FGD-stillingtagen

---

## Vinkel 4 — Disaster Recovery + Business Continuity

### Arkitektur set fra denne vinkel

Mads' §3.4 dækker DR-procedure for HSM, men ikke fuld BCP. Stefans 4 scenarier (a-d) er kun delvist adresseret.

### Fund og risici

- **(a) Tier 3 Issuing CA server fejler**: ikke eksplicit beskrevet. Antagelse: cold standby? warm standby? load-balanced active-active? Med "2x Online Issuing CA" i Stefans diagram kan det være active-active, men HSM-deling skaber kobling
- **(b) Tier 2 HSM fejler**: dette er Mads' egen flagged SPoF (§3.4 "HA-observation"). Ingen RPO/RTO-tal angivet. Det er en **blocker** uden afklaring
- **(c) Tier 1 offline-vault utilgængeligt**: hvis fysisk safe brand/oversvømmelse/katastrofe? Wrap-key og Root HSM begge tabt? Procedure for "Root rebuild" mangler. Dette er "fundamental restart"-scenario
- **(d) Alle wrap-keys tabt**: hvis wrap-key + alle backups væk → kan keys ikke restores; CA-genbygning fra ny ceremony, alle udstedte certifikater skal recreated. Dette er Mads' "alle wrap-keys tabt" — men ingen plan for at undgå det er specificeret (geografisk distribution af wrap-key shares)
- **YubiHSM2 hardware-replacement-strategi**: hvis Yubico EOL'er YubiHSM2 inden 2030, skal CA-keys migreres til ny HSM-platform. Vendor lock-in er ikke adresseret
- **YubiHSM2 hardware-grænser** påvirker DR-sizing: 256 object slots total, 128 KB total. Hvis Tier 2 ICA + Tier 3 Issuing CA + OCSP-responders + log-signing alle deler én HSM, hits man hurtigt slot-loft
- **RPO/RTO-mål mangler**: industri-baseline for høj-tillids PKI er typisk RTO 4 timer for Issuing CA, RTO 24 timer for ICA, RTO 72 timer for Root re-establishment. Mads har ikke kvantificeret

### Anbefalinger

1. **HA-pair Tier 2/3** anbefales: 2× YubiHSM2 per tier i active-passive med wrap-export sync. Cost: ~€2,400 ekstra (2× FIPS-units a ~€1,200) for begge tiers. Risk-reduktion: SPoF elimineres
2. **Geografisk wrap-key distribution**: Shamir 3-of-5 shares opbevares på minimum 3 separate fysiske lokationer (HQ safe + bank-safedeposit + secondary site). Komplet tab kræver 3+ samtidige fysiske kompromitteringer
3. **Quarterly DR-test obligatorisk** med dokumenteret restore-rapport: HSM-replacement, wrap-key-restore, CA-software-rekonstruktion, OCSP/CRL-verifikation. Trine attesterer
4. **Vendor-lock-in mitigation**: PKCS#11-baseret HSM-abstraktion i CA-software (EJBCA, Step-CA understøtter dette). Migration-procedure dokumenteres for at kunne flytte til alternative HSM-platforme (Thales, Entrust, AWS CloudHSM, Utimaco) ved Yubico EOL
5. **RPO/RTO formaliseres**: RTO 4t Issuing, 24t ICA, 72t Root; RPO 0 (alle udstedte certifikater logged før release så ingen "tabte" issuance ved svigt)

### VURDERING: **Blokerende**

### ANBEFALINGER:
- HA-pair beslutning kræves af FGD (cost vs. risk eksplicit)
- DR-test kvartalsvis med Trine-attestation
- RPO/RTO kvantificeres inden go-live

---

## Vinkel 5 — FIPS 140-3 Compliance-validering

### Arkitektur set fra denne vinkel

Mads har korrekt identificeret at FIPS 140-2 er obsolet (FIPS 140-2 → "Historical List" 21. sept 2026). Spørgsmålet er om YubiHSM2 FIPS 140-3 cert er **granted** i dag.

### Fund og risici

- **KRITISK FAKTA-FUND** (verificeret 2026-05-09): YubiHSM2's nuværende FIPS 140-2 cert (CMVP #3916) **udløb 2. maj 2026 — for 7 dage siden**. FIPS 140-3 cert er ikke endnu granted; Yubico's eget public statement er stadig "expected Q2 2026". Det betyder:
  - I dag (2026-05-09) er **YubiHSM2 ikke aktivt FIPS-valideret** under hverken 140-2 eller 140-3
  - Alt køb i dag risikerer en gap-periode hvor enheden ikke er valideret
  - Hvis FGDCorePKI skal go-live "FIPS 140-3 fra dag 1" → vi kan ikke i dag bekræfte at det er muligt
  - Yubico kan miste validation, eller den kan komme om uger/måneder. Vi ved det ikke fra public records
- **Algoritmer (Mads' tabel §4.1)**: RSA-4096, ECDSA P-384/P-521, SHA-384/SHA-512, AES-256-CCM — alle er FIPS-approved per **NIST SP 800-131A Rev 2 (Final)** og draft Rev 3. **Note**: Rev 3 er stadig draft (Oct 2024 IPD). Rev 2 er den autoritative reference indtil Rev 3 finaliseres. Begge accepterer disse algoritmer for "minimum 128-bit security strength" som transitionerer til mandatory ved end-of-2030
- **PQC-blindhed**: Root CA udstedt 2026 med RSA-4096 / ECDSA-P384 og 15-20 års validity vil overleve klassisk-krypto-safe-horizon. NIST har finaliseret ML-DSA (FIPS 204), ML-KEM (FIPS 203), SLH-DSA (FIPS 205) i aug 2024. **Klassisk Root med 20-års-løbetid = strategisk gæld**
- **YubiHSM2 understøtter ikke ML-DSA** i nuværende firmware (ECDSA, EdDSA, RSA only). Når PQC-migration kommer, skal YubiHSM2 enten få FW-update eller udskiftes
- **Audit trail FIPS 140-3 IG 2.4.B**: kræver immutable, exportable audit log fra modulet. YubiHSM2 har audit log på enheden men har historisk haft begrænsninger (log overskrives ved fuld buffer hvis ikke eksporteres rettidigt). Skal verificeres mod aktuel firmware

### Anbefalinger

1. **GO/NO-GO-beslutning ved CMVP-status**: Stefan + Mads tjekker CMVP active list **før hver pre-køb og før go-live**. Hvis 140-3 cert ikke er granted ved go-live, eskalerer Mads til FGD med tre options:
   - **A.** Vent på 140-3 cert (delay)
   - **B.** Køb non-FIPS YubiHSM2 nu, planlæg migration når 140-3 lander (kræver re-keying-ceremony)
   - **C.** Skift til alternativ FIPS 140-3-valideret HSM (Thales Luna, Entrust nShield, Utimaco) — kost-impact betydelig
2. **PQC-strategi formaliseres**: Root CA validity reduceres til **10-12 år** og crypto-agility (hybrid certifikater eller dual-CA-strategi) planlægges. Dokumentér at FGDCorePKI vil migrere til ML-DSA inden 2032-2035
3. **YubiHSM2-firmware tracking**: SRE/Engineering team monitorerer Yubico release notes for ML-DSA-support; planlagt firmware-upgrade-vinduer i CP/CPS §6
4. **Audit log export-frekvens**: minimum hver time eksporteres HSM audit log til immutable WORM-storage; alarm hvis HSM internal log >80% fuld

### VURDERING: **Blokerende**

### ANBEFALINGER:
- CMVP-status verificeres formelt inden go-live; FGD informeres om 140-2/140-3-gap
- Root validity ≤12 år for PQC-headroom
- Audit log export hver time, alarm ved 80% buffer

---

## Vinkel 6 — Network Segmentation + Isolation

### Arkitektur set fra denne vinkel

Mads' §4.2 ISO 27001 A.8.20-control nævner "Tier 2/3 CA-servere på isoleret netværkssegment; Tailscale VPN med ACL-kontrol". Stefans prompt udvider til VLAN, firewall, OCSP-eksponering, ACME/EST/SCEP-endpoints.

### Fund og risici

- **Tier 1 air-gapped**: ingen netværk overhovedet — kun hands-on USB til ceremony. OK i princip
- **Tier 2 (ICA) er online**: hvilket netværks-segment? Tailscale tail-net mellem CA-host og HSM-bridge? Hvis Tier 2 HSM er på Tailscale-net delt med Knowlagecentral-HSM (Bjarne) → blast-radius cross-application
- **Tier 3 (Issuing CA) eksponering**: ACME/EST/SCEP endpoints skal være tilgængelige fra applikations-netværk, men ikke fra internet uden API-gateway og rate-limit
- **OCSP-responder eksponering**: hvis OCSP er public (kunder skal verificere certifikater), så skal OCSP-host være DMZ med stærk DDoS-beskyttelse. Alternativt: **OCSP-stapling** (TLS server inkluderer OCSP-response i handshake) elimineres direkte OCSP-fetch fra klienter — anbefalet
- **CRL Distribution Point (CDP)**: HTTP (ikke HTTPS — Mads har ret) skal være public hvis ekstern brug; brug CDN (Cloudflare, Fastly) for at undgå direkte server-eksponering
- **Mads' "Tailscale" anbefaling** er rimelig for management-plane, men ikke for ACME/EST/SCEP-data-plane (klienter forventer standard HTTPS, ikke Tailscale-onboarding)
- **Zero-trust-principper**: hver CA-til-HSM-forbindelse skal authenticate både retninger; hver intern API-call skal have separate identitet (mTLS klient-cert eller workload-identity)
- **DNS-rebind / SSRF-vektorer**: hvis CA-software fetcher DN-informationer fra eksterne kilder under issuance, skal fetch gå gennem proxy med allow-list

### Anbefalinger

1. **Netværks-zone-model formaliseres**:
   - Zone A (offline air-gap): Tier 1 Root CA, ingen netværk
   - Zone B (HSM-mgmt): Tailscale tail-net, kun HSM-operatør + CA Admin workstations + HSM-bridges
   - Zone C (CA-internal): VLAN for Tier 2/3 CA-software ↔ HSM-bridge, dedicated firewall-rules
   - Zone D (RA + ACME-server): VLAN for RA portal og ACME/EST/SCEP-endpoints; reverse-proxy
   - Zone E (DMZ): OCSP-responder + CRL CDN-origin (DDoS-beskyttet)
   - Zone F (audit): SIEM/log-aggregator, read-only fra alle zones
2. **Firewall regelsæt**: default-deny + explicit allow per zone-pair. mTLS mellem zones for alle service-til-service-kald
3. **OCSP-stapling med must-staple** standard i alle udstedte certifikater (RFC 7633) — eliminerer OCSP DDoS-overflade
4. **CRL via CDN** (Cloudflare/Fastly) — origin CA-server eksponeres ikke
5. **SSRF-mitigation**: alle eksterne network-fetches fra CA-software gennem proxy med allow-list (OWASP API Security Top 10 — SSRF er #7)

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- Zone-model A-F implementeres med eksplicit firewall-regelsæt
- Must-staple som default på alle certifikater
- CRL via CDN; OCSP DMZ med DDoS-protection

---

## Vinkel 7 — SOD + Access Control

### Arkitektur set fra denne vinkel

Mads' §1 + §5 har grundstrukturen. Stefans prompt beder om teknisk implementation: RBAC i CA-software, YubiHSM2 authentication keys per rolle, M-of-N split keys, PAW (Privileged Access Workstations), JIT-access til Root CA.

### Fund og risici

- **YubiHSM2 authentication-key model**: hver "rolle" har egen authentication-key med session-scope. YubiHSM2 understøtter "Domains" og "Capabilities" (fine-grained) — Mads' SOD-matrix mapper ikke til konkrete YubiHSM2 Capabilities. Eks: HSM-operatør har `delete-asymmetric-key` + `wrap-data` + `set-option`; CA Admin har `sign-pkcs` + `sign-eddsa` (men ikke delete eller wrap)
- **M-of-N implementation**: YubiHSM2 understøtter ikke native Shamir-split for object-aktivering; "M-of-N" i ceremony-context henviser til **wrap-key shares** opbevaret på smartcards (FIDO2 nøgler eller PIV-kort). Implementation-detail mangler i Mads' dokument
- **PAW (Privileged Access Workstations)**: ikke nævnt. CA Admin og HSM-operatør skal arbejde fra hardened, dedicated workstations (ingen email, ingen browser, ingen office-suite). Microsoft-koncept overført til Linux/macOS (immutable OS, kun signed binaries)
- **JIT (Just-In-Time) access til Root CA**: ikke nævnt. Root CA-aktivering skal være tids-bundet (1-4 timers vindue ved ceremony) og logged. Implementation: HSM-bridge har "ceremony-mode" der kun aktiveres af CA Manager + Security Officer (dual-control)
- **RA-godkendelse er den klassiske SOD-flaske**: hvis RA og CA Admin er samme person, eller hvis CA Admin kan bypass RA-køen ved hastesager → SOD brudt. Teknisk control: RA-portal skal være separat service med eget identity-system; CA-software accepterer kun signed RA-tokens
- **Audit-log read-access**: PKI Auditor (Trine) skal have read-only adgang til alle audit-logs men ingen skriveadgang. Hardware-WORM eller cryptographic-chained-log forhindrer manipulation

### Anbefalinger

1. **YubiHSM2 capability-mapping** dokumenteres i CP/CPS §6 (per-rolle authentication-key med eksplicit capability-set):
   - Root Ceremony-key (kun under ceremony, dual-control aktivering): `generate-asymmetric-key`, `export-wrapped`, `sign-pkcs`
   - HSM-operatør-key: `wrap-data`, `unwrap-data`, `delete-asymmetric-key`, `set-option`, `audit-set-option`
   - CA Admin-key (Tier 2/3): `sign-pkcs`, `sign-eddsa`, `sign-attestation-certificate`
   - OCSP-responder-key: `sign-ocsp` only
   - Auditor-key: `audit-log-export` only (read-only)
2. **PAW for CA Admin og HSM-operatør**: dedicated NixOS / Fedora Silverblue / macOS-with-DEP-mgmt; full-disk-encryption; YubiKey FIDO2 for OS-login; ingen privater browser/email
3. **JIT Root-aktivering**: CA Manager + Security Officer aktiverer "ceremony-mode" på offline-host via HSM-bridge; auto-deactivation efter 4 timer eller ved manuel close. Logged til audit
4. **Quarterly access-review** (Mads' §3.7 — confirmed) udvides til at verificere YubiHSM2 capability-set per authentication-key (drift detection)
5. **Audit-log immutability**: kombination af HSM-internal log + WORM-storage + cryptographic chained log (Merkle-tree, à la RFC 9162 model) for tamper-evidence

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- YubiHSM2 capability-mapping per rolle dokumenteres i CP/CPS
- PAW for CA Admin og HSM-operatør obligatorisk
- JIT Root-aktivering med 4-timers auto-deactivation

---

## Vinkel 8 — Audit + Monitoring

### Arkitektur set fra denne vinkel

Mads' §4.5 + §3.7 dækker audit-trail-krav (7 år, WORM, signeret) og kvartalvis access-review. Stefans prompt udvider til SIEM, alert-thresholds, Trines audit-frekvens.

### Fund og risici

- **Logs der skal samles** er fragmenteret across: HSM-internal log (export hourly), CA-software log (issuance/revocation), RA-portal log (approve/reject), OS audit (auditd/macOS endpoint), netværk (firewall, NIDS), fysisk PACS (door access), ceremony-protokoller (signede papirer + video)
- **Mads har ikke specificeret SIEM-platform**: optioner inkluderer Splunk, Elastic, Wazuh, OpenObserve. For on-premises FIPS-context anbefales platform med audit-trail integrity (signed log forwarding)
- **Alert-thresholds udefineret**: hvad udløser alarm?
  - HSM internal log >80% fuld (mistet logs = compliance-brud)
  - >5 RA-rejects fra samme rekvirent på 5 minutter (brute force)
  - Issuance for cert med usædvanlig validity (>policy-max)
  - Issuance for high-privilege Subject (kun CA-cert eller code-signing) uden ceremony
  - HSM-session uden authentication-key i allowed-list
  - CRL eller OCSP signing uden delegated-key matching
- **Trines audit-frekvens**: Mads' §3.7 = kvartalvis access review. Fuld PKI-audit (CP/CPS-efterlevelse, certificate sample-review, ceremony-protokol-verifikation) bør være **årligt** (matcher SOC 2 Type II 12-måneders periode) + ad-hoc ved incidents
- **WebTrust for CAs**: industri-standard for ekstern PKI-audit. Hvis FGDCorePKI nogensinde public-trust → WebTrust-audit obligatorisk (audit firms: Deloitte, KPMG, EY der har WebTrust-akkreditering). Trine kan ikke selv attestere WebTrust uden akkreditering — det skal eksternt firma
- **Kontinuerlig monitoring vs. periodisk audit**: industri bevæger sig mod continuous control monitoring (Vanta, Drata, Anecdotes) — for FGDCorePKI er manuel kvartalvis review tilstrækkeligt MVP, men automation bør indplaneres

### Anbefalinger

1. **SIEM-platform-valg** før go-live: Wazuh (open source, on-prem, FIPS-friendly) eller Elastic Security. SIEM-ingestion fra alle kilde-systemer over signed log shipping
2. **Alert-katalog** dokumenteres med thresholds + on-call-rotation. Mads owns alert-design; SRE (Engineering team) implementerer
3. **Trines audit-scope**:
   - Kvartalvis: access-review, sample-cert-review (random 10), HSM audit-log review
   - Årligt: fuld CP/CPS-efterlevelses-audit, ceremony-protokol-review, alle revocation-events review
   - Ad-hoc: ved enhver suspected incident
4. **WebTrust-akkrediteret ekstern audit**: hvis FGDCorePKI går public-trust, planlæg ekstern audit-firma engagement minimum 6 måneder inden go-live (audit period typisk 6+ måneder)
5. **Continuous compliance**: pilot Vanta eller Drata for SOC 2 evidence-collection automation efter MVP

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- SIEM-platform-valg + alert-katalog inden go-live
- Trines audit-scope: kvartalvis + årligt + ad-hoc
- WebTrust-firma engagement hvis public-trust planlægges

---

## Vinkel 9 — Integration Security (Engineering Team)

### Arkitektur set fra denne vinkel

Mads' §2 (Engineering Team) + §6.2 Track A/B/C (Pouls research). Stefans prompt: hvordan sikres applikationers adgang til PKI sikkert via ACME/SCEP/EST + cert-management tooling.

### Fund og risici

- **Protocol-valg**:
  - **ACME (RFC 8555)**: best-of-breed for moderne automation; brugt af Let's Encrypt; understøttes natively af cert-manager (Kubernetes), Certbot, lego, win-acme. **Anbefalet for nye integrationer**
  - **EST (RFC 7030)**: enterprise-fokuseret; støtter TLS-baseret enrollment; mindre udbredelse end ACME men stærkere identity-binding
  - **SCEP (RFC 8894)**: legacy; brugt i mobile/MDM-context (Microsoft Intune, Jamf); svagere i moderne sikkerhed (HTTP-baseret default; signature-only validation)
  - **Manuel CSR**: sidste-resort for engangs-certifikater
- **CA-software med ACME-server-support**:
  - **EJBCA Community/Enterprise**: ACME, SCEP, EST, CMP — alle standards. PKCS#11 support inkl. YubiHSM2 (Yubico har explicit usage-guide)
  - **Step-CA**: ACME-first, modern, Go-based; PKCS#11 support tilstede men ikke YubiHSM2-specifik testet i alle scenarier (kræver verifikation)
  - **Dogtag**: feature-rich men kompleks; PKCS#11 support; legacy enterprise
  - **HashiCorp Vault PKI**: god til mid-tier, men typisk ikke som Tier 1/2 i three-tier setup (mere som Tier 3 issuing til workloads)
  - **Microsoft AD CS**: Windows-centric; ikke relevant for Linux/macOS-centric FGD
- **Engineering team's SDLC for PKI-integrationer**:
  - Code review obligatorisk (Mads + Engineering Lead approval)
  - SAST + DAST på alle PKI-related code
  - Dependency-scan (Snyk, Trivy) — ACME/EST libs er supply-chain-targets
  - Secrets-scan (TruffleHog, Gitleaks) på alle commits
- **Certificate pinning**: applikationer der pinner CA-public-key er **anti-fragile** men hindrer rotation. Anbefales: pin Intermediate CA i stedet for leaf (giver headroom for leaf-rotation uden klient-update). HPKP er deprecated for browsers — for backend/IoT er pinning stadig relevant
- **API-gateway til PKI**: ACME/EST-endpoints skal bag API-gateway med rate-limit (RFC 9110 + cert-issuance specific):
  - Per-applikation: max 100 cert-issuances/dag
  - Per-IP: max 10 ACME challenges/min
  - Global: max 1000 issuance/time (begrænset af YubiHSM2 ~7 RSA-4096-sign/sek = max 25.000/time hardware ceiling, men policy-grænse lavere)
- **Misbrug-detektion**: pattern-detection (samme rekvirent forsøger forskellige Subject-felter, validity-stretching, algorithm-downgrade). RA-portal logger alle attempts + ML-baseret anomaly-detection over tid
- **Secrets management for private keys i applikationer**: applikationer bør **aldrig** persistere private key i filsystem. Anbefales: HashiCorp Vault, AWS Secrets Manager, ephemeral keys (cert lifetime = key lifetime), eller TPM/Secure Enclave på workload-host
- **Performance-realitet**: YubiHSM2 ceiling er ~7 RSA-4096-sign/sek per HSM. Tier 3 Issuing CA + OCSP-responder + log-signing samme HSM = HSM-mætning ved ~5 issuances/sec. Skalering kræver multiple HSMs per tier 3

### Anbefalinger

1. **Protocol-policy**: ACME default for nye integrationer; EST for enterprise/IoT med stronger identity; SCEP kun legacy/MDM-required; manuel CSR kun for engangs-tilfælde
2. **CA-software-anbefaling**: **EJBCA Enterprise** for FGDCorePKI Tier 2/3 (broadest protocol support, YubiHSM2-tested, commercial support, FIPS-friendly). Alternativ open source: EJBCA Community + Step-CA combo
3. **API-gateway med rate-limit** + **per-applikation issuance-quota** (Mads + Engineering Lead designer policy)
4. **Pin Intermediate CA, not leaf** — dokumenteres i developer-guide
5. **Ephemeral cert-pattern**: hvor muligt, leaf cert lifetime = workload session lifetime; private key aldrig persisteret
6. **HSM-throughput-test** før go-live: Engineering team kører load-test mod YubiHSM2 for at bekræfte ~7 RSA-4096-sign/sek og dimensionere antal HSMs per tier korrekt

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- ACME default; EJBCA som CA-software
- API-gateway + rate-limit + issuance-quota per applikation
- HSM-throughput-test og sizing inden go-live

---

## Vinkel 10 — Incident Response + CA Compromise

### Arkitektur set fra denne vinkel

Mads' §3.6 har en grov tabel med 4 alvorlighedsgrader. Stefans prompt udvider med GDPR art. 33, communication plan, retainer.

### Fund og risici

- **(a) Leaf-cert kompromittering**: Mads' "lav" niveau (1-times revocation). Procedure rimelig
- **(b) Issuing CA kompromittering**: Mads' "høj" niveau. **Mass revocation** af alle leaf-certifikater udstedt af kompromitteret Issuing CA — dette skaber CRL-blowup (potentielt MB-størrelse). Delta-CRL eller partitioned CRL (RFC 5280 §5) påkrævet for at klienter kan håndtere
- **(c) Intermediate CA kompromittering**: Root CA signerer ny ICA. Notify cert-users via out-of-band kanaler (email, status-page, kunde-portal). **Time-sensitive**: ny ICA + leaf re-issuance kan tage uger; fallback-CA?
- **(d) Root CA kompromittering**: nukleart. Hele PKI re-builds fra bunden. **Communication plan til alle afhængige parter** (kunder, leverandører, regulators) — kommunikation er den primære arbejdsbyrde, ikke teknik
- **GDPR art. 33-implikationer**: hvis kompromitterede certifikater er knyttet til persondata-flows (TLS-cert beskytter persondata-API, eller cert identificerer fysisk person), så **72-timers notifikations-pligt** til Datatilsynet. Dorthe (DPO) skal være i incident-response-plan fra dag 1
- **Communication plan mangler i Mads' design**:
  - Intern: hvem kommunikerer til hvem hvornår (incident commander = Mads, PR/legal = FGD)
  - Ekstern kunder: status-page (statuspage.io eller Cachet self-hosted), email-blast, kundeportal
  - Regulators: Datatilsynet, evt. ENISA, branchespecifikke (Finanstilsynet hvis finansielle kunder)
  - Public: pressemeddelelse hvis omfang stort
- **Retainer med PKI incident-response specialist**: ikke nævnt i Mads. Anbefales: Improsec (DK) eller NCC Group / Mandiant retainer for forensic CA-investigation. Cost: ~€2.000-5.000/år for retainer (uden faktiske brug); ~€500-2.000/dag ved incident
- **Tabletop-eksercise**: planned scenario-walkthroughs to validate incident response. Mads har ikke planlagt disse
- **CA-key-compromise rapport**: hvis nogensinde Root eller ICA er bekræftet kompromitteret, skal **incident report** publiceres (analog til Mozilla's incident-tracking for public-trust CAs). Selv for private PKI er gennemsigtig rapportering best practice

### Anbefalinger

1. **Incident response plan** udvides til at inkludere:
   - Trigger-table per scenario (a)-(d) med konkrete tidsfrister og roller
   - Communication tree: incident commander → roller → eksterne stakeholders
   - Statuspage (statuspage.io eller Cachet) klar inden go-live
   - Templates for kunde-mails og press-statements
2. **GDPR art. 33-integration**: Dorthe (DPO) modtager auto-notification ved alle "høj" og "kritisk" incidents; 72-timers vurderingscyklus dokumenteret
3. **Retainer med Improsec eller equivalent**: Frederik (pen-test-koordinator) ejer vendor-management; årlig retainer eskaleres til Mads + FGD (cost decision)
4. **Tabletop quarterly**: Mads afholder; deltager: CA Admin, HSM-operatør, RA, Trine (PKI Auditor), Dorthe (DPO). Scenarios roteres: leaf-compromise, ICA-compromise, Root-compromise, insider threat
5. **Incident-report-procedure**: hver "kritisk" incident udløser public-rapport (eller intern hvis privat-PKI) inden 30 dage; chain-of-custody af forensic evidence

### VURDERING: **Kræver afklaring**

### ANBEFALINGER:
- Communication plan + statuspage + templates inden go-live
- Dorthe i auto-notification-loop for alle høj/kritisk incidents (GDPR 72t)
- Retainer med PKI IR-specialist + quarterly tabletops

---

## Samlet vurdering

### Heatmap af de 10 vinkler

| Vinkel | Vurdering |
|---|---|
| 1. Physical Security | Kræver afklaring |
| 2. Key Management Lifecycle | Kræver afklaring |
| 3. Certificate Lifecycle + Revocation | Kræver afklaring |
| 4. DR + Business Continuity | **Blokerende** |
| 5. FIPS 140-3 Compliance | **Blokerende** |
| 6. Network Segmentation | Kræver afklaring |
| 7. SOD + Access Control | Kræver afklaring |
| 8. Audit + Monitoring | Kræver afklaring |
| 9. Integration Security | Kræver afklaring |
| 10. Incident Response | Kræver afklaring |

**0 vinkler er Acceptabel** som beskrevet i Mads' nuværende dokument. Det er **ikke** en kritik af Mads' arbejde — Mads' dokument er et organisations-/governance-krav-dokument, ikke en fuld arkitektur-spec. Analysen viser hvor arkitektur-arbejdet skal fokuseres næste fase.

### Prioriterede blockers (skal løses inden go-live)

**B1. CMVP-status for YubiHSM2 (Vinkel 5)** — `BLOKERENDE`
- Fakta: FIPS 140-2 cert udløb 2. maj 2026; FIPS 140-3 cert ikke endnu granted
- Action: Stefan + Mads tjekker CMVP active list ugentligt; FGD beslutter om delay (vente på 140-3) eller midlertidig non-FIPS-MVP med planlagt re-keying
- Owner: Mads (CISO) + Stefan (koordinering); FGD-godkendelse

**B2. Tier 2 HSM SPoF (Vinkel 4)** — `BLOKERENDE`
- Fakta: Mads har selv flagget; ingen RPO/RTO-tal; HA-pair koster €2.400 ekstra
- Action: FGD beslutter A) HA-pair eller B) accept SPoF med dokumenteret RTO 24-72t
- Owner: Mads + Poul (cost/risk-matrix); FGD-godkendelse

**B3. PQC-blindhed for 15-20-års Root (Vinkel 5 / Vinkel 2)** — `BLOKERENDE` (langsigtet)
- Fakta: NIST har finaliseret ML-DSA aug 2024; Root med 20-års-løbetid overlever klassisk-krypto-safe-horizon
- Action: Reducer Root validity til 10-12 år; dokumenter PQC-migration-strategi i CP/CPS; planlagt YubiHSM2-firmware-tracking
- Owner: Mads + Poul (PQC-roadmap); FGD-godkendelse

### Kræver afklaring (skal afklares inden CP/CPS-godkendelse)

**A1. Datacenter-tier + safe-grade** (Vinkel 1) — Mads + FGD beslutter
**A2. Air-gap-formalisering for Tier 1** (Vinkel 1) — Mads dokumenterer
**A3. Wrap-key Shamir 3-of-5 split** (Vinkel 2) — Mads + Bjarne + Laila design
**A4. Authentication-key rotation policy** (Vinkel 2) — Mads + HSM-operatør policy
**A5. CABF SC-081v3-trajektorie** (Vinkel 3) — afhænger af FGDs public-trust-beslutning
**A6. CT-strategy** (Vinkel 3) — FGD beslutter public-trust eller privat-only
**A7. Validity-justering** (Vinkel 3) — FGD godkender Root 12 år / ICA 8 år / Issuing 4 år
**A8. CA-software-valg** (Vinkel 9) — Poul Track A research → Mads anbefaling → FGD-go
**A9. Netværks-zone-model A-F** (Vinkel 6) — Mads + Engineering Team-lead design
**A10. YubiHSM2 capability-mapping per rolle** (Vinkel 7) — Mads + Bjarne dokumenterer
**A11. SIEM-platform-valg** (Vinkel 8) — Mads + Engineering Team
**A12. WebTrust-strategy** (Vinkel 8) — afhænger af public-trust-beslutning
**A13. Communication plan + statuspage** (Vinkel 10) — Mads + FGD design

### Prioriteret handlingsliste til Laila + Stefan + FGD

**Fase 0 — afklaring (1-2 uger fra dato):**

1. **FGD-go på 5 strategiske beslutninger** (Mads' §7 + Pouls B1-B3):
   - FIPS 140-3 strategi: vent på cert eller MVP nu med plan?
   - HA-pair Tier 2: ja/nej
   - Public-trust eller privat-only? (påvirker CT, CABF, WebTrust, validity)
   - Root validity: 12 år (PQC-headroom) vs. 15-20 år (Mads' default)
   - Bjarne udvidet mandat eller separat HSM-operatør for FGDCorePKI?

2. **Pouls research-tracks** (parallel):
   - Track A: CA-software (EJBCA vs. Step-CA vs. Dogtag) — Pouls anbefaling
   - Track B: HA-cost/risk-matrix — Pouls anbefaling
   - Track C: Cert-management-tooling (cert-manager, Vault, Keyfactor) — Pouls anbefaling

3. **Laila rolle-design** (Mads' §6.1):
   - CA Administrator (FGDCorePKI) — design
   - HSM-operatør tillæg (Bjarne) — design
   - Key Ceremony Officer — design
   - RA (Registration Authority) — design
   - Trine PKI Auditor-udvidelse — design
   - AO og AP for FGDCorePKI — design med PKI-specifikke SOD-begrænsninger

**Fase 1 — arkitektur-spec (3-6 uger):**

4. **CP/CPS draft** (RFC 3647-aligned) — Mads owns; Engineering team udkast §6 (technical security controls); Trine reviewer; FGD-godkender
5. **Network zone-model A-F** detaljeret arkitektur (Mads + Engineering Team)
6. **Capability-matrix** YubiHSM2 per rolle (Mads + Bjarne)
7. **Communication plan + statuspage + templates** (Mads + FGD)

**Fase 2 — pre-go-live (4-8 uger):**

8. **HSM-throughput-test** (Engineering team)
9. **DR-test #1** (kvartalvis cycle starter; Trine attesterer)
10. **Tabletop exercise #1** (Mads afholder)
11. **CMVP-status-tjek** umiddelbart inden go-live (Stefan + Mads)
12. **Root CA key ceremony** (efter alle ovenstående er på plads)

### Konsistens-tjek mod Mads' brief

Pouls analyse er **konsistent med Mads' organisatoriske krav** men finder yderligere arkitektur-/teknik-niveau-gaps Mads' dokument bevidst ikke har adresseret (Mads' scope = governance + roles; Pouls scope = arkitektur + risikoanalyse). Pouls anbefalinger overskriver ikke Mads' SOD-matrix eller rolle-design — de udvider og operationaliserer dem teknisk.

### Note om FGD's 10-vinkler-anmodning

Stefans prompt strukturerede analysen klart, og hver vinkel afdækker et reelt risk-domain. Men flere vinkler overlapper materielt (fx Vinkel 2 + 5 om PQC; Vinkel 4 + 10 om recovery; Vinkel 6 + 9 om netværks-eksponering). Hvis FGD foretrækker en synthesized arkitektur-rapport i stedet for 10 separate sektioner som næste leverance, kan Poul flette anbefalingerne til en samlet teknisk arkitektur-spec. Det er en pakkering-beslutning — substansen er den samme.

---

## Kilder (verificeret 2026-05-09)

**Primære standarder:**
- [RFC 3647 — Internet X.509 PKI Certificate Policy and CPS Framework](https://www.rfc-editor.org/rfc/rfc3647)
- [RFC 5280 — Internet X.509 PKI Certificate and CRL Profile](https://datatracker.ietf.org/doc/html/rfc5280)
- [RFC 6960 — OCSP](https://www.rfc-editor.org/rfc/rfc6960)
- [RFC 7030 — EST](https://www.rfc-editor.org/rfc/rfc7030)
- [RFC 7633 — TLS Feature Extension (must-staple)](https://www.rfc-editor.org/rfc/rfc7633)
- [RFC 8555 — ACME](https://www.rfc-editor.org/rfc/rfc8555)
- [RFC 8659 — CAA records](https://www.rfc-editor.org/rfc/rfc8659)
- [RFC 8894 — SCEP](https://www.rfc-editor.org/rfc/rfc8894)
- [RFC 9162 — Certificate Transparency v2.0](https://www.rfc-editor.org/rfc/rfc9162.html)

**NIST:**
- [NIST SP 800-57 Part 1 Rev 5 — Key Management](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final)
- [NIST SP 800-131A Rev 2 (Final) — Cryptographic Algorithm Transitions](https://csrc.nist.gov/pubs/sp/800/131/a/r2/final)
- [NIST SP 800-131A Rev 3 (Draft, Oct 2024)](https://csrc.nist.gov/pubs/sp/800/131/a/r3/ipd)
- [FIPS 204 — ML-DSA (Module-Lattice-Based Digital Signature)](https://csrc.nist.gov/pubs/fips/204/final)
- [NIST CMVP — Cryptographic Module Validation Program](https://csrc.nist.gov/projects/cryptographic-module-validation-program)
- [CMVP cert #3916 — YubiHSM2 (140-2, sunset 2026-05-02)](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/3916)

**Industri:**
- [CA/Browser Forum Ballot SC-081v3](https://cabforum.org/2025/04/11/ballot-sc081v3-introduce-schedule-of-reducing-validity-and-data-reuse-periods/)
- [DigiCert — TLS Certificate Lifetimes 47 Days Timeline](https://www.digicert.com/blog/tls-certificate-lifetimes-will-officially-reduce-to-47-days)
- [Yubico — YubiHSM2 FIPS product page](https://www.yubico.com/product/yubihsm-2-fips/)
- [Yubico — EJBCA with YubiHSM2 usage guide](https://developers.yubico.com/YubiHSM2/Usage_Guides/EJBCA_with_YubiHSM2.html)
- [YubiHSM2 User Guide (Apr 2026)](https://docs.yubico.com/hardware/yubihsm-2/hsm-2-user-guide/index.html)

**Compliance frameworks:**
- ISO/IEC 27001:2022 (Annex A controls — refereret i Mads' §4.2)
- AICPA SOC 2 Type II Trust Services Criteria 2017 (CC-controls — refereret i Mads' §4.3)
- WebTrust for Certification Authorities (CPA Canada/AICPA)

---

*Dokument-ejer: Poul (analytiker)*
*Distribueret til: Mads (CISO), Laila (HR), Stefan (koordinering), FGD (godkendelse)*
*Næste revision: Efter FGD-go på 5 strategiske beslutninger i §Fase 0*
*Kopi: governance/design-docs/ (compliance-transparens for Trine + Dorthe)*
