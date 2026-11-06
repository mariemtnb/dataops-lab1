# Comptes rendus des TP1, TP2, TP3 et TP4

**Nom :** Mariem Tanabene

## TP1

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

## TP2

**Q1.** Non. Tant que je travaillais sur la branche, `main` n'avait pas la règle 6. Elle est arrivée sur `main` au moment du `git merge --no-ff feature/country-rule`. C'est utile parce que le rapport de la nuit tourne depuis `main` : je peux essayer ma règle sur la branche sans risquer de casser le rapport.

**Q2.** Avec une pull request, quelqu'un d'autre peut relire mon code avant la fusion, et les tests se lancent tout seuls. Un `git merge` sur mon PC ne fait ni l'un ni l'autre.

**Q3.** `md5: 2b3c1607110c06fe4619ef560c7a6356.dir`. C'est l'empreinte du contenu du dossier `data/raw`. Si je change un seul caractère dans un CSV, l'empreinte devient complètement différente, et DVC sait que les données ont changé.

**Q4.** C'est le fichier `data/.gitignore`. Il contient `/raw`, donc Git ne regarde plus ce dossier.

**Q5.** Ils portent le nom de leur empreinte md5. Deux versions du même fichier ont donc deux noms différents et ne s'écrasent jamais.

**Q6.** Le fichier `raw.dvc` ne garde que l'empreinte des données. Quand les données changent, seule l'empreinte change, donc deux lignes suffisent. Mais il faut quand même commiter, sinon cette version des données n'est liée à aucun commit et je ne pourrai pas la retrouver.

**Q7.** Non, le code n'a pas bougé, seules les données ont changé. Le total est passé de 7655.90 (data-v1) à 8835.40 (data-v2).

**Q8.** `git checkout data-v1 -- data/raw.dvc` a seulement remis l'ancien petit fichier pointeur. `dvc checkout` a ensuite remis les vrais CSV de la v1 dans `data/raw`.

**Q9.** Le pointeur dit v1, mais les fichiers sur le disque sont encore ceux de la v2. Je l'ai testé : le pipeline donne toujours 8835.40. C'est dangereux parce qu'il n'y a aucune erreur, et on peut donner un mauvais chiffre en croyant avoir l'ancien.

**Q10.** `git clone` lui a donné le code, l'historique et les fichiers pointeurs, mais pas les CSV. `dvc pull` lui a donné les vrais CSV, pris dans le remote.

**Q11.** Il aurait le pointeur de la v2, mais les fichiers ne seraient pas dans le remote. `dvc pull` ne les trouverait pas, donc il n'aurait pas les nouvelles données et ne pourrait pas obtenir 8835.40.

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

## TP3

**Q1.** Si DuckDB devine les types et qu'un jour une quantité vaut `deux`, soit le chargement plante, soit la colonne change de type sans prévenir. En chargeant tout en texte, le chargement passe toujours et la ligne arrive telle quelle. Ensuite c'est le staging qui met les bons types, et les règles qui rejettent la ligne avec un motif.

**Q2.** Le total naïf vaut 13950.6 en SQL, comme avec l'ancien script Python. Donc ce n'est pas le langage qui rend un pipeline fiable. Ce sont les contrôles : les règles, les tests, et l'arrêt quand quelque chose ne va pas.

**Q3.** Le brut reste intact dans `raw.orders`. Si une règle est fausse, je corrige le SQL et je relance `dbt build`, et tout est recalculé depuis le brut. Je n'ai pas besoin de retourner chercher le fichier d'origine.

**Q4.** Il faudrait mettre `type: snowflake` (ou `bigquery`) à la place de `duckdb`, avec le compte, l'utilisateur, la base et le schéma. Le mot de passe ne doit pas être écrit dans le fichier ni partir sur Git. On le met dans une variable d'environnement, lue avec `{{ env_var('SNOWFLAKE_PASSWORD') }}`, et dans la CI on le garde dans un secret GitHub.

**Q5.** Le staging sert juste à mettre les bons types, il ne décide pas ce qui est bon ou mauvais. Si je supprimais la 1013 ici, elle disparaîtrait sans explication. En la gardant, elle arrive dans `orders_rejected` avec le motif `invalid_date`, et on sait pourquoi elle a été écartée.

**Q6.** Une vue ne garde pas de données, c'est juste une requête enregistrée. Chaque fois qu'on la lit, DuckDB relance la requête sur `raw.orders`. Pour le staging ça va très bien : la requête est simple et rapide, et la vue est toujours à jour avec le dernier fichier chargé.

**Q7.** Par `"warehouse"."main"."stg_orders"`, le vrai nom de la vue dans l'entrepôt.

**Q8.** Elle regroupe les lignes qui ont le même `order_id`, puis les numérote dans l'ordre du fichier. La première ligne prend 1, les suivantes 2, 3, etc. Donc tout ce qui est plus grand que 1 est un doublon, comme avec `keep="first"`. Il faut `_row_number` parce qu'une table SQL n'a pas d'ordre. Sans lui, on ne sait pas laquelle est la première ligne.

**Q9.** Il crée `models/marts/revenue_by_product.sql`, avec un `select` qui fait la somme par produit à partir de `{{ ref('orders_clean') }}`, et il ajoute ses tests dans `models/marts/marts.yml`. Il ne touche à aucun fichier Python.

**Q10.** Quand j'ai fait l'essai à l'étape 5, c'est `unique_orders_clean_order_id` qui a échoué, parce que la commande 1010 était deux fois dans `orders_clean`. Ensuite `revenue_by_country` et son test sont passés en SKIP. C'est ce qu'on veut : le rapport du directeur n'est pas refait avec un doublon, il garde le bon chiffre. J'ai refait l'essai après avoir ajouté les tests unitaires. Cette fois c'est `test_duplicate_order_keeps_the_first` qui échoue en premier, et tout ce qui vient après est en SKIP.

**Q11.**

| Contrôle du TP1 (Python) | Son équivalent dbt |
|---|---|
| `check_schema` | La vue `stg_orders` ne se crée pas s'il manque une colonne, plus les tests `not_null` et `accepted_values` |
| Les règles 1 à 6 dans `clean_orders` | Le `case when` dans `int_orders_checked.sql` |
| `check_reject_rate` | Le test `assert_reject_rate_below_20_percent.sql` |
| `rejected_orders.csv` | La table `orders_rejected` |
| `PIPELINE STOPPED`, rien n'est publié | `dbt build` met en SKIP tout ce qui suit un test raté, et sort avec le code 1 |

**Q12.** Elles venaient du fichier YAML : ce sont les lignes que j'ai écrites dans `given`. dbt les utilise à la place de `stg_orders`. Pour la CI c'est important, parce que les CSV sont sous DVC et ne sont pas sur GitHub. Le test marche quand même, il est rapide et donne toujours le même résultat.

**Q13.** Tout ce qui vient après la source est touché : `stg_orders`, `int_orders_checked`, `orders_clean`, `orders_rejected` et `revenue_by_country`, avec leurs tests. Avec le graphe, je le vois en quelques secondes. Avec 200 scripts SQL et pas de graphe, il faudrait ouvrir les fichiers un par un pour voir qui lit quoi. Ça prendrait des heures, et j'en oublierais sûrement.

**Q14.**

| | Pipeline Python (TP1) | Pipeline dbt (TP3) |
|---|---|---|
| Jour 2 : quel contrôle arrête la chaîne ? | `check_reject_rate` (100 % de rejets, 6 lignes sur 6) | Le test `assert_reject_rate_below_20_percent` |
| Jour 3 : quel message d'erreur ? | `PIPELINE STOPPED: Missing columns: ['unit_price']` | `Table "src" does not have a column named "unit_price"`, `Candidate bindings: "price"` |
| Le rapport de la veille est-il conservé ? | Oui | Oui, toujours 8835.40 |
| Code de sortie | 1 | 1 |

**Q15.** Oui, c'est gênant. La quarantaine montre encore les rejets du jour 1, alors que ce sont les 6 lignes du jour 2 qu'il faudrait regarder. Le pipeline Python, lui, écrivait les rejets avant de vérifier le taux. Pour faire pareil avec dbt, on peut écrire le test du taux de rejet à partir de `orders_rejected`. Comme ça, dbt construit d'abord la quarantaine, puis lance le test, et seul le rapport est bloqué.

### Dernière ligne de `dbt build` (jour 1)

Done. PASS=14 WARN=0 ERROR=0 SKIP=0 NO-OP=0 REUSED=0 TOTAL=14

## TP4

### Les deux liens à rendre

- Exécution verte du workflow CI sur `main` : https://github.com/mariemtnb/dataops-lab1/actions/runs/42203506312
- Exécution manuelle du workflow Nightly sur `main` : https://github.com/mariemtnb/dataops-lab1/actions/runs/42203662179

### Ce que j'ai vu

**Étape 0.** La CI du TP1 était rouge avec `FileNotFoundError: data/raw/orders_day1.csv`. Depuis le TP2 les CSV sont sous DVC, et le remote DVC est un dossier sur mon PC, donc GitHub ne peut pas les récupérer. Personne n'avait touché au code, ce sont les données qui avaient changé de place.

**Étape 1.** `ruff check .` et `python -m pytest` passent (6 tests). Avec `import os` en trop, ruff affiche `F401`. Ma version de ruff est plus récente que celle de la fiche et vérifie aussi l'ordre des imports, donc j'ai dû ajouter une ligne vide dans trois anciens fichiers. DuckDB est passé en 1.5.6, j'ai reconstruit l'entrepôt et `dbt build` donne `PASS=14`.

**Étape 2.** Le contrat est valide sur l'échantillon et sur le jour 2 (13 contrôles). Le jour 3 est refusé avec `Required column 'unit_price' is missing` et le code de sortie 1. Le jour 2 passe parce que le contrat regarde la forme du fichier, pas les montants.

**Étape 3.** Pandera charge le jour 1 (35 lignes, code 0). Il refuse le jour 2 (`more than 20 % of the prices are implausible`) et le jour 3 (`price` en trop, `unit_price` manquante), avec le code 1. Après ces deux refus, `raw.orders` contient toujours le jour 1.

**Étape 4.** `report_md.py 8835.40` affiche les 5 pays et sort avec 0. `report_md.py 9999` affiche `REGRESSION` et sort avec 1.

**Étape 5.** Sur la pull request #1, les trois jobs sont passés au vert dans l'ordre lint, python-tests, data-quality. Le résumé affiche le tableau avec `Total: 8835.40`, et l'artefact `dbt-results` contient `run_results.json`. Après la fusion, la CI sur `main` est verte.

**Étape 6.** Pull request #2 fusionnée, puis lancement à la main de Nightly avec le bouton Run workflow. L'exécution est verte et l'artefact `dbt-docs` contient la documentation.

**Étape 7.** Pull request #3 (`unit_price` renommée en `price`) : lint et python-tests verts, data-quality rouge à l'étape du contrat avec `Required column 'unit_price' is missing`. GitHub affiche « Some checks were not successful ». Je l'ai fermée sans fusionner.

**Étape 8.** Pull request #4 (quantités jusqu'à -5) : lint vert, python-tests rouge avec `test_negative_quantity_is_rejected`, et data-quality ne s'est pas lancé. Je l'ai fermée sans fusionner.

