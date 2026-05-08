---
dato: 2026-05-05
forfatter: Laila
emne: Rapport - Karen + Bjarne roller designet
status: til FGD-godkendelse
---

# Karen + Bjarne — To nye sikkerheds-roller (rapport)

## TL;DR

Jeg har designet to nye roller efter Pouls analyser af YubiHSM-strategi og sikkerhedsarkitektur:

1. **Karen — Security Architect** (Sonnet, deltids 0,3–0,5 FTE)
   - Threat-modelling (STRIDE), secret-management design, code-review (crypto/auth/BOLA), FIPS-roadmap, DPA-review
   - Ikke udvikler — designerer, reviewer, adviserer

2. **Bjarne — HSM-operatør** (Haiku, deltids 0,2–0,3 FTE)
   - Eksekverer rutine-procedurer: key-ceremony, firmware-updates, wrap-key-rotation, DR-tests, Sentry-respons
   - Separation of duties — Karen designer, Bjarne udfører (best-practice for regulerede brancher)

**Modeller**: Karen = Sonnet (ræsonnement over arkitektur). Bjarne = Haiku (rutine-eksekvering, kan eskalere til Sonnet ved incident).

---

## Roller — samlet tabel

| Aspekt | Karen | Bjarne |
|---|---|---|
| **Titel** | Senior security architect | HSM-operatør |
| **Model** | Sonnet (claude-sonnet-4-6) | Haiku (claude-haiku-4-5) |
| **FTE** | 0,3–0,5 (5–7 uger Fase 0–2) | 0,2–0,3 (forberedelse + routine) |
| **Ansvar** | Threat-model, code-review, secret-design, FIPS-roadmap, DPA | Key-ceremony eksekvering, firmware, wrap-key-rotation, DR, Sentry-respons |
| **Produktionskode** | Nej (Edit/Write blokeret) | Nej (Edit/Write blokeret på prod) |
| **Eskalering** | Mads (CISO), evt. via Poul ved research | Sonnet (incident) eller Karen |
| **Hard rule** | Reviewer og designerer kun — skriver ikke kode | Eksekverer kun dokumenterede procedurer — destruktive ops kræver FGD+Karen godkendelse |
| **Work-folder** | Team/Karen/ | Team/Bjarne/runbooks/ |
| **Starter efter** | Laila-design, dag 1 Fase 0 | Karen key-ceremony-runbook + hardware-ankomst |

---

## Ansvarsfordeling & Hand-offs

### Karen ↔ Bjarne: kerneflow

```
Karen designer → Bjarne eksekverer
  ↓
Karen: "Threat-model HSM-installation + 4-eyes runbook" (`.md` i Team/Karen/)
  ↓
Bjarne: "Har læst runbook; klar til at eksekvere når hardware ankommer"
  ↓
Bjarne eksekverer + rapporterer: "Key-ceremony OK; wrap-key exported; Sentry-setup klar"
  ↓
Karen: "Verificerer resultat mod threat-model; giver go for Henrik-integration"
```

**Destruktive ops constraint**: Bjarne må ALDRIG køre `delete-object`, `factory-reset`, `key-erase` uden:
1. Karen beder eksplicit ("denne destruktiv op er nødvendig fordi...") i samme session
2. FGD bekræfter fra Team Inbox eller direkte besked

### Karen → Henrik (backend): code-review på HSM-bridge

```
Karen: "Code-review på HSM-bridge service (Go/Rust)"
  ↓
Henrik: Submitter PR med `/api/sign` endpoints + yubihsm SDK-integration
  ↓
Karen: Reviews for SSRF, key-leakage, audit-log-completeness, error-handling
  ↓
Henrik: Fixes feedback
  ↓
Karen: "Security sign-off ✓"
```

### Karen → Anders (AI-pipeline): Inngest + spend-cap review

```
Karen: Threat-model for AI-pipeline: spend-cap overflow, prompt-injection, ZDR-flag
  ↓
Anders: Implementerer Inngest-jobs + Anthropic-integration
  ↓
Karen: "Verificér Anthropic ZDR aktiveret på prod-workspace før kunde-data"
  ↓
Anders: Checklist bekræftelse
```

### Bjarne → Henrik: HSM-operationale fund

```
Bjarne: "Key-ceremony eksekveret; yubihsm-connector køres på port 12345"
  ↓
Henrik: Tester bridge-forbindelse til Bjarne's HSM-host
  ↓
Hvis fejl: "Session-timeout på 5 min; har du timeout-config?"
  ↓
Bjarne: Rapporterer tilbage, koordinerer fix
```

### Camilla ↔ Karen: DPA + secret-rotation

```
Karen: "Yubico HSM vendor-DPA: kræver HSM serial + firmware-version ved kontrakt"
  ↓
Camilla: Tracker i `Team/Karen/dpa-tracker.md` + opretter remind-task for renewal
  ↓
Bjarne: Rapporterer firmware-version efter update
  ↓
Camilla: Opdaterer DPA-tracker med ny version
```

### Stefan ↔ Karen + Bjarne: reporting & escalation

```
Karen rapporterer til Stefan:
  - Threat-model-resultat ("STRIDE-tabel vedhæftet")
  - Sikkerheds-risici: kritisk/høj/medium/lav
  - Code-review fund ("3 items i Henry PRs, 1 opsummering i Min Inbox/")

Bjarne rapporterer til Stefan:
  - Routine-status ("HSM firmware updated; wrap-key rotated")
  - Incident/anomali ("Sentry HSM-bridge timeout; undersøgt; fix deployed")
  - Runbook-læring ("3 steps ændret baseret på praksis; updated Team/Bjarne/runbooks/")

Stefan prioriterer fixes med relevante developers.
```

---

## Implementering-timeline

| Milepæl | Dato (estimat) | Ansvarlig | Note |
|---|---|---|---|
| FGD godkender begge roller | 2026-05-05 | FGD | Eller beder om justeringer |
| Camilla committer `.claude/agents/` | 2026-05-06 | Camilla | Karen + Bjarne in `.claude/agents/` |
| Camilla opretter Team/Karen/ + Team/Bjarne/runbooks/ | 2026-05-06 | Camilla | Work folders ready |
| FGD har bestilt 2× YubiHSM 2 FIPS (Layer 2: online+offline) | 2026-05-05 (nu!) | FGD | 5–10 dage leveringstid |
| Karen: Fase 0 threat-model + key-ceremony runbook | 2026-05-16 til 2026-05-20 | Karen | Starter dag 1 Fase 0 |
| Hardware ankommer FGD | ~2026-05-15 | Logistics | Verify serial + sikker opbevaring |
| Bjarne: Forberedelse (USB-host-setup, read docs) | 2026-05-20 til 2026-05-22 | Bjarne | Klart når Karen's runbook er færdig |
| Henrik: Bridge-service scaffolding | 2026-05-16 til 2026-05-20 | Henrik | Parallel med Karen |
| Fase 1 MVP launch (uden HSM i kritisk path) | ~2026-05-30 | Team | HSM-integration i fase 1.5 |
| Karen: Code-review Henrik's bridge | 2026-05-28 til 2026-05-29 | Karen | Security sign-off før Bjarne test |
| Bjarne: Eksekvér key-ceremony | ~2026-06-01 | Bjarne | Live installation + wrap-key-generation |
| Fase 1.5: HSM live (invite-token signing) | ~2026-06-05 | Henrik+Bjarne | First production HSM flow |

---

## Praktisk: FGD's næste skridt

1. **Godkend eller juster** Karen + Bjarne roll-definitions nedenfor:
   - `Min Inbox/projects/vidensapp/2026-05-05-karen-security.md` (foreslået `.claude/agents/karen-security.md`)
   - `Min Inbox/projects/vidensapp/2026-05-05-bjarne-hsm-operator.md` (foreslået `.claude/agents/bjarne-hsm-operator.md`)

2. **Bestil hardware NU** (ikke-blokeret på rolle-godkendelse):
   - 2× YubiHSM 2 FIPS (bestilt 2026-05-05, leveringstid 5–10 dage) — **MÆRKE: Oprindelig plan var 1× non-FIPS €728, men FGD skiftede til 2× FIPS for Layer 2 (online+offline)**
   - USB-kabel + bracket (~€50 lokalt køb)
   - Leveringstid: 5–10 dage

3. **Opret Tailscale-konto** (free tier ok til MVP):
   - FGD-konto på tailscale.com
   - Tilføj Vercel + FGD-kontor-host som nodes (kan gøres da Camilla sætter det op)

4. **Efter godkendelse**: Stefan kalder Camilla for commit + arbejdsmappe-oprettelse.

---

## Åbne valg jeg har truffet på Stefans vegne

1. **Karen: model = Sonnet** (ikke Opus)
   - Begrundelse: Sonnet har tilstrækkelig ræsonnement til threat-modeling + code-review. Opus (Pouls model) reserveres til dyb research som Poul allerede dækker. Karen eskalerer til Mads ved compliance-spørgsmål; Mads kan så kalde Poul ved behov for historisk arkæologi eller FIPS-lovpræsentation.

2. **Bjarne: model = Haiku som default** (med Sonnet-eskalering ved incident)
   - Begrundelse: HSM-ops er strukturerede procedurer (dokumenteret i runbooks). Haiku håndterer 80%. Hvis Sentry-alarm kræver diagnose-dybde, kan Bjarne eskalere inden esecution.

3. **Begge roller: Bash-tools OK til read-only** (f.eks. `git log`, `yubihsm-shell --info`, `journalctl`)
   - Begrundelse: Nødvendigt for troubleshooting + audit-verificering. Destruktive Bash-ops blokeret via hard rules + godkendelse-gating.

Hvis FGD har andre præferencer (fx Opus for Karen, eller helt uden Bash), kan jeg tilpasse.

---

## Næste skridt for teamet

1. **FGD**: Godkend roller + bestil hardware
2. **Camilla**: Commit `.claude/agents/` + opret arbejdsmapper
3. **Stefan**: Kalder Karen for Fase 0 threat-modeling
4. **Karen**: Starter threat-model + runbook-design
5. **Bjarne**: Venter på hardware + Karen runbook før eksekvering
6. **Henrik**: Parallelt bridge-service scaffolding

---

**Status**: Karen + Bjarne roll-definitions klar til godkendelse i `Min Inbox/`. Udkastet er klart til commit hvis FGD siger ja.
