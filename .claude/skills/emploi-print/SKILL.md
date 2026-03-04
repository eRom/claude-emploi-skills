---
name: emploi-print
description: >
  Demarre le process complet pour convertir le CV, Lettre de motivation (LM) d'une cible de candidature (entreprise).
  Utilise cette skill des que l'utilisateur veut confirmer sa candidature chez une entreprise (cible), dit "print cible", "prepare le print pour X" ou "/emploi-print X".
  La skill orchestre tout : creation du dossier, convertion md -> docx (skill docx)
---

# /emploi-print — Cible de candidature

## Contexte

Toutes les candidatures vivent dans `candidatures/[cible]/` avec une convention de nommage stricte.

---

## Etape 0 — Initialisation

Si le nom de la cible n'est pas fourni en argument, STOP et previent l'utilisateur par un simple message.

Determine le **slug** de la cible : minuscules, tirets, sans accents (ex: `lna-sante`, `chu-nantes`, `ico`).

- Recherche et lit le dossier : `candidatures/[slug]/`
- Cree la structure de dossier :
```
candidatures/[slug]/
└── print/
```

---

## Etape 1 — CV

Fichier : `candidatures/[slug]/CV_[NOM_PRENOM].md`

Process :
- Converti le fichier en utilisant la **Skill docx**
- Sauvegarde dans le dossier :  `candidatures/[slug]/print/CV_[NOM_PRENOM]-[slug].docx`

---

## Etape 2 — Lettre de motivation (LM)

Fichier : `candidatures/[slug]/LM_[NOM_PRENOM]-[slug].md`

Process :
- Converti le fichier en utilisant la **Skill docx**
- Sauvegarde dans le dossier :  `candidatures/[slug]/print/LM_[NOM_PRENOM]-[slug].docx`

---


## Etape 3 — Final check

Verifie l'existance des 2 fichiers :
- `candidatures/[slug]/print/CV_[NOM_PRENOM]-[slug].docx`
- `candidatures/[slug]/print/LM_[NOM_PRENOM]-[slug].docx`


---

## Regles generales

- **Convention nommage stricte** : tirets `-` comme separateur entre type et slug (ex: `CV_[NOM_PRENOM]-[slug].md`)
- **Fichiers .docx/.pdf** → toujours dans `print/`
- **Si un document manque** → STOP, demander a l'utilisateur
- **Si la cile n'existe pas** → STOP, demander a l'utilisateur
