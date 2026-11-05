# Indicateurs de mesure de la performance au long du projet

Les indicateurs s'organisent autour de quatre axes. Ils ont été définis **en début de projet**, avant tout entraînement : on ne connaît alors ni le niveau de signal des données ni ce qu'un modèle peut raisonnablement atteindre. Aucune cible chiffrée de performance n'est donc fixée a priori : on se donne un **objectif d'orientation**, une **référence de comparaison** et un **moment où la cible sera fixée**.

## 1. Dataset d'entraînement

| Indicateur | Valeur attendue |
|---|---|
| Articles couverts | Toute la base article exploitable : ~1 300 articles chez Sodexma, ~500 chez Friand |
| Nombre de semaines | 52 semaines pour l'entraînement initial, pour prendre en compte la saisonnalité |
| Semaines minimum en réentraînement | Au moins 45 semaines, si des batchs sont corrompus ou incomplets |

## 2. Modèle

### Objectif

**Maximiser la précision** : parmi les articles que l'outil propose, la plus grande part possible doit être de vraies décisions d'achat. Une fausse recommandation fait perdre du temps au category manager et dégrade sa confiance dans l'outil ; une décision manquée est moins coûteuse (le CM continue de décider par ses propres moyens).

### Pourquoi pas de cible chiffrée au départ

Les décisions d'achat sont rares et irrégulières ; on ne peut pas deviner à l'avance un taux de précision réaliste. Fixer un pourcentage arbitraire (par exemple 80 %) risquerait de rendre le projet « échoué » sur un chiffre sans fondement, ou de masquer un modèle inutile derrière un objectif trop bas.

### Indicateurs suivis

| Indicateur | Rôle | Cible |
|---|---|---|
| **Précision** | Objectif principal : part de vraies décisions parmi les alertes | À maximiser ; cible chiffrée à fixer après le premier entraînement |
| Précision de référence (hasard) | Comparaison : un modèle au hasard a une précision égale à la prévalence (taux d'achat de la base) | À battre nettement |
| **Lift** | Précision (ou PR-AUC) rapportée à la référence : combien de fois mieux que le hasard | Strictement supérieur à 1, à maximiser |
| **Rappel** | Garde-fou : part des décisions réelles détectées. Empêche de gagner en précision en ne signalant presque rien | Plancher à définir avec les CM |
| **Alertes par semaine** | Charge pour l'acheteur : nombre d'articles à examiner | Compatible avec le temps disponible des CM |
| PR-AUC | Qualité globale du classement, indépendante du seuil ; sert à comparer les modèles entre eux | À maximiser |

### Comment la cible sera fixée

1. **Référence** : mesurer la prévalence et la précision d'un modèle au hasard.
2. **Premier entraînement** : mesurer précision, rappel et alertes par semaine sur des semaines mises de côté (jamais utilisées pour régler le modèle).
3. **Revue avec les CM** : présenter le compromis précision / rappel / nombre d'alertes, et fixer avec eux le plancher de rappel et la charge acceptable.
4. **Cible de précision** : fixée à partir de ce résultat et révisée à chaque réentraînement.
5. **Garde-fou permanent** : un modèle qui ne bat pas le hasard n'est jamais mis en production, et un nouveau modèle n'en remplace un autre que s'il prédit mieux que lui sur des semaines inédites.

### Suivi dans le temps

La précision et le rappel **réels** sont mesurés chaque semaine en comparant les alertes aux achats effectivement décidés. On suit leur tendance, et non une semaine isolée : le nombre de décisions varie beaucoup d'une semaine à l'autre. Une baisse durable ou une dérive des variables d'entrée déclenche un réentraînement.

## 3. Utilisation

| Indicateur | Valeur attendue |
|---|---|
| Utilisateurs différents par semaine | Égal au nombre de category managers impliqués dans les achats |
| Taux de lignes examinées | À suivre (part des alertes effectivement affichées et traitées) |
| Taux de décisions renseignées | À suivre (validation ou infirmation par les CM) |

## 4. Flux

| Indicateur | Valeur attendue |
|---|---|
| DAG Airflow | 100 % des DAG réussis |
| Batch hebdomadaire disponible | Chaque semaine, dans les délais |
| Fraîcheur des données | Copie de l'ERP à jour pour chaque société ; sinon batch marqué « données incomplètes » |
