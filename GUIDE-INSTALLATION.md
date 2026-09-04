# Boutique BO Escrime — Guide d'installation

Ce dossier contient 2 pages autonomes :
- **index.html** → boutique en ligne (front office, côté client)
- **admin.html** → back office (gestion commandes / fournisseurs / bons de commande), protégé par le mot de passe **Boutique**

Les deux pages sont **responsive** (utilisables sur smartphone comme sur PC).

---

## 1. Base de données : rien à configurer

`index.html` et `admin.html` utilisent la même base Firebase que le reste du club
(`bo-escrime-default-rtdb`), en accès direct via l'API REST — exactement comme le fait déjà
`galerie-bo-escrime`. Aucune clé API, aucun `firebaseConfig` à renseigner : les fichiers fonctionnent
tels quels.

Les données de la boutique sont stockées sous un nouveau nœud `/boutique` dans la même base
(`boutique/commandes`, `boutique/fournisseurs`, `boutique/bonsCommande`), séparé de la galerie, de la
buvette et des essais gratuits.

⚠️ Comme pour la galerie, l'admin (`admin.html`) n'a de "temps réel" qu'approximatif : les données se
rafraîchissent automatiquement toutes les 20 secondes, ou immédiatement via le bouton "🔄 Actualiser"
en haut à droite.

---

## 2. Étape obligatoire : configurer UN SEUL template EmailJS

Votre compte EmailJS est limité à **2 templates au total**, et le premier (`ACK_RESA` /
`template_jj3ultb`) est déjà utilisé par l'essai gratuit — il ne faut pas y toucher.

La boutique a donc été conçue pour fonctionner avec le **second template existant, "Order
Confirmation" (`template_94141gr`)**, réutilisé comme **template générique unique** pour les 5 emails
du système (confirmation client, notification club, demande de modification, bon de commande
fournisseur, article prêt).

Le principe : ce n'est plus EmailJS qui met en forme l'email. C'est le code (`index.html` /
`admin.html`) qui fabrique lui-même le HTML complet de chaque email (logo, couleurs du club, tableau
des articles...) et qui l'envoie tel quel. Le template EmailJS ne fait que transmettre trois variables :
qui reçoit l'email, quel objet, et quel contenu.

### Configuration à faire une seule fois

Ouvrez le template **"Order Confirmation"** dans EmailJS, onglet **Settings**, et réglez :

| Champ | Valeur |
|---|---|
| To Email | `{{to_email}}` |
| Subject | `{{subject}}` |
| Reply To | `{{reply_to}}` |

Puis dans l'onglet **Content**, basculez en mode **"Code editor"** (icône `</>`) et remplacez tout le
contenu par simplement :

```html
{{{content}}}
```

⚠️ **Important** : bien mettre **trois** accolades de chaque côté (`{{{` et `}}}`), pas deux. EmailJS
utilise un moteur de type Mustache : `{{content}}` (2 accolades) échappe le HTML et l'affiche comme du
texte brut illisible, alors que `{{{content}}}` (3 accolades) l'insère tel quel, ce qui est ce qu'on veut
ici puisque `content` contient déjà du HTML complet généré par le site.

C'est tout. Le HTML complet (bandeau, tableau, couleurs...) est déjà généré par le site à chaque envoi
et transmis dans la variable `content` — vous n'avez rien d'autre à écrire dans EmailJS.

### Ce que chaque page envoie avec ce template

| Email | Déclenché depuis | Objet |
|---|---|---|
| Confirmation client | `index.html`, à la validation de commande | Confirmation de votre commande {numéro} |
| Notification club | `index.html`, à la validation de commande | Nouvelle commande boutique {numéro} |
| Demande modification/annulation | `index.html`, bouton "Suivre ma commande" | Demande modification/annulation — {numéro} |
| Bon de commande fournisseur | `admin.html`, onglet Bons de commande | Bon de commande {numéro} — BO Escrime |
| Article(s) prêt(s) | `admin.html`, onglet Réceptions | Votre commande {numéro} est disponible à la salle ! |

Si un jour votre compte EmailJS est mis à niveau (plus de templates disponibles), on pourra
éventuellement repasser à des templates dédiés — mais ce n'est plus nécessaire, le rendu est identique.

## 3. Déploiement (comme pour la galerie / le chrono)

1. Placez `index.html`, `admin.html` et le dossier `images/` dans votre dépôt GitHub
   (par ex. un nouveau repo `boutique-bo-escrime`, ou un sous-dossier `boutique/` du repo existant).
2. Activez GitHub Pages sur le repo.
3. Intégrez la page dans Hostinger :
   - soit via un lien direct depuis le menu du site (ex : bouton "Boutique") vers l'URL GitHub Pages,
   - soit via une iframe, comme pour la galerie.
4. Le back office (`admin.html`) n'a pas besoin d'être lié dans le menu public — gardez-en juste
   l'URL en interne (ex : `boescrime.github.io/boutique-bo-escrime/admin.html`).

---

## 4. Fonctionnement du back office

- **Mot de passe** : `Boutique` (protection simple côté navigateur, cohérente avec les autres outils du club).
- **Onglet Commandes** : liste de toutes les commandes, statut par ligne modifiable manuellement si besoin.
  Le statut global d'une commande = le statut le plus "en retard" parmi ses lignes.
- **Onglet Bons de commande** :
  1. Cochez les lignes "Commandé" à transmettre au fournisseur.
  2. Choisissez un fournisseur déjà enregistré, ou saisissez-en un nouveau (mémorisable, 10 max).
  3. "Générer le bon de commande" → regroupe automatiquement par article + taille.
  4. "Envoyer au fournisseur & valider" → envoie l'e-mail au fournisseur et passe les lignes concernées
     en "En attente fournisseur".
- **Onglet Réceptions** : quand la commande fournisseur arrive, cliquez "Marquer reçu" → les lignes
  passent en "À disposition à la salle" et chaque client concerné reçoit un e-mail automatique.
- **Onglet Fournisseurs** : gestion des fiches fournisseurs (10 maximum), réutilisables pour les
  prochains bons de commande.

---

## 5. Tarifs (modifiables dans le code si besoin)

| Article | Prix |
|---|---|
| T-shirt | 10 € |
| Polo | 35 € |
| Sweat (sans zip) | 40 € |
| Sweat zippé | 45 € |
| Supplément nom du tireur au dos | 6 € |

Ces prix sont définis dans `index.html`, tableau `PRODUCTS` (variable `SUPPLEMENT_NOM` pour le flocage).

---

## 6. Anti-robot

Un captcha "maison" (question mathématique simple, générée aléatoirement) protège le formulaire de
commande — aucune dépendance externe (pas de clé Google reCAPTCHA à gérer).
