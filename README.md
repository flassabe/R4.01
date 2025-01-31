# TP de la ressource R4.01 - Architecture des applications

Ce dépôt contient les versions markdown/C++ des sujets distribués en PDF via le moodle de l'Université Marie et Louis Pasteur. Il vous sera plus facile grâce à ce dépôt d'obtenir le code des exemples au lieu de le copier/coller à partir des PDF (qui ajoutent des caractères non compilables).

Pour installer Qt et Qt Creator, vous devrez installer les paquets suivants :

 - Sous Fedora :
 ```bash
 sudo dnf install qt-creator qt6-qtbase-devel -y
 ```
 - Sous Debian et affiliés :
 ```bash
 sudo apt update && sudo apt install qtcreator qt6-base-dev -y
 ```
 - Windows/MAC OSX : utilisez l'installeur en ligne, ou les lignes de commandes ci-dessus après installation d'un système Linux (double boot, ou machine virtuelle)

Actuellement, les TP disponibles sont :

 - le TP3, [introduction à Qt et ses widgets](TP3)
 - le TP4, [les événements Qt](TP4)
