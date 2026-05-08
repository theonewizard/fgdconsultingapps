# Governance

Governance-dokumenter for FGD-team's knowlagecentral-projekt.

## Ejerskab

- **Mads (CISO)** ejer governance-policies, compliance-roadmaps, SOD-policies, pen-test-planer, audit-trail-design.
- **Poul** lægger design-dokumenter (system-arkitektur, behovs-analyser) der har governance-impact.

## Struktur (forventet)

```
governance/
├── policies/          ← Mads: udviklings-governance, code-review-flow, deploy-gates
├── compliance/        ← Mads: GDPR, ISO 27001, SOC 2, FIPS 140-3 roadmaps
├── threat-models/     ← Mads + Poul: system-niveau threat-models
├── pen-test/          ← Mads: vendor-shortliste, RoE-templates, testing-planer
├── runbooks/          ← Mads: incident-respons, breach-procedure (GDPR 72-timer)
├── dpa/               ← Mads: customer-DPA-templates, leverandør-DPA-tracker
└── design-docs/       ← Poul: system-design der har governance-impact
```

Filer indenfor governance/ er **levende dokumenter** — opdateres kontinuerligt af Mads/Poul. Camilla håndterer git og struktur.
