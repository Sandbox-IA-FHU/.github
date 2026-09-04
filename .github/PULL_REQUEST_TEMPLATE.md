<!--
Deux types de PR dans cette organisation, elles ne se relisent pas pareil.
Gardez la section qui correspond, supprimez l'autre.
Voir CONTRIBUTING.md.
-->

## Ce qui change

<!-- L'effet, pas le contenu du diff. En quelques lignes. -->

## D'où vient la demande

<!-- Issue liée, discussion, besoin exprimé. -->

## Choix faits

<!-- Ce qui n'était pas évident, et pourquoi vous avez tranché ainsi.
     C'est la partie qu'on relit dans six mois. -->

## Volontairement laissé de côté

<!-- Ce que vous auriez pu faire et n'avez pas fait, et pourquoi.
     Le dire soi-même évite trois allers-retours en revue. -->

---

## Si c'est une PR de code

| | |
|---|---|
| Score avant | |
| Score après | |
| Nombre de cas | |
| Coût par unité | |
| Latence médiane / p95 | |

- [ ] `uv run ruff check .` et `uv run pytest` passent
- [ ] Le jeu d'évaluation a tourné, aucune régression sous le seuil
- [ ] Les chiffres ci-dessus portent leur date et la version de modèle utilisée

## Si c'est une PR d'expérience

**Hypothèse testée** (écrite avant le test) :

**Configuration** — modèle et version figée, paramètres, prompt, jeu :

**Résultats** — chiffres, date, nombre de cas :

**Conclusion**, y compris négative :

**Ce que cette expérience ne prouve pas** :
<!-- Le point le plus important et le plus souvent oublié. Un test sur
     20 cas ne prouve rien sur 10 000. -->

---

## Avant de demander la revue

- [ ] J'ai relu mon propre diff, fichier par fichier
- [ ] Aucune donnée réelle, aucune clé, aucun nom de client dans le diff
- [ ] Les notebooks sont vidés de leurs sorties
- [ ] Les identifiants de modèle sont figés, pas des alias mouvants
- [ ] Le statut et la date en tête du README sont à jour
