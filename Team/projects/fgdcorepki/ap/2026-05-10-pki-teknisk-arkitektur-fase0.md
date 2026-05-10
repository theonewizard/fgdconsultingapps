---
dato: 2026-05-10
type: pki-teknisk-arkitektur
projekt: FGDCorePKI
fase: 0
forfatter: Application Provider (FGDCorePKI)
status: udkast
---

# FGDCorePKI — teknisk arkitektur, Fase 0 (udkast)

## 0. Formål, scope og ramme

Dette dokument er **AP's tekniske arkitektur-overblik for Fase 0**. Det dækker CA-hierarki-topologi, YubiHSM2-integration, protokol-scope, OCSP/CRL-distribution, audit-log-arkitektur, algoritme-begrundelse og MVP-afgrænsning.

**Det er ikke** CP/CPS, ikke compliance-erklæring, ikke engineering-timeline. Det er teknisk grundlag for:

- Pouls videre arkitektur-research (Tracks A/B/C)
- Mads' security-review og compliance-vurderinger
- Dorthes DPIA-input (DPIA-1 ceremony-recording — F1, F5, F9)
- AO's roadmap-prioritering og FGD-godkendelse
- Engineering Team Lead's implementerings-plan

**Roller AP IKKE udfylder her:** CA Administrator (drift), HSM-operatør (eksekvering), Engineering Team Lead (build-timeline), Mads (compliance-konklusion), Dorthe (GDPR-konklusion), AO (roadmap-prioritering).

### Låste forudsætninger (FGD-beslutninger der ikke re-scopes i dette dokument)

| # | Beslutning | Reference | Konsekvens for arkitekturen |
|---|---|---|---|
| L1 | EU-only OCSP/CRL-hosting | FGD-beslutning 2026-05-10 i `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ocsp-crl-cdn.md` §9 (Afslutning) | Ingen tredjelands-CDN (Cloudflare US, Fastly US, Akamai global). Ingen hybrid-failover til tredjeland. EU-residency er hard requirement |
| L2 | Non-FIPS MVP for Fase 0 | FGD-beslutning 2026-05-09 (jf. mandat-tekst og Pouls Vinkel 5) | YubiHSM2 non-FIPS-variant er teknisk platform i Fase 0; FIPS 140-3-ready design (ingen FIPS-svage algoritmer); re-keying-event planlagt når CMVP-grant lander |
| L3 | Tre-lags hierarki: Root (offline) / Intermediate (HA-par 2× HSM) / Issuing (online 1× HSM) | Stefan-prompt 2026-05-10 + Pouls Vinkel 4 | Tier 2 HA-par adresserer Pouls B2 (Tier 2 SPoF) — design afspejler HA fra dag 1 |
| L4 | Engineering Team Lead er separat rolle fra AP (Option B) | Stefan-prompt 2026-05-10 | AP designer arkitektur; Lead ejer build-/release-plan; AP er ikke teknisk leder af engineering team |
| L5 | Algoritme-konvolut: ECDSA P-384 (primær) eller RSA 4096; SHA-384/SHA-512 minimum | Stefan-prompt + Pouls Vinkel 5 | Ingen FIPS-svage algoritmer (SHA-1, RSA-2048, ECDSA P-256 er ikke i scope for CA-keys) |
| L6 | Dual-control ved Root CA-operationer: CA Administrator + HSM-operatør + Security Officer (2nd-auth) | Stefan-prompt 2026-05-10 | Root-aktivering kræver tre-personers samtidighed; teknisk implementering via separate authentication-keys på YubiHSM2 + dokumenteret ceremony-script |

### Åbne afhængigheder der ikke afgøres i dette dokument

| # | Spørgsmål | Ejer | Reference |
|---|---|---|---|
| O1 | Ceremony-recording-arkivering: lokation + krypterings-arkitektur | DPIA-1 (Dorthe) — F5 + F9 | `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ceremony-recording.md` |
| O2 | Ceremony-protokol/recording retention: 7-10 år vs. 20 år | DPIA-1 (Dorthe) — F2; Mads + AO; FGD | DPIA-1 §4 + Pouls Vinkel 8 |
| O3 | Root validity 10-12 år (PQC-headroom) vs. 15-20 år | Mads + FGD (Pouls B3) | Pouls Vinkel 3 + Vinkel 5 |
| O4 | Public-trust eller privat-only PKI | FGD | Pouls Vinkel 3 + Vinkel 8 |
| O5 | CA-software-valg (EJBCA Enterprise vs. EJBCA Community + Step-CA combo) | Poul Track A → Mads → FGD | Pouls Vinkel 9 |
| O6 | Public-CMVP-grant for YubiHSM2 FIPS-variant — re-keying-trigger | Mads (overvågning); FGD-go | Pouls Vinkel 5 (B1) |
| O7 | Datacenter-tier + safe-grade for Tier 1 | Mads + FGD | Pouls Vinkel 1 |

---

## 1. CA-hierarki-design

### 1.1 Topologi

```
                    Tier 1: Root CA (offline / air-gapped)
                          1× YubiHSM2 (non-FIPS, Fase 0)
                          - Genereres ved key ceremony
                          - Aktiveres kun ved sub-CA-signing eller CRL-resigning
                          - Ingen netværk; USB hands-on
                          |
                          | (signerer Intermediate-CA-cert)
                          v
              Tier 2: Intermediate CA (online, HA-par)
                          2× YubiHSM2 i active-passive (eller active-active
                          for sign-load) — Pouls anbefalede HA-pair (B2 lukket)
                          - Wrap-export sync mellem HSM A og HSM B
                          - Failover-test kvartalvis
                          - EU-residency lokation (jf. L1 + Pouls Vinkel 1)
                          |
                          | (signerer Issuing-CA-cert + signerer
                          |  Intermediate CRL)
                          v
                Tier 3: Issuing CA (online, single HSM Fase 0)
                          1× YubiHSM2
                          - Issuance via ACME (primær) + manuel CSR
                          - Signerer leaf-certs + Issuing-CRL + OCSP-responses
                          - HSM-throughput-grænse: ~7 RSA-4096-sign/sek;
                            ~30+ ECDSA P-384-sign/sek (skal verificeres af
                            Engineering Lead i load-test inden go-live)
                          - Tier 3 HA-udvidelse: backlog for Fase 1 (afventer
                            usage-data og kapacitets-vurdering — AO's roadmap)
```

### 1.2 Tier 1 — offline Root CA

**Air-gap-definition (foreløbig — afventer Mads' formalisering, Pouls Vinkel 1, A2):**

- Dedikeret hardware uden NIC, uden Wi-Fi, uden Bluetooth, uden cellular modem
- Ingen netværks-stak aktiv (boot fra read-only media; firewall blokerer alle interfaces selv hvis fysisk fjernet udgår)
- Hands-on USB-overførsel kun ved ceremony (signed CSRs ind, signed sub-CA-certs ud)
- USB-overførsel via dedikeret transfer-medie (write-once optisk eller signeret USB med tamper-evident seal)
- Tier 1 host opbevares i fysisk safe (grade afventer Mads + FGD, jf. O7)

**Aktiverings-procedure (designspec — KCO godkender + eksekverer):**

1. Dual-control safe-adgang (to separate personer, hver med egen lås/PIN-share)
2. Tier 1 host bootes fra read-only media; clock-init fra trusted-time-source (lokal RTC eller NTP-from-airgap-relay — afventer Pouls Vinkel 1)
3. YubiHSM2 attaches via USB
4. Dual-control HSM-aktivering: CA Admin authentication-key + Security Officer authentication-key (begge skal være authenticated i samme session for Root-signing capability)
5. Signing-operation udføres; resultatet skrives til transfer-medie
6. HSM detaches; host shutdown; safe-genlukning med ny tomandsregel-protokol
7. Audit-log fra HSM eksporteres til transfer-medie og forwardes til online audit-pipeline

**Ceremony-typer for Tier 1:**

| Ceremony | Frekvens (foreløbig) | Dual-control | Optagelse (DPIA-1 åben) |
|---|---|---|---|
| Root key generation | 1× per Root-validity-cyklus (10-20 år, jf. O3) | CA Admin + HSM-operatør + Security Officer + KCO | JA (afventer DPIA-1) |
| Intermediate-CA-signing | Ved Intermediate rotation (~hvert 4-8. år, afh. af L3/O3) | Samme | JA |
| Tier 1 CRL re-issuance | Mindst 1× pr. 12 måneder (Tier 2 CA-revocation hvis nødvendigt) | Samme | JA |
| Root re-keying (PQC, kompromis) | Ad hoc — undtagelses-event | Samme + udvidet vidne-protokol | JA |

### 1.3 Tier 2 — Intermediate CA, HA-par

**HA-design:**

- 2× YubiHSM2 (HSM-A primær, HSM-B sekundær) i samme EU-region men separate fysiske rack/sikre kabinetter
- Begge HSM'er har samme wrap-key og samme Intermediate-private-key (importeret som wrapped-export under ceremony)
- CA-software (Tier 2 ICA-host) prøver primær først; failover til sekundær ved timeout/fejl
- Wrap-export-sync: når Tier 2 nøglemateriale ændres (re-keying), eksporteres wrapped key til offline media og importeres til HSM-B i kontrolleret session
- Kvartalvis failover-test: simuler primær-svigt, verificér at signing fortsætter på sekundær, swap-tilbage

**Network zone-position (foreløbig — afventer Mads' Zone A-F formalisering, Pouls Vinkel 6):**

- Tier 2 ICA-host i Zone C (CA-internal VLAN)
- Tier 2 HSM-bridge i Zone B (HSM-mgmt — Tailscale tail-net)
- ICA-host ↔ HSM-bridge via mTLS over Tailscale (eller dedicated VLAN — Engineering Lead vælger på baggrund af latency)

**Tier 2 fysisk separation fra Knowlagecentral-HSM (Pouls Vinkel 1):**

- Tier 2 HSM-par opbevares i separat fysisk lokation fra Knowlagecentral-HSM (Bjarne-domæne) for at undgå fælles physical compromise blast-radius
- Krav: separat safe eller separat sikret rum; ikke samme rack; helst geografisk adskilt fra Tier 1 safe også

### 1.4 Tier 3 — Issuing CA

**Konfiguration Fase 0:**

- 1× YubiHSM2 (single — Tier 3 HA er Fase 1+ scope)
- Issuing-private-key signerer: leaf-certs (via ACME), Issuing-CRL, OCSP-responses (delegated key — se §4.3)
- Network position: Zone C (internal); ACME-server i Zone D (RA-zone) kalder Tier 3 via mTLS
- HSM-throughput er begrænsende faktor — skal verificeres af Engineering Lead i load-test inden go-live (jf. Pouls Vinkel 9)

**Tier 3 HA-konsekvens af single-HSM-valg:**

- Fase 0 RTO ved Tier 3 HSM-svigt: foreløbig 4-24 timer (afventer Mads + FGD-bekræftelse af RTO-mål per Pouls Vinkel 4)
- Mitigation: backup-wrapped-key kan importeres til erstatnings-HSM på under 1 time hvis spare-HSM holdes klar
- Fase 1 anbefaling: HA-par også for Tier 3 — backlog-input til AO

---

## 2. YubiHSM2-integration (compliance-kritisk)

Dette afsnit beskriver hvordan YubiHSM2 integreres teknisk; drifts-eksekvering tilhører HSM-operatør og er ikke i AP-scope.

### 2.1 PKCS#11-session-authentication

YubiHSM2 eksponeres mod CA-software via **yubihsm-shell + yubihsm-pkcs11-modul** (standard Yubico-konnektor).

**Authentication-model:**

- HSM understøtter "Authentication Keys" — symmetriske keys (AES-128) der etablerer en sikker session
- Hver authentication-key har: Key ID (unik per HSM), Domain-tilhørsforhold, og Capabilities (fine-grained operation-rettigheder)
- En CA-host åbner en PKCS#11-session ved at præsentere authentication-key-ID + secret; HSM returnerer session-handle med den authentication-key's Capabilities
- Sessioner er midlertidige — auto-timeout (foreslået: 30 min uden aktivitet for online-tier; ved ceremony: 4-timers max-vindue)
- **Ingen authentication-key-secrets i kode, env-vars i klartekst eller logs** — secrets injiceres via OS keyring eller HSM-bridge-proxy (jf. mandat: secrets-håndtering)

### 2.2 Authentication-key-mapping per rolle

Baseret på Pouls Vinkel 7 (capability-mapping). Foreløbigt design — Mads + HSM-operatør (Bjarne) godkender endeligt mapping i CP/CPS §6.

| Authentication-key | Rolle | Capabilities (YubiHSM2-syntax) | Domain | Aktiverings-context |
|---|---|---|---|---|
| `auth-root-ceremony` | KCO + CA Admin + Security Officer (dual-control) | `generate-asymmetric-key`, `sign-pkcs`, `sign-ecdsa`, `export-wrapped`, `get-log-entries` | Root | Kun under Tier 1 ceremony; manuel aktivering via dual-control |
| `auth-hsm-operator` | HSM-operatør | `wrap-data`, `unwrap-data`, `delete-asymmetric-key`, `set-option`, `audit-set-option`, `change-authentication-key` | All | HSM-mgmt (Tier 1, 2, 3) — break-glass-procedure dokumenteret |
| `auth-ica-admin` | Tier 2 CA Admin | `sign-pkcs`, `sign-ecdsa`, `sign-attestation-certificate`, `get-log-entries` | Intermediate | Daglig brug for Tier 2 sub-CA-signing + CRL-signing |
| `auth-issuing-admin` | Tier 3 CA Admin | `sign-pkcs`, `sign-ecdsa`, `get-log-entries` | Issuing | Daglig brug for leaf-issuance |
| `auth-ocsp-responder` | OCSP-responder-tjeneste | `sign-ecdsa` (delegated-key only — se §4.3) | OCSP | Online; kort-levet delegated-key |
| `auth-auditor` | Trine (PKI Auditor) | `get-log-entries`, `get-object-info` (read-only) | All | Kvartalvis review + ad-hoc ved incident |

**Bemærk:** YubiHSM2's Domain-felt (16 bit, 16 mulige domains) er en logisk isolation; en authentication-key knyttet til Domain "Root" kan ikke tilgå nøgler i Domain "Issuing" selvom de er på samme HSM. Dette giver isolation indenfor samme fysiske enhed.

### 2.3 Wrap-key-management

**Wrap-key er den kritiske roden af backup-pyramiden.** Tab eller kompromittering af wrap-key er nær-katastrofalt.

**Wrap-key-design:**

- Algoritme: AES-256-CCM (FIPS-approved, YubiHSM2-native)
- Generation: under Tier 1 key ceremony, før Root-key genereres
- Lagring: aldrig i klartekst udenfor HSM — eksport sker som **Shamir Secret Sharing 3-of-5 split** (Pouls Vinkel 2 — A3)
  - 5 shares fordeles til 5 separate fysiske personer (KCO, CA Admin, HSM-operatør, Security Officer, ekstern juridisk depot)
  - Mindst 3 shares skal samles for at rekonstruere wrap-key — ingen enkelt person kan rekonstruere
  - Shares opbevares geografisk distribueret (FGD HQ safe + bank-safe-deposit + sekundær lokation)
- Rotation: hvert 5. år eller ved verificeret kompromittering (cf. Pouls Vinkel 2)
- Backup-restore: ved HSM-svigt, importeres wrapped Root-key til erstatnings-HSM under en ceremony hvor mindst 3 Shamir-shares samles og wrap-key rekonstrueres in memory på en air-gapped host

**Backup-export-strategi:**

- Hver tier (Root, Intermediate, Issuing) har wrapped-export af sine private keys produceret under generations-ceremoni
- Wrapped-export gemmes på minimum 2 separate offline media (write-once optisk eller hardware-encrypted USB) i separate fysiske safes (geografisk distribueret)
- Wrapped-export er ubrugelig uden wrap-key (hvilket kræver 3 Shamir-shares) → kompromittering af én lagret wrapped-export afslører ikke nøglen
- Audit-trail: hver wrapped-export-bevægelse logges (chain-of-custody-protokol; numbered tamper-evident bags)

### 2.4 Key-object-struktur (Key ID, Domain, Capabilities)

YubiHSM2-objects har faste felter pr. nøgle:

| Felt | Værdi (eksempel — Tier 2 ICA-key) | Kommentar |
|---|---|---|
| Key ID | `0x0010` | Unik per HSM; reserveret-ranges per tier (Root: 0x0001-0x000F; ICA: 0x0010-0x001F; Issuing: 0x0020-0x002F; OCSP delegated: 0x0030+) |
| Label | `fgdcorepki-ica-2026` | Human-readable; inkluderer year for rotation-tracking |
| Algorithm | `ecdsa-p384` (primær) eller `rsa-4096` | Per L5 |
| Domain | `Intermediate` (one-of 16) | Logisk isolation |
| Capabilities | `sign-pkcs`, `sign-ecdsa`, `export-wrapped`, `get-log-entries` | Specifik per nøgle-type — IKKE `delete-asymmetric-key` på CA-keys (kun HSM-operatør authentication-key kan slette) |
| Origin | `generated` (i HSM) eller `imported-wrapped` | Generated = produced i denne HSM; imported = HA-sync fra par-HSM |

### 2.5 HSM hardware-grænser (sizing-input til Engineering Lead)

Fra Pouls Vinkel 4 + 9, til verificering i load-test:

- **256 object slots total** per HSM, **128 KB total memory**
- ~7 RSA-4096-signaturer/sek; ~30+ ECDSA P-384-signaturer/sek (HSM-mæthning)
- Tier 3 single-HSM-budget: Issuing-CA-key + OCSP-delegated-key + log-signing-key + ACME-account-key-attestations → estimat ~10-20 slots, vel under 256-loft
- Audit-log-buffer på HSM er begrænset (skal eksporteres mindst hver time, jf. Pouls Vinkel 5 + §5.2 herunder)

### 2.6 FIPS 140-3-readiness (Fase 0 = non-FIPS)

Per L2: Fase 0 driftes med non-FIPS YubiHSM2-variant. Designet er FIPS-ready betydende:

- Algoritme-valg er allerede FIPS-godkendte (RSA-4096, ECDSA P-384, SHA-384/512, AES-256-CCM)
- Ingen FIPS-forbudte algoritmer (SHA-1, RSA-2048, MD5, DES) i scope
- Ceremony-procedure designed til at kunne re-keyes på FIPS-variant uden CP/CPS-ændringer
- Re-keying-event ved CMVP-grant: ny key ceremony på FIPS-variant; gamle non-FIPS-keys revokeres efter klient-overgang

**B1 (CMVP-status) er en åben Mads-overvågning** — AP designer ikke timeline for re-keying; det ejer Mads + FGD (jf. O6).

---

## 3. Protokol-valg og Fase 0-scope

### 3.1 Protokol-matrix

| Protokol | RFC | Fase 0 | Fase 1+ | Kommentar |
|---|---|---|---|---|
| **ACME** | RFC 8555 | **JA — primær** | Skaleret + flere challenge-typer | Standard automation-protokol; bredt klient-support (cert-manager, Certbot, lego, win-acme); HTTP-01 og DNS-01 challenges Fase 0 |
| **Manuel CSR** | RFC 2986 | JA (sekundær) | Vedligeholdes for engangs-tilfælde | Klassisk PEM-CSR via RA-portal; for cases hvor ACME ikke er muligt (legacy systemer) |
| **EST** | RFC 7030 | NEJ | JA — Fase 1 | Enterprise/IoT med mTLS-baseret enrollment; AP anbefaler scope-til-Fase-1 så Fase 0 fokuserer ACME-modenhed |
| **SCEP** | RFC 8894 | NEJ | Conditional — Fase 1+ | MDM-context (Intune, Jamf); kun hvis konkret integrations-behov; svagere sikkerhed end ACME/EST |
| **CMP** | RFC 4210 | NEJ | NEJ (medmindre konkret behov) | Sjælden i moderne integrations-mønstre |

**Begrundelse for Fase 0 = ACME-only:** Reduceret angrebsoverflade; én protokol-stack at hærde og review; alle FGDCorePKI's primære integrations-cases (interne services, server-certs, container workloads via cert-manager) understøtter ACME natively.

### 3.2 mTLS for enterprise-klienter

ACME-klienter authenticeres mod ACME-server via:

1. **External Account Binding (EAB, RFC 8555 §7.3.4)** — primær model. Klient registrerer ACME-account med EAB-key udstedt af RA. EAB-key er pre-shared og knyttes til klient-identitet i RA-portalen.
2. **mTLS klient-cert** for enterprise-internal automation — klient præsenterer FGDCorePKI-udstedt operator-cert (eller enterprise-CA-cert hvis bridge-tillid sat op) ved ACME-endpoint. Server-side mTLS-validation før ACME-protokol kan tales.
3. **Rate-limiting per EAB/mTLS-identity** — max-issuance-quota per applikation (foreløbigt 100/dag, justeres af AO + Engineering Lead per use-case).

ACME-endpoint placeres i Zone D (RA + ACME), bag API-gateway med rate-limit, request-validation og log-aggregation.

### 3.3 Fase 0 protocol-scope-eksklusioner

- **Ingen public-trust-flow** (Pouls O4) — Fase 0 er privat-PKI; ingen CT-logging, ingen WebTrust, ingen CABF-leaf-validity-trajektorie indarbejdet (designet kan håndtere reduktion til 47 dage senere uden refactoring)
- **Ingen CAA-record-check** ved issuance (kun relevant for public-trust)
- **Ingen OCSP-must-staple** håndhævelse i Fase 0 (anbefaling for Fase 1 — kræver klient-side support hvor relevant)

---

## 4. OCSP/CRL-arkitektur (compliance-kritisk)

### 4.1 EU-only-residency — hard requirement (L1)

**Per FGD-beslutning 2026-05-10 (DPIA-2 lukket Scenario B): ingen tredjelands-CDN, ingen tredjelands-failover, ingen hybrid med tredjeland.**

Dette betyder konkret:

| Komponent | Krav | Acceptable | IKKE acceptable |
|---|---|---|---|
| OCSP-responder origin | EU-region; EU-residency dokumenteret | On-prem EU; EU-cloud-region (fx Hetzner DE, Scaleway FR, OVH FR/PL/DE, Stackit DE, IONOS DE/EU); Vercel EU-region; Bunny.net EU-only-config | Cloudflare US, Fastly US, Akamai global, AWS us-east-*, Vercel global edge, enhver tjeneste hvor data kan routes uden for EU/EØS |
| CRL Distribution Point host | EU-region; EU-residency dokumenteret | Samme som ovenstående; HTTP (RFC 5280-compliant — CRL-fil er signeret) | Samme udelukkelser |
| CDN (hvis brugt) | EU-only-edge-network med kontraktlig EU-region-pinning | Bunny.net EU-zones; Hetzner Cloud Storage; KeyCDN EU-only-config (skal verificeres) | Cloudflare, Fastly, Akamai, AWS CloudFront global |
| DNS for OCSP/CRL endpoints | EU-residency for autoritative DNS-servere | EU-baserede DNS-providers; on-prem DNS | Tredjelands-providers (Cloudflare DNS, Route53 default) |

**Sub-processor-clausul:** Enhver leverandør i OCSP/CRL-stakken skal i DPA garantere EU-only-databehandling for FGDCorePKI's data; ingen CLOUD Act-eksponering. Dorthe ejer DPA-vurdering — AP leverer teknisk dataflow-input.

### 4.2 OCSP-stapling som primær model

Pouls Vinkel 3 + Vinkel 6 anbefaler OCSP-stapling som primær. AP bekræfter:

**OCSP-stapling-arkitektur:**

- TLS-server (cert-aftager) henter OCSP-response fra responder ved cert-renewal eller periodisk (hver time)
- TLS-server inkluderer signed OCSP-response i TLS handshake (Certificate Status Request extension, RFC 6066 §8 / RFC 6961)
- TLS-klient verificerer stapled response uden at skulle kontakte OCSP-responder direkte
- **Effekt:** OCSP-trafik mod responder reduceres dramatisk; IP-logning af cert-validerings-events nær elimineres; DDoS-overflade på OCSP-responder minimeres

**Begrænsning:** Stapling kræver klient-/server-support. Ikke alle TLS-klienter eller -servere implementerer det. For klienter uden stapling-support fungerer OCSP-responder som fallback (standard OCSP-request).

**OCSP-must-staple (RFC 7633)** — anbefaling for Fase 1 (jf. §3.3); ikke håndhævet i Fase 0 for at sikre kompatibilitet med varierende klient-stakke.

### 4.3 OCSP-responder-design

**Delegated-key-arkitektur (RFC 6960 §4.2.2.2.1):**

- OCSP-responder har en **dedikeret signing-key** (delegated fra Issuing CA)
- Delegated-key har kort validity (foreslået: 1 måned med auto-renewal)
- Delegated-key på YubiHSM2 (Domain: OCSP, Capabilities: `sign-ecdsa` only)
- **Performance-trade-off:** Dedikeret OCSP-signing på HSM koster ~73 ms ECDSA P-384 per response — ved 100 req/sek nær HSM-mæthning
- **Mitigation:** OCSP-stapling som primær (jf. §4.2) reducerer responder-trafikken; pre-signed OCSP-responses caches af responder så HSM ikke kaldes per request, kun når response er ved at udløbe (typisk gen-signering hver 30-60 min per cert som har aktiv validering)

**OCSP-response-validity-vindue:**

- `thisUpdate` til `nextUpdate`: 1-7 dage (foreslået: 24 timer Fase 0; tunes baseret på CRL-frekvens)
- Klient cacher response indtil `nextUpdate` — reducerer responder-load yderligere

**OCSP-responder-eksponering:**

- Zone E (DMZ — EU-only) bag DDoS-beskyttelse fra EU-leverandør
- Read-only access til Tier 3 CA's revocation-database (eller pre-signed-OCSP-cache)
- mTLS mellem responder og Tier 3 ICA-host

### 4.4 CRL-design

**CRL-frekvens og distribution:**

- **Tier 1 (Root CRL):** udstedes minimum hver 12 måneder (sjælden Tier 2-revocation); valid for 12-24 måneder med overlap
- **Tier 2 (Intermediate CRL):** udstedes ugentligt eller ved Issuing-CA-revocation; valid for 14 dage med overlap
- **Tier 3 (Issuing CRL):** udstedes minimum dagligt + ved enhver revocation-event; valid for 7 dage med overlap

**Delta-CRL fra dag 1** (Pouls Vinkel 3 — A3): for at håndtere mass-revocation-event uden CRL-blowup. Base CRL hver uge; delta-CRL daglig (eller hyppigere ved aktivitet).

**CRL Distribution Points (CDP):**

- HTTP-baseret URL-extension i hvert udstedt cert (RFC 5280 §4.2.1.13)
- CDP-host i EU (jf. L1)
- CDN-cache foran origin er tilladt — kun hvis CDN er EU-only-konfigureret
- CRL-fil er signeret → HTTP er acceptabel; klient validerer signatur

**CRL-størrelse-overvågning:**

- Alarm hvis CRL > 5 MB → trigger til at gå over til partitioned CRL (RFC 5280 §5)
- Pre-emptiv: partitioned CRL designed-in fra dag 1 for nemmere overgang

### 4.5 Dataflow og PII-vurdering

Per DPIA-2 §1: OCSP-trafik logger **klient-IP-adresse**, **cert-serial**, **timestamp**, **User-Agent**. Klient-IP er PII per art. 4(1) + Recital 30.

**Mitigation Fase 0:**

- OCSP-stapling reducerer direkte responder-trafik dramatisk (jf. §4.2)
- Responder-logs retention: **30-90 dage** (jf. mandat — `retention 30-90 dage for OCSP-request-logs`); Mads + Dorthe godkender endeligt
- Ingen IP-til-cert-korrelation i datawarehouse eller business-analytics — request-logs holdes operationelt isoleret (audit + DDoS-detection only)
- IP-trunkering på edge før logging (sidste oktet maskeres) hvis CDN-leverandøren understøtter det
- Ingen tredjelands-overførsel af responder-logs (EU-only — jf. L1)

---

## 5. Audit-log-arkitektur

### 5.1 Logs der samles og deres kilder

Per Pouls Vinkel 8 og AP-mandat (audit-trail-forpligtelse):

| Log-kilde | Indhold | Eksport-frekvens | Retention | PII-status |
|---|---|---|---|---|
| **YubiHSM2 internal log** | Authentication-key-brug, signing-events, capability-anvendelse, tamper-events | Hver time (max — buffer 80%-alarm) | 7 år | Authentication-key-ID korrelerer til operatør-ID (PII) |
| **CA-software audit-log** (Tier 2 + Tier 3) | Issuance, revocation, policy-decision, CSR-input | Real-time stream til SIEM | 7 år | Subject-DN i issuance-log = PII |
| **RA-portal log** | Approve, reject, requester-identitet, attestation | Real-time | 7 år | Requester-ID = PII |
| **OCSP-request-log** | Klient-IP, cert-serial, timestamp, User-Agent | Real-time | **30-90 dage** (kort) | Klient-IP = PII |
| **CRL-fetch-log** (origin) | Klient-IP, fetch-timestamp | Real-time | 30-90 dage | Klient-IP = PII |
| **OS audit (auditd / equivalent)** | Login-events, sudo, file-access på CA-hosts | Real-time | 1-3 år | Operatør-ID = PII |
| **Network firewall + NIDS** | Connection-events, alerts | Real-time | 1 år | IP-adresser |
| **Fysisk PACS** (door access) | Personer ind/ud af CA-rum | Real-time | 1-3 år | Person-ID = PII |
| **Ceremony-protokol (signed paper + screen-capture)** | Procedure-trin, witness-signatures, kommandoer | Per ceremony | **Afventer DPIA-1 / O2** | Witness-navne = PII |
| **Ceremony-recording (video/audio)** | Biometri (ansigt + stemme) | Per ceremony | **Afventer DPIA-1 / O2** | Art. 9 biometri |

### 5.2 WORM / tamper-evident design

**Tre-lags tamper-evidence (Pouls Vinkel 7 — A5; AP-mandat WORM-krav):**

1. **HSM-internal log** — eksporteres signeret af HSM (RFC 9162-inspireret signed log entries)
2. **Cryptographic chained log** — alle eksporterede HSM-events + CA-software-events + RA-events forwardes til en cryptographic-chained log (Merkle-tree-struktur, tilsvarende RFC 9162 Certificate Transparency log-arkitektur, men intern). Tampering med en historisk entry invaliderer hash-kæden fremad
3. **WORM-storage** — den cryptographic-chained log persisteres på faktisk WORM-medie:
   - **Primær option:** S3 Object Lock (compliance-mode) på EU-region (AWS eu-west-* eller alternativ EU-S3-compatible som Wasabi EU eller Scaleway Object Storage med immutability)
   - **Sekundær option:** WORM-tape eller dedikeret on-prem appliance (NetApp SnapLock eller equivalent)
   - **Krav:** Ingen rolle (heller ikke AP, Camilla, FGD selv) kan modificere eller slette inden retention udløber

**Krav for break-glass-logs (per AP-mandat):**

- Egen separat WORM-bucket / immutable-arkiv
- Hverken AP, Camilla eller AO kan modificere
- Trine auditerer halvårligt som SOC 2 CC7.2-evidens

### 5.3 Retention-matrice — sammenfattet

| Kategori | Retention | Begrundelse | Lås-status |
|---|---|---|---|
| Issuance + revocation audit | 7 år | AP-mandat; SOC 2 CC7.2; PKI baseline | LÅST |
| HSM internal log + chained events | 7 år | Som ovenfor | LÅST |
| OCSP-request-logs | 30-90 dage | Data-minimering (art. 5(1)(c)); kort vindue dækker DDoS-detection | LÅST (mandat) — endelig værdi inden for vindue: Mads + Dorthe |
| CRL-fetch-logs | 30-90 dage | Som ovenfor | Samme |
| OS / netværk / PACS | 1-3 år | Standard SIEM-baseline | Mads bekræfter |
| Ceremony-protokol (papir + screen-capture) | **Afventer DPIA-1 / O2** | F2 i DPIA-1: 7-10 år (audit-baseline) eller 20 år (ekstra sikkerhed) | ÅBEN |
| Ceremony-recording (video/audio) | **Afventer DPIA-1 / O2** | Biometri-proportionalitet (art. 5(1)(e)); evt. differentieret retention | ÅBEN |
| Break-glass-logs | 7 år (matcher issuance-retention) | SOC 2 CC7.2-evidens | LÅST |

### 5.4 Trine's read-access

- Auditor-authentication-key på HSM giver read-only `get-log-entries`
- SIEM-platform (afventer Mads-valg, Pouls Vinkel 8) giver Trine read-only-rolle til alle log-streams
- WORM-storage tillader read for Trine's account; ingen write/delete

---

## 6. Algoritme-begrundelse: ECDSA P-384 vs. RSA 4096

### 6.1 Primær valg: ECDSA P-384

**Begrundelser (intern PKI, Fase 0):**

| Faktor | ECDSA P-384 | RSA 4096 | Vinder |
|---|---|---|---|
| **Sikkerhedsniveau** | ~192-bit ækvivalent | ~152-bit ækvivalent | ECDSA |
| **Signing-performance på YubiHSM2** | ~30+ sign/sek | ~7 sign/sek | ECDSA (4-5× hurtigere) |
| **Signaturstørrelse** | 96 bytes (P-384) | 512 bytes | ECDSA |
| **Cert-størrelse-impact** | Markant mindre cert-chain | Større chain | ECDSA |
| **Network-overhead (TLS handshake)** | Lavere | Højere | ECDSA |
| **FIPS-godkendt** | JA (NIST SP 800-131A Rev 2) | JA | Begge |
| **Klient-kompatibilitet** | Bredt understøttet (alle moderne TLS-stacke) | Universelt | RSA marginalt bredere |
| **PQC-migration-impact** | Skal alligevel migreres til ML-DSA | Samme | Neutralt |
| **HSM-slot-effektivitet** (256-loft) | ECDSA-keys kompakte | RSA-keys større | ECDSA |

**Konklusion:** ECDSA P-384 er primær for nye CA-keys i FGDCorePKI Fase 0 — bedre performance, mindre overhead, samme FIPS-status.

### 6.2 RSA 4096 som sekundær

**Hvor RSA 4096 anvendes:**

- Hvis specifik klient-ekosystem kræver RSA (ældre embedded systemer, visse legacy-OS-stakke)
- Som fallback for cert-profiler hvor ECDSA-support hos klient er usikker
- Beslutning per cert-profil ved cert-policy-godkendelse — ikke en CA-niveau-beslutning

### 6.3 Hash-algoritmer

- SHA-384 minimum for cert-signaturer
- SHA-512 for længere validity-windows og audit-log-chains
- SHA-1 og MD5 forbudte (jf. L5)

### 6.4 PQC-readiness (åben — O3)

ML-DSA (FIPS 204) er finaliseret aug 2024; YubiHSM2 understøtter ikke ML-DSA i nuværende firmware. Crypto-agility-strategi:

- Cert-profil-design tillader hybrid-signaturer (klassisk + ML-DSA) når X.509-extensions stabiliseres (IETF LAMPS WG)
- Re-keying-event ved YubiHSM2 firmware-update med ML-DSA-support (eller HSM-platform-skifte)
- AP designer crypto-agility i dataflow; **strategisk PQC-roadmap ejer Mads + FGD** (jf. O3, Pouls B3)

---

## 7. Fase 0 MVP-afgrænsning

### 7.1 Scope IN — Fase 0

| Område | Komponent | Status |
|---|---|---|
| Tier 1 | Offline Root CA, 1× YubiHSM2 (non-FIPS) | Designet — kræver ceremony-script + DPIA-1-konklusion |
| Tier 2 | Intermediate CA, 2× YubiHSM2 HA-par | Designet — HA-pair adresserer Pouls B2 |
| Tier 3 | Issuing CA, 1× YubiHSM2 | Designet — Tier 3 HA i Fase 1 |
| Protokoller | ACME (RFC 8555) primær + manuel CSR fallback | Designet |
| OCSP | Stapling-primær; delegated-key responder; EU-only-hosting | Designet |
| CRL | Daglig issuance + delta-CRL; EU-only-distribution | Designet |
| Audit-log | WORM + cryptographic chained; 7-års retention på issuance-events | Designet |
| Algoritmer | ECDSA P-384 primær; RSA 4096 sekundær; SHA-384/512 | Designet |
| RA | Manual RA-portal; ACME EAB; mTLS for enterprise | Designet (detail-design owner: Engineering Lead) |
| Dual-control | CA Admin + HSM-operatør + Security Officer for Tier 1 | Designet |

### 7.2 Scope OUT — Fase 0 (planlagt Fase 1+ eller backlog)

| Område | Begrundelse for udskydelse |
|---|---|
| EST (RFC 7030) | Reduceret angrebsoverflade Fase 0; Fase 1+ ved konkret behov |
| SCEP (RFC 8894) | Conditional — kun ved MDM-integrations-behov |
| Tier 3 HA-par | Single-HSM Fase 0; kapacitets-data fra Fase 0 informerer Fase 1-beslutning |
| OCSP-must-staple-håndhævelse | Klient-stak-kompatibilitet skal verificeres bredere |
| CT-logging (RFC 9162) | Kun relevant for public-trust (O4) |
| CABF SC-081v3-leaf-validity-trajektorie (200/100/47-dage) | Kun public-trust-relevant; arkitektur understøtter ned-skalering uden refactoring |
| WebTrust-akkrediteret ekstern audit | Kun ved public-trust |
| FIPS 140-3-driftsmål | Re-keying-event når CMVP-grant lander (O6) — Mads ejer trigger |
| PQC-hybrid-signaturer (ML-DSA dual-cert) | Afventer YubiHSM2 firmware + LAMPS-stabilisering (O3) |

### 7.3 Fase 0 milestones (afhængigheder — ikke timeline)

AP er ikke roadmap-ejer (jf. L4 + AO-mandat). Følgende er **arkitektoniske afhængigheds-milestones** — AO + Engineering Lead ejer datoer:

1. **Pre-ceremoni-1 readiness:** CP/CPS-draft (Mads ejer); ceremony-script (AP udkast → KCO godkender → Mads godkender); DPIA-1-konklusion (Dorthe); datacenter + safe-grade (Mads + FGD); Network zone-model A-F (Mads + Engineering Lead); CA-software-valg (Poul Track A → Mads → FGD); HSM-throughput-test (Engineering Lead)
2. **Tier 1 Root key ceremony** — eksekveres af KCO + HSM-operatør + CA Admin + Security Officer; AP kan deltage som observer (jf. mandat: deltager ikke som autoriserende part)
3. **Tier 2 Intermediate ceremony** — Root signerer Intermediate; HA-import til HSM-B
4. **Tier 3 Issuing ceremony** — Intermediate signerer Issuing
5. **First leaf-issuance via ACME** — pilot-kunde / intern test-service
6. **Tier 3 driftsbekræftelse** (CA Administrator + Mads kvitterer); Trine første audit
7. **Fase 0 produktions-go-live** — FGD-godkendt og dokumenteret

---

## 8. Sammenhæng med øvrige dokumenter

| Dokument | Relation |
|---|---|
| `Team/Poul/2026-05-09-pki-arkitektur-10-vinkler.md` | Primær teknisk grundlag (Vinkler 1, 2, 3, 4, 5, 6, 7, 8, 9, 10) — dette dokument operationaliserer Pouls anbefalinger inden for AP-scope |
| `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ceremony-recording.md` | DPIA-1 (åben) — F1, F5, F9 ejer AP-input. Dette dokument leverer foreløbige tekniske præferencer som **input til DPIA-konklusion**, ikke som færdige beslutninger |
| `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ocsp-crl-cdn.md` | DPIA-2 (lukket Scenario B 2026-05-10) — EU-only-residency er hard requirement (L1) i dette dokument |
| `.claude/agents/fgdcorepki-application-provider.md` | AP-mandat — afgrænser hvad dette dokument må og ikke må sige |
| `.claude/agents/fgdcorepki-application-owner.md` | AO-mandat — definerer SOD og roadmap-ejerskab; AP ejer ikke timeline |

---

## 9. AP-input til DPIA-1 (F1, F5, F9 — foreløbig præference, ikke konklusion)

**F1 — Recording-form:** AP's foreløbige tekniske præference er **screen-capture + audio + hænder uden ansigt** for almindelige ceremonier; **fuld video** kun hvis non-repudiation-formålet eksplicit kræver entydig ansigtsidentifikation. Dorthe ejer endelig beslutning sammen med KCO. Reducer biometri-eksponering hvor proceduremæssigt forsvarligt.

**F5 — Arkiveringslokation:** AP's foreløbige tekniske præference er **on-prem WORM EU-residency** (samme krav som §5.2 break-glass-arkiv); sekundær option er EU-cloud immutable storage (Scaleway, Hetzner, Stackit object storage med EU-region-pinning og immutability/Object-Lock); **tredjelands-overførsel udelukket per L1-konsistens**. Endelig valg afventer Dorthes art. 32-vurdering + Mads' threat-model.

**F9 — Krypterings-arkitektur:**

- AES-256-GCM symmetrisk kryptering af recording-fil
- Symmetrisk file-encryption-key wrappet med ceremony-arkiv-key (separate fra wrap-key for CA-keys; egen Domain på YubiHSM2 eller separat HSM-instance)
- Ceremony-arkiv-key roteres hvert 5. år; gamle keys behoveholdes (revoked, ikke slettet) for at kunne dekryptere historiske recordings
- Key-rotation over 20-årshorisont kræver: tidlig nøgle-arkivering (ikke destrueret), alternativt re-encryption-cyklus hvert 5-10 år til ny kæde
- 3-of-5 Shamir-split for ceremony-arkiv-key så ingen enkelt rolle kan dekryptere uden dual-control

Dorthe ejer endelig vurdering — dette er **AP's tekniske input**, ikke afgørelse.

---

## 10. Åbne spørgsmål — sammenfattet

| # | Spørgsmål | Ejer | Reference |
|---|---|---|---|
| O1 | Ceremony-recording arkiv-lokation + krypterings-arkitektur (DPIA-1 F5+F9) | Dorthe (DPO) ejer konklusion; AP leverer input (§9) | DPIA-1 |
| O2 | Ceremony-protokol/recording retention (7-10 år vs. 20 år) — DPIA-1 F2 | Mads + AO + FGD (Dorthe input) | DPIA-1 §4 |
| O3 | Root validity 10-12 år vs. 15-20 år (PQC-headroom) | Mads + FGD | Pouls Vinkel 3 + 5; B3 |
| O4 | Public-trust eller privat-only PKI | FGD | Pouls Vinkel 3 + 8 |
| O5 | CA-software-valg (EJBCA Enterprise vs. Step-CA combo) | Poul Track A → Mads → FGD | Pouls Vinkel 9 |
| O6 | YubiHSM2 FIPS 140-3 CMVP-grant trigger til re-keying | Mads (overvågning); FGD-go | Pouls Vinkel 5; B1 |
| O7 | Datacenter-tier + safe-grade Tier 1 | Mads + FGD | Pouls Vinkel 1 |
| O8 | RTO/RPO-mål formaliseres (Tier 3 HSM-svigt foreløbig 4-24t) | Mads + FGD | Pouls Vinkel 4 |
| O9 | Network zone-model A-F formaliseres + firewall-regler | Mads + Engineering Lead | Pouls Vinkel 6 |
| O10 | YubiHSM2 capability-mapping endelig CP/CPS-tekst | Mads + HSM-operatør (Bjarne) | Pouls Vinkel 7; §2.2 herinde |
| O11 | SIEM-platform-valg | Mads + Engineering Lead | Pouls Vinkel 8 |
| O12 | OCSP-request-log retention endelig værdi (30 vs. 90 dage) | Mads + Dorthe | §5.3 + AP-mandat |
| O13 | EU-CDN-leverandør-shortlist + DPA-tilgængelighed | Dorthe (DPA) + AP (teknisk-fit) | DPIA-2 §4 F2/F7 |

---

## 11. AP's leverancer som opfølgning på dette dokument

1. Tekniske dataflow-input til DPIA-1 (formaliseret i Dorthe-inbox-format) — inden Dorthes F1/F5/F9-frist
2. ROPA-input til Dorthe når FGDCorePKI-ROPA-sektionen oprettes (Camilla ejer fil-oprettelse): aktiviteter for cert-issuance, OCSP/CRL, audit-log, ceremony-recording
3. Ceremony-script (teknisk udkast) til KCO-review og Mads-godkendelse — afventer DPIA-1 + datacenter-/safe-beslutning
4. Capability-mapping CP/CPS §6-input til Mads (med HSM-operatør-review)
5. Hand-off til Poul (Track A): teknisk dybde-input til CA-software-evaluering — EJBCA YubiHSM2-fit, PKCS#11-kvalitet, ACME-server-modenhed
6. Hand-off til Engineering Team Lead: implementerings-spec for ACME-server, RA-portal, OCSP-responder, CRL-generation-pipeline (når Lead-rolle bemandet)

---

**Forfatter:** Application Provider (FGDCorePKI)
**Dato:** 2026-05-10
**Status:** Udkast — distribueret til Application Owner, Mads (CISO), Dorthe (DPO), Poul (analyst)
**Næste milestone:** AP-input til DPIA-1 (F1, F5, F9) leveret til Dorthe-inbox; afventer Mads' security-review-feedback på dette dokument
