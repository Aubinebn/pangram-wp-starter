# Pangram WP Starter

Starter pour développer des sites WordPress avec [Bedrock](https://roots.io/bedrock/) et le thème [Sage](https://roots.io/sage/), dans un environnement local [DDEV](https://ddev.com/).

## Sommaire

- [Comment le projet est construit](#comment-le-projet-est-construit)
- [Get started](#get-started)
- [Commandes DDEV](#commandes-ddev)
- [Utilisation au quotidien](#utilisation-au-quotidien)
- [Déploiement : rappels](#déploiement--rappels)
- [Documentation](#documentation)

## Comment le projet est construit

Le projet assemble trois briques :

| Brique | Rôle |
| --- | --- |
| **Bedrock** | Structure du projet WordPress. Le cœur WordPress, les plugins et les thèmes sont installés avec Composer, et la configuration est lue dans un fichier `.env`. |
| **Sage** (dans `web/app/themes/your-theme-name`) | Thème de départ : templates Blade, Tailwind CSS 4, build des assets avec Vite, et [Acorn](https://roots.io/acorn/) qui apporte des composants Laravel dans WordPress. |
| **DDEV** (dossier `.ddev/`) | Environnement local dans Docker : PHP 8.5, Apache, MariaDB 10.6, Node 24, Composer 2 et WP-CLI. |

### Arborescence

```
.
├── .ddev/                  # Configuration DDEV (config.yaml)
├── config/
│   ├── application.php     # Configuration principale (remplace wp-config.php)
│   └── environments/       # Surcharges par environnement (development, staging)
├── vendor/                 # Dépendances Composer (non versionné)
├── web/                    # Document root
│   ├── app/                # Équivalent de wp-content
│   │   ├── mu-plugins/
│   │   ├── plugins/        # Installés via Composer (non versionnés)
│   │   ├── themes/
│   │   │   └── your-theme-name/   # Thème Sage du projet
│   │   └── uploads/        # Médias (non versionnés)
│   ├── wp/                 # Cœur WordPress, installé par Composer (non versionné)
│   └── wp-config.php       # Ne pas modifier : il charge config/application.php
├── .env.example            # Modèle de configuration à copier en .env
├── composer.json           # Dépendances PHP du site (WordPress, plugins…)
└── wp-cli.yml
```

À retenir :

- **Ne modifiez pas `web/wp/`** : ce dossier est réécrit par Composer. WordPress se met à jour en changeant la version de `roots/wordpress` dans `composer.json`.
- **L'admin WordPress est sous `/wp/`** : `https://<projet>.ddev.site/wp/wp-admin`.
- **La configuration se fait dans `.env` et `config/`**, jamais dans `wp-config.php`.

## Get started

### Prérequis

- [DDEV](https://ddev.readthedocs.io/en/stable/users/install/) et un moteur Docker (Docker Desktop, OrbStack, Docker dans WSL2…)
- Git


### 1. Récupérer le starter

```bash
git clone <url-du-starter> mon-site
cd mon-site
```

### 2. Nommer le projet DDEV

Dans `.ddev/config.yaml`, remplacez `name: pangram-wp-starter` par le nom de votre projet. Ce nom donne l'URL locale : `https://mon-site.ddev.site`.

### 3. Démarrer l'environnement

```bash
ddev start
```

### 4. Créer le fichier `.env` et générer les salts

```bash
cp .env.example .env
```

Remplacez ensuite les valeurs `generateme` des clés et salts de sécurité (`AUTH_KEY`, `SECURE_AUTH_KEY`, …, `NONCE_SALT`) par des valeurs aléatoires. Deux possibilités :

- **Avec la commande**  :
  ```bash
  ddev salts
  ```
- **Sur le site de Roots** : <https://roots.io/salts.html>

### 5. Installer les dépendances

```bash
ddev composer install
```

### 6. Installer WordPress

Avec WP-CLI :

```bash
ddev wp core install \
  --url='https://mon-site.ddev.site' \
  --title='Mon site' \
  --admin_user=admin \
  --admin_password=motdepasse \
  --admin_email=admin@example.com
```

Vous pouvez aussi ouvrir le site (`ddev launch`) et suivre l'installation dans le navigateur.

### 7. Préparer et activer le thème Sage

1. Renommez le dossier `web/app/themes/your-theme-name` avec le nom de votre thème, par exemple `mon-theme`.
2. Dans `web/app/themes/mon-theme/vite.config.js`, faites correspondre `base` au nouveau nom du dossier :
   ```js
   base: '/app/themes/mon-theme/public/build/',
   ```
3. Renseignez le nom, la description et l'auteur dans l'en-tête de `style.css`.
4. Installez les dépendances et compilez les assets :
   ```bash
   ddev theme-install mon-theme
   ```
   La commande lance `composer install`, `npm install` puis `npm run build` dans le dossier du thème.
5. Activez le thème :
   ```bash
   ddev wp theme activate mon-theme
   ```

Ouvrez le site avec `ddev launch`, et l'admin avec `ddev launch wp/wp-admin`.

## Commandes DDEV

### Commandes du starter

Ces commandes sont propres au starter. Elles se trouvent dans `.ddev/commands/web/` et s'exécutent dans le conteneur web.

| Commande | Description |
| --- | --- |
| `ddev theme-install <nom-du-theme>` | Installe le thème Sage : `composer install`, `npm install` et `npm run build`. Prévient si `base` dans `vite.config.js` ne correspond pas au nom du dossier. |
| `ddev salts` | Génère 8 nouvelles clés et salts WordPress dans `.env`, au même format que <https://roots.io/salts.html>. Crée `.env` à partir de `.env.example` s'il n'existe pas. |

Pour ajouter une commande, créez un script exécutable dans `.ddev/commands/web/` (exécuté dans le conteneur) ou dans `.ddev/commands/host/` (exécuté sur votre machine). Voir la [documentation DDEV](https://docs.ddev.com/en/stable/users/extend/custom-commands/).

### Commandes DDEV courantes

| Commande | Description |
| --- | --- |
| `ddev start` / `ddev stop` | Démarrer / arrêter le projet |
| `ddev launch` | Ouvrir le site dans le navigateur |
| `ddev wp <cmd>` | Lancer une commande WP-CLI |
| `ddev composer <cmd>` | Lancer Composer à la racine du projet |
| `ddev ssh` | Ouvrir un shell dans le conteneur web |
| `ddev export-db --file=dump.sql.gz` | Exporter la base |
| `ddev import-db --file=dump.sql.gz` | Importer une base |
| `ddev mailpit` | Voir les emails envoyés par le site |
| `ddev xdebug on` / `off` | Activer / désactiver Xdebug |
| `ddev restart` | Redémarrer le projet (après une modification de `.ddev/config.yaml`) |
| `ddev describe` | Afficher les URLs et les services du projet |
| `ddev help` | Lister toutes les commandes, y compris celles du starter |

## Utilisation au quotidien

### Ajouter des plugins et des thèmes

Les plugins et thèmes du répertoire wordpress.org s'installent avec Composer, via le dépôt [WP Packages](https://wp-packages.org/) déjà déclaré dans `composer.json` :

```bash
ddev composer require wp-plugin/wordpress-seo
ddev composer require wp-theme/nom-du-theme
ddev wp plugin activate wordpress-seo
```

Les plugins sont installés dans `web/app/plugins/`, et ce dossier n'est pas versionné : seuls `composer.json` et `composer.lock` font foi. Pour un plugin premium ou maison, ajoutez un dépôt dans `composer.json`, ou versionnez le plugin en ajoutant une exception dans `.gitignore`.

Les plugins placés dans `web/app/mu-plugins/` sont toujours chargés. Grâce à `bedrock-autoloader`, cela vaut aussi pour les plugins rangés dans un sous-dossier.

### Mettre à jour WordPress

Changez la version de `roots/wordpress` dans `composer.json`, puis :

```bash
ddev composer update roots/wordpress
```

### Développer le thème

Le guide de développement du thème est dans **[`web/app/themes/your-theme-name/DEVELOPMENT.md`](web/app/themes/your-theme-name/DEVELOPMENT.md)**. Il renvoie vers la [documentation Sage](https://roots.io/sage/docs/) et explique ce qui est propre au starter : compiler le thème, le renommer, lancer les commandes dans DDEV.

### Utiliser `npm run dev` avec DDEV

`npm run dev` lance le serveur de développement Vite (rechargement à chaud du CSS et du JS). Il tourne dans le conteneur DDEV, sur le port `5173`. Par défaut, le navigateur ne peut pas le joindre : sans la configuration ci-dessous, utilisez `npm run build` après chaque modification.

**1. Exposer le port de Vite** dans `.ddev/config.yaml` :

```yaml
web_extra_exposed_ports:
  - name: vite
    container_port: 5173
    http_port: 5172
    https_port: 5173
```

Puis appliquez la modification :

```bash
ddev restart
```

**2. Configurer le serveur Vite** dans `web/app/themes/mon-theme/vite.config.js`, en ajoutant une clé `server` à `defineConfig` :

```js
export default defineConfig({
  base: '/app/themes/mon-theme/public/build/',
  server: {
    host: '0.0.0.0',
    port: 5173,
    strictPort: true,
    origin: `${process.env.DDEV_PRIMARY_URL}:5173`,
    cors: {
      origin: /https?:\/\/([A-Za-z0-9\-\.]+)?(\.ddev\.site)(?::\d+)?$/,
    },
  },
  // … plugins, resolve, etc.
})
```

- `host: '0.0.0.0'` rend Vite accessible depuis l'extérieur du conteneur.
- `origin` est l'URL publique du serveur Vite. Le plugin Laravel l'écrit dans le fichier `public/hot`, et Sage s'en sert pour charger les assets depuis Vite au lieu de `public/build`.
- `cors` autorise le site `*.ddev.site` à charger les assets servis par Vite.

**3. Lancer le serveur** depuis le dossier du thème :

```bash
ddev ssh
cd web/app/themes/mon-theme
npm run dev
```

Ouvrez le site normalement (`https://mon-site.ddev.site`) : les modifications du CSS et du JS s'appliquent sans recharger la page. Quand vous arrêtez le serveur (`Ctrl+C`), le fichier `public/hot` est supprimé et le site utilise à nouveau les assets compilés : relancez `npm run build` pour les mettre à jour.


### Qualité du code

```bash
ddev composer lint       # vérifie le style PHP (Pint)
ddev composer lint:fix   # corrige le style PHP
ddev composer test       # lance les tests (Pest)
```

## Déploiement : rappels

- Le document root du serveur doit pointer sur `web/`, pas sur la racine du projet.
- Sur le serveur, lancez `composer install --no-dev` à la racine **et** dans le dossier du thème, puis `npm ci && npm run build` dans le thème (ou déployez le dossier `public/build` déjà compilé).
- Créez un `.env` propre au serveur avec `WP_ENV='production'` (ou `staging`), les identifiants de la base, les URLs en dur et des clés générées pour cet environnement.

## Documentation

- [Bedrock](https://roots.io/bedrock/docs/)
- [Sage](https://roots.io/sage/docs/)
- [Acorn](https://roots.io/acorn/docs/)
- [DDEV](https://ddev.readthedocs.io/)
- [WP Packages](https://wp-packages.org/)
