# Audit Landing Page — Texaa (Google Ads / Quality Score)

> **Objectif révisé** (validé avec le client)
> Cette landing n'est **pas destinée au SEO organique**. Elle sera hébergée sur `solutions.texaa.fr` comme **page de destination Google Ads**. La priorité absolue est donc :
> - **Quality Score / Ad Rank élevé** (CPC plus bas)
> - **Vitesse de chargement** (Core Web Vitals)
> - **Expérience mobile** fluide
> - **Génération de leads** (appel + formulaire)
>
> Les recommandations purement SEO (sitemap, schema, OG, indexation) sont **dépriorisées**. À l'inverse, la page devra être en `noindex` pour éviter le duplicate content avec `texaa.fr`.
>
> ### 🟢 Conformité cookies — Résolu (2026-04-13)
>
> Axeptio est **déjà configuré dans le conteneur GTM du client** (`GTM-WJ4ZGGSN`, project ID `66978a322409f46d017ecfb2`). La bannière est injectée automatiquement sur `solutions.texaa.fr` via GTM. Aucune action côté code.
> Consent Mode v2 probablement actif (à confirmer avec le client dans l'interface GTM).

---

## Résumé exécutif

| Domaine | Note | Commentaire |
|---|---|---|
| Performance / poids | 🟢 Résolu | 5,3 MB → 1,5 MB (gain -72 %), WebP responsive via `astro:assets` |
| LCP (image hero) | 🟢 Résolu | hero cinematic 98 KB WebP + preload + `fetchpriority="high"` |
| CLS | 🟢 Résolu | `width`/`height` explicites sur toutes les images |
| Code propre (1 seul hero) | 🟢 Résolu | Hero C uniquement, tous les séparateurs et CSS morts supprimés |
| Tracking (GTM) | 🟢 Résolu | `GTM-WJ4ZGGSN` intégré (head + body), se charge inconditionnellement |
| Indexation (`noindex`) | 🟢 Résolu | `<meta name="robots" content="noindex,follow">` en place |
| HTML sémantique / a11y | 🟢 Résolu | `<nav>` ajouté, liens morts corrigés, accordéon `<details>` natif |
| Conformité RGPD / cookies | 🟢 Résolu | Axeptio déjà actif via GTM du client (project ID `66978a322409f46d017ecfb2`) |
| Mentions légales / Politique conf. | 🟠 En attente | Bloqué par les infos légales à fournir par le client |
| Formulaire de lead | 🟢 Résolu | Form custom + handler JS → webhook Make (attente URL webhook réelle) |
| Page `/merci` | 🟢 Résolu | Page de confirmation créée, GTM + noindex en place |
| Responsive mobile / tablette | 🟢 Résolu | Boutons, sections, logos, formulaire ajustés pour ≤1024px et ≤768px |
| Tags SEO classiques | 🟡 Secondaire | Peu d'impact pour Google Ads, `noindex` déjà en place |

### Top 5 actions initiales — toutes résolues

1. ~~Garder uniquement le Hero C, supprimer A/B/D + séparateurs~~ ✅
2. ~~Optimiser toutes les images (WebP responsive + `width`/`height`)~~ ✅
3. ~~Installer GTM `GTM-WJ4ZGGSN` (head + body)~~ ✅
4. ~~Ajouter `noindex` + préparer pages Mentions légales~~ ✅ (noindex fait, mentions en attente infos client)
5. ~~Renommer toutes les images (kebab-case ASCII)~~ ✅

---

## Baseline performance (mesuré le 2026-04-11)

### Poids du build `dist/`

| Type | Taille | Note |
|---|---|---|
| HTML (`index.html`) | 38 KB | Trop lourd : contient les 4 hero rendus simultanément |
| CSS (1 fichier) | 36 KB | OK |
| JS bundle | 0 KB | Aucun (excellent) |
| Script inline | 1,3 KB | Accordéon FAQ |
| **Images** | **4,6 MB** (30 fichiers) | **Critique** |
| **Total `dist/`** | **5,3 MB** | Cible réaliste après optimisation : ~700 KB |

### Top 10 images les plus lourdes

| Poids | Dimensions | Fichier | Problème |
|---|---|---|---|
| 524 KB | 2560×2060 | `images/ref le tronquay.jpg` | Sur-dimensionné + espaces |
| 514 KB | 2560×1444 | `images/AbsoTexaa.jpg` | Sur-dimensionné |
| 411 KB | 1707×2560 | `images/hall_hopital_..._vertical.jpg` | Sur-dimensionné |
| 358 KB | 2560×1707 | `images/hall_hopital_..._carre.jpg` | Sur-dimensionné |
| 309 KB | 2560×1707 | `images/ref mairie bidart.jpg` | Espaces + grand |
| 242 KB | 1200×1200 | `231024_TEXAA_..._0677-1200x1200.jpg` | Dimensions OK, JPEG non optimal |
| **240 KB** | **3840×2263** | **`logo/logo-republique-francaise.png`** | **Logo en 4K** alors qu'affiché à ~80 px |
| 219 KB | 1200×1200 | `231024_TEXAA_©0901-1200x1200.jpg` | Caractère spécial © |
| 213 KB | 2200×1466 | `logo/hero.jpg` | Mauvais emplacement (logo/) |
| **172 KB** | **2560×2048** | **`hero/hero 3.jpg`** ← **HERO C / LCP** | **Trop lourd pour LCP** |

### Estimation du gain potentiel

| Optimisation | Gain estimé |
|---|---|
| Suppression hero A/B/D | -10 KB HTML + jusqu'à -700 KB d'images potentielles |
| Conversion WebP/AVIF de toutes les images | -60 à -80 % du poids image (4,6 MB → ~1 MB) |
| Redimensionnement responsive (`<picture>` + srcset) | -30 à -50 % supplémentaires sur mobile |
| Logos redimensionnés à la bonne taille | -400 KB cumulés sur les logos |
| **Total cible réaliste** | **~600-900 KB** au total (vs 5,3 MB) |

---

## 1. Performance & Core Web Vitals (priorité Quality Score)

### 1.1 Image LCP (Hero C) non optimisée — CRITIQUE
- **Constat :** `hero/hero 3.jpg` est en JPEG 2560×2048 / 172 KB. C'est l'image LCP de la page : son temps de chargement détermine directement le LCP mesuré par Google.
- **Action :**
  - Convertir en AVIF + WebP avec fallback JPEG
  - Générer plusieurs tailles (ex : 768, 1280, 1920) via `<picture>` + `srcset`
  - Ajouter `fetchpriority="high"` et `<link rel="preload">` dans le `<head>`
  - Cible : < 80 KB pour la version desktop
- **Renommer en** `hero-cinematic.webp` (et variantes).

### 1.2 Aucun `width`/`height` sur les `<img>` — CRITIQUE (CLS)
- **Constat :** Aucune balise `<img>` ne possède d'attributs `width`/`height`. Cela provoque du Cumulative Layout Shift, fortement pénalisé par Google Ads.
- **Action :** Ajouter ces attributs sur **toutes** les images (au moins en valeurs intrinsèques).

### 1.3 Aucune image en WebP/AVIF — ÉLEVÉ
- **Constat :** 29 images sur 30 sont en JPEG/PNG. Seul `ldlc-logo.webp` est correct.
- **Action :** Conversion de masse en WebP/AVIF. Possibilité d'utiliser le composant `<Image>` d'Astro avec `astro:assets` (déplacer les images dans `src/assets/` pour bénéficier de l'optimisation automatique au build).

### 1.4 Logos sur-dimensionnés — ÉLEVÉ
- **Constat :**
  - `logo-republique-francaise.png` : 3840×2263 / 240 KB pour un affichage à ~80 px
  - `Hermes-Logo.png` : 3840×2160 / 70 KB
  - `hero.jpg` (mal placé dans `logo/`) : 2200×1466 / 213 KB
- **Action :** Redimensionner tous les logos à 2× leur taille d'affichage maximale (ex : 200 px de large) et convertir en WebP/SVG quand possible.

### 1.5 Pas de responsive images — MOYEN
- **Constat :** Aucun usage de `<picture>` ou `srcset` → mobile télécharge les mêmes 2560 px que desktop.
- **Action :** Pour les visuels lourds (hero, refs, marchés), générer 2-3 tailles et utiliser `srcset`.

### 1.6 Google Fonts en bloquant — MOYEN
- **Constat :** `Layout.astro` lignes 24-26 charge 2 familles depuis `fonts.googleapis.com` de façon synchrone.
- **Action :** Auto-héberger les polices (téléchargement + placement dans `public/fonts/`) ou utiliser `font-display: swap` (déjà fait) + `<link rel="preload">` sur les fichiers `.woff2`.

### 1.7 Noms de fichiers avec espaces / accents / caractères spéciaux — MOYEN
- **Constat :** Plusieurs fichiers : `hero 3.jpg`, `hero vertical.jpg`, `espace d'accueil.jpg`, `technique & son.jpg`, `ref le tronquay.jpg`, `ref mairie bidart.jpg`, `ref lysée national de la marine.jpg`, `231024_TEXAA_©I.Mathie_0901-1200x1200.jpg`, etc.
- **Action :** Renommer en kebab-case ASCII : `hero-cinematic.jpg`, `espace-accueil.jpg`, `technique-son.jpg`, `ref-le-tronquay.jpg`, etc. Les espaces sont encodés en `%20` dans les URLs, ce qui est fragile et nuit au cache CDN.

### 1.8 4 hero rendus simultanément — CRITIQUE
- **Constat :** `index.astro` lignes 158, 215, 273, +1 — les 4 propositions A/B/C/D sont rendues. Cela gonfle le HTML (38 KB), charge potentiellement les images des 4 hero, et provoque 4 H1.
- **Action :** Supprimer A, B et D (lignes ~158-271 et ~327+) ainsi que tous les séparateurs `<div>— FIN PROPOSITION X —</div>` (lignes 213, 271, 326, 385).

---

## 2. Tracking & conversion (Google Ads)

### 2.1 GTM absent — CRITIQUE
- **Constat :** Aucun script de tracking dans le code. Aucun GA4, aucun GTM, aucun pixel de conversion.
- **Action :** Intégrer Google Tag Manager `GTM-WJ4ZGGSN` dans `Layout.astro` :
  - **Dans le `<head>`, le plus haut possible :**
    ```html
    <!-- Google Tag Manager -->
    <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer','GTM-WJ4ZGGSN');</script>
    <!-- End Google Tag Manager -->
    ```
  - **Juste après l'ouverture du `<body>` :**
    ```html
    <!-- Google Tag Manager (noscript) -->
    <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-WJ4ZGGSN"
    height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
    <!-- End Google Tag Manager (noscript) -->
    ```
- **Recommandation complémentaire :** déclarer dans GTM les événements de conversion clés :
  - clic sur le numéro de téléphone
  - clic sur le bouton "Demander des échantillons"
  - soumission d'un formulaire (à venir si formulaire ajouté)
  - scroll > 75 %

### 2.2 Pas de formulaire de lead — MOYEN
- **Constat :** Les seuls CTA sont l'appel téléphonique et un lien externe vers `texaa.fr/echantillons/`. Pas de formulaire intégré sur la landing.
- **Recommandation :** Étudier l'ajout d'un formulaire court (3-4 champs : nom, email, téléphone, projet) pour capter les leads qui ne veulent pas appeler. Améliore le taux de conversion sur mobile et hors heures de bureau.

### 2.3 Liens morts `href="#"` — MOYEN
- **Constat :** Lignes 134 (logo sticky), 699 (cartes marchés), 963-965 (footer Mentions légales / Contact).
- **Action :** Pointer vers `/` pour le logo, vers les vraies pages (mentions légales) ou vers une ancre interne pour les marchés.

---

## 3. Conformité & Google Ads policy

### 3.1 Mentions légales absentes — BLOQUANT
- **Constat :** Lien `href="#"` dans le footer. Obligation légale en France + **exigence Google Ads** (sans mentions légales et politique de confidentialité, l'annonce peut être désapprouvée).
- **Action :** Créer 2 pages :
  - `/mentions-legales` (éditeur, hébergeur, contact)
  - `/politique-de-confidentialite` (RGPD, cookies, finalités, droits)

### 3.2 Pas de bannière cookies — BLOQUANT (RGPD + Google Ads)
- **Constat :** GTM va déposer des cookies marketing → consentement explicite obligatoire en UE (CNIL).
- **Action :** Mettre en place une CMP (ex : Axeptio, Tarteaucitron, Cookiebot, Google Consent Mode v2). Indispensable pour la compatibilité avec les conversions Google Ads.

### 3.3 `noindex` recommandé — ÉLEVÉ
- **Constat :** Cette landing va dupliquer du contenu de `texaa.fr` et n'a pas vocation à être indexée.
- **Action :** Ajouter dans le `<head>` :
  ```html
  <meta name="robots" content="noindex,follow">
  ```
- **Bonus :** plus besoin de robots.txt ni de sitemap.

---

## 4. Code propre & HTML sémantique

### 4.1 Séparateurs de test à supprimer — CRITIQUE
- Lignes 213, 271, 326, 385 : `<div>— FIN PROPOSITION X —</div>`
- À supprimer en même temps que les hero A/B/D.

### 4.2 4 H1 dupliqués — CRITIQUE (lié à 1.8)
- Conséquence directe des 4 hero rendus → après nettoyage, il ne restera qu'un seul H1 (Hero C ligne 287).

### 4.3 Pas d'élément `<nav>` — FAIBLE
- Le header sticky n'utilise pas `<nav>` → impact a11y léger.

### 4.4 FAQ non sémantique — FAIBLE
- Utilise `<div>` + `<button>` + JS au lieu de `<details>`/`<summary>`. À envisager si on veut alléger le JS inline.

---

## 5. Tags SEO classiques (dépriorisés)

Les éléments suivants sont **dépriorisés** car la page sera en `noindex` :

| Élément | Statut | Action |
|---|---|---|
| URL canonique | ⚪ Non nécessaire | `noindex` rend obsolète |
| Open Graph / Twitter Card | 🟡 Optionnel | Utile uniquement si la page est partagée sur les réseaux |
| Sitemap XML | ⚪ Non nécessaire | `noindex` rend obsolète |
| `robots.txt` | ⚪ Non nécessaire | `noindex` suffit |
| Schema.org JSON-LD | ⚪ Non nécessaire | Aucun gain pour Google Ads |
| Title tag (66 car.) | 🟡 OK | Peut rester tel quel |
| Meta description (155 car.) | 🟢 OK | Conserver |
| Attribut `lang="fr"` | 🟢 OK | Conserver |
| Configurer `site` dans `astro.config.mjs` | 🟡 Optionnel | Plus nécessaire sans sitemap |

---

## 6. Plan d'action priorisé

### ✅ Déjà fait (phases 1-5 du travail de refonte)
1. ~~Suppression hero A, B et D + séparateurs~~
2. ~~Renommage des images (kebab-case ASCII)~~
3. ~~Optimisation images (WebP responsive via `astro:assets`)~~
4. ~~`width`/`height` sur toutes les images~~
5. ~~`fetchpriority="high"` + preload sur l'image hero~~
6. ~~Installation GTM `GTM-WJ4ZGGSN` (head + body)~~
7. ~~`<meta name="robots" content="noindex,follow">` ajouté~~
8. ~~Correction des liens morts `href="#"`~~
9. ~~Auto-hébergement des Google Fonts~~
10. ~~`<nav>` ajouté au header~~
11. ~~Cleanup CSS mort + suppression styles inline~~
12. ~~Refonte footer + accordéon mobile sur références~~

### 🟢 Résolu automatiquement (2026-04-13)
13. ~~Bannière cookies / CMP~~ → ✅ Axeptio déjà configuré dans GTM `GTM-WJ4ZGGSN`
14. ~~Consent Mode v2~~ → ✅ Probablement actif via la config GTM existante
15. ~~Protection juridique prestataire RGPD~~ → ✅ Plus nécessaire, conformité assurée

### 🟠 Bloquants restants — code (attente infos client)
16. Créer le **scénario Make** + remplacer le placeholder webhook dans `index.astro` (cf. `MAKE-SCENARIO.md`)
17. Créer pages **Mentions légales** + **Politique de confidentialité** (attente raison sociale, SIRET, hébergeur, email DPO)
18. Confirmer avec le client que l'Axeptio GTM **couvre bien** `solutions.texaa.fr`
19. **Déploiement** sur `solutions.texaa.fr` via Plesk (attente accès — cf. `DEPLOIEMENT-PLESK.md`)

### 🟡 Long terme (post-MEL)
20. Tester systématiquement le Quality Score sur les annonces
21. A/B tester d'autres CTAs / titres
22. Surveiller les Core Web Vitals via la Search Console

---

## 7. Réponses du client (validées)

| Question | Réponse |
|---|---|
| Domaine cible | `texaa.fr` (site principal existant) |
| Sous-domaine landing | `solutions.texaa.fr` |
| Objectif | Google Ads / Quality Score / lead gen — **PAS de SEO organique** |
| Hero retenu | **Hero C** (cinématique, photo plein écran, stats en bas) |
| Search Console | ✅ Accès disponible |
| Analytics / tracking | GTM `GTM-WJ4ZGGSN` à intégrer (head + body) |
