# Quill — Documentation technique complète

---

## Concept

Quill est un SaaS fullstack qui permet à des équipes marketing, freelances et PME de générer du contenu professionnel en quelques secondes grâce à l'IA. L'utilisateur soumet un brief (sujet, secteur, ton), et Quill génère le contenu en streaming via l'API Claude d'Anthropic, directement dans un éditeur rich text intégré.

### Problème résolu

Créer du contenu marketing de qualité prend du temps et nécessite des compétences rédactionnelles. Quill compresse ce temps de 2h à 30 secondes, tout en respectant la voix de la marque.

### Cible

- Freelances marketing & copywriters
- PME sans équipe content dédiée
- Agences gérant plusieurs clients

---

## Architecture générale

```
Client (Next.js App Router)
    ↓ HTTP / SSE
API Routes (Next.js Route Handlers)
    ↓             ↓            ↓
Claude API    PostgreSQL    Stripe API
(Anthropic)   (Prisma ORM)  (Webhooks)
                              ↓
                           Resend
                         (Emails)
```

### Flux principal — Génération de contenu

1. L'utilisateur remplit le formulaire de brief
2. Le client envoie une requête POST à `/api/generate`
3. Le Route Handler vérifie l'auth (NextAuth) et le quota (Prisma)
4. Il ouvre un stream SSE vers le client
5. Il appelle l'API Claude avec le prompt construit
6. Les tokens arrivent en streaming et sont injectés dans Tiptap
7. À la fin du stream, la génération est sauvegardée en base
8. Le compteur d'usage est incrémenté

### Flux paiement — Abonnement Stripe

1. L'utilisateur clique "Passer en Pro"
2. Redirection vers Stripe Checkout
3. Stripe envoie un webhook à `/api/webhooks/stripe`
4. Le Route Handler met à jour le plan en base (Prisma)
5. Resend envoie un email de confirmation

---

## Modèle de données

### User
Compte utilisateur. Lié à un profil de marque, des générations et un abonnement.

### Account / Session / VerificationToken
Tables NextAuth standards pour la gestion des sessions OAuth.

### Subscription
Plan de l'utilisateur (Starter / Pro), quota de générations, IDs Stripe.

### Generation
Chaque contenu généré : type, brief, output, tokens consommés, date.

### BrandProfile
Profil de marque de l'utilisateur : ton, vocabulaire autorisé, mots interdits, exemples.

---

## Features détaillées

### Génération de contenu

**Types supportés**
- Article de blog (800–1500 mots)
- Email marketing (200–400 mots)
- Post LinkedIn (150–300 mots)
- Fiche produit (300–600 mots)
- Tweet (280 caractères max)

**Paramètres du brief**
- Type de contenu
- Secteur d'activité
- Sujet / angle / mots-clés
- Ton (professionnel, conversationnel, inspirant, pédagogique, percutant)

**Streaming**
La réponse Claude est streamée token par token via SSE. Tiptap reçoit les tokens et les insère dans l'éditeur en temps réel — effet machine à écrire.

### Profil de marque

L'utilisateur configure une fois :
- Nom de l'entreprise
- Ton dominant
- Vocabulaire à privilégier
- Mots et expressions à éviter
- Exemples de contenus existants

Ces données sont injectées dans le system prompt envoyé à Claude.

### Éditeur rich text (Tiptap)

- Formatage : gras, italique, titres H1-H3, listes
- Injection streaming token par token
- Actions rapides : copier, régénérer, raccourcir, allonger
- Export PDF via Puppeteer

### Export PDF

Un Route Handler Next.js lance Puppeteer côté serveur, charge le HTML formaté du contenu, applique une CSS print propre et retourne un PDF binaire. Utilise `@sparticuz/chromium-min` pour compatibilité serverless.

### Abonnements

| Plan | Générations/mois | Prix |
|---|---|---|
| Starter | 50 | Gratuit |
| Pro | Illimité | 29€/mois |

Gestion complète via Stripe Billing : checkout, customer portal, webhooks, dunning.

### Emails transactionnels (Resend)

- Bienvenue à l'inscription
- Confirmation d'abonnement Pro
- Alerte quota (80% et 100% atteints)
- Reçu de paiement

Templates écrits en JSX avec React Email, prévisualisables en local.

---

## Sécurité

- Auth via NextAuth (JWT + sessions Prisma)
- Routes API protégées par middleware
- Vérification du quota avant chaque génération
- Validation des webhooks Stripe (signature HMAC)
- Variables sensibles uniquement côté serveur
- Rate limiting sur `/api/generate`

---

## Performance

- Server Components Next.js pour les pages statiques
- Streaming SSE pour la génération (pas de timeout)
- Prisma query optimization avec `select` explicites
- Images optimisées via `next/image`
- Edge-ready middleware pour l'auth

---

## Outils de développement

| Outil | Usage |
|---|---|
| Cursor | IDE IA-assisted |
| Prisma Studio | GUI base de données |
| Stripe CLI | Test des webhooks en local |
| React Email | Preview des templates email |
| PostHog | Analytics produit |
| Linear | Gestion des tickets |

---

## Déploiement & infra

### Environnements

| Env | URL | Branch |
|---|---|---|
| Production | quill.vercel.app | main |
| Preview | quill-git-*.vercel.app | PR/branches |
| Local | localhost:3000 | — |

### CI/CD

Push sur `main` → build Vercel automatique → migrations Prisma → déploiement.

### Base de données

PostgreSQL hébergée sur Railway. Backups automatiques quotidiens.

---

## Roadmap

### v1.0 — MVP
- [x] Auth Google/GitHub
- [x] Génération streaming
- [x] Éditeur Tiptap
- [x] Export PDF
- [x] Abonnements Stripe
- [x] Emails Resend

### v1.1
- [ ] Profil de marque avancé
- [ ] Historique et réutilisation
- [ ] Raccourcir / allonger le contenu

### v2.0
- [ ] Multi-langues
- [ ] Collaboration équipe
- [ ] Templates personnalisés
- [ ] API publique documentée
