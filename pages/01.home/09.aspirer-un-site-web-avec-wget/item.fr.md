---
title: 'Aspirer un site web avec Wget.'
visible: false
---

Une commande bien pratique, maintes fois utilisé par votre serviteur, pour aspirer site HTTP, FTP et tout un tas d'autres possibilités:

	wget -r -k -np --user-agent=Firefox url-du-site

Une petite explication s'impose:

L'option `-r` pour que le téléchargement soit récursif, télécharge aussi les liens de la page.

Le `-k` reconstitue localement le site, les liens sont donc modifiés pour pointer localement.

Le `-np` pour ne pas remonter dans le répertoire parent.

Et `--user-agent=Firefox` pour faire passer Wget pour un Firefox (a remplacé par tout autre user-agent).

Pour les sites qui demandent une authentification, on pourra ajouter :

`--http-user` et `--http-password`


Ajout via les commentaires (merci aux protagonistes):
On peut aussi rajouter l'option `-E` pour convertir le PHP en HTML, ça permet de faciliter la lecture en locale pour le navigateur.

Il y a aussi ["httrack"](https://www.httrack.com)
