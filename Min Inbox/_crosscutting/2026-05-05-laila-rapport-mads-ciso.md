---
dato: 2026-05-05
forfatter: Laila
emne: Mads — CISO (rapport til FGD)
status: udkast til godkendelse
---

# Rapport: Mads — CISO (permanent rolle)

## TL;DR

Jeg har designet **Mads som FGD-teamets permanent CISO** (5. faste rolle ved siden af Stefan, Laila, Poul, Camilla). Mads er ikke en project-rolle som Karen eller Bjarne — han er governance-niveau: sætter sikkerheds-holdet, definerer dev-pipelines + test-krav, sikrer FIPS 140-3/SOC 2/GDPR/ISO 27001 er integreret fra dag 1.

Rol-mandat ligger i `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md` klar til dit godkendelse.

---

## Hvorfor permanent rolle (ikke project)?

| Aspekt | Karen (project) | Bjarne (project) | Mads (permanent) |
|---|---|---|---|
| **Ansvar** | Threat-model + code-review pr. feature | HSM-operatør rutine | Compliance-governance for hele teamet |
| **Periode** | Fase 0–2 (5–7 uger) | Fase 0–2 + driftsfase quarterly | Alle faser → evigt |
| **Scope** | En rolle (security architect) | En rolle (HSM-ops) | Ledelse af security-holdet + relationer til Poul/Stefan |
| **Skalering** | Uændret (Karen's kompetencer) | Uændret (HSM-ops) | Vokser (flere security-roller når systemet skaleres) |
| **Rapport til** | Stefan | Stefan/Karen | Stefan; overordnet over Karen/Bjarne |

**Kort**: Karen + Bjarne har definitiv slutdato (fase 2 launch). Mads skal være der forever fordi compliance ikke stopper efter launch — den skalerer.

---

## Mads' relationer

```
Stefan ←→ Mads (governance-definition together)
  ↓
Poul ←→ Mads (sparring i analyse-fasen; Poul research, Mads compliance-frame)
  ↓
Laila ←→ Mads (co-design af security-roller)
  ↓
Karen ↑ (Mads er Karens overordnede; Karen rapporterer til Mads)
Bjarne ↑ (Mads er Bjarnes overordnede; destruktive ops kræver Mads-godkendelse)
  ↓
Henrik/Sofie/Anders (indirekte via Karen; Mads kan eskalere)
  ↓
Camilla ← Mads (Mads skriver governance; Camilla vedligeholder git/memory)
```

---

## Model-valg: **Sonnet**

| Model | Rationale |
|---|---|
| **Haiku** | For lille — governance er ræsonnement over multi-dimensionale compliance-frameworks, trusselmodeller, organisatoriske relationer |
| **Sonnet** ✓ | Sweet spot — threat-modeling, policy-skrivning, role-definition kræver dyb ræsonnement, men ikke Opus-niveau deep-research (det ejer Poul allerede) |
| **Opus** | Over-provision — ville duplikere Pouls research-niveau; Mads skal kunne kalde Poul når dybde kræves |

Sonnet kan håndtere 80% af governance-arbejde; når compliance-dybde kræves, eskalerer Mads til Poul.

---

## Tools-valg

| Tool | Rationale |
|---|---|
| **Read, Glob, Grep** | Læs hele systemet, analyser kode-struktur, sikkerhedsscanner-output |
| **Write, Edit** | Skriver governance-dokumenter, runbooks, DPA-tracker i `Team/Mads/`; IKKE prodkode |
| **Bash** | Read-only ops kun (`git log`, `gitleaks`, `semgrep`, `trivy`, `npm audit`) — INGEN skrive/delete/commit |
| **Agent** | Kalder Karen, Bjarne, Poul, Laila, Camilla + Stefan |
| **TodoWrite** | Tracke governance-tasks, compliance-tjecklists |

Hard rule: Mads skriver ALDRIG til `.claude/agents/`, `apps/`, `packages/`, eller prod-stier. Kun `Team/Mads/` + governance-mappe.

---

## Konsekvenser for dokumentation

### 1. Projekt-CLAUDE.md (denne mappes instruks)

Skal opdateres med permanent-roller-tabel:

```
| Navn | Rolle | Hvornår du kalder | Model |
|---|---|---|---|
| Stefan | Organisator | altid først | — |
| Laila | AI HR-leder | ny rolle skal designes | haiku |
| Poul | Senior analytiker | behovs-analyse, research | opus |
| Camilla | Senior bibliotekar/git/DB | dokumentation, version-styring | opus |
| **Mads** | **CISO** | **governance-definition, compliance-vurdering, security-hold-sammensætning** | **sonnet** |
```

### 2. Team/_konventioner.md

Skal opdateres:
- Navne-pulje: Mads er allerede der (linjen med "Mads" i listen)
- Fast-roller-tabel: tilføj Mads-røllen
- Ansvar-tabel: Mads skriver til `Team/Mads/`, ikke direkte til `.claude/agents/` (kun Camilla efter godkendelse)

### 3. Memory (hvis FGD vurderer relevant)

Camilla kan oprette `mads-ciso-rolle-2026-05-05.md` i memory/ med:
- Permanent rolle nr. 5
- Overordnet over Karen + Bjarne
- FIPS 140-3 / SOC 2 / GDPR / ISO 27001 ansvar
- Henvis til `.claude/agents/mads-ciso.md` som kilde-of-truth

---

## Praktisk godkendelse-flow

1. **Du godkender** (eller justerer) Mads-rollen i `Team Inbox/_crosscutting/2026-05-05-mads-ciso-mandat.md`
2. **Du skriver tilbage**: "Godkendt" eller "Ønsker ændring i punkt X"
3. **Jeg justerer** hvis nødvendigt og beder Camilla committe
4. **Camilla**:
   - Beder Mads til `.claude/agents/mads-ciso.md` + opretter arbejdsmappe `Team/Mads/`
   - Opdaterer `CLAUDE.md` + `Team/_konventioner.md`
   - Committer alt sammen
5. **Stefan kalder Mads** for Fase 0 compliance-oversigt

---

## Åbne valg (som kræver din vejledning)

1. **Mads' arbejdsmappe-struktur**: skal den være `Team/Mads/`, eller ønsker du dedikeret `governance/`-mappe i repo-rod?
2. **Audit-trail-ansvar**: skal Mads skrive audit-trail-struktur, eller skal det ligge hos Camilla+Henrik?
3. **Customer DPA-template**: skal Mads designe, eller ønsker du at det kommer fra en ekstern template (fx fra advokat)?

Hvis du ikke har modifikationer: "Godkendt, Camilla kan committe" er nok.

---

## Næste skridt

- **Du**: Godkend Mads eller bed om ændringer
- **Camilla**: Committer efter din godkendelse
- **Stefan**: Kalder Mads for fase 0, når du er klar
