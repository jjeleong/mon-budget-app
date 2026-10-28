# Mon Budget — application web React

Application de budget mensuel en français, prête pour Vite, Netlify et GitHub Pages.

## Fonctionnalités

- Budgets mensuels navigables
- Revenus, dépenses fixes, dépenses variables et épargne
- Valeurs prévues et réelles modifiables
- Ajout et suppression de lignes
- Calculs automatiques et graphiques de répartition
- Notes mensuelles
- Export CSV compatible avec Excel / LibreOffice
- Connexion e-mail/mot de passe ou Google via Firebase
- Synchronisation cloud par utilisateur et par mois lorsqu'un projet Firebase est configuré
- Utilisation de l'interface sans Firebase en mode local

## 1. Lancer sur votre ordinateur

Installez Node.js (version 20 ou plus récente), puis dans le dossier du projet :

```bash
npm install
npm run dev
```

Vite affiche l'adresse locale à ouvrir dans le navigateur.

Pour vérifier la version de production :

```bash
npm run build
npm run preview
```

## 2. Activer la sauvegarde cloud (Firebase)

Sans configuration Firebase, l'application reste utilisable dans la session du navigateur, mais la sauvegarde cloud et les comptes ne fonctionnent pas. Les modifications ne sont pas garanties après rechargement dans ce mode : exportez régulièrement le CSV.

1. Créez un projet dans la console Firebase.
2. Ajoutez une application Web et copiez sa configuration.
3. Dans Authentication > Sign-in method, activez **Email/Password** et **Google**. Activez également **Anonymous** si nécessaire pour initialiser une session locale.
4. Créez une base Firestore.
5. Copiez `.env.example` en `.env.local`, puis renseignez les valeurs `VITE_FIREBASE_*`.
6. Redémarrez le serveur de développement ou reconstruisez le site.

### Règles Firestore recommandées

Dans Firestore > Rules, utilisez des règles qui n'autorisent chaque utilisateur qu'à lire et écrire ses propres documents :

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/months/{monthId} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

Publiez les règles après les avoir examinées. Ne mettez jamais de clé de compte de service Firebase dans le code client. Les identifiants Firebase Web sont destinés à l'application cliente, mais doivent être accompagnés de règles Firestore et des restrictions de sécurité appropriées.

## 3. Déployer sur Netlify

Méthode simple :
1. Envoyez ce dossier vers un dépôt GitHub.
2. Dans Netlify, choisissez **Add new site > Import an existing project**.
3. Sélectionnez le dépôt.
4. Build command : `npm run build`
5. Publish directory : `dist`
6. Dans les paramètres du site, ajoutez les variables d'environnement `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, `VITE_FIREBASE_PROJECT_ID`, `VITE_FIREBASE_STORAGE_BUCKET`, `VITE_FIREBASE_MESSAGING_SENDER_ID` et `VITE_FIREBASE_APP_ID`.
7. Relancez un déploiement après l'ajout des variables.

Vous pouvez aussi faire un déploiement manuel en lançant `npm run build`, puis en déposant le dossier `dist` dans l'interface de déploiement manuel Netlify.

## 4. Déployer sur GitHub Pages

1. Créez un dépôt GitHub et poussez-y les fichiers du projet.
2. Dans le dépôt, ouvrez **Settings > Pages**.
3. Choisissez **GitHub Actions** comme source de déploiement.
4. Créez `.github/workflows/deploy.yml` (voir l'exemple ci-dessous).
5. Ajoutez les variables Firebase dans **Settings > Secrets and variables > Actions > Variables**.
6. Dans `vite.config.js`, le `base: './'` est adapté à ce déploiement statique.
7. Poussez sur la branche principale pour déclencher la publication.

Fichier `.github/workflows/deploy.yml` :

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: true
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm install
      - run: npm run build
        env:
          VITE_FIREBASE_API_KEY: ${{ vars.VITE_FIREBASE_API_KEY }}
          VITE_FIREBASE_AUTH_DOMAIN: ${{ vars.VITE_FIREBASE_AUTH_DOMAIN }}
          VITE_FIREBASE_PROJECT_ID: ${{ vars.VITE_FIREBASE_PROJECT_ID }}
          VITE_FIREBASE_STORAGE_BUCKET: ${{ vars.VITE_FIREBASE_STORAGE_BUCKET }}
          VITE_FIREBASE_MESSAGING_SENDER_ID: ${{ vars.VITE_FIREBASE_MESSAGING_SENDER_ID }}
          VITE_FIREBASE_APP_ID: ${{ vars.VITE_FIREBASE_APP_ID }}
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

## À savoir

- Cette version est un projet complet à construire, pas un site déjà publié : elle nécessite votre compte Netlify ou GitHub pour obtenir une URL publique.
- En mode sans Firebase, l'état est local à la page et ne constitue pas un stockage persistant fiable.
- Chaque mois est enregistré dans un document distinct quand Firebase est activé.
