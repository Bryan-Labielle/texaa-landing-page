# Backup — Formulaire custom Texaa (avant migration HubSpot)

> **Contexte** : ce fichier contient le code HTML + CSS du formulaire custom qui existait dans `index.astro` avant son remplacement par le script HubSpot embed.
>
> **Pourquoi conserver ce code ?**
> - Référence design si on veut reproduire l'apparence dans HubSpot (via le CSS custom dans les réglages du formulaire)
> - Rollback possible si HubSpot pose un problème bloquant
> - Source de vérité pour les champs et la logique métier du formulaire (champs demandés, cases à cocher documents souhaités)
>
> **Date du backup** : 2026-04-11

---

## 1. Liste des champs du formulaire

| Champ | Type | Obligatoire | Placeholder |
|---|---|---|---|
| `lastname` | text | ✅ | Votre nom |
| `firstname` | text | ✅ | Votre prénom |
| `city` | text | ✅ | Votre ville |
| `phone` | tel | ✅ | 06 XX XX XX XX |
| `email` | email | ✅ | vous@entreprise.fr |
| `comments` | textarea | ❌ | Décrivez votre projet ou vos questions... |
| `documents[]` | checkbox multi | ❌ | nuancier / brochure / coffret |

### Cases à cocher "Documents souhaités"
1. **Nuancier** — Version papier — teintes & textures
2. **Brochure** — Documentation technique produits
3. **Coffret prescripteur** — Brochure + nuancier textile + mailles

→ À reproduire côté HubSpot comme un champ multi-checkbox.

---

## 2. HTML du formulaire (tel qu'il était dans `index.astro`)

```astro
<div class="action-form-wrapper">
	<h3>Demande de contact / rappel expert</h3>
	<p class="form-subtitle">Recevez notre catalogue d'échantillons ou demandez un rappel pour discuter de votre projet avec un expert.</p>

	<form action="#" method="POST">
		<div class="form-grid">
			<div class="form-group">
				<label for="lastname">Nom *</label>
				<input type="text" id="lastname" name="lastname" placeholder="Votre nom" required>
			</div>

			<div class="form-group">
				<label for="firstname">Prénom *</label>
				<input type="text" id="firstname" name="firstname" placeholder="Votre prénom" required>
			</div>

			<div class="form-group">
				<label for="city">Ville *</label>
				<input type="text" id="city" name="city" placeholder="Votre ville" required>
			</div>

			<div class="form-group">
				<label for="phone">Téléphone *</label>
				<input type="tel" id="phone" name="phone" placeholder="06 XX XX XX XX" required>
			</div>

			<div class="form-group full-width">
				<label for="email">Email professionnel *</label>
				<input type="email" id="email" name="email" placeholder="vous@entreprise.fr" required>
			</div>

			<div class="form-group full-width">
				<label for="comments">Commentaires</label>
				<textarea id="comments" name="comments" rows="4" placeholder="Décrivez votre projet ou vos questions..."></textarea>
			</div>
		</div>

		<!-- Documents souhaités -->
		<div class="form-documents">
			<label class="form-documents-label">Documents souhaités <span>(optionnel)</span></label>
			<div class="form-documents-grid">
				<label class="document-checkbox">
					<input type="checkbox" name="documents" value="nuancier">
					<div class="document-checkbox-content">
						<div class="document-icon document-icon-nuancier">
							<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
								<path stroke-linecap="round" stroke-linejoin="round" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
							</svg>
						</div>
						<div class="document-info">
							<strong>Nuancier</strong>
							<span>Version papier — teintes & textures</span>
						</div>
					</div>
				</label>

				<label class="document-checkbox">
					<input type="checkbox" name="documents" value="brochure">
					<div class="document-checkbox-content">
						<div class="document-icon document-icon-brochure">
							<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
								<path stroke-linecap="round" stroke-linejoin="round" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
							</svg>
						</div>
						<div class="document-info">
							<strong>Brochure</strong>
							<span>Documentation technique produits</span>
						</div>
					</div>
				</label>

				<label class="document-checkbox">
					<input type="checkbox" name="documents" value="coffret">
					<div class="document-checkbox-content">
						<div class="document-icon document-icon-coffret">
							<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
								<path stroke-linecap="round" stroke-linejoin="round" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4" />
							</svg>
						</div>
						<div class="document-info">
							<strong>Coffret prescripteur</strong>
							<span>Brochure + nuancier textile + mailles</span>
						</div>
					</div>
				</label>
			</div>
		</div>

		<div class="form-submit">
			<button type="submit" class="btn btn-primary btn-lg btn-full">
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
					<path stroke-linecap="round" stroke-linejoin="round" d="M17 8l4 4m0 0l-4 4m4-4H3" />
				</svg>
				Envoyer ma demande — Réponse sous 24h
			</button>
		</div>

		<p class="form-note">Vos données sont traitées uniquement dans le cadre de votre demande. Conformément au RGPD, vous pouvez exercer vos droits à tout moment en nous contactant.</p>
	</form>
</div>
```

---

## 3. CSS du formulaire (tel qu'il était dans `Layout.astro`)

### 3.1 Grille + inputs de base

```css
.form-grid {
	display: grid;
	grid-template-columns: 1fr 1fr;
	gap: 16px;
}

.form-group {
	margin-bottom: 4px;
}

.form-group.full-width {
	grid-column: 1 / -1;
}

.form-group label {
	display: block;
	font-family: var(--font-display);
	font-weight: 500;
	font-size: 14px;
	color: var(--gray-700);
	margin-bottom: 6px;
}

.form-group input,
.form-group select {
	width: 100%;
	padding: 12px 16px;
	font-family: var(--font-body);
	font-size: 15px;
	color: var(--gray-800);
	background: var(--gray-50);
	border: 1px solid var(--gray-200);
	border-radius: 8px;
	transition: all var(--transition-fast);
}

.form-group input:focus,
.form-group select:focus {
	outline: none;
	border-color: var(--texaa-red);
	background: var(--white);
	box-shadow: 0 0 0 3px var(--texaa-red-light);
}

.form-group input::placeholder {
	color: var(--gray-400);
}
```

### 3.2 Textarea

```css
.form-group textarea {
	width: 100%;
	padding: 12px 16px;
	font-family: var(--font-body);
	font-size: 15px;
	color: var(--gray-800);
	background: var(--gray-50);
	border: 1px solid var(--gray-200);
	border-radius: 8px;
	resize: vertical;
	min-height: 100px;
	transition: all var(--transition-fast);
}

.form-group textarea:focus {
	outline: none;
	border-color: var(--texaa-red);
	background: var(--white);
	box-shadow: 0 0 0 3px var(--texaa-red-light);
}

.form-group textarea::placeholder {
	color: var(--gray-400);
}
```

### 3.3 Bouton submit + note RGPD

```css
.form-submit {
	margin-top: 24px;
}

.form-submit .btn {
	width: 100%;
}

.form-note {
	text-align: center;
	font-size: 12px;
	color: var(--gray-500);
	margin-top: 16px;
	line-height: 1.6;
}

.btn-full {
	width: 100%;
}
```

### 3.4 Sous-titre du formulaire

```css
.form-subtitle {
	font-size: 15px;
	color: var(--gray-600);
	margin-bottom: 24px;
	line-height: 1.6;
}
```

### 3.5 Cases à cocher "Documents souhaités"

```css
.form-documents {
	margin-top: 24px;
	padding-top: 24px;
	border-top: 1px solid var(--gray-200);
}

.form-documents-label {
	display: block;
	font-family: var(--font-display);
	font-weight: 500;
	font-size: 14px;
	color: var(--gray-700);
	margin-bottom: 16px;
}

.form-documents-label span {
	font-weight: 400;
	color: var(--gray-500);
}

.form-documents-grid {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 12px;
}

.document-checkbox {
	position: relative;
	cursor: pointer;
}

.document-checkbox input {
	position: absolute;
	opacity: 0;
	width: 0;
	height: 0;
}

.document-checkbox-content {
	display: flex;
	flex-direction: column;
	align-items: center;
	text-align: center;
	padding: 20px 12px;
	background: var(--gray-50);
	border: 2px solid var(--gray-200);
	border-radius: 12px;
	transition: all var(--transition-fast);
}

.document-checkbox input:checked + .document-checkbox-content {
	border-color: var(--texaa-red);
	background: var(--texaa-red-light);
}

.document-checkbox:hover .document-checkbox-content {
	border-color: var(--gray-300);
}

.document-icon {
	width: 48px;
	height: 48px;
	display: flex;
	align-items: center;
	justify-content: center;
	background: var(--white);
	border-radius: 8px;
	margin-bottom: 12px;
}

.document-icon svg {
	width: 24px;
	height: 24px;
	color: var(--gray-500);
}

.document-icon-nuancier svg {
	color: #E91E63;
}

.document-icon-brochure svg {
	color: #2196F3;
}

.document-icon-coffret svg {
	color: #FF9800;
}

.document-info strong {
	display: block;
	font-family: var(--font-display);
	font-size: 13px;
	font-weight: 600;
	color: var(--gray-900);
	margin-bottom: 4px;
}

.document-info span {
	font-size: 11px;
	color: var(--gray-500);
	line-height: 1.4;
}
```

### 3.6 Responsive (mobile ≤ 768px)

```css
@media (max-width: 768px) {
	.form-grid {
		grid-template-columns: 1fr;
	}

	.form-documents-grid {
		grid-template-columns: 1fr;
	}

	.document-checkbox-content {
		flex-direction: row;
		text-align: left;
		padding: 16px;
		gap: 16px;
	}

	.document-icon {
		margin-bottom: 0;
	}

	.action-form-wrapper {
		padding: 24px;
	}
}
```

---

## 4. Notes pour la migration HubSpot

Pour reproduire cette expérience dans HubSpot Forms :

### a) Créer les champs dans HubSpot
1. `lastname` (Nom) — texte, obligatoire
2. `firstname` (Prénom) — texte, obligatoire
3. `city` (Ville) — texte, obligatoire — à créer en propriété custom dans HubSpot si elle n'existe pas par défaut
4. `phone` (Téléphone) — téléphone, obligatoire
5. `email` (Email professionnel) — email, obligatoire
6. `comments` (Commentaires) — textarea, optionnel — propriété custom
7. `documents_souhaites` (Documents souhaités) — multiple checkbox, optionnel, valeurs : Nuancier / Brochure / Coffret prescripteur — propriété custom

### b) Redirection après soumission
- Configurer la redirection vers `/merci` dans les options du formulaire HubSpot
- URL : `https://solutions.texaa.fr/merci`
- Cela permet de tracker la conversion Google Ads sur la pageview de `/merci`

### c) Style du formulaire HubSpot
Le style HubSpot embed ne se plie pas facilement à du CSS custom. 3 options :
1. **Laisser le style HubSpot par défaut** (rapide, fonctionnel, pas exactement dans la charte Texaa)
2. **CSS custom via HubSpot** : dans l'éditeur du formulaire HubSpot, onglet "Style & preview", ajouter du CSS personnalisé (les classes HubSpot sont documentées)
3. **Override en CSS global** dans `Layout.astro` : cibler `.hs-form-frame input`, `.hs-form-frame select`, etc. pour appliquer les mêmes tokens design que ceux conservés dans ce backup

→ Recommandation : option 3 (CSS global qui match la charte) car ça garde le rendu cohérent avec le reste du site sans toucher au contenu HubSpot.

### d) reCAPTCHA v3
À activer dans les options du formulaire HubSpot (cf. `AXEPTIO-INTEGRATION.md` § sur reCAPTCHA). Gratuit, invisible, géré 100 % côté HubSpot.

---

## 5. Si besoin de rollback

1. Restaurer le HTML du § 2 à l'emplacement actuel de l'embed HubSpot dans `index.astro`
2. Les CSS (§ 3) sont **toujours présents** dans `Layout.astro` pour l'instant (non supprimés lors de la migration) — à nettoyer plus tard uniquement quand on aura confirmé que HubSpot fonctionne en prod
