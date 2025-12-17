
## 1. Introduction au NoSQL et à Redis

### Qu'est-ce que le NoSQL ?
Les bases de données **NoSQL** (Not Only SQL) sont des alternatives aux bases relationnelles classiques, conçues pour la flexibilité, la scalabilité horizontale et le traitement de données non structurées.

**Les types de bases NoSQL :**
* **Clé-Valeur :** Données stockées par paires (ex: Redis, DynamoDB). Idéal pour le caching.
* **Colonnes :** Organisation en colonnes (ex: Cassandra, HBase). Pour l'analyse Big Data.
* **Documents :** Documents structurés comme JSON/XML (ex: MongoDB). Pour le web/mobile.
* **Graphes :** Gestion des relations complexes (ex: Neo4j).

### Qu'est-ce que Redis ?
**Redis** (Remote Dictionary Server) est une base de données NoSQL de type **Clé-Valeur** fonctionnant **en mémoire**.
Elle est utilisée pour :
* La mise en cache (hautes performances).
* La gestion de sessions.
* Les files d'attente et la messagerie temps réel.
* Les classements (Leaderboards).

---

## 2. Installation et Configuration

### Installation (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install redis-server
````

### Démarrage

Lancer le serveur Redis :

```bash
redis-server
```

*Note : Par défaut, Redis écoute sur `127.0.0.1:6379`.*

### Connexion au client (CLI)

Pour interagir avec le serveur, ouvrez un nouveau terminal :

```bash
redis-cli
```

-----

## 3\. Les Bases : Manipulations Clés-Valeurs (CRUD)

Une fois dans le `redis-cli`, vous pouvez manipuler les données.

### Opérations élémentaires

  * **Créer une clé** : `SET clé valeur`
    ```redis
    SET utilisateur "Alice"
    ```
  * **Lire une clé** : `GET clé`
    ```redis
    GET utilisateur
    # Résultat : "Alice"
    ```
  * **Vérifier l'existence** : `EXISTS clé` (Retourne 1 si existe, 0 sinon).
  * **Supprimer une clé** : `DEL clé`

### Compteurs (Incrémentation)

Utile pour les statistiques ou les vues de pages.

```redis
SET compteur 0
INCR compteur      # Résultat : 1
DECR compteur      # Résultat : 0
```

### Gestion de la durée de vie (TTL)

Redis permet de définir une expiration pour les clés (cache temporaire).

  * **Définir une expiration** (en secondes) :
    ```redis
    EXPIRE utilisateur 120
    ```
  * **Vérifier le temps restant** :
    ```redis
    TTL utilisateur
    ```
      * *Retourne -1* : Pas d'expiration (permanent).
      * *Retourne \> 0* : Secondes restantes.

-----

## 4\. Structures de Données Avancées

Redis n'est pas limité aux chaînes de caractères. Il gère des structures complexes.

### 4.1 Listes (Lists)

Séquences ordonnées d'éléments (permet les doublons). Idéal pour les files d'attente ou historiques.

  * **Ajouter** (`LPUSH` à gauche, `RPUSH` à droite) :
    ```redis
    LPUSH ma_liste "NoSQL"
    RPUSH ma_liste "Redis"
    ```
  * **Lire** (extraire une plage) :
    ```redis
    LRANGE ma_liste 0 -1  # Affiche tout
    ```
  * **Supprimer/Récupérer** :
    ```redis
    LPOP ma_liste  # Retire le premier élément
    RPOP ma_liste  # Retire le dernier élément
    ```

### 4.2 Ensembles (Sets)

Collections d'éléments **uniques** non ordonnés. Idéal pour les tags ou IP uniques.

  * **Ajouter** :
    ```redis
    SADD mon_set "Alice" "Bob" "Alice"
    # "Alice" ne sera ajouté qu'une seule fois.
    ```
  * **Lister le contenu** :
    ```redis
    SMEMBERS mon_set
    ```
  * **Vérifier la présence** :
    ```redis
    SISMEMBER mon_set "Alice"
    ```
  * **Opérations mathématiques** (Union) :
    ```redis
    SUNION set1 set2
    ```

### 4.3 Ensembles Ordonnés (Sorted Sets)

Comme les Sets (uniques), mais chaque élément a un **score**. Idéal pour les classements.

  * **Ajouter avec score** :
    ```redis
    ZADD classement 100 "Alice" 200 "Bob"
    ```
  * **Lire (Tri croissant)** :
    ```redis
    ZRANGE classement 0 -1 WITHSCORES
    ```
  * **Lire (Tri décroissant)** :
    ```redis
    ZREVRANGE classement 0 -1 WITHSCORES
    ```
  * **Connaître le rang** :
    ```redis
    ZRANK classement "Alice"
    ```

### 4.4 Hashes

Stockage d'objets structurés (champs et valeurs). Parfait pour les profils utilisateurs.

  * **Créer un hash** :
    ```redis
    HSET utilisateur:1 nom "Alice" age 30 email "alice@test.com"
    ```
  * **Lire tout le hash** :
    ```redis
    HGETALL utilisateur:1
    ```
  * **Incrémenter un champ spécifique** :
    ```redis
    HINCRBY utilisateur:1 age 1
    ```

-----

## 5\. Fonctionnalités Avancées : Pub/Sub

Le modèle **Publish/Subscribe** permet la messagerie en temps réel. Les émetteurs (publishers) envoient des messages dans des canaux sans connaître les destinataires (subscribers).

### Scénario de test

Ouvrez deux terminaux différents.

**Terminal 1 (L'abonné) :**
Il écoute le canal "news".

```redis
SUBSCRIBE news
```

**Terminal 2 (Le publieur) :**
Il envoie un message dans le canal "news".

```redis
PUBLISH news "Bonjour tout le monde !"
```

*Résultat : Le Terminal 1 reçoit instantanément le message.*

### Astuces Pub/Sub

  * **Canal spécifique utilisateur** : `PUBLISH user:1 "Message privé"`
  * **Abonnement par motif** : `PSUBSCRIBE sport*` (Reçoit les messages de `sport:foot`, `sport:tennis`, etc.)

-----

## 6\. Administration et Gestion des Bases

### Bases de données multiples

Par défaut, Redis propose 16 bases de données isolées (numérotées 0 à 15).

  * **Changer de base** :
    ```redis
    SELECT 1
    ```
  * *Note : Les clés créées dans la DB 0 ne sont pas visibles dans la DB 1.*

### Nettoyage

⚠️ *Commandes dangereuses en production.*

  * **Vider la base actuelle** : `FLUSHDB`
  * **Vider TOUTES les bases** : `FLUSHALL`

### Exploration

  * **Lister toutes les clés** : `KEYS *`

-----

## 7\. Persistance et Performance

Redis fonctionne en mémoire (RAM), ce qui garantit une latence très faible, mais pose la question de la sauvegarde des données en cas de redémarrage.

### Méthodes de Persistance

1.  **Snapshots (RDB - Redis Database) :**

      * Sauvegarde périodique de l'état de la mémoire sur le disque.
      * Commandes : `SAVE` (immédiat, bloquant) ou `BGSAVE` (arrière-plan).
      * *Avantage* : Peu d'impact CPU.
      * *Inconvénient* : Perte possible des dernières données entre deux snapshots.

2.  **Append-Only File (AOF) :**

      * Journalisation de chaque opération d'écriture.
      * Configuration : `CONFIG SET appendonly yes`.
      * *Avantage* : Meilleure durabilité (récupération précise).
      * *Inconvénient* : Fichiers plus gros, peut ralentir les performances.

### Conclusion

Redis est un outil indispensable pour les applications modernes nécessitant vitesse et temps réel. Sa maîtrise passe par la compréhension de ses structures de données et le choix approprié de la stratégie de persistance selon les besoins du projet.

```
#  TP1 - Partie 2: Prise en main de MongoDB
---

## 1. Prérequis & Installation Docker

Lancement du conteneur MongoDB :

```bash
docker run --name mongodb -d -p 27017:27017 -v $(pwd)/data:/data/db mongo:latest
```

Vérifier que le conteneur tourne :
```bash
docker ps
```

## 2. Jeu de données 2 : Base sample_mflix (BSON)

### 2.1 Importation des données
Télécharger et restaurer l'archive BSON :

```bash
# Téléchargement
curl [https://atlas-education.s3.amazonaws.com/sampledata.archive](https://atlas-education.s3.amazonaws.com/sampledata.archive) -o sampledata.archive

# Restauration (mongorestore est nécessaire pour le BSON)
mongorestore --archive=sampledata.archive --port=27017
```

### 2.2 Partie 1 : Filtrage et Projections

**1. 5 films sortis depuis 2015**
```javascript
db.movies.find({ year: { $gte: 2015 } }).limit(5)
```

**2. Films du genre "Comedy"**
```javascript
db.movies.find({ genres: "Comedy" })
```

**3. Films entre 2000 et 2005**
```javascript
db.movies.find(
    { year: { $gte: 2000, $lte: 2005 } },
    { title: 1, year: 1 }
)
```

**4. Films "Drama" ET "Romance"**
```javascript
db.movies.find({ genres: { $all: ["Drama", "Romance"] } })
```

**5. Films sans champ `rated`**
```javascript
db.movies.find({ rated: { $exists: false } }, { title: 1 })
```

### 2.3 Partie 2 : Pipeline d'Agrégation

**6. Nombre de films par année**
```javascript
db.movies.aggregate([
    { $group: { _id: "$year", total: { $sum: 1 } } },
    { $sort: { _id: 1 } }
])
```

**7. Moyenne IMDb par genre**
```javascript
db.movies.aggregate([
    { $unwind: "$genres" },
    { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" } } },
    { $sort: { moyenne: -1 } }
])
```

**8. Nombre de films par pays**
```javascript
db.movies.aggregate([
    { $unwind: "$countries" },
    { $group: { _id: "$countries", total: { $sum: 1 } } },
    { $sort: { total: -1 } }
])
```

**9. Top 5 réalisateurs**
```javascript
db.movies.aggregate([
    { $unwind: "$directors" },
    { $group: { _id: "$directors", total: { $sum: 1 } } },
    { $sort: { total: -1 } },
    { $limit: 5 }
])
```

**10. Films triés par note IMDb (Projection après tri)**
```javascript
db.movies.aggregate([
    { $sort: { "imdb.rating": -1 } },
    { $project: { title: 1, "imdb.rating": 1 } }
])
```

### 2.4 Partie 3 : Mises à jour (Updates)

**11. Ajouter un champ `etat`**
```javascript
db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" } })
```

**12. Incrémenter les votes IMDb (+100)**
```javascript
db.movies.updateOne({ title: "Inception" }, { $inc: { "imdb.votes": 100 } })
```

**13. Supprimer le champ `poster` pour tous les films**
```javascript
db.movies.updateMany({}, { $unset: { poster: "" } })
```

**14. Modifier le réalisateur**
```javascript
db.movies.updateOne(
    { title: "Titanic" },
    { $set: { directors: ["James Cameron"] } }
)
```

### 2.5 Partie 4 : Requêtes Complexes

**15. Films les mieux notés par décennie**
*Utilisation de l'arithmétique pour grouper par tranches de 10 ans.*
```javascript
db.movies.aggregate([
    { $match: { "imdb.rating": { $exists: true } } },
    { $project: {
        title: 1,
        decade: { $subtract: ["$year", { $mod: ["$year", 10] }] },
        "imdb.rating": 1
    }},
    { $group: { _id: "$decade", maxRating: { $max: "$imdb.rating" } } },
    { $sort: { _id: 1 } }
])
```

**16. Titres commençant par "Star" (Regex)**
```javascript
db.movies.find({ title: /^Star/ }, { title: 1 })
```

**17. Films avec plus de 2 genres ($where)**
```javascript
db.movies.find({ $where: "this.genres.length > 2" }, { title: 1, genres: 1 })
```

**18. Films de Christopher Nolan**
```javascript
db.movies.find(
    { directors: "Christopher Nolan" },
    { title: 1, year: 1, "imdb.rating": 1 }
)
```

### 2.6 Partie 5 : Indexation & Performance

**19. Créer un index sur l'année**
```javascript
db.movies.createIndex({ year: 1 })
```

**20. Vérifier les index existants**
```javascript
db.movies.getIndexes()
```

**21. Analyser la performance (Explain)**
*Comparer le "totalDocsExamined" et "executionTimeMillis" avec et sans index.*
```javascript
db.movies.find({ year: 1995 }).explain("executionStats")
```

**22. Supprimer un index**
```javascript
db.movies.dropIndex({ year: 1 })
```

**23. Créer un index composé (Année + Note)**
```javascript
db.movies.createIndex({ year: 1, "imdb.rating": -1 })
```

---
---

## 3. Jeu de données 1 : Base lesfilms (JSON)

### 3.1 Importation des données
Copier le fichier `films.json` dans le conteneur et l'importer :

```bash
docker cp films.json mongodb:/films.json
docker exec -it mongodb mongoimport --db lesfilms --collection films --file /films.json --jsonArray
```

### 3.2 Requêtes de consultation

**1. Vérification et structure d'un document**
```javascript
db.films.count()
db.films.findOne()
```

**2. Films d'action**
```javascript
db.films.find({ genre: "Action" })
db.films.count({ genre: "Action" })
```

**3. Films d'action en France (1963)**
```javascript
db.films.find({ genre: "Action", country: "FR" })
db.films.find({ genre: "Action", country: "FR", year: 1963 })
```

**4. Projections (Affichage sélectif)**
```javascript
// Sans les grades
db.films.find({ genre: "Action", country: "FR" }, { grades: 0 })
// Sans l'identifiant (_id)
db.films.find({ genre: "Action", country: "FR" }, { _id: 0 })
// Titres + Grades uniquement
db.films.find({ genre: "Action", country: "FR" }, { _id: 0, title: 1, grades: 1 })
```

**5. Recherche sur les notes (Tableaux)**
```javascript
// Au moins une note > 10
db.films.find(
    { "grades.grade": { $gt: 10 } },
    { _id: 0, title: 1, grades: 1 }
)

// QUE des notes > 10 (Strictement toutes)
db.films.find(
    { grades: { $not: { $elemMatch: { note: { $lte: 10 } } } } },
    { _id: 0, title: 1, grades: 1 }
)
```

**6. Requêtes diverses**

**Afficher les différents genres présents**
```javascript
db.films.distinct("genre")
```

**Afficher les différents grades attribués**
```javascript
db.films.distinct("grades.note")
```

**Films avec artistes spécifiques (exemple avec liste d'IDs)**
```javascript
ddb.films.find({
  artists: { $in: ["artist:4", "artist:18", "artist:11"] }
})
```

**Films sans résumé**
```javascript
db.films.find({ summary: { $exists: false } })
```

**Films avec Leonardo DiCaprio en 1997**
```javascript
db.films.find({
    "actors.first_name": "Leonardo",
    "actors.last_name": "DiCaprio",
    year: 1997
})
```

**Films avec DiCaprio OU en 1997**
```javascript
db.films.find({
    $or: [
        { "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio" },
        { year: 1997 }
    ]
})
```

---


## 4. Nettoyage

Arrêter et supprimer le conteneur à la fin du TP :

```bash
docker stop mongodb
docker rm mongodb
```

```
