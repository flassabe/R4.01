# TP7 -  Les serveurs de tuiles cartographiques

## Tile map server

Le principe d'un serveur de tuiles cartographiques est de fournir des fonds de cartes géographiques sous forme de fragments qui, une fois assemblés, constituent une carte recouvrant la surface d'affichage cible (une partie de fenêtre de navigateur, un widget graphique Qt, etc.). L'intérêt de la segmentation en tuiles est de permettre de charger à la volée des parties de carte sans devoir créer des images spécifiques sur le serveur, et en permettant une réutilisation des tuiles déjà chargées quand on déplace partiellement la vue. Vous avez probablement tous déjà utilisé des serveurs de tuiles, que ce soit sur Google Maps, OSM, Bing Maps, etc. (on remarque ce tuilage par l'apparition de la carte par carrés).

Dans le cas qui nous concerne, c'est-à-dire OSM, les tuiles sont des images PNG de taille 256x256 pixels. Il y a 20 niveaux de zoom (dans les cartes standard), indexés de 0 à 19. Au premier niveau, il y a une seule tuile, aux coordonnées discrétisées 0, 0, 0 (sous la forme zoom, x, y). L'URL de la tuile est donc :
`https://tile.openstreetmap.org/0/0/0.png`

En effet, les URL sont construites de la manière suivante :
`https://tile.openstreetmap.org/zoom/x/y.png`

Pour le reste des zooms, à chaque niveau de zoom supplémentaire, les dimensions en nombre de tuiles sont multipliées par 2 en x et en y. Il y a donc 4 tuiles au zoom=1, 16 tuiles au zoom=2, 64 au zoom=3, etc. Pour résumer, il y a 4^zoom tuiles à un niveau donné. En conséquence, pour choisir les tuiles à afficher sur une zone d'écran, il est nécessaire de connaître deux informations :

- les coordonnées géographiques du centre de la zone (qui peuvent être obtenues par un résultat Nominatim), pour en déduire la tuile centrale
- les dimensions en pixels de la zone d'affichage (qui permettront de calculer les tuiles environnant la tuile centrale)

Les coordonnées géographiques sont exprimées dans le système de coordonnées WGS84 (World Geodetic System 84), par deux valeurs : la longitude (axe x), et la latitude (axe y). Toutes deux sont exprimées par OSM en degrés à partir de

- l'équateur pour la latitude (positive au nord, négative au sud)
- le méridien de Greenwich pour la longitude (positive à l'est, négative à l'ouest)

Attention, avec d'autres sources de données, latitude et longitude peuvent également être exprimées en degrés, minutes, secondes, ce qui nécessite de convertir les minutes et secondes en degrés.

Vous trouverez la description du calcul pour déterminer des coordonnées de tuile à partir d'une coordonnée WGS84 sur le lien suivant : [Slippy Map Tilenames](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames)

Il faut également souligner un point important : le serveur openstreetmap.org filtre les accès à son contenu à une liste blanche de user agents (firefox, chrome, edge, etc. pour les navigateurs, curl et wget pour les utilitaires). Pour votre propre programme, il vous faut une source de tuiles non filtrée. Vous utiliserez donc le serveur suivant :

Deux possibilités s'offrent à vous : vous pouvez utiliser un serveur de tuiles avec enregistrement (par exemple www.maptilesapi.com), ce qui nécessitera d'ajouter des `headers` à votre requête :

```cpp
// Vous remplacerez cette ligne :
_reply = _network_access->get(QNetworkRequest(QUrl(tile_url);

// Par
_url = QString{"https://maptiles.p.rapidapi.com/fr/map/v1/%1/%2/%3.png"}.arg(_zoom).arg(x).arg(y);
QNetworkRequest request{QUrl{_url}};
QHttpHeaders headers{};
headers.append("x-rapidapi-key", "votre clé d'API sur maptilesapi"); // à adapter à votre compte utilisateur
headers.append("x-rapidapi-host", "maptiles.p.rapidapi.com");
request.setHeaders(headers);
_reply = _network_access->get(request);
```

La seconde solution est d'utiliser une machine virtuelle qui vous est fournie sur [cette page](https://info.iut-bm.univ-fcomte.fr/staff/flassabe/). Cette image de machine virtuelle (VM) doit être importée dans VirtualBox. Cette VM contient les informations géographiques uniquement pour la Franche-Comté, contrairement à un service en ligne qui distribue généralement les cartes de l'ensemble du monde. Pour accéder au serveur web de la VM, vous devrez configurer un tunnel vers le port 80 sur la machine hôte, par exemple en redirigeant son port 8080 local vers le port 80 de la machine invitée (la VM). Votre code utilisera alors comme URL la chaîne suivante : `http://localhost:8080/hot/<zoom>/<x>/<y>.png`, avec zoom, x et y remplacés par les valeurs des tuiles recherchées.

## Travail demandé

Vous implémenterez une requête sur un serveur de tuiles pour obtenir une tuile d'affichage cartographique; puis une série de requêtes pour construire une carte complète sur une zone d'affichage définie par les coordonnées de son centre en WGS84, et les dimensions de la fenêtre d'affichage en pixels (hauteur et largeur).

Le principe de l'affichage est le suivant : quand on centre la carte sur une coordonnée, on calcule la tuile correspondant au centre de l'image. À partir de la position de cette tuile dans la zone de la carte, on va calculer le nombre de tuiles à afficher (horizontal et vertical) grâce à deux paramètres : les dimensions de la zone d'affichage (`height` et `width`). En connaissant le nombre de tuiles en hauteur et en largeur, ainsi que la position relative de la tuile centrale (donc les coordonnées x et y en nombres entiers dans la grille globale des tuiles sont connues), il est ensuite possible facilement de trouver les bornes x et y minimales et maximales à télécharger sur le serveur de cartes.
