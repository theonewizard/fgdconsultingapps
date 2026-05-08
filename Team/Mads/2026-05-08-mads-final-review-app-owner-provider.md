---
dato: 2026-05-08
forfatter: Mads (CISO)
emne: Final-check — Application Owner + Application Provider mandat v2
filer-reviewet:
  - Team/Laila/2026-05-08-udkast-application-owner-mandat-v2.md
  - Team/Laila/2026-05-08-udkast-application-provider-mandat-v2.md
status: GO
---

# Final-check — AO + AP mandat v2

Verificering af de 5 blokerende rettelser fra Mads' 4-eyes.

---

## Punkt 1 — Self-certify-forbud dækker ALLE erklæringer (intern + ekstern)

**AO:** INDARBEJDET. Hard rule linje 72 dækker eksplicit: "hverken over for kunder, internt i teamet, eller i dokumentation". GDPR-sektion linje 85 gentager for marketing/kontrakter. Begge dimensioner dækket.

**AP:** INDARBEJDET. Anti-patterns linje 148: "Vurderer ikke selv om feature er 'compliant'". Hard rule linje 106 forbyder compliance-claims til kunder/revisorer. Internt self-certify er implicit dækket via kravet om Mads-kvittering ved compliance-flagging (linje 82).

Status: OK begge mandater.

---

## Punkt 2 — Break-glass audit trail har immutabilitetskrav

**AO:** Ikke relevant — AO har ikke break-glass-adgang til prod-data. Korrekt udeladt.

**AP:** INDARBEJDET. Linje 76: "Audit-trail for break-glass-sessioner gemmes i uforanderlig form — write-once storage eller eksporteret til separat immutabelt arkiv. Hverken du, Camilla, eller andre roller der producerer eller eksekverer kan modificere dette log. Trine auditerer break-glass-log halvårligt som SOC 2-evidens."

Immutabilitetskrav er præcist, navngiver konkrete roller der ikke må modificere, og knytter til SOC 2-evidens. Stærkt formuleret.

Status: OK.

---

## Punkt 3 — Vibeke-reference er "commit-reviewer (Vibeke, når aktiv)"

**AO:** INDARBEJDET. Ansvarsfordelingstabel linje 135: "Commit-reviewer (Vibeke, når aktiv) reviewes PRs før de merges." Formuleringen følger den godkendte skabelon.

**AP:** INDARBEJDET. To steder: linje 77 (infrastruktur-ændringer via Camilla) og linje 161 (ansvarsfordelingstabel). Hand-off-aftaler linje 172 anvender samme formulering. Konsistent gennemført.

Status: OK begge mandater.

---

## Punkt 4 — Overgangsscenarie-sektion tilføjet i begge

**AO:** INDARBEJDET. Sektion "Overgangsscenarie (periode med kun én besat rolle)" linje 110-114. Dækker: kun-AO, kun-AP, og det absolutte SOD-forbud mod midlertidig dobbeltrolle.

**AP:** INDARBEJDET. Tilsvarende sektion linje 132-136. Spejler AO-sektionen korrekt med rollerne byttet om.

Begge sektioner indeholder det kritiske absolutkrav: "Ingen person må varetage begge roller midlertidigt — også ikke 'lige nu til vi finder nogen'."

Status: OK begge mandater.

---

## Punkt 5 — AP breach-eskalering direkte til Dorthe + Mads, ikke via Owner

**AP:** INDARBEJDET. Linje 110: "Breach-mistanker rapporteres direkte til Dorthe (DPO) og Mads (CISO) inden 24 timer — ikke via Application Owner som mellemled. Tid er kritisk under GDPR art. 33 (72-timers notifikationsvindue). Du behøver ikke afvente Owner-bekræftelse før eskalering."

Formuleringen er eksplicit, navngiver begge modtagere, angiver 24-timers frist, og fjerner Owner som mellemled. Præcist som krævet.

**AO:** Konsistent: AO-mandatet angiver (linje 86) at breach-mistanker eskaleres til Dorthe inden 24 timer — AO er trigger, ikke filter for AP's eskalering.

Status: OK.

---

## Samlet vurdering

Alle 5 blokerende rettelser er korrekt og præcist indarbejdet i begge mandater. Ingen nye show-stoppers identificeret under gennemlæsningen.

**RESULTAT: GO**

Begge mandater er klar til FGD-godkendelse. Dorthe bekræfter sin del separat (GDPR-sektioner er Lailas gengivelse af Dorthes krav — Dorthe skal kvittere for korrekt gengivelse).

---

**Mads (CISO)**
**Dato:** 2026-05-08
