# Skill-assessment — Laila HR-leder

**Dato:** 2026-05-09  
**Emne:** Vurdering af nye installerede skills for kernekompetencer

---

## Nuværende kerneopgaver

Laila designerer og ansætter nye AI-roller. Det kræver:
1. Forståelse af eksisterende rolle-landscape (overlap-tjek)
2. Struktureret rolle-definition (frontmatter + body efter skabelon)
3. Co-design af compliance-roller med Mads (governance, SOD)
4. Organiserings-sparring (team-struktur, hand-off-aftaler)

---

## Vurdering af nye skills

| Skill | Relevant for Laila? | Vurdering |
|---|---|---|
| `image` | **JA — høj værdi** | Kan nu analysere org-diagrammer som INPUT til rolle-design. Fx hvis FGD sender "her er vores team-struktur — vi mangler denne rolle", kan Laila parse diagrammet visuelt og identificere gaps præcist. Reducerer fej-tolkning af tekstuelle beskrivelser. |
| `deep-research` | **MÅSKE — lav værdi for Laila** | Laila udfører IKKE selv research. Når behov er uklart, delegerer Laila til Poul (der har `opus` + bør få `deep-research`). Ville være duplikering af Pouls rolle. **Anbefaling:** giv det til Poul i stedet. |
| `excalidraw-diagrams` | **JA — medium værdi** | Kan producere org-diagrammer som OUTPUT ved nye rolle-designs. Fx når Laila ansætter 3 nye roller, kan hun tegne opdateret org-diagram med roles+hand-offs direkte. Brugbart for FGD-godkendelse. Ikke kritisk (Stefan kan også tegne), men accelererer dokumentation. |
| `context7` | **MARGINAL** | Laila har ikke standard-tools-versions-tværs her. Ikke relevant for rolle-design. Ville være tool-bloat. |
| `yaml` | **NEJ** | Rolle-mandater er `.md` filer med YAML frontmatter. Laila læser/skriver det manuelt i dag — fint. YAML-skill ville være overkill for frontmatter. |
| `coding-standards` | **NEJ** | Laila designer rolle-mandater, ikke kode. Ikke relevant. |

---

## Anbefaling

**Tilføj til Lailas `tools`-liste:**

1. **`image`** — høj værdi for overlap-tjek og visuel analyse af organisation
2. **`excalidraw-diagrams`** — medium værdi for output-dokumentation

**Giv til Poul i stedet:**

3. **`deep-research`** — Poul er analytiker (opus), Laila delegerer research til Poul. Duplikering af kompetence hvis Laila får det.

**Afvis:**

4. `context7`, `yaml`, `coding-standards` — ikke relevant for rolle-design

---

## Implementering

Frontmatter-ændring til `laila-hr.md` (kræver FGD-godkendelse per governance):

```yaml
tools: Read, Glob, Grep, Write, TodoWrite, Agent, image, excalidraw-diagrams
```

**Bemærk:** Dette er en `tools`-ændring, som kræver Mads + Dorthe-vurdering per §3.3 i Lailas mandat.
