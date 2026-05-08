---
dato: 2026-05-07
reviewer: Mads (CISO, bro-reviewer indtil Vibeke aktiveres)
emne: Runde 3 verifikation — Spor 3 (Stefan-CLAUDE.md FGD-override-implementation)
beslutning: GODKENDT
status: leveret
---

# Runde 3 — Spor 3 verifikation

## Resultat: GODKENDT

Laila har i runde 3 implementeret FGD-override korrekt i `CLAUDE.md`. Spor 3 er nu ren, og de 6 andre spor fra runde 2 er fortsat OK. Hele governance-pakken kan committes.

## Verifikation

### Spor 3 — CLAUDE.md linje 30–55

Bekræftet (kontekst af "Hvad du ALDRIG må selv"-blokken):

1. **Mads + Dorthe SAMMEN** (linje 33): "Eskalér til **Mads + Dorthe SAMMEN** (FGD-override 2026-05-06). Begge vurderer ALLE rolle-konfigurations-ændringer for compliance-impact **uafhængigt af trigger**." Eksplicit ikke-trigger-baseret. Korrekt.
2. **§3.3 som superseded** (linje 33): "Reference: `governance/policies/2026-05-06-mads-mandat-stabilitet-governance.md` (§3.3 superseded af FGD-override; se policy-banner)." Historisk reference, ikke operativ. Korrekt.
3. **Ingen "Mads-engangs-undtagelser" som regel** (linje 35): Står som negation: "**Ingen 'Mads-engangs-undtagelser' eller trigger-baserede shortcuts**." Korrekt.
4. **FGD som eneste godkendelses-niveau** (linje 35): "Kun **FGD** kan godkende mandat-ændringer (model, tools, description, hard rules) — **også for project-roller**." Eksplicit og dækker hele rolle-paletten. Korrekt.

### Grep for forbudte mønstre

`grep -nE "trigger-baseret|kun ved persondata|persondata-trigger|Mads konsulterer Dorthe hvis"` returnerer 0 hits hvor mønsteret bruges som operativ regel. De forekomster der findes ("uafhængigt af trigger", "Ingen ... trigger-baserede shortcuts") er negationer — som forventet og acceptabelt.

### Spor 6 — self-modificering i 14 mandater

Bekræftet ved sub-string grep:

- 4 agents: `camilla-bibliotekar.md`, `laila-hr.md`, `mads-ciso.md`, `poul-analytiker.md` — 1 hit hver.
- 4 crosscutting-mandater: `vibeke-...`, `dorthe-dpo`, `trine-compliance-auditor`, `camilla-...-v2` — 1 hit hver.
- 6 project-mandater (knowlagecentral): `sofie`, `henrik`, `anders`, `karen`, `bjarne`, `frederik` — 1 hit hver.

I alt 14 mandater med self-modificerings-klausul intakt. Ingen regression.

### Stage-status

`CLAUDE.md` er staged med `MM`-flag — ændringer i indeks. Resten af governance-pakken er staged som forventet (15+ filer; A/AM/MM-flags). Klar til commit.

## Foreslået commit-message (kategoriseret)

```
governance: implementer FGD-override + self-modificering på tværs af 14 mandater

Stefan-CLAUDE.md (FGD-override 2026-05-06):
- Eskalering for mandat-aendringer kraever Mads + Dorthe SAMMEN, uafhaengigt af trigger
- §3.3 i mandat-stabilitet-policy markeret som superseded
- Kun FGD kan godkende mandat-aendringer — ogsaa for project-roller
- Ingen "engangs-undtagelser" eller trigger-baserede shortcuts

Self-modificerings-klausul tilfoejet i 14 mandater:
- Faste roller: laila-hr, poul-analytiker, camilla-bibliotekar, mads-ciso
- Crosscutting: vibeke, dorthe, trine, camilla-v2
- Knowlagecentral project-roller: sofie, henrik, anders, karen, bjarne, frederik

Bro-review-runde 3 (Mads): GODKENDT.
Reference: governance/reviews/2026-05-07-mads-bro-review-governance-runde3.md
```

(FGD: justér tone/wording efter behov; Camilla håndterer selve commit.)

## Bekræftelser

- Spor 3 ren: ja.
- Spor 1, 2, 4, 5, 6, 7 fra runde 2 fortsat OK: ja.
- Read-only Bash brugt; ingen redigering af kildefiler.
- Staging klar til commit.

—
**Mads (CISO, bro-reviewer)**
