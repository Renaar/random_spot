# Plan de Classe

Application web pour composer le plan de salle d'une classe et tirer au sort les
places des élèves en direct, projetée au beamer.

Un seul fichier : `index.html`. Aucune dépendance, aucun réseau requis.
Elle fonctionne aussi par simple double-clic sur le fichier.

## Arborescence attendue sur le serveur

```
/opt/stacks/plan-de-classe/
├── docker-compose.yml
├── index.html          ← l'application entière
└── README.md
```

Le dossier complet est monté en lecture seule dans nginx : `index.html` est servi
à la racine du port 3003.

## Installation

```bash
sudo mkdir -p /opt/stacks/plan-de-classe
# y copier docker-compose.yml et index.html

sudo ufw allow 3003/tcp

cd /opt/stacks/plan-de-classe
docker compose up -d
```

L'application est ensuite disponible sur `http://<adresse-du-serveur>:3003`
(les ports 3000, 3001 et 3002 restent libres pour les autres applications).

Dans Dockge : ajouter la stack `plan-de-classe`, coller le contenu de
`docker-compose.yml`, puis déployer.

## Mise à jour

L'image nginx ne contient rien de l'application : il suffit de remplacer le fichier.

```bash
cd /opt/stacks/plan-de-classe
sudo cp /chemin/vers/nouveau/index.html ./index.html
```

Le nouveau fichier est servi immédiatement (le dossier est monté, pas copié).
Les élèves et les enseignants doivent simplement recharger la page —
au besoin avec `Ctrl+F5` pour contourner le cache du navigateur.

Redémarrer le conteneur n'est nécessaire que si `docker-compose.yml` change :

```bash
docker compose up -d
```

## Où vivent les données ?

Tout est enregistré dans le `localStorage` du **navigateur qui ouvre la page**,
sous la clé `plan-de-classe/v1` : rien n'est stocké sur le serveur, et rien
n'est partagé entre deux machines. Utilisez l'export JSON (écran « Classes »)
pour sauvegarder ou transférer vos classes.

## Publication sur GitHub Pages (pour tester à distance)

Une seule fois, dans l'interface web du dépôt :
**Settings → Pages → Source : « Deploy from a branch » → branche `claude/plan-de-classe-app-y52xtq`, dossier `/ (root)` → Save.**

L'adresse devient `https://renaar.github.io/random_spot/` et se met à jour
à chaque poussée sur la branche.

Le workflow `.github/workflows/pages.yml` sert uniquement si vous préférez
l'option « Source : GitHub Actions » ; il se lance alors à la main
(onglet Actions → GitHub Pages → Run workflow).
