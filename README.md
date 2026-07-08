# RdV Draft League Builder

Application web pour créer les visuels hexagonaux d'une Draft League
**Pokémon Champions** : fiche de draft d'un joueur (photo + 10 picks) et
face-à-face hebdomadaire entre deux joueurs (photo + 6 Pokémon chacun),
exportables en PNG haute résolution pour les vidéos.

## 🎮 Utiliser l'application

**➡️ Ouvre simplement cette page :** `https://vanilla0731.github.io/RdV-Draft-League-Asset-Builder/`

Aucune installation. Fonctionne sur ordinateur, tablette et mobile.
Une connexion internet est nécessaire (les images des Pokémon sont
récupérées en direct depuis [PokéAPI](https://pokeapi.co)).

## Mode d'emploi

- **Titre / sous-titre** : clique directement sur le texte pour le modifier.
- **Hexagone doré** : clique dessus pour choisir une photo depuis ton appareil.
  Elle est automatiquement centrée et ajustée (jamais déformée).
- **Hexagones Pokémon** : clique, commence à taper le nom anglais du Pokémon,
  et choisis dans la liste de suggestions (↑ ↓ + Entrée, ou clic).
  Le contour de l'hexagone prend la couleur du type du Pokémon.
  Les Pokémon à formes multiples (Mimikyu, Aegislash…) sont reconnus
  par leur nom de base, et les formes sans image (modes de monture de
  Miraidon/Koraidon) affichent automatiquement la forme classique.
- **✕** au survol d'un hexagone rempli : le vider.
- **Draft Sheet / Encounter** : bascule entre la fiche solo et le
  face-à-face 2 joueurs.
- **Export PNG** : télécharge une image haute résolution de la fiche affichée.

⚠️ Rien n'est sauvegardé entre deux visites pour l'instant : exporte ton
visuel en PNG avant de fermer l'onglet.

## Pour les développeurs

Le projet tient dans un seul fichier (`index.html`) : HTML, CSS et
JavaScript vanilla, sans build ni dépendance à installer.
Voir [DEVELOPER.md](DEVELOPER.md) pour l'architecture et les choix techniques.

## Crédits

- Données et artworks Pokémon : [PokéAPI](https://pokeapi.co) (API publique gratuite)
- Export d'image : [html2canvas](https://html2canvas.hertzen.com/)
- Pokémon est une marque de Nintendo / Creatures Inc. / GAME FREAK inc.
  Projet de fan non affilié, non commercial.
