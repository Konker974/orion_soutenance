# Vision produit et organisationnelle : projet Orion

## 1. Vision produit

### Proposition de valeur

> Chaque semaine, Orion indique à chaque category manager les articles qui méritent son attention pour une décision d'achat, et lui montre pourquoi, afin qu'il décide plus vite et avec plus de recul.

Orion est un **assistant**, pas un décideur. Il réduit le travail de recherche et de croisement de données ; la décision reste celle du CM.

### Pour qui

| Utilisateur | Besoin | Ce qu'Orion lui apporte |
|---|---|---|
| Category manager (utilisateur principal) | Savoir quoi examiner cette semaine, sans croiser ventes, stock, commandes et promotions à la main | Une liste courte priorisée, avec les indicateurs de chaque article |
| Direction commerciale | Des achats cohérents d'un acheteur à l'autre, et un outil dont on mesure l'apport | Un suivi de la performance dans le temps |
| Équipe data | Un modèle qui s'améliore et reste sous contrôle | Un cycle de mesure, de réentraînement et de validation |

### Principes produit

1. **Précision avant exhaustivité** : mieux vaut une courte liste fiable qu'une longue liste où le CM perd du temps. Le rappel est suivi pour que la liste ne devienne pas vide de sens.
2. **Transparence** : chaque suggestion s'accompagne des variables qui l'expliquent, vérifiables en un coup d'œil.
3. **Contrôle humain** : le CM valide ou infirme, commente, et son retour est conservé pour améliorer l'outil.
4. **Une seule vérité** : mêmes définitions de données à l'entraînement et en production.
5. **Mesurable** : la performance réelle est calculée chaque semaine, pas seulement annoncée.
6. **Évolutif** : on démarre petit (Sodexma/Friand, décision d'achat) et on élargit par étapes.

### Ce qu'Orion n'est pas

- Un outil qui passe les commandes automatiquement.
- Un outil qui remplace la connaissance métier du category manager.
- Un outil couvrant les achats impulsés par les fournisseurs (Chocorun).

### Feuille de route

| Étape | Contenu |
|---|---|
| **V1** | Prédiction hebdomadaire de la décision d'achat, Sodexma/Friand, interface de consultation et de validation, suivi de performance |
| **V2** | Extension à Sogerep/Sogepar ; enrichissement des données (remplissage du dépôt, groupage des commandes, promotions réelles dans les batchs) ; exploitation du retour des CM |
| **V3** | Suggestion de quantités ; calculs déportés de l'interface vers l'API ; hébergement cible et industrialisation (images dans un registry, déploiement automatisé) |

### Mesure du succès produit

- Les CM utilisent l'outil chaque semaine.
- La précision réelle bat nettement le hasard et progresse avec les réentraînements.
- Le temps de préparation de la commande diminue (à confirmer par entretiens).
- Les CM font confiance à la liste : taux de décisions renseignées, commentaires qualitatifs.

Les indicateurs détaillés sont dans [dossier_de_suivi_de_projet/Indicateurs de mesure de la performance au long du projet.md](<dossier_de_suivi_de_projet/Indicateurs de mesure de la performance au long du projet.md>).

## 2. Vision organisationnelle

### Principe

Orion est autant un projet d'organisation que de technique : la qualité du modèle dépend de la qualité des données saisies dans l'ERP, donc des pratiques des équipes commerciales. L'outil ne tient dans la durée que si les processus d'achat sont renseignés de façon fiable et si quelqu'un est responsable du modèle une fois en production.

### Rôles

| Rôle | Responsabilité dans le fonctionnement cible |
|---|---|
| Category managers | Décider, utiliser et valider les suggestions ; saisir leurs achats dans l'ERP selon la procédure ; challenger le choix des variables |
| Relais métier (Sodexma/Friand, puis Sogerep/Sogepar) | Apporter la connaissance métier à l'équipe data ; signaler les changements de pratique |
| Relais opérationnel | Garantir l'application des procédures de saisie des achats |
| ADV | Garantir la justesse des données enregistrées |
| Direction commerciale | Valider les conditions d'exécution, assurer la participation des équipes |
| Équipe data | Maintenir données, modèle, API et supervision ; décider de la promotion d'un nouveau modèle |
| Développement interface | Faire évoluer l'interface selon les retours utilisateurs |
| DSI | Hébergement et charge sur l'infrastructure |
| Fournisseurs | Informés une fois le projet déployé |

### Processus récurrents

| Rythme | Activité | Responsable |
|---|---|---|
| Au fil de l'eau | Saisie des achats et des promotions dans l'ERP | CM, ADV |
| Hebdomadaire | Production et notation du batch, consultation et validation par les CM, mesure de la performance réelle | Automatisé (Airflow) ; CM ; équipe data |
| Hebdomadaire | Surveillance des alertes (batch manquant, dérive, API) | Équipe data |
| Périodique | Revue de performance avec les CM : précision, rappel, charge, retours qualitatifs | Chef de projet, CM |
| À la demande | Réentraînement et validation d'un nouveau modèle contre le modèle en service, promotion décidée par un humain | Équipe data |

### Gouvernance

- **Pilotage** : le chef de projet suit l'avancement sur MS Planner et valide les jalons ; chaque jalon donne lieu à un billet sur le site du projet, et l'équipe échange sur le canal Teams dédié.
- **Décisions sur le modèle** : tout changement de modèle passe par une comparaison objective avec le modèle en service sur des semaines inédites ; la mise en production reste une décision humaine, réversible.
- **Décisions sur les règles métier** (postulats, variables, seuil) : prises avec les CM, documentées dans le dossier de suivi.
- **Méthode** : cascade pour la mise en place initiale (données, API, interface), cycles agiles ensuite pour l'optimisation du modèle.

### Conduite du changement

- **Associer les CM dès le départ** : entretiens initiaux, choix des variables avec eux, retours intégrés à l'outil.
- **Faire confiance par la preuve** : l'outil montre ses raisons et sa performance réelle, y compris ses erreurs.
- **Communiquer sur les limites** : promotion estimée, données manquantes, signal modeste. Mieux vaut un outil dont on connaît les limites qu'un outil dont on attend trop.
- **Informer les parties prenantes** au bon moment : ADV dès le début, fournisseurs après déploiement.

### Risques organisationnels

| Risque | Parade |
|---|---|
| Saisie de l'ERP incomplète ou tardive, qui dégrade les données | Relais opérationnel, procédure de saisie, contrôle de fraîcheur des données |
| Faible adoption par les CM | Implication dès le début, indicateurs expliqués, retours pris en compte |
| Dépendance à une seule personne pour le run | Documentation, code et définitions partagés selon les règles du service data, procédures de déploiement scriptées |
| Surcharge de l'infrastructure | Mesure de la charge, échange avec la DSI |
| Dérive du périmètre | Version 1 limitée à Sodexma/Friand, feuille de route par étapes |

### Maintenance dans le temps

Le modèle est conçu pour être réentraîné : les batchs notés avec les achats réels viennent enrichir le dataset, la dérive des données est surveillée, un nouveau modèle est comparé à l'ancien avant d'être promu, et le retour arrière est immédiat. La responsabilité de ce cycle revient à l'équipe data, avec les CM pour interpréter les écarts constatés.
