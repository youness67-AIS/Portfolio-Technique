
# 🐋 Challenge Docker Swarm
## Déployer GLPI sur un cluster avec Portainer

## 🗂 Contexte

Hier soir, GLPI a été déployé avec Docker Compose sur une seule machine.  
C’était une première étape, mais ce n’est pas suffisant pour une infrastructure de production.

L’objectif de ce challenge était de passer à **Docker Swarm**, afin d’obtenir un fonctionnement en **cluster**, avec plusieurs **replicas** de l’application GLPI, tout en gardant une seule base de données MariaDB.

Le déploiement devait se faire via **Portainer**, en respectant les consignes du challenge.

> 💡 **Docker Compose vs Docker Swarm**
> - **Compose** → un seul hôte, idéal pour le développement
> - **Swarm** → plusieurs hôtes, orchestration, haute disponibilité
> - La syntaxe reste proche, mais Swarm ajoute la gestion des replicas et des services

---

## 🎯 Objectifs

À la fin de l’exercice, il fallait :

- Initialiser un cluster Docker Swarm
- Déployer une stack via l’interface Portainer
- Configurer GLPI pour tourner en **3 replicas**
- Observer le comportement du load balancer Ingress de Swarm
- Comprendre pourquoi une base de données ne doit pas être multipliée sans réplication adaptée
- Simuler une panne et vérifier la recréation automatique d’un replica

---

## 📋 Contraintes & règles du jeu

> ⚠️ **Important**
>
> ✓ Repartir du `compose.yaml` comme base  
> ✓ Utiliser **Portainer** pour déployer la stack  
> ✓ Tester l’accès à GLPI depuis le navigateur avant de passer à la suite  
> ✗ Ne pas tout faire uniquement en CLI  

---

# 🔧 Déroulement du challenge

## Étape 1 — Vérification de l’environnement

Avant de commencer, nous avons vérifié que l’environnement était déjà prêt :

- Portainer tournait déjà
- Docker Swarm était déjà actif
- Le cluster contenait **1 manager** et **2 autres nœuds**

### Commandes utilisées

```bash
docker ps
docker info | grep -i swarm
docker node ls
hostname -I
```

### Résultat

- Portainer était bien lancé
- Swarm était `active`
- le cluster contenait :
  - `Docker-YNS` → Leader / Manager
  - `Docker2-YNS`
  - `Docker3-YNS`

---

## Étape 2 — Création d’un dossier de travail

Comme le fichier `compose.yaml` n’était pas disponible dans un emplacement propre, nous avons créé un répertoire dédié au challenge.

### Commandes utilisées

```bash
mkdir -p /root/glpi-swarm
cd /root/glpi-swarm
touch /root/glpi-swarm/compose.yaml
nano /root/glpi-swarm/compose.yaml
```

---

## Étape 3 — Premier contenu du `compose.yaml`

Le challenge demandait de partir d’un compose adapté à Swarm.

La base proposée était :

```yaml
services:
  glpi:
    image: diouxx/glpi
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"

  db:
    image: mariadb:11
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
```

Mais dans notre cas, cette version n’était pas suffisante, car :

- le port `80` était déjà utilisé par un autre service
- MariaDB ne pouvait pas démarrer sans variables d’environnement

---

## Étape 4 — Premier déploiement via Portainer

Dans Portainer, nous avons :

1. Ouvert **Stacks**
2. Cliqué sur **Add stack**
3. Nommé la stack : `glpi-swarm`
4. Collé le contenu du compose
5. Cliqué sur **Deploy the stack**

### Premier problème rencontré

Le déploiement a échoué avec une erreur sur le port `80`.

### Erreur

```text
port '80' is already in use by service 'web'
```

### Cause

Le port `80` était déjà occupé par le service `web` présent sur le cluster.

### Solution

Nous avons remplacé :

```yaml
ports:
  - "80:80"
```

par :

```yaml
ports:
  - "8080:80"
```

---

## Étape 5 — Ajout des variables MariaDB

Après correction du port, la stack se déployait, mais MariaDB restait bloquée en `0/1`.

En inspectant le service, nous avons compris que l’image `mariadb:11` ne pouvait pas démarrer sans variables d’environnement minimales.

Nous avons donc ajouté :

```yaml
environment:
  MARIADB_ROOT_PASSWORD: rootpass
  MARIADB_DATABASE: glpi
  MARIADB_USER: glpi
  MARIADB_PASSWORD: glpipass
```

---

## Étape 6 — Compose final déployé

Le fichier finalement utilisé dans Portainer était :

```yaml
services:
  glpi:
    image: diouxx/glpi
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
    ports:
      - "8080:80"

  db:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: rootpass
      MARIADB_DATABASE: glpi
      MARIADB_USER: glpi
      MARIADB_PASSWORD: glpipass
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
```

---

## Étape 7 — Problème DNS / Docker Hub

Même avec le bon compose, MariaDB ne démarrait toujours pas.

### Vérifications effectuées

```bash
docker service ls
docker service ps glpi-swarm_db
docker images | grep mariadb
docker pull mariadb:11
```

### Erreur observée

```text
failed to resolve reference "docker.io/library/mariadb:11": failed to do request:
lookup registry-1.docker.io on 10.0.0.10:53: i/o timeout
```

### Cause

Le serveur utilisait un DNS interne (`10.0.0.10`) qui ne résolvait pas correctement Docker Hub.

### Correction temporaire

```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

Puis nouveau test :

```bash
docker pull mariadb:11
```

### Résultat

L’image MariaDB a bien été téléchargée, puis le service est passé à `1/1`.

---

## Étape 8 — Vérification des services

Une fois MariaDB téléchargée, GLPI a pu démarrer correctement.

### Commandes utilisées

```bash
docker service ls
docker service ps glpi-swarm_glpi
docker service ps glpi-swarm_db
```

### Résultat

- `glpi-swarm_db` → `1/1`
- `glpi-swarm_glpi` → `3/3`

---

## Étape 9 — Test dans le navigateur

GLPI était accessible via :

```text
http://10.0.0.134:8080
```

### Résultat

La page d’installation de GLPI s’affichait correctement.

---

## Étape 10 — Observation des logs

Pour observer le comportement du service GLPI, nous avons utilisé :

```bash
docker service logs -f glpi-swarm_glpi
```

### Observation

Les logs montraient bien l’existence des différents replicas GLPI, mais surtout leur phase d’initialisation et de téléchargement.

---

## Étape 11 — Simulation de panne

Nous avons listé les conteneurs GLPI :

```bash
docker ps | grep glpi
```

### Exemple de conteneurs vus

```text
a2ede83338fe   diouxx/glpi:latest   glpi-swarm_glpi.1.a8ovbtwo5rpx1wvzp30rm6ra5
eab6d01fa4c7   diouxx/glpi:latest   glpi-swarm_glpi.3.orymznd0wgxgu8xan58kktf25
51162b5205b5   diouxx/glpi:latest   glpi-swarm_glpi.2.j64e38irv8jw88bukfo8wgt36
399917dd0c56   mariadb:11           glpi-swarm_db.1.mmgj84llvf7b8d1sx9frvnthu
```

Nous avons supprimé brutalement un conteneur GLPI :

```bash
docker rm -f a2ede83338fe
```

Puis vérifié le service :

```bash
docker service ps glpi-swarm_glpi
```

### Résultat

Swarm a automatiquement recréé le replica manquant.

---

## Étape 12 — Problème sur le nœud `Docker3-YNS`

Après suppression du conteneur, Swarm a tenté de recréer un replica sur `Docker3-YNS`, mais celui-ci restait bloqué en `Preparing`.

### Vérifications utilisées

```bash
docker service ps glpi-swarm_glpi --no-trunc
docker node ls
docker images | grep diouxx
```

Comme le replica ne remontait pas correctement, nous avons temporairement mis le nœud `Docker3-YNS` en mode `drain`.

### Commande utilisée

```bash
docker node update --availability drain Docker3-YNS
```

Swarm a alors replanifié le replica sur `Docker-YNS`.

### Vérification

```bash
docker service ps glpi-swarm_glpi
docker service ls
```

### Résultat

GLPI est revenu à `3/3`.

Ensuite, nous avons réactivé le nœud :

```bash
docker node update --availability active Docker3-YNS
```

Puis vérifié à nouveau :

```bash
docker node ls
docker service ls
```

---

## Étape 13 — Vérifications finales

### État du cluster

```bash
docker node ls
```

### État des services

```bash
docker service ls
```

### Résultat final

```text
glpi-swarm_db         replicated   1/1
glpi-swarm_glpi       replicated   3/3
portainer_agent       global       3/3
portainer_portainer   replicated   1/1
web                   replicated   3/3
```

### Vérification détaillée de GLPI

```bash
docker service ps glpi-swarm_glpi
```

Résultat :

- `glpi-swarm_glpi.1` → Running
- `glpi-swarm_glpi.2` → Running
- `glpi-swarm_glpi.3` → Running

### Vérification détaillée de MariaDB

```bash
docker service ps glpi-swarm_db
```

Résultat :

- `glpi-swarm_db.1` → Running

Cela confirme que la base de données tourne bien avec **un seul replica**.

---

# ⚠️ Pourquoi MariaDB doit rester à 1 replica ?

Dans cette architecture simple, MariaDB ne doit pas être multipliée sans mécanisme de réplication.

Si on met plusieurs replicas de la base :

- chaque instance possède ses propres données
- les volumes ne sont pas synchronisés
- les requêtes peuvent arriver sur des bases différentes
- les données deviennent incohérentes

Exemple :

```text
Requête 1 → Replica BDD #1
Requête 2 → Replica BDD #2
Requête 3 → Replica BDD #3
```

Chaque replica répond avec son propre état.

C’est pour cela que, dans ce challenge, **MariaDB doit rester en `replicas: 1`**.

---

# ✅ Conclusion

Le challenge est **réussi**.

Nous avons validé :

- Docker Swarm actif
- Déploiement via Portainer
- GLPI en **3 replicas**
- MariaDB en **1 replica**
- Accès à GLPI dans le navigateur
- Simulation de panne réussie
- Recréation automatique d’un conteneur GLPI par Swarm
- Retour à un cluster stable après rééquilibrage

---

# 📝 Remarque importante

Le challenge prévoyait un accès à GLPI via le port `80`, mais dans notre cas ce port était déjà utilisé par un autre service (`web`).

Nous avons donc adapté le déploiement pour utiliser :

```yaml
ports:
  - "8080:80"
```

GLPI était donc accessible via :

```text
http://10.0.0.134:8080
```

Cette adaptation ne remet pas en cause la réussite du challenge, car le comportement attendu de Swarm a bien été démontré.
