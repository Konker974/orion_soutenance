# Cahier des charges : projet Orion

Aide à la décision d'achat pour les Category Managers. Ce document précise la fiche projet ([fiche_projet.md](fiche_projet.md)) : besoin, périmètre, exigences et critères d'acceptation.

## 1. Contexte et besoin

Chaque semaine, les Category Managers (CM) décident quels articles commander auprès des fournisseurs. Cette décision demande de croiser à la main plusieurs sources (ventes, stock, commandes en cours, promotions, délais), ce qui est long et varie d'un acheteur à l'autre.

**Besoin** : une aide à la commande qui, chaque semaine, propose les articles à examiner et montre pourquoi, pour faire gagner du temps sans retirer la décision au CM.

**Principe** : un algorithme d'apprentissage automatique entraîné sur l'historique des décisions d'achat, alimenté par le datawarehouse Astro, exposé via une API et une interface web.

## 2. Objectifs

| # | Objectif |
|---|---|
| O1 | Prédire chaque semaine, article par article, la décision d'achat (acheter ou non) |
| O2 | Donner pour chaque proposition les indicateurs qui la justifient |
| O3 | Servir ces résultats aux CM via une interface simple, selon leur portefeuille d'articles |
| O4 | Recueillir leurs décisions réelles pour mesurer et améliorer l'algorithme |
| O5 | Maintenir le modèle dans le temps (surveillance, réentraînement) |

## 3. Périmètre

**Inclus**
- Sodexma et Friand en première version (urgence du besoin), puis Sogerep et Sogepar.
- Cycle hebdomadaire, sur les articles actifs et suivis en stock.
- Chaîne complète : données ERP (Sage), enrichissement dans le datawarehouse, algorithme, API, interface, automatisation.

**Exclus**
- Chocorun : les achats y sont impulsés par les fournisseurs.
- Passage automatique des commandes dans l'ERP : l'outil suggère, le CM décide.
- Données externes non disponibles au départ (contexte géopolitique, groupage logistique, remplissage du dépôt) : prévues dans des évolutions.

## 4. Exigences fonctionnelles

| Réf. | Exigence |
|---|---|
| F1 | Accès réservé aux personnes désignées, chacune limitée aux sociétés de son périmètre |
| F2 | Liste hebdomadaire des articles à commander, selon le portefeuille de l'utilisateur |
| F3 | Indicateurs à l'origine de chaque suggestion (ventes, stock, achats en cours, promotion, délai…) |
| F4 | Indicateurs disponibles pour tous les articles du portefeuille, y compris ceux non suggérés, pour vérification |
| F5 | Export Excel des tableaux de F3 et F4 |
| F6 | L'utilisateur valide ou infirme chaque suggestion ; la décision est stockée |
| F7 | Saisie optionnelle de la quantité effectivement commandée et d'un commentaire, stockés pour analyse |
| F8 | Suivi de la performance de l'algorithme dans le temps, consultable par l'administration |
| F9 | Proposition de quantités adaptées à la situation (stock, promotions à venir) |

## 5. Exigences non fonctionnelles

- **Performance du modèle** : la précision prime (une fausse recommandation coûte plus qu'un achat manqué) ; le rappel est suivi. Les seuils chiffrés sont fixés après l'analyse préliminaire, à partir d'une référence (hasard) et de la charge supportable par les CM.
- **Charge de travail** : la liste hebdomadaire doit rester lisible par un CM (de l'ordre de quelques dizaines d'articles, pas de centaines).
- **Explicabilité** : choix d'un algorithme dont les décisions s'expliquent aux utilisateurs.
- **Traçabilité** : chaque proposition indique la version du modèle et des données qui l'ont produite.
- **Fiabilité** : calcul hebdomadaire automatisé, supervisé et alerté (100 % des DAG Airflow réussis).
- **Évolutivité** : réentraînement possible avec les nouvelles données ; remplacement d'un modèle contrôlé et réversible.
- **Sécurité** : droits par société, pas de donnée client dans les journaux.
- **Infrastructure** : s'appuyer sur la stack data existante, sans surcharger les ressources IT.

## 6. Données

- Sources : tables Sage (ventes, achats, mouvements de stock, articles, tarifs) copiées dans le datawarehouse Astro.
- Historique d'entraînement : 52 semaines, pour couvrir la saisonnalité.
- Une seule définition des variables et de la cible, utilisée à l'identique pour l'entraînement et la production.
- Postulats métier (décision hebdomadaire, délais d'achat anormaux, promotions connues à l'avance…) documentés dans [dossier_de_suivi_de_projet/dataset_entraienement_et_presupposes.md](dossier_de_suivi_de_projet/dataset_entraienement_et_presupposes.md).

## 7. Critères de succès

| Axe | Indicateur | Cible |
|---|---|---|
| Modèle | Précision et lift par rapport au hasard | Seuil fixé après analyse préliminaire ; modèle nettement meilleur que le hasard |
| Usage | Utilisateurs distincts par semaine | Égal au nombre de CM impliqués dans les achats |
| Flux | DAG Airflow réussis | 100 % |
| Données | Couverture du dataset | ~1 300 articles Sodexma, ~500 Friand ; au moins 45 semaines pour un réentraînement |
| Gain | Temps de préparation de la commande | Réduit, à valider par des entretiens avant/après |

## 8. Planification

Méthode en cascade pour la mise en place (données, API, interface), puis cycles agiles pour l'optimisation de l'algorithme.

| Phase | Contenu |
|---|---|
| 1 | Intégration des données d'achat dans Sage |
| 2 | Enrichissement des données dans le datawarehouse |
| 3 | Développement de l'algorithme |
| 4 | Développement de l'interface utilisateur |
| 5 | Automatisation et amélioration continue |

Échéance de la première version : **fin septembre 2025**. Suivi des tâches sur MS Planner.

## 9. Organisation

- **Chef de projet, data et algorithme** : Alexandre Pignolet (pilotage, données, API, algorithme).
- **Développement de l'interface** : Alexandre Robert.
- **Relais métier** : référent commercial Sodexma/Friand, puis Sogerep/Sogepar ; relais opérationnel pour la saisie des achats.
- **Sponsors** : direction commerciale Sodexma/Friand.
- **Budget** : aucun budget spécifique, ressources internes.

## 10. Risques

| Risque | Mitigation |
|---|---|
| Éparpillement | Suivi sur Planner, échéance fixée |
| Périmètre trop large | Version 1 limitée à Sodexma/Friand |
| Surcharge de l'infrastructure | Mesure de la charge, échange avec la DSI |
| Modèle peu performant | Seuil de performance défini à l'avance ; garde-fou : pas de mise en production si le modèle ne bat pas le hasard |
| Données incomplètes ou mal saisies | Relais opérationnel sur la saisie des achats, postulats documentés |

## 11. Livrables

1. Datasets d'entraînement et de production (SQL/dbt dans le datawarehouse).
2. Algorithme entraîné, versionné, avec documentation de sa construction.
3. API de suggestion et de suivi, déployée.
4. Interface utilisateur (consultation, validation, exports).
5. Automatisation hebdomadaire, supervision et suivi de performance.
6. Dossier de suivi de projet et documentation.
