# TP8 - Intégration et architecture

Votre application est maintenant capable de gérer la plupart des interactions avec l'utilisateur et le réseau. Elle est en revanche probablement difficile à faire évoluer du fait de l'empilement des modifications effectuées depuis le premier TP Qt jusqu'à celui où vous avez intégré les tuiles cartographiques. Ce TP va donc viser à perfectionner l'architecture de façon à pouvoir modifier facilement le code et faire évoluer l'application.

Pour architecturer notre application, il est d'abord nécessaire de lister ce qu'elle est censée faire afin d'identifier les rôles de ses différents éléments. Dans les grandes lignes, nous devons fournir les éléments suivants :

- une fenêtre d'application comportant :
	+ une barre de menu avec Fichier -> quitter, et Aide -> À propos
	+ un panneau latéral avec un champ de recherche, un bouton de recherche et une zone affichant les résultats
	+ un panneau central qui affiche la carte de la zone choisie
- une fenêtre *À propos* contenant un texte pour présenter l'application (notamment le ou les auteurs de l'application)