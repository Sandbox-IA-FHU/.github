# Guide de contribution

Ces règles s'appliquent à tous les dépôts de l'organisation. Elles sont peu
nombreuses et toutes applicables : une règle qu'on ne peut pas suivre
décrédibilise les autres.

## Mise en route

Sur un poste — prérequis : Python 3.13, `uv`, Git 2.40+.

```bash
uv sync
uv run ruff check .
uv run pytest
```

Sur le serveur de staging, rien ne s'installe au niveau du système : les
mêmes vérifications tournent en conteneur, avec votre propre Docker rootless
(`docker compose run --rm <service> pytest`, voir le README du dépôt).

Ces commandes doivent fonctionner sur tout dépôt de l'organisation, sans
étape supplémentaire non documentée, par les deux voies. Si ce n'est pas le
cas pour un dépôt, c'est un défaut du dépôt : ouvrez une issue.

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

## Le circuit d'un projet

Un projet = un dépôt, **public**, créé depuis le modèle correspondant
(`agentic-model`, `rag-model`).

**Vous ne créez pas de dépôt.** Demandez-le à Florian (responsable), en lui
disant si le projet est **individuel ou à deux**. Il crée le dépôt, vous y
ajoute en rôle **Write**, pose les protections et crée la branche `staging`.
Vous clonez, puis vous passez sur `staging` :

```bash
git clone https://github.com/Sandbox-IA-FHU/<nom-du-projet>.git
cd <nom-du-projet>
git switch staging
```

Chaque dépôt de projet a deux branches, sur le modèle du dépôt de
production :

| Branche | Rôle |
|---|---|
| `staging` | Branche de travail. On y pousse directement. |
| `main` | Version validée, celle qui est importée en production. |

Le circuit complet :

1. vous poussez votre travail sur `staging` ;
2. quand un état est prêt, vous ouvrez une PR `staging` → `main` ;
3. Florian la relit et l'approuve — c'est le relecteur obligatoire désigné
   par `.github/CODEOWNERS` ;
4. Florian la fusionne, en **merge commit** ;
5. quand le projet est validé, Florian l'importe dans le dépôt de production.
   Vous n'avez pas accès à ce dépôt et ne cherchez pas à y accéder.

La PR `staging` → `main` se fusionne toujours en merge commit, **jamais en
squash ni en rebase** : ces deux modes recréent les commits sur `main`, et
`staging` divergerait de `main` à chaque fusion.

Les protections, posées par Florian **sur chaque dépôt** :

| Branche | Règles |
|---|---|
| `main` | PR obligatoire, approbation du code owner sur le dernier push, fusion en merge commit, pas de suppression, pas de force-push |
| `staging` | push direct autorisé, pas de suppression, pas de force-push |

Vous avez le rôle Write : vous ne pouvez modifier ni les réglages du dépôt ni
ses protections. C'est voulu — un administrateur de dépôt pourrait les
supprimer.

Après un import en production, **les évolutions continuent dans le dépôt de
projet**. Un correctif fait directement en production est reporté dans le
dépôt de projet par une PR, pour que les deux versions ne divergent pas.

## Branches de travail personnelles

Pour un travail que vous voulez isoler avant de le pousser sur `staging`,
une branche personnelle est la bienvenue. Sur un **projet à deux**, c'est la
règle : chacun travaille sur sa branche personnelle, puis l'intègre dans
`staging`.

Nommée en français, descriptive de la valeur apportée :

```
extraction-lignes-facture
comparaison-modeles-classification
correction-timeout-embeddings
```

Pas `test`, pas `fix`, pas `ma-branche`, pas `dev2`. Dans six mois, le nom de
la branche est parfois la seule trace de l'intention.

Créer sa branche depuis `staging` à jour :

```bash
git switch staging
git pull --ff-only
git switch -c <branche-personnelle>
```

Avant d'intégrer, se mettre à jour avec ce qui a déjà été intégré dans
`staging`, puis relancer les tests :

```bash
git fetch origin
git rebase origin/staging
```

Puis intégrer dans `staging` et pousser :

```bash
git switch staging
git pull --ff-only
git merge --ff-only <branche-personnelle>
git push origin staging
```

Si `git merge --ff-only` ou `git push` refuse, l'autre a intégré
entre-temps : revenez sur votre branche et recommencez à `git fetch origin`.

On rebase plutôt qu'on ne fusionne, pour garder un historique linéaire et
lisible. Une branche personnelle déjà poussée puis rebasée se repousse avec
`git push --force-with-lease`. Ne rebasez jamais `staging` ni `main` : elles
sont partagées, et le force-push y est interdit.

## Pour le responsable

Création d'un dépôt de projet, une fois par projet :

```bash
gh repo create Sandbox-IA-FHU/<nom-du-projet> --template Sandbox-IA-FHU/<modele> --public
gh api -X PUT repos/Sandbox-IA-FHU/<nom-du-projet>/collaborators/<compte-github> -f permission=push
git clone https://github.com/Sandbox-IA-FHU/<nom-du-projet>.git
cd <nom-du-projet>
git switch -c staging
git push -u origin staging
```

- `permission=push` correspond au rôle Write. Jamais Admin.
- « Use this template » ne copie que `main` : `staging` se crée ici.
- Les protections du tableau ci-dessus se posent ensuite sur le dépôt lui-même
  (Settings → Rules → Rulesets). En plan gratuit, les rulesets
  d'organisation ne sont pas appliqués : seuls ceux de dépôt le sont.

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
- Les jetons par tâche, le nombre de tours et la latence sont indiqués s'ils
  ont bougé

### PR d'expérience

Elle apporte un **résultat**, pas une fonctionnalité. Le code peut être
imparfait, c'est normal — ce qui est relu, c'est la validité de la conclusion.

Ce qu'on attend :

- L'hypothèse testée, écrite avant le test
- La configuration complète : modèle (nom exposé, modèle réel, révision),
  niveau de réflexion, paramètres, prompt
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
| `tests` | `pytest`, avec une adresse de LLM injoignable et une clé factice |
| `donnees` | Aucun `.env` suivi, aucune donnée hors de `evaluations/jeux/`, aucun fichier de plus de 5 Mo, aucun notebook avec ses sorties |

L'adresse injoignable du job `tests` n'est pas un oubli : **aucun test ne
doit appeler un vrai modèle**. C'est lent, ça charge un serveur partagé, et le
résultat change d'une exécution à l'autre — un test qui échoue une fois sur
trois ne sert à rien. Les appels se simulent.

Un dépôt peut ajouter ses propres jobs à côté du workflow commun. Le modèle
`agentic-model` ajoute ainsi `conteneur`, qui construit l'image et lance les
tests dedans.

Le check `donnees` filtre par **emplacement**, pas par extension : un jeu
d'évaluation au format CSV est légitime dans `evaluations/jeux/` et refusé
partout ailleurs. Le contrôle de taille complète le dispositif — c'est lui qui
rattrape un export réel déposé dans la zone autorisée sous une extension
anodine.

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
