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

## Charte graphique

### Identité visuelle

Quill adopte une esthétique **premium et éditoriale** — sérieux sans être austère, moderne sans être générique. Le design system s'articule autour d'un contraste fort entre un bleu pétrole ancré (`#3A6B91`), un gold chaleureux (`#D4AF37`) et un fond navy profond (`#1C2A39`). L'ensemble évoque l'univers de l'écriture et de la création, avec une touche craft.

---

### Palette de couleurs

| Nom | Hex | Rôle |
|---|---|---|
| Quill Blue | `#3A6B91` | Couleur primaire — CTAs, liens actifs, focus states, icônes d'action |
| Parchment | `#F1E9DA` | Surface chaude — texte secondaire sur fond sombre, éléments neutres |
| Off-white | `#F8F9FA` | Fond clair — pages en mode light, zones de contenu |
| Ink Gold | `#D4AF37` | Accent — badges Pro, highlights, éléments premium, notifications |
| Pure White | `#FFFFFF` | Texte principal sur fond sombre, fond de cartes en mode light |
| Deep Navy | `#1C2A39` | Fond sombre — background principal de l'app, sidebar, overlays |

**Règles d'utilisation :**
- Le bleu domine à 60-70% du poids visuel dans les composants actifs
- Le gold est réservé aux éléments premium et aux accents — ne pas surcharger
- Ne jamais utiliser le gold comme fond de texte long
- Le navy est le fond par défaut — éviter les fonds blancs dans les vues dashboard

**Variables CSS :**
```css
:root {
  --color-blue:      #3A6B91;   /* Primaire */
  --color-cream:     #F1E9DA;   /* Surface chaude */
  --color-offwhite:  #F8F9FA;   /* Fond clair */
  --color-gold:      #D4AF37;   /* Accent */
  --color-white:     #FFFFFF;   /* Texte / fond */
  --color-dark:      #1C2A39;   /* Fond sombre */
}
```

---

### Typographie

Quill utilise deux polices complémentaires qui créent un contraste fort entre l'éditorial et le fonctionnel.

#### Bodoni Moda — Police display (titres)

```
Famille   : Bodoni Moda
Catégorie : Serif
Graisses  : 600 (Semi-Bold), 700 (Bold)
Source    : Google Fonts
Usage     : H1, H2, H3, logo wordmark, titres de section
```

Bodoni Moda apporte l'identité éditoriale et le prestige visuel de Quill. Ses empattements contrastés et sa géométrie classique évoquent la presse et l'édition haut de gamme.

#### Geist Sans — Police texte (interface)

```
Famille   : Geist
Catégorie : Sans-Serif
Graisses  : 300 (Light), 400 (Regular), 500 (Medium), 600 (Semi-Bold)
Source    : Vercel / npm (geist)
Usage     : UI, body text, labels, éditeur, formulaires
```

Geist assure la lisibilité maximale dans l'interface. Sa neutralité et sa lisibilité à petite taille en font le choix idéal pour les composants fonctionnels.

#### Échelle typographique

| Niveau | Taille | Police | Graisse | Usage |
|---|---|---|---|---|
| Display | 72px | Bodoni Moda | 700 | Titre hero, landing page |
| H1 | 40px | Bodoni Moda | 700 | Titres de page principaux |
| H2 | 28px | Bodoni Moda | 600 | Titres de section |
| H3 | 22px | Bodoni Moda | 600 | Sous-sections, titres de cards |
| Body | 16px | Geist | 400 | Texte courant, descriptions |
| Label | 12px | Geist | 500 | Labels de formulaires, captions |
| Eyebrow | 10px | Geist | 500 | Labels de section en uppercase + letter-spacing |

**Règles :**
- Bodoni Moda uniquement pour les titres — jamais pour le body text ou les labels
- Letter-spacing `0.22em` sur les eyebrows en uppercase
- Line-height `1.15` pour les titres display, `1.6` pour le body

---

### Logo

#### Icône

L'icône Quill représente un **nuage surmonté d'un laptop**, avec une **courbe de Bézier** affichée sur l'écran — symbolisant la génération IA de contenu (cloud computing + création graphique/éditoriale). Les nœuds de la courbe sont dessinés en gold (`#D4AF37`) sur fond parchment (`#F1E9DA`).

#### Wordmark

`QUILL` en Bodoni Moda Bold avec letter-spacing de `0.08em`. Toujours en majuscules.

#### Déclinaisons

| Version | Fond | Couleur logo | Usage |
|---|---|---|---|
| Principale | Deep Navy `#1C2A39` | Parchment + Gold | App, dark mode, présentations |
| Claire | Off-white `#F8F9FA` | Navy + Gold | Documents, exports PDF, fond blanc |
| Icône seule | Quill Blue `#3A6B91` | Parchment | Favicon, app icon, avatar |

#### Règles logo

**À faire :**
- Utiliser sur fond uni uniquement (navy, white, blue)
- Respecter un espace minimal autour du logo équivalent à la hauteur de la lettre "Q"
- Conserver les proportions originales à tout moment

**À éviter :**
- Déformer, étirer ou compresser le logo
- Modifier les couleurs sans validation du design system
- Placer le logo sur fond photographique ou texturé
- Utiliser l'icône seule sans le wordmark dans les contextes éditoriaux

---

### Composants UI

#### Boutons

| Variante | Fond | Texte | Usage |
|---|---|---|---|
| Primary | `#3A6B91` | `#FFFFFF` | Action principale (Générer, Enregistrer) |
| Gold | `#D4AF37` | `#1C2A39` | Upgrade Pro, actions premium |
| Outline | Transparent | `#FFFFFF` | Actions secondaires |
| Ghost | `rgba(255,255,255,0.06)` | `#F1E9DA` | Actions tertiaires (Copier, Raccourcir) |

Border-radius : `8px` (sm), `10px` (md), `8px` (lg CTA).

#### Badges & Pills

| Type | Fond | Texte | Bordure | Usage |
|---|---|---|---|---|
| Pro | `rgba(212,175,55,.15)` | `#D4AF37` | `rgba(212,175,55,.25)` | Plan payant |
| Starter | `rgba(58,107,145,.2)` | `#7ab3d4` | `rgba(58,107,145,.3)` | Plan gratuit |
| Nouveau | `rgba(241,233,218,.12)` | `#F1E9DA` | `rgba(241,233,218,.2)` | Nouveauté |
| Archivé | `rgba(28,42,57,.8)` | `#6a8a9a` | `rgba(255,255,255,.1)` | Contenu archivé |

Pills de ton : border-radius `20px`, bordure `0.5px`, fond transparent → fond blue teinté à l'état actif.

#### Champs de saisie

- Fond : `rgba(255,255,255,0.05)`
- Bordure repos : `rgba(255,255,255,0.1)`
- Bordure focus : `#3A6B91` + glow `rgba(58,107,145,0.15)`
- Border-radius : `8px`
- Hauteur : `44px` (standard), `36px` (compact)

#### Système d'espacement

Base `4px`. Tous les espacements sont des multiples de 4.

| Token | Valeur | Tailwind |
|---|---|---|
| xs | 4px | `p-1` / `gap-1` |
| sm | 8px | `p-2` / `gap-2` |
| md | 12px | `p-3` / `gap-3` |
| lg | 16px | `p-4` / `gap-4` |
| xl | 24px | `p-6` / `gap-6` |
| 2xl | 32px | `p-8` / `gap-8` |
| 3xl | 48px | `p-12` / `gap-12` |

#### Border-radius

```css
--radius-sm : 6px   /* Badges, pills compactes */
--radius-md : 10px  /* Boutons, inputs */
--radius-lg : 16px  /* Cards, modales */
--radius-xl : 24px  /* Containers, sections */
```

---

### Application du design system

#### Dans le code (Tailwind config)

```js
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      'quill-blue':  '#3A6B91',
      'quill-gold':  '#D4AF37',
      'quill-dark':  '#1C2A39',
      'quill-cream': '#F1E9DA',
    },
    fontFamily: {
      display: ['Bodoni Moda', 'serif'],
      sans:    ['Geist', 'sans-serif'],
    },
  }
}
```

#### Fichiers de référence

| Fichier | Description |
|---|---|
| `quill-charte-graphique.html` | Document HTML interactif complet |
| `quill-charte-graphique.pdf` | Export PDF pour partage et impression |
| `quill-charte-graphique.pptx` | Deck PowerPoint 8 slides |

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
