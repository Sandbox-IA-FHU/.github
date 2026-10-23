# Cycle de vie des dépôts

En IA, la plupart des projets qu'on démarre ne vont pas en production. C'est
normal et attendu : un POC sert à répondre à une question, et « non, ça ne
marche pas » est une réponse valide qui a de la valeur.

Le problème n'est donc pas l'échec, c'est l'ambiguïté. Un dépôt dont personne
ne sait s'il est vivant, abandonné ou concluant fait perdre du temps à tout le
monde — à commencer par celui qui l'a écrit, six mois plus tard.

D'où la règle : **tout dépôt porte un statut explicite, à jour.**

## Les cinq statuts

Le statut est marqué par un topic GitHub sur le dépôt, et répété dans l'en-tête
de son README.

### `statut-exploration`

On cherche. Le dépôt répond à une question ouverte : est-ce que tel modèle sait
faire telle tâche, est-ce que telle approche tient sur nos volumes.

- Le README **doit** énoncer la question posée, dès le premier commit
- Le code peut être sale, les notebooks peuvent traîner
- Personne d'autre ne dépend de ce dépôt
- **Durée de vie attendue : quelques semaines**

On en sort dans un sens ou dans l'autre. Un dépôt en exploration sans commit
depuis 60 jours passe en revue : soit il repart, soit il est gelé ou archivé.
Il ne reste pas en exploration par inertie.

### `statut-actif`

Le sujet est validé et le travail continue. Ce qui devient exigible :

- Un jeu d'évaluation existe, avec des métriques et un seuil écrit
- Les dépendances sont figées (`uv.lock` commité)
- Les tests passent, `ruff` est propre
- Le README dit comment lancer le projet, avec des commandes vérifiées
- La charge (jetons par tâche, nombre de tours) et la latence sont mesurées
  et notées, avec leur date

Un dépôt en exploration ne passe pas en actif tant que le jeu d'évaluation
n'existe pas. C'est le seul point non négociable de la transition : sans mesure,
on ne saura jamais si les modifications suivantes améliorent ou dégradent.

### `statut-importe`

Le projet a été validé par Florian et **importé dans le dépôt de production**
par le responsable. C'est l'issue attendue d'un dépôt `statut-actif` qui a
tenu son seuil.

- Ce qui est importé est toujours la branche `main`, après fusion d'une PR
  `staging` → `main`.
- Le dépôt **reste vivant** : les évolutions continuent ici, sur `staging`,
  puis repassent par une PR vers `main` et un nouvel import. Tout ce qui est
  exigible en `statut-actif` le reste.
- Un correctif fait directement en production est **reporté ici par une PR**,
  dans la foulée. Deux versions qui divergent, c'est un import suivant qui
  écrase un correctif sans que personne ne le voie.

Le README note chaque import, dans un tableau à part de l'en-tête :

```markdown
| Date d'import | Commit de `main` importé | Version de production |
|---|---|---|
| AAAA-MM-JJ | `<commit court>` | vX.Y.Z |
```

La ligne est ajoutée par Florian au moment de l'import (ou à sa demande),
avec le verdict de l'en-tête mis à jour : « importé en production le …, en
version … ».

### `statut-gele`

Le travail s'arrête, mais le sujet n'est pas clos — attente d'un arbitrage,
d'un budget, d'une donnée, d'une décision client.

- Le README dit **sur quoi** on attend et **qui** doit trancher
- L'état est laissé fonctionnel : quelqu'un doit pouvoir reprendre
- Aucune modification n'est attendue

Un gel qui dure plus de six mois est un archivage qui n'ose pas dire son nom.
On le convertit.

### `statut-archive`

Le sujet est clos. Le dépôt est passé en lecture seule (*Archive* dans les
réglages GitHub).

**On n'archive pas sans écrire la conclusion.** C'est la seule chose que le
dépôt laissera derrière lui. Le README final répond à quatre questions :

1. Quelle question posait ce projet ?
2. Quelle est la réponse, avec les chiffres et leur date ?
3. Pourquoi on s'arrête — techniquement, économiquement, ou parce que le besoin
   a disparu ?
4. Qu'est-ce qu'on referait autrement, et sous quelle condition ça vaudrait le
   coup de reprendre ?

Un archivage sans conclusion écrite transforme des semaines de travail en
répertoire mort. C'est la perte la plus bête de toutes.

## On ne supprime jamais un dépôt

Aucune exception, y compris pour les POC ratés — surtout pour eux. Le résultat
négatif est ce qui évite de refaire deux fois la même impasse.

La seule raison de supprimer serait une fuite de donnée sensible. Dans ce cas,
ce n'est pas un archivage mais un incident : voir [SECURITY.md](SECURITY.md),
et supprimer le dépôt ne suffit de toute façon pas à traiter la fuite.

## En-tête de README obligatoire

Tout dépôt de l'organisation commence par ce bloc, quel que soit son statut :

```markdown
> **Statut** : exploration
> **Mise à jour** : 2026-09-04
> **Question posée** : est-ce qu'un modèle sait extraire les lignes d'une
> facture scannée avec assez de fiabilité pour éviter la saisie manuelle ?
> **Verdict** : en cours
```

Trois règles sur ce bloc :

- La date est celle de la dernière modification réelle. Une date qui ne bouge
  pas alors que le dépôt vit, c'est pire que pas de date : ça ment.
- La question est formulée en une phrase, en français, compréhensible par
  quelqu'un qui n'est pas sur le projet.
- Le verdict n'est pas rempli à la fin. On le met à jour en cours de route,
  au fil de ce qu'on apprend.

## Revue périodique

Une fois par mois, passer les dépôts en revue et vérifier :

- les statuts correspondent à la réalité
- aucun dépôt en exploration n'est immobile depuis plus de 60 jours
- aucun gel ne dure depuis plus de six mois
- chaque dépôt `statut-importe` a son dernier import noté, et aucun correctif
  de production n'attend d'être reporté
- les dates d'en-tête sont crédibles

Cinq minutes par mois. C'est ce qui évite de se retrouver, dans deux ans, avec
trente dépôts dont on n'ose plus toucher aucun.
