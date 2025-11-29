📑 Table des Matières
Introduction à MongoDB et au NoSQL

Installation et Configuration avec Docker

Importation des Données

Requêtes sur la Base lesfilms

Requêtes sur la Base sample_mflix

Agrégations MongoDB

Opérations de Mise à Jour

Indexation et Analyse de Performances

Arrêt du Serveur MongoDB

Conclusion

1. Introduction à MongoDB et au NoSQL
MongoDB est une base de données NoSQL orientée documents. Elle stocke les informations sous forme de documents JSON/BSON et offre flexibilité, performance et scalabilité horizontale.

Ce TP permet d’apprendre :

Le modèle documentaire

Les requêtes simples et avancées

Le pipeline d’agrégation

Les opérations CRUD

L’indexation

L’usage de Docker pour héberger MongoDB

Deux bases sont utilisées :

lesfilms (fichier JSON)

sample_mflix (archive BSON)

2. Installation et Configuration avec Docker
Lancer MongoDB
Bash

docker run --name mongodb -d -p 27017:27017 -v $(pwd)/data:/data/db mongo:latest
Vérifier l’état du conteneur
Bash

docker ps
3. Importation des Données
3.1 Importer la base lesfilms
Copier le fichier JSON dans le conteneur :

Bash

docker cp films.json mongodb:/films.json
Importer les données :

Bash

mongoimport --db lesfilms --collection films --file /films.json --jsonArray
3.2 Importer la base sample_mflix (BSON)
Télécharger l’archive :

Bash

curl https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive
Importer :

Bash

mongorestore --archive=sampledata.archive --port=27017
4. Requêtes sur la Base lesfilms
Vérification des données
JavaScript

db.films.count()
db.films.findOne()
Films d’action
JavaScript

db.films.find({ genre: "Action" })
db.films.count({ genre: "Action" })
db.films.find({ genre: "Action", country: "FR" })
db.films.find({ genre: "Action", country: "FR", year: 1963 })
Projections
Sans grades :

JavaScript

db.films.find({ genre: "Action", country: "FR" }, { grades: 0 })
Sans identifiant :

JavaScript

db.films.find({ genre: "Action", country: "FR" }, { _id: 0 })
Titres + grades :

JavaScript

db.films.find(
  { genre: "Action", country: "FR" },
  { _id: 0, title: 1, grades: 1 }
)
Notes supérieures à 10
Au moins une note supérieure à 10 :

JavaScript

db.films.find(
  { "grades.note": { $gt: 10 } },
  { _id: 0, title: 1, grades: 1 }
)
Toutes les notes strictement supérieures à 10 :

JavaScript

db.films.find(
  { grades: { $not: { $elemMatch: { note: { $lte: 10 } } } } },
  { _id: 0, title: 1, grades: 1 }
)
Autres requêtes
Genres distincts :

JavaScript

db.films.distinct("genre")
Films sans résumé :

JavaScript

db.films.find({ summary: { $exists: false } }, { title: 1 })
Films avec Leonardo DiCaprio en 1997 :

JavaScript

db.films.find({
  "actors.first_name": "Leonardo",
  "actors.last_name": "DiCaprio",
  year: 1997
})
Films avec DiCaprio OU en 1997 :

JavaScript

db.films.find({
  $or: [
    { "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio" },
    { year: 1997 }
  ]
})
5. Requêtes sur la Base sample_mflix
Films récents (depuis 2015) :

JavaScript

db.movies.find({ year: { $gte: 2015 } }).limit(5)
Films Comedy :

JavaScript

db.movies.find({ genres: "Comedy" })
Films entre 2000 et 2005 :

JavaScript

db.movies.find(
  { year: { $gte: 2000, $lte: 2005 } },
  { title: 1, year: 1 }
)
Films Drama et Romance :

JavaScript

db.movies.find({ genres: { $all: ["Drama", "Romance"] } })
Films sans champ rated :

JavaScript

db.movies.find({ rated: { $exists: false } })
6. Agrégations MongoDB
Nombre de films par année :

JavaScript

db.movies.aggregate([
  { $group: { _id: "$year", total: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])
Moyenne IMDb par genre :

JavaScript

db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" } } },
  { $sort: { moyenne: -1 } }
])
Nombre de films par pays :

JavaScript

db.movies.aggregate([
  { $unwind: "$countries" },
  { $group: { _id: "$countries", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])
Top 5 des réalisateurs :

JavaScript

db.movies.aggregate([
  { $unwind: "$directors" },
  { $group: { _id: "$directors", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
7. Opérations de Mise à Jour
Ajouter un champ :

JavaScript

db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" } })
Incrémenter un champ :

JavaScript

db.movies.updateOne({ title: "Inception" }, { $inc: { "imdb.votes": 100 } })
Supprimer un champ :

JavaScript

db.movies.updateMany({}, { $unset: { poster: "" } })
Modifier un réalisateur :

JavaScript

db.movies.updateOne(
  { title: "Titanic" },
  { $set: { directors: ["James Cameron"] } }
)
8. Indexation et Analyse de Performances
Créer un index :

JavaScript

db.movies.createIndex({ year: 1 })
Afficher les index existants :

JavaScript

db.movies.getIndexes()
Analyser l’exécution :

JavaScript

db.movies.find({ year: 1995 }).explain("executionStats")
Supprimer un index :

JavaScript

db.movies.dropIndex({ year: 1 })
Créer un index composé :

JavaScript

db.movies.createIndex({ year: 1, "imdb.rating": -1 })
9. Arrêt du Serveur MongoDB
Bash

docker stop mongodb
docker rm mongodb
