---
name: emploi-veille
description: >
  Lance une veille emploi complete avec 4 agents en parallele : scanner offres,
  scanner acteurs, analyse marche, puis synthese consolidee.
  Utilise cette skill des que l'utilisateur dit "lance une veille", "veille emploi",
  "/veille", "quoi de neuf sur le marche", ou "scan le marche".
  La skill orchestre tout : creation de la team, lancement des agents, consolidation, rapport final.
---

# /emploi-veille — Veille Emploi Multi-Agents

## Contexte

Lire le profil candidat dans `.claude/skills/emploi-veille/references/profil-candidat.md`.
La veille decouvre des opportunites et les score. Elle ne produit ni CV, ni LM, ni pitch — c'est le role de `/emploi-cible`.
Les prompts des agents vivent dans `.claude/skills/emploi-veille/references/prompt-agent-*.md`. Les lire dynamiquement, jamais les dupliquer.

---

## Etape 0 — Initialisation

1. Determiner `$DATE` = date du jour au format `YYYY-MM-DD`
2. Lister dynamiquement `candidatures/` pour construire la liste **DEJA CIBLE** (noms des sous-dossiers existants)
3. Lire les fichiers de reference :
   - `.claude/skills/emploi-veille/references/profil-candidat.md` — bloc profil candidat
   - `.claude/skills/emploi-veille/references/grille-scoring.md` — grille de scoring 4 axes (/12)
4. Creer le dossier de sortie : `cibles/$DATE/`
5. Resoudre les variables template :
   - `{{DATE}}` = `$DATE`
   - `{{ZONE_GEO}}` = [configure par `/emploi-init`]
   - `{{RAYON_KM}}` = [configure par `/emploi-init`]
   - `{{POSTES_VISES}}` = [configure par `/emploi-init`]
   - `{{SECTEURS}}` = [configure par `/emploi-init`]
   - `{{FRESHNESS_DAYS}}` = `60`
   - `{{AXE_SECTORIEL}}` = [configure par `/emploi-init`]

---

## Etape 1 — Creation de la Team

1. `TeamCreate` avec le nom `"veille-emploi"`
2. `TaskCreate` x 4 :
   - **Task 1** : Scanner Offres — `activeForm: "Scanning les offres d'emploi"`
   - **Task 2** : Scanner Acteurs — `activeForm: "Scanning l'ecosysteme acteurs"`
   - **Task 3** : Contexte Marche — `activeForm: "Analysant le marche"`
   - **Task 4** : Synthese — `activeForm: "Consolidant les resultats"` — **bloquee par Task 1, 2, 3**

---

## Etape 2 — Lancement des 3 agents en parallele

Lire les prompts depuis les fichiers source :
- `.claude/skills/emploi-veille/references/prompt-agent-offres.md`
- `.claude/skills/emploi-veille/references/prompt-agent-acteurs.md`
- `.claude/skills/emploi-veille/references/prompt-agent-marche.md`

Dans **un seul message**, lancer 3 appels `Agent` en parallele :

### Agent 1 — scanner-offres
- `subagent_type: "general-purpose"`
- `team_name: "veille-emploi"`
- `name: "scanner-offres"`
- Prompt = contenu de `prompt-agent-offres.md` avec variables resolues + profil-candidat + grille-scoring + liste DEJA CIBLE
- Output attendu : `cibles/$DATE/veille-offres-$DATE.md`

### Agent 2 — scanner-acteurs
- `subagent_type: "general-purpose"`
- `team_name: "veille-emploi"`
- `name: "scanner-acteurs"`
- Prompt = contenu de `prompt-agent-acteurs.md` avec variables resolues + profil-candidat + grille-scoring + liste DEJA CIBLE
- Output attendu : `cibles/$DATE/veille-acteurs-$DATE.md`

### Agent 3 — analyste-marche
- `subagent_type: "general-purpose"`
- `team_name: "veille-emploi"`
- `name: "analyste-marche"`
- Prompt = contenu de `prompt-agent-marche.md` avec variables resolues + profil-candidat + grille-scoring + liste DEJA CIBLE
- Output attendu : `cibles/$DATE/veille-marche-$DATE.md`

---

## Etape 3 — Attente et resilience

- Attendre la completion des 3 agents
- **Si un agent echoue** : noter l'echec dans le rapport, continuer avec les resultats des autres
- Ne pas bloquer la synthese si 2 agents sur 3 ont livre

---

## Etape 4 — Agent Synthese

Lire le prompt depuis `.claude/skills/emploi-veille/references/prompt-agent-synthese.md`.

Lancer l'agent synthese :
- `subagent_type: "general-purpose"`
- `team_name: "veille-emploi"`
- `name: "synthese"`
- Prompt = contenu de `prompt-agent-synthese.md` avec :
  - Les 3 fichiers de sortie des agents (ceux qui existent)
  - Le profil-candidat
  - La grille-scoring
  - La liste DEJA CIBLE
- Output attendu : `cibles/$DATE/rapport-veille-$DATE.md`

---

## Etape 5 — Shutdown de la Team

1. `SendMessage` type `shutdown_request` vers chaque agent (scanner-offres, scanner-acteurs, analyste-marche, synthese)
2. `TeamDelete` une fois tous les agents arretes

---

## Etape 6 — Presentation des resultats

Lire `cibles/$DATE/rapport-veille-$DATE.md` et presenter a l'utilisateur :

1. **Tableau** des cibles triees par score decroissant (score /12)
2. **Recommandation** : pour chaque cible avec score >= 7/12, recommander `/emploi-cible [slug]`
3. **Demander a l'utilisateur** lesquelles il veut traiter

Format de presentation :
```
| # | Cible | Score | Secteur | Pourquoi |
|---|-------|-------|---------|----------|
| 1 | xxx   | 9/12  | ...     | ...      |

Recommandation : lance `/emploi-cible xxx` pour les cibles >= 7/12.
Lesquelles on attaque ?
```

---

## Regles generales

- **Prompts dynamiques** : toujours lire `.claude/skills/emploi-veille/references/prompt-agent-*.md` au runtime, jamais copier leur contenu dans cette skill
- **Liste DEJA CIBLE** : toujours `ls candidatures/` au runtime, jamais de liste statique
- **Resilience** : si 1 agent plante, on continue avec les 2 autres + la synthese
- **Pas de CV/LM/pitch** : la veille decouvre, `/emploi-cible` traite
- **Dossier de sortie** : `cibles/$DATE/` — 4 fichiers attendus
- **Ton** : efficace, pas de blabla. Resultats, scores, recommandations.
