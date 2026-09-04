# Règles données

**À lire avant votre premier commit.** C'est le document le plus important de
l'organisation : une erreur de code se corrige, une donnée publiée ne se
dépublie pas.

## La règle

**Aucune donnée réelle dans un dépôt.** Ni fichier source, ni extrait, ni
échantillon, ni copie « juste pour tester ». Même anonymisée. Même dans une
branche. Même temporairement.

Un dépôt est public ou le deviendra, sera cloné, sauvegardé, indexé. Ce qui y
entre ne se retire pas : effacer un fichier ne l'efface pas de l'historique
Git, et l'historique a déjà été copié chez tous ceux qui ont cloné.

## Pourquoi « je l'ai anonymisé » ne suffit pas

C'est l'erreur la plus fréquente, et elle est faite de bonne foi.

**La ré-identification.** Retirer le nom ne suffit pas. Un code postal, une date
de naissance et un métier suffisent souvent à désigner une personne unique. Sur
un jeu métier, la combinaison de trois colonnes anodines identifie parfois une
seule ligne possible.

**Les champs libres.** Vous avez remplacé la colonne `nom`. Mais la colonne
`commentaire` contient « rappelé M. Martin au 06… ». Les zones de saisie libre
contiennent toujours ce qu'on n'y attendait pas.

**Les résidus.** Un fichier Excel garde ses onglets masqués, ses cellules
supprimées et son historique de révisions. Un PDF « caviardé » garde son texte
sous le rectangle noir. Un Parquet garde des statistiques par colonne (min, max)
qui parlent.

**Le nom du fichier.** `export_facturation_[client]_2026.csv` renseigne déjà sur
un client, avant même qu'on l'ouvre. Idem pour les noms de branches et les
messages de commit.

Conclusion : on ne cherche pas à anonymiser correctement. On ne met pas la
donnée réelle dans le dépôt, point.

## Ce qui est autorisé

- **Des jeux synthétiques**, fabriqués à la main ou générés. Le script de
  génération est commité, pas seulement son résultat.
- **Des jeux publics** de référence, avec leur source et leur licence citées.
- **Des exemples inventés**, écrits par vous, qui ressemblent à la structure
  réelle sans en reprendre le contenu.

Pour un jeu d'évaluation, c'est le point sensible : il faut que les cas
ressemblent au réel pour que la mesure vaille quelque chose. La bonne méthode
est de reproduire la **forme** (structure, longueur, style, cas tordus,
fautes de frappe) avec un **contenu inventé**. C'est plus long. C'est le prix.

## Où vivent les données réelles

Hors dépôt, sur le poste ou le stockage prévu, à un chemin passé par variable
d'environnement :

```python
CHEMIN_DONNEES = os.environ["CHEMIN_DONNEES"]  # jamais de chemin en dur
```

Et dans `.gitignore`, dès le premier commit — avant même d'avoir des données :

```gitignore
data/
*.csv
*.parquet
*.xlsx
.env
```

Mettre le `.gitignore` en place **avant** est ce qui évite l'accident. Après,
il est trop tard : le fichier est déjà dans l'historique.

## Les trois pièges spécifiques à l'IA

Ceux-là ne figurent dans aucune règle de dev classique, et ce sont les plus
fréquents ici.

### 1. Les sorties de notebook

Un `.ipynb` commité contient les **résultats affichés** des cellules. Le
`df.head()` que vous avez lancé sur les vraies données est dans le fichier,
en clair, même si vous avez ensuite modifié le code.

Videz les sorties avant de commiter :

```bash
uv run jupyter nbconvert --clear-output --inplace mon_notebook.ipynb
```

Mieux : sortir les notebooks du dépôt une fois l'exploration terminée. Un
notebook n'est pas un livrable, c'est un brouillon.

### 2. Les traces et les logs

Loguer un prompt, c'est loguer la donnée qu'il contient. Un fichier de trace
d'un système RAG contient les extraits de documents récupérés — donc le contenu
métier, intégralement.

Les fichiers de traces ne sont jamais commités. Si une trace est utile à
partager, elle est reconstruite sur des données inventées.

### 3. Ce qui part chez le fournisseur de modèle

Tout ce qu'on envoie dans un appel API sort de l'infrastructure. Avant le
premier appel sur des données qui ne sont pas les vôtres, vérifier :

- la politique de rétention du fournisseur, et sa durée
- si les données sont utilisées pour de l'entraînement (et comment désactiver)
- où sont situés les serveurs, pour la question RGPD
- ce que dit le contrat, s'il y en a un

Ces réponses ont une date de péremption : elles changent avec les conditions
d'utilisation. On les revérifie, on ne les suppose pas.

**Aucun nom de client** ne figure dans un dépôt, y compris dans les
commentaires, les noms de branche et les messages de commit. On écrit « le
client », « un groupe de distribution ».

## Avant le premier commit

- [ ] `.gitignore` en place, avec `data/`, `.env` et les extensions de fichiers
- [ ] `git status` ne montre aucun fichier de données
- [ ] Aucun chemin absolu vers un partage réseau dans le code
- [ ] Notebooks vidés de leurs sorties
- [ ] Aucun nom de client, nulle part
- [ ] Aucune clé API, même expirée, même en commentaire

## Si une donnée a fuité

Ne pas essayer de réparer discrètement. Un `git push --force` mal fait aggrave
la situation et fait perdre la trace de ce qui s'est passé.

1. **Prévenir Florian immédiatement.** Avant toute manipulation.
2. Ne pas supprimer le dépôt — on perdrait la trace de l'étendue de la fuite.
3. Noter l'heure du commit fautif et ce qu'il contenait exactement.
4. La suite (réécriture d'historique, notification, mesures RGPD) se décide
   ensemble, pas dans la panique.

Signaler une fuite qu'on a soi-même causée n'est jamais reproché. La cacher,
si.
