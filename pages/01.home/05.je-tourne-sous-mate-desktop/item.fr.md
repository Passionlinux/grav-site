---
title: 'Je tourne sous Mate-desktop!'
visible: false
---

Après avoir tant décrié ce bureau que je trouve toujours autant "d'un autre temps", me voici dessus, c'est pas si terrible que ça, on va dire que ça a prit un sacré coup de vieux. À vrai dire, je trouve XFCE moins vieillot, le look à la GNOME2 n'a jamais été pour moi un exemple de modernité, toujours un "je ne sais quoi" qui me donnait envie d'aller voir ailleurs. Regarder par exemple le look de Mate par défaut:

![mate1.20.3-sur-ma-debian](https://download.tuxfamily.org/passionlinux/captures/mate1.20-debian-2018-11-08%2020-38-54.png)

Et GNOME 2.10 en 2005 sur une SUSE pro 9.3, distribution très classe à l'époque:

![gnome2.10-suse9.3](https://download.tuxfamily.org/passionlinux/captures/suse93_03.png)

Ou bien GNOME 2.10 sur une Mandriva 2006:

![gnome2.10-mandriva2006](https://download.tuxfamily.org/passionlinux/captures/mandriva2006_03.png)

Rien, pas de changements significatifs, mise à part le moteur GTK ne tournant plus sur du GTK2 mais en GTK3. Du reste, je suis pas sûr que celui qui est resté avec une distribution tournant sous GNOME 2.32 verrait la différence tellement qu'elle est minime.

Ça me déplaît pas, juste que c'est au bas mot un bureau qui fait réellement son age, c'est-à-dire dix ans de trop! Il était déjà vieillot en 2010 et il fait donc encore plus son age de nos jours. J'ai l'impression qu'on s'accroche à une époque révolue, pas que je pense que GNOME 3 soit forcément le top, mais dans l'ensemble, Mate fait son age, c'est d'un autre temps, presque un autre siècle. À coté de ça, je trouve Xfce bien moins âgé de vue, quoique les deux se ressemblant énormément alors on va dire que c'est pareil. En contre partie, pour ceux qui fuient GNOME 3 --et son bureau un peu dur à prendre en main dans un premier temps car une fois les reconnaissances effectuées, il est bien difficile de revenir à un bureau plus classique--, il reste Plasma 5 qui ne donne pas ce coté un peu dépassé de Mate, au contraire, j'irais volontiers dessus sauf que ma Debian est avec un Plasma 5.8 et que cette version est merdique, il reste aussi et toujours autant bordélique avec un nombre de réglages impressionnants, trop pour moi.

Mais revenons à ce Mate sur une Debian stable, sur Stretch actuellement on a la version 1.16, une éternité pour certain, mais nous avons de la chance, en tout cas les utilisateurs de ce bureau ont de la chance, le mainteneur de celui-ci chez Debian et au contraire de tous les autres bureaux --y compris Xfce qui sort pourtant peu de versions--, propose dans le [dépôt backport](https://wiki.debian.org/fr/Backports) une version plus à jour. Normalement depuis Jessie, il met à disposition la dernière version qui sort entre temps. C'est comme ça que nous avons eu une 1.18 et [une 1.20.3](https://packages.debian.org/stretch-backports/mate-desktop) actuellement, c'est de celle-ci qui est question aujourd’hui.

Tout d'abord, commençons par l'installation, j'étais donc sur une Stretch avec GNOME 3.22 --peut être la raison du choc d'époque dont je souffre--, je commençais un peu à désespérer avec mon PC, je dois avant tout me situer, désolé. Utilisateur de KDE depuis 3.4, un peu de GNOME de 2.10 a 2.32 et depuis le 3.14 entièrement et uniquement, je vois bien où ça me mène: droit vers un changement de matériel, oui mon PC n'est plus en forme, oui il a plus de dix ans maintenant, oui je devrais soit le changer, soit le modifier. Je le vois avec Plasma 5, bien que tout est vivace, j'ai des artefacts insupportables avec depuis la version 5.12 et ce peu importe la distribution (base de RPM ou de DEB), dans le déplacement des fenêtres, dans les menus de Firefox et Thunderbird, c'est pas tout le temps mais par période, ça vient et ça repart, ... Rien de grave, je pense plus à un soucis de carte graphique qui doit chauffer ou être en fin de vie. De toute façon, ce PC a bien un soucis d'ordre matériel puisque les démarrages à froid se font dans la douleur, il fige deux à trois fois sans plus répondre, obligé de couper sauvagement la machine autant de fois qu'il fige. Puis une fois qu'il a chauffé un peu, ça va comme si de rien était! Si, quand il me fait des crises au démarrage, ça s'accompagnent de bips anormaux au moment de l'écran constructeur, des fois aussi les ventilateurs s'affolent...

Donc Plasma pour moi est un bureau parfait, c'est vivace, ça consomme pas, pas beaucoup plus que Mate à vrai dire comme vous pouvez le voir ici:

![plasma5.14-sur-openSUSETumbleweed-htop](https://download.tuxfamily.org/passionlinux/captures/opensusetumbleweed-htop-20181106_223501.png)

![mate1.20-sur DebianStretch-htop](https://download.tuxfamily.org/passionlinux/captures/mate-debian-htop2018-11-08%2020-40-56.png)

Tout irait bien dans le meilleur des mondes si ce n'est que, ça ne va pas, sa configuration est bordélique, on sait plus où donner de la tête --KDE a toujours été comme ça--, on cherche comment optimiser tout ça au lieu de faire ce que l'on doit faire. Donc non, pas pour moi, j'arrête là enfin j'espère --plutôt bien puisque je ne suis pas dessus sur le principal depuis Jessie--.

GNOME-Shell dit GNOME 3 est top pour moi, enfin c'est ce que je pensais avant de le voir avec Wayland qui pour le moment n'est pas utilisable pour moi sans ralentis, c'est un bureau très simple à prendre en main, avec peu de fonctions, qui permet de ce concentrer sur ce que l'on fait et non sur comment le faire, j'aime ça. Sauf que c'est comme tout, contrairement à KDE qui a su se refaire une santé et faire un régime pour sa consommation mémoire, GNOME ne sait pas si bien faire, il dépasse de loin les autres bureaux, c'est une horreur, 1 Go de RAM à son lancement même si j'ai réussi à le faire tomber dans les 700 Mo (je ne sais pas comment et n'y arrive plus). Sa version 3.30 à été optimisé et [les développeurs ont travaillé](https://linuxfr.org/news/parution-de-gnome-3-30#toc-un-coup-de-boost) sur sa consommation de RAM, mais c'est Wayland qui me pose un soucis, il est par défaut sur cette version, si bien que l'on peut avoir des lacunes avec certains programmes sous Xorg, sauf que chez moi, Wayland n'est pas des plus sympa et clairement il doit être lui aussi optimisé car quand nous avons un menu GNOME qui freeze 1 à 2 secondes avant de répondre bloquant par là même notre souris et l'interface entière, chose que Xorg ne faisait pas, c'est qu'il y a anguilles sous roches. Peut être dû à mon souci matériel, sait-on jamais mais reste étrange en sachant que c'est le cas sur plusieurs machines.

Xfce est et reste encore à ce jour un bureau sympa, je fais des excursions dessus de temps en temps, quand le reste m'insupporte mais soyons honnête deux minutes, il n'a jamais été pensé pour être complet, il faut donc aller chercher des applications dans les deux autres environnements et même si Xfce s'accorde à essayer de garder un semblant d'homogénéité, un certain GNOME fait tout pour que ses applications soient bien reconnaissables dans les autres environnements en faisant en sorte que ça garde le look de GNOME.

Reste pas grand chose et c'est là où je me suis dis, pourquoi pas! Pourquoi pas Mate? Pourquoi ne pas essayer, j'ai pas dit adopter, je dis essayer ce GNOME 2 vu qu'il rencontre un certain succès avec Mint linux ou Manjaro.

Donc, ceci étant dit, parlons de son installation, sous Debian un `apt install mate-desktop-environment-extras` suffira pour l'installer dans une version 1.16. Maintenant si comme moi vous voulez une version récente, la dernière en l'occurence, il faudra faire:

    # apt -t stretch-backports install mate-desktop-environment-extras

Notez bien le `-t stretch-backports` qui fait en sorte de dire à APT d'aller le chercher dans le dépôt *stretch-backports*, lui et toutes ses dépendances. Par la suite, si il y a des updates, il ira directement le mettre à jour depuis backport.

Notez également qu'avec cette commande, j'installe l'environnement complet + les extras.

Que dire dessus qui n'a pas été dit depuis le début de ce siècle? C'est léger mais c'est pas non plus un truc de fou, Plasma 5.14 fait aussi bien voir mieux pour bien plus. Non rien, c'est du vu et revu, ça fonctionne et c'est déjà ça. On va garder ce truc quelques temps et on verra, même si je pense que je vais faire un retour sous GNOME qui ne quittera pas la version 3.22 tant que Stretch reste la version stable.
