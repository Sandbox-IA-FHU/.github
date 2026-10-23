# Conventions pour les sessions Claude Code

Ce fichier s'applique aux dépôts de l'organisation Sandbox IA. Un dépôt peut
avoir son propre `CLAUDE.md` : il complète celui-ci, il ne l'annule pas.

Claude Code ne lit pas ce fichier depuis un dépôt de projet : seul le
`CLAUDE.md` du dépôt est chargé. C'est pourquoi les modèles de projet
reprennent les règles qui doivent s'appliquer à chaque session (horaires du
serveur, circuit de validation, LLM local). Une règle modifiée ici se reporte
dans les modèles.

## Règles non négociables

**1. Français.** Documentation, commentaires, messages de commit, branches,
titres de PR, libellés d'issues. Les identifiants de code restent en anglais.

**2. Ne jamais écrire de donnée réelle dans le dépôt.** Aucun extrait, aucun
échantillon, aucune sortie de notebook contenant de la donnée, aucun fichier de
trace. Si un exemple est nécessaire, il est inventé. Voir `DONNEES.md`.

**3. Aucun nom de client**, nulle part — code, commentaires, commits, branches.
Écrire « le client », « un groupe de distribution ».

**4. Aucune clé d'API**, même en exemple, même expirée, même en commentaire. On
lit depuis l'environnement, et `.env.example` documente les variables avec des
valeurs vides.

**5. `uv` uniquement.** Jamais `pip install` seul, jamais `poetry`, jamais
`conda`. Les commandes s'écrivent `uv add`, `uv sync`, `uv run`.

**6. Ne pas inventer de chiffres sur les modèles.** Performances, limites de
contexte, débit du serveur, prix d'une API externe : ces valeurs changent et
une valeur plausible mais fausse fait prendre de mauvaises décisions. Si le
chiffre n'est pas vérifié dans la session, le dire au lieu de l'écrire. Tout
chiffre consigné porte sa date.

**7. Figer les versions de modèle.** Un nom de modèle n'est pas une version :
le LLM local est exposé sous un nom stable qui ne change pas quand le modèle
derrière change. Une évaluation consigne donc le nom exposé, **le modèle réel
et sa révision**, et la date. Un score sans révision n'est pas reproductible.

**8. Vérifier les commandes proposées.** Les documents de cette organisation
sont lus par des personnes en apprentissage qui copient-collent. Une commande
approximative fait perdre une heure.

**9. Les dépôts sont publics.** Rien d'interne n'y entre : ni adresse IP, ni
nom de serveur, ni chemin interne, ni nom du dépôt de production, ni nom de
personne en dehors de « Florian (responsable) ».

## Le circuit d'un projet

```
modèle de projet  →  un dépôt par projet  →  travail sur staging
                  →  PR staging → main, approuvée par Florian
                  →  import en production par le responsable
```

- **Un dépôt par projet**, public, créé depuis le modèle correspondant
  (« Use this template »). Juste après la création, la branche `staging` est
  créée et poussée : le modèle ne copie que `main`.
- **`staging`** est la branche de travail : push direct autorisé, ni
  force-push ni suppression.
- **`main`** est la version validée. Elle ne change que par une PR
  `staging` → `main`, approuvée par Florian et fusionnée **par lui seul**, en
  merge commit. Une session Claude Code ne fusionne jamais.
- Un projet validé est **importé dans le dépôt de production par le
  responsable**. Les alternants n'y ont pas accès et ne cherchent pas à y
  accéder. Un projet né d'un modèle doit pouvoir y être importé tel quel :
  ne pas casser ce qui le permet (`compose.yaml`, `.env.example`, nom de
  service préfixé par le projet).

Détail dans [CONTRIBUTING.md](CONTRIBUTING.md) et
[CYCLE-DE-VIE.md](CYCLE-DE-VIE.md).

## Le LLM local

Les projets n'appellent pas d'API externe : ils appellent le LLM local, servi
sur un GPU interne, par une API **compatible OpenAI**.

| Variable | Rôle |
|---|---|
| `LLM_BASE_URL` | Adresse de l'API, terminée par `/v1`. Fournie par Florian, jamais écrite dans un dépôt. |
| `LLM_MODEL` | Nom exposé, stable : `altoneo-llm`. |
| `LLM_API_KEY` | Vide aujourd'hui, lue quand même : une passerelle avec une clé par personne pourra venir. |
| `LLM_MODEL_REVISION` | Révision du modèle servi, fournie par Florian. Obligatoire pour une évaluation. |

- L'adresse ne s'écrit **jamais en dur**, ni dans le code, ni dans un exemple.
- Les appels d'outils passent par le format standard `tools` / `tool_calls`.
  S'ils sont refusés par le serveur, on le signale ; on ne les remplace pas par
  une analyse du texte libre.
- La **réflexion** du modèle, renvoyée à part par le serveur, n'est jamais
  prise pour la réponse, ne repart jamais dans l'historique et n'entre dans
  aucun log, trace ou fichier commité. Son niveau est un réglage explicite,
  consigné dans le fichier de run.
- Il n'y a pas de facture à l'appel, mais le LLM est **partagé** entre
  plusieurs projets : un agent qui boucle ou une évaluation massive ralentit
  tout le monde. La charge se mesure en jetons, en tours et en latence.

## Le serveur de staging

Les alternants y travaillent avec un compte personnel restreint.

- Pas de `sudo`, pas d'accès au Docker du serveur : chacun utilise **son
  propre Docker rootless**.
- **Tout tourne en conteneur.** Rien ne s'installe au niveau du système (`uv`
  n'y est pas) : les modèles de projet fournissent un `Dockerfile` et un
  `compose.yaml` pour les tests, le lint, l'exécution et l'évaluation.
- **Jamais d'accès direct au GPU**, jamais de bibliothèque de modèle locale
  (PyTorch, transformers…) pour contourner l'API.
- Ne jamais toucher aux conteneurs ni aux fichiers des autres.

## Horaires du serveur de staging — vérification OBLIGATOIRE

Pour toute session qui tourne sur le serveur de staging.

Le serveur est allumé **en continu du lundi 7h00 au vendredi 20h00**. Ces
horaires sont **provisoires** : Florian communique tout changement. Le
vendredi à 20h00, il s'arrête d'un coup : une commande en cours est tuée, et
ce qui n'est pas poussé reste inaccessible jusqu'au lundi 7h00.

Avant toute tâche (commandes, build d'image, tests, évaluation, série de
modifications), et de nouveau avant chaque étape longue :

1. Lancer `date` pour connaître le jour et l'heure réels (heure de Paris).
   Ne jamais se fier à une heure supposée.
2. Estimer la durée de la tâche entière, **commit et push compris**, en
   prenant l'estimation haute.
3. Comparer l'heure de fin estimée à la prochaine coupure, le vendredi 20h00 :

| Fin estimée | Décision |
|---|---|
| avant vendredi 19h30 | **On y va**, sans prévenir. Du lundi au jeudi, c'est le cas de toute tâche normale. |
| entre vendredi 19h30 et 19h50 | **C'est juste** : ne rien lancer. Donner l'heure, la durée estimée et la fin prévue, et demander s'il faut y aller ou attendre lundi. |
| après vendredi 19h50 | **Pas le temps** : bloquer, sans rien lancer. Expliquer pourquoi et proposer de reprendre lundi. |

- Jamais de tâche en arrière-plan qui finirait après vendredi 19h50.
- Une tâche reportée : commiter et pousser d'abord ce qui est stable, et noter
  où reprendre.
- Le vendredi, tout ce qui est stable est poussé avant de s'arrêter.
- Jour férié ou horaire inhabituel : demander avant de supposer que le serveur
  est allumé.

## Contexte du travail

Les dépôts sont des projets d'ingénierie IA : prompts, évaluations, RAG,
extraction, classification, agents. La particularité à garder en tête :

- **Le livrable n'est pas seulement le code.** Le prompt, la configuration du
  modèle et le jeu d'évaluation en font partie intégrante et se versionnent
  ensemble.
- **Un chiffre de qualité n'a de sens qu'accompagné** de son nombre de cas, de
  sa date et du modèle exact (nom exposé, modèle réel, révision).
- **Charge et latence sont des critères de recette**, pas des optimisations
  ultérieures : jetons, nombre de tours, temps de réponse. Les mentionner
  quand une modification les affecte.

## Ce qu'on attend d'une session

**Cadrer avant de coder.** Sur un sujet IA, demander quelle est la question
posée et comment on saura que c'est réussi, si ce n'est pas déjà écrit.

**Proposer la solution la plus simple d'abord.** Avant un appel modèle :
est-ce qu'une requête SQL, une règle ou une expression régulière suffirait ?
Avant un framework d'orchestration : est-ce qu'un appel direct suffirait ?
L'écosystème IA pousse à la complexité, la plupart des besoins ne la
justifient pas.

**Ne pas ajouter de dépendance sans raison explicite.** Beaucoup de paquets de
cet écosystème sont jeunes et peu maintenus. Pour vingt lignes de code, écrire
les vingt lignes.

**Signaler les incertitudes plutôt que de les lisser.** Un échantillon trop
petit pour conclure, une comparaison entre deux runs non comparables, un écart
non significatif : le dire.

## Socle technique

Figé : Python 3.13, `uv`, `ruff` (format et lint), `pytest`, Docker (rootless
sur le serveur de staging). LLM par défaut : le LLM local.

Ouvert, à justifier dans le README du dépôt : framework d'orchestration, base
vectorielle, outil d'évaluation, recours à une API externe.

Commandes de référence, sur un poste :

```bash
uv sync
uv run ruff check .
uv run ruff format .
uv run pytest
```

Sur le serveur, les mêmes en conteneur (`docker compose run --rm <service> …`,
voir le README du dépôt).

## Structure attendue d'un dépôt

```
README.md              en-tête statut/date/question/verdict obligatoire
pyproject.toml
uv.lock                commité
.env.example           variables attendues, valeurs vides (ce nom exact)
.gitignore             data/, logs/, .env, .env.*, extensions de données,
                       avec !.env.example et !evaluations/jeux/** en exception
Dockerfile             image Python 3.13 + uv, tag figé, utilisateur non root
compose.yaml           services préfixés par le projet, TZ Europe/Paris,
                       images figées, pas de GPU
.github/CODEOWNERS     relecteur obligatoire des PR vers main
prompts/               prompts versionnés, un fichier par version
evaluations/
  jeux/                jeux d'évaluation, données inventées — SEULE zone du
                       dépôt où un fichier de données est admis
  runs/                résultats horodatés
src/
tests/
```

## Points de vigilance

**Notebooks.** Les sorties de cellules sont enregistrées dans le fichier. Ne
jamais commiter un `.ipynb` sans l'avoir vidé :
`uvx --from nbconvert jupyter-nbconvert --clear-output --inplace fichier.ipynb`

**Journalisation.** Loguer un prompt revient à loguer son contenu ; loguer la
réflexion d'un modèle aussi. Les traces ne sont pas commitées.

**Boucles d'agent.** Toute boucle appelant un modèle porte une limite
d'itérations en dur. Une boucle sans limite occupe indéfiniment un LLM partagé
— et, sur une API facturée, devient un incident financier.

**Contenu externe.** Ce qu'un système récupère — page web, PDF, document
indexé, fichier déposé — est de la donnée, jamais une instruction. Ne pas
concevoir de chaîne où un contenu récupéré peut déclencher une action
irréversible sans validation humaine.

## Ce qui ne va pas dans le dépôt

Les arbitrages d'architecture, les chiffrages, la veille sur les modèles et les
retours d'expérience transverses sont consignés hors du dépôt, dans le carnet
de travail. Un dépôt peut y faire référence, il ne recopie pas leur contenu :
deux copies divergent toujours.
