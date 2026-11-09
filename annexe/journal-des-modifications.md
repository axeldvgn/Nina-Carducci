# Journal des modifications – Nina Carducci

Liste de toutes les modifications faites sur le site, pour rédiger le rapport d'optimisation.
À tenir à jour à chaque nouvelle modification.

## Historique git

| Commit | Titre | Contenu |
|---|---|---|
| `b1a91ce` | Ajout des fichiers | Version d'origine du site (sert pour les audits « avant ») |
| `47836d8` | Correction W3C | Correction des erreurs du validateur W3C dans `index.html` |
| `d803917` | Opti des images et du portfolio | Images en WebP, filtres du portfolio, `defer` sur les scripts |
| *(non commité)* | | Ordre des scripts (jQuery avant Bootstrap) |

---

## 1. Performances

### 1.1 Images (commit `d803917`)

**Avant :** 15 images, 30,9 MB au total. Photos de plus de 4000 px de large, en JPG et en PNG.

**Modifications :**
- Conversion des 14 photos JPG et PNG au format **WebP**. L'icône `instagram.png` (721 octets) reste en PNG.
- Redimensionnement à la taille d'affichage réelle :
  - carrousel (`assets/images/slider/`) : 1920 px de large ;
  - galerie (`assets/images/gallery/`) : 800 px ;
  - `nina` et `camera` : 800 px.
- Compression en qualité 75, sans perte visible.
- Mise à jour des `src` dans `index.html` (`.jpg` / `.png` remplacés par `.webp`).

**Après :** 0,82 MB au total, soit **97,3 % de gain**.

| Groupe | Avant | Après |
|---|---|---|
| `slider/` (3 images) | 9,2 MB | 425 Ko |
| `gallery/` (9 images) | 17,8 MB | 323 Ko |
| `nina` + `camera` | 3,8 MB | 70 Ko |

Détail par image :

| Image | Avant | Après | Dimensions |
|---|---|---|---|
| slider/edward-cisneros | 5,70 MB | 171 Ko | 1920×887 |
| slider/nicholas-green | 1,90 MB | 83 Ko | 1920×888 |
| slider/ryoji-iwata | 1,62 MB | 171 Ko | 1920×888 |
| concerts/aaron-paul | 1,04 MB | 80 Ko | 800×1067 |
| concerts/austin-neill | 1,47 MB | 20 Ko | 800×533 |
| entreprise/ali-morshedlou | 1,10 MB | 14 Ko | 800×1200 |
| entreprise/jason-goodman | 0,71 MB | 16 Ko | 800×541 |
| entreprise/mateus-campos-felipe | 1,92 MB | 26 Ko | 800×490 |
| mariage/hannah-busing | 1,78 MB | 13 Ko | 800×640 |
| mariage/jakob-owens | 6,28 MB | 57 Ko | 800×1200 |
| portraits/ade-tunji | 1,00 MB | 53 Ko | 800×1199 |
| portraits/nino-van-prattenburg | 2,53 MB | 44 Ko | 800×1202 |
| nina | 2,16 MB | 48 Ko | 800×798 |
| camera | 1,66 MB | 22 Ko | 800×798 |

### 1.2 Chargement des scripts

**Commit `d803917` :** ajout de l'attribut `defer` sur les 4 scripts du `<head>` (Bootstrap, jQuery, `maugallery.js`, `scripts.js`).
- **Pourquoi :** sans `defer`, chaque script bloque l'affichage de la page pendant son téléchargement (Lighthouse : « ressources qui bloquent le rendu »).
- Avec `defer`, les scripts se téléchargent en parallèle et s'exécutent après la lecture du HTML, dans leur ordre d'écriture.

**Non commité :** jQuery déplacé **avant** Bootstrap dans `index.html`.
- **Bug corrigé :** avec `defer`, Bootstrap s'exécutait avant jQuery. Il ne pouvait donc pas ajouter `.modal()` à jQuery. Au clic sur une photo, rien ne s'ouvrait, et la console affichait :
  `Uncaught TypeError: $(...).modal is not a function` (`maugallery.js:120`).
- **Ordre final :** jQuery → Bootstrap → `maugallery.js` → `scripts.js`.

---

## 2. Bugs JavaScript (`assets/maugallery.js`)

### 2.1 Filtres du portfolio – `filterByTag()` (commit `d803917`) – Résolu

**Problèmes :**
1. Le bouton de filtre cliqué ne prenait pas la couleur de sélection. Seule la classe `active-tag` lui était ajoutée, alors que la classe `active` lui était retirée.
2. À chaque clic, toutes les photos étaient cachées (`hide()`), puis réaffichées avec une animation (`show(300)`). Effet visuel de rechargement.

**Corrections :**
- `$(this).addClass("active-tag")` remplacé par `$(this).addClass("active active-tag")`.
- Suppression du `hide()` qui cachait toutes les photos au début de la boucle.
- Chaque photo est maintenant **soit affichée, soit cachée**, en une seule condition avec `||` :
  `if ($(this).data("gallery-tag") === tag || tag === "all") { show() } else { hide() }`
- Suppression des animations (`show(300)` remplacé par `show()`).

### 2.2 Flèches de la lightbox – `prevImage()` / `nextImage()` – À faire

**Problème :** au clic sur précédent ou suivant, l'image ne change pas. La boucle trouve la position (`index`) de l'image affichée, puis le code réaffiche `imagesCollection[index]`, c'est-à-dire la même image.

**Correction prévue (pas encore enregistrée dans le fichier) :**
- `prevImage()` :
  `next = imagesCollection[index - 1] || imagesCollection[imagesCollection.length - 1];`
- `nextImage()` :
  `next = imagesCollection[index + 1] || imagesCollection[0];`
- **Bords de la liste :** sur la 1re image, `index - 1` donne `undefined`, donc le `||` prend la dernière image. Sur la dernière image, `index + 1` donne `undefined`, donc le `||` prend la première.

---

## 3. Accessibilité

- `alt` de l'icône Instagram : `""` remplacé par `"instagram"` (commit `d803917`). À améliorer.

---

## 4. Cahier de recette

| ID | Action | Résultat initial | Résultat attendu | Statut |
|---|---|---|---|---|
| 1 | Cliquer sur un filtre de catégorie | Bouton non surligné, toutes les photos disparaissent puis réapparaissent | Bouton surligné, seules les photos de la catégorie s'affichent | Résolu |
| 2 | Cliquer sur le filtre « Tous » | – | Toutes les photos s'affichent | Résolu |
| 3 | Cliquer sur une photo | La lightbox s'ouvre | La lightbox s'ouvre | Résolu (régression due au `defer`, corrigée par l'ordre des scripts) |
| 4 | Flèches précédent / suivant de la lightbox | L'image ne change pas | Image précédente / suivante de la catégorie active | À faire |

---

## 5. Reste à faire

- **Performances :**
  - fichiers minifiés (`bootstrap.min.css`, `bootstrap.bundle.min.js`, CSS et JS du site) ;
  - `width` / `height` et `loading="lazy"` sur les images ;
  - Google Fonts réduit aux variantes utilisées.
- **SEO :**
  - meta description et `<title>` plus précis ;
  - un seul `<h1>` (3 actuellement) ;
  - balises sémantiques ;
  - `alt` descriptifs.
- **Schema.org :** bloc JSON-LD de l'activité locale.
- **Réseaux sociaux :** balises Open Graph et Twitter Card.
- **Accessibilité :**
  - `alt` du carrousel (`"..."`) et `alt` génériques ;
  - `<label for>` sur le formulaire ;
  - `Previous` / `Next` en français ;
  - flèches et filtres utilisables au clavier ;
  - contraste à vérifier.
- **Mesures :** Lighthouse avant (commit `b1a91ce`) et après, WAVE avant et après.
- **Rapport :** `annexe/Rapport-intervention-Nina-Carducci.odt` à compléter.
