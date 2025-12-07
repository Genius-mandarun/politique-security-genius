# Politique de Gestion des Mots de Passe
**Document de référence pour tous les comptes : utilisateurs, administrateurs, comptes de service et techniques**

---

## 1. Objectif et portée

Cette politique définit les exigences minimales de sécurité pour tous les mots de passe utilisés dans l’organisation, y compris :

- Comptes utilisateurs (clients, collaborateurs, partenaires).
- Comptes administrateurs (systèmes, infrastructure, applications, cloud).
- Comptes de service et comptes techniques (services Windows/Linux, jobs, intégrations, comptes applicatifs).
- Comptes locaux et comptes d’urgence.

Elle s’aligne sur les recommandations modernes (NIST SP 800‑63B et bonnes pratiques industrie). [web:150][web:153][web:164]

---

## 2. Exigences générales de complexité et de longueur

### 2.1. Longueur

- Longueur minimale recommandée : **12 à 16 caractères** pour les comptes standards. [web:153][web:168]
- Longueur minimale pour les comptes à privilèges (admins, comptes critiques) : **16 à 20+ caractères**. [web:159][web:162]
- Longueur maximale supportée : au moins **64 caractères**, sans restriction artificielle sur les caractères utilisables. [web:150][web:155]

### 2.2. Complexité

- Priorité donnée aux **phrases de passe** (plusieurs mots, espaces autorisés) plutôt qu’à une complexité arbitraire difficile à retenir. [web:153][web:163]
- Tous les types de caractères doivent être acceptés (lettres, chiffres, symboles, espaces). [web:151][web:155]
- Pas d’exigence de rotation périodique “automatique” (tous les 30/60/90 jours) sauf en cas de suspicion de fuite, changement de rôle ou risque identifié. [web:153][web:157]

---

## 3. Blocage des mots de passe faibles

### 3.1. Blocklist et mots de passe compromis

Les systèmes d’authentification doivent :

- Rejeter les mots de passe figurant dans des **listes de mots de passe compromis** (fuites connues, dictionnaires publics). [web:150][web:157]
- Rejeter les mots de passe trop courants :  
  Exemples : [translate:password123], [translate:qwerty], [translate:azerty], [translate:123456789], etc. [web:157][web:168]
- Rejeter les mots de passe contenant des informations personnelles évidentes (nom/prénom, nom de l’entreprise, adresse e‑mail du compte, etc.). [web:154][web:160]

### 3.2. Réutilisation

- Interdiction de réutiliser les anciens mots de passe récents (historique configurable selon la criticité du système). [web:154][web:160]
- Recommandation forte de ne **jamais réutiliser un mot de passe** entre plusieurs services ou systèmes (gestionnaire de mots de passe recommandé). [web:150][web:163]

---

## 4. Politique par type de compte

### 4.1. Comptes utilisateurs (clients, employés, partenaires)

- Longueur minimale : 12–16 caractères ; phrase de passe encouragée. [web:153][web:163]
- Complexité raisonnable (éventuellement 3 types de caractères sur 4) sans imposer de règles impossible à mémoriser. [web:155][web:168]
- MFA fortement recommandé pour l’accès aux zones sensibles (espace client, données personnelles, configuration critique). [web:154][web:160]

### 4.2. Comptes administrateurs / comptes à privilèges

- Longueur minimale : 16–20+ caractères, générés idéalement par un gestionnaire de mots de passe. [web:159][web:162]
- MFA **obligatoire** (TOTP, WebAuthn, clé U2F, carte à puce, selon le contexte). [web:158][web:160]
- Politiques plus strictes autorisées (fine‑grained password policy dans AD, GPO spécifiques, etc.). [web:162][web:167]
- Interdiction de se connecter avec un compte admin pour des usages non administratifs (navigation web, mail, bureautique).

### 4.3. Comptes de service et comptes applicatifs

- Mots de passe générés de manière **aléatoire** via un générateur cryptographique ou directement par le coffre de secrets. Longueur recommandée : 20+ caractères. [web:155][web:156]
- Stockage **uniquement** dans un coffre de secrets / gestionnaire sécurisé (Vault, KMS, Secret Manager, PAM, etc.), jamais dans le code, les dépôts Git ou des fichiers en clair. [web:156][web:160]
- Rotation régulière planifiée (ex. tous les 90 jours ou plus fréquemment selon la criticité) et rotation immédiate en cas de suspicion de fuite. [web:156][web:169]

---

## 5. Stockage, transmission et réinitialisation

### 5.1. Stockage

- Les mots de passe ne sont jamais stockés en clair.  
- Côté serveur, ils sont **hachés** avec un algorithme moderne (Argon2id, bcrypt, scrypt) avec paramètres de coût adaptés et salage unique. [web:155][web:164]
- Interdiction totale de stocker des mots de passe dans :
  - logs,  
  - tickets de support,  
  - e‑mails, chats, documents partagés,  
  - captures d’écran ou outils de suivi. [web:154][web:160]

### 5.2. Transmission

- Toute saisie de mot de passe doit être effectuée sur des connexions chiffrées (HTTPS/TLS). [web:154][web:146]
- Les mots de passe ne doivent **jamais** être envoyés en clair par e‑mail ou SMS.  
- La réinitialisation doit se faire via un lien unique, limité dans le temps, ou via un canal sécurisé (SSO, portail interne). [web:154][web:160]

### 5.3. Réinitialisation / oubli

- En cas d’oubli, l’utilisateur reçoit un lien de réinitialisation à usage unique (token signé, expiration courte). [web:154][web:160]
- Les procédures de reset pour comptes à privilèges exigent **une étape d’authentification forte supplémentaire** (MFA, vérification par support avec procédure). [web:156][web:169]

---

## 6. Verrouillage de compte & protection contre brute‑force

- Limiter les tentatives de login par compte et/ou par IP, avec backoff exponentiel ou délai entre tentatives. [web:154][web:163]
- Après plusieurs échecs (ex. 5–10 tentatives), appliquer :
  - un verrouillage temporaire, ou  
  - un défi (captcha, MFA additionnelle), selon le contexte. [web:154][web:160]
- Les administrateurs doivent surveiller les logs d’authentification pour repérer :
  - des nombres anormalement élevés d’échecs,  
  - des accès depuis des pays/AS inhabituels,  
  - des comportements automatiques (brute‑force, credential stuffing). [web:160][web:163]

---

## 7. MFA (authentification multi‑facteurs)

- MFA obligatoire pour :
  - tous les comptes administrateurs,  
  - accès à la console d’admin, panneaux backoffice, interfaces de gestion critiques,  
  - accès aux secrets (vaults, consoles cloud). [web:158][web:160]
- MFA fortement recommandé pour les comptes utilisateurs sensibles (accès à des informations financières, données de santé, etc.). [web:154][web:163]

---

## 8. Sensibilisation et bonnes pratiques utilisateur

- Les utilisateurs doivent être informés que :
  - un mot de passe **ne doit jamais être partagé**, même avec l’IT ou la sécurité,  
  - un gestionnaire de mots de passe est recommandé pour générer et stocker des mots de passe uniques et longs,  
  - les mêmes mots de passe ne doivent pas être réutilisés entre services personnels et professionnels. [web:150][web:160]
- Des campagnes régulières (e‑learning, rappels, guides) rappellent les bonnes pratiques et les risques liés aux mots de passe faibles ou réutilisés. [web:160][web:163]

---

## 9. Contrôles techniques et audits

- Les systèmes d’authentification doivent :
  - appliquer la blocklist de mots de passe compromis,  
  - enregistrer les tentatives de connexion et les changements de mots de passe,  
  - offrir des mécanismes de MFA. [web:150][web:168]
- Des audits réguliers doivent vérifier :
  - l’absence de stockage en clair ou dans le code,  
  - la configuration correcte des algorithmes de hash,  
  - la présence de politiques spécifiques pour comptes à privilèges et comptes de service. [web:154][web:156]

---

## 10. Exceptions et cas particuliers

- Toute exception à cette politique (ancien système, contrainte technique) doit :
  - être documentée,  
  - être approuvée par la fonction sécurité,  
  - comporter des mesures compensatoires (MFA, restrictions réseau, monitoring renforcé, etc.). [web:160][web:169]

---

## Conclusion

Cette politique fournit un cadre global pour la gestion des mots de passe dans l’organisation :

- Longueur suffisante et blocklist des mots de passe faibles/compromis.
- Distinction claire entre comptes utilisateurs, administrateurs et comptes de service.
- Stockage sécurisé (hash moderne), transmission chiffrée et réinitialisation contrôlée.
- Intégration de MFA pour les usages sensibles et les comptes à privilèges.
- Processus d’audit et de sensibilisation continue.

Elle doit être appliquée dans l’ensemble des systèmes (applications internes, SaaS, infrastructure, annuaires, bases de données, équipements réseau) et adaptée si nécessaire en fonction des contraintes réglementaires et métiers.
