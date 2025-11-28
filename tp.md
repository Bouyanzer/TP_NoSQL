# Introduction à Redis et aux Bases de Données NoSQL

Ce document sert de référence et de guide pratique pour explorer **Redis** et les concepts fondamentaux des **Bases de Données NoSQL**. Il vise à offrir une ressource claire et concise sur les commandes et structures de données essentielles de Redis.

## Qu'est-ce qu'une Base de Données NoSQL ?

Le terme NoSQL (Not Only SQL) désigne une catégorie de systèmes de gestion de bases de données qui s'éloignent du modèle relationnel classique (SQL). Ces bases sont optimisées pour les besoins des applications contemporaines, en mettant l'accent sur la **flexibilité des schémas**, la **scalabilité horizontale** et la capacité à gérer efficacement des **données non structurées ou semi-structurées**.

Leurs principaux avantages incluent :

*   **Flexibilité accrue** dans la gestion des schémas de données.
*   **Scalabilité horizontale** (ajout de serveurs pour mieux gérer la charge).
*   **Traitement efficace** de données non structurées ou semi-structurées.

### Les Différents Types de Bases NoSQL

| Type de Base | Description | Cas d'Usage Idéal | Exemples |
| :--- | :--- | :--- | :--- |
| **Clé-Valeur** | Les données sont stockées sous forme de paires clé-valeur. | Caching, sessions utilisateur, paniers d'achat. | **Redis**, Amazon DynamoDB |
| **En Colonnes** | Les données sont organisées en colonnes plutôt qu’en lignes. | Analyses rapides de grandes quantités de données, séries temporelles. | Apache Cassandra, HBase |
| **Documentaires** | Les données sont stockées sous forme de documents structurés (JSON, BSON, XML). | Applications web et mobiles, catalogues de produits. | MongoDB, Couchbase |
| **Graphes** | Les données sont représentées sous forme de nœuds et de relations. | Réseaux sociaux, systèmes de recommandation, détection de fraude. | Neo4j, ArangoDB |

## Qu'est-ce que Redis ?

**Redis** (REmote DIctionary Server) est un magasin de données en mémoire, open source et extrêmement **rapide**, souvent classé comme une base de données NoSQL de type clé-valeur.

Grâce à sa rapidité, Redis est couramment utilisé pour le **caching**, la gestion de **sessions**, les **files d'attente** et les systèmes de **messagerie temps réel** (Pub/Sub). Ce guide détaille ses fonctionnalités et les structures de données complexes qu'il propose.

---

## Guide Pratique Redis

### 1. Démarrage de Redis

Pour commencer à utiliser Redis, vous devez lancer le serveur et le client.

| Action | Commande | Description |
| :--- | :--- | :--- |
| Lancer le serveur Redis | `redis-server` | Démarre le service Redis. (Écoute par défaut sur le port 6379) |
| Démarrer un client Redis | `redis-cli` | Ouvre l'interface de ligne de commande pour interagir avec le serveur. |

### 2. Gestion des Clés (CRUD)

Les commandes de base pour manipuler les paires clé-valeur.

| Opération | Commande | Exemple | Résultat |
| :--- | :--- | :--- | :--- |
| **Créer/Mettre à jour** | `SET <clé> <valeur>` | `SET demo "Bonjour"` | Crée la clé `demo` avec la valeur `"Bonjour"`. |
| **Lire** | `GET <clé>` | `GET demo` | Retourne `"Bonjour"`. |
| **Supprimer** | `DEL <clé>` | `DEL demo` | Retourne `1` si la clé est supprimée, `0` sinon. |
| **Incrémenter** | `INCR <clé>` | `INCR compteur` | Incrémente la valeur numérique de la clé. |
| **Décrémenter** | `DECR <clé>` | `DECR compteur` | Décrémente la valeur numérique de la clé. |

### 3. Durée de Vie d'une Clé (TTL)

Redis permet de définir une durée de vie (Time To Live - TTL) pour les clés, les faisant expirer automatiquement.

| Commande | Description | Exemple |
| :--- | :--- | :--- |
| `EXPIRE <clé> <secondes>` | Définit la durée de vie d'une clé en secondes. | `EXPIRE macle 120` |
| `TTL <clé>` | Vérifie la durée de vie restante. | `TTL macle` |

**Résultats de `TTL` :**
*   `-1` : La clé n’a pas de durée de vie (permanente).
*   `> 0` : Nombre de secondes restantes avant expiration.

### 4. Structures de Données Avancées

Redis prend en charge plusieurs structures de données avancées au-delà de la simple clé-valeur.

#### 4.1. Listes (Lists)

Les listes sont des collections ordonnées d'éléments, idéales pour les files d'attente ou les journaux d'événements.

| Commande | Description | Exemple |
| :--- | :--- | :--- |
| `RPUSH <clé> <valeur>` | Ajoute un élément à la **droite** (fin) de la liste. | `RPUSH mesCours "BDA"` |
| `LPUSH <clé> <valeur>` | Ajoute un élément à la **gauche** (début) de la liste. | `LPUSH mesCours "NoSQL"` |
| `LRANGE <clé> <début> <fin>` | Affiche une plage d'éléments. | `LRANGE mesCours 0 -1` (Tous les éléments) |
| `LPOP <clé>` | Supprime et retourne le premier élément (gauche). | `LPOP mesCours` |
| `RPOP <clé>` | Supprime et retourne le dernier élément (droite). | `RPOP mesCours` |

#### 4.2. Ensembles (Sets)

Les ensembles sont des collections non ordonnées qui garantissent l'**unicité** des éléments.

| Commande | Description | Exemple |
| :--- | :--- | :--- |
| `SADD <clé> <membre>` | Ajoute un ou plusieurs membres à l'ensemble. | `SADD utilisateurs "Alice" "Bob"` |
| `SMEMBERS <clé>` | Affiche tous les membres de l'ensemble. | `SMEMBERS utilisateurs` |
| `SREM <clé> <membre>` | Supprimer un membre de l'ensemble. | `SREM utilisateurs "Abdellah"` || `SUNION <clé1> <clé2>` | Retourne l'union de deux ensembles. | `SUNION ensemble1 ensemble2` |

#### 4.3. Ensembles Ordonnés (Sorted Sets)

Similaires aux ensembles, mais chaque membre est associé à un **score numérique** pour le classement. Idéal pour les classements (leaderboards).

| Commande | Description | Exemple |
| :--- | :--- | :--- |
| `ZADD <clé> <score> <membre>` | Ajoute un membre avec un score. | `ZADD scores 10 "Alice"` |
| `ZRANGE <clé> <début> <fin>` | Affiche les membres par ordre **croissant** de score. | `ZRANGE scores 0 -1` |
| `ZREVRANGE <clé> <début> <fin>` | Affiche les membres par ordre **décroissant** de score. | `ZREVRANGE scores 0 -1` |
| `ZRANK <clé> <membre>` | Trouve la position (indice) d'un membre. | `ZRANK scores "Alice"` |

#### 4.4. Hachages (Hashes)

Les hachages permettent de stocker des paires champ-valeur au sein d'une seule clé Redis, parfait pour représenter des objets (ex: profil utilisateur).

| Commande | Description | Exemple |
| :--- | :--- | :--- |
| `HSET <clé> <champ> <valeur>` | Ajoute ou met à jour un champ dans le hachage. | `HSET utilisateur:1 nom "Alice" age "30"` |
| `HGETALL <clé>` | Affiche tous les champs et leurs valeurs. | `HGETALL utilisateur:1` |

### 5. Pub/Sub (Publication/Souscription)

Le système Pub/Sub permet une communication en temps réel entre émetteurs (Publishers) et récepteurs (Subscribers) via des canaux.

| Commande | Description |
| :--- | :--- |
| `SUBSCRIBE <canal>` | S'abonne à un canal spécifique. |
| `PUBLISH <canal> "<message>"` | Publie un message sur un canal. |
| `PSUBSCRIBE <motif>` | S'abonne à tous les canaux correspondant à un motif (ex: `mes*`). |

**Scénario d'Exemple :**

1.  **Abonnement (Client 1) :**
    ```bash
    SUBSCRIBE mesCours
    ```
2.  **Publication (Client 2) :**
    ```bash
    PUBLISH mesCours "Un nouveau cours sur NoSQL"
    ```
3.  **Résultat dans Client 1 :**
    ```
    1) "message"
    2) "mesCours"
    3) "Un nouveau cours sur NoSQL"
    ```

### 6. Fonctions Utiles de Gestion

| Commande | Description |
| :--- | :--- |
| `KEYS *` | Affiche toutes les clés dans la base de données active. **Attention :** À utiliser avec prudence en production car cela peut bloquer le serveur. |
| `SELECT <numéro>` | Change la base de données active (Redis en propose 16, de 0 à 15). |
| `FLUSHDB` | Supprime **toutes** les clés de la base de données active. |
| `FLUSHALL` | Supprime **toutes** les clés de **toutes** les bases de données. |

---

## Contribution

Toute suggestion, amélioration ou correction à ce guide est la bienvenue. N'hésitez pas à contribuer !

