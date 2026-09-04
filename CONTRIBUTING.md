# Guide de contribution

Ces règles s'appliquent à tous les dépôts de l'organisation. Elles sont peu
nombreuses et toutes applicables : une règle qu'on ne peut pas suivre
décrédibilise les autres.

## Mise en route

Prérequis : Python 3.13, `uv`, Git 2.40+.

```bash
uv sync
uv run ruff check .
uv run pytest
```

Ces trois commandes doivent fonctionner sur tout dépôt de l'organisation, sans
étape supplémentaire non documentée. Si ce n'est pas le cas pour un dépôt,
c'est un défaut du dépôt : ouvrez une issue.

On utilise `uv`, jamais `pip` seul, jamais `poetry`, jamais `conda`. Le fichier
`uv.lock` est commité : deux personnes qui installent le même projet à deux
semaines d'écart doivent obtenir les mêmes versions. En IA c'est plus critique
qu'ailleurs — une version mineure de bibliothèque cliente change parfois le
comportement par défaut d'un appel modèle.

## Avant de coder

Le cadrage précède le code. Avant d'ouvrir l'éditeur, deux choses doivent être
écrites, même en trois lignes :

1. **La question à laquelle on répond**, en français, compréhensible par
   quelqu'un d'extérieur au projet.
2. **Comment on saura que c'est réussi** — la mesure, le nombre de cas, le
   seuil. Voir [EVALUATION.md](EVALUATION.md).

Coder d'abord et mesurer ensuite est le réflexe le plus coûteux en IA : on
passe des semaines à améliorer une impression.

## Branches

Nommées en français, descriptives de la valeur apportée :

```
extraction-lignes-facture
comparaison-modeles-classification
correction-timeout-embeddings
```

Pas `test`, pas `fix`, pas `ma-branche`, pas `dev2`. Dans six mois, le nom de
la branche est parfois la seule trace de l'intention.

On rebase plutôt qu'on ne fusionne, pour garder un historique linéaire et
lisible.

## Commits

En français, à l'impératif, décrivant l'effet et non l'action mécanique :

```
Ajouter le jeu d'évaluation sur les factures scannées
Réduire le coût par extraction en regroupant les appels
Figer la version du modèle d'embedding
```

Pas de Conventional Commits (`feat:`, `fix:`) — on n'en tire aucune valeur ici
et ça ajoute une convention à apprendre pour rien.

Pas de `wip`, pas de `maj`, pas de `.`. Un commit dont le message ne dit rien
est un commit qu'il faudra ouvrir pour comprendre.

Quand le choix n'est pas évident, le corps du message explique **pourquoi** —
c'est ce qu'on cherche en relisant l'historique, pas le quoi, qui est déjà
dans le diff.

## Les deux types de pull request

C'est la particularité de l'organisation : une PR de code et une PR
d'expérience ne se relisent pas de la même façon.

### PR de code

Elle modifie un système qui doit fonctionner. Ce qu'on attend :

- `ruff` propre, `pytest` vert
- Le jeu d'évaluation a tourné, et les résultats sont dans la PR
- Aucune régression sous le seuil défini
- Le coût et la latence sont indiqués s'ils ont bougé

### PR d'expérience

Elle apporte un **résultat**, pas une fonctionnalité. Le code peut être
imparfait, c'est normal — ce qui est relu, c'est la validité de la conclusion.

Ce qu'on attend :

- L'hypothèse testée, écrite avant le test
- La configuration complète : modèle **et version figée**, paramètres, prompt
- Les chiffres obtenus, avec leur date et le nombre de cas
- La conclusion, y compris quand elle est négative
- Ce que l'expérience ne prouve pas

Ce dernier point est le plus important et le plus souvent oublié. Un test sur
vingt cas ne prouve rien sur dix mille. Le dire soi-même vaut mieux que se le
faire dire en revue.

## Contenu d'une PR

Le titre décrit l'effet, pas le contenu du diff. « Fiabiliser l'extraction des
montants » plutôt que « Modification de extract.py ».

Le corps répond à quatre questions : ce qui change, d'où vient la demande, quels
choix ont été faits et pourquoi, ce qui a été volontairement laissé de côté.

Le gabarit est pré-rempli à l'ouverture de la PR.

## Avant de demander une revue

- [ ] Relisez votre propre diff, fichier par fichier
- [ ] `uv run ruff check .` et `uv run pytest` passent
- [ ] `git diff` ne contient aucune donnée, aucune clé, aucun nom de client
- [ ] Les notebooks sont vidés de leurs sorties
- [ ] Les chiffres annoncés portent leur date

La relecture de son propre diff attrape la moitié des remarques de revue. C'est
cinq minutes qui en font gagner trente à quelqu'un d'autre.

## Faire une revue

Distinguez ce qui bloque de ce qui est une préférence. Une remarque non
bloquante commence par `détail :` — sans ça, l'auteur ne peut pas savoir ce
qu'il doit traiter et se retrouve à tout traiter.

Points à regarder en priorité sur un projet IA :

- **La conclusion est-elle soutenue par les chiffres ?** C'est le vrai sujet.
  Un échantillon trop petit, un jeu d'éval qui ressemble trop aux exemples du
  prompt, une comparaison entre deux runs à configurations différentes.
- **La version du modèle est-elle figée ?** Un alias non figé rend le résultat
  invalide dès la prochaine mise à jour côté fournisseur.
- **Rien n'a fuité ?** Données, clés, noms.
- Puis seulement : la qualité du code.

Approuver, c'est partager la responsabilité du résultat. Si vous n'avez pas
compris pourquoi un chiffre est ce qu'il est, demandez avant d'approuver.

## Intégration continue

Chaque dépôt appelle le workflow commun de l'organisation, en créant un fichier
`.github/workflows/ci.yml` qui contient :

```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: Sandbox-IA-FHU/.github/.github/workflows/reusable-python-ci.yml@main
```

Trois vérifications tournent alors sur chaque push et chaque PR :

| Check | Ce qu'il vérifie |
|---|---|
| `qualite` | `ruff check` et `ruff format --check` |
| `tests` | `pytest`, avec des clés d'API volontairement invalides |
| `donnees` | Aucun fichier de données suivi, aucun notebook avec ses sorties |

Les clés invalides du job `tests` ne sont pas un oubli : **aucun test ne doit
appeler un vrai fournisseur de modèle**. C'est lent, c'est facturé, et le
résultat change d'une exécution à l'autre — un test qui échoue une fois sur
trois ne sert à rien. Les appels se simulent.

Un dépôt en exploration qui n'a pas encore de tests peut passer
`lancer-tests: false` en paramètre. Le check `donnees`, lui, ne se désactive
pas.

## Quand vous bloquez

Cherchez trente minutes, puis demandez. Au-delà, vous ne cherchez plus, vous
tournez en rond.

Dans l'ordre : lisez le message d'erreur en entier (vraiment en entier), relisez
le README du dépôt, cherchez dans les Discussions, puis posez la question en
donnant ce que vous avez déjà essayé.

Voir [SUPPORT.md](SUPPORT.md).
