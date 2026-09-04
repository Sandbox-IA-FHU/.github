# Dépôt `.github` de l'organisation Sandbox IA

Ce dépôt contient les fichiers communs appliqués par défaut à tous les dépôts
de l'organisation. Il n'héberge aucun projet.

## Ce qu'il contient

| Fichier | Rôle |
|---|---|
| [`profile/README.md`](profile/README.md) | Page d'accueil publique de l'organisation |
| [`EVALUATION.md`](EVALUATION.md) | Comment on mesure qu'un système IA fonctionne |
| [`DONNEES.md`](DONNEES.md) | Ce qui n'entre jamais dans un dépôt |
| [`CYCLE-DE-VIE.md`](CYCLE-DE-VIE.md) | Statuts des dépôts, archivage, revue mensuelle |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Branches, commits, pull requests, revue |
| [`SECURITY.md`](SECURITY.md) | Clés, risques propres aux systèmes IA, signalement |
| [`SUPPORT.md`](SUPPORT.md) | Où et comment demander de l'aide |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Règles de vie |
| [`CLAUDE.md`](CLAUDE.md) | Conventions pour les sessions Claude Code |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE) | Gabarits d'issues : expérimentation, régression, demande de POC |
| [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) | Gabarit de pull request |
| [`.github/workflows/`](.github/workflows) | Workflow d'intégration continue réutilisable |

## Comment fonctionne l'héritage

Trois règles, et trois pièges.

**Fichier par fichier.** Un dépôt qui fournit son propre `CONTRIBUTING.md`
remplace celui-ci pour lui seul ; les autres fichiers continuent d'être hérités.

**Les workflows ne s'héritent pas.** Contrairement aux fichiers de communauté,
un workflow doit être appelé explicitement par chaque dépôt. Le mode d'emploi
est en tête de
[`reusable-python-ci.yml`](.github/workflows/reusable-python-ci.yml).

**La visibilité compte.** Ce dépôt public n'alimente que les dépôts publics de
l'organisation. Pour couvrir des dépôts privés, il faut un dépôt `.github`
privé distinct — sinon les fichiers ne sont tout simplement pas repris, sans
avertissement.

Piège supplémentaire : renommer un job du workflow casse silencieusement tout
ruleset qui l'exige comme check obligatoire. Un check requis dont le nom ne
correspond à aucun job n'est jamais satisfait, et la PR reste bloquée sans
message explicite.

## Modifier ces fichiers

Une modification ici s'applique à tous les dépôts de l'organisation, présents
et futurs. Une coquille se propage partout.

Donc : par pull request, jamais en poussant directement sur `main`.

Trois choses à vérifier avant de proposer un changement :

- **La règle est applicable.** Une règle qu'on ne peut pas suivre décrédibilise
  toutes les autres. Mieux vaut pas de règle qu'une règle ignorée.
- **Les commandes sont vérifiées.** Elles sont copiées-collées par des
  personnes en apprentissage ; une commande approximative fait perdre une heure.
- **Aucune donnée sensible.** Ce dépôt est public : pas de clé, pas de nom de
  client, pas de chemin réseau interne, pas d'adresse personnelle.

## Langue

Français partout, y compris les noms de fichiers de documentation. Les
identifiants techniques imposés par GitHub (`workflows`, `ISSUE_TEMPLATE`,
`SECURITY.md`) gardent évidemment leur nom réglementaire.
