# FGD Consulting — AI-team arbejdsmappe

Dette repository er FGD Consultings interne AI-team workspace. Det rummer rolle-mandater, governance-policies, analysedokumenter, inbox-flow og persistent AI-hukommelse til organisationens AI-agenter.

Sproget er **dansk** — se [Team/_konventioner.md](Team/_konventioner.md) for detaljer.

---

## AI-teamet — permanente roller

| Navn | Rolle | Model |
|---|---|---|
| **Stefan** | Organisator / CEO — orkestrerer, delegerer, samler resultater til FGD | opus |
| **Laila** | HR-leder — designer og ansætter nye AI-roller | haiku |
| **Poul** | Senior analytiker — research, behovs-analyse, dyb-dyk | opus |
| **Camilla** | Bibliotekar / git / memory — dokumentation, version-styring, database, persistent hukommelse | opus |
| **Mads** | CISO — governance, compliance (FIPS 140-3 / SOC 2 / GDPR / ISO 27001), threat-modelling | sonnet |
| **Dorthe** | DPO — GDPR-compliance, ROPA, DPIA, breach-respons (art. 38(3)-uafhængig) | opus |
| **Trine** | Compliance-auditor — ISO 27001:2022 + SOC 2 Type II evidens-indsamling og audit | sonnet |
| **Vibeke** | Commit-reviewer — read-only commit-gate; ingen commit til git origin uden Vibeke-godkendelse | sonnet |

Rolle-mandater: [.claude/agents/](.claude/agents/)

---

## Mappestruktur

```
/
├── CLAUDE.md                        ← Stefans instrukser (loades automatisk af Claude Code)
├── README.md                        ← denne fil (vedligeholdes af Camilla før push)
├── .claude/
│   └── agents/                      ← AI-rolle-mandater (governance-låste)
├── Min Inbox/                       ← teamet → FGD: færdige resultater, udkast, statusrapporter
│   └── projects/<projekt>/
├── Team Inbox/                      ← FGD → teamet: opgaver, info, vedhæftede dokumenter
│   └── projects/<projekt>/
├── Team/                            ← rollernes arbejdsmapper
│   ├── Camilla/
│   ├── Dorthe/
│   ├── Laila/
│   ├── Mads/
│   ├── Poul/
│   ├── _konventioner.md             ← fulde konventioner (sprog, filnavne, inbox-flow, eskalering)
│   └── projects/<projekt>/<rolle>/  ← project-roller (når aktive)
├── governance/                      ← Mads: policies, compliance, threat-models, godkendelseslog
│   ├── godkendelseslog.md           ← 4-eyes godkendelseshistorik
│   ├── policies/
│   ├── compliance/
│   └── change-requests/
└── arkiv/                           ← arkiverede dokumenter (spejler hoved-katalog)
```

---

## Workflow

**FGD lægger opgaver i `Team Inbox/projects/<projekt>/`**. Stefan læser ved sessionstart, identificerer den rette rolle og uddelegerer via `Agent`-værktøjet. Resultater samles i `Min Inbox/projects/<projekt>/` med et kort resumé til FGD.

```
Team Inbox/  →  Stefan (orkestrering)  →  rolle-arbejde i Team/<rolle>/  →  Min Inbox/
```

**Commit-gate**: Ingen `git commit` uden Vibekes godkendelse. Ingen `git push` til origin uden separat FGD-godkendelse.

Se [Team/_konventioner.md](Team/_konventioner.md) for fuldt inbox-flow, eskalerings-paths og hand-off-mønstre.

---

## Governance

- **Rolle-mandater er konfigurations-låste** under SOC 2 CC8.1 og ISO 27001 A.8.32. Ændringer til `model`, `tools`, `description` eller hard rules kræver Mads + Dorthe-vurdering og eksplicit FGD-godkendelse.
- **4-Eyes Princip**: kritiske beslutninger kræver ≥ 2 uafhængige reviewers. Log i [governance/godkendelseslog.md](governance/godkendelseslog.md).
- **Self-modificering er forbudt**: ingen rolle må redigere sit eget mandat.
- **Memory** (AI-persistens) ejes af Camilla og er placeret uden for repo'et: `~/.claude/projects/.../memory/`.

---

## Aktivt projekt

**Knowlagecentral** — en Evernote-inspireret videns-app med AI-pipeline, HSM-baseret key management og enterprise compliance-krav (FIPS 140-3, SOC 2 Type II, GDPR).

> Knowlagecentral giver konsulenter og deres kunder et sikkert, AI-beriget noteværktøj — fra råt tekstindhold til struktureret viden, bygget til de brancher der ikke kan gå på kompromis med sikkerhed.

Nøgle-dokumenter:
- Arkitektur: [Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md](Team/Poul/2026-05-05-knowlagecentral-analyse-v3.md)
- Compliance: [governance/compliance/](governance/compliance/)
- Governance-policies: [governance/policies/](governance/policies/)
