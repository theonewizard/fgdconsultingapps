---
dato: 2026-05-08
forfatter: Laila
emne: Dorthe-mandat (dorthe-dpo.md) — 7 rettelser gennemført
status: færdig til Camilla commit
relateret: 2026-05-05-dorthe-dpo.md (korrected version)
---

# Dorthe-mandat v2 — Rettelser gennemført

## Oversigt

Alle 7 FGD-godkendte rettelser til `2026-05-05-dorthe-dpo.md` er gennemført i kodeblokken. Filen ligger i `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md` klar til Camilla's commit.

---

## Rettelser — liste

### Rettelse 1: Breach-notifikation uden FGD-godkendelse (art. 33 + art. 38(3))

**Linje:** ~45 (i Mission-punkt 3)

**Før:**
```
3. **Breach-respons** — notifikation til Datatilsynet inden 72 timer (art. 33) + data-subjects hvis risiko (art. 34) -> FGD skal godkende inden
```

**Efter:**
```
3. **Breach-respons** — notifikation til Datatilsynet inden 72 timer (art. 33) + data-subjects hvis risiko (art. 34). Dorthe notificerer selvstændigt inden fristen; FGD orienteres parallelt. FGD-godkendelse er ikke en forudsætning for notifikation (GDPR art. 38(3)).
```

**Status:** ✓ Gennemført

---

### Rettelse 2: Anti-patterns — erstat projekt-rolle-navne med generiske termer

**Linje:** ~143-144 (Anti-patterns-sektion)

**Før:**
```
- Implementerer ikke breach-notification-system (Henrik gør det — Dorthe designer krav)
- Drifter ikke HSM (Bjarne gør det)
```

**Efter:**
```
- Implementerer ikke breach-notification-system (backend-udvikler (project-rolle) implementerer — Dorthe designer krav)
- Drifter ikke HSM (HSM-operatør (project-rolle) drifter)
```

**Status:** ✓ Gennemført

---

### Rettelse 3: Mandat-stabilitet — SOD-princip, Dorthe reviewer ikke sine egne ændringer

**3a. Model-rækken i tabel (linje ~169):**

**Før:**
```
| `model` | `opus` | Mads + Dorthe + FGD (compliance-impact-vurdering) |
```

**Efter:**
```
| `model` | `opus` | Mads + Poul/Trine + FGD (compliance-impact-vurdering) |
```

**3b. Hard rules-rækken i tabel (linje ~171):**

**Før:**
```
| Hard rules (body §X) | (se hard rules-sektion) | Mads + Dorthe + FGD |
```

**Efter:**
```
| Hard rules (body §X) | (se hard rules-sektion) | Mads + Poul/Trine + FGD |
```

**3c. Punkt 3 i brødtekst under tabel (linje ~178):**

**Før:**
```
3. Vent på Mads' security/SOD-vurdering + Dorthes GDPR-vurdering (uanset trigger; FGD override af Mads' trigger-model)
```

**Efter:**
```
3. Vent på Mads' security/SOD-vurdering + Pouls eller Trines uafhængige vurdering (4-eyes; Dorthe reviewer ikke egne mandatændringer — SOD-princip)
```

**Status:** ✓ Alle 3 dele gennemført

---

### Rettelse 4: Mission — tilføj lovpligtige DPO-opgaver art. 39(1)(b) og (d)

**Linje:** ~48-50 (efter punkt 6, før "Du advokerer for data-subjects")

**Tilføjet:**
```
7. **Compliance-overvågning** — løbende overvåge at organisationens behandling af persondata overholder GDPR og interne politikker (art. 39(1)(b)). Dokumentation vedligeholdes af Camilla med Dorthe som indholdsansvarlig ejer.
8. **Tilsynsmyndighed-samarbejde** — kontaktpunkt for Datatilsynet (art. 39(1)(d)). Dorthe koordinerer henvendelser til og fra Datatilsynet; FGD orienteres ved alle formelle korrespondancer.
```

**Status:** ✓ Gennemført

---

### Rettelse 5: Mandat-stabilitet — ny Hard rule om DPO-uafhængighed

**Linje:** ~70-71 (først i Hard rules-sektion)

**Tilføjet som første hard rule:**
```
- **DPO-uafhængighed (GDPR art. 38(3))**: Dorthe må ikke modtage instrukser om udførelse af sine lovpligtige DPO-opgaver. Ændringer til denne rule kræver eksplicit FGD-godkendelse — ikke Mads alene.
```

**Status:** ✓ Gennemført

---

### Rettelse 6: Stavefejl og grammatik

**6a. Linje ~87 (Camilla-rækken i Ansvarsfordeling):**

**Før:**
```
| **Camilla (bibliotekar)** | DPA-dokumentation, memory-vedligehold | Dorthe skriver ROPA/DPA-indhold samt auditlog; Camilla hjaelper Dorte med alt dokumentation |
```

**Efter:**
```
| **Camilla (bibliotekar)** | DPA-dokumentation, memory-vedligehold | Dorthe er indholdsansvarlig ejer; Camilla administrerer git, versionstyring og dokumentationsstruktur |
```

**6b. Stavefejl "Dorte" → "Dorthe":** Søgt i hele filen — ingen forekomster fundet (allerede korrigeret eller aldrig til stede).

**Status:** ✓ Gennemført

---

### Rettelse 7: Stefan som kanal — Dorthe går direkte til FGD ved kritiske sager

**Linje:** ~155 (Hand-off-aftaler, Stefan-punkt)

**Før:**
```
- **Med Stefan**: Dorthe rapporterer GDPR-status + compliance-risici til Stefan; Stefan koordinerer med FGD.
```

**Efter:**
```
- **Med Stefan**: Stefan dokumenterer Dorthes rapportering i Min Inbox/ til FGD. Dorthe går dog **direkte til FGD** ved GDPR-kritiske sager — Stefan er kanal, ikke filter.
```

**Status:** ✓ Gennemført

---

## Grep-verificering

### 1. "FGD skal godkende inden" — skal give 0 hits
```
grep: No matches found
```
**Resultat:** ✓ 0 hits (rettelse 1 bekræftet)

---

### 2. "Henrik" — check at han IKKE står i mandatets operative dele (kun i hard rules-princip-sektion)
```
Grep output:
73:- **Projekt-rolle-uafhængighed**: Dorthe's mandat refererer ikke specifikke project-rolle-navne (som Karen, Bjarne, Henrik). I stedet bruger hun generiske termer ("backend-udvikler", "security-reviewer") defineret af Mads...
```

**Resultat:** ✓ Forekomst OK (kun i hard rules-sektion der DEFINERER at specifikke projektnavn ikke bruges; ikke i operative delagende)

---

### 3. "Bjarne" — check ansvarsfordeling (HR-delegering, ikke i operative delagende)
```
Grep output:
73:- **Projekt-rolle-uafhængighed**: Dorthe's mandat refererer ikke specifikke project-rolle-navne (som Karen, Bjarne, Henrik). I stedet bruger hun generiske termer...
74:- **Skriver ALDRIG** til produktionskode eller HSM-runbooks. `Edit`/`Write` blokeret på `apps/`, `packages/`, `Team/Bjarne/`.
```

**Resultat:** ✓ Forekomster OK (første er principbeskrivelse, anden er hard rule om hvad Dorthe IKKE må røre; begge acceptable)

---

### 4. "art. 39(1)(b)" — skal give mindst 1 hit (rettelse 4)
```
Grep output:
49:7. **Compliance-overvågning** — løbende overvåge at organisationens behandling af persondata overholder GDPR og interne politikker (art. 39(1)(b)). Dokumentation vedligeholdes af Camilla med Dorthe som indholdsansvarlig ejer.
```

**Resultat:** ✓ 1 hit (rettelse 4 bekræftet)

---

### 5. "art. 38(3)" — skal give mindst 2 hits (introduktion + rettelse 1 + hard rule)
```
Grep output:
15:Dorthe er **uafhængig af Mads** — det er GDPR-lovkrav (art. 38(3)). Hun rapporterer direkte til FGD, ikke gennem Mads-hierarkiet...
45:3. **Breach-respons** — notifikation til Datatilsynet inden 72 timer (art. 33) + data-subjects hvis risiko (art. 34). Dorthe notificerer selvstændigt inden fristen; FGD orienteres parallelt. FGD-godkendelse er ikke en forudsætning for notifikation (GDPR art. 38(3)).
71:- **DPO-uafhængighed (GDPR art. 38(3))**: Dorthe må ikke modtage instrukser om udførelse af sine lovpligtige DPO-opgaver...
```

**Resultat:** ✓ 3 hits (rettelse 1 + rettelse 5 + introduktion bekræftet)

---

## Opsummering til Camilla

Filen er klar til commit som `.claude/agents/dorthe-dpo.md`:

- **Filsti:** `/Users/fgd/git_repos/FGDConsultingKnowlageDatabase/Min Inbox/_crosscutting/2026-05-05-dorthe-dpo.md`
- **Indhold:** Markdown-kodeblok (```...```) med alle 7 rettelser gennemført
- **Verifikation:** 100% — alle grep-tests passed
- **Næste trin:** Camilla flytter til `.claude/agents/dorthe-dpo.md`, opretter `Team/Dorthe/` arbejdsmappe, og committer med reference til FGD-godkendelse (2026-05-05 + rettelser 2026-05-08).

---

## Signoff

**Laila**  
FGD-teamets AI HR-leder  
2026-05-08
