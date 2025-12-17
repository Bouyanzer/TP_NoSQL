
## 1. Introduction

Le **MapReduce** est présenté dans la vidéo comme un concept très utilisé en **Big Data** pour effectuer des calculs **parallèles**. L’idée générale est de traiter un grand ensemble de données (souvent des documents JSON en NoSQL) en deux étapes :
- une étape **Map** qui transforme des documents indépendamment (document par document),
- puis une étape **Reduce** qui agrège les résultats en regroupant par clé.

Dans ce TP, les démonstrations sont faites avec **CouchDB**, un SGBD documentaire (NoSQL) qui :
- expose une **API REST** (via HTTP),
- fournit une interface graphique,
- et propose un moteur d’exécution MapReduce (en JavaScript) intégré.


---

## 2. CouchDB : ce qu’on retient de la présentation

### 2.1 Nature du système (documents JSON)

D’après la présentation, CouchDB :
- stocke des données sous forme de **documents JSON** (autodécrits),
- autorise des documents potentiellement hétérogènes (pas de schéma strict imposé),
- et expose ses fonctionnalités via une API REST.

Un point important est souligné : même si le système accepte des documents “sans schéma”, en pratique il existe souvent un **schéma implicite** (sinon l’application doit gérer trop de cas et l’exploitation devient coûteuse).

### 2.2 Accès REST : GET, PUT, POST, DELETE

D’après le rappel fait dans la vidéo, l’API REST s’appuie principalement sur quatre verbes HTTP :
- **GET** : demander la représentation d’une ressource ;
- **PUT** : créer une ressource (ou remplacer) ;
- **POST** : envoyer un contenu / demander une action à une ressource ;
- **DELETE** : supprimer une ressource.

L’idée clé est que l’on peut piloter CouchDB avec n’importe quel client HTTP (ex. `curl`).

### 2.3 Lancement : installation locale ou Docker

Deux modes sont évoqués :
- installation locale (Apache CouchDB, open source) ;
- Docker.

Il est insisté sur le fait que, via Docker, il faut **gérer les volumes** afin d’éviter la perte de données si le conteneur est arrêté ou supprimé.

CouchDB écoute par défaut sur le port **5984**.

### 2.4 Interface graphique

On constate que CouchDB fournit une interface graphique accessible via :

- `http://localhost:5984/_utils`

Cette interface permet de parcourir bases, documents, vues, pagination, etc., mais la manipulation via requêtes HTTP reste centrale dans le TP.

---

## 3. Manipulations CouchDB : synthèse (CRUD)

Dans cette section, on reformule les manipulations en mode “constat” (sans décrire la vidéo comme une suite d’actions).

### 3.1 Vérifier que CouchDB répond (GET)

D’après les tests effectués, un **GET** sur l’URL du serveur CouchDB renvoie une réponse JSON qui confirme que le service est accessible et répond correctement.

### 3.2 Créer une base de données (PUT)

D’après l’exemple, la création d’une base (ex. `films`) se fait par un **PUT** vers l’URL correspondant au nom de la base.  
On constate que la réponse attendue est un JSON indiquant que l’opération est **OK**, ce qui signifie que la ressource “base de données” a été créée.

### 3.3 Lire les informations d’une base (GET)

Une fois la base créée, on constate qu’un **GET** sur l’URL de cette base renvoie ses informations (métadonnées, statistiques de base, etc.).

### 3.4 Insérer un document (PUT) et rôle de l’identifiant

D’après les exemples, l’insertion d’un document peut se faire en choisissant explicitement un identifiant (ex. `doc`, puis `document1`).  
On constate un comportement important :
- si un document avec le même identifiant existe déjà, CouchDB renvoie un **conflit**.

On observe également que CouchDB associe une notion de versionnement aux documents via le champ **`_rev`**.

### 3.5 Insérer un document depuis un fichier (POST)

On constate qu’il est possible d’insérer un document JSON depuis un fichier en utilisant **POST**.  
Une remarque importante est donnée :
- si aucun identifiant n’est précisé, CouchDB attribue automatiquement un **identifiant unique**.

### 3.6 Documents JSON imbriqués et conséquences (redondance/incohérence)

D’après l’explication, CouchDB (et les bases documentaires) autorisent des documents JSON **imbriqués**.  
On constate une différence essentielle avec le modèle relationnel :
- le relationnel vise à limiter la redondance (normalisation) ;
- en documentaire, on accepte plus facilement la duplication pour simplifier l’accès local aux informations.

Conséquence mise en avant : cette duplication peut conduire à des **incohérences** (ex. même acteur écrit différemment selon les documents, ou mise à jour non répercutée partout).

### 3.7 Insertion en masse (bulk import)

On constate qu’il existe une insertion “en masse” : au lieu d’insérer document par document, on envoie un document JSON contenant une collection de documents, ce qui permet un import groupé.  
Après import, la base contient la collection et l’interface permet de la parcourir (pagination, vues tabulaires, etc.).

---

## 4. MapReduce : concepts (présentation accessible)

### 4.1 Pourquoi MapReduce ?

D’après l’explication, MapReduce sert à exécuter des calculs **parallèles** sur un grand volume de données, notamment parce que les documents (JSON) sont **indépendants** : on peut appliquer le même traitement sur chacun sans dépendance directe.

### 4.2 Étape Map : transformation et émission

On constate que la fonction **map** :
- prend un document en entrée,
- applique un traitement (extraction, nettoyage, transformation),
- puis **émet** des résultats intermédiaires sous forme de couples (clé, valeur).

Point important : l’émission vise à préparer un regroupement par clé (équivalent logique d’un **GROUP BY**).

### 4.3 Tri / regroupement (sort & shuffle)

D’après la description, une phase intermédiaire organise les résultats :
- regroupement par clé,
- préparation à l’agrégation.

Même en mode centralisé, on observe déjà le regroupement : les mêmes clés apparaissent ensemble.

### 4.4 Étape Reduce : agrégation

On constate que la fonction **reduce** :
- reçoit une clé et l’ensemble des valeurs associées,
- effectue une agrégation (somme, comptage, etc.),
- produit un résultat final par clé.

Particularité soulignée : dans CouchDB, la reduce peut être **optionnelle**, ce qui permet d’inspecter les résultats intermédiaires produits par map.

---

## 5. Exemples MapReduce sur une collection de films (idée générale)

### 5.1 Exemple : nombre de films par année

D’après l’exemple, si l’on veut compter des films par année :
- **Map** : émettre (année → 1) ou (année → information) ;
- **Reduce** : sommer les 1 (ou compter les éléments) pour obtenir un total par année.

### 5.2 Exemple : nombre de films par acteur

On constate que si un film contient plusieurs acteurs (tableau) :
- **Map** : pour chaque acteur, émettre (acteur → 1) ;
- **Reduce** : sommer pour obtenir le nombre de films associés à chaque acteur.

---


## 6. Modélisation “PageRank-like” : matrice de liens web $M$

### 6.1 Problème et intuition (matrice très grande)

On considère une matrice $M$ de taille $N \times N$ :
- la ligne $i$ décrit la page $P_i$,
- $M_{ij}$ est le poids du lien de la page $P_i$ vers $P_j$ (importance du lien).

Comme $N$ peut être très grand, on cherche une représentation en **documents structurés** (JSON), typiquement en représentation **creuse** : ne stocker que les liens existants.

### 6.2 Proposition de modèle documentaire (collection $C$)

**Un document par page $P_i$** (ligne $i$) :

```json
{
  "_id": "Pi",
  "links": [
    { "to": "Pj", "w": 0.42 },
    { "to": "Pk", "w": 0.10 }
  ]
}
```

- `_id` identifie la page $P_i$.
- `links` contient la liste des liens sortants (les colonnes $j$ où $M_{ij} \neq 0$).
- chaque lien contient :
  - `to` : identifiant de la page cible,
  - `w` : le poids $M_{ij}$.

La **collection $C$** est l’ensemble de ces documents.

---

## 7. MapReduce : calcul de la norme des vecteurs (lignes de $M$)

### 7.1 Rappel : norme d’un vecteur (ligne $i$)

La ligne $i$ est un vecteur à $N$ dimensions :

$$
V_i = (M_{i1}, M_{i2}, \dots, M_{iN})
$$

Sa norme est :

$$
\|V_i\| = \sqrt{\sum_{j=1}^{N} M_{ij}^2}
$$

Avec la représentation creuse, on somme seulement sur les liens présents dans `links`.

### 7.2 Traitement MapReduce proposé

#### Fonction Map (principe)
Pour chaque page $P_i$, pour chaque lien $(i \to j)$ de poids $w = M_{ij}$, on émet :
- **clé** = $P_i$,
- **valeur** = $w^2$.

Map (JavaScript, style CouchDB) :

```js
function(doc) {
  if (!doc.links) return;
  for (var k = 0; k < doc.links.length; k++) {
    var w = doc.links[k].w;
    emit(doc._id, w * w);
  }
}
```

#### Fonction Reduce (principe)
Pour chaque clé $P_i$, on somme toutes les valeurs $w^2$ :

```js
function(keys, values, rereduce) {
  return sum(values);
}
```

#### Interprétation du résultat
Le reduce renvoie :

$$
S_i = \sum_j M_{ij}^2
$$

La norme s’obtient ensuite par :

$$
\|V_i\| = \sqrt{S_i}
$$

---

## 8. MapReduce : produit matrice–vecteur $\varphi = M \cdot W$

### 8.1 Rappel : produit matrice–vecteur

$$
\varphi_i = \sum_{j=1}^{N} M_{ij} \cdot w_j
$$

Hypothèse imposée : le vecteur $W$ tient en RAM et est accessible comme variable statique par Map/Reduce.

### 8.2 Traitement MapReduce proposé

#### Map (émission par lien)
Pour chaque lien $(i \to j)$, on calcule une contribution partielle $M_{ij} \cdot w_j$ et on l’émet avec la clé $P_i$.

```js
// W est supposé accessible (variable statique)
// Exemple : var W = { "P1": 0.2, "P2": 0.9, ... };

function(doc) {
  if (!doc.links) return;

  for (var k = 0; k < doc.links.length; k++) {
    var j = doc.links[k].to;   // page cible
    var mij = doc.links[k].w;  // poids M_ij
    var wj = W[j];             // composante du vecteur W
    emit(doc._id, mij * wj);
  }
}
```

#### Reduce (somme des contributions)

```js
function(keys, values, rereduce) {
  return sum(values);
}
```

#### Résultat
Pour chaque page $P_i$, le reduce renvoie :

$$
\varphi_i = \sum_j (M_{ij} \cdot w_j)
$$

On obtient ainsi le vecteur $\varphi$ sous forme (clé = page $P_i$, valeur = $\varphi_i$).

---
#  — Exercices 

Commande générale utilisée :

```js
db.films.mapReduce(mapFunction, reduceFunction, {
  out: "<nom_sortie>"
});
```

---

## 1) Compter le nombre total de films dans la collection

**Map**
```js
var mapTotalFilms = function () {
  emit("total_films", 1);
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapTotalFilms, reduceSum, { out: "mr_total_films" });
```

---

## 2) Compter le nombre de films par genre

**Map**
```js
var mapFilmsParGenre = function () {
  if (this.genre) {
    this.genre.forEach(function (g) {
      emit(g, 1);
    });
  }
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapFilmsParGenre, reduceSum, { out: "mr_films_par_genre" });
```

---

## 3) Compter le nombre de films réalisés par chaque réalisateur

**Map**
```js
var mapFilmsParRealisateur = function () {
  if (this.director) {
    emit(this.director, 1);
  }
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapFilmsParRealisateur, reduceSum, { out: "mr_films_par_realisateur" });
```

---

## 4) Compter le nombre d’acteurs uniques apparaissant dans tous les films

Idée : on émet une fois chaque acteur comme clé ; le reduce renvoie 1 par acteur.  
Le **nombre d’acteurs uniques** = nombre de documents dans la collection de sortie.

**Map**
```js
var mapActeurs = function () {
  if (this.actors) {
    this.actors.forEach(function (a) {
      emit(a, 1);
    });
  }
};
```

**Reduce**
```js
var reduceUnique = function (key, values) {
  return 1;
};
```

**Exécution**
```js
db.films.mapReduce(mapActeurs, reduceUnique, { out: "mr_acteurs_uniques" });
```

**Résultat (compte des acteurs uniques)**
```js
db.mr_acteurs_uniques.count();
```

---

## 5) Lister le nombre de films par année de sortie

**Map**
```js
var mapFilmsParAnnee = function () {
  if (this.year !== undefined && this.year !== null) {
    emit(this.year, 1);
  }
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapFilmsParAnnee, reduceSum, { out: "mr_films_par_annee" });
```

---

## 6) Calculer la note moyenne par film à partir du tableau `grades`

On calcule la moyenne du tableau `grades` (champ `score`), puis on émet (titre → moyenne).

**Map**
```js
var mapMoyenneParFilm = function () {
  if (this.grades && this.grades.length > 0) {
    var sum = 0;
    for (var i = 0; i < this.grades.length; i++) {
      sum += this.grades[i].score;
    }
    var mean = sum / this.grades.length;
    emit(this.title, mean);
  }
};
```

**Reduce**
```js
var reduceAvg = function (key, values) {
  return Array.sum(values) / values.length;
};
```

**Exécution**
```js
db.films.mapReduce(mapMoyenneParFilm, reduceAvg, { out: "mr_moyenne_par_film" });
```

---

## 7) Calculer la note moyenne par genre

On calcule la moyenne du film, puis on l’émet pour chacun de ses genres (genre → moyenneDuFilm).

**Map**
```js
var mapMoyenneParGenre = function () {
  if (this.grades && this.grades.length > 0 && this.genre) {
    var sum = 0;
    for (var i = 0; i < this.grades.length; i++) {
      sum += this.grades[i].score;
    }
    var avgFilm = sum / this.grades.length;

    this.genre.forEach(function (g) {
      emit(g, avgFilm);
    });
  }
};
```

**Reduce**
```js
var reduceAvg = function (key, values) {
  return Array.sum(values) / values.length;
};
```

**Exécution**
```js
db.films.mapReduce(mapMoyenneParGenre, reduceAvg, { out: "mr_moyenne_par_genre" });
```

---

## 8) Calculer la note moyenne par réalisateur

On calcule la moyenne du film, puis on l’émet (réalisateur → moyenneDuFilm).

**Map**
```js
var mapMoyenneParRealisateur = function () {
  if (this.director && this.grades && this.grades.length > 0) {
    var sum = 0;
    for (var i = 0; i < this.grades.length; i++) {
      sum += this.grades[i].score;
    }
    var avgFilm = sum / this.grades.length;
    emit(this.director, avgFilm);
  }
};
```

**Reduce**
```js
var reduceAvg = function (key, values) {
  return Array.sum(values) / values.length;
};
```

**Exécution**
```js
db.films.mapReduce(mapMoyenneParRealisateur, reduceAvg, { out: "mr_moyenne_par_realisateur" });
```

---

## 9) Trouver le film avec la note maximale la plus élevée

Pour chaque film, on calcule **la note max** de son tableau `grades`, puis on garde le meilleur film global.

**Map**
```js
var mapMaxParFilm = function () {
  if (this.grades && this.grades.length > 0) {
    var maxScore = this.grades[0].score;
    for (var i = 1; i < this.grades.length; i++) {
      if (this.grades[i].score > maxScore) maxScore = this.grades[i].score;
    }
    emit("best_film", { title: this.title, max: maxScore });
  }
};
```

**Reduce**
```js
var reduceBestFilm = function (key, values) {
  var best = values[0];
  for (var i = 1; i < values.length; i++) {
    if (values[i].max > best.max) best = values[i];
  }
  return best;
};
```

**Exécution**
```js
db.films.mapReduce(mapMaxParFilm, reduceBestFilm, { out: "mr_best_film" });
```

---

## 10) Compter le nombre de notes supérieures à 70 dans tous les films

**Map**
```js
var mapNotesSup70 = function () {
  if (this.grades && this.grades.length > 0) {
    var count = 0;
    for (var i = 0; i < this.grades.length; i++) {
      if (this.grades[i].score > 70) count++;
    }
    emit("notes_sup_70", count);
  }
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapNotesSup70, reduceSum, { out: "mr_notes_sup_70" });
```

---

## 11) Lister tous les acteurs par genre, sans doublons

**Map**
```js
var mapActeursParGenre = function () {
  if (this.genre && this.actors) {
    for (var i = 0; i < this.genre.length; i++) {
      for (var j = 0; j < this.actors.length; j++) {
        emit(this.genre[i], this.actors[j]);
      }
    }
  }
};
```

**Reduce** (déduplication robuste, y compris en cas de re-reduce)
```js
var reduceUniqueActorsList = function (key, values) {
  var seen = {};
  for (var i = 0; i < values.length; i++) {
    var v = values[i];

    if (Array.isArray(v)) {
      for (var k = 0; k < v.length; k++) {
        seen[v[k]] = 1;
      }
    } else {
      seen[v] = 1;
    }
  }
  return Object.keys(seen);
};
```

**Exécution**
```js
db.films.mapReduce(mapActeursParGenre, reduceUniqueActorsList, { out: "mr_acteurs_par_genre" });
```

---

## 12) Trouver les acteurs apparaissant dans le plus grand nombre de films

Étape 1 : compter le nombre de films par acteur.

**Map**
```js
var mapFilmsParActeur = function () {
  if (this.actors) {
    this.actors.forEach(function (a) {
      emit(a, 1);
    });
  }
};
```

**Reduce**
```js
var reduceSum = function (key, values) {
  return Array.sum(values);
};
```

**Exécution**
```js
db.films.mapReduce(mapFilmsParActeur, reduceSum, { out: "mr_films_par_acteur" });
```

Pour obtenir l’acteur “top” (et gérer les ex æquo) :
```js
var top = db.mr_films_par_acteur.find().sort({ value: -1 }).limit(1).next().value;
db.mr_films_par_acteur.find({ value: top });
```

---

## 13) Classer les films par lettre de grade majoritaire (A, B, C, …)

On cherche la lettre la plus fréquente dans `grades[*].grade`, puis on groupe les films par cette lettre.

**Map**
```js
var mapGradeMajoritaire = function () {
  if (this.grades && this.grades.length > 0) {
    var freq = {};
    for (var i = 0; i < this.grades.length; i++) {
      var g = this.grades[i].grade;
      freq[g] = (freq[g] || 0) + 1;
    }

    var major = null;
    for (var k in freq) {
      if (major === null || freq[k] > freq[major]) {
        major = k;
      }
    }

    emit(major, this.title);
  }
};
```

**Reduce** (concatène et supporte les re-reduce)
```js
var reduceList = function (key, values) {
  var out = [];
  for (var i = 0; i < values.length; i++) {
    var v = values[i];
    if (Array.isArray(v)) out = out.concat(v);
    else out.push(v);
  }
  return out;
};
```

**Exécution**
```js
db.films.mapReduce(mapGradeMajoritaire, reduceList, { out: "mr_films_par_grade_majoritaire" });
```

---

## 14) Calculer la note moyenne par année de sortie des films

On calcule la moyenne du film, puis on l’émet (année → moyenneDuFilm).

**Map**
```js
var mapMoyenneParAnnee = function () {
  if (this.year !== undefined && this.year !== null && this.grades && this.grades.length > 0) {
    var sum = 0;
    for (var i = 0; i < this.grades.length; i++) {
      sum += this.grades[i].score;
    }
    var avgFilm = sum / this.grades.length;
    emit(this.year, avgFilm);
  }
};
```

**Reduce**
```js
var reduceAvg = function (key, values) {
  return Array.sum(values) / values.length;
};
```

**Exécution**
```js
db.films.mapReduce(mapMoyenneParAnnee, reduceAvg, { out: "mr_moyenne_par_annee" });
```

---

## 15) Identifier les réalisateurs dont la note moyenne de tous leurs films est supérieure à 80

On calcule d’abord la moyenne de chaque film, puis on agrège par réalisateur via un couple (sum, count).  
La sélection “> 80” se fait dans `finalize`.

**Map**
```js
var mapRealisateurAvgGT80 = function () {
  if (this.director && this.grades && this.grades.length > 0) {
    var sum = 0;
    for (var i = 0; i < this.grades.length; i++) {
      sum += this.grades[i].score;
    }
    var avgFilm = sum / this.grades.length;

    emit(this.director, { sum: avgFilm, count: 1 });
  }
};
```

**Reduce**
```js
var reduceSumCount = function (key, values) {
  var sum = 0;
  var count = 0;

  for (var i = 0; i < values.length; i++) {
    sum += values[i].sum;
    count += values[i].count;
  }

  return { sum: sum, count: count };
};
```

**Finalize**
```js
var finalizeAvgGT80 = function (key, value) {
  var avg = value.sum / value.count;
  return (avg > 80) ? avg : null;
};
```

**Exécution**
```js
db.films.mapReduce(mapRealisateurAvgGT80, reduceSumCount, {
  out: "mr_realisateurs_avg_gt80",
  finalize: finalizeAvgGT80
});
```

Pour afficher uniquement les réalisateurs retenus (valeur non nulle) :
```js
db.mr_realisateurs_avg_gt80.find({ value: { $ne: null } });
```

---

## 9. Conclusion

Ce TP relie :
- **CouchDB** (documents JSON + API REST + interface graphique),
- les manipulations CRUD via **GET/PUT/POST/DELETE**,
- et **MapReduce** :
  - Map = traitement indépendant par document + émission (clé, valeur),
  - regroupement par clé (analogie “GROUP BY”),
  - Reduce = agrégation (somme, comptage, etc.),
  - particularité CouchDB : reduce optionnelle et résultats intermédiaires visibles.

Enfin, la modélisation d’une matrice de liens web sous forme de documents (un document par page) s’inscrit naturellement dans le cadre MapReduce pour effectuer des calculs de type “vecteur” (normes) et “produit scalaire” (matrice–vecteur).

