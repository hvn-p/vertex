# Conception de Vertex

Document temporaire. Chaque décision le quitte dès qu'elle passe dans une issue ou dans
le code ; on le supprime quand il est vide. Les termes du jeu sont en anglais, définis
dans le glossaire du `CLAUDE.md`.

## Le jeu

Un Survivor : une run, des vagues d'ennemis, une montée de niveau qui propose des choix
tirés au hasard. Ce qui le distingue : le joueur construit son sort pièce par pièce, et
l'ordre des pièces change ce que fait le sort. Le catalogue reste court, les
combinaisons explosent.

Rendu 2D dans un canvas, sans 3D. Coop jusqu'à 4 joueurs, solo possible.

## Le sort

- Un **Spell** est une suite ordonnée de **Sequences**. L'ordre est libre.
- Types de séquence connus : `Form` (crée quelque chose : projectile, rayon, zone) et
  `Impact` (ce qui se passe au contact ou à la fin).
- Chaque séquence a des prises où se branchent des **Modifiers** : ciblage, trajectoire,
  taille, vitesse, élément. La trajectoire est un modifieur, pas une séquence.
- Une **Piece** est ce qu'on trouve : une séquence ou un modifieur. Chaque pièce se
  branche sur un type de prise.
- Chaque séquence agit sur ce qui existe à son point de la suite : au départ le joueur,
  ensuite ce que les séquences précédentes ont créé. Le joueur édite une liste, le jeu
  l'exécute comme un arbre.
- Toute séquence a un sens à toute position. Aucun ordre n'est invalide.

Exemple, mêmes pièces, deux sorts :

- `Form: projectile → Impact: burst 8` : un projectile qui éclate en 8 au contact.
- `Impact: burst 8 → Form: projectile` : 8 éclats autour du joueur, chacun tire un
  projectile.

L'éditeur de sort est le cœur du jeu : le joueur doit voir ce que fait sa séquence.

## Découverte et plans

- Les pièces découvertes entrent dans une bibliothèque du profil. Celles qui restent à
  découvrir y apparaissent cachées (fond noir, point d'interrogation).
- Un **Blueprint** est un modèle de sort sauvegardé dans le profil. On peut en avoir
  plusieurs. Il ne contient que des pièces déjà découvertes. La première run se fait
  sans plan.
- Partir avec un plan se débloque, c'est une amélioration du profil. Une fois débloqué,
  on part d'un plan issu d'une run passée (et on peut le modifier), ou d'un plan neuf
  construit avec les pièces découvertes.
- Pendant la run, il faut quand même trouver les pièces. Le plan est un modèle, pas un
  moule : une pièce nouvelle peut le faire dévier.
- Le tirage ne favorise pas les pièces du plan. Une amélioration future du profil
  pourra le faire, c'est un mécanisme distinct.
- Un plan ne se teste pas hors run.
- Le sort final de chaque run peut se garder pour être montré (grimoire). Proposé, pas
  encore validé.

## Ce que le style impose

Le jeu s'appuie sur le style Heptapode (`~/dotfiles/styles/heptapode`) et sert à le
faire évoluer. Ce qu'il décide pour le style remonte dans la fiche de la bibliothèque.

- Un univers sérieux et sombre, pas d'arcade : pas de formes géométriques connues
  (étoile, croix, engrenage), pas de créatures dessinées. Le fond est presque noir.
- Aucun dégradé sur les entités. Une entité est faite de petites formes qui ont chacune
  leur opacité ; c'est l'ensemble qui donne l'effet. La lumière fait exception. Le
  principe est celui de `water.tsx` dans le portfolio.
- Directions retenues, d'après le spike `spikes/render/heptapode.html` :
  - **logogram** : un anneau d'encre brisé, d'épaisseur irrégulière, avec des coulures.
    Pour des ennemis et pour des attaques.
  - **presence** : des membres inégaux qui sortent d'un centre qu'on ne voit jamais,
    éclairés seulement au bout. Pour des ennemis.
  - **shard** : un éclat debout, plus noir que l'air, dont seule l'arête tournée vers la
    lumière s'allume. Pour des ennemis ; c'est la seule direction qui tient déjà en
    petite taille.
  - **cloud** : une nuée autour d'un point de lumière. Plutôt pour des attaques, ou
    l'attaque spéciale d'un ennemi (une zone dont les dégâts montent vers le centre).
- Écartés : `signal` (ne se lit pas comme un ennemi), `slit`, et les rendus de formes
  géométriques des spikes `looks.html` et `shapes.html` (trop arcade).
- Plus une entité est petite, moins elle a de détails : chaque taille se dessine pour
  elle-même, une petite entité n'est pas une grande réduite. Un grain ne descend jamais
  sous 2 px : une petite entité perd des détails, pas de l'épaisseur.
- Trois tailles, validées sur maquette (`spikes/render/sizes.html`) : petite (28 px,
  en groupes), normale (56 px, la taille courante), grande (112 px).
- Ces maquettes donnent une direction, pas les ennemis du jeu : chaque ennemi aura son
  but et ses attaques spéciales, à concevoir. La présence sera sans doute le premier.
- Les cosmétiques ne touchent que le rendu : ils changent la façon dont une forme se
  dessine, jamais ce qu'elle fait.

## Architecture

- La simulation et le rendu sont séparés. Le multijoueur ne partage que la simulation,
  les cosmétiques ne vivent que dans le rendu, et les petites formes d'une entité ne
  sont jamais simulées : le rendu les calcule à partir de l'état de l'entité.
- Pour le multijoueur, dès le premier prototype : une simulation qui avance par pas de temps fixes,
  un générateur aléatoire à graine pour tout tirage, le sort décrit comme une donnée
  (il passe par le réseau et se stocke dans le profil), et le joueur n'agit que par ses
  entrées.
- Performances : on optimise avant de plafonner. Une limite ne se pose que là où
  l'optimisation ne suit plus, jamais avant.
- Rendu en WebGL2, les petites formes calculées par le GPU. Mesuré par le spike
  `spike/render` le 10/10/2026 (Radeon 890M intégrée, fenêtre 1600×900, 60 images par
  seconde tenues) :

  | Rendu | Petites formes tenues |
  |---|---|
  | Canvas 2D, un appel par forme | 10 000 |
  | Canvas 2D, regroupé par opacité | 61 000 |
  | WebGL2, formes calculées en JavaScript | 150 000 |
  | WebGL2, formes calculées par le GPU | 1 200 000 |

  Au-delà, c'est la simulation du spike (écrite sans optimisation) qui cède, plus le
  rendu. Une seule machine mesurée.

## À trancher

- Les autres types de séquence (`Trigger` ?).
- Le moment où l'on édite son sort en coop, où la partie ne peut pas se mettre en pause
  pour un seul joueur.
- La combinaison des éléments : table écrite à la main (feu + glace = un nouvel élément)
  ou réactions entre états posés sur l'ennemi.
- Le modèle réseau : chaque client simule la même partie, ou un serveur fait foi.
- Le contenu de la progression du profil, au-delà des pièces et des plans.
- La difficulté : niveaux, types d'ennemis.
- L'avatar : garder le losange envisagé au départ, ou le dessiner dans les directions
  retenues.
- La mort de chaque direction : celles du spike n'ont pas encore été validées.
- Le compte, les groupes, l'invitation par lien, plus tard le matchmaking.
- Le nom du jeu publié (`vertex` est un nom de code).
