# Documentation développeur — Draft League Builder

## Stack technique

Une seule page HTML autonome (`index.html`) :
HTML + CSS + JavaScript vanilla. Aucun framework, aucune étape de build,
aucune dépendance npm à installer.

Dépendances externes chargées au runtime via CDN / API publique :

| Ressource | Usage |
|---|---|
| Google Fonts (`Bebas Neue`, `Inter`, `JetBrains Mono`) | Typographie |
| `html2canvas` (cdnjs) | Export de la fiche en PNG |
| `pokeapi.co` (API REST publique, sans clé) | Données et artworks des Pokémon |

## ⚠️ Important : environnement d'exécution

Ce fichier est un vrai site web statique : il doit tourner dans un **vrai
onglet de navigateur**, avec un accès réseau complet — ce qui est
exactement ce que fournit GitHub Pages.

Si ce fichier est ouvert dans un environnement qui **sandboxe les requêtes
sortantes** (par exemple l'aperçu d'artefact intégré à Claude.ai, ou tout
autre iframe avec une Content-Security-Policy restrictive), les appels
`fetch()` vers `pokeapi.co` seront bloqués par le navigateur et échoueront
avec une erreur réseau générique. **Ce n'est pas un bug de l'application** —
c'est une restriction volontaire de sécurité de ce type d'environnement,
qui ne s'applique pas au site publié ni au fichier ouvert en local.

## Structure du dépôt

```
.
├── index.html              # l'application entière (servie par GitHub Pages)
├── README.md               # documentation utilisateur / page d'accueil du dépôt
├── DEVELOPER.md            # ce fichier
└── GUIDE-PUBLICATION.md    # comment publier et mettre à jour sur GitHub Pages
```

GitHub Pages sert `index.html` à la racine de l'URL du site. Toute
modification poussée sur la branche principale est automatiquement
redéployée en une à deux minutes.

Un numéro de version est affiché en bas de page (élément `.app-version`
dans `index.html`, hors de la zone exportée en PNG). Pense à l'incrémenter
à chaque mise à jour publiée : c'est le moyen le plus simple de vérifier
qu'un utilisateur est bien sur la dernière version (Ctrl+F5 sinon).

## Logique clé du code

- **`normalizeName(raw)`** — met en minuscule, translittère les accents
  (`Flabébé` → `flabebe`), gère `♀`/`♂` (`Nidoran♀` → `nidoran-f`), retire
  `. : '`, remplace les espaces par des tirets. Sert à faire correspondre la
  saisie utilisateur au format des slugs PokeAPI (`mr-mime`, `tapu-koko`,
  `type-null`…).

- **`fetchPokemon(rawName)`** — appelle
  `GET https://pokeapi.co/api/v2/pokemon/{slug}`. En cas de 404, tente un
  **fallback via l'endpoint espèce** (`/pokemon-species/{slug}`) : certains
  Pokémon à formes multiples n'existent dans `/pokemon/` que sous le nom de
  leur forme par défaut (ex. `mimikyu` → `mimikyu-disguised`,
  `aegislash` → `aegislash-shield`, `giratina` → `giratina-altered`).
  L'espèce liste ses `varieties` ; on prend celle marquée `is_default` et on
  la charge. Par ailleurs, certaines formes existent dans la base mais n'ont
  **aucune image** (tous les champs de sprites sont `null`) : c'est le cas
  des formes purement utilitaires comme les modes de monture de Miraidon
  (`miraidon-low-power-mode`, `-drive-mode`, `-aquatic-mode`, `-glide-mode`)
  ou les builds de Koraidon. Dans ce cas, l'app retombe automatiquement sur
  l'artwork de la forme par défaut de l'espèce et affiche un toast informatif.
  Trois échecs distincts pour des messages ciblés :
  - `not-found` → 404 sur les deux endpoints (nom probablement mal orthographié)
  - `network` → requête bloquée ou impossible (voir encadré ci-dessus)
  - `no-image` → aucune image exploitable, même après repli sur la forme par défaut

- **Autocomplétion (`ensureAllNames`)** — au premier clic sur un hexagone,
  la liste complète des noms est chargée une seule fois via
  `GET /pokemon?limit=20000` (~1300 entrées, quelques dizaines de Ko) puis
  mise en cache pour la session. À chaque frappe (dès 2 caractères), le
  champ propose jusqu'à 8 suggestions : correspondances par **préfixe**
  d'abord, puis par **sous-chaîne**, après passage par `normalizeName` (donc
  insensible à la casse et aux accents). Navigation clavier ↑/↓ + Entrée ;
  la sélection à la souris utilise `mousedown` (et non `click`) pour se
  déclencher avant le `blur` de l'input qui ferme l'éditeur. En cas d'échec
  du chargement de la liste (hors ligne, sandbox), la promesse en cache est
  invalidée pour permettre une nouvelle tentative, et la saisie manuelle
  reste fonctionnelle.

- **`createTeamSlot(sizeClass, tagText)`** / **`createProfileSlot(sizeClass)`**
  — fabriques génériques (retournent un élément DOM autonome avec ses propres
  écouteurs d'événements) réutilisées pour :
  - les 10 hexagones de la fiche solo
  - les 12 hexagones du mode Face-à-face (2 joueurs × 6)
  - les 3 hexagones « photo » (1 en solo + 1 par joueur en Face-à-face)

- **Géométrie des hexagones** — entièrement gérée en CSS via des variables
  personnalisées (`--hex-w-sm`, `--hex-w-lg`, `--hex-w-md`, `--hex-w-xs`, et
  leurs équivalents `--hex-h-*`) et `clip-path: polygon(...)`. L'effet nid
  d'abeille vient d'un décalage calculé en CSS (`.hex-row--offset`,
  `.hex-row--offset-xs`), pas en JavaScript.

- **`TYPE_COLORS`** — table des 18 couleurs de type Pokémon, utilisée pour
  teinter dynamiquement le contour de l'hexagone (`--ring-color`) une fois un
  Pokémon chargé.

- **Export PNG** —
  `html2canvas(document.getElementById('exportCard'), { useCORS: true, scale: 2 })`.
  `useCORS` gère seul le CORS nécessaire pour capturer les sprites
  cross-origin ; les balises `<img>` visibles n'ont volontairement **pas**
  l'attribut `crossOrigin`, pour ne jamais risquer de casser leur simple
  affichage (voir historique : un `crossOrigin="anonymous"` posé trop tôt
  provoquait des hexagones vides).

- Seul `#exportCard` est capturé : la barre d'outils (bascule de mode,
  réinitialiser, exporter) est en dehors de cette zone et n'apparaît donc
  jamais dans le PNG exporté. Le mode actuellement inactif (`display:none`)
  est ignoré automatiquement par `html2canvas`.

## Modifier les couleurs / polices

Tout est piloté par les variables CSS en tête de fichier
(`:root { --c-ink, --c-gold, --c-panel, ... }`) et les imports de police en
haut du `<head>`. Aucune recompilation n'est nécessaire : éditer le HTML et
recharger la page suffit.

## Idées d'évolution

- Réorganisation des Pokémon par glisser-déposer entre hexagones
- Sauvegarde locale (`localStorage`/`indexedDB`) — possible uniquement une
  fois le fichier ouvert hors d'un environnement sandboxé
- Historique multi-semaines pour le mode Face-à-face
- Choix entre plusieurs sprites (shiny, formes régionales) pour un même
  Pokémon
