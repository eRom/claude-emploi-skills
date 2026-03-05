---
name: emploi-cible
description: >
  Demarre le process complet pour une nouvelle cible de candidature (entreprise).
  Utilise cette skill des que l'utilisateur mentionne une nouvelle entreprise a prospecter,
  dit "nouvelle cible", "je veux candidater chez X", "on attaque X", ou "/emploi-cible".
  La skill orchestre tout : creation du dossier, recherche, CV, LM, pitch, contacts, suivi.
---

# /emploi-cible — Nouvelle cible de candidature

## Contexte

Lire le profil candidat dans `.claude/skills/emploi-veille/references/profil-candidat.md`.
Toutes les candidatures vivent dans `candidatures/[cible]/` avec une convention de nommage stricte.
Le prompt de recherche de reference est dans `.claude/skills/emploi-cible/references/research-cible-prompt.md`.

**Convention de nommage :** deriver `NOM_PRENOM` du profil candidat (format `Prenom_Nom`, underscores). Utiliser pour les noms de fichiers CV et LM.

---

## Etape 0 — Initialisation

Si le nom de la cible n'est pas fourni en argument, demande-le.

Determine le **slug** de la cible : minuscules, tirets, sans accents (ex: `lna-sante`, `chu-nantes`, `ico`).

Cree la structure de dossier :
```
candidatures/[slug]/
└── print/
```

---

## Etape 1 — Recherche

Lance une recherche approfondie sur la cible en suivant le prompt dans `.claude/skills/emploi-cible/references/research-cible-prompt.md`.

Variables a remplir :
- `{{ORGANISATION}}` : nom complet
- `{{DESCRIPTION_COURTE}}` : groupe, localisation, secteur
- `{{DOMAINE}}` : IT / Architecture SI / domaine pertinent (selon la cible)
- `{{CHEMIN_FICHIER_SORTIE}}` : `candidatures/[slug]/Recherches-[slug].md`

Sauvegarde les resultats dans `candidatures/[slug]/Recherches-[slug].md`.

Apres la recherche, presente une synthese a l'utilisateur :
- Fit global (sur 10)
- Contact cle identifie
- Portail de candidature
- Angle d'attaque recommande

---

## Etape 2 — Arsenal documents

Cree les documents dans l'ordre suivant. Pour chaque document, utilise les resultats de la recherche et le profil candidat (`.claude/skills/emploi-veille/references/profil-candidat.md`).

### 2a. CV personnalise
Fichier : `candidatures/[slug]/CV_[NOM_PRENOM]-[slug].md`

Adapte le CV de reference (disponible dans d'autres dossiers candidatures pour s'inspirer) :
- Headline et resume cibles sur le poste vise
- Mise en avant des experiences les plus pertinentes
- Projets techniques recents toujours presents
- Mentionner les atouts differenciants du profil si pertinent

### 2b. Lettre de motivation
Fichier : `candidatures/[slug]/LM_[NOM_PRENOM]-[slug].md`

Structure :
- Accroche personnalisee a la cible (eviter les formules generiques)
- Parcours → competence cle → lien avec la mission de la cible
- Utiliser les atouts differenciants du profil candidat pour personnaliser l'angle
- Adapter le ton au secteur de la cible

### 2c. Pitch oral (~2 minutes)
Fichier : `candidatures/[slug]/Pitch-[slug].md`

Structure : Accroche → Parcours → Preuve (projets recents) → Projection
Ajouter notes de jeu et "pourquoi [cible] ?" preparee.

---

## Etape 3 — Contacts LinkedIn

Pour chaque contact cle identifie a l'etape 1 :

Fichier : `candidatures/[slug]/LinkedIn-[slug]-[Prenom]_[Nom].md`

Contenu :
- Message LinkedIn <= 300 caracteres
- Icebreakers specifiques a ce contact
- Profil LinkedIn si trouve

---

## Etape 4 — Email RH (si portail ou email identifie)

Fichier : `candidatures/[slug]/Email-[slug]-RH.md`

Email court (5-6 lignes max) :
- Objet : `Candidature — [Poste] | [NOM]`
- Corps : poste vise, 1 phrase profil, 1 phrase differenciante, CV en PJ, LinkedIn

---

## Etape 5 — Suivi

Cree `candidatures/[slug]/Suivi-[slug].md` :

```markdown
# Suivi — [Nom complet cible]
> Mise a jour : [date]

**Poste vise :** [intitule]
**Contact cle :** [Prenom Nom] ([fonction])

---

## Checklist

- [ ] Recherche terminee
- [ ] CV envoye
- [ ] LM envoyee
- [ ] Contact envoye
- [ ] Entretien prepare
- [ ] Reponse recue

---

## Journal

| Date | Action |
|------|--------|
```

---

## Etape 6 — Mise a jour du suivi global

Ajoute la cible dans `suivi_contacts.md` si des contacts ont ete identifies (une section par employeur, statut par defaut).

---

## Regles generales

- **Profil candidat** : toujours lire `.claude/skills/emploi-veille/references/profil-candidat.md` au runtime
- **Convention nommage stricte** : tirets `-` comme separateur entre type et slug (ex: `CV_Prenom_Nom-slug.md`)
- **Fichiers .docx/.pdf** → toujours dans `print/`
- **Si un document manque** → continuer sans bloquer
- **Longueur messages LinkedIn** → <= 300 caracteres, toujours
- **Ton** : pragmatique et sincere, jamais vendeur. Authenticite > marketing.
