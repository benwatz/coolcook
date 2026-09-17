# Coolcook — App de recettes et de planification de repas

PWA de recettes de cuisine (projet nom de code "Flemmequiche" pour son premier jeu de données :
recettes one-pot/quasi one-pot, peu de vaisselle, 30 minutes max de préparation) construite à
partir de septembre 2026. Ce fichier sert de mémoire portable entre les machines sur lesquelles
le propriétaire travaille, en complément du dépôt Git qui est la seule source de vérité partagée.

## Stack technique

- Fichier unique `index.html` : HTML/CSS/JS vanilla, **pas de framework, pas de build step** —
  même approche que [Cantrip](https://github.com/benwatz/cantrip), projet JDR du même auteur, dont
  ce projet reprend le squelette PWA de départ.
- Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com">`).
- Police Cinzel (Google Fonts) via `class="font-cinzel"` — reprise du scaffold initial, à
  reconsidérer si elle ne convient pas à l'identité visuelle de l'app cuisine.
- Persistance : `localStorage`, clé `coolcook_state`. Pas de backend pour l'instant.
- PWA : `manifest.json` + `icon.svg` + `sw.js` (service worker, stratégie réseau d'abord avec
  fallback cache, `CACHE_NAME` versionné `coolcook-vN` — **à incrémenter à chaque changement
  significatif des assets statiques**, comme dans Cantrip).

## État actuel

Squelette de départ uniquement (page d'accueil statique dans `index.html`) : aucune fonctionnalité
de recettes n'est encore implémentée dans l'app. Les données et la logique métier restent à
construire à partir des sources ci-dessous.

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
