---
triggers:
  - /init
  - initialise
  - setup
  - configure mon espace
  - premier lancement
description: "Initialise l'espace de travail complet pour la recherche d'emploi : arbre de dossiers, cache, profil candidat, grille de scoring."
---

# /init — Setup complet de l'espace de travail

> **Objectif :** Bootstrapper toute la structure de travail pour un nouvel utilisateur.
> **Idempotent :** Si la structure existe deja, proposer reinitialisation ou annulation.

---

## Etape 0 — Detection

Verifier si la structure existe deja :

```
Verifier l'existence de :
- cache/
- cibles/
- candidatures/
- organisation.md
- .claude/skills/emploi-veille/references/profil-candidat.md
- .claude/skills/emploi-veille/references/grille-scoring.md
```

**Si au moins 3 de ces elements existent** → Demander a l'utilisateur :
- "Reinitialiser tout" (ecrase)
- "Annuler" (stop)
- "Completer" (ne creer que ce qui manque)

**Sinon** → Continuer.

---

## Etape 1 — Collecte d'informations

La collecte se fait en 4 tours. Utiliser le bon outil selon le type de donnee :
- **Texte libre** → question conversationnelle directe (pas de `AskUserQuestion`)
- **Choix ferme** → `AskUserQuestion` avec options predefinies
- **Contenu riche** → `AskUserQuestion` avec option "Importer un fichier" (CV, LinkedIn, etc.)

### Tour 1 — Identite (conversationnel)

Poser ces questions directement dans le chat, en un seul message. L'utilisateur repond en texte libre.

```
Pour configurer ton profil, j'ai besoin de quelques infos :

1. **Nom complet** (ex: "Jean-Baptiste Poquelin")
2. **Age ou date de naissance** (ex: "51 ans" ou "15/01/1622")
3. **Localisation** (ex: "Nantes, France")
4. **Situation actuelle** (ex: "En recherche active", "En poste, en veille", etc.)
```

Variables collectees : `{{NOM}}`, `{{AGE}}`, `{{LOCALISATION}}`, `{{SITUATION}}`

### Tour 2 — Parcours et competences (AskUserQuestion)

Proposer le choix de la source via `AskUserQuestion` :

**Question :** "Comment veux-tu renseigner ton parcours et tes competences ?"

| Option | Description |
|--------|-------------|
| **Importer un fichier** | Fournir un CV (md, pdf, docx), un export LinkedIn, ou tout document contenant le parcours. L'agent parse et extrait les infos. |
| **Dicter dans le chat** | Decrire le parcours, les competences et les certifications en texte libre. L'agent structure en tableau et listes. |

**Si fichier** → demander le chemin, lire le fichier, extraire :
- `{{PARCOURS_TABLE}}` : formater en tableau markdown `| Periode | Poste | Entreprise |`
- `{{COMPETENCES}}` : formater en liste a puces par domaine
- `{{CERTIFICATIONS}}` : formater en liste a puces (ou "Aucune")

**Si texte libre** → poser la question dans le chat, structurer la reponse.

Presenter le resultat a l'utilisateur pour validation avant de continuer.

### Tour 3 — Objectifs (AskUserQuestion + conversationnel)

**3a. Secteur cible** via `AskUserQuestion` :

**Question :** "Quel secteur privilegies-tu ?"

| Option | Description |
|--------|-------------|
| Sante / e-sante | Hopitaux, MedTech, editeurs SIH, GRADeS |
| Finance / Banque | FinTech, banques, assurances |
| Industrie / IoT | Industrie 4.0, IoT, systemes embarques |
| Generaliste / Multi-secteurs | Pas de preference sectorielle |

Variable : `{{AXE_SECTORIEL}}` (l'utilisateur peut aussi repondre "Other" pour un secteur custom)

**3b. Postes et atouts** (conversationnel) :

```
Maintenant, quelques precisions :

1. **Postes vises** — quels intitules de poste cherches-tu ?
   (ex: "Chef de Projet IT", "Architecte SI", "CTO")
2. **Atouts differenciants** — qu'est-ce qui te distingue des autres candidats ?
   (ex: projets perso, double competence, certifications rares, experience unique)
```

Variables : `{{POSTES_VISES}}`, `{{ATOUTS}}`

### Tour 4 — Zone geographique (conversationnel)

Poser en un seul message :

```
Derniere etape — ta zone de recherche :

1. **Zone ideale** (ex: "Nantes centre / metropole")
2. **Zone acceptable** (ex: "Loire-Atlantique, Pays de la Loire, full remote")
3. **Rayon max en km** (ex: "50")
```

Variables : `{{ZONE_GEO_IDEAL}}`, `{{ZONE_GEO_ACCEPTABLE}}`, `{{RAYON_KM}}`
Variable derivee : `{{LOCALISATION_IDEALE}}` = ville principale extraite de `{{ZONE_GEO_IDEAL}}`

---

## Etape 2 — Arbre de dossiers

Creer les dossiers suivants (mkdir -p) :

```
cache/companies/
cache/job-boards/
cibles/
candidatures/
.claude/skills/emploi-veille/references/
.claude/skills/emploi-cible/references/
```

---

## Etape 3 — Generation des fichiers

Pour chaque template dans `.claude/skills/emploi-init/references/`, lire le contenu, remplacer les `{{PLACEHOLDERS}}` par les valeurs collectees, et ecrire le fichier de sortie :

| Template source | Fichier de sortie |
|----------------|-------------------|
| `references/cache-readme-template.md` | `cache/README.md` |
| `references/job-boards-template.json` | `cache/job-boards/registry.json` |
| `references/cibles-readme-template.md` | `cibles/README.md` |
| `references/organisation-template.md` | `organisation.md` |
| `references/profil-candidat-template.md` | `.claude/skills/emploi-veille/references/profil-candidat.md` |
| `references/grille-scoring-template.md` | `.claude/skills/emploi-veille/references/grille-scoring.md` |

### Variables derivees

Calculer avant la generation :

| Variable derivee | Source | Logique |
|-----------------|--------|---------|
| `{{NOM_PRENOM}}` | `{{NOM}}` | Remplacer espaces par `_` (ex: "Jean_Dupont") |
| `{{LOCALISATION_IDEALE}}` | `{{ZONE_GEO_IDEAL}}` | Extraire la ville principale |

### Fichier `job-boards-template.json`

Apres copie, mettre a jour le champ `last_updated` avec la date du jour (`YYYY-MM-DD`).

---

## Etape 4 — Variables veille

Si `.claude/skills/emploi-veille/SKILL.md` existe, mettre a jour la section "Variables template" avec les valeurs personnalisees :

| Variable veille | Source |
|----------------|--------|
| `{{ZONE_GEO}}` | `{{ZONE_GEO_IDEAL}}`, `{{ZONE_GEO_ACCEPTABLE}}` |
| `{{RAYON_KM}}` | `{{RAYON_KM}}` |
| `{{POSTES_VISES}}` | `{{POSTES_VISES}}` (format pipe-separated) |
| `{{SECTEURS}}` | Derive de `{{AXE_SECTORIEL}}` + termes connexes |
| `{{AXE_SECTORIEL}}` | `{{AXE_SECTORIEL}}` (ex: "Secteur sante") |

**Si le fichier n'existe pas** → ignorer cette etape (la skill veille sera configuree manuellement).

> **Note :** Les skills `/veille`, `/new-cible` et leurs agent prompts sont generiques — ils lisent `profil-candidat.md` et `grille-scoring.md` au runtime. Seules les variables par defaut de `veille/SKILL.md` doivent etre mises a jour ici.

---

## Etape 5 — Recapitulatif

Afficher un recapitulatif a l'utilisateur :

```markdown
## Espace de travail initialise

### Structure creee
- cache/ (companies, job-boards, README)
- cibles/ (README)
- candidatures/
- organisation.md

### Profil configure
- Profil candidat : .claude/skills/emploi-veille/references/profil-candidat.md
- Grille scoring : .claude/skills/emploi-veille/references/grille-scoring.md
- Job boards : cache/job-boards/registry.json (8 sources FR)

### Prochaine etape
Lance `/veille` pour decouvrir tes premieres opportunites.
```

---

## Placeholders — Reference complete

| Placeholder | Type | Utilise dans |
|------------|------|-------------|
| `{{NOM}}` | string | profil-candidat |
| `{{AGE}}` | string | profil-candidat |
| `{{LOCALISATION}}` | string | profil-candidat |
| `{{SITUATION}}` | string | profil-candidat |
| `{{PARCOURS_TABLE}}` | markdown table | profil-candidat |
| `{{COMPETENCES}}` | markdown list | profil-candidat |
| `{{CERTIFICATIONS}}` | markdown list | profil-candidat |
| `{{POSTES_VISES}}` | markdown list | profil-candidat |
| `{{ATOUTS}}` | markdown list | profil-candidat |
| `{{ZONE_GEO_IDEAL}}` | string | profil-candidat |
| `{{ZONE_GEO_ACCEPTABLE}}` | string | profil-candidat |
| `{{RAYON_KM}}` | number | profil-candidat, grille-scoring |
| `{{NOM_PRENOM}}` | string (derive) | organisation |
| `{{LOCALISATION_IDEALE}}` | string (derive) | grille-scoring |
| `{{AXE_SECTORIEL}}` | string | grille-scoring, veille defaults |
