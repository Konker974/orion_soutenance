# Soumettre une nouvelle version du modèle Orion

Ce guide s'adresse au data scientist qui veut proposer une nouvelle version du modèle de prédiction des
achats : autres réglages, autre algorithme, nouvelles features. Il peut travailler où il veut (notebook,
poste local). Orion ne verrouille rien : la sécurité tient en deux règles.
- On ne promeut qu'une version que `POST /model/validation` déclare **conforme** : elle doit **prédire
  mieux que le champion** sur au moins un batch finalisé.
- On promeut **dans l'interface MLflow**.

Le serveur MLflow est celui de srvetl. Son adresse figure dans `dev/docker-compose.dev.yml` (`MLFLOW_TRACKING_URI`), et les comptes sont décrits dans `~/mlflow/README.md` sur srvetl.

---

## Le flux de bout en bout

Dans MLflow, l'**expérience** `orion_achat` représente le problème à résoudre, la prédiction d'achat. Chaque **run**
est une tentative d'un data scientist. Une **version** de `orion_achat_predictor` est une tentative proposée
pour la production. L'alias `champion` désigne la version qui produit les prédictions.

| Étape | Qui | Où | Ce qui se passe |
|---|---|---|---|
| 1. Expérimenter | data scientist | notebook / poste, branché sur MLflow | entraînements sur les datasets disponibles, chacun journalisé comme run de `orion_achat` (§2 à §4) |
| 2. Choisir | data scientist | interface MLflow | comparaison des runs entre eux et avec les métriques du champion, sur les mêmes données d'entraînement ou un dataset de même structure. Ces vérifications sont à sa charge. |
| 3. Proposer | data scientist | interface MLflow | le run retenu est enregistré comme version (« Register model ») avec l'alias `challenger` (§5) |
| 4. Comparer sur le dernier batch | data scientist | API : `POST /model/validation` | **pas d'entraînement** : l'API fait prédire le candidat et le champion sur le dernier batch finalisé, puis les compare (§6) |
| 5. Soumettre | data scientist | interface MLflow | réponse `conforme` collée dans la description de la version, puis demande de promotion à un admin |
| 6. Contre-vérifier | admin | API : `POST /model/validation` | même appel, même résultat (aucun entraînement) : c'est l'API qui tranche, pas le notebook |
| 7. Promouvoir | admin | interface MLflow | `precedent` sur l'ancien champion, `champion` sur la nouvelle version (§7) |
| 8. Contrôler | admin | API : `GET /model` | la nouvelle version est servie, sans redéploiement ni redémarrage |
| 9. Prédire | Airflow, le week-end | API : `POST /predictions/run` | rien ne change côté Airflow : l'API résout `champion` et charge la nouvelle version (§8) |
| 10. Mesurer | Airflow, à S+2 | API : `/batches/prepare-retraining`, `/monitoring/evaluation-report` | performance réelle du nouveau champion. Son batch finalisé devient la référence du prochain candidat. |

Le travail amont (étapes 1 et 2) n'est pas contrôlé par Orion. Le contrôle commence à l'étape 4, et la
promotion reste un geste manuel de l'admin.

---

## 1. Ce qu'une version doit fournir

`GET /model/validation/exigences` renvoie toujours la version à jour de ce contrat.

| Élément | Où | Exigence |
|---|---|---|
| Modèle | artefact `model` du run | estimateur ou `Pipeline` scikit-learn avec `predict_proba`, enregistré par `mlflow.sklearn.log_model` |
| Version de scikit-learn | automatique (MLmodel) | **identique à celle de l'API** (`sklearn_version` des exigences, 1.9.0 aujourd'hui) |
| `feature_cols` | `metadata` de `log_model` | liste ordonnée des colonnes présentées au modèle, toutes présentes dans `gold__orion.ml_features_batch` (`colonnes_disponibles`) |
| `decision_threshold` | `metadata` | seuil de décision dans ]0, 1[ : au-delà, l'article est signalé |
| `version_dataset` | `metadata` | dataset d'entraînement utilisé |
| `derniere_semaine_entrainement` | tag (run ou version) | dernière `date_coupure` vue à l'entraînement, au format ISO `AAAA-MM-JJ` |
| `auteur` | tag | qui a produit la version |
| `description` | tag, ou description de la version | ce qui change par rapport au champion, et pourquoi |
| `code_source` | tag | où retrouver le code : commit git, lien du notebook |

Les tags sont lus **sur la version, sinon sur le run d'origine**. Il suffit donc de les poser au moment du log : une version enregistrée depuis l'interface les retrouve.

L'API applique le modèle tel quel, sans transformation : `pipeline.predict_proba(lignes[feature_cols])[:, 1] >= decision_threshold`.
Toute préparation des variables (standardisation, encodage…) doit donc faire partie du `Pipeline`.

## 2. Récupérer le champion et les données

```python
import mlflow, mlflow.sklearn

mlflow.set_tracking_uri("http://<serveur MLflow srvetl>:5000")   # identifiants : MLFLOW_TRACKING_USERNAME / _PASSWORD
uri = "models:/orion_achat_predictor@champion"
champion = mlflow.sklearn.load_model(uri)
info = mlflow.models.get_model_info(uri)
info.metadata          # feature_cols, decision_threshold, version_dataset, regle_seuil
```

L'interface MLflow montre aussi le run du champion : réglages, métriques de validation croisée et du holdout, et `table_seuils.csv`.

**Données d'entraînement.** MLflow ne conserve que la description du dataset (nom, empreinte, chemin), pas les données. Il faut les prendre :
- soit dans l'export versionné sur srvetl : `~/etl_stack_prod/ressources/orion/datasets/entrainement/<version_dataset>/`, avec `dataset.csv` et `manifest.json`. L'empreinte SHA-256 du CSV y est, pour vérifier une copie ;
- soit directement dans la table dbt `gold__orion.ml_dataset_entrainement` (Astro).

Une ligne correspond à un article × une semaine (`date_coupure`), et la cible est `achat`. Le dictionnaire des features est dans
`dataset_entraienement_et_presupposes.md`.

## 3. La règle qui conditionne l'évaluation : laisser des semaines de côté

La validation compare le candidat et le champion sur des **batchs finalisés** : des batchs hebdomadaires réels
de S-2, dont les achats réels ont déjà été comparés aux prédictions (`operation_achat` renseigné par
`/batches/prepare-retraining`). Il en faut au moins un. Elle ne retient que les batchs
**postérieurs à la dernière semaine d'entraînement des deux modèles** : un modèle ne peut pas être jugé sur
des semaines qu'il a déjà vues.

En pratique :
- `GET /model/validation/exigences` donne `champion.derniere_semaine_entrainement` et la liste `batchs_complets`.
- Il faut entraîner sur des semaines **antérieures** aux batchs sur lesquels on veut être jugé. Pour une comparaison sur les 2 derniers batchs finalisés, on arrête l'entraînement avant le plus ancien des deux.
- Le tag `derniere_semaine_entrainement` doit dire la vérité. Il ne bloque rien, mais c'est lui qui garantit que la comparaison est honnête.

Pour l'instant, un seul batch finalisé est disponible : on compare donc sur `nb_batchs: 1`. Avec environ 40 achats
réels par batch, les métriques d'une seule semaine sont bruitées (voir le §9). Dès que d'autres batchs sont
finalisés, on peut demander jusqu'à `nb_batchs_max` batchs, et `par_batch` dans la réponse permet de vérifier que
l'écart tient d'une semaine à l'autre.

## 4. Entraîner et logger

Exemple minimal, à adapter :

```python
import mlflow, mlflow.sklearn

mlflow.set_experiment("orion_achat")
with mlflow.start_run(run_name="essai_C1_arcsinh"):
    mlflow.log_params({"C": 1.0, "preparation": "arcsinh_standard"})
    # ... entraînement de `pipeline` sur les semaines <= derniere_semaine ...
    mlflow.log_metrics({"holdout_pr_auc": pr_auc})          # libre : utile pour comparer les runs
    mlflow.set_tags({
        "derniere_semaine_entrainement": "2026-09-14",
        "auteur": "prenom.nom",
        "description": "arcsinh sur les ventes, C=1",
        "code_source": "https://.../notebook.ipynb@<commit>",
    })
    mlflow.sklearn.log_model(
        pipeline, name="model",
        input_example=X.head(5),
        metadata={"feature_cols": feature_cols, "decision_threshold": 0.7, "version_dataset": "orion_ds_2026W38_..."},
    )
```

Il est possible de passer directement `registered_model_name="orion_achat_predictor"` à `log_model` pour créer
la version tout de suite. Le parcours conseillé est de logger plusieurs essais, puis de choisir.

## 5. Comparer et enregistrer, dans l'interface MLflow

1. Ouvrir l'expérience `orion_achat`, cocher les runs à confronter, puis cliquer **Compare** : réglages et métriques côte à côte.
   Le run du champion y figure aussi, avec ses métriques de référence.
2. Ouvrir le run retenu, puis l'artefact `model`, puis cliquer **Register model** et choisir `orion_achat_predictor`. MLflow crée la version N.
3. Poser l'alias `challenger` sur cette version : c'est ainsi que l'admin la retrouve.

Les tags et la description peuvent être complétés à ce moment sur la page de la version.

## 6. Valider

```bash
# Exigences et batchs disponibles
curl -s https://<portail>/api/orion/model/validation/exigences | jq

# Comparaison sur le dernier batch finalisé inconnu des deux modèles
curl -s -X POST https://<portail>/api/orion/model/validation \
     -H 'Content-Type: application/json' \
     -d '{"model_uri": "models:/orion_achat_predictor@challenger", "nb_batchs": 1}' | jq

# Ou sur des batchs choisis
     -d '{"model_uri": "models:/orion_achat_predictor@challenger", "batch_ids": ["2026_W39", "2026_W40"]}'
```

En local (stack de `dev/`), l'API est sur `http://localhost:8000` (sans `/api/orion`), avec une documentation interactive sur `/docs`.

**Lire la réponse.** `verdict` vaut `conforme` si toutes les `conditions` sont remplies :

| Condition | Vérifie |
|---|---|
| `contrat_modele` | chargeable par `mlflow.sklearn`, `predict_proba`, même version de scikit-learn que l'API |
| `metadonnees` | `feature_cols`, `decision_threshold` dans ]0, 1[, `version_dataset` |
| `tags_obligatoires` | les 4 tags (sur la version ou le run) |
| `features_en_production` | `feature_cols` existent dans `gold__orion.ml_features_batch` |
| `semaines_non_vues` | assez de batchs finalisés postérieurs à l'entraînement des deux modèles |
| `features_dans_batchs` | les colonnes sont présentes et sans valeur manquante dans les batchs retenus |
| `pr_auc` | PR-AUC du candidat **strictement supérieure** à celle du champion : il doit prédire mieux |
| `precision_au_seuil` | précision du candidat à son seuil ≥ celle du champion à son seuil |
| `rappel_au_seuil` | rappel du candidat à son seuil ≥ `rappel_min` (0,20) |

`metriques` (batchs empilés) et `par_batch` donnent aussi le lift et les alertes par semaine, pour les deux modèles.
Le champion est recalculé sur les mêmes lignes : le `prediction_proba` stocké dans un batch peut venir d'un ancien champion.

La route ne modifie rien et peut être appelée autant de fois que nécessaire.

**Soumettre à l'admin.** L'API ne garde pas trace du verdict : elle n'a pas de compte MLflow en écriture.
Le data scientist colle donc la réponse `conforme` (date, batchs, métriques) dans la description de la
version `challenger`, puis prévient l'admin.

## 7. Promouvoir, dans l'interface MLflow (admin)

Seul le compte `alexandre` peut déplacer `champion`.
1. **Contre-vérifier** : relancer `POST /model/validation` sur `models:/orion_achat_predictor@challenger` avec les mêmes batchs.
   Le résultat doit être le même que celui collé par le data scientist (il n'y a aucun entraînement).
2. Sur la version actuellement `champion`, poser l'alias `precedent`.
3. Sur la nouvelle version, poser l'alias `champion`.
4. Contrôler :
   - `GET /api/orion/model` doit renvoyer la nouvelle version ;
   - les logs doivent contenir l'événement `model_loaded` ;
   - Grafana doit afficher la nouvelle version dans `orion_model_info`.

**Retour arrière** : remettre `champion` sur la version `precedent`. L'effet est immédiat, au prochain appel.

## 8. Après la promotion : le batch du week-end et les semaines suivantes

- **Prédiction.** Airflow appelle `POST /predictions/run` comme d'habitude, sans paramètre de modèle :
  - l'API résout l'alias `champion` à chaque appel, voit que la version a changé et la charge ;
  - elle prédit avec les `feature_cols` et le `decision_threshold` de cette version.
- **Traçabilité.** La réponse contient `model_version`, `model_alias` et `model_run_id`, qu'Airflow écrit dans le JSON
  du batch. Chaque batch indique donc quel modèle l'a produit, ce qui permet de remonter jusqu'au run et à son dataset.
- **Cohérence.** Un batch correspond à un seul appel, donc il est entièrement produit par un seul modèle. Il suffit
  de promouvoir avant le lancement du DAG du week-end.
- **Mesure réelle.** À S+2, le batch est finalisé par `/batches/prepare-retraining`, et `/monitoring/evaluation-report`
  mesure la performance réelle du nouveau champion. Les rapports de drift suivent automatiquement ses `feature_cols`.
- **Boucle.** Ce batch finalisé sert de référence au candidat suivant : chaque nouveau champion doit avoir battu le précédent.

## 9. Points de vigilance

- **Champion sans dates d'entraînement.** Si le champion n'a pas le tag `derniere_semaine_entrainement`, ni de
  `semaines_entrainement` sur son run, la condition `semaines_non_vues` échoue pour tout candidat.
  À poser une fois dans l'interface MLflow (v1 : `2026-06-08`).
- **Un seul batch, c'est peu.** Un batch finalisé compte environ 40 achats réels. Une victoire sur un seul batch peut
  venir du hasard. On commence avec `nb_batchs: 1` faute de mieux, puis on augmente, jusqu'à `nb_batchs_max`,
  à mesure que des batchs se finalisent.
- **Features.** Une version qui utilise une colonne absente de `ml_features_batch` ferait échouer le batch du
  week-end (erreur 422). La validation le refuse en amont (voir le §10).

## 10. Ajouter une feature

Une nouvelle feature doit exister **en production** avant de pouvoir servir :
1. **dbt** (dépôt `etl_stack_prod`, macros `macros/orion/orion_ml.sql`) : ajouter la colonne au dataset d'entraînement (`ml_dataset_entrainement`) **et** aux features des batchs (`ml_features_batch`), avec la même définition.
2. Attendre au moins un batch **finalisé** qui contient la colonne : avant, `features_dans_batchs` échoue.
3. Régénérer un dataset d'entraînement (DAG `orion_dataset_entrainement`), entraîner, puis soumettre.
4. Mettre à jour le dictionnaire des features (`dataset_entraienement_et_presupposes.md`).

Pour retirer une feature, on fait l'inverse : on promeut d'abord un modèle qui ne l'utilise plus, et ensuite seulement on la retire de dbt.
Sinon, le champion en place ne trouve plus sa colonne (erreur 422 à la prédiction).

## 11. Reproduire un entraînement de référence

`scripts/entrainer_modele.py` reste la voie outillée : découpage temporel, validation croisée, seuil,
garde-fou contre un modèle qui ne bat pas le hasard. Il pose lui-même métadonnées et tags (voir le
README, « 1. Produire le modèle »). Le modèle qu'il enregistre est réentraîné sur tout le dataset :
il ne pourra être validé que sur des batchs finalisés postérieurs à la fin de ce dataset.

## Démonstration rapide

**Sans rien entraîner.** Soumettre le champion lui-même :
`{"model_uri": "models:/orion_achat_predictor@champion"}`.
Toutes les conditions de contrat passent et les métriques sont identiques des deux côtés. Le verdict est pourtant
`non_conforme`, à cause de `pr_auc` : faire aussi bien ne suffit pas, il faut prédire mieux.

**Avec un réentraînement.** Changer un seul hyperparamètre :
1. Charger le champion, puis cloner son Pipeline : `sklearn.base.clone(champion)`.
2. Modifier la régularisation, par exemple `C` de 0,1 à 1,0 : `pipeline.set_params(modele__C=1.0)`.
3. Réentraîner sur les semaines antérieures aux batchs visés.
4. Logger avec les métadonnées du champion, ou recalculer le seuil.
5. Comparer les deux runs dans MLflow, enregistrer, puis appeler la validation.
