---
title: ' Convertir plusieurs images en PDF.'
visible: false
---

J'ai eu besoin d'envoyer des images que j'avais déjà numérisé dans un format *.JPG*  mais dans un format *.PDF*, ne cherchez pas c'est de l'administratif et puis ce format me permet de regrouper plusieurs images ensemble et dans un seul fichier.

Voici donc quelques commandes pour convertir des images au format PDF.

Tout d'abord installer `imagemagick`:

    sudo apt-get install imagemagick

Conversion de plusieurs images au format PDF:

    $ convert img1.jpg img2.jpg img3.jpg  file.pdf 

Définition de la dimension de la page:

    $ convert -page 800x600 img1.jpg img2.jpg img3 file.pdf

Définition de la dimension de l'image:

    $ convert -size 800x600  1600x1200 img1.jpg img2.jpg img3 file.pdf

Redimensionner l'image:

    $ convert -resize 50% img1.jpg img2.jpg file.pdf

Conversion d'un grand nombre de fichiers au format d'image:

    $ convert *.jpg file.pdf
