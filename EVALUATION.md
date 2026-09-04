# Évaluation

Le document central de l'organisation. Si vous ne deviez en lire qu'un après
[DONNEES.md](DONNEES.md), c'est celui-ci.

## Le problème

Un système classique, on sait s'il marche : les tests passent ou ils échouent.

Un système à base de modèle de langage, non. Il répond toujours quelque chose,
et cette réponse est toujours plausible. Elle est parfois fausse, et rien dans
sa forme ne le signale. On ne peut donc pas juger « à l'œil » : sur cinq
exemples testés à la main, on retient ceux qui marchent, et on développe une
conviction qui n'est pas un résultat.

Pire : sans mesure, on ne peut pas **améliorer**. Vous modifiez un prompt pour
corriger un cas qui échouait. Il passe. Trois autres cas se sont mis à échouer,
vous ne le voyez pas. Vous appelez ça une amélioration. C'est le mode d'échec le
plus courant, et il est invisible sans jeu d'évaluation.

**Un système IA sans jeu d'évaluation ne peut pas progresser. Il peut seulement
donner l'impression de progresser.**

## Le jeu d'évaluation

### Le construire avant

Avant le premier prompt. C'est contre-intuitif, et c'est le point sur lequel
tout le monde triche au début.

La raison est simple : un jeu construit après coup est construit à partir de ce
que le système sait déjà faire. On choisit inconsciemment les cas qui passent.
Le jeu ressemble alors au système, et il valide n'importe quoi.

Construit avant, il représente le besoin réel — indépendamment de ce qu'on
saura en faire.

### Combien de cas

Ces seuils portent sur le jeu **principal**, celui sur lequel vous lisez le
score. La réserve décrite plus bas se prend **en plus**, pas dedans.

- **En dessous de 20** : aucune conclusion possible. Un cas qui bascule fait
  bouger le score de 5 points.
- **30 à 50** : suffisant pour un POC exploratoire, en gardant en tête qu'un
  écart de moins de 10 points entre deux configurations n'est probablement pas
  significatif.
- **100 à 300** : ce qu'il faut pour un système qu'on envisage sérieusement.
- **Au-delà** : rarement nécessaire ici, et coûteux à maintenir.

Concrètement, un POC démarre donc autour de **40 à 65 cas fabriqués** : 30 à 50
pour le jeu principal, une dizaine ou quinze de plus pour la réserve. Prélever
la réserve dans les 30 cas ferait passer le jeu principal sous le seuil des 20,
là où plus rien n'est interprétable.

Mieux vaut 40 cas bien choisis que 400 cas qui se ressemblent tous.

### Ce qu'il doit contenir

Un jeu composé uniquement de cas simples donne 95 % et ne mesure rien. La
composition compte plus que le volume :

| Part | Type de cas |
|---|---|
| ~50 % | Cas nominaux, représentatifs de l'usage courant |
| ~30 % | Cas difficiles : formulations ambiguës, données incomplètes, formats inhabituels |
| ~20 % | **Cas où la bonne réponse est de refuser** : hors périmètre, information absente, question piège |

La dernière catégorie est la plus négligée et la plus révélatrice. Un système
qui répond avec assurance à une question dont la réponse n'existe pas dans ses
données est un système dangereux, et aucun jeu de cas nominaux ne le détecte.

### Avec quelles données

Des données inventées. Jamais de données réelles — voir [DONNEES.md](DONNEES.md).

La contrainte est réelle : pour qu'une mesure vaille, les cas doivent ressembler
au terrain. La méthode est de reproduire la **forme** (structure, longueur,
registre, fautes de frappe, abréviations métier, cas tordus observés) avec un
**contenu entièrement inventé**. C'est du travail. C'est ce qui donne sa valeur
au reste.

### Le jeu de réserve

Fabriquez 20 à 30 % de cas **en plus** du jeu principal, mis de côté, sur
lesquels vous ne regardez jamais le détail des erreurs.

Pourquoi : à force d'ajuster le prompt en regardant les cas qui échouent, on
finit par le spécialiser sur ces cas précis. Le score monte, la performance
réelle non. C'est le même phénomène que le surapprentissage, appliqué au
prompt — et il arrive vite, en une poignée d'itérations.

Le jeu de réserve ne sert qu'à vérifier, ponctuellement, que le score principal
n'est pas devenu une illusion. Si l'écart entre les deux dépasse quelques
points, votre prompt est spécialisé sur vos exemples.

## Les trois axes

On mesure toujours les trois ensemble. Un seul ne veut rien dire.

### Qualité

Choisissez la méthode la moins sophistiquée qui répond à la question.

**Comparaison exacte** — quand la sortie est contrainte : une catégorie, un
montant, un booléen, du JSON. Fiable, gratuit, instantané. À privilégier
systématiquement : concevoir la tâche pour que la sortie soit vérifiable
automatiquement vaut mieux que concevoir une mesure sophistiquée.

**Règles de vérification** — la réponse contient-elle le bon montant, cite-t-elle
la bonne source, respecte-t-elle le format attendu. Suffit dans beaucoup plus de
cas qu'on ne croit.

**Jugement humain** — la référence. Coûteux, mais c'est le seul étalon. Sur
40 cas, une passe de relecture prend une heure. À faire au moins une fois, en
début de projet, pour calibrer tout le reste.

**Jugement par un modèle** — un modèle note les réponses d'un autre. Utile pour
passer à l'échelle, avec trois précautions non négociables :

- Le juge doit être **calibré contre du jugement humain** sur au moins 20 cas.
  Sans ça vous ne mesurez pas la qualité, vous mesurez l'accord entre deux
  modèles — ce qui n'est pas la même chose et peut être élevé alors que les deux
  ont tort.
- Le juge doit noter selon une **grille explicite** à critères binaires, pas une
  note sur 10. « La réponse cite-t-elle une source présente dans le document ?
  oui/non » est mesurable ; « qualité globale : 7 » ne l'est pas.
- Le juge ne doit **pas être le modèle évalué**. Il se note trop bien.

### Coût

En euros pour l'ensemble du jeu, puis ramené à l'unité métier : par facture
traitée, par question posée, par document indexé.

Le coût par unité est ce qui permet de dire si le projet tient : 4 centimes par
document sur 200 documents par jour, c'est environ 2 000 € par an, et ça se
compare à ce que coûte le traitement manuel. Le coût par appel API ne se compare
à rien.

Notez le nombre de jetons en entrée et en sortie, séparément — c'est ce qui
permet de recalculer le coût quand les tarifs changent, sans tout relancer.

### Latence

Le temps de réponse médian **et** le pire cas observé. La médiane rassure, c'est
le pire cas qui décide si l'usage est tenable.

Un système à 2 secondes en médiane mais 25 secondes au 95e centile est
inutilisable en interactif, et parfaitement acceptable en traitement par lots.
La question n'est jamais « est-ce rapide », mais « est-ce compatible avec
l'usage prévu ».

## Le seuil

Écrivez-le avant de mesurer, dans le README :

> Seuil : au moins 85 % de bonnes extractions sur le jeu d'évaluation, dont
> 100 % sur les montants. Coût maximum 3 centimes par document. Latence médiane
> sous 5 secondes.

Sans seuil écrit à l'avance, on ajuste l'exigence au résultat obtenu — 78 %
devient « plutôt encourageant » un lundi et « insuffisant » le jeudi. Le seuil
n'a pas à être juste du premier coup : il a à être écrit, et à être révisé
explicitement quand on le change, en disant pourquoi.

Distinguez deux niveaux : le seuil de **non-régression** (en dessous, la
modification est refusée) et le seuil d'**acceptation** (au-dessus, le système
est utilisable en production).

## Reproductibilité

Un run se rejoue, ou n'existe pas.

### Figer la version du modèle

Le point le plus important de cette page, et le plus souvent raté.

Un identifiant de modèle sans version pointe vers une cible mouvante. Le
fournisseur met à jour, votre score bouge de plusieurs points, vous cherchez
dans votre code ce qui a changé. Rien n'a changé chez vous.

**Toute évaluation utilise un identifiant de modèle figé**, et cet identifiant
figure dans le fichier de run. Un score sans version de modèle n'est pas
interprétable — ni deux mois plus tard, ni par quelqu'un d'autre.

### Le fichier de run

Chaque évaluation produit un fichier versionné dans `evaluations/runs/` :

```json
{
  "date": "2026-09-04",
  "modele": "<identifiant complet, version figée>",
  "parametres": { "temperature": 0, "max_tokens": 1024 },
  "prompt": "prompts/extraction_v3.txt",
  "jeu": "evaluations/jeux/factures_v2.jsonl",
  "nb_cas": 47,
  "resultats": {
    "reussite": 0.87,
    "reussite_montants": 1.0,
    "cout_total_eur": 0.42,
    "cout_par_document_eur": 0.009,
    "latence_mediane_s": 3.1,
    "latence_p95_s": 8.4
  },
  "commentaire": "Les 6 échecs sont tous des factures manuscrites."
}
```

Le champ `commentaire` est celui qu'on relit. Un score seul dit qu'il reste des
erreurs ; il ne dit pas si elles sont concentrées sur un type de cas qu'on
pourrait exclure du périmètre.

`temperature: 0` pour toute évaluation. On ne mesure pas sur du bruit.

## Quand évaluer

- Avant toute modification de prompt, pour avoir le point de départ
- Après, pour vérifier ce qu'elle a réellement produit
- À chaque changement de modèle ou de version, systématiquement
- Avant toute présentation de chiffres à quelqu'un
- Périodiquement sur un système en service : les modèles évoluent sous vous

Un prompt modifié sans évaluation avant/après est une modification dont
personne, y compris son auteur, ne connaît l'effet.

## Erreurs fréquentes

**Comparer deux runs à configurations différentes.** Un seul paramètre change à
la fois. Deux changements simultanés produisent un résultat ininterprétable.

**Conclure sur un écart faible.** Sur 40 cas, 82 % et 85 % ne sont pas
distinguables : c'est un cas de différence. Annoncer une amélioration sur cette
base est une erreur de méthode, pas une approximation.

**Ne regarder que le taux global.** 90 % avec les 10 % d'échecs concentrés sur
le champ le plus critique, c'est un système inutilisable affiché comme un
succès. Découpez par type de cas.

**Oublier les cas de refus.** Voir plus haut : c'est ce qui distingue un système
prudent d'un système qui invente avec assurance.

**Annoncer un chiffre sans sa date ni son nombre de cas.** « On est à 87 % » ne
veut rien dire. « 87 % sur 47 cas, le 4 septembre 2026, avec tel modèle » est
une information.

## Où consigner les résultats

Le fichier de run et le jeu d'évaluation vivent dans le dépôt : ce sont les
sources.

Les enseignements transverses — ce qui marche en général, les modèles écartés,
les ordres de grandeur de coût — se consignent hors du dépôt, dans le carnet de
travail, et sont partagés en Discussion quand ils servent à d'autres.
