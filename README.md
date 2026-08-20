# DEM'S

Site vitrine + back-office administrable pour **DEM'S**, une entreprise de
prestation de services basée à Maroua, région de l'Extrême-Nord, Cameroun —
livraison et installation d'équipements informatiques pour établissements
scolaires, maintenance matérielle/logicielle, logiciels de gestion scolaire,
développement logiciel sur mesure pour toute entreprise, fourniture de
matériel didactique et production de gadgets publicitaires.

- **IMMATRICULATION :** P098416372154B
- **E-mail :** demanougraphicr@gmail.com
- **Téléphone :** 698 64 06 70 / 651 01 03 33
- **Localisation :** Maroua, Cameroun
- **Devise :** *« Innovons ensemble pour des établissements scolaires
  d'exception »*
- **Production :** https://dems.dmsacad.com

## Stack technique

Architecture découplée API REST + SPA — le frontend ne communique jamais
directement avec MySQL, toujours via l'API Laravel (`/api/...` public,
`/api/admin/...` protégé par Sanctum).

| Couche | Technologie |
|---|---|
| Backend | Laravel 11 (PHP 8.2+), MySQL |
| Authentification (admin uniquement) | Laravel Sanctum, auth par cookie SPA, rôle admin unique |
| Frontend | React 18 + Vite SPA, React Router, Tailwind CSS, i18next |
| Mobile | Wrapper Android Capacitor 8 — charge le site de production en direct, pas de bundle hors-ligne |

## Arborescence du dépôt

```
dems/
├── backend/                  API Laravel
│   ├── app/Http/Controllers/Api/       endpoints publics (services, portfolio, clients, contact, settings)
│   ├── app/Http/Controllers/Admin/     CRUD admin protégé par Sanctum
│   ├── database/migrations|seeders/
│   └── storage/app/public/             images uploadées, servies via storage:link
├── frontend/                 SPA React (Vite)
│   ├── src/pages/                      Home, Services, ServiceDetail, Portfolio, Clients, Contact, admin/
│   ├── src/api/                        client API (axios)
│   ├── android/                        projet Android Capacitor
│   └── scripts/                        génération de l'icône/splash à partir des visuels de marque
├── illustration/              visuels sources, importés via les seeders (absent du dépôt git)
└── AP-ICONs/                  visuels sources de l'icône de l'app (absent du dépôt git)
```

## Démarrage en local

Prérequis : PHP 8.2+ avec `gd`/`pdo_mysql`, Composer, MySQL (XAMPP convient),
Node 18+.

**Backend**

```bash
cd backend
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed      # seed services/portfolio/clients/settings, crée l'utilisateur admin
php artisan storage:link
php artisan serve --port=8001
```

Le seeder affiche une seule fois le mot de passe admin généré — à récupérer
immédiatement, il n'est stocké nulle part ailleurs en clair. Connexion sur
`/admin` avec `demanougraphicr@gmail.com`, puis changer le mot de passe.

**Frontend**

```bash
cd frontend
cp .env.example .env            # définir VITE_API_BASE_URL=http://localhost:8001
npm install
npm run dev                     # http://localhost:5173
```

Une fois le seed effectué, modifier le contenu via le panneau admin plutôt
que de relancer les seeders, sauf réinitialisation volontaire d'un
environnement neuf.

## Application mobile (Android)

`frontend/android/` est un wrapper Capacitor (`com.dmsacad.dems`) qui charge
directement `https://dems.dmsacad.com` (`server.url` dans
`capacitor.config.json`) — une simple coquille native, pas un bundle
hors-ligne : les changements de contenu/API sont donc pris en compte
immédiatement, sans reconstruction de l'app. Ne reconstruire que pour les
changements de coquille native / icône / splash :

```bash
cd frontend
npx cap sync android
npx capacitor-assets generate --android   # à relancer si les visuels dans resources/ changent
cd android
./gradlew.bat assembleDebug      # APK debug
./gradlew.bat bundleRelease      # AAB signé pour le Play Store
./gradlew.bat assembleRelease    # APK signé
```

Nécessite `JAVA_HOME` pointé sur un JDK 21 (le JBR intégré à Android Studio :
`C:\Program Files\Android\Android Studio\jbr`). La configuration de
signature de release (`frontend/android/key.properties`,
`frontend/android/keystore/`) est exclue du dépôt via `.gitignore`.

## Déploiement

- **Frontend :** GitHub Actions (`.github/workflows/deploy.yaml`) build
  `frontend/` et déploie `dist/` en FTP vers la production à chaque push sur
  `main`.
- **Backend :** déploiement manuel — voir le guide local `DEPLOYMENT.md`
  (variables d'environnement à changer en production, étapes de
  migration/seed, notes sur le stockage des images, checks post-déploiement).
  Non versionné dans git ; conservé comme document de référence local, aux
  côtés de `scope.md`, `implementation_plan.md`, `CLAUDE.md` et `AUDIT.md`.

## Projet lié, distinct

`dmsacad_backend_dev/` (répertoire voisin sur cette machine) est le propre
logiciel de gestion scolaire de DEM'S — un codebase Laravel différent, sans
rapport avec ce dépôt. Utile uniquement comme référence pour les conventions
Laravel de cette équipe.
