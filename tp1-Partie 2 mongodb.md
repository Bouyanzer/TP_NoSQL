# TP MongoDB – Bases de Données NoSQL  
## Polytech Nancy – Prise en Main de MongoDB

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
