Ce glossaire vise à établir le vocabulaire et les correspondances entre des termes utilisés dans le suivi de projet

## Métier

- **Category manager** : personne qui prend la décision d'achat. C'est le public cible pour l'utilisation de l'algorithme mis en place (= acheteur, décisionnaire).
- **Décision d'achat** : ligne d'achat (demande, préparation, commande, livraison, facture) de quantité positive, valorisée, hors avoirs fournisseurs, datée par sa date de demande. C'est ce que l'algorithme cherche à prédire.
- **Batch** : fichier produit chaque semaine, contenant tous les articles du catalogue avec leurs features, leur score et leur alerte éventuelle.
- **Alerte** : article dont le score dépasse le seuil de décision ; il est proposé à l'acheteur pour examen.

## Données

- **Coupure** : date de référence d'une semaine cible, le lundi 00:00 de cette semaine. Les features n'utilisent que des faits datés avant la coupure ; la cible porte sur les décisions de la semaine qui suit. Une ligne du dataset est un article × une coupure. Dans la documentation, « batch » et « coupure » désignent parfois la même semaine.
- **Feature (variable)** : donnée d'entrée du modèle, calculée pour un article à une coupure (ventes, stock, délai…).
- **Cible** : ce que l'on prédit, ici `achat` (1 si l'article a au moins une décision d'achat dans la semaine, 0 sinon).
- **Prévalence** : part de lignes positives dans les données. Ici environ 3,4 % : un article sur trente est acheté une semaine donnée. Un modèle au hasard a une précision égale à la prévalence.
- **Semaine ISO** : semaine du lundi au dimanche, numérotée selon la norme ISO 8601.

## Apprentissage

- **Régression logistique** : modèle qui combine les features par une somme pondérée et la transforme en un score entre 0 et 1.
- **Pondération des classes (`class_weight="balanced"`)** : chaque classe est pondérée par l'inverse de sa fréquence, pour que le modèle ne néglige pas la classe rare. Conséquence : le score n'est pas une probabilité calibrée, c'est un score de classement.
- **Régularisation (C)** : réglage qui limite l'amplitude des coefficients du modèle pour éviter qu'il n'apprenne le bruit. Un C petit régularise fortement, un C grand peu.
- **Standardisation** : mise à la même échelle des variables (moyenne nulle, écart-type 1).
- **arcsinh** : transformation proche du logarithme qui écrase les valeurs extrêmes, tout en acceptant zéro et les valeurs négatives.
- **Holdout** : lot de données (ici les 8 dernières semaines) mis de côté, jamais utilisé pour régler le modèle, qui sert à une évaluation honnête.
- **Plis ou folds** : découpes successives des données pour la validation croisée. Ici 5 plis temporels à fenêtre croissante : on apprend sur le passé, on valide sur les semaines suivantes. Un pli à une seule classe est ignoré.
- **Validation croisée** : technique qui répète l'apprentissage et l'évaluation sur plusieurs découpes pour choisir les réglages sans toucher au holdout.
- **Surapprentissage** : le modèle apprend le détail des données d'entraînement au lieu de la règle générale, et se dégrade sur des données nouvelles.

## Mesures de performance

- **Précision (precision)** : parmi les alertes levées, part de vrais achats. Une précision de 0,17 signifie qu'environ 17 alertes sur 100 sont de vraies décisions d'achat. Métrique prioritaire du projet (postulat 12).
- **Rappel (recall)** : parmi les vrais achats, part signalée par une alerte. Un rappel de 0,22 signifie qu'on détecte environ un achat sur cinq.
- **Faux positif / faux négatif** : alerte à tort (le modèle signale un article qui n'est pas acheté) / achat manqué (non signalé). Le projet juge un faux positif plus coûteux.
- **Seuil de décision** : valeur de score à partir de laquelle une alerte est levée. Le monter augmente la précision et diminue le rappel et le nombre d'alertes ; le baisser fait l'inverse.
- **Courbe précision-rappel** : courbe donnant la précision obtenue pour chaque niveau de rappel, quand on fait varier le seuil.
- **PR-AUC (average precision)** : aire sous la courbe précision-rappel. Adaptée aux cibles rares. Un modèle au hasard obtient la prévalence.
- **Lift** : PR-AUC divisée par la prévalence. 1 = hasard, 4 = quatre fois mieux que le hasard.
- **Alertes par semaine** : nombre d'articles signalés à l'acheteur sur une semaine à un seuil donné. Mesure la charge de travail.

## Exploitation

- **MLflow / Model Registry** : outil de suivi des entraînements et de versionnement des modèles. Le modèle est enregistré sous le nom `orion_achat_predictor`.
- **Alias `challenger` / `champion`** : étiquettes posées sur une version du modèle. `challenger` est un candidat qui vient d'être entraîné ; `champion` est la version servie en production.
- **Dérive (drift)** : changement de la distribution des données d'entrée par rapport à la référence d'entraînement. Peut annoncer une baisse de performance. Mesurée avec Evidently.
- **Vérité terrain** : ce qui s'est réellement passé (les achats effectivement décidés dans la semaine), par comparaison avec lequel on mesure la performance réelle du modèle.
- **dbt** : outil qui définit en SQL les features et la cible, utilisés de façon identique pour l'entraînement et pour les batchs.
- **Airflow (DAG)** : orchestrateur qui lance chaque semaine les étapes du cycle (features, notation, rapports).
