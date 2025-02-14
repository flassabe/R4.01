# TP5 - Affichage d'une carte

Dans cette session, vous utiliserez les tuiles fournies avec ce sujet pour afficher une carte complète dans une zone de dessin Qt.

## Introduction au dessin 2D avec Qt

Pour dessiner des formes par pixel (*raster*), il est nécessaire de définir une sous-classe de `QWidget`. Cette sous-classe de `QWidget` que vous définirez, doit redéfinir la méthode protégée [paintEvent](https://doc.qt.io/qt-6/qwidget.html#paintEvent).

Pour afficher une image, il faut plusieurs étapes :

- Charger l'image à l'aide de la classe QPixmap, en passant le chemin du fichier image au constructeur de la classe
- définir la partie de l'image à afficher dans un rectangle (instance de [QRect](https://doc.qt.io/qt-6/qrect.html)) dont l'origine et la taille définissent quelle portion de l'image est affichée
- définir le rectangle (origine et dimensions) où l'image sera affichée dans le widget d'affichage
- appeler la méthode `drawPixmap` de `QPainter` pour afficher l'image

## Exemple simple d'affichage d'une tuile au centre

Les deux fichiers utilisés par le main sont mainwindow.h :

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QWidget>
#include <QScopedPointer>
#include <QMainWindow>

class drawing_surface: public QWidget {
    Q_OBJECT
public:
    drawing_surface(QWidget *parent=nullptr): QWidget(parent) {}

protected:
    void paintEvent(QPaintEvent *event) override;
};

class main_window: public QMainWindow {
    Q_OBJECT

    QScopedPointer<drawing_surface> surface;

public:
    main_window(QWidget *parent=nullptr);
};

#endif // MAINWINDOW_H

```

Et mainwindow.cpp :

```cpp
#include "mainwindow.h"
#include <QPainter>

main_window::main_window(QWidget *parent): QMainWindow(parent) {
    surface.reset(new drawing_surface(this));
    setCentralWidget(surface.get());
}

void drawing_surface::paintEvent(QPaintEvent *event) {
    QSize widget_size = size();
    QPixmap pixmap{"tile.png"};
    QRectF source(0.0, 0.0, pixmap.size().width(), pixmap.size().height());

    QPainter painter(this);
    int half_height = pixmap.size().height() / 2;
    int half_width = pixmap.size().width() / 2;
    int screen_half_height = widget_size.height() / 2;
    int screen_half_width = widget_size.width() / 2;
    QRectF target(screen_half_width - half_width,
                  screen_half_height - half_height,
                  2 * half_width,
                  2 * half_height);
    painter.drawPixmap(target, pixmap, source);
}

```

## Travail demandé

Pour afficher la carte, vous devrez passer par les étapes suivantes :

- Déclarer et affecter à votre élément central de fenêtre principale une instance d'une classe dérivée `QWidget` et dédiée à l'affichage de la carte. Redéfinir sa méthode `paintEvent`.
- Pour chaque tuile, il faut afficher son image à la position correcte de l'écran	
	
Pour obtenir vos tuiles, vous choisirez un niveau de zoom entre 5 et 15. La carte à afficher sera centrée sur les coordonnées WGS84 (6.839349, 47.64263) sous forme (longitude, latitude). Consultez l'introduction du TP7 pour une description du système de tuiles cartographiques. Vous nommerez vos fichiers Z-X-Y.png lorsque vous les téléchargez pour vous permettre d'afficher facilement la bonne tuile au bon endroit.
