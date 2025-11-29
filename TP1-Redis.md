
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
```
