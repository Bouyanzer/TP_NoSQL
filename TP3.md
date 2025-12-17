## TP : Le partitionnement (Sharding) sous MongoDB 


## 1. Introduction

Dans les bases de données NoSQL, la gestion de volumes importants de données et de charges élevées constitue un enjeu majeur. Contrairement à la réplication, dont l’objectif principal est d’assurer la tolérance aux pannes et la haute disponibilité, le **sharding** permet de distribuer les données et la charge de travail sur plusieurs nœuds.

Le sharding devient indispensable lorsque :
- le volume de données dépasse la capacité d’un seul serveur ;
- le nombre de requêtes simultanées devient trop important ;
- la montée en charge horizontale est nécessaire.

Dans ce TP, une architecture MongoDB shardée est mise en place. Elle repose sur trois composants essentiels :
- un **serveur de configuration** (*Config Server*) ;
- un **routeur** **mongos** ;
- **deux shards** (*serveur1* et *serveur2*), chacun configuré sous forme de **replica set**.

---

## 2. Architecture du cluster shardé

Le cluster shardé permet à MongoDB de répartir automatiquement les données et la charge entre plusieurs serveurs.

### 2.1 Config Server (CSRS)

Le **Config Server** conserve la **métadonnée de partitionnement** : il maintient les informations permettant de savoir **dans quel fragment (chunk) se trouvent les données**. Il stocke notamment :
- la répartition des chunks ;
- les clés de sharding ;
- les zones de données ;
- l’état des shards.

**Point critique :** si le serveur de configuration tombe en panne, **l’ensemble du cluster devient inopérant**. Pour cette raison, il doit **impérativement être répliqué**, même si le TP se limite à un seul répertoire.

### 2.2 Mongos (Routeur)

Le **mongos** agit comme **point d’entrée unique** pour les clients :
- reçoit les requêtes des applications ;
- consulte les métadonnées stockées dans les config servers ;
- redirige les opérations vers les shards concernés ;
- agrège les résultats avant de les renvoyer au client.

> Remarque du TP : selon la version, le routeur **mongos** peut ou non nécessiter une mise en place particulière (l’énoncé invite à vérifier dans la documentation officielle, car cela a évolué selon les versions).

### 2.3 Shards

Les **shards** contiennent les **données réelles**. Dans ce TP, chaque shard est configuré comme un **replica set**, ce qui permet d’assurer une tolérance aux pannes locale.

---

## 3. Mise en place de l’environnement

### 3.1 Préparation (terminaux + répertoires)

Le TP demande d’ouvrir **six terminaux** pour séparer clairement :
- config server ;
- routeur mongos ;
- initialisation des replica sets ;
- shard 1 ;
- shard 2 ;
- client.

Trois répertoires de stockage sont créés :
- un répertoire pour le config server ;
- un répertoire pour chaque shard.

---

## 4. Démarrage du cluster MongoDB

### 4.1 Démarrage du Config Server

Commande fournie :

```bash
mongod --configsvr --replSet replicaconfig --dbpath configsvrdb --port 27019
```

Explication (issue de l’énoncé) :
- `mongod` : lance le serveur MongoDB ;
- `--configsvr` : indique que ce serveur joue le rôle de **config server** dans un cluster shardé ;
- `--replSet replicaconfig` : nom du replica set du config server ;
- `--dbpath configsvrdb` : répertoire de stockage des données du config server ;
- `--port 27019` : port d’écoute du config server.

> L’énoncé précise qu’il faut **initialiser le replica set**, même si on n’a qu’un seul répertoire.

### 4.2 Démarrage du routeur mongos

Commande fournie :

```bash
mongos --configdb replicaconfig/localhost:27019
```

Explication (issue de l’énoncé) :
- `mongos` : démarre le processus mongos (routeur) ;
- `--configdb replicaconfig/localhost:27019` : indique l’adresse et le replica set des config servers que le mongos utilisera pour obtenir la configuration du cluster.

### 4.3 Démarrage des shards

Commandes fournies :

```bash
mongod --replSet replicashard1 --dbpath serv1/ --shardsvr --port 20004
mongod --replSet replicashard2 --dbpath serv2/ --shardsvr --port 20005
```

Explication (issue de l’énoncé) :
- `--replSet` : nom du replica set pour chaque shard ;
- `--dbpath` : répertoire de stockage des données du shard ;
- `--shardsvr` : indique que le serveur joue le rôle de shard dans un cluster ;
- `--port` : port d’écoute du shard.

### 4.4 Ajout des shards au cluster

Depuis le routeur mongos (commandes fournies) :

```js
sh.addShard("replicashard1/localhost:20004")
sh.addShard("replicashard2/localhost:20005")
```

À ce stade, le cluster shardé est fonctionnel, mais aucune base n’est shardée par défaut.

---

## 5. Activation du sharding

### 5.1 Activation sur la base de données

Commande fournie :

```js
sh.enableSharding("mabasefilms")
```

L’énoncé rappelle que les bases de données MongoDB ne sont pas shardées par défaut : il faut activer explicitement le sharding.

### 5.2 Activation sur la collection

Commande fournie :

```js
sh.shardCollection("mabasefilms.films", { "titre": 1 })
```

Explications (issues de l’énoncé) :
- `sh.shardCollection` : shard une collection ;
- `"mabasefilms.films"` : collection `films` dans la base `mabasefilms` ;
- `{ "titre": 1 }` : clé de partitionnement (shard key) utilisée pour distribuer les documents entre les shards.

---

## 6. Insertion des données et observation

Une fois le cluster en place, l’énoncé demande d’exécuter le programme Python du cours qui insère des films générés aléatoirement. Par défaut, **un million de films** sont insérés **un par un**.

Au fur et à mesure des insertions :
- les chunks sont créés ;
- les chunks trop volumineux sont automatiquement splittés ;
- le **balancer** migre les chunks entre les shards pour équilibrer la charge.

> Remarque de l’énoncé : en production, pour un grand nombre de documents, on privilégie l’insertion en masse (batch) et l’ajustement de la taille des chunks pour optimiser les performances.

---

## 7. Concepts clés (rappel structuré)

### 7.1 Clé de sharding

La **clé de sharding** est le champ choisi pour répartir les documents entre les shards. Son choix influence directement :
- la distribution des données ;
- le routage des requêtes ;
- l’équilibrage de charge.

Critères rappelés (rapport fourni) :
- forte cardinalité ;
- répartition uniforme ;
- usage fréquent dans les requêtes ;
- absence de monotonie.

### 7.2 Hot shard

Un **hot shard** est un shard surchargé, souvent causé par une clé monotone, ce qui concentre les écritures sur un seul shard.

### 7.3 Chunks, splitting, balancer

- **Chunk** : unité logique représentant un intervalle de valeurs de la clé de sharding.  
- **Splitting** : division automatique d’un chunk lorsqu’il dépasse une taille seuil.  
- **Balancer** : processus chargé de redistribuer les chunks pour corriger les déséquilibres.

---

## 8. Stratégies de sharding (rappel)

- **Sharding hashé** : utilisé pour répartir uniformément les insertions séquentielles.  
- **Sharding par plage (ranged)** : privilégié lorsque les requêtes par intervalle sont fréquentes.  
- **Zone sharding** : associe des plages de données à des shards spécifiques.

---

## 9. Commandes usuelles (rappel)

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

## 10. Performances et résilience (rappel)

Les requêtes multi-shards sont envoyées aux shards concernés puis **agrégées par mongos**.

Si un shard devient indisponible, les données qu’il héberge deviennent inaccessibles lorsqu’il n’est pas répliqué. La réplication permet de limiter cet impact.

L’optimisation des performances repose sur :
- une bonne clé de sharding ;
- une indexation adaptée ;
- la limitation des requêtes multi-shards.

---

## 11. Réponses aux questions (1 → 25)

### 1. Qu’est-ce que le sharding dans MongoDB et pourquoi est-il utilisé ?
Le sharding est un mécanisme de **partitionnement** des données qui permet de distribuer les documents et la charge sur plusieurs nœuds. Il est utilisé lorsque le volume de données ou la charge dépasse ce qu’un seul serveur peut gérer.

### 2. Quelle est la différence entre le sharding et la réplication dans MongoDB ?
- La réplication vise la **tolérance aux pannes** et la **haute disponibilité**.  
- Le sharding vise la **distribution des données** et la **montée en charge**.

### 3. Quels sont les composants d’une architecture shardée (mongos, config servers, shards) ?
- **Config servers** : stockent les métadonnées de sharding ;  
- **mongos** : routeur, point d’entrée des clients ;  
- **shards** : serveurs qui stockent les données (souvent en replica set).

### 4. Quelles sont les responsabilités des config servers (CSRS) dans un cluster shardé ?
Les config servers stockent la **métadonnée de partitionnement** : notamment la répartition des chunks, les clés de sharding, les zones et l’état des shards. Ils sont indispensables au fonctionnement du cluster.

### 5. Quel est le rôle du mongos router ?
Le mongos reçoit les requêtes, consulte la configuration auprès des config servers, **redirige** vers les shards concernés et **agrège** les résultats.

### 6. Comment MongoDB décide-t-il sur quel shard stocker un document ?
La distribution dépend de la **clé de sharding** (shard key) définie sur la collection. Cette clé permet d’associer les documents à des chunks, eux-mêmes rattachés à un shard.

### 7. Qu’est-ce qu’une clé de sharding et pourquoi est-elle essentielle ?
La clé de sharding est le champ choisi pour répartir les documents entre shards. Elle est essentielle car elle détermine la distribution des données, le routage des requêtes et l’équilibrage.

### 8. Quels sont les critères de choix d’une bonne clé de sharding ?
Une bonne clé :
- a une forte cardinalité ;
- répartit uniformément les documents ;
- est utilisée fréquemment dans les requêtes ;
- évite la monotonie.

### 9. Qu’est-ce qu’un chunk dans MongoDB ?
Un chunk est une unité logique qui représente un intervalle (lié à la clé de sharding) et qui regroupe une partie des données shardées.

### 10. Comment fonctionne le splitting des chunks ?
Lorsqu’un chunk dépasse une taille seuil, il est automatiquement divisé (splitté) en plusieurs chunks plus petits.

### 11. Que fait le balancer dans un cluster shardé ?
Le balancer migre des chunks entre shards pour **équilibrer la charge** et corriger les déséquilibres.

### 12. Quand et comment le balancer déplace-t-il des chunks ?
Le balancer déplace des chunks lorsqu’il observe un déséquilibre de répartition (charge/données) entre shards, en migrant des chunks d’un shard vers un autre.

### 13. Qu’est-ce qu’un hot shard et comment l’éviter ?
Un hot shard est un shard surchargé. Il apparaît typiquement quand la clé de sharding est monotone. On l’évite en choisissant une clé mieux répartie (ex. en s’appuyant sur des stratégies comme le sharding hashé).

### 14. Quels problèmes une clé de sharding monotone peut-elle engendrer ?
Une clé monotone concentre les écritures sur une même zone de la clé, ce qui peut concentrer la charge sur un shard (hot shard) et créer des déséquilibres.

### 15. Comment activer le sharding sur une base de données et sur une collection ?
- Sur la base :
```js
sh.enableSharding("mabasefilms")
```
- Sur la collection :
```js
sh.shardCollection("mabasefilms.films", { "titre": 1 })
```

### 16. Comment ajouter un nouveau shard à un cluster MongoDB ?
Depuis mongos :
```js
sh.addShard("replicaSet/host:port");
```

### 17. Comment vérifier l’état du cluster shardé (commandes usuelles) ?
Commandes usuelles :
```js
sh.status();
db.stats();
db.collection.stats();
```

### 18. Dans quels cas faut-il envisager d’utiliser un hashed sharding key ?
Quand on souhaite une répartition uniforme, notamment pour des insertions séquentielles.

### 19. Dans quels cas faut-il privilégier un ranged sharding key ?
Quand les requêtes par intervalle (plages) sont fréquentes.

### 20. Qu’est-ce que le zone sharding et quel est son intérêt ?
Le zone sharding permet d’associer des plages de données à des shards spécifiques, pour mieux contrôler la distribution.

### 21. Comment MongoDB gère-t-il les requêtes multi-shards ?
Les requêtes multi-shards sont envoyées aux shards concernés puis les résultats sont agrégés par mongos avant d’être renvoyés.

### 22. Comment optimiser les performances de requêtes dans un environnement shardé ?
Les optimisations reposent sur :
- une bonne clé de sharding ;
- une indexation adaptée ;
- la limitation des requêtes multi-shards.

### 23. Que se passe-t-il lorsqu’un shard devient indisponible ?
Les données de ce shard deviennent inaccessibles si elles ne sont pas disponibles via réplication. La réplication limite l’impact en assurant une tolérance aux pannes locale.

### 24. Comment migrer une collection existante vers un schéma shardé ?
La logique consiste à activer le sharding sur la base, puis à shard(er) la collection avec `sh.shardCollection(...)`. La distribution et l’équilibrage se feront ensuite via chunks/splitting/balancer.

### 25. Quels outils ou métriques utiliser pour diagnostiquer les problèmes de sharding ?
Commandes de diagnostic usuelles :
- `sh.status()` (état du cluster shardé) ;
- `db.stats()` ;
- `db.collection.stats()`.

---

## 12. Conclusion

Ce TP met en évidence :
- l’importance du rôle central des **config servers** (métadonnées de sharding) ;
- le rôle de **mongos** (routage et agrégation) ;
- les mécanismes **chunks / splitting / balancer** ;
- l’impact crucial du choix de la **clé de sharding** sur la distribution et les performances.

En observant l’insertion d’un grand volume de documents, on visualise concrètement la création des chunks, leur découpage, et leur migration automatique entre shards pour équilibrer la charge.
