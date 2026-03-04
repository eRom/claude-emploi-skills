# claude-emploi-skills

Skills Claude Code pour automatiser une recherche d'emploi structuree : veille multi-agents, arsenal candidature, scoring, et generation de documents.

## Prerequis

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installe et configure

## Installation

```bash
# 1. Cloner le repo
git clone https://github.com/VOTRE_USER/claude-emploi-skills.git

# 2. Copier les skills dans votre projet
cp -r claude-emploi-skills/.claude/skills/emploi-* VOTRE_PROJET/.claude/skills/

# 3. Nettoyage
rm -rf claude-emploi-skills
```

## Premier lancement

```
/emploi-init
```

L'assistant vous pose quelques questions (identite, parcours, objectifs, zone geo) puis genere votre profil candidat, la grille de scoring et toute l'arborescence de travail.

## Workflow

```
/emploi-init  →  /emploi-veille  →  /emploi-cible [nom]  →  /emploi-print [nom]
  (setup)        (decouverte)       (arsenal complet)        (export .docx)
```

## Skills

### `/emploi-init` — Setup de l'espace de travail

Initialise la structure complete : dossiers, cache de recherche, profil candidat, grille de scoring, registre job boards. Idempotent — detecte si la structure existe deja.

**Fichiers generes :**
- `cache/` — Cache de recherche local (companies, job boards)
- `cibles/` — Rapports de veille dates
- `candidatures/` — Un sous-dossier par cible
- `organisation.md` — Convention de nommage
- Profil candidat et grille de scoring (dans les references veille)

### `/emploi-veille` — Veille emploi multi-agents

Lance 4 agents en parallele :
1. **Scanner Offres** — Scrape les job boards (Indeed, APEC, LinkedIn, WTTJ, etc.)
2. **Scanner Acteurs** — Cartographie les employeurs du secteur dans la zone
3. **Analyste Marche** — Tendances, salaires, gap analysis
4. **Synthese** — Consolide, deduplique, priorise avec scoring /12

Produit un rapport dans `cibles/YYYY-MM-DD/` avec recommandations `/emploi-cible [slug]` pour les cibles >= 7/12.

### `/emploi-cible [nom]` — Arsenal candidature complet

Pour une cible donnee, genere tout l'arsenal :
- Recherche approfondie sur l'entreprise
- CV personnalise
- Lettre de motivation
- Pitch oral (2 min)
- Messages LinkedIn (<=300 car.)
- Email RH
- Fiche de suivi avec checklist

### `/emploi-print [nom]` — Export .docx

Convertit le CV et la LM d'une cible en fichiers `.docx` prets a envoyer.

## Structure du repo

```
.claude/
└── skills/
    ├── emploi-init/          # SKILL.md + 6 templates
    │   └── references/
    ├── emploi-cible/         # SKILL.md + prompt de recherche
    │   └── references/
    ├── emploi-veille/        # SKILL.md + 4 prompts agents
    │   └── references/
    └── emploi-print/         # SKILL.md
```

## Grille de scoring

Chaque cible est evaluee sur 4 axes (0-3 chacun, total /12) :

| Axe | Description |
|-----|-------------|
| Competences techniques | Match avec le profil du candidat |
| Localisation | Proximite ou remote |
| Responsabilite | Niveau du poste (junior → CTO) |
| Secteur cible | Pertinence sectorielle |

**Verdicts :** 10-12 = prioritaire · 7-9 = prometteuse · 4-6 = surveiller · 0-3 = ignorer

## Licence

MIT — voir [LICENSE](LICENSE)
