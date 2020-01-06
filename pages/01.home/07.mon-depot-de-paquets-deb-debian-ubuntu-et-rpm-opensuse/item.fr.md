---
title: 'Mon dépôt de paquets DEB (Debian, Ubuntu) et RPM (openSUSE).'
visible: false
---

Le projet openSUSE est un modèle à prendre en compte, ils ne font pas leurs soupes dans leurs coins et pour eux uniquement, ce qu'ils font est pour tous, peut importe. Leurs OBS (une sorte de sourceforge) est un exemple de cette ouverture vers les autres. 

Depuis OBS, on peut faire des paquets pour toutes (ou presque) les plus importantes distributions, des RPM pour openSUSE (bien-sur) mais aussi pour Fedora, Mageia, ..., des DEB pour Debian, Ubuntu, des paquets pour Archlinux...

Ça fait un moment que j'utilise OBS(OSC) pour faire des paquets ou du moins maintenir à jour des paquets pour la distribution openSUSE, en plus d'en faire aussi pour Debian/Ubuntu. Jusqu’à présent, je ne voyais pas l’intérêt de le signaler, du moins pour openSUSE c'est toujours pas intéressant puisque les paquets qui sont [dans mon dépôts](https://download.opensuse.org/repositories/home:/seb95passionlinux/) sont intégrés --poussé-- dans les dépôts officiels et ensuite une fois accepté, je les vire ce qui fait qu'il y en a peu. Je donne l'adresse et la méthode pour l'ajouter juste pour les personnes me demandant par mail mon home d'OBS. 

Pour openSUSE, on peut fouiller par https via son navigateur adoré depuis l'adresse https://download.opensuse.org/repositories/home:/seb95passionlinux/.

Pour openSUSE Tumbleweed, exécutez en tant que root :

    zypper addrepo https://download.opensuse.org/repositories/home:seb95passionlinux/openSUSE_Tumbleweed/home:seb95passionlinux.repo
    zypper refresh

Pour openSUSE Leap 15.1, exécutez en tant que root :

    zypper addrepo https://download.opensuse.org/repositories/home:seb95passionlinux/openSUSE_Leap_15.1/home:seb95passionlinux.repo
    zypper refresh

Pour openSUSE Leap 15.0, exécutez en tant que root :

    zypper addrepo https://download.opensuse.org/repositories/home:seb95passionlinux/openSUSE_Leap_15.0/home:seb95passionlinux.repo
    zypper refresh

Pensez à accepter la clé.

Jusqu'à présent du coté de Debian ce n'était pas intéressant car les paquets se trouvaient bien sur un dépôt mais qui ne pouvait pas être pris en compte par APT --quelque chose de ce style--, sauf que depuis pas si longtemps --j'ai vu ça la semaine dernière--, ce soucis n'est plus et il est facile d'ajouter mon home d'OBS pour Debian dans son source.list comme un dépôt classique.

Du coup j'en profite pour partager mes paquets rétro-portés depuis SID pour Stable, principalement des jeux.
Alors avant tout pour rassurer certains, je vais expliquer comment je fais les paquets, je prends les sources depuis Debian --principalement mais pas que--, ensuite j'utilise `Pbuilder` --en fait c'est `Pdebuild`, mais c'est pareil--, ce qui me fait de beaux paquets bien propres. Puis enfin je balance sur OBS mes 3 fichiers sources, le fichier DSC, l'archive des sources amonts et l'archive de la débianisation. Rien de plus sinon que j'ajoute généralement à la version "prschav".

Pour Debian/Ubuntu, on peut fouiller par https via son navigateur adoré depuis l'adresse https://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/.

Pour Debian Unstable, exécutez en tant que root :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/Debian_Unstable/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/Debian_Unstable/Release.key -O Release.key
    apt-key add - < Release.key
    apt-get update

Pour Debian Testing, exécutez en tant que root :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/Debian_Testing/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/Debian_Testing/Release.key -O Release.key
    apt-key add - < Release.key
    apt-get update

Pour Debian 10, exécutez en tant que root :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/Debian_10/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/Debian_10/Release.key -O Release.key
    apt-key add - < Release.key


Pour xUbuntu 19.10, exécutez :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/xUbuntu_19.10/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list"
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/xUbuntu_19.10/Release.key -O Release.key
    sudo apt-key add - < Release.key
    sudo apt-get update

Pour xUbuntu 19.04, exécutez :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/xUbuntu_19.04/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list"
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/xUbuntu_19.04/Release.key -O Release.key
    sudo apt-key add - < Release.key
    sudo apt-get update

Pour xUbuntu 18.04, exécutez :
Keep in mind that the owner of the key may distribute updates, packages and repositories that your system will trust (more information).

    sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/seb95passionlinux:/debian/xUbuntu_18.04/ /' > /etc/apt/sources.list.d/home:seb95passionlinux:debian.list"
    wget -nv https://download.opensuse.org/repositories/home:seb95passionlinux:debian/xUbuntu_18.04/Release.key -O Release.key
    sudo apt-key add - < Release.key
    sudo apt-get update

Pour le moment c'est tout récent mais plusieurs autres paquets rétro-portés devraient rejoindre la troupe.
