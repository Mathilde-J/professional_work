### ARCHITECURE GLOBALE
#### Structure

Application de backoffice pour le pilotage des éditions du jeu Ma Petite Planète en Next.js 13+ utilisant App router. Elle utilise Supabase comme Backend avec une base de données PostgreSQL, MobX comme gestionnaire d'état et i18n pour gérer l'intertionalisation (français/anglais).

```
backoffice_ecu/
├── app/                          # Next.js App Router directory
│   ├── auth/
│   │   └── callback/
│   ├── components/               # React components
│   │   └── components specific for the app
│   ├── context/                  # React contexts
│   │   ├── ParentElementContext.ts
│   │   ├── StoresContext.ts
│   │   ├── ToasterContext.tsx
│   │   └── UnsaveParents.tsx
│   ├── stores/                   # Mobx stores
│   │   └── stores used to handle pages' data and state
│   ├── [locale]/                 # Internationalized routes
│   │   ├── clientProvider.tsx
│   │   ├── layout.tsx            # Commun layout for protected and anon pages
│   │   ├── (anon-page)/
│   │   │    ├── layout.tsx       # Commun layout for anon pages
│   │   ├── (protected-page)/
│   │   │   └── layout.tsx        # Commun layout for protected pages
│   │   └── loading-redirect/
│   ├── global.css
│   ├── i18n.ts
│   ├── icon.png
│   ├── layout.tsx                # Commun layout for all pages (For Stores provider)
│   ├── supabaseActionClient.ts
│   └── supabaseActionServer.ts
├── database/                     # Database operations
│   ├── create/
│   │   ├── ...
│   │   └── files used to create data in the database
│   ├── drop/
│   │   ├── ...
│   │   └── files used to delete data in the database
│   ├── export/
│   │   ├── ...
│   │   └── files used to export data from the database
│   ├── listen/
│   │   ├── ...
│   │   └── files used to listen for changes from some tables
│   ├── read/
│   │   ├── ...
│   │   └── files used to retrieve data from the database
│   ├── storage/
│   │   ├── ...
│   │   └── files that handles storage intreactions with the database
│   └── update/
│       ├── ...
│       └── files used to update data in the database
├── locales/                      # Internationalization files
│   ├── en/
│   │   └── ... JSON files for translations
│   └── fr/
│       └── ... JSON files for translations
├── models/                       # TypeScript models
│   ├── ...
│   └── models files
├── public/                       # Static assets
│   └── assets/
├── types/                        # TypeScript type definitions
│   ├── ...
│   └── Files for typescript types
└── utils/                        # Utility functions
    ├── constant.ts
    ├── hooks/                    # Custom React hooks
    ├── i18n/                     # i18n integration
    ├── mixpanel/                 # Analytics tracking
    └── supabase/                 # Supabase integration
```

En résumé :
- app/: Contient l'application Next.js avec App Router aves les composants spécifiques de l'application, les contexts react, les stores
- database/: Contient les CRUD des différentes entités aevc Supabase.
- locales/: Contient les fichiers de traduction pour en français et anglais.
- models/: Les models et interfaces pour TypeScript.
- types/: Les types pour TypeScript.
- utils/: Fonctions utilisatires, constants, hooks, i18n.