---
dato: 2026-05-09
type: dpia-screening
projekt: FGDCorePKI
behandlings-aktivitet: Key ceremony video/lydoptagelse (Root + Intermediate CA)
initieret-af: Dorthe (DPO)
status: SCREENING-IGANGSAT
kræver-fuld-dpia: JA
basis:
  - Team/Dorthe/2026-05-09-dorthe-final-check-fgdcorepki-mandater.md (GR-2, ROPA #5)
  - Team/Dorthe/2026-05-09-dorthe-4eyes-fgdcorepki-mandater.md (Fund GR-2)
  - Team/Laila/2026-05-09-udkast-pki-governance-roller.md v2 §3 (Key Ceremony Officer)
referencer-gdpr:
  - art. 9 (særlige kategorier — biometri til identifikation)
  - art. 35 (DPIA — high-risk processing)
  - art. 35(3)(b) (systematisk og omfattende behandling)
  - art. 35(3)(c) (systematisk overvågning, evt. relevant)
  - art. 5(1)(c) (data-minimering)
  - art. 5(1)(e) (opbevaringsbegrænsning)
  - art. 6 (lawful basis)
  - art. 13/14 (information til registrerede)
edpb-wp29-kriterier-undersøgt:
  - Sensitive data (biometri/art. 9)
  - Long-term retention (20 år)
  - Identifikations-formål
modtagere: FGD, Mads (CISO), Stefan (orkestrator), KCO (når rolle bemandes)
---

# DPIA-screening: Key ceremony video/lydoptagelse

## 1. Behandlings-beskrivelse

**Hvad behandles:**
- Videooptagelse (billede + lyd) af nøgle-ceremoni-deltagere under Root CA og Intermediate CA key generation, key activation, key recovery og key destruction
- Optagelsen omfatter:
  - Witnesses' og Key Ceremony Officers' (KCO) ansigt og stemme
  - Smartcard-/HSM-operatørens hænder under fysisk håndtering
  - Skærmoptagelse af HSM-CLI/UI-kommandoer
  - Verbal recitation af protokolskridt og signering
- Materialet kan kobles 1:1 med navngivne deltagere (KCO, witnesses, HSM-operatør)
- Opbevaringsperiode per nuværende mandat-tekst: **minimum 20 år** med WORM-garanti, krypteret arkivering

**Af hvem:**
- Dataansvarlig: FGD (CVR — eget juridiske ansvar)
- Behandler: FGDCorePKI-teamet (KCO som ceremony-leder; AP som teknisk ansvarlig for arkiv-flow)
- Adgang til arkivmateriale (least-privilege): Trine (auditor), Mads (CISO), Dorthe (DPO), juridisk vidne ved retsligt krav

**Til hvilket formål:**
1. **Non-repudiation** — beviskæde for at Root/Intermediate-nøgle er genereret korrekt og uden uautoriseret materiale-eksponering
2. **Audit-trail** — evidens til intern audit (Trine), ekstern auditor (WebTrust/ETSI hvis relevant), tilsynsmyndighed
3. **Forensisk reference** ved senere mistanke om CA-kompromittering (kan sammenholdes med ceremony-protokol)

**Behandlings-kontekst:**
- Ceremoni er sjælden (Root: typisk 1× per ~10 år; Intermediate: 1-3× per år ved rotation)
- Antal data-subjects per optagelse: 4-8 (KCO + 2-4 witnesses + HSM-operatør + evt. juridisk vidne)
- Optagelsen er statisk (ikke løbende monitorering)
- Lokation: fysisk sikret rum (Bjarne-domæne); ikke offentlig

---

## 2. Vurdering af DPIA-trigger (GDPR art. 35)

DPIA er obligatorisk når behandlingen er "sandsynligvis høj risiko" jf. art. 35(1). EDPB/WP29-vejledning (WP248 rev.01) opstiller 9 kriterier; opfyldelse af 2+ kriterier udløser fuld DPIA.

| # | WP29-kriterium | Opfyldt? | Begrundelse |
|---|---|---|---|
| 1 | Evaluering eller scoring | Nej | Ingen profilering eller scoring |
| 2 | Automatiseret beslutningstagning med retsvirkning | Nej | Ceremoni er manuel; optagelse er passiv evidens |
| 3 | Systematisk overvågning | **Delvist** | Optagelsen er enkeltstående pr. ceremoni, men 20-årsbevaring + multi-event giver overvågnings-karakter over tid |
| 4 | Sensitive data eller data af meget personlig karakter | **JA** | Biometri (ansigt/stemme) — art. 9 hvis brugt til identifikation; identifikation er netop formålet (non-repudiation kræver entydig kobling til deltager) |
| 5 | Behandling i stor skala | Nej | Få deltagere per ceremoni; lav frekvens |
| 6 | Matching eller kombination af datasæt | **Delvist** | Optagelse korreleres med ceremony-protokol (signed paper), HSM-audit-log, smartcard-ID — kombineret giver komplet identifikations-evidens |
| 7 | Data om sårbare personer | Nej | Deltagere er medarbejdere i tjenstlig kapacitet |
| 8 | Innovativ teknologi | Nej | Standard videooptagelse |
| 9 | Behandling der forhindrer udøvelse af rettighed eller adgang til service | Nej | Påvirker ikke registreredes adgang til ydelse |

**Antal opfyldte kriterier:** 1 fuld (#4 biometri) + 2 delvise (#3 langtidsovervågning, #6 datakombination)

**Konklusion:** **DPIA-trigger opfyldt.** WP29-tærskel (2+ kriterier) er opfyldt selv ved konservativ vurdering hvor delvise kun tæller som halve. Art. 9-kategorien alene er normalt tilstrækkelig til at kræve DPIA i dansk tilsynspraksis (Datatilsynet har vejledt at biometri-baseret identifikation er DPIA-pligtig).

**Yderligere DPIA-pligt-grundlag:**
- Datatilsynets liste over behandlinger der altid kræver DPIA (jf. art. 35(4)) inkluderer behandling af biometriske data til entydig identifikation
- 20-årsbevaring overstiger væsentligt PKI-baseline (7 år) og kræver selvstændig proportionalitets-vurdering jf. art. 5(1)(e)

---

## 3. Foreløbig konklusion

**Kræver fuld DPIA: JA.**

Begrundelser i rangorden:
1. **Art. 9 biometri til identifikation** — udløser i sig selv DPIA-pligt
2. **20-årsbevaring** ikke automatisk proportional med audit-formålet og kræver formålsbestemt vurdering
3. **Datakombination** (video + protokol + HSM-log + smartcard-ID) muliggør detaljeret rekonstruktion af deltagers handlinger over år

DPIA skal være afsluttet og godkendt **inden første Root CA key generation-ceremoni**. Hvis ceremoni planlægges før DPIA er færdig, skal ceremoni udskydes — alternativt afholdes som "tør-kørsel" uden optagelse (men dette opfylder ikke non-repudiation-formålet).

---

## 4. Afventende information

Inden fuld DPIA kan gennemføres skal følgende afklares:

| # | Spørgsmål | Ansvar | Frist |
|---|---|---|---|
| F1 | Skal optagelsen omfatte fuld video (ansigt synligt) eller alternativ form (audio + screen + hænder uden ansigt; ansigtsmasking; kun screen-capture)? | KCO + AP + Dorthe | Inden ceremony-script låses |
| F2 | Er 20-årsbevaring det reelle krav, eller drevet af WebTrust/ETSI-auditkrav (typisk 7-10 år)? Hvis WebTrust ikke er i scope, kan retention reduceres | Mads (CISO) + AO | Før DPIA-mitigation-fase |
| F3 | Hvilken lawful basis for hver deltagerkategori (KCO/witnesses/HSM-operatør/eksternt juridisk vidne)? Samtykke (art. 6(1)(a)) er sandsynligt for eksterne; retlig forpligtelse (art. 6(1)(c)) eller legitim interesse (art. 6(1)(f)) for interne medarbejdere | Dorthe + AO + HR | Før DPIA-konsultation |
| F4 | Hvilken art. 9-undtagelse anvendes? Sandsynligt art. 9(2)(a) (eksplicit samtykke) eller art. 9(2)(g) (væsentlig samfundsinteresse — svag) | Dorthe (vurdering) | Før DPIA-konklusion |
| F5 | Arkiveringslokation: on-prem WORM (EU-residency) eller cloud-immutable (kræver tredjelands-vurdering hvis ikke EU)? | AP + Mads | Før DPIA-mitigation-fase |
| F6 | Adgangskontrol til arkivet: hvor stramt least-privilege, og hvilken procedure for retskrav-aktiveret access? | Mads + Trine | Før DPIA-konklusion |
| F7 | Skal deltagere have ret til indsigt (art. 15) i optagelse de medvirker i? Hvordan håndteres art. 17-anmodning (sletning)? PKI-non-repudiation-formålet trækker mod fortsat opbevaring (art. 17(3)(b) — retlig forpligtelse) | Dorthe | Inden DPIA-konklusion |
| F8 | Information til deltagere (art. 13/14): tidspunkt, format, samtykke-formular eller ikke-samtykke-baseret information? | Dorthe + KCO | Før første ceremoni |
| F9 | Krypterings-arkitektur for arkivet: hvilke nøgler, hvor opbevares de, hvordan håndteres key-rotation over 20 år? | AP + Mads + HSM-operatør | Før DPIA-konklusion |

---

## 5. Næste skridt

| # | Handling | Ansvar | Tidsramme |
|---|---|---|---|
| 1 | Stefan beder FGD bekræfte DPIA-prioritet (kræves inden Fase 0 produktionstag — bør køres umiddelbart efter mandat-aktivering) | Stefan | Inden 5 hverdage |
| 2 | Indhente svar på F1-F9 fra interessenter (sekventielt eller parallelt hvor muligt) | Dorthe koordinerer | 2-3 uger |
| 3 | Konsultation med Mads (CISO) — security-mitigation-options for biometri-eksponering | Dorthe + Mads | Parallelt med F1-F9 |
| 4 | Udkast til fuld DPIA-rapport (formålsbestemmelse, nødvendighed/proportionalitet, risikovurdering for registrerede, mitigation-tiltag, lawful basis, rest-risici) | Dorthe | Efter F1-F9 lukket |
| 5 | DPIA-review af KCO + Mads + AP (4-eyes) | Mads + KCO + AP | 1 uge efter DPIA-udkast |
| 6 | DPIA-konklusion til FGD med anbefaling: GO / GO-MED-MITIGATION / Datatilsyn-konsultation (art. 36) hvis høj rest-risiko | Dorthe → FGD | Efter review |
| 7 | Hvis art. 36 udløses: konsultation af Datatilsynet inden første ceremoni | Dorthe (DPO-kontaktpunkt) | 8-uge tilsyns-svartid + buffer |
| 8 | Update af ROPA #5 (key-ceremony-optagelse) ved aktivering af `governance/dpa/ropa.md` | Dorthe → Camilla | Når ROPA-fil oprettes |
| 9 | Information-template (art. 13/14) til deltagere: dansk + engelsk; underskrives før ceremoni | Dorthe | Inden første ceremoni |

---

## 6. Foreløbige mitigation-anbefalinger (input til fuld DPIA)

Disse er ikke konklusioner, men vil indgå i DPIA-mitigations-fase:

1. **Reducér biometri-eksponering hvor muligt:** alternativer til fuld video (audio + screen-capture + hænder uden ansigt); brug kun fuld video hvis non-repudiation-formålet kræver entydig ansigtsidentifikation
2. **Differentiér retention:** ceremoni-protokol (signed paper) + screen-capture kan opbevares 20 år; biometri-tunge dele (ansigt/stemme) kan eventuelt slettes efter kortere periode (fx 7-10 år) hvis non-repudiation-vinduet udløber
3. **Eksplicit samtykke (art. 9(2)(a))** fra alle deltagere som primær lawful basis; nægter en deltager samtykke skal alternativ deltagelse være mulig (fx udskift witness)
4. **EU-residency** for arkivet (on-prem WORM eller EU-cloud) — ingen tredjelands-overførsel
5. **Adgangskontrol:** dual-control for access til arkiv (Trine + Mads + Dorthe; aktivering kræver 2 personer)
6. **Information ex ante:** deltagere modtager fuld information (art. 13) i god tid før ceremoni; samtykke indhentes skriftligt og kan trækkes tilbage indtil ceremoni begynder
7. **Rettighedsudøvelse:** deltagers art. 15-anmodning besvares; art. 17-anmodning afvises typisk under art. 17(3)(b) (retlig forpligtelse for audit-evidence) men begrænsning (art. 18) overvejes hvis non-repudiation-vinduet er udløbet

---

## 7. Risiko-flagging (foreløbig)

**Høj-risiko-elementer der kræver mitigation eller Datatilsyns-konsultation (art. 36):**
- Biometri-data med 20-årsbevaring uden klar proportionalitets-begrundelse
- Manglende informeret samtykke vil sandsynligvis være "høj rest-risiko" og udløse art. 36

**Mellem-risiko-elementer:**
- Deltagers begrænsede mulighed for at udøve art. 17 (sletning) under projekt-livstid
- Krypterings-nøgle-rotation over 20 år (operationel risiko for tab af adgang vs. tab af konfidentialitet)

**Lav-risiko-elementer (med mitigation):**
- Adgang til arkivet (kan håndteres med least-privilege + dual-control)
- Information til deltagere (kan håndteres med standard art. 13-template)

---

## 8. Sammenhæng med øvrige DPIA-screeninger

Denne screening er parallel med:
- `Team/Dorthe/2026-05-09-dpia-screening-fgdcorepki-ocsp-crl-cdn.md` (conditional, afhænger af arkitekturvalg)

Ingen overlappende behandlingsaktiviteter; uafhængige DPIA-vurderinger.

---

**Forfatter:** Dorthe (DPO)
**Dato igangsat:** 2026-05-09
**Status:** SCREENING-IGANGSAT — afventer F1-F9 og FGD-prioritets-bekræftelse
**Næste milestone:** Fuld DPIA-udkast efter F1-F9 lukket (forventet 3-4 uger fra screening-go)
