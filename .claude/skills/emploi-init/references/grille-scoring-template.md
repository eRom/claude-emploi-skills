# Grille de Scoring — Cibles Emploi

> Score sur 12 points (4 axes x 0-3). Utilisee par tous les agents.

---

## Axes de scoring

| Axe | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| **Competences techniques** | Aucun lien avec le profil | Quelques competences transferables | Bonnes correspondances | Match quasi-parfait |
| **Localisation** | >100km, pas remote | Remote partiel, >50km | {{LOCALISATION_IDEALE}} ou full remote | {{LOCALISATION_IDEALE}} centre |
| **Responsabilite** | Executant junior | Chef de projet standard | Architecte / resp. technique | Direction technique / CTO |
| **{{AXE_SECTORIEL}}** | Aucun lien | {{AXE_SECTORIEL}} peripherique | {{AXE_SECTORIEL}} directe | {{AXE_SECTORIEL}} — coeur de metier |

---

## Verdicts

| Score | Verdict | Action |
|-------|---------|--------|
| **10-12** | Cible prioritaire | `/new-cible` immediat |
| **7-9** | Cible prometteuse | Approfondir — candidat `/new-cible` |
| **4-6** | Cible secondaire | Surveiller |
| **0-3** | Hors cible | Ignorer |

---

## Regle anti-inflation

**Score 3 = justifie par un fait verifiable.**

- Un score de 3 sur n'importe quel axe doit etre appuye par une preuve concrete (URL, offre, donnee publique).
- En cas de doute → attribuer le niveau inferieur.
- Ne jamais arrondir vers le haut par optimisme.

---

## Format de scoring (par cible)

```markdown
### [Nom de la cible]

| Axe | Score | Justification |
|-----|-------|---------------|
| Competences techniques | X/3 | [justification] |
| Localisation | X/3 | [justification] |
| Responsabilite | X/3 | [justification] |
| {{AXE_SECTORIEL}} | X/3 | [justification] |
| **Total** | **X/12** | |

**Verdict :** [action recommandee]
```
