---
title: 'test de page defaut'
visible: false
---

Comme je voulais voir, je fais juste un test comme ça!

Remplacer un serveur de fichiers Windows dans un domaine, par un serveur sous GNU/Linux n’est pas simple. La documentation est souvent en anglais et on a tendance à croire que Samba peut tout faire.



Je vous présente un cas concret et fonctionnel qui permet de faire un serveur Samba, membre d’un domaine sous Microsoft Active Directory. Grâce aux ACL (_[[Access Control List]]_), nous aurons une gestion fine des droits sur les partages comme avec un serveur sous Windows Server avec NTFS. L’exemple est sous Ubuntu, mais transposable à d’autres distributions GNU/Linux.

----

[La documentation de Samba qui m’a aidé](https://wiki.samba.org/index.php/Joining_a_Samba_DC_to_an_Existing_Active_Directory)
[Documentation partielle d’Ubuntu-fr, pour faire un contrôleur de domaine Samba](https://doc.ubuntu-fr.org/tutoriel/samba_ad_dc_members)

----

Introduction
============
Je suis administrateur système en entreprise. Il y a deux ans je me suis demandé comment remplacer nos serveurs de fichiers Windows 2008r2 par du GNU/Linux. Ça mʼa pris deux ans (à temps perdu) pour trouver de la documentation et comprendre comment ce foutu truc pouvait fonctionner. Lʼéchéance de la fin du support de Windows 2008r2 arrivant rapidement, je m’y suis attelé sérieusement fin août 2019.
    
Nous nʼavons plus de serveur de fichiers sous Windows depuis mi‑septembre : les vingt-cinq partages réseau sont répartis sur six machines virtuelles (VM) sous Ubuntu 18.04 (une machine virtuelle sur chaque hyperviseur de notre grappe de serveurs [Proxmox VE](https://fr.wikipedia.org/wiki/Proxmox_VE). Les [Active Directory](https://fr.wikipedia.org/wiki/Active_Directory) tournent encore sous Windows, ce sera la prochaine étape.
    
Ma grande difficulté a été de comprendre qu’on ne pouvait pas tout faire dans Samba, notamment définir les droits dʼaccès. Pour faire un partage [[CIFS]], il nous faut différentes couches logicielles, chacune faisant son boulot dans la plus grande tradition unixienne.



# Résumé du fonctionnement
## Samba
Samba va permettre au client de se connecter en exportant le partage, il va regarder si le client a le droit de se connecter (uniquement les utilisateurs authentifiés du domaine). Nous nʼallons pas définir avec Samba qui a le droit de lire ou écrire.



## Les ACL
Utilisées avec le système de fichiers ext4, les ACL permettent dʼautoriser les accès aux fichiers (genre on nʼautorisera pas les commandes numériques des machines outils dans les ateliers à aller sur le partage de la comptabilité). Mais plus finement, on pourra interdire à un groupe ou une personne d’écrire dans un sous‑répertoire (exemple réel : les douanes renvoient par Internet via un programme d’exportation, des fichiers XML que nous devons conserver dix ans, les commerciaux doivent pouvoir les lire mais pas les modifier ; donc, avec les ACL, nous empêchons de modifier ce répertoire à tout le monde, sauf à lʼutilisateur avec lequel tourne ce programme d’exportation). 



## LVM
Pour éviter que les rigolos du marketing ne prennent trop dʼespace disque, nous voulons limiter lʼespace de stockage disponible par partage. Sous GNU/Linux, les quotas sont par utilisateur ou groupe, cʼest pratique pour les espaces de stockage personnels, mais ce nʼest pas fonctionnel dans le cadre d’un département. LVM est génial dans ce cas, on crée un volume logique (LV) par partage, et nous pouvons facilement l’agrandir si besoin. Je pense que ça doit être utilisable sur ZFS ou Btrfs (pour autant quʼon ait les ACL).



Prérequis
=========
Les prérequis sont :
    
- avoir un domaine avec Active Directory fonctionnel (cʼest nécessaire dans le cadre de cet article) ;
- avoir des notions dʼadministration GNU/Linux : savoir ce quʼest LVM, savoir installer un GNU/Linux, connaître SSH et utiliser un éditeur en terminal (`nano` est très simple), etc. ;
- avoir des notions dʼadministration Windows (savoir ce quʼest un Active Directory, un partage réseau, connaître les droits NTFS, les droits des partages…) ;
- avoir un GNU/Linux opérationnel (dans notre cas une Ubuntu serveur 18.04), qui a une adresse IP, un nom de machine qui correspond au nom quʼon lui donnera dans le domaine et qui est dans la zone DNS de votre domaine (`IN A`).



Préparation de la machine virtuelle
===================================
Donc, dans notre exemple, il nous faut une machine virtuelle sous Ubuntu 18.04 à jour, avec deux disques virtuels (le premier pour le système, le deuxième pour les partages). Bien sûr, dans ce cas, le choix dʼUbuntu est complètement arbitraire, vous pouvez le faire avec votre distribution préférée (attention, hors du monde des dérivés de Debian, les fichiers de configuration peuvent être stockés différemment).
    
**Attention**, dans le cadre de Proxmox, il nous faut du KVM et pas de conteneur LXC (car nous avons besoin des ACL, et nous ne voulons pas triturer le système de fichiers de notre hôte). Bien sûr, si vous êtes sous Proxmox, vos disques sont dans [Ceph](https://fr.wikipedia.org/wiki/Ceph), dans du LVM, du ZFS ou dans un LUN ; si vous utilisez un disque virtuel au format QCOW2, vous allez vivre dangereusement.



Installation des logiciels d’authentification sur le domaine
------------------------------------------------------------
Nous allons installer les logiciels nécessaires pour le bon fonctionnement de Samba dans le domaine :
    
- `realmd` est un orchestrateur qui va utiliser les logiciels PAM, NSS, winbind, etc., pour communiquer avec le domaine ;
- `winbind` sert à faire de l’authentification sous GNU/Linux avec un Active Directory.
    
```bash
apt install acl realmd libnss-winbind
apt install winbind
```







Créer lʼespace de stockage des partages
---------------------------------------
### Créer la partition qui sera le support de vos partages
Pour pouvoir gérer des partages avec des espaces de stockage différents, nous allons utiliser le gestionnaire de volumes logiques [[LVM]]. Il nous faut tout dʼabord une partition pour accueillir le groupe de volumes (VG) ou les volumes logiques (LV) seront stockés. Nous pouvons la créer avec `fdisk` ou un autre outil, sur un disque nommé ici `sdb` : 
    
```sh
fdisk /dev/sdb
```
    






Entrer `n` pour créer une nouvelle partition, accepter les choix par défaut. Entrer `w` pour enregistrer et sortir.



### Créer le groupe de volumes LVM
On nomme ce groupe de volumes « samba » par commodité. On créera les volumes logiques (LV) plus tard :
    
```sh
vgcreate samba /dev/sdb1
```







Joindre la machine virtuelle au domaine
---------------------------------------
Comme un exemple est plus simple pour expliquer, nous allons dire que notre domaine est _example.com_. Dans ce cadre `ad001.example.com` est notre premier serveur Active Directory et `ad002.example.com` notre deuxième. Ils tournent encore sous Windows.



### Installer Kerberos
`apt install krb5-user`
    
Kerberos va permettre l’authentification des utilisateurs via le domaine. Pour cela nous devons le renseigner sur notre domaine en configurant le fichier `/etc/krb5.conf` :
    
```bash
# Ajouter dans [realms], à la fin des autres entrées par exemple
 EXAMPLE.COM = {
                kdc = ad001.example.com  # l’adresse de notre premier ad (serveur active directory)
                kdc = ad002.example.com  # l’adresse de notre deuxième ad 
                admin_server = ad001.example.com  # l’AD principal
                default_domain = example.com  # notre domaine, dans ce cas example.com
        }

# et dans [domain_realm], à la fin
        .example.com = .EXAMPLE.COM
        example.com = EXAMPLE.COM

# en bas du fichier, ajouter ce nouveau segment si vous avez des dinosaures dans votre réseau (vieux Windows, par exemple)
[login]
        krb4_convert = true
        krb4_get_tickets = false
```







### Installer Samba
`apt install samba`
    
Samba cʼest le cœur, nous allons le configurer comme serveur de fichiers membre du domaine _example.com_. Voici les modifications à faire dans le fichier `/etc/samba/smb.conf`, le reste on ne le touche pas, on laisse la configuration par défaut :
    
```bash
# Dans workgroup nous mettons le nom NetBIOS de notre domaine (sans le .com et en majuscules)
workgroup = EXAMPLE
# ici nous pouvons cacher des fichiers qui ne seront pas visibles par les utilisateurs (facultatif)
hide files = /lost+found/
# on définit le rôle de notre serveur (dans notre cas membre d’un domaine)
server role = member server
# nous activons la gestion des ACL dans Samba
# rajouter en dessous de map to guest = bad user
nt acl support = yes
inherit acls = Yes
username map = /etc/samba/user.map
map acl inherit = Yes
map archive = no
map hidden = no
map read only = no
map system = no
store dos attributes = yes
inherit permissions = Yes
security = ADS
realm = EXAMPLE.COM
# fin de l’insertion en dessous de map to guest = bad user
```







Enfin, on configure Samba pour qu’il utilise les utilisateurs du domaine à l’aide de la correspondance des identifiants. Voici les modifications à faire dans `/etc/samba/smb.conf` :
    
```bash
# décommenter la ligne template shell = /bin/bash
# en dessous de cette ligne rajouter ce qui suit
idmap config example : schema_mode = rfc2307
idmap config example : range = 10000000-29999999
idmap config example : default = yes
idmap config example : backend = rid
idmap config * : range = 20000-29999
idmap config * : backend = tdb
winbind use default domain = true
```







### Installer un serveur de temps
`apt install ntp`
    
Dans un domaine, toutes les machines (clients ou serveurs) doivent être à la même heure, au risque de ne pas pouvoir se connecter ou accéder à des ressources. Sous GNU/Linux, nous allons utiliser le programme NTP pour être à la même heure que le domaine. Nous allons prendre comme référentiel le serveur Active Directory principal. On configure le fichier `/etc/ntp.conf` :
    
```bash
# Commenter (avec un #) les pools x.ubuntu.pool… et le pool ntp.ubuntu.com
# en dessous rajouter
NTPSERVERS="ad001.example.com"

# options supplémentaires pour ntpupdate (je ne sais pas pourquoi mais c’est là dans les docs)
NTPOPTIONS="-u"
```







### On plonge la machine virtuelle dans lʼAD (Active Directory)
Cʼest équivalent à joindre un domaine sur un ordinateur sous Windows.
    
Nous allons redémarrer NTP et Samba pour prendre en charge nos modifications :
    
```sh
service ntp restart
service smbd restart
```
    






Avec la commande `net ads` nous allons joindre la machine virtuelle au domaine.
Remplacer _Administrator_ par l’utilisateur administrateur de votre domaine, _NOMDUSERVEUR_ par le nom de votre serveur, _example.com_ par le nom DNS de votre domaine. Ça va vous demander un mot de passe, vous devrez mettre le mot de passe de lʼadministrateur de votre domaine.
    
```sh
net ads join -S ad001 -U Administrator joined NOMDUSERVEUR to realm example.com
```
    






Vous devez voir en réponse :
    
```sh
Joined 'NOMDUSERVEUR' to dns domain 'EXAMPLE.COM'
```
    






Si vous voyez `DNS update failed: NT_STATUS_UNSUCCESSFUL`, ce nʼest pas grave, il faudra ajouter vous-même le nom de la machine dans votre zone DNS.



Nous allons redémarrer winbind, vu que nous sommes dans lʼAD :
    
```sh
service winbind restart
```







### Configurer NSS
Nous allons configurer le _Name Service Switch_, c’est un programme qui va chercher, entre autres, les infos sur les utilisateurs lors de leur connexion (habituellement `/etc/passwd` et `/etc/group`). Dans notre cas, nous allons indiquer en premier la méthode « _files_ » (les utilisateurs locaux) et en second _winbind_, qui est un programme qui va interroger le domaine Windows.
    
```sh
# Modifier /etc/nsswitch.conf comme ci‑dessous
passwd:         files winbind
group:          files winbind
shadow:         files winbind
```







### On teste pour vérifier si tout est bon jusqu’à maintenant
```sh
realm discover -v example.com
```







Vérifiez que ça vous remonte les adresses IP des AD. Le résultat doit contenir :
    
```sh
Performing LDAP DSE lookup on: x.x.x.x (une fois par serveur AD)
```







Préparation de PAM
------------------
PAM signifie _Pluggable Authentication Modules_. NSS va interroger lʼAD et, grâce à PAM, nous allons autoriser lʼutilisateur à se connecter au travers de Samba. Par défaut, PAM autorise seulement les utilisateurs locaux, nous devons modifier plusieurs de ses mécanismes dʼauthentification pour quʼil accepte les utilisateurs du domaine.



### Le mécanisme account
Le mécanisme _account_ fournit une seule primitive. Il vérifie si le compte demandé est disponible : si le compte n’est pas arrivé à expiration, si lʼutilisateur est autorisé à se connecter à cette heure de la journée, etc. ([dʼaprès Wikipédia](https://fr.wikipedia.org/wiki/Pluggable_Authentication_Modules#Modules_PAM)).
    
```bash
# Modifier le fichier /etc/pam.d/common-account comme ci-dessous
account [success=2 new_authtok_reqd=done default=ignore]        pam_unix.so
account [success=1 new_authtok_reqd=done default=ignore]        pam_winbind.so
```
    






Après avoir enregistré, taper la commande suivante :
    
```
pam-auth-update
```
    






Répondre `no`.



### Le mécanisme auth
Le mécanisme _auth_ fournit deux primitives ; il assure l’authentification réelle, éventuellement en demandant et en vérifiant un mot de passe, et il définit des « certificats dʼidentité » tels que lʼappartenance à un groupe ou des « tickets » Kerberos ([d’après Wikipédia](https://fr.wikipedia.org/wiki/Pluggable_Authentication_Modules#Modules_PAM)).
    
``` bash
# Modifier le fichier /etc/pam.d/common-auth comme ci‑dessous
auth    [success=2 default=ignore]      pam_unix.so nullok_secure
auth    [success=1 default=ignore]      pam_winbind.so krb5_auth krb5_ccache_type=FILE cached_login try_first_pass
```







### Le mécanisme password
Le mécanisme _password_ fournit une seule primitive : il permet de mettre à jour le jeton d’authentification (en général un mot de passe), soit parce qu’il a expiré, soit parce que lʼutilisateur souhaite le modifier ([d’après Wikipédia](https://fr.wikipedia.org/wiki/Pluggable_Authentication_Modules#Modules_PAM)).
    
```bash
# Modifier le fichier /etc/pam.d/common-password comme ci-dessous
password        [success=2 default=ignore]      pam_unix.so obscure sha512
password        [success=1 default=ignore]      pam_winbind.so use_authtok try_first_pass
```







### Le mécanisme session
Le mécanisme _session_ fournit deux primitives : mise en place et fermeture de la session. Il est activé une fois quʼun utilisateur a été autorisé afin de lui permettre dʼutiliser son compte. Il lui fournit certaines ressources et certains services, par exemple en montant son répertoire personnel, en rendant sa boîte aux lettres disponible, en lançant un agent SSH, etc. ([d’après Wikipédia](https://fr.wikipedia.org/wiki/Pluggable_Authentication_Modules#Modules_PAM)).
    
```bash
# Modifier le fichier /etc/pam.d/common-session comme ci‑dessous, après la ligne :
# and here are more per-package modules (the "Additional" block)
session required        pam_unix.so
session optional        pam_systemd.so
session optional        pam_winbind.so
session optional        pam_mkhomedir.so
session required        pam_mkhomedir.so skel=/etc/skel
```







Dernières finitions
-------------------
On reconfigure le fuseau horaire (cette commande est propre à Ubuntu, Debian et leurs dérivées) :
    
```sh
 dpkg-reconfigure tzdata
```
    






Mettre Europe → Paris si vous êtes sur le fuseau de Paris.
    
Et pour finir, on redémarre :
    
```
reboot
```
    






La machine virtuelle est prête pour accueillir son premier partage.



Création d’un partage
=====================
Pour l’exemple, nous allons créer un partage dénommé « salami » de 80 Gio sur notre machine virtuelle.



Préparation
-----------
Tout d’abord, une précaution :
    
```
vgdisplay 
```
    






Regardez _Free PE / Size_ pour vérifier qu’il y a assez de place disponible. Ben oui, cʼest idiot, mais ça va mieux marcher sʼil y a encore de la place !



Création du partage
-------------------
Nous créons le point de montage :
    
```
mkdir /home/salami
```







### Création du volume logique et formatage du système de fichiers en ext4
```
lvcreate -n salami -L 80g samba
mkfs -t ext4 /dev/samba/salami
```







### Montage du volume logique
On cherche lʼidentifiant du système de fichiers :
    
```sh
blkid 
```
    






Copier l’UUID fournit sur la ligne `/dev/mapper/samba-salami` (seulement la partie entre guillemets). Éditer le fichier `/etc/fstab` pour monter le dossier au démarrage de la machine, on y remplace l’UUID par celui obtenu avec `blkid`. Attention, sans les guillemets !
    
```bash 
UUID=7c1ffb53-89ac-432f-9f3c-e35349555820       /home/salami  ext4 defaults,acl,user_xattr,barrier=1,usrquota,grpquota 0 1
```







Vous aurez remarqué des options inhabituelles sur notre ligne de montage :
    
- `acl`, car nous définissons des ACL sur cette partition ;
- `user_xattr`, attributs étendus du système de fichiers, Samba en a besoin ;
- `barrier=1`, pour garantir que les transactions TDB sont protégées contre les coupures de courant imprévues ;
- `usrquota` et `grpquota` sont nécessaires pour utiliser des quotas par utilisateur (dans le cas d’un partage de répertoires personnels, par exemple).



**Note :** Dans la doc de Samba, il est écrit que ext4 intègre par défaut `acl`, `user_xattr` et `barrier=1`, donc il nʼy aurait plus besoin de les déclarer.



Nous enlevons lʼespace réservé. Sans cette commande le système réserve 5 % pour _root_, c’est important sur une partition système pour ne pas bloquer la machine. Dans notre cas, ce nʼest pas important. Et 5 % de gros volume, ça fait beaucoup dʼespace réservé pour rien du tout :
    
```
tune2fs -m 0 /dev/mapper/samba-salami
```
    






Enfin, nous montons les partages :
    
```sh
mount -a
```
    






Vérifiez que la ligne `/dev/mapper/samba-salami` est présente.
Une dernière précaution — de taille !
    
```
df -h 
```







### Définition des droits dʼaccès avec les ACL
Pourquoi de ne pas se contenter de `Valid User` dans Samba ?
    
Si vous avez un domaine, cʼest que vous avez besoin d’une gestion centralisée des utilisateurs. Vous avez donc un certain nombre d’utilisateurs et de groupes et l’on va probablement vous demander, par exemple, que lʼaccès d’un sous‑dossier soit limité au personnel d’un service pour la lecture et autorisé seulement aux cadres pour lʼécriture. Si vous nʼavez que Samba pour gérer ça, vous devrez refaire un partage réseau par demande, vous nʼallez pas vous en sortir et vos utilisateurs non plus.
    
Vous pourrez aussi utiliser NFS pour les postes GNU/Linux qui accèdent à ce serveur. Dans ce cas, il n’y a pas de `force user` dans NFS, donc les ACL seront nécessaires.
    
_Je vous conseille par expérience de mettre uniquement des droits pour des groupes et de ne pas mettre des autorisations par personne. Par exemple, si M. Michu quitte l’entreprise et est remplacé par Mme Tartenpion, il suffira de copier le profil de M. Michu pour que Mme Tartenpion hérite directement des bons droits._
    
_Dans notre cas, nous allons créer un groupe dans le domaine qui se nomme « salami-share » et nous y mettons les personnes qui auront le droit de lire et dʼécrire. Rien ne vous empêche de faire, par exemple, un groupe qui nʼaura quʼun accès en lecture._
    
Le `chmod` suivant permet dʼéviter que le partage hérite de lʼaccès en lecture pour tout le monde.



```
chmod 700 /home/salami
```
    






Définition des ACL sur la racine du partage (dans notre exemple pour `domain admin` et `salami-share`) :
    
```
setfacl -R -m g:"EXAMPLE\domain admins":rwx /home/salami
setfacl -R -m g:"EXAMPLE\salami-share":rwx /home/salami
```
    






Définition des ACL par défaut du partage (les droits que prendront chaque dossier ou fichier créé sur notre partage ; dans notre exemple, pour `domain admin` et `salami‑share`) :
    
```
setfacl -R -m d:g:"EXAMPLE\domain admins":rwx /home/salami
setfacl -R -m d:g:"EXAMPLE\salami-share":rwx /home/salami
```







### Définition du partage dans Samba
Pour éviter que les postes sous Windows voient le dossier `lost+found`, on ancre le partage dans un sous‑dossier. Je sais, cʼest du bricolage ; merci de ne pas taper.
    
```
mkdir /home/salami/salami
```
    






Modification de Samba.
    
À la fin du fichier `/etc/samba/smb.conf`, copiez ce quʼil y a ci‑dessous :
    
```bash
[salami]
path = /home/salami/salami
writable = yes
printable = no
browseable=yes
Valid Users = @"Domain Users"
```
    






On relance Samba pour rendre le partage est disponible sur le réseau :
    
```
/etc/init.d/smbd restart
```
    






Ou avec systemd :
    
```
systemctl smbd restart
```







Conclusion
==========
Voilà, j’espère que cette petite aide va permettre à dʼautres de pouvoir remplacer facilement un serveur de fichiers Windows par un serveur GNU/Linux. Vous pourrez même administrer Samba depuis Windows : _Gestion de lʼordinateur_ → _Se connecter à un autre ordinateur_ (_Outils système_ → _Dossiers partagés_), vous indiquez votre serveur Samba, il vient une erreur, vous cliquez sur OK.
    
Et pour finir, une petite anecdote. Pour les sauvegardes, nous utilisons [Rsnapshot](https://rsnapshot.org/). Donc, sur les machines qui font tourner Rsnapshot, nous montions le serveur Windows en CIFS et nous faisions la sauvegarde comme ça ; il fallait de quarante minutes à une heure pour le faire tourner.
    
Depuis quʼon est entièrement sous GNU/Linux, un cycle ne prend plus que quinze à vingt minutes (évidemment, Rsnapshot se connecte maintenant avec SSH + Rsync).