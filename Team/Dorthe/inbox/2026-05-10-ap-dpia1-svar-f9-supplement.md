---
dato: 2026-05-10
fra: Application Provider (FGDCorePKI)
projekt: FGDCorePKI
type: teknisk-supplement
til: Dorthe (DPO)
emne: F9-supplement — finaliseret krypteringsdesign efter F2-beslutning (7 år)
reference-hoveddokument: Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md
reference-arkitektur: Team/projects/fgdcorepki/ap/2026-05-10-pki-teknisk-arkitektur-fase0.md
udløsende-beslutning: FGD-beslutning 2026-05-10 — F2 retention = 7 år for ceremony-recordings
status: supplement til DPIA-1 svar (afløser de "betinget af F2"-dele af §4)
---

# F9-supplement — Krypteringsdesign finaliseret efter F2 = 7 år

## 0. Formål og scope

DPIA-1 svaret af 2026-05-10 (`Team/Dorthe/inbox/2026-05-10-ap-dpia1-svar.md`) markerede F9.3 og dele af F9.4 som **[Afventer F2]**. FGD har 2026-05-10 truffet beslutning: **ceremony-recording retention = 7 år** (ikke 20 år).

Dette supplement finaliserer:

1. **F9.3** — krypteringsdesign og rotation-strategi (variant a vs. b) under 7-års retention
2. **F9.4** — cipher-agility-krav under 7-års retention

Andre dele af DPIA-1 svaret (F1, F5, og F9.1/F9.2/F9.5/F9.6) er ikke berørt af F2-beslutningen og forbliver gældende som angivet i hoveddokumentet.

Labeling-konvention fra hoveddokumentet anvendes uændret ([AP-teknisk], [Kræver Mads], [Kræver FGD], [AP-anbefaling]). Mærket **[Afventer F2]** udgår; alle tidligere F2-betingede konklusioner er nu låst.

---

## 1. Sammenfatning af supplement

| Område | Konklusion under 7-års retention |
|---|---|
| **F9.3 rotation-strategi** | **Variant (a) — Bevarings-strategi** anbefales primært. Re-encryption-ceremoni (variant b) er ikke proportional ved 7 år. |
| **F9.3 CAK-rotation-cadence** | **7-års cadence** (én CAK pr. retention-vindue) anbefales primært. Sekundær option: 5-års cadence hvis Mads ønsker overlap-margen. Tidligere 5-års default fra hoveddokumentet kan reduceres. |
| **F9.4 cipher-agility-krav** | **Simplere krav** — én cipher-generation forventes at dække hele retention-perioden. Re-encryption-readiness bibeholdes som rest-mitigation, men er ikke planlagt eksekvering. |
| **Live CAK-generationer over tid** | Maksimum **2 CAK-generationer** "live" på et givet tidspunkt (aktiv + senest udfaset, sidstnævnte med kun `unwrap-data`). Mod 4-5 generationer i 20-års-scenariet. |
| **Shamir-share-vedligehold** | Reduceret — 1-2 generationer × 5 fysiske shares = 5-10 shares aktive samtidig (mod 20-25 ved 20 år). |

**[AP-anbefaling primær]**: Variant (a) bevarings-strategi med 7-års CAK-rotation-cadence, AES-256-GCM som lock'et cipher gennem retention-perioden, ingen planlagte re-encryption-ceremonier.

**[Kræver Mads]** Endelig krypto-strategi-godkendelse + valg mellem 5-årig vs. 7-årig CAK-rotation-cadence.

---

## 2. F9.3 finaliseret — Variant (a) anbefales

### 2.1 Hvorfor variant (a) under 7 år (mod variant (b) ved 20 år)

DPIA-1 svaret §4 F9.3 angav betingelses-tabellen:

| F2 lander på | AP-anbefaling | Begrundelse |
|---|---|---|
| 7-10 år | **Variant (a) — Bevarings-strategi** | Kun 1-2 CAK-generationer live; bevarings-overhead er lav; re-encryption-risiko ikke berettiget |
| 20 år | Variant (b) — Re-encryption-strategi | 4-5 generationer ikke håndterbart; konfidentialitet over 20 år kræver cipher-agility |

F2 er låst på **7 år**. Konklusionen er entydig: **variant (a) anbefales**.

Uddybende argumenter:

1. **Antal live CAK-generationer**: Med 7-års retention og 7-årig CAK-rotation-cadence er der typisk 1 aktiv generation. Hvis 5-årig cadence vælges, er der maksimum 2 generationer live. Bevarings-overhead er trivielt.
2. **Konfidentialitets-horisont**: AES-256-GCM med 7-års konfidentialitets-horisont er **godt indenfor** sikker margen. NIST + ENISA + BSI vurderer pr. 2025-2026 AES-256-GCM som forsvarbar mod kendte angreb i 15-20+ år forude. PQC-trussel mod symmetric-keys (Grover) reducerer effektiv sikkerhed til ~128-bit, hvilket fortsat er tilstrækkeligt.
3. **Re-encryption-risiko vs. gevinst**: Re-encryption-ceremoni introducerer chain-of-custody-svækkelse (plaintext berører re-encryption-host) og operationel risiko (fejl mid-event kan koste adgang). Ved 7 år er gevinsten ved planlagt re-encryption marginal og ikke proportional til risikoen.
4. **GDPR art. 5(1)(c) data-minimering**: Færre live keys + færre Shamir-share-fysiske-arkiver = mindre angrebsoverflade for biometri-data. Variant (a) er bedre data-minimering ved kort retention.

### 2.2 Konsekvenser for CAK-rotation-cadence

DPIA-1 svaret hovedtekst nævnte "5-årig CAK-rotation" som default (jf. §4 F9.2-diagram og §6 F9-anbefaling). Dette default kan revideres under 7-års retention.

| Cadence-option | Live-generationer max | Anbefaling under 7-års retention |
|---|---|---|
| **7 år** (matcher retention) | **1 generation** | **[AP-anbefaling primær]** Maksimal enkelhed; én CAK dækker hele retention-vinduet for recordings krypteret i den periode. Ved retention-udløb destrueres CAK og alle dens recordings samtidig — ren sletnings-procedure. |
| **5 år** (overlap-margen) | **2 generationer** | **[AP-anbefaling sekundær]** Hvis Mads ønsker rotation-cadence kortere end retention som forsvar mod ukendt cipher-svækkelse i CAK-levetid; 2 generationer er stadig håndterbart. |
| **3 år** (aggressiv) | 3 generationer | **[AP-frarådes]** Marginal sikkerhedsgevinst; uforholdsmæssig ceremoni-overhead under 7-års retention. |

**[AP-anbefaling]**: **7-årig cadence** primært. Hvis Mads ønsker overlap-margen mod ukendt risiko i CAK-livscyklus (fx mistanke om HSM-firmware-svækkelse), eskalér til **5-årig cadence** — fortsat inden for variant (a)-rammen, ingen re-encryption nødvendig.

**Sletnings-procedure under 7-årig cadence**:

1. CAK-gen-1 oprettes 2026 — krypterer recordings 2026-2033
2. 2033 udløber retention for recordings 2026: filer slettes (jf. F5.5-procedure); CAK-gen-1 destrueres samtidig (sidste reference fjernet)
3. Ny CAK-gen-2 (oprettet 2033) krypterer recordings 2033-2040
4. Ingen "rotated men live" generation overlapper i normal-drift — én generation ad gangen

Dette er en betydelig forenkling vs. 5-års cadence + 20 års retention (4-5 overlappende generationer).

### 2.3 Konsekvenser for Shamir-share-fysisk-vedligehold

Under 7-årig CAK-rotation: **1 generation × 5 shares = 5 fysiske shares aktive** ad gangen.

Ved skift til ny CAK-generation (hvert 7. år):

1. Ny CAK-gen-2 oprettes på HSM under ceremony
2. Nye 5 shares splittes og distribueres til de 5 depot-lokationer (KCO HQ-safe, CA Admin HQ-safe-2, HSM-operator off-site safe, Security Officer bankboks (jf. parallel beslutningsnote 2026-05-10), 5. depot per FGD-låsning)
3. Gamle 5 shares destrueres efter formel verifikation af at ny CAK fungerer (test-encrypt + test-decrypt-roundtrip via 3-of-5 recovery-rehearsal)
4. Gamle CAK-gen-1 destrueres når retention-udløb tømmer dens recordings (jf. 2.2 step 2)

**[AP-anbefaling]**: Shamir-share-destruktion ved generation-skift udføres som dual-control ceremony, audit-logget. Dette er en ny ceremony-type (kortere end key-generation) der bør CP/CPS-dokumenteres.

### 2.4 Hvad ændrer sig vs. hoveddokumentet

| Hoveddokument-reference | Tidligere formulering | Finaliseret formulering |
|---|---|---|
| §4 F9.2 ASCII-diagram, "(gen 1: 2026-2031)" osv. | 5-års rotation, op til 4-5 live generationer | **7-års rotation, 1 live generation ad gangen (primær)** eller 5-års/2 live (sekundær) |
| §4 F9.3 betingelses-tabel | Variant (a) hvis 7-10 år; variant (b) hvis 20 år | **Variant (a) finaliseret** — F2=7 år låst |
| §6 F9-anbefaling | "5-årig CAK-rotation; bevarings-strategi (a) hvis F2 lander 7-10 år, re-encryption-strategi (b) hvis F2 lander 20 år" | **"7-årig CAK-rotation (primær) eller 5-årig (Mads-option); bevarings-strategi (a) finaliseret; ingen planlagt re-encryption"** |
| §5 åbne afhængigheder, AD1 (F2-konklusion) | Åbent | **LUKKET 2026-05-10 — F2 = 7 år** |
| §5 åbne afhængigheder, AD4 (Mads' godkendelse af 5-årig cadence) | Åbent | **Justeret — Mads' valg er nu mellem 7-årig (primær) og 5-årig (sekundær) cadence** |

---

## 3. F9.4 finaliseret — Cipher-agility-krav forenklet

### 3.1 Cipher-agility-behov under 7-års retention

DPIA-1 svaret §4 F9.4 angav at variant (b) gav indbygget cipher-agility som biprodukt af planlagte re-encryption-ceremonier. Variant (a) accepterede at algorithm-skifte ville kræve en ekstra-ceremoni.

Under 7-års retention er konteksten markant ændret:

| Aspekt | Under 20 år (oprindelig analyse) | **Under 7 år (finaliseret)** |
|---|---|---|
| Sandsynlighed for at AES-256-GCM kompromitteres inden retention-udløb | Reel — 20-års horisont er længe | **Lav** — AES-256-GCM forventes forsvarbar mod kendte angreb i 15-20+ år |
| Sandsynlighed for at PQC-trussel kræver symmetric-cipher-skifte inden retention-udløb | Medium-høj — Grover-effektiv-128-bit forbliver tilstrækkeligt, men cryptanalytic-progress kan ændre vurdering | **Lav** — symmetric-PQC-trussel (Grover) er allerede modnet i risk-modellen; AES-256 reducerer til 128-bit effektivt og er stadig tilstrækkeligt |
| Cipher-agility som design-krav | Stærkt — re-encryption-readiness obligatorisk | **Svagere** — re-encryption-readiness er rest-mitigation, ikke planlagt eksekvering |
| Algorithm-skifte-omkostning | Variant (b) = marginal merkost; variant (a) = høj | **Hypothetisk** — kun aktuelt hvis uventet cipher-svækkelse opdages |

**[AP-anbefaling]**: 
- **AES-256-GCM lock'es som primær cipher** for hele 7-års retention-perioden under variant (a)
- **Cipher-agility bevares som arkitektur-egenskab** (HSM understøtter ChaCha20-Poly1305 og forventet PQC-symmetric-equivalents) — men der laves ikke planlagt re-encryption-ceremoni
- **Trigger for ad-hoc re-encryption**: Hvis cryptanalytic-fund eller NIST/ENISA/BSI-advisory under retention-perioden anbefaler skifte, eskaleres det som **change-request** under governance-processen (Mads + Dorthe + FGD)

### 3.2 HSM-cipher-agility-readiness (uændret)

YubiHSM2 understøtter:

- AES (CCM, CMAC, GCM) — primær valgt
- ECDH/ECDSA P-256, P-384, P-521 — for asymmetric (ikke relevant for CAK selv, men for HSM-niveau wrapping)
- HMAC-SHA-256/384/512

PQC-symmetric-equivalents kommer via firmware-update når NIST PQC-standardisering modnes for symmetric-side. **[AP-teknisk]** AP og Bjarne følger Yubico-firmware-roadmap; PQC-readiness for asymmetric-keys er prioriteret andetsteds (jf. arkitektur-dokumentet) og berører ikke ceremony-arkiv-CAK direkte.

### 3.3 Hvad ændrer sig vs. hoveddokumentet

| Hoveddokument-reference | Tidligere formulering | Finaliseret formulering |
|---|---|---|
| §4 F9.4 omkostnings-estimat-tabel | Variant (a) + algorithm-skifte = "Høj" omkostning | **Uændret som hypothetisk omkostning, men sandsynlighed for udløsning under 7-års retention er lav** |
| §4 F9.4 konklusion | "Variant (b) giver bedre cipher-agility-readiness" | **Cipher-agility-readiness er rest-mitigation, ikke planlagt eksekvering — variant (a) er tilstrækkelig under 7-års retention** |

---

## 4. Konsoliderede AP-anbefalinger (lukket version af DPIA-1 §6 F9)

Hoveddokumentet §6 F9-anbefaling, **finaliseret formulering**:

> **F9** (finaliseret 2026-05-10): AP anbefaler per-fil AES-256-GCM DEK wrappet med ceremony-arkiv-key (CAK) på separat YubiHSM2 Domain "CeremonyArchive" med 3-of-5 Shamir Secret Sharing for CAK-recovery; **7-årig CAK-rotation-cadence (primær) eller 5-årig (Mads-option for overlap-margen) — én aktiv CAK-generation ad gangen under variant (a) bevarings-strategi**; **ingen planlagte re-encryption-ceremonier**; cipher-agility bevares som arkitektur-egenskab og rest-mitigation; auditor-viewing-session med dual-witness + RAM-only decryption + break-glass-log-integration sikrer at hverken Trine eller HSM-operatør kan solo-decrypt.

---

## 5. Opdaterede åbne afhængigheder

| # (fra hoveddokument) | Status efter F2-beslutning |
|---|---|
| AD1 — F2 retention-konklusion | **LUKKET 2026-05-10 — 7 år** |
| AD4 — Mads' godkendelse af CAK-rotation-cadence | **Justeret** — Mads' valg er nu mellem 7-årig primær (AP-anbefaling) og 5-årig sekundær |
| AD5 — Bjarnes input på Domain vs. separat HSM | Uændret åbent |
| AD9 — Context7-verificering af Object Lock-feature-status (Hetzner) | Uændret åbent (vedrører F5.2, ikke F9) |
| AD10 — Mads vurdering Enterprise vs. Compliance-mode | Uændret åbent |

**Nye afhængigheder fra dette supplement**:

| # | Input | Ejer | Påvirker | Frist |
|---|---|---|---|---|
| AD11 | Mads-valg mellem 7-årig (primær) og 5-årig (sekundær) CAK-rotation-cadence | Mads | F9.2 key-hierarki ASCII-diagram-finalisering; CP/CPS-procedure-dokumentation | Pre-ceremony |
| AD12 | Trigger-definition for ad-hoc re-encryption (cryptanalytic-advisory governance) | Mads + Dorthe | F9.4 cipher-agility rest-mitigation | Inden første ceremony, eller senest ved første halvårlige Trine-audit |

---

## 6. Spørgsmål til Dorthe (supplement)

Hoveddokumentets §7 spørgsmål er uændret gældende. Dette supplement tilføjer:

1. **Variant (a)-bekræftelse**: Er Dorthe enig i at variant (a) bevarings-strategi under 7-års retention er proportional GDPR-mæssigt (data-minimering art. 5(1)(c) og integritets-/fortrolighedsprincippet art. 5(1)(f))?
2. **Cadence-præference**: Har Dorthe en GDPR-præference mellem 7-årig og 5-årig CAK-rotation-cadence — fx ift. "data-minimering ved levetid for keys der beskytter art. 9-data"?
3. **Ad-hoc re-encryption-trigger**: Hvis Mads + Dorthe + FGD senere vurderer at en cryptanalytic-advisory kræver ad-hoc re-encryption inden retention-udløb, ønsker Dorthe at være co-signatar på trigger-beslutningen (parallelt til Mads), eller er CISO-eskalering tilstrækkelig med DPO-notifikation?

---

**Forfatter**: Application Provider (FGDCorePKI)
**Dato**: 2026-05-10
**Status**: Supplement til DPIA-1 svar — afløser §4 F9.3 og §4 F9.4 "[Afventer F2]"-dele; øvrige dele af hoveddokumentet uændret gældende
**Distribution**: Dorthe (DPO) primær; kopi til Application Owner og Mads (CISO) for awareness; AP-arkiv `Team/projects/fgdcorepki/ap/`
**Næste milestone**: DPIA-1-konklusion kan nu finaliseres på F9-siden; afventer Mads' valg af cadence (AD11) inden CP/CPS-procedure-dokumentation
