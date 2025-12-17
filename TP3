# Le partitionnement (Sharding) sous MongoDB


## 1. Introduction

Dans les bases de données NoSQL, la gestion de volumes importants de données et de charges élevées constitue un enjeu majeur. Contrairement à la réplication, dont l’objectif principal est d’assurer la tolérance aux pannes et la haute disponibilité, le sharding permet de distribuer les données et la charge de travail sur plusieurs nœuds.

Le sharding devient indispensable lorsque :
- le volume de données dépasse la capacité d’un seul serveur
- le nombre de requêtes simultanées devient trop important
- la montée en charge horizontale est nécessaire

Dans ce TP, une architecture MongoDB shardée est mise en place. Elle repose sur trois composants essentiels :
- un serveur de configuration (Config Server)
- un routeur mongos
- deux shards, chacun configuré sous forme de replica set

---

## 2. Architecture du cluster shardé

L’architecture globale du cluster shardé permet à MongoDB de répartir automatiquement les données et la charge entre plusieurs serveurs.

### 2.1 Config Server (CSRS)

Le Config Server stocke l’ensemble des métadonnées du cluster shardé, notamment :
- la répartition des chunks
- les clés de sharding
- les zones de données
- l’état des shards

Il constitue un point critique du système. Sans les config servers, le cluster devient totalement inutilisable. Pour cette raison, ils doivent obligatoirement être répliqués, même si un seul nœud est utilisé dans le cadre du TP.

### 2.2 Mongos (Routeur)

Le mongos agit comme un point d’entrée unique pour les clients. Il :
- reçoit les requêtes des applications
- consulte les métadonnées stockées dans les config servers
- redirige les opérations vers les shards concernés
- agrège les résultats avant de les renvoyer au client

### 2.3 Shards

Les shards contiennent les données réelles. Dans ce TP, chaque shard est configuré comme un replica set, ce qui permet d’assurer une tolérance aux pannes locale.

Cette architecture permet à MongoDB de distribuer automatiquement les données et d’équilibrer la charge entre les shards.

---

## 3. Mise en place de l’environnement

### 3.1 Préparation

Six terminaux sont ouverts afin de séparer clairement les rôles suivants :
- config server
- routeur mongos
- initialisation des replica sets
- shard 1
- shard 2
- client

Trois répertoires de stockage sont créés :
- un répertoire pour le config server
- un répertoire pour chaque shard

---

## 4. Démarrage du cluster MongoDB

### 4.1 Démarrage du Config Server

```bash
mongod --configsvr --replSet replicaconfig --dbpath configsvrdb --port 27019
```

Même avec un seul nœud, le config server est initialisé comme un replica set, conformément aux exigences de MongoDB.

### 4.2 Démarrage du routeur mongos

```bash
mongos --configdb replicaconfig/localhost:27019
```

Le mongos se connecte au config server afin de récupérer les informations de configuration du cluster shardé.

### 4.3 Démarrage des shards

```bash
mongod --replSet replicashard1 --dbpath serv1/ --shardsvr --port 20004
mongod --replSet replicashard2 --dbpath serv2/ --shardsvr --port 20005
```

Chaque shard dispose :
- d’un replica set dédié
- d’un répertoire de données propre
- d’un rôle explicite de shard server

### 4.4 Ajout des shards au cluster

Depuis le routeur mongos :

```js
sh.addShard("replicashard1/localhost:20004");
sh.addShard("replicashard2/localhost:20005");
```

À ce stade, le cluster shardé est fonctionnel, mais aucune base de données n’est encore partitionnée.

---

## 5. Activation du sharding

### 5.1 Activation sur la base de données

```js
sh.enableSharding("mabasefilms");
```

Par défaut, MongoDB ne shard aucune base sans action explicite de l’administrateur.

### 5.2 Activation sur la collection

```js
sh.shardCollection("mabasefilms.films", { "titre": 1 });
```

La clé de sharding détermine :
- la distribution des documents
- le routage des requêtes
- l’équilibrage de la charge

---

## 6. Insertion des données et observation

Un programme Python fourni par le cours est exécuté afin d’insérer un grand nombre de documents (jusqu’à un million de films).

Chaque document est inséré individuellement. Au fur et à mesure des insertions :
- les chunks sont créés
- les chunks trop volumineux sont automatiquement splittés
- le balancer migre les chunks entre les shards afin d’équilibrer la charge

---

## 7. Concepts clés du sharding

### 7.1 Clé de sharding

La clé de sharding est le champ utilisé pour répartir les documents entre les shards. Son choix est essentiel pour garantir l’équilibre et les performances.

Critères d’une bonne clé de sharding :
- forte cardinalité
- répartition uniforme
- usage fréquent dans les requêtes
- absence de monotonie

### 7.2 Hot shard

Un hot shard est un shard surchargé. Il apparaît généralement lorsque la clé de sharding est monotone, ce qui concentre les écritures sur un seul shard.

### 7.3 Chunks et équilibrage

- **Chunk** : unité logique représentant un intervalle de valeurs de la clé de sharding  
- **Splitting** : division automatique d’un chunk lorsqu’il dépasse une taille seuil  
- **Balancer** : processus chargé de redistribuer les chunks pour corriger les déséquilibres  

---

## 8. Stratégies de sharding

- **Sharding hashé** : utilisé pour répartir uniformément les insertions séquentielles  
- **Sharding par plage** : privilégié lorsque les requêtes par intervalle sont fréquentes  
- **Zone sharding** : permet d’associer des plages de données à des shards spécifiques  

---

## 9. Commandes usuelles de gestion

Ajouter un shard :
```js
sh.addShard("replicaSet/host:port");
```

Activer le sharding sur une base :
```js
sh.enableSharding("nomBase");
```

Sharder une collection :
```js
sh.shardCollection("nomBase.collection", { champ: 1 });
```

Vérifier l’état du cluster :
```js
sh.status();
db.stats();
db.collection.stats();
```

---

## 10. Performances et résilience

Les requêtes multi-shards sont envoyées aux shards concernés puis agrégées par le mongos.

Si un shard devient indisponible, les données qu’il héberge deviennent inaccessibles lorsqu’il n’est pas répliqué. La réplication permet de limiter cet impact.

L’optimisation des performances repose sur :
- une bonne clé de sharding
- une indexation adaptée
- la limitation des requêtes multi-shards

---

## 11. Conclusion

Ce TP permet de comprendre en détail l’architecture et les mécanismes du sharding sous MongoDB. Il met en évidence l’importance du choix de la clé de sharding, du rôle central des config servers et du balancer, ainsi que les impacts du partitionnement sur la performance et la scalabilité du système.
