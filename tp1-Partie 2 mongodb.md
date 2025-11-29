#  TP Prise en main de MongoDB
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
