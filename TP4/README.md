# TP4 - Les événements Qt

## Architecture d'événements Qt

Qt permet l'interaction avec les éléments de l'interface via deux types de méthodes particulières :

- les signaux
- les slots

Les signaux sont émis par des objets Qt (il faut hériter de `QObject`) lors de changements d'état : clic, saisie de texte, etc. Les slots sont appelés en réponse à des signaux. La correspondance entre signaux et slots se fait en connectant un slot sur un signal.

Définir des signaux et des slots nécessite deux choses :

- une classe qui hérite (directement ou indirectement) de `QObject`
- insérer la macro `Q_OBJECT` au début de la classe.

Par exemple, la classe qui suit pourra définir des signaux et des slots :

```cpp
class une_classe: public QObject {
	Q_OBJECT

public slots:
	void event_received();
	
signals:
	void event_sent();
};
```

On connecte ensuite un signal sur un slot à l'aide de `QObject::connect`. Par exemple, on pourrait connecter entre eux deux objets de type `une_classe` de la manière suivante :

```cpp
// On ajoute à la classe la méthode make_event() :
public:
	void send_event() {
		emit event_sent();
	}
// ...

// Dans le main :
une_classe c1, c2;
QObject::connect(&c1, &une_classe::event_sent, &c2, &une_classe::event_received)
c1.send_event(); // -> provoque l'appel de event_received de c2
```

Les QWidgets ont des signaux définis en fonction de leur rôle (clic pour les boutons, etc.). Ces événements sont décrits dans la documentation sous la section signaux.

## Travail demandé

À partir de l'interface définie au TP précédent, vous définirez le traitement des événements suivants :

- clic sur le bouton : si la case de saisie de texte est vide, ne rien faire. Si elle est renseignée, vider la liste de textes et la remplacer par le texte dans la zone de saisie, répété 5 fois et concaténé de 1 à 5. Par exemple, si la zone de saisie contient "L'IUT c'est trop bien", la liste de texte contiendra :
```
L'IUT c'est trop bien 1
L'IUT c'est trop bien 2
L'IUT c'est trop bien 3
L'IUT c'est trop bien 4
L'IUT c'est trop bien 5
```
- Clic sur File > Quit : quitte l'application
- Clic sur Help > About : affiche une fenêtre avec un texte décrivant l'application (à cette occasion, vous lui donnerez un nom, vous y ajouterez votre nom comme auteur, et une ou deux lignes de description, ainsi que la date).

Si vous n'avez pas terminé le TP précédent, finissez-le avant de passer à la gestion des événements.
