---
dato: 2026-05-08
initieret-af: FGD (direkte instruks i chat)
koordineret-af: Stefan
status: afventer-vurdering
berørte-filer:
  - CLAUDE.md
  - .claude/agents/stefan-organisator.md
  - .claude/agents/camilla-bibliotekar.md (Hard Rule 8 + invalidity-klausul)
  - .claude/agents/laila-hr.md
  - .claude/agents/poul-analytiker.md
  - .claude/agents/mads-ciso.md
  - .claude/agents/dorthe-dpo.md
  - .claude/agents/trine-compliance-auditor.md
  - .claude/agents/vibeke-commit-reviewer.md
---

# Change-request: Invalidity-klausul + Camilla Hard Rule 8

## Baggrund

FGD præciserede 2026-05-08 i chat: FGD's godkendelse af en mandat-ændring er ikke gyldig uden
at Mads (CISO) og Dorthe (DPO) begge har afgivet skriftlig vurdering. En FGD-godkendelse
afgivet uden disse vurderinger er ugyldig og Camilla må ikke handle på den.

## Ændring 1 — Invalidity-klausul i alle mandater

**Felt**: Hard rules / Mandat-stabilitet §5 (alle 8 mandater + CLAUDE.md)

**Nuværende tekst** (trin 5 i mandat-stabilitets-sektioner):
> "5. Først efter **eksplicit FGD-go** må Camilla committe ændringen"

**Foreslået tekst**:
> "5. Først efter **eksplicit FGD-go** — og kun når trin 3's skriftlige vurderinger fra Mads
> OG Dorthe begge foreligger i `governance/change-requests/` — må Camilla committe ændringen.
> **FGD-godkendelse uden disse vurderinger er ugyldig og Camilla må ikke handle på den.**"

**Og tilsvarende i CLAUDE.md** efter bullet 4 om FGD-godkendelse:
> "**FGD's godkendelse er ugyldig** uden skriftlige vurderinger fra Mads (security/SOD/ISO 27001)
> OG Dorthe (GDPR/persondata). Stefan må aldrig præsentere en mandat-ændring for FGD uden at
> begge vurderinger foreligger i `governance/change-requests/`. En FGD-godkendelse afgivet i
> fravær af disse er ugyldig — Camilla må ikke committe på baggrund af den."

**Rationale**: Tightening af eksisterende governance-krav. Ingen ny compliance-forpligtelse
indføres — eksisterende krav gøres eksplicit uomgåeligt.

## Ændring 2 — Camilla Hard Rule 8 (README-vedligehold)

**Felt**: Hard rules (nyt punkt 8 i `.claude/agents/camilla-bibliotekar.md`)

**Indhold**:
Ny regel: inden `git push` til origin læser Camilla README.md og opdaterer den hvis
team-tabel, mappestruktur eller aktivt projekt er forældet. Resultat rapporteres til Stefan.

**Rationale**: Sikrer at README.md afspejler faktisk tilstand ved enhver push til origin.
Ingen persondata-behandling. Ingen SOD-risiko.

## Vurderingsanmodning

**Mads (CISO)**: Vurder security/SOD-impact af begge ændringer. Specifikt:
- Invalidity-klausulen: øger den compliance-niveau uden at introducere nye SOD-problemer?
- Hard Rule 8: er der security-risici ved at Camilla læser/opdaterer README.md som del af push-flow?

**Dorthe (DPO)**: Vurder GDPR/persondata-impact af begge ændringer. Specifikt:
- Invalidity-klausulen: påvirker den DPO's uafhængighed (art. 38(3)) eller breach-notification-flow?
- Hard Rule 8: behandler README.md-vedligehold persondata?

## Forventet output

- `Team/Mads/2026-05-08-vurdering-invalidity-klausul.md` (Mads' skriftlige vurdering)
- `Team/Dorthe/2026-05-08-vurdering-invalidity-klausul.md` (Dorthes skriftlige vurdering)
- Stefan samler og præsenterer for FGD
