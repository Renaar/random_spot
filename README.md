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

## Déploiement local (Ubuntu Server + Docker + Dockge)

Prérequis : Docker et le plugin Compose déjà installés (c'est le cas si Dockge
tourne). Le port **3003** doit être libre.

### 1. Récupérer les fichiers sur le serveur

Le plus simple, et le plus pratique pour les mises à jour :

```bash
sudo git clone -b claude/plan-de-classe-app-y52xtq \
  https://github.com/Renaar/random_spot.git /opt/stacks/plan-de-classe
```

Sans git sur le serveur, copiez simplement `index.html` et `docker-compose.yml`
dans `/opt/stacks/plan-de-classe/` (clé USB, `scp`, partage réseau).

### 2. Ouvrir le port dans le pare-feu

```bash
sudo ufw allow 3003/tcp
```

### 3. Démarrer

En ligne de commande :

```bash
cd /opt/stacks/plan-de-classe
docker compose up -d
```

Ou dans Dockge : le dossier étant sous `/opt/stacks/`, la stack
**plan-de-classe** apparaît toute seule dans la liste — il suffit de cliquer
sur « Start ». Dockge lit le `docker-compose.yml` déjà présent.

### 4. Vérifier

```bash
docker compose ps                       # doit afficher "running"
curl -I http://localhost:3003           # doit répondre 200 OK
```

Depuis n'importe quel poste du réseau de l'école, y compris le PC du beamer :
`http://<adresse-ip-du-serveur>:3003`

Relevez l'adresse du serveur avec `hostname -I`. Une adresse fixe (réservation
DHCP sur le routeur) évite d'avoir à la rechercher chaque année.

### 5. Mettre à jour plus tard

```bash
cd /opt/stacks/plan-de-classe
sudo git pull
```

Le nouveau fichier est servi immédiatement : le dossier est monté dans nginx,
pas copié dedans. **Aucun redémarrage du conteneur n'est nécessaire.** Côté
navigateur, un `Ctrl+F5` garantit qu'on ne relit pas l'ancienne version en cache.

Sans git : remplacez `index.html`, c'est tout.

Redémarrer n'est utile que si vous modifiez `docker-compose.yml` :

```bash
docker compose up -d
```

### 6. Emporter ses classes vers le serveur

Les données vivent dans le navigateur, **et le stockage est propre à chaque
adresse**. Les classes créées sur `renaar.github.io` ne suivent donc pas
toutes seules vers `http://<serveur>:3003`.

1. Sur l'ancienne adresse : écran **Classes → Exporter (JSON)**.
2. Sur la nouvelle : écran **Classes → Importer…**, choisir le fichier.

Même manipulation pour passer d'un ordinateur à un autre. C'est aussi la
sauvegarde : un export JSON de temps en temps, rangé dans vos documents.

### En cas de souci

| Symptôme | Cause probable | Solution |
|---|---|---|
| `port is already allocated` au démarrage | le port 3003 est pris | `sudo ss -lntp \| grep 3003`, puis changer `"3003:80"` en `"3004:80"` |
| Page inaccessible depuis un autre poste | pare-feu | `sudo ufw status`, vérifier la règle `3003/tcp` |
| Modifications invisibles après mise à jour | cache du navigateur | `Ctrl+F5` |
| Page blanche | `index.html` absent du dossier monté | `ls /opt/stacks/plan-de-classe/` |

Remarque : le dossier entier est servi par nginx, donc `README.md` (et `.git`
si vous avez cloné) sont aussi accessibles sur le port 3003. Sans conséquence —
le dépôt est public et le service reste sur le réseau interne — mais si vous
préférez ne publier que l'application, déplacez `index.html` dans un
sous-dossier `app/` et montez `./app` au lieu de `./`.

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
