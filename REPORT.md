# TP 1 – D'un script fragile à un pipeline testé

**Noms :** Mariem Tanabene

---

## Étape 0

**Q1. Pourquoi écrire les bibliothèques dans `requirements.txt` plutôt que de taper `pip install pandas` ?**

Parce que le fichier garde la liste exacte des bibliothèques et de leurs versions. Quelqu'un d'autre (ou moi, sur un autre PC) peut refaire le même environnement avec une seule commande, et on évite le classique « chez moi ça marche ».

---

## Étape 1

**Q2. Combien de lignes le rapport affiche-t-il ? Pourquoi ?**

Il affiche 8 lignes alors que la boutique ne vend que dans 5 pays. Le script voit `tunisia`, `MOROCCO` et ` Tunisia` (avec un espace) comme des pays à part, donc il les sépare de `Tunisia` et `Morocco`.

**Q3. Les problèmes trouvés dans le fichier**

| order_id | Problème | Dimension |
|---|---|---|
| 1004 | quantité négative (-2) | Validité |
| 1008 | `customer_id` vide | Complétude |
| 1013 | date impossible (2026-13-45) | Validité |
| 1010 | la commande apparaît deux fois | Unicité |
| 1006, 1016, 1023 | pays écrit `tunisia`, `MOROCCO`, ` Tunisia` | Cohérence |
| 1019 | clavier à 4500 au lieu de 45 | Exactitude |

**Q4. Le total est-il trop haut ou trop bas ? Peut-on le savoir sans recalculer ?**

Il est trop haut, à cause surtout du prix de 4500 et du doublon. Mais sans recalculer, on ne peut pas être sûr : certaines erreurs le gonflent (prix, doublon) et d'autres le baissent (la quantité négative). Elles se mélangent, donc on ne voit rien à l'œil nu.

---

## Étape 2

**Q5. Qu'a-t-on gagné en découpant le script en fonctions ?**

1. Chaque étape peut être testée toute seule, avec un petit tableau.
2. Quand quelque chose ne va pas, on sait dans quelle étape chercher, et on peut modifier une étape sans casser les autres.

---

## Étape 3

**Q6. Quel est le bon total ? De combien l'ancien script se trompait-il ?**

Le bon total est **7655.90**. L'ancien script affichait 12771.10, soit 5115.20 de trop, ce qui fait environ **67 %** d'erreur.

**Q7. Quelle commande explique l'essentiel de l'erreur ? Quelle règle l'a attrapée ?**

La commande **1019** (un clavier à 4500 au lieu de 45). C'est la règle 4, `invalid_unit_price`, qui l'a rejetée.

**Q8. Pourquoi garder les lignes rejetées dans un fichier au lieu de les supprimer ?**

Pour pouvoir savoir ce qui a été écarté et pourquoi, prévenir la source pour qu'elle corrige, et retraiter les lignes plus tard si besoin. Si on les supprime sans trace, on perd des données en silence, ce qui est le problème qu'on essaie justement de résoudre.

---

## Étape 4

**Q9. Que s'est-il passé quand on a cassé la règle 3 ?**

En remplaçant `> 0` par `> -5`, la quantité -2 n'est plus rejetée. Le test `test_negative_quantity_is_rejected` a échoué, car on obtenait deux lignes propres au lieu d'une. Après avoir remis `> 0`, les 5 tests passent de nouveau.

**Q10. Ce que vérifie un test unitaire, et ce que vérifie un data check**

- Un test unitaire vérifie que **mon code** fait bien ce qu'il doit faire, sur un petit tableau écrit à la main.
- Un data check vérifie que **les données du jour** sont acceptables, à chaque exécution du pipeline.

---

## Étape 6

**Q11. Comparaison entre l'ancien script et le pipeline**

| | Ancien script | Pipeline |
|---|---|---|
| Jour 2 : message d'erreur ? | Non, aucune erreur | Oui : `PIPELINE STOPPED: Reject rate 100% is above the limit of 20% (6 of 6 rows rejected)` |
| Jour 2 : total affiché | 270200.0 (faux) | Aucun, rien n'est publié |
| Jour 3 : message d'erreur ? | Plante avec `KeyError: 'unit_price'` | `PIPELINE STOPPED: Missing columns: ['unit_price']` |

**Q12. Qu'est-ce qui a changé dans le système source ?**

- **Jour 2 :** les prix sont multipliés par 100 (449 est devenu 44900). Ça ressemble à un passage des euros aux centimes.
- **Jour 3 :** la colonne `unit_price` a été renommée en `price`.

**Q13. Sans pipeline, quel jour est le plus dangereux ?**

Le jour 2. Le script ne plante pas et sort un total 100 fois trop grand mais qui a l'air normal, donc il peut être envoyé au directeur commercial sans que personne ne s'en rende compte. Le jour 3, le script plante, donc au moins on voit qu'il y a un problème.

**Q14. Qui ou quoi peut utiliser le code de sortie 1 ?**

Un planificateur ou orchestrateur (cron, Airflow…), un outil d'intégration continue comme GitHub Actions, ou un système de monitoring. Ils lisent ce chiffre pour savoir si le job a réussi, arrêter les étapes suivantes ou envoyer une alerte.
