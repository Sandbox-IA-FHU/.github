# Sandbox IA · IT Solutions

Bac à sable d'ingénierie IA du groupe Altonéo. On y apprend à construire des
systèmes à base de modèles de langage avec les mêmes exigences qu'en production :
une expérience qu'on peut rejouer, un résultat qu'on peut chiffrer, une donnée
qu'on ne fait pas fuiter.

Ce n'est pas un espace de bidouille. Tout ce qui est ici est destiné à être
relu, repris et défendu devant quelqu'un d'autre.

## Par où commencer

| Vous voulez… | Allez voir |
|---|---|
| Comprendre les règles du jeu | [Cycle de vie des dépôts](https://github.com/Sandbox-IA-FHU/.github/blob/main/CYCLE-DE-VIE.md) |
| Savoir comment on mesure un système IA | [Évaluation](https://github.com/Sandbox-IA-FHU/.github/blob/main/EVALUATION.md) |
| Contribuer : branches, commits, PR | [Guide de contribution](https://github.com/Sandbox-IA-FHU/.github/blob/main/CONTRIBUTING.md) |
| Manipuler de la donnée | [Règles données](https://github.com/Sandbox-IA-FHU/.github/blob/main/DONNEES.md) — **à lire avant le premier commit** |
| Signaler une clé qui a fuité | [Sécurité](https://github.com/Sandbox-IA-FHU/.github/blob/main/SECURITY.md) |
| Poser une question | Les Discussions de l'organisation |

## Les six principes

**1. Une expérience non reproductible n'est pas un résultat.**
Un run se rejoue ou n'existe pas. Il faut versionner ensemble : le prompt exact,
le modèle **et sa version figée**, les paramètres (température, max tokens), et
le jeu d'entrée. « J'ai essayé, ça marchait bien » n'est pas une conclusion.

**2. On définit la mesure avant de construire.**
Avant le premier prompt : sur quoi juge-t-on que ça marche, sur combien de cas,
et à partir de quel seuil ? Un système IA sans jeu d'évaluation ne peut pas être
amélioré — on ne fait que déplacer les erreurs sans le voir.

**3. Un chiffre sans date est un chiffre faux.**
Les prix, les performances et les limites des modèles bougent tous les mois.
Toute mesure, tout chiffrage, tout benchmark porte sa date. Sans date, on le
considère comme périmé.

**4. Le coût et la latence sont des critères de recette.**
Au même rang que la qualité. Une réponse parfaite à 12 secondes et 40 centimes
n'est pas une réponse acceptable. On mesure les trois ensemble, toujours.

**5. Aucune donnée réelle dans un dépôt.**
Ni fichier, ni extrait, ni sortie de notebook, ni ligne de log. Même anonymisée.
Voir [DONNEES.md](https://github.com/Sandbox-IA-FHU/.github/blob/main/DONNEES.md) — c'est la règle qui n'a aucune exception.

**6. Le modèle n'est pas la solution par défaut.**
Avant d'appeler un LLM : est-ce qu'une requête SQL, une règle métier ou une
expression régulière ne suffirait pas ? Un appel modèle coûte de l'argent,
ajoute de la latence et introduit du non-déterminisme. Il faut le mériter.

## Socle technique

Ce qui est **figé** — on ne discute pas, c'est le même partout :

| | |
|---|---|
| Langage | Python 3.13 |
| Dépendances | `uv` (jamais pip seul, jamais poetry, jamais conda) |
| Qualité | `ruff` pour le formatage **et** le lint |
| Tests | `pytest` |
| Secrets | variables d'environnement, `.env` jamais commité |

Ce qui est **ouvert** — on choisit par projet, et on écrit pourquoi :
framework d'orchestration, fournisseur de modèle, base vectorielle, outil
d'évaluation. L'écosystème bouge trop vite pour figer ça à l'échelle de
l'organisation. Le choix se justifie dans le README du dépôt concerné.

## Cycle de vie

Chaque dépôt porte un statut visible, marqué par un topic GitHub :

`statut-exploration` → `statut-actif` → `statut-gele` → `statut-archive`

Un POC qui échoue est un résultat, pas un échec : on l'archive avec sa
conclusion écrite. **On ne supprime jamais un dépôt** — on perdrait la seule
chose qu'il avait produite, la réponse à la question qu'il posait.

Détail des statuts : [CYCLE-DE-VIE.md](https://github.com/Sandbox-IA-FHU/.github/blob/main/CYCLE-DE-VIE.md).

## Langue

Français partout : documentation, noms de branches, messages de commit,
titres de PR, libellés d'issues. Les identifiants de code restent en anglais
(noms de variables, de fonctions, de classes), comme partout ailleurs.
