Ce document a pour objectif de documenter les étapes, les conditions, et les postulats établis lors de la constitution du dataset d'entrainement et des features.

Cible : la cible est la décision d'achat --> à une date donnée, un category manager a pris la décision de démarrer un processus d'achat sur un article donné. Nous chercherons à prédire cette décision via un booléen. On va aborder cette prédiction comme une opération de classification sur la base des artciles disponibles dans le catalogue à une date précise : achat ou non achat.
Les décisions d'achat sont en général espacés, car les délais entre les demandes d'achat (décision d'achat initiale) et l'entrée des marchandises en dépot est généralement longue (plusieurs mois).


Postulats :
1-Les décisions d'achat sont prises une fois par semaine et peuvent se dérouler sur toute une semaine (on peut traiter par exemple un fournisseur par jour)
2-On part des lignes de ventes, car les tables de l'erp stockent les quantités par type de document avec les dates. On a ici un grain suffisament fin : 1 ligne = 1 article, avec son historique de date et quantité.
3-On part du principe que la date de la colonne dl_datede est la date de référence pour la décision d'achat.
4-On part du principe q'un délai d'achat trés court est une anomalie, un délai d'achat très court est une ligne d'achat pour une facture, dont la date de demande d'achat (colonne dl_datede) est la meme que celle de la facture (do_date) ou dont le décalage est inférieur à 8 semaines. On estime que ces délais très courts proviennent d'une décision d'achat non reportée dans l'erp au moment de la commande mais plusieurs semaines plus tard.
4 bis- Sur les données historiques, toutes les décisions sont devenues des factures : on connaît la date de facture et la date de demande, donc on sait appliquer le postulat 4. Pour les batchs, la notation est exécutée le dimanche et les bureaux ferment le vendredi : on part du principe qu'une demande d'achat encore ouverte le dimanche ne sera pas facturée rapidement, elle est donc comptée comme une décision d'achat.
5-On part du principe qu'une décision d'achat est toujours bonne et fondée sur une réflexion profonde de la part de l'acheteur.
6- On part du principe que l'on va entrainer l'algorithme sur 52 semaines, pour pouvoir inclure les variations saisonnières
7- On part du principe que la décision d'achat est fondée principalement sur les états des stocks et des ventes passées et prévisionnelles
8- On part du principe que l'on connait si l'article sera ou a été positionné pour une offre promotionnelle dans les 4 mois suivants la date de la prédiction / décision d'achat
8 bis- Les promotions sont connues des mois à l'avance mais saisies dans l'erp au dernier moment, pour des raisons purement opérationnelles : la date de saisie des promotions n'est pas prise en compte. Dans l'historique, les promotions sont donc fiables. Pour qu'elles soient complètes sur toute la fenêtre de 4 mois, la dernière semaine du dataset d'entrainement est la plus récente dont la date de coupure + 4 mois précède la date de la copie de l'erp. Pour les batchs hebdomadaires, la feature promotion est ESTIMÉE (tirage au taux moyen hebdomadaire d'articles en promotion observé dans le dataset d'entrainement, figé en dur) : elle proviendra plus tard d'un autre dataset en cours de constitution. LIMITE ASSUMÉE : le modèle apprend sur des promotions réelles et prédit sur des promotions estimées, ce qui baisse la fiabilité des prédictions.
9- On part du principe qu'une semaine démarre le lundi et se termine le dimanche. Les semaines sont numérotées selon la norme ISO 8601 (année ISO et numéro de semaine ISO).
10- la décision d'achat est agnostique à l'activité de la semaine en cours car elle se prend sur les éléments arretés sur la semaine w-1
11- on travaille sur la base de l'erp directement pour constituer le dataset d'entrainement
12 — La métrique prioritaire à maximiser est la précision (precision), afin de limiter les fausses recommandations d'achat. Une décision d'achat erronée (faux positif) est considérée comme plus coûteuse qu'une décision d'achat manquée (faux négatif). Le niveau de rappel devra néanmoins être suivi afin de s'assurer que cette recherche de précision ne conduise pas à écarter une proportion excessive de décisions d'achat pertinentes.
13- on sait qu'il manque des données pour avoir un niveau d'efficacité précis mais nous ne disposons pas de ces données actuellement. des travaux en cours permettront de capter ces données et d'enrichir les datasets d'entrainement dans le futur (remplissage du depot à la date de livraison, impact des phénomènes géopolitiques, groupage des commandes pour optimiser les frais logistiques, ...)

Etapes de la constitution du dataset d'entrainement : 
1- on isole les articles qui seront utilisés dans le datasets : actifs, et suivis en stock
    - les articles fictifs de gestion (référence commençant par Z) sont exclus ;
    - un article n'est retenu pour une semaine que s'il existait à la date de coupure ;
    - (ajout du 17/10/2026) les articles dormants sont exclus : il faut au moins une vente, un mouvement de stock du dépôt principal ou une décision d'achat dans les 52 semaines qui précèdent la coupure. Évalué à chaque coupure, ce filtre s'applique à l'identique aux batchs.
2- on définit la période utilisée pour la constitution du dataset
3- on crée une table articles X semaines en numérotant les semaines (index semaine-année car on les semaines pourront etre glissantes d'une année sur l'autre lors de l'établissement du dataste d'entraienement et des réentrainements)
4- Pour chaque ligne article X semaine, on ajoute les features qui peuvent paraitre utiles à la décision d'acheter ou non cet article sur une semaine données : 
    a- les ventes de la semaine passées peuvent laisser entrevoir des ventes importantes et une décision d'achat urgente
    b- la dernière date d'achat et la quantité peuvent avoir un impact sur la néessiter d'acheter un article
    c- l'état des stocks détermine si on a une couverture suffisante
    d- si l'article est positionné sur une vente promotionnelle dans les mois à venir, on peut estimer qu'un flux important de vente y sera associé
    e- le délai prévisionnel d'acheminement de la marchandise détermine à quel moment passer la commande pour rester en flux tendus

Mise en œuvre (décisions du 17/10/2026)
=======================================

Principe : une seule définition des features et de la cible, en SQL (dbt, projet etl_stack_prod), réutilisée à l'identique pour constituer le dataset d'entrainement, calculer les features de chaque batch hebdomadaire et noter ce batch. Un batch noté a donc exactement le schéma et les définitions du dataset d'entrainement : il peut lui être concaténé pour les réentrainements.

Source des données :
- copies des tables de l'erp Sage de Sodexma et Friand dans le schéma bronze d'Astro (alimentées par Airbyte) : F_DOCLIGNE (ventes, achats, mouvements de stock dans une seule table), F_ARTICLE, F_ARTSTOCK, F_TARIF, F_TARIFSELECT ;
- les dates métier font foi (date de demande pour les décisions, date de mouvement pour le stock) : la date de création des lignes n'est pas exploitable dans l'historique (lignes transformées en factures, copies remplacées en entier à chaque synchronisation) ;
- la copie de l'erp n'est pas rafraîchie régulièrement : sa date de fraîcheur (dernière modification présente) est mesurée à chaque exécution.

Coupure : pour une semaine cible, la date de coupure C est le lundi 00:00 de cette semaine. Les features n'utilisent que des faits datés avant C (postulat 10), la cible porte sur les décisions de la semaine [C, C + 7 jours). Numérotation des semaines : semaines ISO (postulat 9).

Période (étape 2) : 52 coupures consécutives (postulat 6). La dernière est la plus récente qui vérifie C + 4 mois <= date de la copie de l'erp (postulat 8 bis). Avec la copie arrêtée au 14/10/2026 : coupures du 16/06/2025 (semaine 2025-W25) au 08/06/2026 (semaine 2026-W24).

Stock (feature c) : seul le dépôt 1 « Depot » (stock vendable) est retenu ; les dépôts abîmés/périmés, PLV, déstockage, camion, etc. sont exclus. Le stock à la coupure est reconstitué ainsi : stock actuel (F_ARTSTOCK) moins les mouvements datés après la coupure (entrée = +quantité, sortie = -quantité, en valeurs absolues car les retours clients ont une quantité négative). Contrôle : la somme de tous les mouvements redonne exactement le stock actuel de chaque article et de chaque dépôt.

Cible : une décision d'achat est une ligne d'achat (demande, préparation, commande, livraison, facture) de quantité positive, valorisée, hors avoirs fournisseurs (pièces AFA), datée par sa date de demande (postulat 3), et qui n'est pas une anomalie du postulat 4. La cible vaut 1 si l'article a au moins une décision d'achat dans la semaine.

Fraîcheur des batchs : si la copie de l'erp d'une société est antérieure au vendredi de la semaine w-1, le batch est quand même produit mais marqué « données incomplètes » : il est noté et affiché, mais exclu des datasets de réentrainement.

Dictionnaire technique (une ligne par article × semaine cible) :
- clés : societe (sod, fri), reference_article, identifiant_article_hash, date_coupure, annee_iso, semaine_iso ;
- ventes_semaine_w1 (feature a) : ventes nettes du dépôt 1 sur la semaine w-1 ;
- ventes_semaine_w2 (feature a) : ventes nettes du dépôt 1 sur la semaine w-2 ;
- ventes_moyennes_hebdo_52s (postulat 7) : ventes nettes des 52 semaines précédentes / 52 ;
- time_since_last_buy (feature b) : jours depuis la dernière décision d'achat (730 = aucune sur 2 ans) ;
- prev_achat_qte (feature b) : quantité de cette dernière décision ;
- stock_a_date (feature c) : stock du dépôt 1 à la coupure ;
- achat_en_cours_a_date (feature c) : quantités décidées sur 52 semaines et pas encore réceptionnées ;
- couverture_stock_semaines (feature c) : (stock + en cours) / ventes hebdomadaires moyennes, bornée à 104 ;
- promotion_4_mois (feature d) : promotion de l'article dans les 4 mois (réelle à l'entrainement, estimée en batch, postulat 8 bis) ;
- delai_acheminement_jours (feature e) : délai moyen observé entre décision et réception sur 2 ans ;
- achat (cible) : 1 si décision d'achat dans la semaine cible.
Le détail de chaque définition est documenté dans le code dbt (macros/orion/orion_ml.sql, modèles models/gold/sales/grossiste/orion/ml/) et dans la documentation dbt.

Limites assumées (à exposer dans les présentations) :
- la feature promotion est estimée dans les batchs (postulat 8 bis) ;
- environ 4 mois séparent la fin du dataset d'entrainement de la mise en production, conséquence du postulat 8 bis ;
- les décisions d'achat sont très irrégulières d'une semaine à l'autre (nombreuses semaines sans aucune décision, d'autres avec plus de 50 articles) ;
- la notation hebdomadaire compte les demandes encore ouvertes (postulat 4 bis) : certaines seront plus tard requalifiées en anomalies.
