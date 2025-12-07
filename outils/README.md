# Outils pour la vérification de la sécurité frontend

## 1. Outils navigateurs

- DevTools du navigateur (Chrome, Firefox, Edge)  
  - Vérifier les en-têtes HTTP (CSP, HSTS, X-Frame-Options, Referrer-Policy, etc.). [web:76]  
  - Inspecter cookies, localStorage, sessionStorage, IndexedDB. [web:82]  
  - Contrôler les erreurs JS et les logs dans la console. [web:72]  

## 2. Tests end‑to‑end (E2E) / UI

- Cypress  
  - Scénarios login/logout, routing protégé, redirections, formulaires, XSS basiques. [web:81]  

- Playwright  
  - Alternative à Cypress pour tester les mêmes scénarios de sécurité en multi‑navigateurs. [web:87]  

## 3. Analyse statique / qualité du code

- ESLint + plugins sécurité  
  - Détection d’`eval`, `innerHTML` dangereux, patterns XSS/DOM, mauvaises pratiques JS. [web:76]  

- Semgrep  
  - Règles dédiées JS/TS/frontend (XSS, DOM-based XSS, secrets dans le code, etc.). [web:74]  

- npm audit / Snyk  
  - Analyse des vulnérabilités dans les dépendances npm du frontend. [web:61]  

## 4. Proxies et scanners de sécurité

- OWASP ZAP  
  - Proxy interceptant pour voir et modifier les requêtes, scanner XSS, headers, redirections, etc. [web:77]  

- Burp Suite (Community ou Pro)  
  - Tests plus avancés sur les flux front/API, détection de failles web classiques. [web:80]  

## 5. Monitoring / erreurs côté client

- Sentry  
  - Remontée d’erreurs JS et d’exceptions côté navigateur (staging + prod). [web:75]  

- Datadog / LogRocket (optionnel)  
  - Monitoring des sessions utilisateurs et comportements anormaux du frontend. [web:63]  

## 2.4. Outils de test et de vérification sécurité backend

Les outils suivants sont recommandés pour vérifier la conformité du backend avec cette politique et les référentiels OWASP (ASVS, OWASP Top 10, OWASP API Security). [web:16][web:18]

### 2.4.1. Tests manuels & proxies

- **Navigateurs + DevTools**  
  - Vérification des réponses API (codes, headers sécurité, messages d’erreur). [web:82]

- **Proxies & scanners web**  
  - **OWASP ZAP** : proxy + scanner DAST pour tester XSS, injections, erreurs, headers, API REST/GraphQL. [web:77][web:88]  
  - **Burp Suite (Community/Pro)** : tests manuels avancés sur les endpoints sensibles (auth, sessions, injections, IDOR/BOLA). [web:80][web:90]

### 2.4.2. Tests automatisés (CI/CD)

- **Tests E2E / API**  
  - Postman / Newman, k6, Playwright API, ou équivalent : scénarios automatiques sur les endpoints backend (auth, droits, erreurs, limites). [web:78][web:72]

- **SAST (Static Application Security Testing)**  
  - **Semgrep**, **SonarQube**, **CodeQL** : analyse du code backend pour détecter injections, mauvaises pratiques crypto, auth fragile, etc. [web:115][web:119]

- **SCA (Software Composition Analysis)**  
  - **Dependabot**, **Snyk**, `npm audit` / `pip-audit` / `composer audit` : détection des vulnérabilités dans les dépendances et frameworks. [web:61][web:118]

### 2.4.3. Monitoring & détection en production

- **Plateformes de logs & APM**  
  - ELK / Loki / Datadog / New Relic : centralisation des logs backend, métriques, alertes (pics de 401/403, 5xx, erreurs répétées). [web:79][web:118]

- **Gestion d’incidents / SIEM**  
  - Intégration éventuelle avec un SIEM (Splunk, Wazuh, etc.) pour corréler les logs backend avec les autres composants (frontend, réseau, IAM). [web:97][web:115]
