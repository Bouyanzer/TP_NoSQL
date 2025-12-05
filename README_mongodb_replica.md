
# Rapport de TP — Mise en place d’un Replica Set MongoDB

## 🎯 1. Objectif du TP

Ce TP a pour but d’apprendre à :

- Mettre en place un **cluster MongoDB** utilisant un **replica set**.  
- Comprendre :
  - la réplication **Primary / Secondary**,
  - la réaction du cluster en cas de **panne** d'un nœud,
  - les règles de **lecture** depuis un secondary,
  - l’ajout et l’utilité d’un **arbitre** (*arbiter*).

Toutes les manipulations sont réalisées localement en simulant plusieurs serveurs à l’aide de **ports différents** et **répertoires séparés**.

---

## 🧠 2. Rappels théoriques sur la réplication MongoDB

### 2.1. Communication entre nœuds  
Les nœuds échangent constamment des messages (heartbeats, synchronisation…), permettant :

- la réplication des données,  
- la détection des pannes,  
- la cohérence globale du cluster.

---

### 2.2. Panne d’un Secondary  
Si un secondary tombe :

- le primary détecte l’absence de heartbeat,  
- il marque le nœud comme inactif,  
- la charge est répartie entre les autres nœuds.

---

### 2.3. Panne du Primary et élection  
Si le primary tombe :

- une **élection automatique** se déclenche,  
- les secondary négocient pour élire un nouveau primary,  
- ceci repose sur des algorithmes de consensus (Raft, Paxos…).

---

### 2.4. Problème de partition réseau (split-brain)  
Si le cluster est coupé en deux groupes isolés :

- les deux parties pourraient élire un primary → incohérence,  
- MongoDB empêche cela via la **règle de majorité** :  
  **seul le groupe majoritaire peut élire un primary.**

---

### 2.5. Architecture Primary / Secondary  
Règles par défaut :

- **Écriture → uniquement sur le primary**,  
- **Lecture → sur le primary** (cohérence forte),  
- Lectures possibles sur secondary → risque de données obsolètes.

---

### 2.6. Réplication asynchrone et journal  
Processus :

1. Écriture du client sur le primary,  
2. Stockage dans le **journal (log)**,  
3. Accusé de réception,  
4. Réplication vers les secondary ensuite.

---

## ⚙️ 3. Mise en place du Replica Set

### 3.1. Paramètres utilisés

| Élément | Valeur |
|--------|--------|
| Nom du replica set | `monReplicaSet` |
| Ports | 27018, 27019, 27020 |
| Répertoires | disque1, disque2, disque3 |

---

### 3.2. Création des répertoires

```bash
mkdir disque1 disque2 disque3
```

---

### 3.3. Démarrage des 3 serveurs MongoDB

```bash
mongod --replSet monReplicaSet --port 27018 --dbpath disque1
mongod --replSet monReplicaSet --port 27019 --dbpath disque2
mongod --replSet monReplicaSet --port 27020 --dbpath disque3
```

---

## 🚀 4. Initialisation du Replica Set

### 4.1. Connexion au premier nœud

```bash
mongo --port 27018
```

### 4.2. Initialisation

```javascript
rs.initiate()
```

### 4.3. Ajout des autres membres

```javascript
rs.add("localhost:27019")
rs.add("localhost:27020")
```

---

## 📊 5. Inspection du Replica Set

### 5.1. Configuration (statique)

```javascript
rs.config()
```

### 5.2. État en temps réel (dynamique)

```javascript
rs.status()
```

### 5.3. Vérifier si on est connecté au primary

```javascript
db.isMaster()
```

---

## 🧪 6. Manipulations de données

### 6.1. Sur le primary (27018)

```bash
mongo --port 27018
```

#### Création base + collection :

```javascript
use demo1
db.createCollection("person")
```

#### Insertion :

```javascript
db.person.insert({ nom: "Dupont" })
db.person.insert({ nom: "Durand" })
db.person.insert({ nom: "Codard" })
```

---

### 6.2. Sur un secondary (27019)

Connexion :

```bash
mongo --port 27019
```

#### Lecture non autorisée :

```
not master and slaveOk=false
```

#### Autoriser la lecture :

```javascript
rs.slaveOk()
db.person.find()
```

#### Écriture impossible :

```javascript
db.person.insert({ nom: "Martin" })
// not master
```

---

## 🔥 7. Simulation d’une panne du Primary

### 7.1. Arrêt du primary

```bash
Ctrl + C
```

### 7.2. Nouveau primary

```bash
mongo --port 27019
rs.status()
db.person.find()
```

---

## 🛡️ 8. Ajout d’un arbitre

### 8.1. Répertoire :

```bash
mkdir arbitre1
```

### 8.2. Démarrage :

```bash
mongod --replSet monReplicaSet --port 27021 --dbpath arbitre1
```

### 8.3. Ajout au Replica Set :

```javascript
rs.addArb("localhost:27021")
```

---

## ✔️ 10. Conclusion

Ce TP permet de comprendre :

- le fonctionnement d’un replica set MongoDB,  
- la réplication asynchrone,  
- la gestion de panne et l’élection automatique,  
- l’usage d’un arbitre pour maintenir la majorité.

MongoDB fournit ainsi haute disponibilité et tolérance aux pannes.
