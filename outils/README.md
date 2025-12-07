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
