# TP8 - Intégration et architecture

Votre application est maintenant capable de gérer la plupart des interactions avec l'utilisateur et le réseau. Elle est en revanche probablement difficile à faire évoluer du fait de l'empilement des modifications effectuées depuis le premier TP Qt jusqu'à celui où vous avez intégré les tuiles cartographiques. Ce TP va donc viser à perfectionner l'architecture de façon à pouvoir modifier facilement le code et faire évoluer l'application.

Pour architecturer notre application, il est d'abord nécessaire de lister ce qu'elle est censée faire afin d'identifier les rôles de ses différents éléments. Dans les grandes lignes, nous devons fournir les éléments suivants :

- une fenêtre d'application comportant :
	+ une barre de menu avec Fichier -> quitter, et Aide -> À propos
	+ un panneau latéral avec un champ de recherche, un bouton de recherche et une zone affichant les résultats
	+ un panneau central qui affiche la carte de la zone choisie
- une fenêtre *À propos* contenant un texte pour présenter l'application (notamment le ou les auteurs de l'application)

## Le panneau de cartographie

Le panneau de cartographie étant l'élément le plus complexe, il est décrit plus en détails dans cette section. À partir de la position du centre, de la taille de la zone et du niveau de zoom, le panneau doit permettre d'afficher un pavage de tuiles, issues d'un serveur de tuiles (Tile Map Server). Le panneau doit permettre les manipulations suivantes :

- "sauter" à une position (à partir de la liste des résultats Nominatim du panneau latéral)
- déplacer la carte autour de sa position actuelle (par cliquer-glisser)
- zoomer/dézoomer sur la même zone de carte, à l'aide de la molette de la souris

Chacune de ces manipulations doit remettre à jour l'affichage pour visualiser la zone souhaitée, au niveau de détail souhaité.

## Éléments attendus et facultatifs

À la fin de la série de séances sur la ressource R4.01, vous devrez remettre votre code afin qu'il soit évalué. Le code devra être remis en binôme, ce qui vous forcera à prendre en compte la collaboration dans les contraintes de développement, et donc rendra encore plus critique l'architecture de votre code.

### Modalités de l'évaluation

L'évaluation porte sur le développement des fonctions listées ci dessous, dont certaines sont optionnelles (elles rapportent des points en plus) et sont indiquées par la mention *(optionnel)*. L'évaluation prend en compte l'exactitude de la fonction développée (50%), mais aussi son intégration dans l'architecture de l'application (50%).

Liste des fonctions attendues :

- Création de la fenêtre principale :
	+ Titre
	+ Barre de menus
	+ Menus
	+ Panneau Nominatim
	+ Widget de la carte
	+ Barre de statut avec les coordonnées pointées par la souris (optionnel)
- Création de la fenêtre *"À propos"*
- Panneau Nominatim :
	+ Champ de recherche
	+ Bouton de recherche (déclenche la requête)
	+ Liste de résultats (obtenus par la réponse Nominatim)
	+ Centrage sur la carte par double-clic
	+ Recherche en appuyant *"Entrée"* au lieu du bouton (optionnel)
- Panneau de la carte :
	+ Affichage de toutes les tuiles nécessaires, dans l'ordre correct (les tuiles ne doivent pas être mélangées)
	+ Gestion du zoom avec la molette (zoom avant, zoom arrière)
	+ Gestion du déplacement cliquer-glisser
	+ Saut à une position donnée
	+ Zoom avant avec double-clic sur un point de la carte (optionnel)
	+ Pour tout zoom (double-clic ou molette, avant ou arrière) : conservation du point sous la souris à la même position de l'écran (optionnel)
	+ Gestion des tuiles déjà disponibles dans le cache de l'application (optionnel)
	
### Modalités de remise du code

Pour remettre votre code, vous aurez deux possibilités :

- Par Moodle : un devoir sera créé dans Moodle pour vous permettre de déposer votre code sous forme d'une archive (.zip, .tgz, .tar.gz, .tar.xz, .tar.bz2, .rar). L'archive doit contenir uniquement : les fichiers C++ et les entêtes (.h ou .hpp), ainsi que le fichier projet .pro (mais pas le .user.pro).
- En partageant un dépôt git : dans ce cas, le dépôt doit être accessible publiquement, et son URL sera déposée dans le devoir Moodle, dans un fichier texte nommé `url.txt`. Le code peut être sur github, gitlab ou toute autre instance de service git, du moment qu'il est accessible.

Assurez vous que votre code se compile et s'exécute correctement dans un environnement Linux avec Qt en version 6.X.

Le devoir sera à rendre jusqu'au 4 avril 2025, 23:59:00 GMT+02.

## Architecture

Le but de l'architecture est de faciliter la compréhension, la maintenance, la testabilité et l'évolution du code. En conséquence, le code doit être structuré (l'architecture), de façon à ce que chaque élément (ça peut être une classe, un module, un sous-système) soit concentré sur un seul objectif, clairement délimité, et avec lequel on interagit au travers d'une interface explicitement définie.

Qt vous fournit des classes permettant de suivre certains modèles d'architectures (par exemple, une variante de Model-View-Controller), et C++ vous permet d'architecturer votre code comme vous le souhaitez (ce qui signifie que vous pouvez aussi bien faire des choses correctes et performantes que des horreurs que personne ne voudra aller patcher).

L'objectif de décomposer en éléments ayant un périmètre restreint est multiple :

- faciliter la maintenance : un élément avec un petit périmètre, ne réalisant que peu d'actions est plus facile à comprendre, donc à debugguer ou modifier.
- faciliter les évolutions : si quelque chose doit changer dans l'application (une source de données, un algorithme important, etc.), il est plus facile de ne changer que cet élément sans trop d'impact sur le reste quand le code est bien réparti et structuré.
- faciliter les tests : on sait plus facilement écrire les tests (et en limiter le nombre), si un composant est simple et clair dans ses missions.
- travailler en équipe : généralement, il faut être plus compétent pour comprendre et corriger un code écrit que pour l'écrire. Quand vous travaillez en équipe, vous êtes fréquemment amené à vous approprier le code d'un collègue. Si ce code est mal structuré, vous aurez bien plus de difficultés à réaliser votre travail dessus que s'il est correctement structuré (et documenté, mais ceci sort du cadre de cette ressource). Ça marche également dans l'autre sens, et **dans le temps** !. C'est-à-dire que vous serez vous-même votre propre collègue du futur quand vous devrez reprendre du code que vous avez écrit il y a des semaines/mois/années. Mieux il sera structuré, plus rapide sera votre contribution du moment.

Pour architecturer votre code, vous serez confrontés généralement à au moins deux étapes de conception de l'architecture (en particulier avant d'avoir acquis une bonne expérience). La première étape sera l'étude de conception que vous allez faire en amont de l'écriture du code. La seconde étape sera constituée d'ajustements faits pendant l'implémentation (des écueils non anticipés, de légères modifications des spécifications, etc.). Plus vous serez expérimentés, plus la première étape permettra de prendre de décisions par rapport à la seconde étape.

### Architecture à la conception

Une fois connus les objectifs et les fonctionnalités de l'application qu'on vous demande de développer, vous pourrez décomposer suivant deux directions :

- les fonctionnalités (en détaillant ce qui est attendu avec des actions simples, sans pour autant aller jusqu'à un algorithme qui détaille instruction par instruction)
- le domaine (vue, modèle, contrôle)

Cette décomposition permet d'identifier des ensembles qui peuvent être séparés dans des entités différentes, et des ensembles qui ne le peuvent pas et doivent être regroupés au sein d'une entité donnée. À cette étape, vous pouvez vous aider également en vous posant des questions sur des évolutions non demandées mais crédibles (par exemple, dans notre cas, sur la carte, on pourrait se dire *"Que se passe-t-il si mes tuiles ne font pas 256x256px mais sont fournies en 512x512 ?"*, ou *"Qu'est-ce qui change et qu'est-ce qui ne change pas si mon cache est une base de données relationnelle au lieu de fichiers d'images ?"*, etc.)

Dans le contexte de notre application cartographique, je vous propose de prendre l'exemple d'une partie de l'application concernant l'affichage de la carte. Une façon naïve de procéder à l'affichage consiste à écrire une classe pour l'affichage (une classe héritant de QWidget et dans laquelle on redéfinira notamment la méthode protégée `paintEvent`), et à lui donner comme attributs un gestionnaire d'accès réseau (pour les requêtes de tuiles), une liste des tuiles nécessaires pour remplir l'affichage, les bornes de la zone affichée, ainsi que des méthodes pour gérer les interactions utilisateur (cliquer-glisser, molette) et pour calculer les tuiles nécessaires. Dans une application comme celle ci, c'est encore possible de faire ce type d'architecture, car l'application fait quelques centaines de lignes de code. Cependant, ça la rend peu aisée à maintenir, faire évoluer et tester.

Si on reprend le principe de découper suivant deux axes, on peut sans trop rentrer dans les détails, déterminer les choses suivantes :

- Découpage par domaine :
	+ Vue : charge des images et les affiche pour former une carte; mise à jour par le contrôleur
	+ Modèle : la gestion des tuiles (dispos ou pas, requêtes, réponses aux requêtes); le modèle va également déclencher des mises à jour de la vue, éventuellement en passant par le contrôleur (pas strictement indispensable dans notre cas, mais utile si on envisageait un développement sur le long terme)
	+ Contrôleur : les événements souris, qui appellent des mises à jour de la vue
- Découpage par fonctionnalité :
	+ Afficher une tuile, un ensemble de tuiles
	+ Calculer l'empreinte de l'écran (pixels et tuiles)
	+ Centrer sur un point
	+ Déplacer la vue
	+ Zoomer (avant/arrière)
	+ Vérifier la présence d'une tuile localement
	+ Télécharger une tuile
	
**Exemple détaillé avec le zoom**

Pour le zoom, on va considérer qu'on a les classes suivantes :

- map_display : l'enfant de QWidget qui affiche la carte (classe qui a le rôle de *view*)
- event_manager : une classe pour gérer les événements et leur impact sur l'état de l'application (classe qui prend le rôle de *controller*)
- tile_manager : une classe pour gérer les tuiles (classe qui prend le rôle de *model*)

Au niveau de la classe `event_manager`, une méthode sera définie pour modifier l'attribut du zoom de la zone d'affichage. Cette méthode pourra simplement mettre à jour la valeur du zoom du panneau concerné (dont la classe a un pointeur en attribut), et déclencher une mise à jour de l'affichage (avec la méthode `update` de `QWidget`).

Du côté de la classe `map_display`, les choses sont un peu plus complexes : la méthode `update` appellera à un moment donné la méthode `paintEvent`, que vous devez redéfinir. La classe `map_display` a parmi ses attributs un pointeur vers son `tile_manager`, et un pointeur vers son `event_manager`. Lors de l'appel au `paintEvent`, il faut faire le calcul de l'empreinte de la carte, de son centre, et des tuiles qui seront nécessaires pour remplir cette surface. Ce sont également des attributs maintenus par la classe `map_display`. Les coordonnées des tuiles nécessaires vont être passées au `tile_manager`.

Le `tile_manager` va chercher si une tuile est dans le cache (répertoire `data` de l'application). Si c'est le cas, la tuile sera retournée à `map_display` pour affichage, sinon, une requête au serveur sera faite et la gestion de la réponse à la requête devra également mettre à jour l'affichage. Pour permettre ceci, le `tile_manager` aura un pointeur vers le `map_display`.

Bien entendu, la proposition ci-dessus n'est pas la seule possible, ni la plus performante et flexible. Néanmoins, elle vous donne un exemple de procédure à suivre pour concevoir votre application avec une structure évolutive.

**Réserves sur cette solution**

De nombreux cas d'utilisation, issus du web, font qu'on assimile souvent la base de données (qui serait dans notre cas la liste de tuiles déjà disponibles sur la machine qui exécute l'application) au modèle, les pages affichées à la vue (qui correspondrait dans notre cas à l'affichage de la carte), et les classes permettant de manipuler les vues et les données comme contrôleurs (dans notre application, ce serait le gestionnaire d'événements). Mais ce n'est pas nécessairement si simple dans le cas d'une application comme la nôtre.

En effet, notre modèle est bien plus qu'un modèle, puisqu'il doit se charger de vérifier la présence de tuiles, avant de décider ou non de faire des requêtes vers un TMS, qui peut founir divers types de tuiles (et de formats/tailles). Il serait donc opportun de re-découper cette classe en plusieurs classes spécialisées chacune dans une des tâches.

### Architecture en cours de développement

Il se peut que vous ayez mal prévu certains développement, ou considéré une manière de faire qui s'avérera contre-productive lors du développement (le diable étant dans les détails, il est facile de concevoir les grandes lignes de vos composants et de vous rendre compte qu'une opération qui avait l'air facile conceptuellement ne l'est en réalité pas du tout quand vous passez à l'implémentation). Dans ce cas, il est fréquemment nécessaire de casser des entités en entités plus petites même si elles restent liées fortement, ou de déplacer des rôles d'une entité à une de ses "voisines" (une entité qui est en interfaçage directe avec).

Sur la base de l'application cartographique, un exemple de problème qui peut se poser à vous se situe au niveau de la manipulation de la carte. En effet, on part du principe (quand on conçoit), qu'on définit le centre géographique (i.e. WGS84) de la carte, à partir duquel on peut effectuer si nécessaire des requêtes de téléchargements de tuiles, puis le rendu des tuiles nécessaires. On passe alors à l'implémentation du cliquer-glisser pour déplacer la carte, et on se rend compte que définir dans la vue de la carte son centre en tant que point WGS84 n'est pas pratique : pour obtenir le centre WGS84, il faut trouver la position en pixels dans l'écran, la convertir en position en pixels dans la projection au zoom actuel, puis convertir en WGS84, pour calculer le décalage (en WGS84), en déduire le nouveau centre, et tout reconvertir dans le référentiel des tuiles. Et on peut être amené à se dire qu'il serait plus simple que la vue cartographique soit définie par son centre dans le référentiel projeté en pixels (c'est-à-dire dans une image carrée de 2^zoom sur 2^zoom) et de travailler directement en pixels (puisque les dimensions de la zone d'affichage et les mouvements de la souris sont en pixels).

Une solution à ce problème sera de déplacer la frontière entre la gestion du centrage en WGS84 (uniquement utile quand on passe par un "saut" depuis un résultat Nominatim) et sa conversion dans le référentiel des tuiles.