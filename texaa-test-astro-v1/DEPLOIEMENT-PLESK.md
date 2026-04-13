# Déploiement de la landing Astro sur `solutions.texaa.fr` (Plesk)

Ce projet Astro est en mode **statique** (pas d'`output: 'server'` dans `astro.config.mjs`), donc `npm run build` génère un dossier `dist/` avec uniquement du HTML/CSS/JS.

C'est l'idéal pour Plesk : **pas besoin de Node.js sur le serveur**, juste un hébergement de fichiers statiques.

---

## 1. Accès à demander au client

### a) Accès Plesk (le plus simple)
- URL du panel Plesk (ex: `https://serveur.texaa.fr:8443`)
- Identifiant + mot de passe d'un compte avec droits sur le domaine `texaa.fr`
- Idéalement le rôle **"Administrateur du domaine"** (pas besoin du root serveur)

### b) Accès DNS
- Soit le DNS est géré dans Plesk → l'accès Plesk suffit
- Soit le DNS est ailleurs (OVH, Cloudflare, Gandi…) → demander qui gère la zone DNS de `texaa.fr`
- Il faudra créer un **A record** `solutions.texaa.fr` → IP du serveur (ou un CNAME vers le domaine principal)

### c) Accès FTP/SFTP (optionnel mais pratique)
- Hôte, port, login, mot de passe SFTP
- Permet d'uploader le `dist/` sans passer par l'interface Plesk

---

## 2. Étapes côté Plesk

1. Aller dans le domaine `texaa.fr` → **Ajouter un sous-domaine**
2. Nom : `solutions` → Plesk crée automatiquement un dossier (souvent `httpdocs/solutions` ou `solutions.texaa.fr/`)
3. Activer **Let's Encrypt SSL** sur le sous-domaine (onglet "Certificats SSL/TLS")
4. Dans **Hosting Settings** du sous-domaine, le support PHP peut être **désactivé** (c'est du statique pur)
5. Vérifier que le DNS A record est bien créé (Plesk le fait automatiquement si le DNS est interne)

---

## 3. Build et upload

1. Build local :
   ```bash
   npm run build
   ```
2. Uploader **le contenu** du dossier `dist/` (pas le dossier lui-même) dans le répertoire racine du sous-domaine, via SFTP ou le File Manager Plesk
3. Tester `https://solutions.texaa.fr`

---

## 4. Points d'attention WordPress

- Le WordPress du client tourne sur `texaa.fr` ou `www.texaa.fr` → **aucun conflit** avec un sous-domaine, ce sont des Virtual Hosts séparés dans Plesk
- Pas de risque de casser le WP du client tant qu'on touche uniquement au sous-domaine
- Si le WP a un fichier `.htaccess` avec des règles globales, vérifier qu'il n'impacte pas le sous-domaine (normalement non, chaque sous-domaine a son propre docroot)

---

## 5. Workflow de déploiement futur

Pour éviter de re-uploader manuellement à chaque modif :
- **SFTP + script** (ex: `rsync` ou une extension VS Code type SFTP) → le plus simple
- **Git via Plesk** : si Plesk a Git activé, on peut brancher un repo qui pull automatiquement, mais ça demande Node.js sur le serveur pour build
- **Build local + rsync** : workflow recommandé au début

Exemple de commande rsync (à adapter avec les vrais accès) :
```bash
rsync -avz --delete dist/ user@serveur:/var/www/vhosts/texaa.fr/solutions.texaa.fr/
```

---

## Récap rapide

| Étape | Action |
|---|---|
| 1 | Demander accès Plesk + (DNS si externe) + SFTP au client |
| 2 | Créer sous-domaine `solutions` dans Plesk |
| 3 | Activer Let's Encrypt SSL |
| 4 | `npm run build` en local |
| 5 | Upload du contenu de `dist/` via SFTP |
| 6 | Tester `https://solutions.texaa.fr` |
