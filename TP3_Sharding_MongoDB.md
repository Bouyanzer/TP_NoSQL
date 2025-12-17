# TP3 – Exploration NoSQL
## Le partitionnement (Sharding) sous MongoDB

## 1. Introduction

Dans les bases de données NoSQL, la gestion de volumes importants de données et de charges élevées constitue un enjeu majeur. Contrairement à la **réplication**, dont l’objectif principal est d’assurer la tolérance aux pannes et la haute disponibilité, le **sharding** permet de **distribuer les données et la charge de travail sur plusieurs nœuds**.

Le sharding devient indispensable lorsque :
- le volume de données dépasse la capacité d’un seul serveur,
- le nombre de requêtes simultanées devient trop important,
- la montée en charge horizontale est nécessaire.

Dans ce TP, une architecture MongoDB shardée est mise en place. Elle repose sur trois composants essentiels :
- un **serveur de configuration (Config Server)**,
- un **routeur mongos**,
- deux **shards**, chacun configuré sous forme de **replica set**.

---

## 2. Architecture du cluster shardé

### 2.1 Config Server (CSRS)

Le **Config Server** stocke l’ensemble des **métadonnées du cluster shardé** :
- répartition des chunks,
- clés de sharding,
- zones de données,
- état des shards.

Il constitue un **point critique** du système et doit obligatoirement être répliqué.

### 2.2 Mongos (Routeur)

Le **mongos** agit comme un **point d’entrée unique** pour les clients. Il :
- reçoit les requêtes,
- consulte les métadonnées,
- redirige les opérations vers les shards concernés,
- agrège les résultats.

### 2.3 Shards

Les shards contiennent les données réelles. Chaque shard est configuré comme un **replica set**, assurant une tolérance aux pannes locale.

---

## 3. Mise en place de l’environnement

Six terminaux sont ouverts afin de séparer les rôles :
- config server,
- routeur mongos,
- initialisation des replica sets,
- shard 1,
- shard 2,
- client.

Trois répertoires de stockage sont créés :
- un pour le config server,
- un pour chaque shard.

---

## 4. Démarrage du cluster MongoDB

### 4.1 Config Server

```bash
mongod --configsvr --replSet replicaconfig --dbpath configsvrdb --port 27019
```

### 4.2 Routeur mongos

```bash
mongos --configdb replicaconfig/localhost:27019
```

### 4.3 Shards

```bash
mongod --replSet replicashard1 --dbpath serv1/ --shardsvr --port 20004
mongod --replSet replicashard2 --dbpath serv2/ --shardsvr --port 20005
```

### 4.4 Ajout des shards

```js
sh.addShard("replicashard1/localhost:20004");
sh.addShard("replicashard2/localhost:20005");
```

---

## 5. Activation du sharding

```js
sh.enableSharding("mabasefilms");
sh.shardCollection("mabasefilms.films", { "titre": 1 });
```

---

## 6. Insertion des données et observation

Un programme Python permet l’insertion jusqu’à **un million de films**.  
Au cours de l’insertion :
- création des chunks,
- splitting automatique,
- migration par le balancer.

---

## 7. Concepts clés

### Clé de sharding
- forte cardinalité
- répartition uniforme
- usage fréquent
- absence de monotonie

### Chunk
Un chunk est une plage de valeurs de la clé de sharding.

### Balancer
Il équilibre automatiquement les chunks entre les shards.

---

## 8. Stratégies de sharding

- **Hashed sharding**
- **Range sharding**
- **Zone sharding**

---

## 9. Commandes utiles

```js
sh.status();
db.stats();
db.collection.stats();
```

---

## 10. Conclusion

Ce TP met en évidence le rôle du sharding dans la scalabilité de MongoDB et l’importance d’un bon choix de clé de sharding.
