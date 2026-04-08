# Quill — AI Content Generator

> SaaS fullstack de génération de contenu IA pour les équipes marketing et les PME.

---

## Aperçu

Quill permet de générer des articles de blog, emails marketing, posts LinkedIn et fiches produits en quelques secondes, à partir d'un simple brief. L'IA s'adapte au ton et au vocabulaire de votre marque.

<!-- SCREENSHOT DASHBOARD -->
<!-- Ajouter capture du dashboard principal ici -->

---

## Features

- Génération de contenu en streaming temps réel (Claude API)
- 5 types de contenu : article, email, LinkedIn, fiche produit, tweet
- Profil de marque personnalisé (ton, vocabulaire, exemples)
- Éditeur rich text avec export PDF
- Historique des générations
- Abonnements Starter / Pro via Stripe
- Auth Google & GitHub via NextAuth
- Emails transactionnels via Resend

---

## Stack technique

| Couche | Technologie |
|---|---|
| Framework | Next.js 14 (App Router) |
| Base de données | PostgreSQL + Prisma |
| Auth | NextAuth v5 |
| Éditeur | Tiptap |
| Export PDF | Puppeteer |
| Paiements | Stripe Billing |
| Emails | Resend + React Email |
| Hébergement | Vercel + Railway |
| IA | Claude API (Anthropic) |

---

## Installation

### Prérequis

- Node.js 20+
- PostgreSQL (local ou Railway)
- Comptes : Anthropic, Stripe, Resend

### Setup

```bash
# Clone
git clone https://github.com/TON_USERNAME/quill.git
cd quill

# Dépendances
npm install

# Variables d'environnement
cp .env.example .env
# Remplir les valeurs dans .env

# Base de données
npx prisma migrate dev --name init

# Lancer en dev
npm run dev
```

### Variables d'environnement

```env
DATABASE_URL=
NEXTAUTH_URL=
NEXTAUTH_SECRET=
ANTHROPIC_API_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
RESEND_API_KEY=
```

---

## Structure du projet

```
quill/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/
│   │   │   ├── dashboard/
│   │   │   ├── generate/
│   │   │   ├── history/
│   │   │   └── settings/
│   │   └── api/
│   │       ├── auth/
│   │       ├── generate/
│   │       ├── export/
│   │       └── webhooks/stripe/
│   ├── components/
│   │   ├── ui/
│   │   ├── editor/
│   │   ├── generate/
│   │   └── layout/
│   ├── lib/
│   │   ├── auth/
│   │   ├── claude/
│   │   ├── db/
│   │   ├── resend/
│   │   └── stripe/
│   └── types/
├── prisma/
│   └── schema.prisma
└── public/
```

---

## Déploiement

```bash
# Build
npm run build

# Migrations en production
npx prisma migrate deploy
```

Déploiement automatique sur Vercel à chaque push sur `main`.

---

## Roadmap

- [ ] Profil de marque avancé
- [ ] Multi-langues
- [ ] Collaboration en équipe
- [ ] Templates personnalisés
- [ ] API publique

---

## Licence

MIT
