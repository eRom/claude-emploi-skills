# Cache de recherche local

> Systeme de cache pour eviter les recherches web redondantes. **Toujours consulter le cache AVANT de lancer une recherche web.**

## Structure

```
cache/
├── companies/          # JSON par entreprise deja recherchee
│   └── {slug}.json     # Schema : name, sector, location, key_contacts, last_updated, status, notes
├── job-boards/         # URLs connues et fiabilite des sites emploi
│   └── registry.json   # Registre central des job boards
└── README.md
```

## Schema entreprise (`companies/{slug}.json`)

```json
{
  "name": "Nom complet",
  "slug": "nom-pour-fichiers",
  "sector": "Sante numerique | ESN | Public hospitalier | ...",
  "location": "Ville, departement",
  "website": "https://...",
  "size": "PME | ETI | GE | TPE",
  "key_contacts": [
    {
      "name": "Prenom Nom",
      "role": "Titre",
      "linkedin_contacted": false,
      "email": "",
      "date_contact": ""
    }
  ],
  "candidature_status": "non_demarre",
  "dossier_path": "candidatures/slug/",
  "last_updated": "YYYY-MM-DD",
  "notes": ""
}
```

## Regles d'utilisation

1. **Avant toute recherche web sur une entreprise** : verifier `cache/companies/{slug}.json`
2. **Apres toute recherche** : mettre a jour ou creer l'entree cache
3. **Job boards** : verifier `cache/job-boards/registry.json` avant de fetcher une URL
