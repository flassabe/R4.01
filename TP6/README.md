# TP6 - Le réseau avec Qt

## Principe de fonctionnement

Le réseau dans Qt repose comme les autres modules sur l'association de signaux et de slots. La différence avec les widgets se joue simplement sur la source des signaux qui sont émis, dans le cas du réseau, par des événements réseau, tels que connexion, déconnexion, réception de données, etc. Dans ce TP, nous utiliserons les API de Qt pour accéder à des données exposées par un serveur web. Le réseau à plus bas niveau (sockets) ne sera pas abordé, d'une part parce que l'objectif de notre application requiert de procéder à des requêtes HTTPS, d'autre part parce que vous connaissez le principe des sockets grâce à une autre ressource du B.U.T.

### Le gestionnaire de réseau

Avant toute chose, il est nécessaire d'ajouter le module réseau au fichier projet, grâce à la ligne suivante :

```cpp
QT += network
 ```

Qt nécessite ensuite d'instancier un gestionnaire de réseau pour effectuer des requêtes web. La classe à instancier est un `QNetworkAccessManager`. Comme toute classe issue de `QObject`, son constructeur prend en paramètre un pointeur sur son parent.

### La requête

Pour obtenir des données par HTTPS, il vous faut une requête qui détermine l'URL accédée, c'est-à-dire la ressource demandée au serveur. Une requête est de type `QNetworkRequest`. Une requête prend en paramètre une URL de type `QUrl`, qui peut être construite à partir d'une chaîne C standard entre double-quotes :

```cpp
QNetworkRequest query{QUrl{"https://www.whatismypublicip.com"}};
```

Le gestionnaire de réseau peut ensuite envoyer cette requête au serveur via un des modes possibles (get, post ou put). Il est nécessaire de connecter le signal `finished` du gestionnaire à une classe et un slot qui traiteront la réponse à la requête :

```cpp
#include <QNetworkAccessManager>
#include <QNetworkRequest>

QNetworkRequest query{QUrl{"https://www.whatismypublicip.com"}};
QNetworkAccessManager manager{};
manager.get(query);
my_reply reply{};
QObject::connect(&manager, &QNetworkAccessManager::finished, &reply, &my_reply::read_response);
```

### La réponse

À la réception de la réponse d'une requête, un signal est émis. Vous devez connecter le signal à un slot pour pouvoir exploiter la réponse. La réponse est présentée sous forme d'une `QNetworkReply`, qui est un enfant de `QIODevice`. Son contenu peut donc être manipulé (presque) comme n'importe quelle entrée-sortie. Pour avoir un slot, il vous faut donc une classe dérivée plus ou moins directement de `QObject` :

```cpp
#include <QObject>
#include <QNetworkReply>
#include <iostream>

class my_reply: public QObject {
public:
	my_reply(): QObject() {}
	~my_reply() {}
	
public slots:
	bool read_response(QNetworkReply *reply);
};

bool my_reply::read_response(QNetworkReply *reply) {
	while (reply->canReadLine()) {
		cout << reply->readLine().toStdString();
	}
	return true;
}
```

## Openstreetmap et Nominatim

Openstreetmap (abrégé en OSM) est un service de cartographie open source, crowdsourced et open data, c'est-à-dire que son code est distribué librement (open source), ses données sont fournies par des contributeurs quels qu'ils soient (crowd sourced) et accessibles librement (open data). Nominatim est le service de recherche qui permet, à partir d'une chaîne de caractères recherchée, d'obtenir des résultats correspondant associé à leur géocodage (donc leur position géographique). L'exemple d'écran du TP3 montre un résultat de recherche du terme "Paris" dans Nominatim.

Une documentation détaillée de ce qui est abordé dans ce TP concernant Nominatim est disponible sur le [wiki du projet Nominatim](https://nominatim.org/release-docs/develop/api/Overview/).

Brièvement, vous constuisez une requête sous la forme d'une URL de type GET, que vous envoyez au serveur. Celui-ci répond par un contenu sous forme d'une chaîne de caractères représentant une structure, soit de JSON, soit de XML.

Il vous sera conseillé d'utiliser une URL de la forme suivante :

`https://nominatim.openstreetmap.org/search?q=YOUR_SEARCH&format=xml&polygon=0&addressdetails=0`

où YOUR_SEARCH sera remplacé par l'expression recherchée. Vous pouvez changer le format de sortie, en fonction de ce que vous serez le plus à même de savoir analyser en C++. Les formats possibles sont :

- json
- xml
- geojson
- geocodejson
- jsonv2 (valeur par défaut)

Qt propose un parser de JSON (QJsonDocument), ainsi qu'un parser de XML (QXmlStreamReader). La réponse contient notamment les coordonnées des objets retournés.

## Mise en application

Vous devrez implémenter une requête sur le service Nominatim d'Openstreetmap pour obtenir les réponses correspondant à une chaîne de recherche. Les résultats de cette requête doivent être listés dans la zone de liste de l'interface (en affichant le champ `display_name` et en associant ses coordonnées à l'entrée, de manière à pouvoir y accéder dans les séances ultérieures).
