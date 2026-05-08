# Vibeke — Commit-reviewer rolle | Godkendelses-rapport

**Status**: Udkast til FGD-godkendelse  
**Design-dato**: 2026-05-05  
**Baseret på**: Mads' SOD-vurdering (`governance/policies/2026-05-05-mads-sod-vurdering-reviewer-rolle.md`)

---

## TL;DR

Mads anbefaler **ny dedikeret rolle — Vibeke** som dokumentations-reviewer & commit-gate. Trine kan ikke bære funktionen uden at bryde revisor-uafhængighed (hun auditer sin egen kontrol). Vibeke er pure read-only (ingen Edit/Write), kalder kun Stefan + Mads, og pauserer commits helt hvis hun ikke svarede — ingen backup-reviewer-risiko. Design følger Mads' specifikation §3 punkt-for-punkt.

**Mandat-udkast**: `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md`

---

## Hvorfor ny rolle, ikke Trine-udvidelse?

**3 konflikter hvis Trine barer gate'en**:

1. **Audit-uafhængighed (primær). **Trine indsamler audit-evidens fra git-historikken ("vi committer kun efter godkendelse" = SOC 2 CC8.1-kontrol). Hvis hun selv *er* gate'en, beviser hun sit eget arbejde — ekstern SOC 2 Type II-revisor vil afvise som ikke-uafhængig.

2. **Compliance-evidens-konflikt**. Trines mandat siger "Camilla leverer raw data; Trine analyserer." Hvis Trine godkender commits, kontrollerer hun inputtet til sin egen analyse.

3. **Scope-overload**. Trine ejer SoA + Trust Service Criteria + månedsrapporter + audit-cyklus. Dagligt commit-review vil flaskehalse hendes arbejde.

**Vibeke løser alle tre**: Hennes role er *ren gate* uden eget produktions-arbejde (ingen Edit/Write) → ingen selv-referencer → uafhængighed skabes by-design.

---

## Vibeke versus Karen versus andre review-roller

**Vibeke**: Dokumentations-reviewer & commit-gate.
- Fokus: Validerer at indhold matcher rapport, ingen secrets, korrekt placering/navngivning/frontmatter, konvention-compliance.
- Autoritet: Godkender/afviser *ethvert* Camilla-commit før `git commit`.
- Scope: Read-only, pure gate, alltid-på.

**Karen** (Security Architect): Kode-kvalitet + threat-modeling.
- Fokus: Design-review af HSM-integration, secret-handling, API-sikkerhed.
- Autoritet: Konsultativ (ikke gatekeeper) — Karen er project-rolle.
- Scope: Teknisk arkitektur, ikke dokumentation eller git-operationer.

**Trine** (Compliance Auditor): Post-hoc audit-evidens.
- Fokus: Indsamler evidens om at kontroller arbejder (fx "vi skriver kun til korrekt mappe" via git-log).
- Autoritet: Rapporterer til FGD/SOC2-revisor uafhængigt.
- Scope: Ikke daglig gatekeeper.

**Mads** (CISO): Governance policy + eskalering.
- Fokus: SOD-principper, threat-models, compliance-policies.
- Autoritet: Policy-ejer; kan justere Vibeke/Camilla mandat.
- Scope: Systemisk, ikke per-commit.

**Trin**: Alle tre reviewer-typer arbejder sammen uden konflikt:
1. Vibeke godkender commit (konvention + secrets-scan).
2. Camilla committer.
3. Trine læser git-log post-hoc + rapporterer at gate fungerte (SOC 2-evidens).
4. Karen sparrer på arkitektur parallelt (ikke blokker Vibeke).
5. Mads ejer overordnet policy.

---

## Vibekes design — kernepunkter

### Read-only by-design

**Problem**: Hvem reviewer reviewer'en?  
**Løsning**: Vibeke har ZERO skrive-adgang (ingen Edit/Write). Hun læser, verificerer, siger "godkendt/afvist". Hendes output er **Agent-besked**, ikke filer i repo.

Spørgsmålet "hvem reviewer reviewer'en" forsvinder — hun har intet eget arbejde at reviewe.

### Bash whitelist

Bash tilladt kun for read-only git-ops: `git status`, `git diff`, `git diff --cached`, `git log`, `git show`, `ls`, `cat`.

Forbudt: `git commit`, `git push`, `git add`, `git reset`, `git rm`, `rm`, `mv`, `chmod`, etc.

**Teknisk håndhævelse**: Camilla implementerer deny-liste via `settings.local.json` på rolle-basis. Prompt-baseret "belove mig ikke at committe" er ikke tilstrækkelig til SOD (Camilla havde instruks-overtrædelser; vi håndhæver nu teknisk).

### Agent-begrænset til Stefan + Mads

Vibeke kalder IKKE Camilla (reviewer-bias: Camilla kan "forsvare" sin commit eller bede smårettelser før godkendelse).  
Vibeke kalder IKKE Trine (audit-bias: Trine skal auditer Vibekes output uafhængigt).  
Vibeke kalder kun Stefan (eskalering) og Mads (security/mønster-problemer).

### Always-on, ingen backup

Hvis Vibeke ikke svarede, pauses ALLE commits — der er **ingen backup-reviewer**. Mads er kun midlertidig bro under initial-setup (1-2 dage).

Denne design er *renere* end "hvem er backup-reviewer?" (svaret er: der er ingen, og det tvinger organisationen til at havde Vibeke tilgængelig eller eskalere til FGD).

---

## Practical: FGD-godkendelses-trin

**Før Vibeke kan aktiveres, skal FGD**:

1. **Godkende rolle-design** — accepterer du at Vibeke er read-only gatekeeper uden eget arbejde?
2. **Godkende policy**:
   - Camilla må IKKE committe uden Vibeke-godkendelse? JA/NEJ?
   - Pause commits helt hvis Vibeke ikke svarede? JA/NEJ?
   - Trine er UDELUKKET fra daglig commit-gate? JA/NEJ?
3. **Godkende implementation-krav**:
   - Camilla implementerer Bash-whitelist via `settings.local.json`? Eller prompt-baseret OK indtil fase 2?
   - Test-commit kørt under observation før live? JA/NEJ?

Når FGD godkender:
1. Camilla committer `Min Inbox/_crosscutting/2026-05-05-vibeke-commit-reviewer-mandat.md` fra staging.
2. Camilla lægger agenten ud (Vibeke).
3. Stefan arrangerer første test-commit (Camilla → Vibeke → Stefan-resultat-not → Mads-debrief).
4. Vibeke er live.

---

## Næste skridt

1. **FGD godkender mandat-udkastet** — svar på de 3 policy-spørgsmål ovenfor.
2. **Camilla implementerer**:
   - Flyt mandat til Team Inbox/projects/vidensapp/ (efter godkendelse).
   - Commit + push.
   - Implementér Bash-whitelist i `settings.local.json` (hvis FGD accepterer teknisk håndhævelse nu).
3. **Test-commit** under observation (Stefan/Mads tilstede).
4. **Live**: Vibeke er gatekeeper for alle fremtide Camilla-commits.

---

**Leveret af**: Laila (HR)  
**Baseret på**: Mads' SOD-vurdering  
**Arkiv-note**: Lailas tidligere reviewer-design-udkast (`arkiv/Min Inbox/2026-05-05-laila-reviewer-rolle-design-til-godkendelse.md`) arkiveres som superseded.
