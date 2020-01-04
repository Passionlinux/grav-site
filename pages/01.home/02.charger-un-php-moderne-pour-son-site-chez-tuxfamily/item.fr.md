---
title: 'Charger un PHP moderne pour son site chez Tuxfamily'
date: '02-01-2020 20:19'
visible: false
---

Comme vous pouvez le savoir, je suis chez Tuxfamily pour mon blog, c'est très bien et je les remercie vivement. Le soucis comme il peut se présenter et que je ne m'en étais pas rendu compte n'ayant actuellement que le besoin d'un serveur Web (Apache ou autre) puisque j'utilise HUGO pour générer mon site en static, c'est que pour des applications du type CMS comme Wordpress et tant d'autres, il nous faut PHP et la version par défaut de PHP n'est plus supportée par des CMS trop modernes comme pour [Grav](https://getgrav.org/) qui nous engueule:

> You are running PHP 5.6.40-12+0~20190902.20+debian10~1.gbpc72558, but Grav needs at least PHP 7.1.3 to run.

Il suffit d'ajouter ceci à notre .htaccess:

    AddType application/x-httpd-php7 .php
    

[Source](https://forum.tuxfamily.org/post/3190/#p3190)
