# Stack Technique Frontend – Entreprise IT

Ce document liste uniquement les **technologies frontend** à utiliser pour construire l’interface du site de l’entreprise IT.

---

## 1. Framework Frontend

### Next.js 14 (React)
Framework moderne basé sur React.

**Avantages :**
- Très performant  
- Facile à déployer  
- Compatible SEO  
- Standard dans l’industrie  

---

## 2. Langages & standards

- **TypeScript** (obligatoire)  
- **JavaScript moderne (ES2023)**  
- **HTML5**  
- **CSS3**  

---

## 3. UI & Design System

### Tailwind CSS
Outil de stylisation moderne, utility-first, sans injection CSS dynamique.

### Bibliothèques de composants

- **shadcn/ui** (React + Tailwind) – composants réutilisables, theming propre  
- **Radix UI** – composants accessibles (ARIA) et sécurisés  

---

## 4. Build & Bundling

*(géré automatiquement par Next.js)*

- **SWC / Turbopack** (compiler Next.js)  
- Minification  
- Tree-shaking  

---

## 5. Gestion d’images & assets

- Optimisation intégrée de Next.js (`next/image`)  
- Formats modernes : **WebP**, **AVIF**  

---

## 6. Typographie & icônes

- **Google Fonts optimisées Next.js** (via `next/font`)  
- **Lucide Icons** ou **Heroicons**  

---

## 7. Formulaires côté frontend

- **React Hook Form**  
- **Zod** (validation schéma côté client)  

---

## 8. Sécurité frontend (technos & configuration)

### 8.1 Librairies

- **DOMPurify** (uniquement si HTML dynamique externe doit être affiché)  
- Optionnel : helper type Helmet pour faciliter la configuration CSP si besoin côté app  

### 8.2 CSP & headers

- **Content Security Policy (CSP)** configurée au niveau de la plateforme (Vercel / Cloudflare) et vérifiée avec DevTools (onglet Network / Security).  
- Headers de sécurité à appliquer via l’hébergement / reverse proxy :  
  - `X-Frame-Options: DENY`  
  - `X-Content-Type-Options: nosniff`  
  - `Referrer-Policy: strict-origin-when-cross-origin`  
  - `Permissions-Policy` (désactivation caméra/micro/GPS par défaut)  

---

## 9. Outils Dev & Qualité

- **ESLint**  
  - Configuration incluant des **règles de sécurité** (plugins recommandés selon stack).  
- **Prettier**  
- **Husky** (hooks Git)  
- **Lint-staged** (lint sur fichiers modifiés)  

---

## 10. Hébergement frontend recommandé

- **Vercel**  
- **Cloudflare Pages**

Les deux offrent :
- HTTPS automatique  
- CDN global  
- Intégration facile avec Next.js  
- Support des headers de sécurité et de la CSP  

---

## 11. CDN & optimisation

- CDN global intégré à Vercel ou Cloudflare  
- Cache automatique  
- Compression Brotli  
- Gestion des assets versionnés (hash dans les noms de fichiers)  

---

# Résumé rapide

| Domaine        | Technologie                           |
|---------------|----------------------------------------|
| Framework      | Next.js 14 (React)                    |
| Langage        | TypeScript                            |
| UI             | Tailwind CSS                          |
| Composants     | shadcn/ui, Radix UI                   |
| Formulaires    | React Hook Form + Zod                 |
| Sécurité front | CSP + headers + DOMPurify (si besoin) |
| Hébergement    | Vercel / Cloudflare Pages             |
| Outils Dev     | ESLint (avec règles sécu), Prettier, Husky, Lint-staged |

---