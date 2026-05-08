---
dato: 2026-05-08
forfatter: Dorthe (DPO)
type: Final-check (verifikation, ikke fuldt review)
emne: Application Owner v2 + Application Provider v2 — verifikation af 5 blokerende rettelser
basis-filer:
  - Team/Laila/2026-05-08-udkast-application-owner-mandat-v2.md
  - Team/Laila/2026-05-08-udkast-application-provider-mandat-v2.md
relateret-4eyes:
  - Team/Dorthe/2026-05-08-dorthe-4eyes-app-owner-provider.md
status: GO
---

# Final-check — Application Owner v2 + Application Provider v2

**Type:** Verifikations-check, ikke fuldt review. Bekræfter at de 5 blokerende rettelser fra mit 4-eyes review er korrekt indarbejdet.

---

## Verifikation af de 5 punkter

| # | Punkt | Owner v2 | Provider v2 | Status |
|---|---|---|---|---|
| 1 | Cross-rolle constraint: ingen ad-hoc-undtagelser | Linje 77 | Linje 130 | OK |
| 2 | DPO-uafhængigheds-sektion (4 bullets, art. 38(3)) | Linjer 91-96 | Linjer 112-117 | OK |
| 3 | Eskalationspligt ved tvivl (Provider) | n/a | Linje 109 | OK |
| 4 | ROPA-notifikationsformat + art. 30-konsekvens (Provider) | n/a | Linje 105 | OK |
| 5 | Policy-reference §3.3 superseded af FGD-override 2026-05-06 | Linje 106 | Linje 127 | OK |

**Alle 5 punkter bekræftet indarbejdet i v2-udkastene.**

---

## Detaljerede observationer

**Punkt 1 (cross-rolle constraint):** Korrekt formulering identisk i begge mandater. Lukker hullet hvor AO + AP i fællesskab kunne lave ad-hoc-undtagelser uden Mads/Dorthe-godkendelse.

**Punkt 2 (DPO-uafhængighed):** 4 bullets dækker GDPR art. 38(3): (a) ikke-instruktion, (b) eskalationspath FGD direkte, (c) ingen filter-position, (d) ikke-sanktionering. Identisk implementering i begge — det er korrekt.

**Punkt 3 (eskalationspligt ved tvivl):** Provider har eksplicit "default: hvis i tvivl, vent på DPO-svar — implementer ikke". Det er den formulering jeg bad om. Owner-mandatet har også fået samme bullet (linje 89) — bonus, ikke krævet, men styrker mandatet.

**Punkt 4 (ROPA-notifikationsformat):** Provider linje 105 indeholder alle 4 minimum-felter (hvad, hvilken ROPA-aktivitet, dato, drevet-af) plus art. 30-konsekvens-formulering ("blokerer change-request, redegør over for DPO + Trine"). Dette gør GDPR art. 30-pligten enforceable og kobler sanktion til processen.

**Punkt 5 (policy-supersedance):** Begge mandater refererer eksplicit FGD-override 2026-05-06 og at §3.3 i Mads-policyen er superseded. Det er nødvendigt for at undgå at en fremtidig læser falder tilbage på den oprindelige policy-tekst.

---

## Show-stoppers

**Ingen.** Alle 5 blokerende rettelser er indarbejdet korrekt. Der er ingen GDPR-relaterede show-stoppers tilbage fra mit 4-eyes review.

---

## Mindre observation (ikke blokerende)

Owner v2 linje 89 har også fået "Eskalations-pligt ved tvivl"-bullet (ud over Provider-mandatet). Det var ikke et af mine 5 krav, men det er en god styrkelse. Ingen action.

---

## Konklusion

**GO** fra DPO-perspektiv.

Begge mandater (Application Owner v2 + Application Provider v2) er klar til FGD-godkendelse for så vidt angår mine 5 blokerende GDPR/uafhængigheds-rettelser. Mads' security/SOD-vurdering køres parallelt og er ikke en del af min vurdering her.

---

**Forfatter:** Dorthe (DPO)  
**Dato:** 2026-05-08  
**Status:** GO — verificeret
