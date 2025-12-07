# Politique de Sécurité Backend
**Document de référence pour les APIs et services backend (REST, GraphQL, gRPC, monolithes, microservices)**

---

## 1. Introduction

Le backend est l’autorité finale en matière de sécurité : il gère l’authentification, l’autorisation, la validation des données, les accès aux bases de données, les logs et la logique métier.  
Cette politique définit les mesures à appliquer à chaque service backend, de la conception jusqu’à l’exploitation (dev, CI/CD, prod, monitoring).  

Elle est alignée avec les bonnes pratiques OWASP, en particulier OWASP ASVS (Application Security Verification Standard), OWASP Top 10 et OWASP API Security.  

---

## 2. Principes de base

1. **Ne jamais faire confiance au client** : tout ce qui vient du frontend, d’une autre API ou d’un partenaire est considéré comme non fiable.  
2. **Moindre privilège** : chaque composant (user, service, compte technique) ne possède que les droits minimums nécessaires.  
3. **Défense en profondeur** : les protections sont empilées (validation, auth, filtrage réseau, logs, WAF, etc.).  
4. **Traçabilité** : tous les événements critiques sont journalisés, corrélés et surveillés.  

---

## 3. Authentification & gestion des identités

### 3.1. Mots de passe

- Hashage avec un algorithme moderne (Argon2id ou bcrypt avec facteur de coût élevé).  
- Salt unique par mot de passe, généré de manière sécurisée.  
- Politique de mot de passe : longueur minimale, complexité raisonnable, tentatives limitées.  
- Fonction de réinitialisation sécurisée (token à durée limitée, usage unique, non prédictible).  

### 3.2. Sessions, tokens et SSO

- Sessions serveur signées ou tokens (JWT) signés avec clés fortes, rotation régulière des secrets de signature.  
- Rotation des refresh tokens, invalidation à la déconnexion ou en cas de suspicion (vol, fuite).  
- Temps de vie limité des tokens d’accès (access tokens courts, refresh tokens plus longs mais surveillés).  
- Pour le SSO : usage de protocoles standards (OAuth2, OpenID Connect) et d’un IdP fiable.  

### 3.3. MFA (authentification multi-facteurs)

- MFA obligatoire pour les comptes administrateurs et les accès sensibles (consoles, backoffice, panel support).  
- Gestion sécurisée des OTP / WebAuthn / facteurs additionnels.  

---

## 4. Autorisation (RBAC / ABAC) et contrôle d’accès

### 4.1. RBAC (Role-Based Access Control)

- Définir des rôles clairs : par exemple `USER`, `ADMIN`, `SUPPORT`, `SYSTEM`.  
- Associer les permissions aux rôles, **pas aux utilisateurs directement**.  
- Vérifier systématiquement le rôle côté backend avant toute action sensible (création/suppression, admin, accès à d’autres comptes).  

### 4.2. ABAC (Attribute-Based Access Control) – optionnel

- Ajouter des conditions basées sur des attributs (organisation, région, type de device, heure, IP, statut, etc.) pour les cas à risque.  
- Utiliser ABAC pour les scénarios avancés (multi-tenant, restrictions légales, contextuelles).  

### 4.3. Multi-tenant & séparation des données

- Toutes les requêtes doivent être filtrées par tenant/organisation au niveau backend (ex : `WHERE tenant_id = ?`).  
- Ne jamais utiliser un identifiant de tenant envoyé par le client comme unique source de vérité sans vérification.  
- Empêcher qu’un utilisateur d’un tenant A puisse accéder aux données d’un tenant B (BOLA/IDOR).  

---

## 5. Validation & sanitation des données côté serveur

### 5.1. Validation systématique

- Valider **tous** les inputs : path params, query, body, headers, fichiers uploadés.  
- Utiliser des schémas de validation (par ex. JSON Schema, DTO avec contraintes) pour chaque endpoint.  
- Imposer des limites : longueur maximale, types stricts (int, string, email, URL), formats (regex), valeurs autorisées (enum).  

### 5.2. Sanitation

- Normaliser/sanitiser les données avant stockage ou log (ex : supprimer les caractères de contrôle, neutraliser les séquences suspectes).  
- Éviter d’enregistrer du HTML/JS brut, ou le marquer comme tel et l’encoder correctement au moment de l’affichage côté frontend.  

---

## 6. Protection contre les injections (SQL, NoSQL, Commande)

### 6.1. SQL

- Utiliser des requêtes paramétrées / prepared statements ou un ORM sérieux.  
- Interdiction de concaténer des fragments SQL à partir de données utilisateur.  
- Limiter les droits de l’utilisateur de base de données utilisé par l’application (pas de `SUPER`, pas de `DROP` global, etc.).  

### 6.2. NoSQL (MongoDB, Elasticsearch, etc.)

- Empêcher l’injection d’opérateurs ou de pipelines non contrôlés (`$where`, agrégations dynamiques venant du client).  
- Valider strictement les structures de requête autorisées.  

### 6.3. Command injection

- Éviter autant que possible d’appeler le shell; si nécessaire, utiliser des APIs de haut niveau et whitelister les arguments.  
- Jamais de concat de strings utilisateur dans une commande shell.  

---

## 7. Sécurité des API (REST, GraphQL, gRPC)

### 7.1. Authentification obligatoire

- Tous les endpoints sont protégés par auth, sauf ceux explicitement publics (health check, landing API, etc.).  
- Éviter les “backdoors” techniques (endpoints de debug, flags non documentés).  

### 7.2. OWASP API Top 10 (principes clés)

- **BOLA / IDOR** : toujours vérifier que l’utilisateur a le droit d’accéder à l’objet ciblé (ex : `resource.owner_id == user.id`).  
- **Mass assignment** : n’exposer que les champs modifiables explicitement (DTO en entrée), pas de binding direct du body sur l’entité.  
- **Rate limiting** : limiter les requêtes sur les endpoints sensibles (login, reset, endpoints lourds).  
- **Exposition de données** : filtrer les champs renvoyés (pas de colonnes internes, pas de secrets dans les réponses API).  

### 7.3. GraphQL (si utilisé)

- Limiter la profondeur et la complexité des requêtes pour éviter les DoS logiques.  
- Désactiver ou restreindre l’introspection en production si possible.  
- Mettre en place un contrôle d’accès fin par type/champ (RBAC/ABAC au niveau du resolver).  

---

## 8. Chiffrement, secrets & configuration

### 8.1. Chiffrement en transit

- HTTPS/TLS obligatoire pour toutes les communications externes.  
- Chiffrement interne (mTLS) recommandé pour les environnements multi-tenant ou non totalement de confiance.  

### 8.2. Chiffrement au repos

- Chiffrer les colonnes sensibles (PII, mots de passe d’autres systèmes, tokens longue durée, données médicales, financières).  
- Utiliser des algos modernes (AES-GCM, ChaCha20-Poly1305) avec gestion correcte des IV et clés.  

### 8.3. Gestion des secrets

- Tous les secrets (mots de passe DB, clés API, clés JWT, tokens OAuth) sont stockés hors du code : variables d’environnement, coffre-fort de secrets (Vault, KMS, Secret Manager, etc.).  
- Rotation régulière des secrets critiques.  
- Interdiction de committer des secrets dans le dépôt ou les artefacts.  

---

## 9. Journaux, audit & monitoring

### 9.1. Journaux

- Journaliser les événements importants : connexions, échecs de login, changement de mots de passe, modifications de droits, actions critiques (suppression, exports massifs).  
- Utiliser un format structuré (JSON) pour permettre la recherche et la corrélation.  

### 9.2. Centralisation et alertes

- Envoyer les logs vers une plateforme centralisée (ELK, Loki, SIEM).  
- Configurer des alertes sur :
  - pics de 401/403  
  - augmentations de 5xx  
  - nombre élevé de tentatives de login/OTP échouées  
  - patterns anormaux (scans d’URLs, payloads suspects).  

### 9.3. Traçabilité & conformité

- Conserver les journaux d’audit (qui a fait quoi, quand) selon les exigences légales ou internes.  
- Protéger les logs contre la modification ou la suppression non autorisée.  

---

## 10. Gestion des erreurs & messages

- Ne jamais renvoyer de stacktrace brute au client.  
- Messages d’erreur côté client : génériques (ex : « Une erreur est survenue » ou « Accès refusé »).  
- Détails techniques (stack, query, config) uniquement dans les logs internes.  
- Éviter d’inclure des données sensibles dans les erreurs loguées (masker les PII, tokens, secrets).  

---

## 11. Durcissement de l’infrastructure backend

### 11.1. Serveurs et containers

- OS et middleware régulièrement mis à jour (patching).  
- Services inutiles désactivés, ports fermés.  
- Containers exécutés avec des utilisateurs non root et images minimales durcies.  

### 11.2. Réseau

- Segmentation réseau :  
  - front → gateway/API → services internes → DB.  
- Bases de données jamais exposées directement à Internet.  
- Pare-feu/WAF devant les services exposés publiquement.  

### 11.3. WAF et API Gateway

- WAF avec règles OWASP (ModSecurity CRS ou équivalent).  
- API Gateway pour centraliser :
  - auth et rate limiting  
  - quotas par client  
  - protection contre les attaques volumétriques simples.  

---

## 12. Cycle de vie, CI/CD & dépendances

### 12.1. Intégration continue (CI)

- Tests unitaires et d’intégration obligatoires sur chaque commit/merge.  
- Analyse statique (SAST) sur le code backend (ex : Semgrep, SonarQube, CodeQL).  
- Analyse des dépendances (SCA) : Dependabot, Snyk, `npm audit`/`pip-audit`/`composer audit`, etc.  

### 12.2. Déploiement (CD)

- Déploiements reproductibles et versionnés, avec possibilité de rollback.  
- Secrets injectés au runtime (env, secret manager), pas dans l’image ni le code.  
- Environnements isolés : dev, test, staging, prod avec règles de sécurité cohérentes.  

---

## 13. Résilience, sauvegardes & continuité

- Sauvegardes régulières des bases de données, configs critiques et secrets (sous forme chiffrée).  
- Procédures de restauration testées (exercices réguliers).  
- Limiter les droits sur les systèmes de backup pour réduire l’impact d’un ransomware ou d’un compte compromis.  

---

## 14. Gouvernance & processus de revue

### 14.1. Checklist sécurité par Pull Request

Avant chaque merge backend, vérifier :

- [ ] Authentification : gestion correcte des sessions/tokens, mots de passe hashés.  
- [ ] Autorisation : vérification des rôles et permissions côté serveur.  
- [ ] Validation des entrées : schémas, limites, sanitation.  
- [ ] Injections : pas de concat SQL, NoSQL, commandes.  
- [ ] API : pas de endpoint sensible sans auth, pas de mass assignment.  
- [ ] Logs : événements critiques journalisés, pas de données sensibles en clair.  
- [ ] Config : pas de secrets dans le code, variables d’environnement utilisées.  

### 14.2. Revue de sécurité périodique

- Revue régulière des règles de firewall, WAF, gateway.  
- Revue des rôles et permissions utilisateurs/admins.  
- Tests d’intrusion/pentest backend périodiques.  

---

## 15. Niveaux de criticité & exigences associées

### Niveau 1 – Services non sensibles

- Ex : contenus publics, blogs, landing pages.  
- Exigences réduites mais toujours :
  - Auth/autorisation correctes si nécessaire  
  - Protection injection  
  - Logs basiques  

### Niveau 2 – Services sensibles (B2B, SaaS)

- Ex : plateformes clients, données personnelles.  
- Exigences :
  - Application complète de cette politique  
  - SAST + SCA + revue de code  
  - Monitoring et alertes en place  

### Niveau 3 – Services critiques (finance, santé, infra)

- Ex : paiement, données médicales, systèmes critiques.  
- Exigences supplémentaires :
  - MFA étendu  
  - Revue de sécurité externe (audit/pentest)  
  - Chiffrement renforcé (au repos + mTLS)  
  - Processus d’incident response documenté et testé  

---

## Conclusion

Cette politique définit un cadre complet pour sécuriser toute application backend :

- Authentification et sessions robustes  
- Autorisation (RBAC/ABAC) et isolation multi-tenant  
- Validation et protection contre les injections  
- Sécurisation des API (REST/GraphQL/gRPC)  
- Chiffrement, gestion des secrets et configuration  
- Journaux, audit, monitoring et alertes  
- Durcissement de l’infrastructure et sécurisation du cycle de vie (CI/CD)  

Ce document doit être adapté à chaque projet en fonction de sa criticité, mais les principes généraux restent obligatoires pour tout backend mis en production.
