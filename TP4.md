
## 1. Introduction

Le **MapReduce** est présenté dans la vidéo comme un concept très utilisé en **Big Data** pour effectuer des calculs **parallèles**. L’idée générale est de traiter un grand ensemble de données (souvent des documents JSON en NoSQL) en deux étapes :
- une étape **Map** qui transforme des documents indépendamment (document par document),
- puis une étape **Reduce** qui agrège les résultats en regroupant par clé.

Dans ce TP, les démonstrations sont faites avec **CouchDB**, un SGBD documentaire (NoSQL) qui :
- expose une **API REST** (via HTTP),
- fournit une interface graphique,
- et propose un moteur d’exécution MapReduce (en JavaScript) intégré.

Le TP est réalisé **en mode centralisé** (un seul nœud) dans un premier temps.

---

## 2. Présentation de CouchDB (vidéo 1)

### 2.1 Nature du système (SGBD documentaire)

CouchDB est présenté comme un système :
- relativement simple à installer et à utiliser,
- qui stocke des données sous forme de **documents JSON** (documents “autodécrits”),
- accessible via une **API REST**.

Le cours insiste sur l’idée suivante : même si CouchDB permet d’insérer des documents hétérogènes “sans schéma”, en pratique il existe souvent un **schéma implicite** (sinon l’exploitation des données côté application devient coûteuse et complexe).

### 2.2 CouchDB via API REST : les verbes HTTP

La vidéo rappelle les 4 verbes HTTP “les plus importants” utilisés pour manipuler les ressources :
- **GET** : récupérer la représentation d’une ressource,
- **PUT** : créer une ressource (ou remplacer),
- **POST** : envoyer un contenu à une ressource / demander un service,
- **DELETE** : supprimer une ressource.

L’objectif est de comprendre que CouchDB peut être manipulé avec n’importe quel client HTTP (par exemple **curl**), puisqu’il s’agit d’une API REST.

### 2.3 Installation / lancement (Docker ou installation “en dur”)

Deux options sont mentionnées :
- installation locale (“en dur”) : Apache CouchDB, open source, soutenu par la fondation Apache ;
- utilisation de Docker.

La vidéo insiste sur un point si Docker est utilisé :
- **attention aux volumes** : si les volumes ne sont pas persistés/mappés, les données peuvent être perdues si le conteneur est arrêté/supprimé.

CouchDB écoute par défaut sur le port **5984**, ce qui est utilisé dans les exemples.

### 2.4 Interface graphique de CouchDB

CouchDB propose un client graphique accessible à l’adresse :

- `http://localhost:5984/_utils`

La vidéo invite à explorer cette interface (affichage des bases, documents, vues, pagination, etc.), mais montre aussi que le TP peut être réalisé uniquement en ligne de commande.

---

## 3. Manipulations CouchDB (CRUD) montrées dans la vidéo

Dans cette section, on résume les manipulations demandées (GET/PUT/POST/DELETE) et les points importants observés.

### 3.1 Vérifier que CouchDB répond (GET)

L’idée est d’envoyer une requête GET à CouchDB : on obtient une représentation JSON confirmant que le service répond.

### 3.2 Créer une base de données (PUT)

Pour créer une base (exemple : `films`), la vidéo montre l’usage du verbe **PUT** vers l’URL de la base.

Résultat attendu : réponse JSON indiquant que l’opération est **OK** (la ressource “base” est créée).

### 3.3 Lire les informations d’une base (GET)

Une fois la base créée, un **GET** sur l’URL de la base renvoie les caractéristiques/infos de cette base.

### 3.4 Insérer un document (PUT) + notion d’identifiant

La vidéo illustre l’insertion d’un document en choisissant un identifiant (ex : `doc`, puis `document1`).  
Point important observé :
- si on tente d’insérer un document avec un identifiant déjà existant, CouchDB renvoie un **conflit**.

La vidéo mentionne également la présence d’un champ de version de document :
- **`_rev`** : CouchDB versionne les documents.

### 3.5 Insérer un document depuis un fichier (POST)

La vidéo montre qu’on peut envoyer un document JSON depuis un fichier, en utilisant **POST** et l’option permettant de transmettre le contenu du fichier.

Remarque importante donnée :
- si aucun identifiant n’est fourni, CouchDB attribue un **identifiant unique** automatiquement.

### 3.6 Documents JSON imbriqués et schéma implicite

La vidéo souligne la différence majeure avec les bases relationnelles :
- en relationnel, on normalise pour éviter redondance et incohérences ;
- en documentaire (JSON), on peut imbriquer des structures et dupliquer des informations.

Conséquence mise en avant :
- on peut se retrouver avec des **incohérences** (ex. le même acteur écrit différemment dans deux documents, ou une date de naissance pas mise à jour partout).

### 3.7 Insertion en masse (bulk)

La vidéo montre l’insertion d’une **collection** de films en une seule opération (au lieu d’insérer document par document).  
Idée :
- on envoie un document JSON contenant une liste de documents, pour importer la collection en masse.

Après import, l’interface indique (dans l’exemple) un certain nombre de films importés, et permet de les parcourir (pagination et affichage).

---

## 4. MapReduce : concepts expliqués (vidéo 2)

### 4.1 Pourquoi MapReduce ?

MapReduce est présenté comme une manière de faire des calculs parallèles sur de très grandes collections de documents. La raison mise en avant :
- les documents JSON sont **indépendants** : on peut appliquer une même transformation à chacun d’eux.

### 4.2 Fonction Map : transformation + émission de paires (clé, valeur)

La fonction **map** :
- prend un document en paramètre,
- réalise un traitement,
- et **émet** 0, 1 ou plusieurs résultats intermédiaires.

La vidéo insiste sur le fait que ce n’est pas un “return classique” :
- on **émet** des éléments intermédiaires destinés à être regroupés et agrégés plus tard.

Concept clé : la map définit une **clé intermédiaire**, appelée aussi clé de groupement, qui correspond à l’idée de **GROUP BY**.

### 4.3 Phase intermédiaire : tri / regroupement (sort & shuffle)

La vidéo mentionne une phase intermédiaire (sans détailler en profondeur) :
- les résultats émis sont triés et regroupés par clé,
- puis (dans un cadre distribué) envoyés vers des nœuds qui feront l’agrégation.

Dans ce TP, on reste en **centralisé**, mais on observe déjà la notion de regroupement par clé (années regroupées, acteurs regroupés, etc.).

### 4.4 Fonction Reduce : agrégation

La fonction **reduce** :
- prend une clé et l’ensemble des valeurs associées,
- applique une fonction d’agrégation (somme, comptage, etc.),
- et renvoie un résultat agrégé.

Particularité soulignée dans la vidéo :
- avec CouchDB, la fonction reduce **n’est pas obligatoire** : on peut faire une vue “map-only” et observer les résultats intermédiaires.

La vidéo mentionne aussi l’existence de fonctions d’agrégation proposées par défaut (ex. somme, count, stat), et la possibilité d’écrire sa propre fonction.

---

## 5. Exemples MapReduce présentés sur la collection de films

La vidéo part d’une base contenant une collection de films (documents JSON) et montre deux analyses.

### 5.1 Exemple 1 : nombre de films par année

**Objectif :** compter le nombre de films pour chaque année.

**Map (idée)** : pour chaque film :
- clé = année de sortie,
- valeur = titre (ou une constante 1).

On observe que les résultats sont déjà regroupés par année (clés triées).

**Reduce (idée)** :
- agrégation par année : somme des valeurs (si on a émis 1) ou “compter” les éléments.

Résultat final attendu :
- pour chaque année, un nombre total de films.

### 5.2 Exemple 2 : nombre de films par acteur

**Objectif :** compter le nombre de films pour chaque acteur.

Observation sur les documents :
- le champ des acteurs est un **tableau** (plusieurs acteurs par film).

**Map (idée)** : pour chaque film, pour chaque acteur du tableau :
- clé = acteur (prénom + nom),
- valeur = titre (ou 1).

**Reduce (idée)** :
- pour chaque acteur (clé), on fait une somme (ou un comptage) afin d’obtenir le nombre de films.

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

