## Table des Matières
1. [Introduction à MongoDB et au NoSQL](#1-introduction-à-mongodb-et-au-nosql)
2. [Installation et Configuration avec Docker](#2-installation-et-configuration-avec-docker)
3. [Importation des Données](#3-importation-des-données)
4. [Requêtes sur la Base lesfilms](#4-requêtes-sur-la-base-lesfilms)
5. [Requêtes sur la Base sample_mflix](#5-requêtes-sur-la-base-sample_mflix)
6. [Agrégations MongoDB](#6-agrégations-mongodb)
7. [Opérations de Mise à Jour](#7-opérations-de-mise-à-jour)
8. [Indexation et Analyse de Performances](#8-indexation-et-analyse-de-performances)
9. [Arrêt du Serveur MongoDB](#9-arrêt-du-serveur-mongodb)
10. [Conclusion](#10-conclusion)

---

## 1. Introduction à MongoDB et au NoSQL

MongoDB est une base de données NoSQL orientée documents.  
Elle stocke les informations sous forme de documents JSON/BSON et offre flexibilité, performance et scalabilité horizontale.

Ce TP permet d’apprendre :
- Le modèle documentaire
- Les requêtes simples et avancées
- Le pipeline d’agrégation
- Les opérations CRUD
- L’indexation
- L’usage de Docker pour héberger MongoDB

Deux bases sont utilisées :
- lesfilms (fichier JSON)
- sample_mflix (archive BSON)

---

## 2. Installation et Configuration avec Docker

### Lancer MongoDB

```bash
docker run --name mongodb -d -p 27017:27017 -v $(pwd)/data:/data/db mongo:latest
Vérifier l’état du conteneur
bash
Copier le code
docker ps
3. Importation des Données
3.1 Importer la base lesfilms
Copier le fichier JSON dans le conteneur :

bash
Copier le code
docker cp films.json mongodb:/films.json
Importer les données :

bash
Copier le code
mongoimport --db lesfilms --collection films --file /films.json --jsonArray
3.2 Importer la base sample_mflix (BSON)
Télécharger l’archive :

bash
Copier le code
curl https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive
Importer :

bash
Copier le code
mongorestore --archive=sampledata.archive --port=27017
4. Requêtes sur la Base lesfilms
Vérification des données
js
Copier le code
db.films.count()
db.films.findOne()
Films d’action
js
Copier le code
db.films.find({ genre: "Action" })
db.films.count({ genre: "Action" })
db.films.find({ genre: "Action", country: "FR" })
db.films.find({ genre: "Action", country: "FR", year: 1963 })
Projections
Sans grades :

js
Copier le code
db.films.find({ genre: "Action", country: "FR" }, { grades: 0 })
Sans identifiant :

js
Copier le code
db.films.find({ genre: "Action", country: "FR" }, { _id: 0 })
Titres + grades :

js
Copier le code
db.films.find(
  { genre: "Action", country: "FR" },
  { _id: 0, title: 1, grades: 1 }
)
Notes supérieures à 10
Au moins une note supérieure à 10 :

js
Copier le code
db.films.find(
  { "grades.note": { $gt: 10 } },
  { _id: 0, title: 1, grades: 1 }
)
Toutes les notes strictement supérieures à 10 :

js
Copier le code
db.films.find(
  { grades: { $not: { $elemMatch: { note: { $lte: 10 } } } } },
  { _id: 0, title: 1, grades: 1 }
)
Autres requêtes
Genres distincts :

js
Copier le code
db.films.distinct("genre")
Films sans résumé :

js
Copier le code
db.films.find({ summary: { $exists: false } }, { title: 1 })
Films avec Leonardo DiCaprio en 1997 :

js
Copier le code
db.films.find({
  "actors.first_name": "Leonardo",
  "actors.last_name": "DiCaprio",
  year: 1997
})
Films avec DiCaprio ou en 1997 :

js
Copier le code
db.films.find({
  $or: [
    { "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio" },
    { year: 1997 }
  ]
})
5. Requêtes sur la Base sample_mflix
Films récents (depuis 2015) :

js
Copier le code
db.movies.find({ year: { $gte: 2015 } }).limit(5)
Films Comedy :

js
Copier le code
db.movies.find({ genres: "Comedy" })
Films entre 2000 et 2005 :

js
Copier le code
db.movies.find(
  { year: { $gte: 2000, $lte: 2005 } },
  { title: 1, year: 1 }
)
Films Drama et Romance :

js
Copier le code
db.movies.find({ genres: { $all: ["Drama", "Romance"] } })
Films sans champ rated :

js
Copier le code
db.movies.find({ rated: { $exists: false } })
6. Agrégations MongoDB
Nombre de films par année :

js
Copier le code
db.movies.aggregate([
  { $group: { _id: "$year", total: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])
Moyenne IMDb par genre :

js
Copier le code
db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" } } },
  { $sort: { moyenne: -1 } }
])
Nombre de films par pays :

js
Copier le code
db.movies.aggregate([
  { $unwind: "$countries" },
  { $group: { _id: "$countries", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])
Top 5 des réalisateurs :

js
Copier le code
db.movies.aggregate([
  { $unwind: "$directors" },
  { $group: { _id: "$directors", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
7. Opérations de Mise à Jour
Ajouter un champ :

js
Copier le code
db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" } })
Incrémenter un champ :

js
Copier le code
db.movies.updateOne({ title: "Inception" }, { $inc: { "imdb.votes": 100 } })
Supprimer un champ :

js
Copier le code
db.movies.updateMany({}, { $unset: { poster: "" } })
Modifier un réalisateur :

js
Copier le code
db.movies.updateOne(
  { title: "Titanic" },
  { $set: { directors: ["James Cameron"] } }
)
8. Indexation et Analyse de Performances
Créer un index :

js
Copier le code
db.movies.createIndex({ year: 1 })
Afficher les index existants :

js
Copier le code
db.movies.getIndexes()
Analyser l’exécution :

js
Copier le code
db.movies.find({ year: 1995 }).explain("executionStats")
Supprimer un index :

js
Copier le code
db.movies.dropIndex({ year: 1 })
Créer un index composé :

js
Copier le code
db.movies.createIndex({ year: 1, "imdb.rating": -1 })
9. Arrêt du Serveur MongoDB
bash
Copier le code
docker stop mongodb
docker rm mongodb
10. Conclusion
Ce TP a permis d’explorer MongoDB via :

L'importation de données JSON et BSON

Les requêtes simples et avancées

Le pipeline d’agrégation

Les opérations de mise à jour

L’étude des index et de leur impact sur les performances

MongoDB est une solution flexible et performante pour manipuler des données semi-structurées.
