---
title: 'Il est Grav lui!'
visible: false
header_image_file: 'https://download.tuxfamily.org/passionlinux/distributions_linux/distributions_linux.jpg'
---

Quel jeu de mot pourrie, oui je n'ai pas été loin pour le faire.

![Grav](https://download.tuxfamily.org/passionlinux/logiciels/Screenshot_2020-01-02%20Grav%20-%20A%20Modern%20Flat-File%20CMS.png)

Je sais pas pourquoi, je regarde de nouveau du coté des CMS, peut être que ma solution statique m'ennuie un peu, trop simple, trop routinier, j'ai donc été voir de nouveau les anciens, comme [SPIP](https://www.spip.net/fr_rubrique91.html), [Dotclear](https://fr.dotclear.org/), Wordpress... SPIP sera certainement un choc "enfin" dans son administration, si seulement je retrouvais l'endroit où j'ai vu que l'équipe travail sur une administration bien plus moderne. Je n'arrive plus avec ce genre de CMS, il faut tout un tas de choses comme PHP, MYSQL... Je n'ai plus envie de me prendre la tête avec une base de données, c'est un blog, un simple blog pas besoin d'une armada de fonctions pour un simple blog.

J'ai relancé les [Guppy](https://www.freeguppy.org/), [PluXml](https://www.pluxml.org/) et autres, je n'y arrive pas avec, coté simplicité on y est mais il faut de suite charger dans PluXml un semblant d'éditeur et je n'en trouve pas à mon goût. J'ai goûté au markdown et c'est dure d'en revenir, c'est un langage simple, compréhensible, ça ressemble beaucoup mine de rien à l'éditeur de SPIP et celui-ci a eu longtemps ma préférence.

Donc il me faut un truc aussi simpliste que PluXml avec un éditeur markdown, j'ai trouvé, ou plutôt c'est [Alionet](https://blog.alionet.org/fr) qui m'a mit sur le bon chemin. Ça s'appel [Grav](https://getgrav.org/) et c'est un mixte entre sites dynamiques à la Wordpress et sites statiques comme j'utilise actuellement, ça n'utilise pas de base de données proprement parlé, du moins pas comme on le sous-entend habituellement, pas de MYSQL ou de SQLITE, non à la place ce sont des fichiers plats, un CMS Flat-file comme peut l'être PluXml. Cette fois et contrairement à PluXml, les articles ne sont pas dans un format XML mais de simples fichiers markdown encore plus simple pour être lisibles par l'humain.

Mais avant de parler proprement de Grav, on va situer un peu mes besoins et le contexte. J'utilise un générateur de sites statiques depuis maintenant 2 ans, j'en ai changé mais principalement j'utilise [Hugo](https://gohugo.io/). Il y a toujours des points positifs et des négatifs, dans les +, on a:

- la rapidité,
- la simplicité extrême,
- besoin juste d'un serveur pour stocker le site, pas besoin de PHP ou autre, seulement un serveur apache ou s'approchant,
- la légèreté,
- la sécurité,
- on travaille directement avec les outils de notre PC, un simple éditeur,
- on a aussi le fait qu'on ne s'occupe que de faire vivre notre site sans la gestion, pas besoin de mettre le moteur du site à jour... 

dans les moins, on a:

- on ne peut modifier le site que sur la machine où on a notre "build", à moins de mettre celui-ci sur une clé et de se promener avec,
- c'est du statique, donc on oublie tout ce qui est en dynamique comme les commentaires, à moins là encore de contourner le problème...,
- besoin pour la moindre correction de recommencer toute l'étape de "compilation" du site,
- pour ma part c'est tout.

Quand je suis parti sur du statique, c'est que je voulais avoir le plus rapide, le plus sécurisé, le plus léger, j'en ai eu pour mon grade, j'ai eu exactement ce que j'avais demandé, alors j'admets que pour l'heure le changement n'est pas encore d'actualité, c'est juste l'excitation de changer mais ce que l'on me promet donne le tournis. On me dit que je vais avoir un Wordpress sans base de données et avec un éditeur markdown, où je signe?

Pour l'installer rien de compliqué, on décompresse l'archive, j'ai pris l'archive [Grav+admin](https://getgrav.org/download/core/grav-admin/1.6.19) pour me simplifier la vie car sinon on peut le gérer avec les fichiers et des commandes à la linux. Il ne lui faut seulement un PHP récent pour tourner et rien d'autre.

Je vais le tester en douce, je ferai des captures d'images pour le montrer, on verra si ça arrive à me détacher de mon eldorado qu'est le site statique.