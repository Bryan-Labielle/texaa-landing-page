# Conformité cookies — Statut RGPD du projet

> ## 🟢 RÉSOLU — Axeptio déjà actif via GTM (découvert le 2026-04-13)
>
> **Le client a déjà Axeptio configuré dans son conteneur GTM `GTM-WJ4ZGGSN`.** La bannière cookies est automatiquement injectée sur toutes les pages qui chargent ce GTM — y compris `solutions.texaa.fr`.
>
> ### Ce que ça signifie
> - **Conformité RGPD** : ✅ gérée automatiquement, aucun travail côté prestataire
> - **Consent Mode v2** : ✅ probablement actif (à confirmer dans l'interface GTM)
> - **Performances Google Ads** : ✅ non dégradées (signaux de conversion préservés)
> - **Code Astro** : ✅ aucun script Axeptio dans `src/`, tout passe par GTM au runtime
>
> ### Infos techniques
> - **Project ID Axeptio** : `66978a322409f46d017ecfb2` (configuré dans GTM, vraisemblablement le même compte que `texaa.fr`)
> - **Injection** : GTM charge le SDK Axeptio (`static.axept.io/sdk.js`) qui injecte un `<div class="axeptio_mount">` au runtime. Ce div n'est PAS dans le HTML statique — il est créé dynamiquement par JS.
> - **Vérification** : `curl -s https://solutions.texaa.fr | grep axeptio` ne retourne rien (normal, c'est injecté côté client par GTM)
>
> ### Historique de cette section
> 1. **2026-04-11** : le client avait initialement dit « pas de bannière cookies ». Protection juridique préparée (§ 8).
> 2. **2026-04-13** : découverte que GTM injecte déjà Axeptio automatiquement. Le problème ne se pose plus.
> 3. La **protection juridique prestataire** (§ 8) reste dans ce fichier à titre de référence — utile pour d'autres projets ou si la config GTM change.
>
> ### Point à vérifier avec le client
> - Confirmer que le compte Axeptio (project ID `66978a322409f46d017ecfb2`) est bien géré par le client ou son agence
> - Vérifier dans l'interface Axeptio que le domaine `solutions.texaa.fr` est bien couvert (pas de filtrage par domaine qui exclurait le sous-domaine)
> - Vérifier dans GTM que le **Consent Mode v2** est bien activé sur les balises Google Ads

---

## 1. Infos à demander au client

### a) Création du compte Axeptio
- **Le client doit créer le compte** avec son email pro (`xxx@texaa.fr`) sur [axeptio.eu](https://www.axeptio.eu)
- C'est important : il restera propriétaire du compte et de la **preuve de consentement** (exigible en cas de contrôle CNIL)
- Alternative : créer le compte avec ton email puis ajouter le client comme admin

### b) Identifiants du projet Axeptio
Une fois un "Project" créé dans Axeptio, il fournit :
```
clientId: "5fbe6e032a1b1c5c9c8b4567"   ← UUID
cookiesVersion: "texaa-fr-eu"           ← slug du projet
```
Ces deux valeurs sont **indispensables** pour intégrer le widget dans `Layout.astro`.

### c) Liste exhaustive des services à tracker

À demander au client :

> *« Quels services de tracking et conversion vont tourner via GTM `GTM-WJ4ZGGSN` ? Uniquement Google Ads, ou aussi Analytics, Meta, LinkedIn ? »*

Services probables pour une landing Google Ads :

| Service | Cookies déposés | Consent Mode |
|---|---|---|
| Google Ads (conversion tracking) | `_gcl_au`, `_gcl_aw`, `_gcl_dc` | `ad_storage` + `ad_user_data` + `ad_personalization` |
| Google Analytics 4 | `_ga`, `_ga_*` | `analytics_storage` |
| Google Ads remarketing | `IDE`, `test_cookie` | `ad_storage` + `ad_personalization` |
| Meta Pixel (si utilisé) | `_fbp` | catégorie marketing custom |
| LinkedIn Insight Tag (si utilisé) | `li_*` | catégorie marketing custom |

### d) Contenu légal (mentions + politique de confidentialité)
Axeptio génère **automatiquement** la politique de cookies à partir des services configurés, mais il faut :
- **Raison sociale** : ........................
- **SIRET / RCS** : ........................
- **Adresse du siège social** : ........................
- **Email contact RGPD / DPO** : ........................
- **Hébergeur du site** : Plesk du client ? OVH ? autre ?
- **Téléphone contact** : ........................

Ces infos servent aussi pour les pages **Mentions légales** et **Politique de confidentialité** (créées en parallèle).

### e) Accès GTM
À demander au client :

> *« Qui a accès à GTM `GTM-WJ4ZGGSN` (toi, l'agence média, ou un autre prestataire) ? Pour activer le Consent Mode v2 côté GTM, j'aurai besoin d'un accès en édition. »*

Sans cet accès, le tracking ne fonctionnera pas correctement même avec Axeptio en place côté front.

### f) Configuration du widget Axeptio (interface Axeptio)
Choix à faire avec le client :
- **Langues** : français uniquement, ou aussi anglais ?
- **Position de la bannière** : "Notice" (bandeau bas) **recommandé** pour ne pas dégrader le LCP — vs "Modal" (popup central) plus intrusif
- **Couleurs** : palette Texaa (rouge `#FF0000`)
- **Texte d'intro** : court paragraphe expliquant l'usage des cookies (Axeptio fournit un template par défaut)

---

## 2. Modifications de code à faire (une fois les infos reçues)

### a) `src/layouts/Layout.astro` — Ajouter Axeptio dans le `<head>`

À placer **avant** le snippet GTM existant :

```html
<!-- Axeptio CMP + Google Consent Mode v2 -->
<script type="text/javascript" is:inline>
  window.axeptioSettings = {
    clientId: "XXXXXXXXXXXXXXXX",       // ← à remplacer par le clientId fourni
    cookiesVersion: "texaa-fr-eu",       // ← à remplacer par le cookiesVersion fourni
    googleConsentMode: {
      default: {
        analytics_storage: "denied",
        ad_storage: "denied",
        ad_user_data: "denied",
        ad_personalization: "denied",
        wait_for_update: 500
      }
    }
  };

  (function(d, s) {
    var t = d.getElementsByTagName(s)[0], e = d.createElement(s);
    e.async = true; e.src = "//static.axept.io/sdk.js";
    t.parentNode.insertBefore(e, t);
  })(document, "script");
</script>
```

### b) GTM existant — aucune modification nécessaire
Le snippet GTM `GTM-WJ4ZGGSN` reste chargé inconditionnellement. Axeptio bloque ses balises filles via Consent Mode jusqu'à acceptation. C'est la méthode officielle recommandée par Google et Axeptio.

### c) `src/pages/index.astro` — Lien "Gérer mes cookies" dans le footer

Obligatoire légalement (retrait du consentement) :

```astro
<button class="footer-link" type="button" onclick="openAxeptioCookies()">
  Gérer mes cookies
</button>
```

À ajouter dans la `<div class="footer-links">` à côté de "Mentions légales" et "Contact".

Note : `openAxeptioCookies()` est exposé globalement par le SDK Axeptio.

### d) Configuration côté GTM (interface Tag Manager)
À faire par celui qui a accès à `GTM-WJ4ZGGSN` :
1. Activer le **Consent Mode** dans les paramètres du conteneur
2. Sur chaque balise Google Ads / GA4 :
   - Cocher "Built-in consent settings"
   - Renseigner les types de consentement requis (`ad_storage`, `analytics_storage`, etc.)
3. Tester en mode preview que les balises ne se déclenchent **qu'après** acceptation

---

## 3. Limites du plan gratuit Axeptio

| Limite | Plan gratuit |
|---|---|
| Visiteurs uniques / mois | **50 000** |
| Domaines | 1 |
| Langues | 1 |
| Branding Axeptio dans la bannière | Présent (logo discret) |
| Support | Communautaire uniquement |
| Stats détaillées de consentement | Limitées |
| A/B test | Non |

**Pour une landing Google Ads qui démarre sur `solutions.texaa.fr`, le plan gratuit est largement suffisant.** À surveiller : si le compte Ads génère plus de 50k visites/mois, prévoir le passage en payant (~30-50 €/mois).

---

## 4. Checklist à transmettre au client

```
☐ 1. Compte Axeptio créé sur axeptio.eu (avec email pro Texaa)
☐ 2. Project Axeptio créé → clientId fourni : ............................
☐ 3. cookiesVersion fourni : ............................
☐ 4. Services GTM utilisés (cocher) :
       ☐ Google Ads — conversion tracking
       ☐ Google Ads — remarketing
       ☐ Google Analytics 4
       ☐ Meta Pixel (Facebook)
       ☐ LinkedIn Insight Tag
       ☐ Autre : ............................................
☐ 5. Personne ayant accès à GTM-WJ4ZGGSN : ............................
☐ 6. Infos pour mentions légales :
       - Raison sociale : ............................
       - SIRET : ............................
       - Adresse siège : ............................
       - Email contact RGPD : ............................
       - Téléphone : ............................
       - Hébergeur du site : ............................
☐ 7. Choix UX bannière Axeptio :
       - Mode : Notice (recommandé) / Modal
       - Langues : FR uniquement / FR + EN
☐ 8. Texte d'intro de la bannière (Axeptio fournit un template par défaut)
```

---

## 5. Rappel — Risques de non-conformité

| Risque | Probabilité |
|---|---|
| Amende CNIL (jusqu'à 4 % du CA mondial ou 20 M€) | Faible mais réelle pour une PME |
| Plainte d'un visiteur | Possible |
| **Perte de signaux Google Ads → CPA dégradé, CPC plus élevé** | **Quasi-certaine** sans Consent Mode v2 |
| Désapprobation éventuelle de l'annonce Google Ads | Possible si Google détecte l'absence de mentions légales / politique cookies |

---

## 6. Lien avec les autres items en attente client

Cette intégration Axeptio est **bloquée par les mêmes inputs client** que :
- Page **Mentions légales** (`/mentions-legales`)
- Page **Politique de confidentialité** (`/politique-de-confidentialite`)
- Page **Politique de cookies** (souvent intégrée à la politique de confidentialité)

→ **Idéalement, regrouper toutes les questions en un seul mail au client** pour ne pas faire 3 allers-retours.

---

## 7. Capture de leads / CRM (à valider avec le client)

La landing a vocation à générer des leads via Google Ads. Il faut décider **où et comment** ces leads sont collectés.

### Options possibles

| Outil | Gratuit ? | CRM intégré ? | Pour qui ? |
|---|---|---|---|
| **HubSpot CRM Free** | ✅ à vie (1M contacts) | ✅ complet | Le client veut centraliser dans un vrai CRM |
| **Brevo** (ex-Sendinblue) | ✅ 300 emails/jour | ✅ basique | Client français, déjà familier avec l'écosystème FR |
| **Tally** | ✅ très généreux | ❌ formulaires uniquement | Client veut juste recevoir les leads par mail |
| **Formspree** | ✅ 50 soumissions/mois | ❌ | Solution la plus minimaliste |
| **Cal.com** | ✅ | Prise de RDV | Si le commercial préfère un calendrier qu'un formulaire |
| **Airtable + form** | ✅ | Stockage tabulaire | Si le client utilise déjà Airtable |

### Recommandation pour ce projet

**HubSpot CRM Free** est probablement le meilleur choix par défaut :
- Gratuit à vie, jusqu'à 1 000 000 de contacts
- Formulaire embarqué via snippet JS, validation + antispam intégrés
- Notification email instantanée à chaque lead
- Pipeline de deals + tracking de l'historique d'interaction
- Conformité RGPD gérée nativement

**Limites du plan gratuit HubSpot à connaître** :
- Branding "Powered by HubSpot" sur les formulaires
- 2000 envois email/mois (largement suffisant pour des notifications)
- Pas de workflows automatisés (envoi auto d'email de confirmation, lead scoring, attribution)
- Pas d'A/B test sur les formulaires
- Reporting basique uniquement

→ Si besoin de workflows ou de retirer le branding : **Marketing Hub Starter** (~20 €/mois)

### Questions à poser au client

> 1. *« As-tu déjà un CRM utilisé par tes commerciaux ? (HubSpot, Salesforce, Pipedrive, Brevo, autre…) Si oui, on s'y intègre directement plutôt que d'en créer un nouveau. »*
> 2. *« Combien de leads attends-tu par mois sur cette landing ? »*
> 3. *« Qui va traiter les leads et comment ? (notification email, dashboard CRM, transfert direct au commercial…) »*

### Implémentation côté code (HubSpot)

```html
<!-- À placer dans la section formulaire de index.astro -->
<div id="hubspot-form-container"></div>
<script charset="utf-8" type="text/javascript" src="//js-eu1.hsforms.net/forms/embed/v2.js" defer></script>
<script type="text/javascript">
  hbspt.forms.create({
    region: "eu1",
    portalId: "XXXXXXXX",      // ← fourni par le client
    formId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    target: "#hubspot-form-container"
  });
</script>
```

⚠️ **Important** : ce script HubSpot dépose des cookies tiers. Il **doit être chargé après le consentement cookies** via Axeptio (ou tout autre CMP). Sinon, double infraction RGPD.

→ Bonne pratique : déclarer HubSpot comme un service Axeptio et ne charger le snippet qu'après acceptation utilisateur.

---

## 8. Protection juridique du prestataire (freelance)

> **Contexte** : Si le client refuse explicitement de mettre en place une bannière de cookies (Axeptio ou autre), c'est **lui** qui prend le risque légal. Mais en tant que prestataire, tu peux être considéré comme **co-responsable** si tu n'as pas formellement averti par écrit.

### Risques pour toi en tant que prestataire

| Risque | Détail |
|---|---|
| **Mise en cause par le client** | Si la CNIL inflige une amende à ton client, il peut tenter de se retourner contre toi en invoquant un "défaut de conseil" |
| **Réputation** | Ta signature professionnelle est associée à un site non conforme |
| **Co-responsabilité** | En droit français, le prestataire technique a un **devoir de conseil et d'alerte** sur la conformité légale d'un site qu'il livre |
| **Travail non payé** | Si le site doit être refait pour cause de non-conformité, tu peux être tenu de corriger gratuitement |

### Comment se protéger : les 4 leviers

#### 1. Document écrit d'avertissement (mail recommandé)

**Avant la mise en ligne**, envoyer au client un email récapitulatif clair signalant :
- L'obligation légale (article 82 LIL, RGPD, recommandations CNIL 2020)
- Les risques encourus (amende CNIL, perte de signaux Google Ads, plainte visiteur)
- La solution technique recommandée (Axeptio gratuit) avec son coût (0 €)
- Le délai de mise en place
- **La demande de validation explicite** : « Confirme-moi par retour de mail si tu valides la mise en place d'Axeptio, ou si tu choisis d'assumer le risque sans bannière. »

→ **L'objectif : obtenir une trace écrite** de la décision du client. Ce mail vaut preuve juridique en cas de litige ultérieur.

**Modèle de mail à adapter** :
```
Objet : Conformité RGPD landing solutions.texaa.fr — décision à valider

Bonjour [Prénom],

Avant la mise en ligne, je dois attirer ton attention sur une obligation légale.

La landing intègre Google Tag Manager (conversion Google Ads + tracking). Cela
implique le dépôt de cookies marketing soumis à consentement préalable, en
application de :
  - l'article 82 de la loi Informatique et Libertés
  - le RGPD
  - les recommandations CNIL de 2020

Sans bannière de consentement conforme + Consent Mode v2 :
  - Risque CNIL : amende administrative (jusqu'à 4 % du CA mondial)
  - Risque opérationnel : depuis mars 2024, Google dégrade les signaux de
    conversion et casse les audiences de remarketing si le Consent Mode n'est
    pas en place. Concrètement, ton CPA grimpera et ton CPC aussi.

Solution recommandée : Axeptio (plan gratuit, 50k visites/mois, 0 €).
J'ai préparé toute la documentation technique (cf. AXEPTIO-INTEGRATION.md
fourni avec le projet).

Merci de me confirmer par retour de mail :
  ☐ Option A : "Je valide la mise en place d'Axeptio. Voici les infos
                 demandées (compte créé, clientId, etc.)"
  ☐ Option B : "Je choisis de ne pas mettre en place de bannière cookies
                 et j'assume les risques légaux et opérationnels associés."

Sans réponse écrite, je ne peux pas livrer le site en production.

Bonne journée,
[Ton prénom]
```

#### 2. Clause d'exonération dans le devis / contrat

**Ajouter dans le devis et/ou les CGV** une clause qui transfère la responsabilité au client en cas de non-conformité de son fait.

**Modèle de clause à intégrer** :
```
ARTICLE X — CONFORMITÉ LÉGALE ET RGPD

Le PRESTATAIRE attire l'attention du CLIENT sur les obligations légales
applicables aux sites web français en matière de :
  - protection des données personnelles (RGPD)
  - dépôt de traceurs et cookies (article 82 de la loi Informatique et
    Libertés du 6 janvier 1978 modifiée, recommandations CNIL)
  - mentions légales obligatoires (LCEN, article 6-III)

Le PRESTATAIRE recommande expressément la mise en place :
  - d'une bannière de consentement aux cookies conforme aux recommandations
    CNIL (Consent Management Platform)
  - de pages "Mentions légales" et "Politique de confidentialité"

La mise en œuvre effective de ces dispositifs relève de la décision et de
la responsabilité du CLIENT. En cas de refus ou de non-mise en œuvre par
le CLIENT, ce dernier reconnaît expressément assumer l'intégralité des
risques juridiques, financiers et opérationnels qui en découlent, et
décharge le PRESTATAIRE de toute responsabilité à ce titre, conformément
à l'article 1112-1 du Code civil relatif au devoir d'information.

Le CLIENT s'engage par ailleurs à fournir au PRESTATAIRE l'ensemble des
informations légales nécessaires (raison sociale, SIRET, hébergeur, contact
DPO, finalités de traitement) dans un délai de [X] jours suivant la
demande, sous peine de blocage de la mise en production du site.
```

#### 3. Mention sur le devis / la facture

**Ajouter une ligne explicite** sur le devis ou la facture finale :

```
NOTE : La conformité RGPD (bannière de cookies, mentions légales, politique
de confidentialité) n'est PAS incluse dans cette prestation. Le CLIENT a été
informé des obligations légales applicables et reconnaît assumer la
responsabilité de leur mise en œuvre.
```

Ou inversement, si Axeptio est inclus :

```
NOTE : La prestation inclut l'intégration technique d'une bannière de
cookies (Axeptio plan gratuit) configurée selon les recommandations CNIL.
Le CLIENT reste seul responsable du contenu juridique des pages "Mentions
légales" et "Politique de confidentialité".
```

#### 4. Limitation de responsabilité chiffrée

Dans les **CGV** ou le devis, ajouter un plafond de responsabilité :

```
ARTICLE Y — LIMITATION DE RESPONSABILITÉ

La responsabilité du PRESTATAIRE, tous dommages confondus, est expressément
limitée au montant total HT effectivement perçu au titre de la prestation
concernée. En aucun cas le PRESTATAIRE ne pourra être tenu responsable :
  - des dommages indirects (perte de chiffre d'affaires, perte de clientèle,
    perte de données, manque à gagner, atteinte à l'image)
  - des amendes administratives ou pénales infligées au CLIENT
  - des conséquences d'un refus du CLIENT de mettre en œuvre les
    recommandations techniques ou légales du PRESTATAIRE
```

### Checklist de protection — récap

| Action | À faire quand ? | Importance |
|---|---|---|
| ☐ Mail d'avertissement avec demande de validation écrite | **Avant la mise en ligne** | 🔴 Critique |
| ☐ Clause RGPD dans le devis signé | **Avant le démarrage** | 🔴 Critique |
| ☐ Mention explicite sur la facture finale | **À la livraison** | 🟠 Important |
| ☐ Clause de limitation de responsabilité (CGV) | **À mettre une fois pour tous tes clients** | 🔴 Critique |
| ☐ Sauvegarder les échanges écrits dans un dossier dédié | **En continu** | 🟠 Important |

### Points juridiques de référence (à citer en cas de besoin)

- **Article 82** de la loi Informatique et Libertés (cookies / consentement)
- **Articles 12 à 14 du RGPD** (information et transparence vis-à-vis des personnes concernées)
- **Article 1112-1 du Code civil** (devoir précontractuel d'information)
- **Recommandations CNIL** sur les cookies et autres traceurs (mises à jour 2020/2021)
- **LCEN, article 6-III** (mentions légales obligatoires des éditeurs de site)

### ⚠️ Limites de cette protection

- **Tu n'es pas avocat.** Pour un projet à fort enjeu (e-commerce, gros volume de leads, données sensibles santé/finance), recommander au client de **consulter un juriste** ou un DPO externe pour valider sa conformité.
- **La clause d'exonération a des limites** : un juge peut la requalifier si elle est abusive ou si tu as manifestement manqué à ton devoir de conseil. D'où l'importance du **mail d'avertissement écrit**.
- **Pour les très petits projets** (< 1000 €), le risque juridique est faible mais le mail d'avertissement reste une bonne pratique systématique.

---

## 9. Synthèse "à faire avant livraison" — contexte actuel

### 🟢 Résolu automatiquement (Axeptio via GTM)

| # | Action | Statut |
|---|---|---|
| 1 | Bannière cookies / CMP | ✅ Géré par Axeptio via GTM |
| 2 | Consent Mode v2 | ✅ Probablement actif (à confirmer dans GTM) |
| 3 | Protection prestataire RGPD | ✅ Plus de risque — la conformité est assurée |

### 🟠 Actions techniques restantes (code)

| # | Action | Statut |
|---|---|---|
| 4 | Créer le scénario Make + remplacer le placeholder webhook dans `index.astro` | ⏳ |
| 5 | Récupérer les infos légales client pour les mentions légales | ⏳ |
| 6 | Créer pages Mentions légales + Politique de confidentialité | ⏳ |
| 7 | Confirmer avec le client que l'Axeptio GTM couvre bien `solutions.texaa.fr` | ⏳ |
| 8 | Test final avant déploiement sur `solutions.texaa.fr` | ⏳ |

### 📝 Conservés à titre de référence

La **protection juridique prestataire** (§ 8) reste dans ce fichier. Elle n'est plus nécessaire pour ce projet (Axeptio déjà actif) mais les modèles de clauses, mails et mentions facture sont **réutilisables pour d'autres projets clients**.
