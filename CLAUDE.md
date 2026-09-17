# Coolcook — App de recettes et de planification de repas

PWA de recettes de cuisine (projet nom de code "Flemmequiche" pour son premier jeu de données :
recettes one-pot/quasi one-pot, peu de vaisselle, 30 minutes max de préparation) construite à
partir de septembre 2026. Ce fichier sert de mémoire portable entre les machines sur lesquelles
le propriétaire travaille, en complément du dépôt Git qui est la seule source de vérité partagée.

## Stack technique

- Fichier unique `index.html` : HTML/CSS/JS vanilla, **pas de framework, pas de build step** —
  même approche que [Cantrip](https://github.com/benwatz/cantrip), projet JDR du même auteur, dont
  ce projet reprend le squelette PWA de départ.
- **Pas de Tailwind** : le CSS est le design system "Classical" adapté, réimplémenté à la main
  dans le `<style>` de `index.html` (tokens en custom properties `--color-*`/`--space-*`/
  `--radius-*`/`--shadow-*` puis classes composants `.btn`, `.tag`, `.field`/`.input`,
  `.seg`/`.seg-opt`, `.card`, `.hr`, `.text-muted`, `.elev-sm`). **Toute nouvelle couleur doit
  utiliser `var(--...)`** plutôt qu'un hex en dur.
- Police **Poppins** (Google Fonts) pour les titres comme pour le texte, poids 700 pour tous les
  titres/boutons/libellés de card (remplace la paire Cormorant Garamond/Lora du design system
  d'origine, et le Cinzel du squelette PWA initial).
- Persistance : `localStorage`, clé `coolcook_state`. **Ne contient que `convives`** (le nombre de
  convives par défaut, partagé entre l'accueil et Paramètres) — tout le reste de l'état est
  éphémère (objet `ui`, non persisté). Pas de backend.
- PWA : `manifest.json` + `icon.svg` + `sw.js` (service worker, stratégie réseau d'abord avec
  fallback cache, `CACHE_NAME` versionné `coolcook-vN` — **à incrémenter à chaque changement
  significatif des assets statiques**, comme dans Cantrip). `flemmequiche-recettes-v2.json` fait
  partie des `CORE_ASSETS` pour que l'app fonctionne hors ligne.
- `.claude/launch.json` : configuration de serveur statique local (`npx http-server`, port 4173)
  pour prévisualiser l'app. **Un simple `file://` ne suffit pas** : le chargement des recettes
  passe par `fetch()`, bloqué par CORS sur `file://`.

## Structure du code (`index.html`)

Application à état unique en IIFE (`(function () { 'use strict'; ... })()`), rendu par réécriture
complète de `innerHTML` à chaque changement (pas de diffing, pas de framework) — même pattern que
Cantrip.

- `state` : données persistées (uniquement `convives`). Sauvegardé via `saveState()`.
- `ui` : état éphémère non persisté — `screen` (`'home'|'list'|'detail'|'settings'`), `tab`
  (onglet mémorisé pour le retour depuis le détail), `recipes` (chargées depuis le JSON),
  `loading`/`loadError`, filtres d'accueil (`filterCategory`/`filterValue`), `suggestions`/
  `suggestionMessage`, `selectedRecipeId`/`detailPortions`, filtres de liste (`listQuery`/
  `listUstensile`/`listTemps`).
- `render()` : réécrit l'en-tête + le contenu (`renderHome()`/`renderList()`/`renderDetail()`/
  `renderSettings()` selon `ui.screen`) + `renderTabBar()`. **Restaure le focus et la position du
  curseur** de l'élément actif avant re-render (repéré par son `id`) : sans ça, le champ de
  recherche perdrait le focus à chaque caractère tapé, puisque tout le DOM est recréé.
- `bindEvents()` : attaché **une seule fois** au démarrage (délégation sur `#app` via
  `data-action`), contrairement à Cantrip qui ré-attache après chaque rendu. Trois listeners :
  `click` (boutons et cards), `change` (radios des contrôles segmentés, `<select>` de filtre) et
  `input` (champ de recherche uniquement).
- Les recettes sont chargées par `fetch()` depuis `flemmequiche-recettes-v2.json` — **source unique
  de vérité**, jamais dupliquées en dur dans `index.html`.

## Écrans

Quatre écrans, barre d'onglets basse à 3 entrées (Accueil / Recettes / Paramètres) ; le détail
recette est *poussé par-dessus* l'onglet courant et le bouton "← Retour" revient à `ui.tab`.

1. **Accueil** — stepper "Nombre de convives" (bornes 1–12), filtre facultatif par catégorie
   (contrôle segmenté Aucun/Légume/Féculent/Protéine, puis `<select>` des valeurs uniques tirées
   des recettes), bouton "Valider" qui tire 3 recettes au hasard dans le pool filtré, bouton
   "Relancer" qui retire 3 nouvelles recettes. Si le pool contient moins de 3 recettes, affiche
   tout ce qui existe + un message. Les cards affichent les quantités **recalculées pour le nombre
   de convives** dans les tags (féculent/protéine/légume principal).
2. **Recettes** — recherche texte (titre + ingrédients + catégorisation), contrôle segmenté
   ustensile (dérivé des données) et temps (Tous / ≤15 / ≤20 / ≤25 min), **filtres cumulables** en
   temps réel, compteur "{n} recette(s)" + bouton "Réinitialiser" visible seulement si un filtre
   est actif, message dédié si 0 résultat.
3. **Détail recette** — titre, tags temps/ustensile, stepper "Portions" qui recalcule toutes les
   quantités (`quantité × portions_affichées / portions_recette`, arrondi à 1 décimale, entier si
   rond — `fmtQty()`), liste d'ingrédients puis étapes numérotées. Les portions initiales valent
   le nombre de convives si on vient de l'accueil, les portions de la recette si on vient de la
   liste.
4. **Paramètres** — stepper "Convives par défaut" (même valeur que l'accueil, état partagé et
   persisté) ; le reste est à venir.

## Décisions de conception

- **Favoris hors scope** pour cette version (explicitement écarté dans le handoff de design) — ne
  pas les réintroduire sans en rediscuter.
- `mainLegume()` ignore volontairement `oignon` et `ail` pour choisir le légume "principal" affiché
  sur les cards de l'accueil (ce sont des aromates présents dans presque toutes les recettes, peu
  informatifs comme étiquette).
- Le tirage des suggestions utilise un mélange de Fisher-Yates (`shuffled()`) plutôt qu'un
  `sort(() => Math.random() - 0.5)`, qui est biaisé.

## Données et spécifications existantes

- `flemmequiche-recettes-v2.json` — pool de recettes (v2.0, dernière maj 17/09/2026). Structure :
  `meta` (description du projet, principes de conception PD-01/02/03, historique des révisions
  d'unités) puis `recettes[]`, chaque recette ayant `id`, `titre`, `temps_preparation_minutes`,
  `ustensiles[]`, `portions`, `categorisation` (`feculent`/`proteine`/`legumes[]`),
  `ingredients[]` (`nom`/`quantite`/`unite`) et `etapes[]`. Unités harmonisées en v2.0 : `pièce`,
  `cuillère à soupe`, `cuillère à café` (accentuation française correcte, alignée sur le
  référentiel d'ingrédients ci-dessous).
- Documents Google Docs (non versionnés, `.gdoc`, cloud-only — voir `.gitignore`) dans
  `G:\Mon Drive\Cuisine\Coolcook\` :
  - `App cuisine - specs v1.3.gdoc` — cahier des charges fonctionnel courant, source de vérité
    pour le référentiel d'unités de mesure entre autres.
  - `App cuisine - plan de developpement.gdoc` — plan de développement.
  - `Flemmequiche - Liste des ingrédients et fonctionnalités V6.gdoc` — liste d'ingrédients et
    fonctionnalités.
  Ces trois documents sont à consulter avant toute évolution fonctionnelle : ils ne sont pas
  dupliqués ici pour éviter la désynchronisation, ce fichier ne fait que pointer vers eux.
- **Handoff de design Claude Design** (livré en zip le 17/09/2026, non versionné) : prototype
  `Coolcook.dc.html` + `styles.css` (design system "Classical") + `README.md` de handoff, source
  de l'UI décrite dans "Écrans" ci-dessus. Les prototypes `.dc.html` tournent sur un runtime
  propriétaire et **ne sont pas copiables tels quels** : l'UI a été réimplémentée en vanilla dans
  `index.html`. Le `recipes-data.js` du handoff est une transcription **identique** de
  `flemmequiche-recettes-v2.json` — c'est le JSON du dépôt qui reste la source unique.

## Déploiement

- Dépôt : https://github.com/benwatz/coolcook (public, compte GitHub "benwatz")
- **Prod** (GitHub Pages, HTTPS, installable en PWA) : https://benwatz.github.io/coolcook/ —
  déploiement automatique à chaque push sur `master`, qui sert directement `index.html` à la
  racine (pas de pipeline CI, pas de dossier `dist`). `.nojekyll` présent pour désactiver le
  traitement Jekyll.
- **Pas de preprod** pour ce projet (décision explicite — contrairement à Cantrip qui a un
  environnement Netlify séparé).

## Git — spécifique à cette machine, à reconfigurer sur toute nouvelle installation

L'identité Git de ce dépôt est configurée **localement** (pas globalement), pour ne pas exposer
l'email réel dans l'historique public :
```
git config user.name "benwatz"
git config user.email "124379495+benwatz@users.noreply.github.com"
```
Cette config est dans `.git/config`, donc **pas transmise par un `git clone`** — à relancer sur
toute nouvelle machine avant de committer.

## Points d'attention

- Le dossier du projet vit dans Google Drive (`G:\Mon Drive\Cuisine\Coolcook`), synchronisé en
  parallèle de Git — même configuration que Cantrip. Attention aux conflits si le dossier est
  modifié sur deux machines sans avoir push/pull Git entre les deux (la sync Drive et Git peuvent
  se marcher dessus).
- Les fichiers `.gdoc` sont des pointeurs Google Docs cloud-only : les indexer avec `git add`
  fait échouer la commande (erreur de lecture). Ils sont exclus via `.gitignore` (`*.gdoc`) —
  ne pas les retirer de cette exclusion.
