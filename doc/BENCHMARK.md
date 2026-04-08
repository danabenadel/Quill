# Benchmark stack — Quill

---

## Framework frontend

### Next.js 14 ✅ Choix recommandé

| Critère | Score |
|---|---|
| DX | 9.5 / 10 |
| Performance | 9 / 10 |
| Streaming SSE | 10 / 10 |
| Écosystème | 10 / 10 |

App Router + Server Components natifs. Les Route Handlers gèrent le streaming SSE vers Claude sans config. Déploiement Vercel en 1 clic. Référence absolue pour les SaaS en 2025.

---

### Nuxt 3 — Alternative

| Critère | Score |
|---|---|
| DX | 8.8 / 10 |
| Performance | 8.5 / 10 |
| Streaming SSE | 8 / 10 |
| Écosystème | 7.5 / 10 |

Excellent si tu préfères Vue. Nitro server intègre bien le streaming. Moins de ressources communautaires que Next sur les patterns IA. Viable mais pas optimal pour ce projet.

---

### Remix — Alternative

| Critère | Score |
|---|---|
| DX | 7.8 / 10 |
| Performance | 8.8 / 10 |
| Streaming SSE | 7.5 / 10 |
| Écosystème | 6.5 / 10 |

Très bon pour les apps data-heavy avec loaders. Moins adapté au streaming IA, et l'écosystème est plus restreint. Niche mais solide.

---

## Base de données

### PostgreSQL + Prisma ✅ Choix recommandé

| Critère | Score |
|---|---|
| Fiabilité | 10 / 10 |
| DX Prisma | 9.5 / 10 |
| Scalabilité | 9 / 10 |
| Coût de départ | 8.5 / 10 |

Référence pour les SaaS. Prisma ORM offre la type-safety, les migrations auto et une DX excellente. Hébergeable sur Supabase (gratuit jusqu'à 500 MB) ou Railway. Le choix évident.

---

### PlanetScale + Drizzle — Alternative

| Critère | Score |
|---|---|
| Fiabilité | 9 / 10 |
| DX Drizzle | 8.5 / 10 |
| Scalabilité | 9.5 / 10 |
| Coût de départ | 7 / 10 |

MySQL serverless très scalable. Drizzle est plus léger que Prisma. Mais PlanetScale a supprimé son free tier en 2024 — coût dès le départ. Moins adapté pour un MVP.

---

### MongoDB — Déconseillé

| Critère | Score |
|---|---|
| Fiabilité | 8 / 10 |
| DX | 7.5 / 10 |
| Scalabilité | 8.5 / 10 |
| Coût de départ | 9 / 10 |

Pas adapté : les données du projet (users, plans, générations) sont naturellement relationnelles. MongoDB apporte de la complexité sans bénéfice ici.

---

## Authentification

### NextAuth v5 ✅ Choix recommandé

| Critère | Score |
|---|---|
| Intégration Next | 10 / 10 |
| Facilité de setup | 9 / 10 |
| Providers OAuth | 10 / 10 |
| Coût | Gratuit |

Natif Next.js, zéro friction. Google/GitHub OAuth en 15 min. Gère les sessions JWT + Prisma adapter pour persister en base. La référence pour ce projet.

---

### Clerk — Alternative premium

| Critère | Score |
|---|---|
| Intégration Next | 9.5 / 10 |
| Facilité de setup | 10 / 10 |
| Providers OAuth | 10 / 10 |
| Coût | Payant (~$25/mois après 10k MAU) |

DX exceptionnelle, UI auth pré-faite, MFA inclus. Gratuit jusqu'à 10 000 MAU puis ~$25/mois. Excellent pour aller vite, mais crée une dépendance vendor et coûte plus.

---

### Supabase Auth — Alternative

| Critère | Score |
|---|---|
| Intégration Next | 8.2 / 10 |
| Facilité de setup | 8.5 / 10 |
| Providers OAuth | 9 / 10 |
| Coût | Gratuit |

Pertinent si tu utilises Supabase comme BDD aussi. Sinon, ajouter Supabase juste pour l'auth quand tu as déjà Prisma/Postgres est redondant.

---

## Éditeur rich text

### Tiptap ✅ Choix recommandé

| Critère | Score |
|---|---|
| Extensibilité | 10 / 10 |
| Compatibilité React | 10 / 10 |
| Streaming ready | 9.5 / 10 |
| Poids bundle | 8 / 10 |

Basé sur ProseMirror, headless par défaut (tu stylises toi-même). Parfait pour injecter du texte streamé token par token. Extensions markdown, mentions, collaboration disponibles.

---

### Quill — Alternative

| Critère | Score |
|---|---|
| Extensibilité | 7 / 10 |
| Compatibilité React | 7.2 / 10 |
| Streaming ready | 6.5 / 10 |
| Poids bundle | 8.5 / 10 |

Plus simple à bootstrapper mais moins flexible. Peu maintenu depuis 2020. L'intégration React via react-quill est fragile. À éviter en 2025.

---

### Lexical (Meta) — Alternative avancée

| Critère | Score |
|---|---|
| Extensibilité | 10 / 10 |
| Compatibilité React | 10 / 10 |
| Streaming ready | 9 / 10 |
| Poids bundle | 9 / 10 |

Puissant et moderne (créé par Meta). Courbe d'apprentissage plus raide que Tiptap. À considérer si tu veux un éditeur très custom, sinon Tiptap est plus rapide à mettre en place.

---

## Export PDF

### Puppeteer (serveur) ✅ Choix recommandé

| Critère | Score |
|---|---|
| Fidélité de rendu | 10 / 10 |
| Mise en page | 10 / 10 |
| Complexité de setup | 7 / 10 |
| Performance | 8 / 10 |

Rendu HTML → PDF parfait pixel. Tu stylises avec du CSS normal, pas une API propriétaire. Fonctionne en Route Handler Next.js. Le plus pro des rendus.

> **Note infra :** Les Edge Functions Vercel ne supportent pas Puppeteer. Utilise `@sparticuz/chromium-min` pour les environnements serverless, ou héberge sur Railway.

---

### react-pdf — Alternative légère

| Critère | Score |
|---|---|
| Fidélité de rendu | 7.2 / 10 |
| Mise en page | 7 / 10 |
| Complexité de setup | 9.5 / 10 |
| Performance | 9 / 10 |

Génère le PDF côté client sans serveur. Plus simple à démarrer mais la mise en page se fait via une API JSX spécifique (pas du CSS standard). Rendu moins fidèle au rich text.

---

### jsPDF — Déconseillé

| Critère | Score |
|---|---|
| Fidélité de rendu | 5 / 10 |
| Mise en page | 4.5 / 10 |
| Complexité de setup | 9 / 10 |
| Performance | 8.5 / 10 |

Génère du PDF programmatique ligne par ligne. Aucune notion de HTML/CSS — la mise en page d'un document riche devient un enfer. À éviter absolument pour ce projet.

---

## Paiements & abonnements

### Stripe Billing ✅ Choix recommandé

| Critère | Score |
|---|---|
| Fiabilité | 10 / 10 |
| DX SDK | 9.5 / 10 |
| Webhooks | 10 / 10 |
| Frais | 1.4% + €0.25 / transaction |

Standard de l'industrie. Customer Portal, gestion des plans, dunning management (relance impayés), factures automatiques. L'intégration avec Next.js est documentée partout.

---

### Lemon Squeezy — Alternative simplifiée

| Critère | Score |
|---|---|
| Fiabilité | 8.2 / 10 |
| DX SDK | 8.5 / 10 |
| Webhooks | 8 / 10 |
| Frais | 5% + $0.50 / transaction |

Merchant of record : ils gèrent la TVA mondiale à ta place. Moins de config fiscale mais frais plus élevés. Bon choix si tu vises un marché international dès le départ.

---

## Emails transactionnels

### Resend + React Email ✅ Choix recommandé

| Critère | Score |
|---|---|
| DX | 10 / 10 |
| Délivrabilité | 9.2 / 10 |
| Free tier | 3 000 emails / mois |
| Templates React | 10 / 10 |

API first, SDK Node natif, templates écrits en JSX avec React Email. Prévisualisation dans le browser. 3 000 emails/mois gratuits. Le duo parfait pour ce projet Next.js.

---

### SendGrid — Alternative

| Critère | Score |
|---|---|
| DX | 6.5 / 10 |
| Délivrabilité | 9 / 10 |
| Free tier | 100 emails / jour |
| Templates React | Non |

Très fiable mais interface vieillissante et DX datée. Les templates sont en HTML brut. Pertinent si tu connais déjà l'outil, sinon Resend est objectivement meilleur aujourd'hui.

---

## Hébergement & infra

### Vercel + Railway ✅ Choix recommandé

| Critère | Score |
|---|---|
| DX déploiement | 10 / 10 |
| Performance CDN | 9.5 / 10 |
| Free tier | 8.5 / 10 |
| Scalabilité | 9.2 / 10 |

Vercel pour Next.js (preview deploys, Edge Network, streaming natif). Railway pour PostgreSQL (~$5/mois). Le duo le plus rapide à mettre en prod. Zéro DevOps.

---

### Fly.io — Alternative

| Critère | Score |
|---|---|
| DX déploiement | 7.8 / 10 |
| Performance CDN | 8.8 / 10 |
| Free tier | 8 / 10 |
| Scalabilité | 9 / 10 |

Full-stack en containers Docker, Postgres inclus. Plus de contrôle que Vercel mais plus de config. Pertinent si tu veux héberger Puppeteer (lourd pour les edge functions Vercel).

---

### AWS / GCP — Overkill pour un MVP

| Critère | Score |
|---|---|
| DX déploiement | 4 / 10 |
| Performance CDN | 9.5 / 10 |
| Free tier | 6 / 10 |
| Scalabilité | 10 / 10 |

Puissance maximale mais complexité DevOps considérable. Inutile pour un portfolio project ou un MVP SaaS. Tu passes plus de temps sur l'infra que sur le produit.

---

## Résumé de décision — Stack finale

| Couche | Choix retenu | Raison principale |
|---|---|---|
| Framework | Next.js 14 (App Router) | Streaming SSE natif, tout-en-un |
| Base de données | PostgreSQL + Prisma | Type-safe, migrations auto, Supabase gratuit |
| Auth | NextAuth v5 | Natif Next, OAuth Google en 15 min |
| Éditeur rich text | Tiptap | Headless, injection streaming parfaite |
| Export PDF | Puppeteer + @sparticuz/chromium-min | Rendu HTML/CSS fidèle au pixel |
| Paiements | Stripe Billing | Standard industrie, webhooks robustes |
| Emails | Resend + React Email | DX top, templates JSX, 3k/mois gratuit |
| Hébergement | Vercel + Railway | Zéro DevOps, preview deploys |
