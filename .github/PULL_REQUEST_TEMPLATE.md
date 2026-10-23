<!--
Deux types de PR dans cette organisation, elles ne se relisent pas pareil.
Gardez la section qui correspond, supprimez l'autre.
Dans un dépôt de projet, la PR va de staging vers main ; Florian l'approuve
et la fusionne en merge commit. Voir CONTRIBUTING.md.
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
| Jetons par tâche (entrée + sortie) | |
| Tours, médiane / max | |
| Latence médiane / p95 | |

- [ ] `ruff check`, `ruff format --check` et `pytest` passent (par `uv` ou en conteneur)
- [ ] Si `compose.yaml` ou le `Dockerfile` a changé : `docker compose config -q` passe
- [ ] Le jeu d'évaluation a tourné, aucune régression sous le seuil
- [ ] Les chiffres ci-dessus portent leur date, le modèle (nom exposé, modèle réel, révision) et le niveau de réflexion

## Si c'est une PR d'expérience

**Hypothèse testée** (écrite avant le test) :

**Configuration** — modèle (nom exposé, modèle réel, révision), niveau de
réflexion, paramètres, prompt, jeu :

**Résultats** — chiffres, date, nombre de cas, jetons par tâche, tours,
latence :

**Conclusion**, y compris négative :

**Ce que cette expérience ne prouve pas** :
<!-- Le point le plus important et le plus souvent oublié. Un test sur
     20 cas ne prouve rien sur 10 000. -->

---

## Avant de demander la revue

- [ ] J'ai relu mon propre diff, fichier par fichier
- [ ] Aucune donnée réelle, aucune clé, aucun nom de client dans le diff
- [ ] Les notebooks sont vidés de leurs sorties
- [ ] Aucune adresse ni aucun nom de serveur dans le diff : l'adresse du LLM vient de `LLM_BASE_URL`
- [ ] La réflexion du modèle n'apparaît dans aucun log, trace ou fichier commité
- [ ] Le statut et la date en tête du README sont à jour
