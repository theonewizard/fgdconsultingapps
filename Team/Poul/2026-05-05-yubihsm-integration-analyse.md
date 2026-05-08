---
dato: 2026-05-05
forfatter: Poul (analyse) + Stefan (FGD-beslutninger)
emne: YubiHSM 2 integration — hosting, use cases, arkitektur, rolle-anbefaling (Karen + Bjarne)
status: leveret + FGD-beslutninger truffet
relateret: Team/Poul/2026-05-05-secrets-fips-analyse.md
relateret: Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md
---

# YubiHSM 2 Integration — FGD's vej til HSM-understøttelse

## 1. FGD's beslutninger (2026-05-05)

| Spørgsmål | Beslutning | Impact |
|---|---|---|
| **FIPS-certificering** | Non-FIPS YubiHSM 2 nu (€728); FIPS-enheder når kunde betaler | Starts dev-fase med non-FIPS; ikke blokket af cert-venting |
| **Hardware-budget** | 1× non-FIPS YubiHSM 2 (€728, engangs) | MVP ikke påvirket; indkøb skal ske nu (leveringstid 5–10 dage) |
| **Hosting-model** | On-prem FGD-kontor + Tailscale-tunnel | Single point of failure i MVP (accepteret risiko); 2× FIPS-enheder + colocation senere |
| **Roller** | 2 nye roles — Karen (rådgiver) + Bjarne (operatør) | Separation of duties; Karen designeres først |
| **FIPS-cert-timing** | Vent; bruk "FIPS-aware best practices" | Reduserer compliance-tryk, sætter HSM parallelt med Infisical-fase-2 |
| **Cost-ramme** | €728 hardware + ~$0–10/md Tailscale | Under $250/md samlet til v3 launch (ingen FIPS-enheder endnu) |

---

## 2. Hovedkonklusioner fra Pouls analyse

### Hardware — YubiHSM 2 v2.4 vs. FIPS-variant

| Aspekt | Non-FIPS v2.4 | FIPS v2.2 | Konsekvens |
|---|---|---|---|
| **Pris** | €728 (SAL, tilgængelig nu) | €1.187,50 (FIPS-cert) | FGD køber non-FIPS nu; FIPS udskudt |
| **Certificering** | Ingen | FIPS 140-3 (forventes Q2 2026; v2.2 cert udløb 2. maj 2026) | Ikke at vente i MVP |
| **USB-form-factor** | Nano A (passe i pocket) | Nano A | Samme |
| **Sessions** | 16 concurrent | 16 concurrent | Samme |
| **Object-slots** | 256 | 256 | Samme |
| **Algoritme-sæt** | RSA, ECDSA, EdDSA (Ed25519), AES-CCM-Wrap, HMAC-SHA*, ECDH | Subset (FIPS-aligned) | Non-FIPS har ECB/CBC; vi bruger CCM for invites |
| **Performance-estimate** | ECDSA-P256 ~73ms; AES-CCM ~10ms | Samme | Acceptable for magic-link signing |

**Konsekvens**: Bestil non-FIPS nu; giver 5–10 dage leveringstid. FIPS-enheder til kunde-betalt fase-2–3.

### FIPS 140-2 sunset

- **CMVP decision** (1. apr 2022): Stop accept af nye FIPS 140-2 valideringer
- **Sunset date** (21. sept 2026): Alle 140-2 certs på "Historical List"
- **FGD's position**: "FIPS-aware best practices" acceptabel for SMV-målgruppe; fuld Level 3-cert ikke påkrævet i MVP

---

## 3. Arkitektur for FGD's valg (on-prem Tailscale)

### Model A: On-prem + Tailscale-tunnel (VALGT)

```
┌─────────────────────────────────────────────┐
│ Vercel Function (Node.js Fluid Compute)     │
│ - route: /api/sign/invite                   │
│ - calls: HSM-bridge service via Tailscale   │
└────────────────┬────────────────────────────┘
                 │ HTTPS via Tailscale VPN
                 │ (no open ports; ACL-based)
                 ↓
    ┌────────────────────────────────┐
    │ FGD-kontor (on-prem host)      │
    │  - HSM-bridge service (Go/Rust)│
    │  - Tailscale client (Vercel)   │
    │  - yubihsm-connector :12345    │
    └────────────┬───────────────────┘
                 │ USB
                 ↓
         ┌───────────────┐
         │ YubiHSM 2     │
         │ (non-FIPS)    │
         └───────────────┘
```

**Konsekvenser**:
- FGD-kontor skal ha stabil internet + UPS til HSM-host
- Tailscale free tier OK til MVP; business tier (~$10/md) hvis ACL-kompleksitet senere
- Single point of failure (accepteret risiko; ikke prod endnu)
- Parallelt: design KMS-backed secret-rotation via Infisical (fase 2) så ikke alle secrets afhænger af HSM

### Model B (fravalgt): AWS CloudHSM via Hetzner

- €870/md + data-transfer; colocation kompleks; overkill uden kunde-sponsor

---

## 4. Use cases pr. fase

| Fase | HSM use case | Timing | Note |
|---|---|---|---|
| **Fase 1 MVP** | Ingen | — | Magic-link signing forbliver software; HSM udvikles parallelt |
| **Fase 1.5** | Magic-link invite-token signing (HMAC-SHA256) | Efter HSM operationelt | First production HSM flow; 1–2 dage test |
| **Fase 2** | Field-level encryption + audit-log signing | +1–2 uger efter fase 1.5 | Kræver bridge-service stabilitet; Karen review |
| **Fase 3+** | Clerk Enterprise BYOK (hvis kunde-kontrakt kræver) | TBD | Only if customer mandates (financial vertical) |

**MVP-fokus**: Fase 1 release delay = NOPE. HSM-integration ved fase 1.5 er acceptable; test på non-prod branches først.

---

## 5. Roller: Karen + Bjarne

### Karen — Security Architect (NY)

**Profil**:
- Deltids-rådgiver, Sonnet 4.6 eller 3.5 (confirm w/ Laila)
- **FTE**: 0,3–0,5 (prioritet på HSM-threat-model + key-ceremony runbook)
- **Startdato**: Post-Laila-design (1–2 dage efter Go)

**Ansvar**:
- HSM architecture review (Tailscale + bridge-service design)
- Threat-model STRIDE (HSM theft, USB sniffing, Tailscale compromise)
- Key-ceremony design + runbook (installation, wrap-key-rotation, DR-test procedure)
- Secure code review på Henrik's HSM-bridge (Go/Rust)
- FIPS-roadmap vurdering (når kunde-request kommer)
- DPA-review (HSM vendor terms, data-flow dokumenation)

**Tools**: Read, Glob, Grep, WebSearch, WebFetch, Bash (read-only), Agent (Camilla+Poul), TodoWrite. **Aldrig** Edit/Write til prod-kode.

**Deliverables**:
- Threat-model dokument (1–2 sider)
- Key-ceremony runbook (Team/Karen/2026-05-xx-key-ceremony-runbook.md)
- Secure code review af HSM-bridge før deploy
- FIPS roadmap (hvornår + cost hvis kunde kræver Level 3)

---

### Bjarne — HSM Operatør (NY)

**Profil**:
- Operatør-rolle, Haiku 4.5 (eller Sonnet 3.5 hvis kompleks ops)
- **FTE**: 0,2–0,3 (weekly ops check + ad-hoc HSM-events)
- **Startdato**: Post-Karen key-ceremony-design; efter non-FIPS hardware leveres

**Ansvar**:
- Execute key-ceremony per Karen's runbook (installation, wrap-key-generation)
- HSM firmware-updates (quarterly check, staging-test først)
- Wrap-key-rotation (semi-annual eller post-security-event)
- DR-testing (restore fra backup, failover til cold-backup)
- Sentry alarm response (HSM connectivity, session-exhaust)
- Never destructive (no delete, factory-reset uden explicit FGD + Karen-approvals)

**Tools**: Read, Bash (`yubihsm-shell` + ops-commands), Edit på runbooks i Team/Bjarne/, Agent (Karen+Camilla), TodoWrite. **Aldrig** prod-kode.

**Deliverables**:
- Execution-logs per key-ceremony
- Firmware-update-report
- DR-test results + remediation-notes
- Sentry-response log

---

### Henrik — HSM-bridge developer (eksisterende)

**Tilføjet ansvar**:
- Design + build HSM-bridge service (Go eller Rust, ikke Node.js — undgå USB-stack-bug i Node)
- Implement `/api/sign/*` endpoints i Next.js (callthrough til bridge)
- Audit-log integration (sign request + HSM response)

**Bridge-spec**:
```
POST http://localhost:12345/sign
{
  "key_id": 1,
  "algorithm": "HMAC-SHA256",
  "data": "<base64-encoded-invite-token>",
  "signature_format": "hex"
}

Response:
{
  "signature": "<hex-encoded HMAC>",
  "timestamp": "2026-05-15T10:30:00Z"
}
```

---

## 6. Cost breakdown + timeline

### Hardware cost (non-FIPS MVP)

- **YubiHSM 2 non-FIPS**: €728 (engangs)
- **Leveringstid**: 5–10 dage til DK (order nu)
- **USB-kabel + bracket**: ~€50 (lokalt køb)

**Total hardware MVP**: €778 (~6000 DKK)

### Hosting cost (Tailscale)

- **Tailscale free tier**: $0/md (up to 20 devices, adequate for MVP+phase 1)
- **Tailscale Business** (if ACL grows): ~$10/md per user/device

### Personnel cost

- **Karen** (threat-model + runbook): ~40–60 timer over 2–3 uger @ 0,3–0,5 FTE = ~$800–1200 (internal est.)
- **Bjarne** (ops prep + execution): ~20–30 timer over 3 weeks @ 0,2–0,3 FTE = ~$400–600
- **Henrik** (bridge-service): already budgeted under knowlagecentral backend

### Infrastructure cost (ongoing)

- **On-prem HSM host**: $0 (FGD's existing server + UPS upgrade TBD)
- **Anthropic API** (no HSM sign-cost): included in existing Vercel AI SDK budget
- **Neon Postgres**: no HSM-specific surcharge; RLS + encryption at-rest covered

**Total till phase 2**: ~€778 hardware + $0–10/md Tailscale + Karen/Bjarne FTE = **under $250/md** till FIPS-enheder purchased.

### Timeline

| Milestone | Date | Dep | Note |
|---|---|---|---|
| Order non-FIPS HSM | 2026-05-05 | FGD | 5–10 dage delivery |
| HW arrives + unbox | ~2026-05-15 | Bjarne | Verify serial, store secure |
| Karen: threat-model | ~2026-05-16 to 2026-05-20 | Karen | STRIDE + runbook design |
| Bjarne: key-ceremony prep | ~2026-05-20 to 2026-05-22 | Bjarne | Set up USB host, test yubihsm-shell |
| Henrik: bridge-service scaff | ~2026-05-16 to 2026-05-20 | Henrik | Go skeleton + endpoint sig |
| **Phase 1 MVP launch** | ~2026-05-30 | Team | No HSM in critical path |
| Karen: code-review bridge | ~2026-05-28 to 2026-05-29 | Karen | Security sign-off before test-deploy |
| Bjarne: execute key-ceremony | ~2026-06-01 | Bjarne | Installation + wrap-key gen |
| **Phase 1.5: HSM live** | ~2026-06-05 | Henrik+Bjarne | Invite-token signing via HSM |

---

## 7. Critical blockers before Bjarne can start

1. **FGD orders non-FIPS YubiHSM 2 (€728)** — do today; delivery 5–10 days
2. **FGD creates Tailscale account** (free tier) + adds Vercel + FGD-kontor host as nodes
3. **Karen design complete** (key-ceremony runbook) — before Bjarne touches hardware
4. **HSM-bridge repo scaffolding** (Henrik) — ready for Bjarne integration-test
5. **Laila designs Karen + Bjarne mandates** + approves tool-sets

**Non-blocking for phase 1**: HSM ops. Phase 1 uses software signing; HSM goes live phase 1.5.

---

## 8. Future phases (FIPS + multi-HSM)

### Phase 2–3: FIPS 140-3 enheder + redundancy

Når kunde-betaler kommer:

1. **Order 2× YubiHSM 2 FIPS v2.2** (€1.187,50 each)
   - Primary: upgrade FGD-kontor host
   - Secondary: Hetzner colocation (warm standby + backup-wrap-keys)
2. **Implement multi-HSM failover** (Henrik bridge-service)
   - Health-check + automatic failover to secondary
   - Wrap-key sync between primaries (encrypted)
3. **Full FIPS 140-3 compliance review** (Karen)
   - DPA updates for HSM vendor (Yubico)
   - Pentest scope expansion
   - Cert readiness report

**Cost estimate**: €2375 (2× FIPS units) + €500 Hetzner colocation-setup + 40 hour engineering = ~$4500–5500 one-time + $50–100/md ongoing.

---

## 9. Sources & references

- Yubico YubiHSM 2 product page: https://yubico.com/product/yubihsm-2/
- Yubico YubiHSM 2 FIPS variant: https://yubico.com/product/yubihsm-2-fips/
- YubiHSM docs: https://docs.yubico.com/hardware/yubihsm-2/
- FIPS 140-2 sunset (NIST): https://csrc.nist.gov/projects/cryptographic-module-validation-program/fips-140-2
- FIPS 140-3 spec: https://csrc.nist.gov/publications/detail/fips/140-3/final
- Tailscale pricing: https://tailscale.com/pricing
- Tailscale funnel (secure tunnel): https://tailscale.com/blog/tailscale-funnel/
- Karen role — see secrets-fips-analyse.md (5.1–5.4)
- Pouls fulde sekretarie-analyse: Team/Poul/2026-05-05-secrets-fips-analyse.md

---

## 10. Åbne spørgsmål til FGD

1. **On-prem UPS upgrade**: Budget til FGD-kontor power-backup? (Anbefalet: 2 timer UPS min.)
2. **Tailscale account**: Dine kontakt-detaljer for Team Inbox-registrering?
3. **Henrik's HSM-bridge lang**: Go eller Rust? (Anbefaling: **Go** for typesikkerhed + yubihsm-go SDK).
4. **Bjarne-timing**: Kan Bjarne starte efter Laila-design (~5 dage), eller skal han binde ressourcer før HW ankommer?

---

**Status**: Analyse + FGD-beslutninger på plads. Afventer: (1) Laila: Karen + Bjarne mandate-design, (2) FGD: hardware-bestilling + Tailscale-setup, (3) Henrik: bridge-scaffolding start.

**Næste skridt for Stefan**: Hand off til Laila for rolle-design; notér hardware-bestilling deadline (i dag).
