# Scénario Make — Formulaire lead `solutions.texaa.fr`

> **Architecture** : Le formulaire du site fait un `fetch()` POST JSON vers un webhook Make. Make vérifie le token reCAPTCHA v3, puis exécute 4 actions : email de confirmation au visiteur, 2 emails admin à des destinataires différents, et ajout d'une ligne dans Google Sheets.

---

## 1. Overview du scénario

```
[Webhooks: Custom webhook]
           │
           ▼
[HTTP: Verify reCAPTCHA v3] ──score < 0.5──▶ [STOP — bot détecté]
           │
           ▼ (score >= 0.5 = humain)
           ├──▶ [Email: Confirmation visiteur]
           ├──▶ [Email: Notification admin #1]
           ├──▶ [Email: Notification admin #2]
           └──▶ [Google Sheets: Add a Row]
```

Les 4 actions après reCAPTCHA peuvent être configurées **en série** (plus sûr) ou **en parallèle via un Router** (plus rapide). Pour commencer, je recommande la **série** — c'est plus simple à déboguer.

---

## 2. Payload JSON reçu par le webhook

```json
{
  "lastname": "Dupont",
  "firstname": "Marie",
  "establishment_type": "enseignement",
  "city": "Bordeaux",
  "phone": "0612345678",
  "email": "marie.dupont@exemple.fr",
  "comments": "Je cherche une solution pour mon open-space de 200m²",
  "documents": ["nuancier", "brochure"],
  "recaptcha_token": "03AGdBq24Pf...(long token)...",
  "submitted_at": "2026-04-11T17:40:00.000Z",
  "source": "solutions.texaa.fr"
}
```

### Valeurs possibles pour `establishment_type`
| Valeur | Label affiché côté visiteur |
|---|---|
| `espaces-travail` | Espaces de travail |
| `enseignement` | Enseignement |
| `restaurant` | Restaurant |
| `hotel` | Hôtel |
| `hopitaux` | Hôpitaux |
| `autres` | Autres |

### Valeurs possibles pour `documents` (array)
| Valeur | Label |
|---|---|
| `nuancier` | Nuancier |
| `brochure` | Brochure |
| `coffret` | Coffret prescripteur |

Le tableau peut être vide (aucun document coché) ou contenir 1 à 3 valeurs.

---

## 3. Étapes de mise en place côté Make

### Étape 3.1 — Créer le scénario
1. Aller sur [make.com](https://www.make.com) → **Create a new scenario**
2. Choisir le module **Webhooks → Custom webhook**
3. Cliquer **Add** → nommer le webhook `texaa-landing-lead`
4. Cliquer **Copy address to clipboard** → **coller l'URL dans `src/pages/index.astro`** à la ligne `const WEBHOOK_URL = '...'`
5. Cliquer **Run once** dans Make pour le mettre en mode écoute
6. Soumettre le formulaire sur le site pour que Make reçoive un premier payload (sert de "structure reference")

### Étape 3.2 — Module HTTP : Vérification reCAPTCHA v3

Juste après le webhook, ajouter un module **HTTP → Make a request** pour vérifier le token reCAPTCHA.

**Configuration du module HTTP** :
- **URL** : `https://www.google.com/recaptcha/api/siteverify`
- **Method** : POST
- **Body type** : `application/x-www-form-urlencoded`
- **Fields** :
  - `secret` = `6LfdGbcsAAAAAKN4VclTWnL5xiSVacNSAaalQWLY`
  - `response` = `{{1.recaptcha_token}}`

**Réponse de Google** (JSON) :
```json
{
  "success": true,
  "score": 0.9,
  "action": "submit_lead",
  "hostname": "solutions.texaa.fr"
}
```

**Ajouter un Filter** entre le module HTTP et le module Email #1 :
- **Condition** : `{{2.data.score}}` **greater than or equal to** `0.5`
- **Label** : "Score reCAPTCHA suffisant"

→ Si le score est < 0.5 (probable bot), le scénario **s'arrête** et rien n'est envoyé. Le visiteur voit le message d'erreur côté front.

**Cas où `recaptcha_token` est vide** (reCAPTCHA n'a pas pu charger côté visiteur) :
- Ajouter une **2e route** dans un Router : si `{{1.recaptcha_token}}` est vide, laisser passer quand même (pour ne pas perdre de vrais leads dont le navigateur bloque reCAPTCHA). À toi de décider si tu acceptes ce cas ou non.

### Étape 3.3 — Module Email #1 : Confirmation visiteur

**Module à choisir** (au choix selon les besoins du client) :
- **Gmail** (si le client a un Google Workspace)
- **Email (SMTP)** (si serveur SMTP dédié)
- **Microsoft 365 Email** (si Exchange / Outlook)

**Configuration** :
- **To** : `{{1.email}}` (l'email du visiteur)
- **Subject** : `Votre demande est bien reçue — Texaa`
- **Content type** : HTML
- **Body** : cf. template **T1** ci-dessous

### Étape 3.3 — Module Email #2 : Notification admin #1

- **To** : `admin1@texaa.fr` *(à remplacer par l'adresse réelle)*
- **Subject** : `🔔 Nouveau lead — {{1.firstname}} {{1.lastname}} ({{1.establishment_type}})`
- **Content type** : HTML
- **Body** : cf. template **T2** ci-dessous

### Étape 3.4 — Module Email #3 : Notification admin #2

- **To** : `admin2@texaa.fr` *(à remplacer par l'adresse réelle)*
- **Subject** : `Nouveau lead solutions.texaa.fr — {{1.establishment_type}}`
- **Content type** : HTML
- **Body** : cf. template **T3** ci-dessous (version synthétique)

### Étape 3.5 — Module Google Sheets : Add a Row

1. Module **Google Sheets → Add a Row**
2. Se connecter au compte Google du client
3. Créer (ou sélectionner) un Google Sheet nommé `Texaa - Leads solutions.texaa.fr`
4. Créer un onglet `Leads` avec les colonnes suivantes en ligne 1 :

| Colonne | En-tête | Mapping Make |
|---|---|---|
| A | Date | `{{formatDate(1.submitted_at; "DD/MM/YYYY HH:mm")}}` |
| B | Nom | `{{1.lastname}}` |
| C | Prénom | `{{1.firstname}}` |
| D | Type d'établissement | `{{1.establishment_type}}` |
| E | Ville | `{{1.city}}` |
| F | Téléphone | `{{1.phone}}` |
| G | Email | `{{1.email}}` |
| H | Commentaires | `{{1.comments}}` |
| I | Documents souhaités | `{{join(1.documents; ", ")}}` |
| J | Source | `{{1.source}}` |

→ Dans le module Make : sélectionner le Sheet + l'onglet `Leads` et mapper chaque cellule vers les valeurs ci-dessus.

### Étape 3.6 — Tester
1. Dans Make, cliquer **Run once**
2. Soumettre le formulaire sur le site (version preview locale ou staging)
3. Vérifier :
   - ✅ Email de confirmation reçu sur l'adresse du visiteur test
   - ✅ Email #1 reçu sur admin1
   - ✅ Email #2 reçu sur admin2
   - ✅ Nouvelle ligne ajoutée dans Google Sheets
4. Si tout est OK → activer le scénario (toggle **Scheduling** sur ON, mode "Immediately")

---

## 4. Templates d'emails (HTML prêts à coller)

### T1 — Email de confirmation visiteur

**Subject** : `Votre demande est bien reçue — Texaa`

**Body (HTML)** — copier-coller tel quel dans Make, puis remplacer les variables via le sélecteur Make :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Votre demande est bien reçue — Texaa</title>
</head>
<body style="margin:0; padding:0; font-family: Arial, Helvetica, sans-serif; background:#f5f5f5; color:#262626;">
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background:#f5f5f5; padding:32px 16px;">
    <tr>
      <td align="center">
        <table role="presentation" width="560" cellpadding="0" cellspacing="0" style="background:#ffffff; border-radius:12px; overflow:hidden; box-shadow:0 4px 20px rgba(0,0,0,0.06);">
          <tr>
            <td style="background:#171717; padding:32px 40px; text-align:center;">
              <div style="font-size:22px; font-weight:700; color:#ffffff; letter-spacing:0.02em;">TEXAA</div>
              <div style="font-size:12px; color:#a3a3a3; margin-top:4px;">Fabricant français depuis 1979</div>
            </td>
          </tr>
          <tr>
            <td style="padding:48px 40px 32px;">
              <div style="display:inline-block; padding:6px 14px; background:rgba(255,0,0,0.08); color:#FF0000; font-size:11px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; border-radius:100px; margin-bottom:20px;">Demande reçue</div>
              <h1 style="margin:0 0 16px; font-size:28px; color:#171717; letter-spacing:-0.02em;">Merci {{1.firstname}} !</h1>
              <p style="margin:0 0 16px; font-size:16px; line-height:1.6; color:#525252;">Votre demande nous est bien parvenue. Un expert Texaa va étudier votre projet et vous recontactera dans les meilleurs délais.</p>
              <p style="margin:0 0 32px; font-size:16px; line-height:1.6; color:#525252;">En attendant, vous pouvez nous joindre directement au <strong style="color:#171717;">05 56 46 01 63</strong> pour toute question.</p>
              <div style="background:#fafafa; border:1px solid #e5e5e5; border-radius:10px; padding:20px 24px; margin-bottom:32px;">
                <div style="font-size:11px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; color:#737373; margin-bottom:12px;">Récapitulatif de votre demande</div>
                <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="font-size:14px; color:#404040;">
                  <tr><td style="padding:6px 0; width:140px; color:#737373;">Secteur</td><td style="padding:6px 0;">{{1.establishment_type}}</td></tr>
                  <tr><td style="padding:6px 0; color:#737373;">Ville</td><td style="padding:6px 0;">{{1.city}}</td></tr>
                  <tr><td style="padding:6px 0; color:#737373;">Téléphone</td><td style="padding:6px 0;">{{1.phone}}</td></tr>
                  <tr><td style="padding:6px 0; color:#737373;">Documents</td><td style="padding:6px 0;">{{join(1.documents; ", ")}}</td></tr>
                </table>
              </div>
              <div style="text-align:center;">
                <a href="tel:+33556460163" style="display:inline-block; padding:14px 28px; background:#FF0000; color:#ffffff; text-decoration:none; font-weight:600; font-size:15px; border-radius:8px;">Appelez-nous : 05 56 46 01 63</a>
              </div>
            </td>
          </tr>
          <tr>
            <td style="background:#fafafa; padding:24px 40px; text-align:center; border-top:1px solid #e5e5e5;">
              <div style="font-size:12px; color:#737373; line-height:1.6;">Texaa — Solutions acoustiques textiles sur-mesure — Bordeaux, France</div>
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
</body>
</html>
```

---

### T2 — Email notification admin #1 (complet)

**Subject** : `Nouveau lead — {{1.firstname}} {{1.lastname}} ({{1.establishment_type}})`

**Body (HTML)** :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Nouveau lead — solutions.texaa.fr</title>
</head>
<body style="margin:0; padding:0; font-family: Arial, Helvetica, sans-serif; background:#f5f5f5; color:#262626;">
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background:#f5f5f5; padding:32px 16px;">
    <tr>
      <td align="center">
        <table role="presentation" width="640" cellpadding="0" cellspacing="0" style="background:#ffffff; border-radius:12px; overflow:hidden;">
          <tr>
            <td style="background:#171717; padding:24px 40px;">
              <div style="font-size:12px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; color:#a3a3a3;">Nouveau lead — solutions.texaa.fr</div>
              <div style="font-size:22px; font-weight:700; color:#ffffff; margin-top:4px;">{{1.firstname}} {{1.lastname}}</div>
              <div style="font-size:14px; color:#a3a3a3; margin-top:2px;">{{1.establishment_type}} — {{1.city}}</div>
            </td>
          </tr>
          <tr>
            <td style="padding:32px 40px;">
              <div style="font-size:11px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; color:#737373; margin-bottom:16px;">Coordonnées</div>
              <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="font-size:14px; color:#262626; margin-bottom:24px;">
                <tr><td style="padding:8px 0; width:160px; color:#737373;">Nom complet</td><td style="padding:8px 0;"><strong>{{1.firstname}} {{1.lastname}}</strong></td></tr>
                <tr><td style="padding:8px 0; color:#737373;">Email</td><td style="padding:8px 0;"><a href="mailto:{{1.email}}" style="color:#262626;">{{1.email}}</a></td></tr>
                <tr><td style="padding:8px 0; color:#737373;">Téléphone</td><td style="padding:8px 0;"><a href="tel:{{1.phone}}" style="color:#262626;">{{1.phone}}</a></td></tr>
                <tr><td style="padding:8px 0; color:#737373;">Ville</td><td style="padding:8px 0;">{{1.city}}</td></tr>
                <tr><td style="padding:8px 0; color:#737373;">Secteur</td><td style="padding:8px 0;"><strong>{{1.establishment_type}}</strong></td></tr>
              </table>
              <div style="font-size:11px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; color:#737373; margin-bottom:8px;">Commentaires</div>
              <div style="padding:16px; background:#fafafa; border-left:3px solid #d4d4d4; border-radius:4px; font-size:14px; line-height:1.6; color:#404040; margin-bottom:24px;">{{1.comments}}</div>
              <div style="font-size:11px; font-weight:600; text-transform:uppercase; letter-spacing:0.08em; color:#737373; margin-bottom:8px;">Documents demandés</div>
              <div style="font-size:14px; color:#262626; margin-bottom:24px;">{{join(1.documents; ", ")}}</div>
              <div style="padding-top:24px; border-top:1px solid #e5e5e5; font-size:12px; color:#737373;">Source : {{1.source}}</div>
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
</body>
</html>
```

---

### T3 — Email notification admin #2 (synthétique)

**Subject** : `Nouveau lead solutions.texaa.fr — {{1.establishment_type}}`

**Body (texte simple)** :

```
Nouveau lead reçu depuis solutions.texaa.fr

Nom : {{1.firstname}} {{1.lastname}}
Email : {{1.email}}
Téléphone : {{1.phone}}
Ville : {{1.city}}
Secteur : {{1.establishment_type}}
Commentaires : {{1.comments}}
Documents demandés : {{join(1.documents; ", ")}}
```

> T3 est volontairement court — notification rapide pour le 2e admin.

---

## 5. Mapping des valeurs lisibles (optionnel mais recommandé)

Le champ `establishment_type` envoie des valeurs techniques (`espaces-travail`, `hopitaux`…). Pour que les emails affichent des libellés lisibles, ajouter un module **Tools → Switch** dans Make juste après le webhook :

| Input `{{1.establishment_type}}` | Output `secteur_label` |
|---|---|
| `espaces-travail` | Espaces de travail |
| `enseignement` | Enseignement |
| `restaurant` | Restaurant |
| `hotel` | Hôtel |
| `hopitaux` | Hôpitaux |
| `autres` | Autres |

Puis dans les emails, utiliser `{{secteur_label}}` au lieu de `{{1.establishment_type}}`.

Même logique possible pour `documents` :
- `nuancier` → Nuancier
- `brochure` → Brochure
- `coffret` → Coffret prescripteur

---

## 6. Checklist de déploiement

| # | Action | Responsable |
|---|---|---|
| 1 | Créer le scénario Make avec les 4 modules | Toi |
| 2 | Récupérer l'URL du webhook Make | Toi |
| 3 | Remplacer `REPLACE_ME_WITH_REAL_WEBHOOK_ID` dans `src/pages/index.astro` par l'URL réelle | Toi |
| 4 | Configurer les templates d'email (T1, T2, T3) dans les modules | Toi |
| 5 | Récupérer auprès du client : email de confirmation (from), admin1@..., admin2@... | Client |
| 6 | Créer le Google Sheets partagé + l'onglet Leads avec les 10 colonnes | Toi + Client |
| 7 | Connecter le compte Google Sheets à Make | Toi + Client |
| 8 | Tester le scénario end-to-end avec un lead fictif | Toi |
| 9 | Activer le scénario (toggle Scheduling = ON) | Toi |
| 10 | Rebuild + déploiement sur `solutions.texaa.fr` | Toi |

---

## 7. Points d'attention

### Rate limiting / spam
Make n'inclut pas de protection anti-spam native côté webhook. Si tu constates du spam :
- Ajouter un **Filter** dans Make après le webhook qui vérifie la longueur min du commentaire ou la validité de l'email
- Ou réintégrer un honeypot côté form (on a volontairement retiré)
- Ou activer **reCAPTCHA v3** côté HubSpot si on revient à HubSpot plus tard

### Coût Make
Le plan gratuit Make donne **1000 opérations / mois**. Chaque soumission = 5 opérations (webhook + 4 modules). Donc **~200 leads/mois max en gratuit**. Au-delà, plan Core à 9 €/mois (10k opérations).

### Si Make est down
Le script JS détecte l'erreur et affiche un message demandant d'appeler le numéro. Le visiteur ne perd pas sa saisie (le formulaire reste rempli).

### RGPD
Pas de CMP sur le site (décision client). Les données personnelles transitent par Make (Irlande/EU), Google Sheets (Google Workspace) et les services email utilisés. Vérifier que tous les sous-traitants sont déclarés dans la politique de confidentialité du client.
