# Politique de Sécurité Base de Données
**Document de référence pour la sécurité des bases de données (SQL, NoSQL, Cloud DB)**

---

## 1. Introduction

La base de données contient les informations les plus sensibles (données clients, identifiants, journaux, secrets techniques). Sa compromission a un impact direct sur la confidentialité, l’intégrité et la disponibilité des systèmes.  

Cette politique définit les mesures de sécurité minimales applicables à toutes les bases de données (SQL, NoSQL, hébergées ou managées) dans les environnements de développement, test, pré‑production et production. [web:131][web:138]  

---

## 2. Principes généraux

1. **Séparation des couches** : la base de données n’est jamais exposée directement à Internet, seulement accessible depuis les services backend autorisés. [web:139]  
2. **Moindre privilège** : chaque compte DB (appli, admin, service) dispose du minimum de droits nécessaires. [web:136][web:138]  
3. **Chiffrement systématique** : données sensibles chiffrées au repos et en transit. [web:131][web:132]  
4. **Traçabilité & audit** : toutes les actions critiques sont journalisées (connexion, schéma, droits, accès sensibles). [web:139][web:143]  

---

## 3. Architecture & exposition réseau

- La base de données est installée sur un réseau interne / privé (VLAN, VPC), non accessible directement depuis Internet. [web:139][web:136]  
- L’accès est restreint aux seules adresses IP / security groups des serveurs applicatifs et bastions d’admin.  
- Les ports d’écoute (3306, 5432, 1433, 27017, etc.) sont filtrés par pare‑feu (host / réseau / cloud security groups). [web:139][web:148]  
- Les outils d’administration (pgAdmin, SSMS, phpMyAdmin, consoles cloud) ne sont accessibles que via VPN, bastion sécurisé ou SSO fort. [web:136][web:145]  

---

## 4. Comptes, rôles & privilèges

### 4.1. Comptes applicatifs

- Un compte DB **distinct** par application et par environnement (dev, test, prod). [web:131][web:136]  
- Droits limités :
  - généralement `SELECT/INSERT/UPDATE/DELETE` sur les schémas nécessaires,  
  - pas de `CREATE/DROP DATABASE`, pas de `ALTER` global, pas de `GRANT`/`REVOKE`.  
- Aucun usage de comptes `root`, `sa`, `postgres` ou équivalents par les applications. [web:138][web:142]  

### 4.2. Comptes administrateurs

- Comptes admins nominatifs (pas de compte “admin” partagé), avec mots de passe forts et rotation régulière. [web:136][web:139]  
- Usage des rôles intégrés (ex : `db_owner`, `db_datareader`, `db_datawriter`) plutôt que des privilèges ad‑hoc dispersés. [web:145][web:148]  

### 4.3. Rôles & niveau de ligne / colonne

- Utilisation de **rôles** DB pour regrouper les droits, puis assignation des rôles aux comptes. [web:138][web:137]  
- Pour les données très sensibles, usage si possible :
  - de **Row‑Level Security** (RLS) pour limiter les lignes visibles par utilisateur/tenant,  
  - de **Column‑level permissions** ou vues pour cacher certaines colonnes (ex : PII, secrets). [web:131][web:142]  

---

## 5. Configuration & durcissement

- Suppression des comptes par défaut inutiles, bases de démo et fonctionnalités non utilisées. [web:136][web:139]  
- Paramétrage sécurisé des logs, des temps de session, des politiques de mots de passe, et des limites de ressources (max connexions, taille requêtes). [web:139][web:145]  
- Désactivation de fonctionnalités dangereuses si non nécessaires (ex : `xp_cmdshell` sur SQL Server, `LOCAL INFILE` MySQL, `trust authentication` Postgres). [web:136][web:148]  

---

## 6. Protection contre les injections & requêtes dangereuses

### 6.1. Côté application

- Toutes les requêtes sont faites avec des **requêtes paramétrées / prepared statements** ou via un ORM ; aucune concaténation de fragments SQL à partir de données utilisateur. [web:135][web:147]  
- Les clauses dynamiques (ORDER BY, LIMIT, champs de tri) sont basées sur des **allowlists** (liste de colonnes autorisées). [web:135]  

### 6.2. Côté base de données

- Limitation des privilèges pour réduire l’impact d’une injection réussie (compte applicatif sans droits de DDL ni d’admin). [web:138][web:136]  
- Encapsulation possible des accès sensibles dans des **vues** ou **procédures stockées** avec validation interne. [web:138][web:139]  

---

## 7. Chiffrement & confidentialité des données

### 7.1. Chiffrement en transit

- Toutes les connexions DB utilisent TLS/SSL (paramètres `sslmode=require`, `encrypt=true`, etc. selon le moteur). [web:132][web:134]  
- Les certificats sont gérés de manière sécurisée (CA de confiance, rotation, pas de certs auto‑signés en prod).  

### 7.2. Chiffrement au repos

- Activation du chiffrement au repos :
  - soit au niveau disque/volume (ex : LUKS, EBS encrypted, disque chiffré),  
  - soit au niveau base de données (TDE : Transparent Data Encryption) si proposé par le moteur. [web:131][web:137]  
- Les colonnes contenant des données hautement sensibles (PII, santé, finance, secrets) sont chiffrées au niveau applicatif ou avec des fonctions de chiffrement intégrées. [web:133][web:131]  

### 7.3. Masquage & anonymisation

- Les environnements **dev/test/staging** utilisent des données :
  - soit synthétiques,  
  - soit pseudonymisées/anonymisées pour éviter d’exposer des données réelles. [web:133][web:137]  

---

## 8. Journalisation & audit

- Activation des logs :
  - connexions/authentifications (succès/échecs),  
  - changements de schéma (DDL),  
  - changements de privilèges (GRANT/REVOKE),  
  - accès aux tables critiques si possible (auditing). [web:139][web:138]  
- Export et centralisation des logs DB vers une plateforme d’analyse (SIEM / ELK / autre) pour corrélation avec les logs applicatifs. [web:143][web:132]  
- Mise en place d’alertes sur :
  - pics de connexions échouées,  
  - changements de rôles inattendus,  
  - accès anormal à des tables sensibles (volume ou heures inhabituelles). [web:133][web:149]  

---

## 9. Sauvegardes, restauration & dumps

- Sauvegardes régulières de toutes les bases de données critiques, avec rétention adaptée au besoin métier et réglementaire. [web:132][web:149]  
- Sauvegardes **chiffrées** (au moins au niveau du stockage des backups).  
- Tests de restauration planifiés (exercices réguliers) pour vérifier la capacité à reconstruire un environnement à partir des sauvegardes. [web:139][web:143]  
- Interdiction de laisser des fichiers de dump (ex : `.sql`, `.bak`, `.dump`) non chiffrés sur des systèmes partagés ou des buckets accessibles. [web:136][web:133]  

---

## 10. Bases de données NoSQL & Cloud managées

### 10.1. NoSQL (MongoDB, Redis, Elasticsearch, etc.)

- Authentification et contrôle d’accès **obligatoires** (pas de DB NoSQL accessible sans mot de passe/clé). [web:141][web:137]  
- TLS activé entre clients et cluster, et si possible entre nœuds.  
- Désactivation de fonctions dangereuses (`eval`, opérateurs `$where`, scripts non contrôlés). [web:141][web:144]  

### 10.2. Bases gérées (RDS, Cloud SQL, Cosmos DB, etc.)

- Utiliser les options de sécurité fournies par le cloud :
  - chiffrement au repos et en transit,  
  - restrictions réseau (VPC, firewall, peering),  
  - intégration IAM (rôles cloud). [web:132][web:134]  
- Limiter l’usage de comptes “super admin” cloud et journaliser toutes les actions d’admin via les logs cloud natifs.  

---

## 11. Tests de sécurité base de données

- Intégrer la base de données dans les **tests de sécurité applicatifs** :
  - tests d’injection (via les API/backend),  
  - tests d’accès non autorisés (tentatives de lecture/écriture hors périmètre). [web:135][web:143]  
- Réaliser périodiquement des **revues de configuration DB** (checklist de durcissement du moteur utilisé). [web:136][web:139]  
- Inclure la DB dans le périmètre des **tests d’intrusion** backend (accès direct DB, élévation de privilèges, exfiltration). [web:143][web:131]  

---

## 12. Gouvernance & responsabilités

- Un ou plusieurs **DBA / responsables DB** sont désignés pour :
  - maintenir la configuration de sécurité,  
  - gérer les comptes et rôles,  
  - suivre les mises à jour et avis de sécurité du moteur DB. [web:131][web:139]  
- Les développeurs doivent suivre les **guides OWASP (SQL Injection, Database Security Cheat Sheets)** pour toute évolution schema / requêtes. [web:135][web:138]  

---

## Conclusion

Cette politique définit les contrôles minimaux pour sécuriser les bases de données :

- Architecture réseau cloisonnée  
- Comptes et privilèges strictement limités  
- Chiffrement au repos et en transit  
- Logs et audit des actions sensibles  
- Sauvegardes sécurisées et restaurables  
- Durcissement spécifique pour SQL, NoSQL et bases cloud

Elle doit être appliquée et adaptée pour chaque moteur (PostgreSQL, MySQL/MariaDB, SQL Server, Oracle, MongoDB, Redis, Elasticsearch, etc.) via des guides de durcissement propres à chaque technologie.
