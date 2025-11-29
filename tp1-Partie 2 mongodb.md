📖 Introduction

Ce projet présente l’ensemble des manipulations réalisées dans le cadre du TP Bases de Données NoSQL utilisant MongoDB, un SGBD documentaire flexible et performant.

Objectifs du TP :

Installer MongoDB via Docker

Importer deux jeux de données : lesfilms et sample_mflix

Explorer les documents

Réaliser des requêtes, agrégations et mises à jour

Manipuler les index et analyser leurs effets

🚀 1. Installation de MongoDB avec Docker

Lancer un conteneur MongoDB :

docker run --name mongodb -d -p 27017:27017 -v $(pwd)/data:/data/db mongo:latest


Vérifier qu’il tourne :

docker ps

📥 2. Importation des données
👉 Importation du fichier films.json

Copier dans le conteneur :

docker cp films.json mongodb:/films.json


Importer dans MongoDB :

mongoimport --db lesfilms --collection films --file /films.json --jsonArray

👉 Importation du jeu de données BSON sample_mflix

Téléchargement :

curl https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive


Importation :

mongorestore --archive=sampledata.archive --port=27017

🧭 3. Accéder à MongoDB
docker exec -it mongodb mongosh


Sélection de la base :

use lesfilms

🎞️ 4. Requêtes – Base lesfilms
🔍 4.1. Vérification des données
Objectif	Commande
Nombre de documents	db.films.count()
Aperçu d’un document	db.films.findOne()
🎬 4.2. Films d’action
db.films.find({ genre: "Action" })
db.films.count({ genre: "Action" })


Films d’action produits en France :

db.films.find({ genre: "Action", country: "FR" })


Films d’action français en 1963 :

db.films.find({ genre: "Action", country: "FR", year: 1963 })

🧩 4.3. Projections

Sans le champ grades :

db.films.find({ genre: "Action", country: "FR" }, { grades: 0 })


Sans _id :

db.films.find({ genre: "Action", country: "FR" }, { _id: 0 })


Titres + grades :

db.films.find({ genre: "Action", country: "FR" }, { _id: 0, title: 1, grades: 1 })

⭐ 4.4. Notes supérieures à 10

Au moins une note > 10 :

db.films.find(
  { genre: "Action", country: "FR", "grades.note": { $gt: 10 } },
  { _id: 0, title: 1, grades: 1 }
)


Toutes les notes > 10 :

db.films.find(
  {
    genre: "Action",
    country: "FR",
    grades: { $not: { $elemMatch: { note: { $lte: 10 } } } }
  },
  { _id: 0, title: 1, grades: 1 }
)

🗂️ 4.5. Requêtes supplémentaires

Genres distincts :

db.films.distinct("genre")


Films sans résumé :

db.films.find({ summary: { $exists: false } }, { title: 1 })


Films avec Leonardo DiCaprio en 1997 :

db.films.find(
  { "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio", year: 1997 }
)

🎥 5. Requêtes – Base sample_mflix
🎯 5.1. Requêtes simples

Films sortis depuis 2015 :

db.movies.find({ year: { $gte: 2015 } }).limit(5)


Films Comedy :

db.movies.find({ genres: "Comedy" })


Films entre 2000 et 2005 :

db.movies.find(
  { year: { $gte: 2000, $lte: 2005 } },
  { title: 1, year: 1 }
)


Films Drama et Romance :

db.movies.find({ genres: { $all: ["Drama", "Romance"] } })


Films sans rated :

db.movies.find({ rated: { $exists: false } })

📊 6. Agrégations – sample_mflix
Nombre de films par année
db.movies.aggregate([
  { $group: { _id: "$year", total: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])

Moyenne IMDb par genre
db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" } } },
  { $sort: { moyenne: -1 } }
])

Films par pays
db.movies.aggregate([
  { $unwind: "$countries" },
  { $group: { _id: "$countries", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])

Top 5 des réalisateurs
db.movies.aggregate([
  { $unwind: "$directors" },
  { $group: { _id: "$directors", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])

🛠️ 7. Mises à jour

Ajouter un champ :

db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" } })


Incrémenter un champ :

db.movies.updateOne(
  { title: "Inception" },
  { $inc: { "imdb.votes": 100 } }
)


Supprimer un champ :

db.movies.updateMany({}, { $unset: { poster: "" } })

⚡ 8. Indexation

Créer un index :

db.movies.createIndex({ year: 1 })


Voir les index :

db.movies.getIndexes()


Comparer les performances :

db.movies.find({ year: 1995 }).explain("executionStats")


Supprimer un index :

db.movies.dropIndex({ year: 1 })


Créer un index composé :

db.movies.createIndex({ year: 1, "imdb.rating": -1 })

🧹 9. Arrêter MongoDB
docker stop mongodb
docker rm mongodb

🏁 Conclusion

Ce TP permet une bonne prise en main des concepts essentiels de MongoDB :

Manipulation de documents JSON/BSON

Requêtes complexes et projections

Pipeline d’agrégation

Mises à jour et gestion d’index

Utilisation de Docker pour simplifier la gestion du serveur

MongoDB se révèle un outil puissant, rapide et idéal pour des données non structurées ou évolutives.
