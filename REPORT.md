# Comptes rendus – TP1, TP2 et TP3

**Nom :** Mariem Tanabene

---

# TP1

**Q1.** Le fichier `requirements.txt` garde la liste des bibliothèques et de leurs versions. Avec une seule commande, n'importe qui peut refaire le même environnement sur son PC.

**Q2.** Le rapport affiche 8 lignes au lieu de 5. Le script croit que `tunisia`, `MOROCCO` et ` Tunisia` (avec un espace) sont des pays différents.

**Q3.**

| order_id | Problème | Dimension |
|---|---|---|
| 1004 | quantité négative | Validité |
| 1008 | client manquant | Complétude |
| 1013 | date impossible (2026-13-45) | Validité |
| 1010 | commande en double | Unicité |
| 1006, 1016, 1023 | pays écrit de plusieurs façons | Cohérence |
| 1019 | prix de 4500 au lieu de 45 | Exactitude |

**Q4.** Le total est trop haut (surtout à cause du prix de 4500 et du doublon). Mais on ne peut pas le savoir sans recalculer, car certaines erreurs l'augmentent et d'autres le diminuent.

**Q5.** Avantages des fonctions : on peut tester chaque étape seule, et on trouve plus vite d'où vient un problème.

**Q6.** Le bon total est 7655.90. L'ancien script donnait 12771.10, donc 5115.20 de trop, soit environ 67 % d'erreur.

**Q7.** La commande 1019 (prix de 4500). Elle a été rejetée par la règle 4, `invalid_unit_price`.

**Q8.** On garde les lignes rejetées pour savoir ce qui a été écarté et pourquoi, et pour pouvoir les corriger plus tard. Si on les supprime, on perd des données sans le savoir.

**Q9.** Avec `> -5`, la quantité -2 n'est plus rejetée et le test `test_negative_quantity_is_rejected` échoue. Quand j'ai remis `> 0`, les 5 tests sont repassés.

**Q10.** Un test unitaire vérifie que mon code est correct. Un data check vérifie que les données du jour sont bonnes.

**Q11.**

| | Ancien script | Pipeline |
|---|---|---|
| Jour 2 : erreur ? | Non | Oui : `PIPELINE STOPPED`, 100 % de lignes rejetées |
| Jour 2 : total | 270200.0 (faux) | Rien n'est publié |
| Jour 3 : erreur ? | Oui, il plante : `KeyError: 'unit_price'` | `PIPELINE STOPPED: Missing columns: ['unit_price']` |

**Q12.** Jour 2 : les prix sont multipliés par 100 (449 devient 44900). Jour 3 : la colonne `unit_price` s'appelle maintenant `price`.

**Q13.** Le jour 2 est le plus dangereux : le script ne plante pas et donne un total faux qui a l'air normal. Le jour 3, il plante, donc on voit le problème.

**Q14.** Un planificateur (cron, Airflow), un outil de CI comme GitHub Actions, ou un outil de monitoring. Ils lisent le code de sortie pour savoir si le job a réussi et envoyer une alerte si besoin.

---

# TP2

**Q1.** Non, `main` n'avait pas la règle 6 pendant que je travaillais sur la branche. Elle est arrivée sur `main` avec la commande `git merge --no-ff feature/country-rule`. C'est utile parce que le rapport tourne depuis `main` : on teste d'abord sur la branche, sans risquer de casser le rapport.

**Q2.** Une pull request permet à quelqu'un de relire le code avant la fusion, et de lancer les tests automatiquement. Un `git merge` local ne fait ni l'un ni l'autre.

**Q3.** `md5: 2b3c1607110c06fe4619ef560c7a6356.dir`. C'est l'empreinte du contenu du dossier `data/raw`. Si je change un seul caractère dans un CSV, l'empreinte change complètement, et DVC voit que les données ont été modifiées.

**Q4.** C'est le fichier `data/.gitignore` (il contient `/raw`) qui dit à Git d'ignorer les CSV.

**Q5.** Les fichiers de `dvc-storage` portent le nom de leur empreinte md5. Comme ça, deux versions d'un même fichier ne s'écrasent jamais.

**Q6.** Le fichier `raw.dvc` ne contient que l'empreinte, donc quand les données changent, seules deux lignes changent. Il faut quand même commiter, sinon cette version n'est liée à aucun commit et on ne pourra pas la retrouver.

**Q7.** Non, le code n'a pas changé, seulement les données. Le total est passé de 7655.90 (data-v1) à 8835.40 (data-v2).

**Q8.** `git checkout data-v1 -- data/raw.dvc` a remplacé seulement le petit fichier pointeur par celui de la v1. `dvc checkout` a ensuite remis les vrais CSV de la v1 dans `data/raw`.

**Q9.** Si on oublie `dvc checkout`, le pointeur dit « v1 » mais les fichiers sont encore ceux de la v2. J'ai testé : le pipeline affiche toujours 8835.40. C'est dangereux car il n'y a pas d'erreur, et on peut donner un mauvais chiffre en pensant avoir l'ancien.

**Q10.** `git clone` a donné au collègue le code, l'historique et les fichiers pointeurs, mais pas les CSV. `dvc pull` a téléchargé les vrais CSV depuis le remote.

**Q11.** Si j'avais oublié `dvc push`, le collègue aurait le pointeur de la v2 mais pas les données. `dvc pull` ne trouverait pas les fichiers, donc il n'aurait pas le total 8835.40.

### Historique Git

```
* da457e8 (HEAD -> main, tag: data-v2) Data v2: late orders for 2026-10-05
* 4f42ae5 Add the DVC remote
* 9b9e8a7 (tag: data-v1) Track raw data with DVC
*   94526f2 Merge the country rule
|\
| * 5be5ce3 (feature/country-rule) Reject orders from countries the shop does not sell to
|/
* 131cdd1 Finish TP1, add DVC to requirements
* 9bc147a (origin/main) Add pipeline, data checks and unit tests
* c2a23a8 Add pipeline, data checks and unit tests
* 2e059d5 Empty project with requirements
```

---

## TP3

**Q1.** Si on laisse DuckDB deviner, il choisit un type en regardant les valeurs. Le jour où une quantité vaut `deux`, soit le chargement plante, soit DuckDB passe toute la colonne en texte sans prévenir, et la ligne fautive est perdue ou mal lue. En chargeant tout en `VARCHAR`, le chargement ne plante jamais et la ligne arrive telle quelle. C'est la couche staging qui choisit les types (avec `try_cast`), et les règles de qualité qui rejettent la ligne avec un motif clair.

**Q2.** Le total naïf vaut 13950.6 en SQL comme en Python. Le langage ne rend pas un pipeline fiable. Ce qui le rend fiable, ce sont les contrôles : les règles de qualité, les tests et l'arrêt de la chaîne quand quelque chose ne va pas.

**Q3.** Le brut est toujours dans `raw.orders`, intact. Si une règle est fausse, on corrige le fichier SQL et on relance `dbt build` : toutes les tables sont recalculées à partir du brut, sans retourner chercher le fichier source. En ETL, la donnée d'origine aurait déjà été transformée avant d'arriver dans l'entrepôt.

**Q4.** On remplacerait `type: duckdb` et `path` par `type: snowflake` (ou `bigquery`), avec le compte, l'utilisateur, la base, le schéma et le warehouse. Le mot de passe ne doit jamais être écrit dans le fichier ni commité dans Git. On le lit depuis une variable d'environnement avec `password: "{{ env_var('SNOWFLAKE_PASSWORD') }}"`, et dans la CI on le met dans un secret (GitHub Actions secrets). On peut aussi garder `profiles.yml` hors du dépôt, dans `~/.dbt/`.

**Q5.** Le staging ne fait que typer et réparer, il ne décide pas ce qui est valide. Si on supprimait la ligne 1013 ici, elle disparaîtrait sans motif et n'arriverait jamais dans la quarantaine `orders_rejected`. En la gardant, la couche intermediate peut la rejeter avec le motif `invalid_date`, et on sait ce qui a été écarté et pourquoi.

**Q6.** Une vue ne stocke pas de données : c'est une requête enregistrée. À chaque lecture, DuckDB relance le `select` sur `raw.orders`. C'est acceptable ici car le staging est simple et rapide, il n'est lu que par le modèle suivant, et il est toujours à jour avec le dernier brut chargé. On n'a pas besoin de stocker une copie du brut juste avec d'autres types.

**Q7.** dbt a remplacé `{{ ref('stg_orders') }}` par le nom complet de la relation : `"warehouse"."main"."stg_orders"` (base `warehouse`, schéma `main`, vue `stg_orders`).

**Q8.** `row_number() over (partition by order_id order by _row_number)` découpe les lignes en groupes de même `order_id`, puis numérote les lignes de chaque groupe dans l'ordre du fichier : 1, 2, 3… La première ligne d'un `order_id` a le numéro 1, les suivantes ont 2 ou plus, donc `> 1` marque les doublons, comme `keep="first"`. On a besoin de `_row_number` car une table SQL n'a pas d'ordre : sans lui, « la première ligne » n'a pas de sens et le résultat pourrait changer d'une exécution à l'autre. Pandas, lui, garde l'ordre du fichier dans l'index.

**Q9.** Il crée un fichier `models/marts/revenue_by_product.sql` (un `select product, round(sum(quantity * unit_price), 2) as revenue from {{ ref('orders_clean') }} group by product`) et ajoute ses tests dans `models/marts/marts.yml`. Il ne touche à aucun fichier Python : dbt trouve seul sa place dans le graphe grâce au `ref()`.

**Q10.** Pendant l'étape 5 (sans test unitaire), c'est `unique_orders_clean_order_id` qui a échoué (FAIL 1) : la commande 1010 apparaissait deux fois dans `orders_clean`. Ensuite `revenue_by_country` et son test `unique_revenue_by_country_country` sont passés en SKIP (`PASS=9 ERROR=1 SKIP=2`). C'est ce qu'on veut : le rapport du directeur n'a pas été reconstruit avec le doublon, il garde le dernier bon chiffre. Après avoir ajouté les tests unitaires, j'ai refait l'essai : cette fois c'est le test unitaire `test_duplicate_order_keeps_the_first` qui échoue en premier, et `int_orders_checked` et tout ce qui suit sont en SKIP (`PASS=1 ERROR=1 SKIP=12`). L'erreur est arrêtée encore plus tôt, avant même de lire les vraies données.

**Q11.**

| Contrôle du TP1 (Python) | Son équivalent dbt |
|---|---|
| `check_schema` | La création de la vue `stg_orders` : si une colonne manque, elle échoue (`Binder Error`) ; plus les tests `not_null` / `accepted_values` sur les colonnes |
| Les règles 1 à 6 dans `clean_orders` | Le `case when` de `int_orders_checked.sql`, colonne `reject_reason` |
| `check_reject_rate` | Le test singulier `assert_reject_rate_below_20_percent.sql` |
| `rejected_orders.csv` | La table `orders_rejected` |
| `PIPELINE STOPPED`, rien n'est publié | `dbt build` : un test qui échoue met tous les modèles suivants en SKIP, code de sortie 1 |

**Q12.** Les données venaient du fichier YAML lui-même : les lignes écrites à la main dans `given`. dbt remplace `ref('stg_orders')` par ces lignes, sans lire l'entrepôt. C'est important pour la CI : le test n'a besoin ni des CSV (qui sont sous DVC, donc pas dans le dépôt Git), ni d'un entrepôt rempli. Il est rapide, donne toujours le même résultat, et teste la logique SQL, pas les données du jour.

**Q13.** Tous les descendants de la source `raw.orders` sont touchés : `stg_orders`, `int_orders_checked`, `orders_clean`, `orders_rejected`, `revenue_by_country`, avec tous leurs tests. Avec le graphe, la réponse prend quelques secondes (`source:raw.orders+`). Sans graphe, dans 200 scripts SQL, il faudrait chercher à la main quels fichiers lisent quelles tables, puis recommencer pour chaque table trouvée : des heures, avec un vrai risque d'en oublier.

**Q14.**

| | Pipeline Python (TP1) | Pipeline dbt (TP3) |
|---|---|---|
| Jour 2 : quel contrôle arrête la chaîne ? | `check_reject_rate` : `PIPELINE STOPPED: Reject rate 100% is above the limit of 20% (6 of 6 rows)` | Le test `assert_reject_rate_below_20_percent` (FAIL 1), puis les marts et leurs tests en SKIP |
| Jour 3 : quel message d'erreur ? | `PIPELINE STOPPED: Missing columns: ['unit_price']` | `ERROR creating sql view model main.stg_orders` : `Binder Error: Table "src" does not have a column named "unit_price"`, `Candidate bindings: "price"`, puis 13 étapes en SKIP |
| Le rapport de la veille est-il conservé ? | Oui, `revenue_by_country.csv` n'est pas réécrit | Oui, la table `revenue_by_country` n'est pas reconstruite : toujours 8835.40 |
| Code de sortie | 1 | 1 |

**Q15.** Oui, c'est un problème. La quarantaine doit dire « voici ce qui a été rejeté aujourd'hui », mais au jour 2 elle montre les rejets du jour 1. On ne voit pas les 6 lignes du jour 2 (prix en centimes), alors que ce sont justement celles qu'il faut analyser. Le pipeline Python, lui, écrivait `rejected_orders.csv` avant de contrôler le taux. Solution : faire tourner le test du taux de rejet après `orders_rejected`, en l'écrivant sur `{{ ref('orders_rejected') }}` (nombre de rejets) et `{{ ref('stg_orders') }}` (nombre total). dbt construit alors d'abord la quarantaine, puis lance le test, et seuls `orders_clean` et `revenue_by_country` sont bloqués. On peut aussi lancer `dbt build --select orders_rejected` avant le `dbt build` complet, ou rendre `orders_rejected` incrémental pour garder l'historique des rejets de chaque jour (grâce à `_source_file` et `_loaded_at`).

### Dernière ligne de `dbt build` (jour 1)

```
Done. PASS=14 WARN=0 ERROR=0 SKIP=0 NO-OP=0 REUSED=0 TOTAL=14
```