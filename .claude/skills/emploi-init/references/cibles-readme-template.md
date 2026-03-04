# Veille & Decouverte de Cibles Emploi

> **Objectif :** Decouvrir des opportunites, les scorer, recommander `/new-cible [slug]`.
> **Hors perimetre :** Recherche approfondie, CV, LM, pitchs, contacts (= `/new-cible`).

---

## Architecture

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Agent 1          │  │  Agent 2          │  │  Agent 3          │
│  Scanner Offres   │  │  Scanner Acteurs  │  │  Contexte Marche  │
│  (job boards)     │  │  (ecosysteme)     │  │  (tendances)      │
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
         │                     │                      │
         └─────────────┬───────┘──────────────────────┘
                       ▼
              ┌────────────────┐
              │  Agent Synthese │
              │  Consolidation  │
              │  + Priorisation │
              └────────┬───────┘
                       ▼
            rapport-veille-YYYY-MM-DD.md
                       │
              "Lance /new-cible X"
```

---

## Integration workflow

```
cibles/YYYY-MM-DD/rapport-veille-*.md  -->  "/new-cible [slug]"  -->  candidatures/[slug]/
(decouverte)                     (recherche + arsenal)      (CV, LM, pitch, suivi)
```

Le rapport de veille est **en amont** de `/new-cible`. Il decouvre, `/new-cible` traite.

---

## Sortie attendue

Chaque agent produit un fichier dans `cibles/YYYY-MM-DD/` :
- `cibles/YYYY-MM-DD/veille-offres-YYYY-MM-DD.md`
- `cibles/YYYY-MM-DD/veille-acteurs-YYYY-MM-DD.md`
- `cibles/YYYY-MM-DD/veille-marche-YYYY-MM-DD.md`
- `cibles/YYYY-MM-DD/rapport-veille-YYYY-MM-DD.md` (synthese finale)
