# Comptes rendus – TP1 et TP2

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