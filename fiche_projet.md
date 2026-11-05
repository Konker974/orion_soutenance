
En cours



👁️‍🗨️Résumé Exécutif
Description succincte du projet : Contexte, but et importance du projet.

Le projet Orion a pour but de mettre à disposition un ensemble de solutions d'aide à l'achat à destination des Category Manager pour effectuer. Le projet est composé de plusieurs phases : 
1. Intégration des données d'achat dans Sage
2. Enrichissement des données au niveau du data Warehouse
3. Développement de l'algorithme de suggestion d'achat (action+ quantité)
4. Développement de l'interface utilisateur
5. Automatisation et amélioration continue




🎯 Objectifs du projet

- Objectifs spécifiques : Ce que le projet vise à accomplir.
- Résultats attendus : Les produits livrables ou les résultats concrets du projet


Apporter une solution technique complète d'aide à la prise de décision pour les commandes fournisseurs (API de prise de commande live et fonctionnelle)
1. Développer une solution qui s'appuira sur les données disponibles et enrichies pour recommander les achats nécessaires et les quantités sur une base hebdomadaire ** (voir les fréquences aux entretiens et en analyse préliminaire)
2. Optimiser l'algorithme pour que celui-ci minimise les erreurs
3. Déployer cet algorithme sur une ressource adaptée et la servir via une interface intuitive et intercactive
4. Prévoir sa maintenance dans le temps (réapprentissage)






❔Justification du projet

- Problèmes à résoudre ou opportunités à saisir : Pourquoi le projet est nécessaire.
- Bénéfices attendus : Avantages pour l’organisation ou les parties prenantes


Ce projet permettra d'augmenter la productivité des équipes commerciales en leur faisant gagner du temps pour la décision d'achat. De plus, l'algorithme fournira les détails des choix appliqués afin de donner du recul aux décideurs finaux.
L'interface pourra exposer les variables importantes menant aux décisions, ce qui permettra de vérifier rapidement la justesse et évitera d'effectuer des recherches complexes avec croisement des données (gain de temps).




✒️Portée du projet

- Délimitation du projet : Ce qui est inclus et ce qui est exclu.
- Critères de succès : Comment sera mesuré le succès du projet.


Pour l'API de commande : on va déployer en premier lieu sur Sodexma/Friand (le critère principal de choix se porte sur l'urgence du besoin). Il sera ensuite rapidement déployé pour Sogerep/Sogepar. Chocorun est hors scope pour le moment étant donné que les achats sont impulsés par les fournisseurs.




🙋‍♂️Parties prenantes (externes à équipe projet, voir externes à l'organisation)

- Liste des parties prenantes : Qui sont les personnes ou groupes impliqués et impactés par le projet.
- Rôles et responsabilités : Ce que chaque partie prenante apportera au projet.



Nom entité	Rôle ou impact
Category Managers	En première ligne de ce projet, les Catmans devront superviser la justesse des choix des variables qui entreront dans l'analyse et orienteront l'équipe data dans la recherche des variables (features). Ils participent également activement à renseigner l'ERP sur tous leurs processus d'achat selon les procédures en vigueur et éventuelles modifications en cours de route.
Direction commerciale Sodex/Fri	Valide les conditions d'exécution du projet et s'assure de la participation de ses équipes
ADV	Sont informés de la teneur du projet et participent à la justesse des données enregistrées
Fournisseurs	Informés de la démarche une fois le projet déployé

Commentaire




💯Exigences du projet
Exigences fonctionnelles et non fonctionnelles : Ce que le projet doit réaliser en termes de fonctionnalités et de performance

API d'aide à la commande : 
- Mise à disposition des personnes désignées 
- Suggérer les articles à commander sur la semaine, selon le profil utilisateur (son portefeuille articles)
- Proposer des quantités adaptées aux différentes situations (stock, promos à venir, etc ...)
- Mettre à disposition les indicateurs ayant permis d'aboutir à ces propositions ; permettre un export en excel de ce tableau
- Mettre à disposition ces indicateurs pour tous les articles du portefeuille de l'utilisateur, même pour ceux dont la commande n'est pas suggérée par l'algorithme (pour vérification) ; permettre un export en excel de ce tableau
- Permettre à l'utilisateur de valider ou non la décision d'achat, ainsi que de renseigner les quantités effectivement commandées ; et stocker ces éléments pour amélioration future de l'algorithme
- Permettre à l'utilisateur de commenter sa décision d'achat pour chaque suggestion de l'outil, et stocker ce commentaire pour analyse qualitative
- Mesurer la performance globale de l'agorithme à travers le temps






🏛️Contraintes

- Contrainte de temps : Dates importantes et échéances.
- Contrainte de budget : Budget alloué au projet.
- Contrainte de ressources : Disponibilité des ressources humaines, matérielles et technologiques.


Pas de budget spécifique à allouer. Les ressources sont internes et déjà en place.




📜Hypothèses de travail
Hypothèses de travail : Les conditions supposées vraies pour la planification et l’exécution du projet.

Les données doivent être rendues disponibles par les équipes commerciales, soit via l'ERP, soit via des partages de fichiers de travail actifs (éviter des modèles de fichiers qui ne sont plus utilisés).




☣️Risques initiaux

- Identification des risques : Les risques potentiels identifiés dès le début du projet.
- Plans de mitigation : Stratégies pour atténuer ces risques.


Texte

Description	Mitigation
S'éparpiller	Suivi de l'avancée du projet sur Planner, et échéance à fin septembre 2025
Chercher à couvrir un périmètre trop large dès le début	On va se cantonner au périmètres Sodexma/Friand pour la 1ere version de l'algorithme
Surcharger l'infrastucture IT	Mesurer la charge et travailler avec le DSI si elle devient critique
Aboutir à un modèle non performant	Définir un seuil de performance en %





🔗 Equipe de Projet

- Structure de l’équipe : Composition de l’équipe de projet.
- Rôles et responsabilités : Ce que chaque membre de l’équipe fera.



Nom	Rôle	Missions
Alexandre Pignolet <alexandre.pignolet@sodexma.fr>	🧑‍🍳 Chef de projet	Piloter le projet, valider les étapes (jalons), maîtriser le temps et le budget
Alexandre Pignolet <alexandre.pignolet@sodexma.fr>	☎️ Responsable de la communication	Communiquer sur l'avancement du projet aux personnes concernées
Alexandre Pignolet <alexandre.pignolet@sodexma.fr>	⚒️ Technicien projet	Mettre en place le framework et répondre à l'équipe projet sur des demandes techniques
Alexandre Pignolet <alexandre.pignolet@sodexma.fr>	Supervision du développement de l'algorithme	Définit, l'algorithme à utiliser, et supervise la performance d'une solution vs les autres solutions disponibles. Supervise l'optimisation des paramètres
Florèle ?	Relai technique commercial Sodexma/Friand	Apporte un soutien permanent pour la connaissance métier à l'équipe data. Devra être disponible pour des réunions et des questions permettant de faire avancer le projet.
Marion ? Sandra ?	Relai technique commercial Sogerep/Sogepar	Apporte un soutien permanent pour la connaissance métier à l'équipe data. Devra être disponible pour des réunions et des questions permettant de faire avancer le projet.
Alexandre Robert <alexandre.robert@cdmarksoi.com>	Technicien développement	Développe l'interface utilisateurs
Alexandre Pignolet <alexandre.pignolet@sodexma.fr>	Technicien BDD	Assure que les données nécessaires soient mises à disposition pour le développement de l'algorithme
Fayçal en direct ?	Relai opérationnel	Garantit la mise en application des process de saisie des achats

Commentaire éventuel




🗂️ Approche méthodologique

- Méthodologie : Méthode de gestion de projet utilisée (Agile, Waterfall, etc.).
- Outils et techniques : Outils et pratiques spécifiques pour la gestion du projet.


Assignation des tâches par AP sur MS Planner
Gestion du code du projet selon la méthodologie du service data déjà en place
La mise en place initiale (données, API, interface) suivra une méthodologie en cascade puis l'optimisation de l'algorithme sera effectuée en cycle agile.




📸 Plan de communication

- Stratégie de communication : Comment les informations seront partagées entre les parties prenantes.
- Fréquence et format des rapports : Quand et comment les rapports d’avancement seront produits.


La communication sera assurée via le site web du projet. L'atteinte des jalons donnera lieu à un billet d'actualité sur le site web.
La communication de l'équipe projet passera par le canal teams affecté.




👌 Approbations
Signatures des sponsors et du chef de projet : Approbation formelle de la charte de projet par les personnes autorisées.


Nom	Visa
Thomas	
Nicolas	
Fayçal	
AP	





🗄️ Références et documents supplémentaires
Documents liés : Références à d'autres documents importants comme le business case, les études de faisabilité, etc..

Site web du datawarehouse Astro qui supporte l'infrastructure data du projet : Astro - Home
Le guide d'entretien et 1 ou 2 entretiens
L'experience map sur la situation initiale (à concevoir après les entretiens)
Description de la création, de l'optimisation, du déploiement et de la maintenance de l'algorithme (pas le code)
Scripts d'automatisation du service





