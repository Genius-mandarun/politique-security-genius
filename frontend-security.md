# Politique de Sécurité Frontend
**Document de référence pour les applications web modernes (SPA, PWA, sites classiques)**

---

## 1. Introduction

Le frontend représente une surface d'attaque importante : il s'exécute dans le navigateur, sur un environnement non maîtrisé, et manipule directement le DOM, les données utilisateur, les tokens, les scripts tiers, et les interactions avec l'API.

Cette politique définit toutes les mesures à appliquer dans toutes les phases : conception, développement, build, déploiement, exploitation.

Elle est alignée avec les bonnes pratiques OWASP, en particulier OWASP ASVS (Application Security Verification Standard) et OWASP Top 10.

---

## 2. Politique de contenu & en-têtes de sécurité

### 2.1. Content Security Policy (CSP) stricte

**Objectif :** empêcher l'exécution de scripts injectés (XSS).

**Exemple de CSP solide :**

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://apis.google.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data:;
  object-src 'none';
  frame-ancestors 'none';
```

### 2.2. Headers de sécurité à respecter

Le frontend doit concevoir ses composants pour fonctionner avec :

- **X-Content-Type-Options:** nosniff
- **X-Frame-Options:** DENY
- **Referrer-Policy:** strict-origin-when-cross-origin
- **Permissions-Policy:** camera=(), geolocation=(), microphone=()
- **Cross-Origin-Opener-Policy:** same-origin

### 2.3. HTTPS et HSTS

Le frontend ne doit jamais contenir :

- URLs en HTTP
- Ressources mixtes

Exemple : pas de `<img src="http://...">`.

---

## 3. Gestion sécurisée des données côté navigateur

### 3.1. Interdiction de stocker des informations sensibles

Ne jamais stocker dans `localStorage` / `sessionStorage` :

- tokens JWT
- données personnelles sensibles
- identifiants ou rôles de sécurité

**Utiliser des cookies HttpOnly + Secure + SameSite.**

### 3.2. Minimalisme du stockage

Garder uniquement l'essentiel : thème, langue, préférences non sensibles.

Si un stockage sensible est nécessaire → le chiffrer (ex : CryptoJS).

```javascript
const encrypted = CryptoJS.AES.encrypt(email, SECRET);
localStorage.setItem("email", encrypted);
```

### 3.3. Aucun log sensible

Ne jamais logguer :

- tokens
- réponses API complètes
- erreurs techniques internes

En production, réduire les logs au minimum.

---

## 4. XSS, Injection DOM & traitement de l'entrée utilisateur

### 4.1. L'entrée utilisateur est toujours non fiable

Toujours valider :

- format
- longueur
- type
- caractères autorisés

### 4.2. Encodage systématique à l'affichage

Éviter tout ce qui insère du HTML brut.

**Mauvais :**
```javascript
element.innerHTML = input;
```

**Bon :**
```javascript
element.textContent = input;
```

### 4.3. Interdiction des fonctions dangereuses

Ne jamais utiliser :

- `eval()`
- `new Function()`
- `document.write()`
- `innerHTML` non contrôlé
- `setTimeout("string")`

### 4.4. Sanitize obligatoire pour du HTML dynamique

Exemple avec DOMPurify :

```javascript
const safe = DOMPurify.sanitize(userHtml);
container.innerHTML = safe;
```

---

## 5. Authentification, sessions et protection CSRF côté client

### 5.1. Cookies HttpOnly respectés

Le frontend ne doit jamais tenter d'accéder aux cookies HttpOnly via `document.cookie`.

### 5.2. Flows login / logout sécurisés

À la déconnexion :

- purge du storage
- purge des caches PWA
- redirection vers `/login`
- effacement de toute donnée temporaire

```javascript
localStorage.clear();
sessionStorage.clear();
window.location.href = "/login";
```

### 5.3. Support des tokens CSRF

Si l'API fournit un token anti-CSRF, le frontend doit le renvoyer systématiquement.

```javascript
fetch("/api/update", {
  method: "POST",
  headers: { "X-CSRF-Token": csrfToken }
});
```

### 5.4. Interdiction des requêtes sensibles cross-origin automatiques

Pas de POST/PUT/DELETE envoyés sans contrôle explicite.

---

## 6. Scripts tiers, iframes & supply chain

### 6.1. Minimiser et auditer les scripts tiers

Charger uniquement les scripts indispensables (analytics, chat…).

### 6.2. Subresource Integrity (SRI)

Protéger les scripts CDN :

```html
<script src="https://cdn…" 
        integrity="sha384-xxxx" 
        crossorigin="anonymous"></script>
```

### 6.3. Protection contre le clickjacking

Côté JS :

```javascript
if (window.self !== window.top) {
  window.top.location = window.location;
}
```

---

## 7. UX de sécurité & gestion des erreurs

### 7.1. Messages d'erreur génériques

Pas de détails techniques.

**Mauvais :**
```
"Erreur : utilisateur introuvable dans SQL."
```

**Bon :**
```
"Identifiants incorrects."
```

### 7.2. Gestion UI du token expiré

- Détecter le 401/403
- Nettoyer les données
- Rediriger vers la page login
- Afficher un message simple : "Votre session a expiré, merci de vous reconnecter."

### 7.3. Pas de secrets dans le bundle JS

Ne jamais inclure une clé API privée visible dans le code compilé.

---

## 8. Sécurité du routing, navigation & redirections

### 8.1. Masquage des fonctionnalités non autorisées

Un utilisateur non admin ne doit pas voir les boutons admin.

Utiliser des routes protégées (ProtectedRoute, guards Angular, etc.).

### 8.2. Nettoyage des paramètres d'URL

Les query string peuvent contenir du JavaScript.

**À faire :** encoder/échapper systématiquement avant affichage dans le DOM.

### 8.3. Redirections sécurisées

Toujours vérifier que la redirection cible appartient au domaine attendu.

**Ne jamais faire :**
```javascript
window.location.href = userProvidedUrl;
```

**Valider la destination :**
```javascript
const allowedDomains = ["https://example.com", "https://partner.com"];
if (allowedDomains.some(d => redirectUrl.startsWith(d))) {
  window.location.href = redirectUrl;
}
```

---

## 9. Upload de fichiers & sécurisation côté client

### 9.1. Filtrage stricte des fichiers

```html
<input type="file" accept="image/png,image/jpeg">
```

### 9.2. Vérification du type MIME et de la taille

```javascript
if (!file.type.startsWith("image/")) {
  alert("Format non autorisé");
}
if (file.size > 2 * 1024 * 1024) {
  alert("Fichier trop grand");
}
```

### 9.3. Prévisualisation sûre

Utiliser `URL.createObjectURL()` pour les images.

Ne jamais rendre un fichier utilisateur comme HTML/JS dans une iframe non sandboxée.

---

## 10. Permissions navigateur & APIs sensibles

### 10.1. Gestion explicite des permissions

- Caméra, micro, géolocalisation, notifications, clipboard, Web Bluetooth, etc.
- Ne demander la permission que quand c'est nécessaire, pas au chargement de la page.

### 10.2. Clipboard / copy-paste

- Attention quand tu écris dans le presse-papiers (pas de données sensibles).
- Éviter de lire le presse-papiers sauf besoin clair, et informer l'utilisateur.

### 10.3. Web Storage / WebAuthn / WebCrypto

- Bien encadrer les usages de WebCrypto (clé dérivée de mot de passe, etc.).
- Documenter clairement les données qui restent dans le navigateur et comment les purger (logout, reset).

---

## 11. PWA, Service Workers & gestion du cache

### 11.1. Ne pas mettre en cache des données sensibles

Le service worker doit exclure les réponses API privées.

Ne pas mettre en cache (Cache Storage) les réponses contenant :

- données privées
- tokens
- informations sensibles

### 11.2. Purge des caches au logout

```javascript
caches.keys().then(keys => {
  keys.forEach(key => caches.delete(key));
});
```

### 11.3. Scope minimal du service worker

Limiter le scope à ce qui est nécessaire (ex : `/app/` plutôt que `/`).

---

## 12. Build, sourcemaps & livraison du code

### 12.1. Pas de sourcemaps exposés en production

Les sourcemaps révèlent tout le code source.

Si nécessaire, les mettre sur un domaine interne / protégé.

### 12.2. Minification & tree-shaking

- Réduit la taille
- Supprime du code inutile
- Masque la logique interne

### 12.3. Versionnement par hash

Utiliser :
```
main.32ab7d.js
```

Au lieu de :
```
main.js
```

Cela force le navigateur à récupérer la bonne version et évite le cache poisoning.

---

## 13. Monitoring sécurité côté UI

### 13.1. Collecter les erreurs JS sans données sensibles

Envoyer (de manière anonyme) les erreurs vers un service de monitoring :

- Sentry
- Datadog
- LogRocket

### 13.2. Circuit breaker

Si plusieurs API critiques échouent :

- Afficher un message global de sécurité
- Désactiver temporairement les actions sensibles

---

## 14. UX Anti-phishing

### 14.1. Indiquer clairement les liens externes

- Icône de lien externe
- Couleur différente
- Confirmation avant d'ouvrir un site de paiement

### 14.2. Double confirmation pour actions sensibles

Ex : suppression de compte, changement de mot de passe, transactions.

### 14.3. Avertissement avant de quitter le domaine

Surtout pour les paiements ou les partenaires tiers.

---

## 15. Mapping formel OWASP ASVS

### 15.1. Objectif

Pour chaque section du document, pointer vers les exigences ASVS correspondantes.

### 15.2. Exemples de mapping

| Section du document | Exigences ASVS | Contrôles |
|---|---|---|
| XSS & DOM | V5, V18 | Encodage, validation, sanitize |
| Auth & sessions | V2, V3 | Gestion des tokens, cookies HttpOnly |
| Storage | V9 | Pas de secrets en localStorage |
| Erreurs & logging | V10 | Messages génériques, logs sans PII |
| Scripts tiers & configuration | V14 | SRI, CSP, audit des dépendances |
| Routing & DOM | V18 | Contrôle d'accès UI, redirections |
| Upload | V18 | Filtrage, type MIME, taille |
| PWA & caches | V14 | Pas de cache sensible |

### 15.3. Exigences complémentaires ASVS niveau 2/3

- Protections multilingues et i18n
- Gestion "untrusted data in UI" avancée
- Encodage fin selon niveau ASVS

---

## 16. Automatisation & tests de sécurité frontend

### 16.1. Tests unitaires sécurité

- Vérification de l'encodage des données
- Absence d'API dangereuses (`eval`, `innerHTML` non contrôlé…)
- Tests de composants qui encodent correctement la sortie

### 16.2. Tests E2E sécurité

- Simulation XSS (injection payload dans formulaires)
- Simulation redirections ouvertes
- Tests de roles UI (admin vs user)
- Tests du comportement après expiration de session
- Vérification du logout (purge du storage, caches)

### 16.3. Analyse statique JS/TS dans la CI

Intégrer :

- **ESLint** avec règles sécurité (eslint-plugin-security, eslint-plugin-react-security)
- **Semgrep** (règles XSS, DOM, patterns dangereux)
- Pipeline bloque si violation critique

---

## 17. Anti-abuse & protection contre l'automatisation UI

### 17.1. Délais et limitations côté client

- Délais entre tentatives de soumission
- Désactivation temporaire d'un bouton après X erreurs
- Captcha (Turnstile, reCAPTCHA) déclenché automatiquement après X tentatives

### 17.2. UI résistante aux brute-force

- Affichage de messages progressifs ("Trop de tentatives, réessayez dans X secondes")
- Limitations visibles du nombre de requêtes

### 17.3. Protections contre overlays trompeurs / UI clickjacking avancée

- Vérifier les iframes (ne pas afficher les éléments critiques dans une iframe externe)
- Informer l'utilisateur de tout changement de contexte
- Double confirmation pour actions sensibles

---

## 18. Multi-tenant, i18n & accessibilité liée à la sécurité

### 18.1. Multi-tenant

- Ne pas exposer `tenantId` comme élément de confiance
- Cloisonner UI (thème, logo, environnements)
- Appliquer les règles par organisation dans l'app

### 18.2. Internationalisation (i18n)

- Échapper les textes venant des fichiers de traduction
- Éviter le HTML dans les JSON linguistiques
- Empêcher l'injection via une chaîne traduite (utiliser `textContent` plutôt que `innerHTML`)

### 18.3. Accessibilité et sécurité

- Dialogues critiques accessibles (ARIA)
- Focus automatique sur les actions sensibles
- Messages d'erreur lisibles via lecteur d'écran
- Confirmations visuelles et auditives

---

## 19. Gouvernance & processus de revue sécurité

### 19.1. Checklist obligatoire par Pull Request

À vérifier avant tout merge :

- [ ] XSS / DOM (pas de innerHTML, eval, encoding OK)
- [ ] Routing / redirections (domaine vérifié, routes protégées)
- [ ] Storage (pas de tokens en localStorage)
- [ ] Erreurs et messages (pas de détails techniques)
- [ ] Scripts tiers & SRI (intégrité vérifiée)
- [ ] PWA & caches (pas de données sensibles)
- [ ] Permissions navigateur (caméra/micro/GPS à la demande)
- [ ] Upload (filtrage, taille, type MIME)
- [ ] Build (pas de secrets, sourcemaps gérés)

### 19.2. Gestion des secrets frontend

- Scans automatiques (GitLeaks, TruffleHop)
- Hooks pre-commit pour détecter les secrets
- Procédure en cas de fuite :
  - Rotation des clés
  - Révocation
  - Redeploy complet du bundle

### 19.3. Formation continue

L'équipe doit être formée régulièrement sur :

- OWASP ASVS
- OWASP Top 10
- Nouvelles attaques DOM
- Cas d'étude d'incidents réels

---

## 20. Table de criticité & exigences associées

### Niveau 1 – Applications simples

**Exemples :** blog, site vitrine.

**Exigences :**
- Contrôles essentiels uniquement (CSP basique, XSS, HTTPS)
- Sous-ensemble de la checklist

### Niveau 2 – Applications sensibles (B2B, SaaS)

**Exemples :** plateforme SaaS, app collaborative, e-commerce.

**Exigences :**
- Appliquer le document complet
- Tests automatisés (unit + E2E)
- Checklist PR obligatoire
- Formation de l'équipe

### Niveau 3 – Applications critiques (finance, santé, infra)

**Exemples :** banque, plateforme médicale, système critique.

**Exigences supplémentaires :**
- Logs UI avancés (traçabilité des actions)
- Revues sécurité systématiques (architecture, code)
- Durcissement PWA renforcé
- Double confirmation systématique + logs d'audit
- Mapping ASVS niveau 3 complet
- Pen-testing régulier
- Incident response plan

---

## Conclusion

Cette politique constitue une référence complète pour sécuriser toute application frontend :

- **Techniques :** XSS, CSRF, sessions, routing, scripts tiers, DOM, i18n, PWA, automatisation
- **Processus :** gouvernance, revue PR, checklist, formation, gestion des secrets
- **Standards :** alignement ASVS, OWASP Top 10
- **Contexte :** multi-tenant, accessibilité, anti-phishing, monitoring

Elle est exhaustive, professionnelle, et adaptée aux standards industriels.

**Statut :** Document de référence — À adapter par projet selon le niveau de criticité.

---

**Version :** 1.0  
**Date :** Décembre 2025  
**Auteur :** Équipe Sécurité  
**Révision :** Annuelle ou à la suite d'incident
