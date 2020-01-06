---
title: 'Construire des paquets DEB pour Debian avec Pbuilder.'
visible: false
---

Aujourd'hui je vais tenter de faire un tuto simple et minimaliste sur Pbuilder et comment le régler. Cela fera office de suites aux tutos pour faire des paquets Debian 1.[Construire des paquets DEB pour Debian (première partie)](https://passiongnulinux.tuxfamily.org/post/2018-04-20-construire-des-paquets-deb-pour-debian/) et 2.[Construire des paquets DEB pour Debian (deuxième partie)](https://passiongnulinux.tuxfamily.org/post/2018-04-21-construire-des-paquets-deb-pour-debian-suite/).

[Pbuilder](https://packages.debian.org/sid/pbuilder) est un système de construction pour faire des paquets Debian dans un chroot minimaliste, il créé un système Debian minimal, il télécharge et installe les dépendances de construction et ensuite compile le paquet dans ce chroot. Il permet aussi entre autre de fournir un moyen simple de mettre en place des systèmes de test comme dans "un bac à sable".

> pbuilder construit un système chroot et construit un paquet dans le chroot. C'est un système idéal à utiliser pour vérifier qu'un paquet a des dépendances de construction correctes. Il utilise abondamment apt, et un miroir local ou une connexion rapide à un miroir Debian est idéal, mais pas nécessaire.

> "pbuilder create" utilise debootstrap pour créer une image chroot.

> "pbuilder update" met à jour l'image à l'état actuel de testing / unstable / what

> "pbuilder build" prend un fichier * .dsc et construit un binaire dans l'image chroot.

> pdebuild est un wrapper pour les développeurs Debian, permettant d'autoriser l'exécution de pbuilder comme "debuild", en tant qu'utilisateur normal.

Principalement c'est l’outil que j'utilise aussi bien sur ma machine pour ma propre utilisation que pour envoyer mon travail sur les serveurs de Debian pour des paquets officiels. On passera en revue l'installation, la configuration (de manière simple et minimale) et son utilisation.

On pourrait le comparer un peu dans le principe à [OBS et OSC d'openSUSE](https://passiongnulinux.tuxfamily.org/post/2018-01-01-comment-participer-a-opensuse-obs/).

On va commencer par se faciliter la vie, ne pas lancer Pbuilder en root mais en tant que simple utilisateur avec *sudo*. Pour cela rien de bien compliqué, on va completer notre fichier /etc/sudoers, en rajoutant quelques lignes:

    Cmnd_Alias PBUILDER = /usr/sbin/pbuilder, /usr/bin/pdebuild, /usr/bin/debuild-pbuilder, /usr/sbin/cowbuilder
    Defaults!PBUILDER env_keep+="DIST ARCH HOME"
    
    <your user>  ALL=(ALL) SETENV: PBUILDER

On pourra faire ceci avec `visudo` ou bien la nouvelle façon de faire avec des fichiers dans le dossier /etc/sudoers.d (voir la [documentation Ubuntu](https://doc.ubuntu-fr.org/sudoers)). Pour le moment, je reste encore à modifier le fichier, on verra pour la suite si je m'adapte à cette façon de faire qui peut apporter pas mal d'améliorations comme de simples liens vers des fichiers venant de dossiers personnels de chacun.

> La gestion de sudo est améliorée dans les dernières versions (Debian version 1.7.2).
> 
> Pour modifier le fonctionnement de la commande sudo, l'administrateur du système ne modifie plus le fichier /etc/sudoers mais positionne des fichiers de personnalisation dans le répertoire /etc/sudoers.d.
> 
> Premier avantage, vous pouvez définir autant de fichiers que de modifications (voir le §2). Le nom est libre et peut donc faire référence à l'élément personnalisé. Exemples : 10-sysctl 20-userX 30-apt.
> 
> Le deuxième avantage est que la création d'un de ces fichiers ne nécessite aucun droit d'administration. L'intervention de l'administrateur se limite à la copie du(des) fichier(s) sous /etc/sudoers.d et application du mask 0440.
> 
> Le troisième avantage. Vous disposez d'un aperçu des modifications en listant simplement le contenu du répertoire /etc/sudoers.d.
> 
> Le quatrième avantage est que l'administrateur peut annuler à tout moment une autorisation particulière de la commande sudo en supprimant le fichier de personnalisation correspondant.
> 
> Le dernier avantage est qu'en supprimant tous les fichiers de personnalisation, vous êtes certain de revenir à la configuration par défaut de sudo.
> 
> L'ancienne méthode reste utilisable.
> 
> La configuration de sudo est enregistrée dans le fichier de configuration /etc/sudoers.
> 
> La modification de ce fichier s'effectue à travers un utilitaire de vérification appelé visudo.

Ceci étant fait, il faut se faire un fichier de configuration pour notre Pbuilder, le fameux `~/.pbuilderrc`.

On pourrait adapter notre fichier à notre utilisation avec l'excellente [documentation de Debian](https://wiki.debian.org/PbuilderTricks). Pour mon besoin plus que minimal, j'utilise un fichier des plus simplistes et limités en fonction:

    # this is your configuration file for pbuilder.
    # the file in /usr/share/pbuilder/pbuilderrc is the default template.
    # /etc/pbuilderrc is the one meant for overwriting defaults in
    # the default template
    #
    # read pbuilderrc.5 document for notes on specific options.
    MIRRORSITE=http://ftp.fr.debian.org/debian/
    COMPONENTS="main contrib non-free"
    OTHERMIRROR="deb http://ftp.fr.debian.org/debian buster-backports main contrib non-free"
    AUTO_DEBSIGN=yes

On verra dans un future proche si je modifie ce fichier un peu plus, dans ce cas je le partagerai.
Dans le cas du fichier actuel, c'est des plus compréhensibles, mais on va quand même expliquer. La partie *MIRRORSITE* est comme son nom l'indique, quel miroir utiliser, dans mon cas c'est celui de la France. *COMPONENTS* indique quelles sections des dépôts Debian  seront utilisées, les populaires *main, contrib et non-free*. *OTHERMIRROR* est l'endroit où on peut indiquer d'autres dépôts comme ceux de Deb-Multimedia, dans mon cas juste les *backports* de Stable actuelle. Pour finir *AUTO_DEBSIGN* que je positionne sur *yes*, pour signer automatiquement mes paquets avec ma clef personnel (c'est aussi un truc à faire mais c'est optionnel).

On va installer ensuite les paquets Debian pour la fabrication des paquets, en l'occurence `devscripts` et `pbuilder`:

    # apt install devscripts pbuilder

Ensuite on va créer son système, que ce soit une stable, une testing ou une SID; pour le moment et pour satisfaire mes besoin pour ce tuto, on se contentera de stable:

    ~$ sudo pbuilder create --distribution stable
     
Dans mon exemple plus bas, qui sera tronqué, on voit la création du chroot, le choix de la distribution stable, la date, le téléchargement et l’installation de notre distribution minimal:

    sebastien@debiacerlinux:~$ sudo pbuilder create --distribution stable

    Nous espérons que vous avez reçu de votre administrateur système local les consignes traditionnelles. Généralement, elles se concentrent sur ces trois éléments :

    #1) Respectez la vie privée des autres.
    #2) Réfléchissez avant d'utiliser le clavier.
    #3) De grands pouvoirs confèrent de grandes responsabilités.

    [sudo] Mot de passe de sebastien : 
    I: Distribution is stable.
    I: Current time: Tue Nov 19 22:39:53 CET 2019
    I: pbuilder-time-stamp: 1574199593
    I: Building the build environment
    I: running debootstrap
    /usr/sbin/debootstrap
    I: Target architecture can be executed
    I: Retrieving InRelease 
    I: Checking Release signature
    I: Valid Release signature (key id 6D33866EDD8FFA41C0143AEDDCC9EFBF77E11517)
    I: Retrieving Packages 
    I: Validating Packages 
    I: Resolving dependencies of required packages...
    I: Resolving dependencies of base packages...
    I: Checking component main on http://ftp.fr.debian.org/debian...
    [...]

Notez qu'on aurait très bien pu utiliser le nom de code de la distribution au lieu de son numéro (ici buster au lieu de stable):

    ~$ sudo pbuilder create --distribution buster

Une fois fait, il suffit d'avoir les sources d'un paquet Debian pour commencer à bidouiller. Il y a plusieurs façons de faire, dans mon cas je suis plus friand d'utiliser `pdebuild` que `pbuilder`, mais en gros `pbuilder` s'utilise avec un fichier *.DSC*, tandis que `pdebuild` s’utilise directement depuis un dossier */debian*.

Pour mon exemple, je reprends mon paquet *Ghostwriter* et les sources de SID que je télécharge dans un dossier */home/sebastien/Paquets/DEB/nom-du-paquet*, pour plusieurs raisons, vu que c'est un paquet dont je m'occupe, je sais comment il est fait, les dépendances sont présentes dans Stable, la version de Stable est la 1.7.4 et donc il est faisable et intéréssant de le prendre en exemple pour montrer un rétroportage depuis des sources SID pour une Stable.

    ~$ mkdir ~/Paquets/DEB/Ghostwriter
    ~$ apt-get source ghostwriter
    Lecture des listes de paquets... Fait
    Note : la maintenance du paquet de « ghostwriter » est réalisée dans le système de suivi de versions « Git » à l'adresse :
    https://salsa.debian.org/seb95-guest/ghostwriter.git
    Veuillez utiliser la commande :
    git clone https://salsa.debian.org/seb95-guest/ghostwriter.git
    pour récupérer les dernières mises à jour (éventuellement non encore publiées) du paquet.
    Nécessité de prendre 1 206 ko dans les sources.
    Réception de :1 http://ftp.fr.debian.org/debian sid/main ghostwriter 1.8.0-2 (dsc) [2 047 B]
    Réception de :2 http://ftp.fr.debian.org/debian sid/main ghostwriter 1.8.0-2 (tar) [1 196 kB]
    Réception de :3 http://ftp.fr.debian.org/debian sid/main ghostwriter 1.8.0-2 (diff) [8 324 B]
    1 206 ko réceptionnés en 1s (1 017 ko/s)   
    dpkg-source: info: extraction de ghostwriter dans ghostwriter-1.8.0
    dpkg-source: info: extraction de ghostwriter_1.8.0.orig.tar.gz
    dpkg-source: info: extraction de ghostwriter_1.8.0-2.debian.tar.xz
    dpkg-source: info: using patch list from debian/patches/series
    dpkg-source: info: mise en place de fix-ime.patch
    dpkg-source: info: mise en place de appstream-metadata-location.patch


    $ ls
    ghostwriter-1.8.0                  ghostwriter_1.8.0-2.dsc
    ghostwriter_1.8.0-2.debian.tar.xz  ghostwriter_1.8.0.orig.tar.gz

Donc soit on utilise `pbuilder` ou `pdebuild`, on verra les deux pour notre exemple:

### PBUILDER:

    $ sudo pbuilder build ghostwriter_1.8.0-2.dsc
    [sudo] Mot de passe de sebastien : 
    I: pbuilder: network access will be disabled during build
    I: Current time: Tue Nov 19 23:34:19 CET 2019
    I: pbuilder-time-stamp: 1574202859
    I: Building the build Environment
    I: extracting base tarball [/var/cache/pbuilder/base.tgz]
    I: copying local configuration
    I: mounting /proc filesystem
    I: mounting /sys filesystem
    I: creating /{dev,run}/shm
    I: mounting /dev/pts filesystem
    I: redirecting /dev/ptmx to /dev/pts/ptmx
    I: policy-rc.d already exists
    I: Obtaining the cached apt archive contents
    I: Copying source file
    I: copying [ghostwriter_1.8.0-2.dsc]
    I: copying [./ghostwriter_1.8.0.orig.tar.gz]
    I: copying [./ghostwriter_1.8.0-2.debian.tar.xz]
    I: Extracting source
    gpgv: unknown type of key resource 'trustedkeys.kbx'
    gpgv: keyblock resource '/home/sebastien/.gnupg/trustedkeys.kbx': General error
    gpgv: Signature made Sat Jul 27 08:40:45 2019 UTC
    gpgv:                using RSA key 65A12DF4FE31AD6BAC4D76AE3355F4D63B5821CC
    gpgv:                issuer "bartm@knars.be"
    gpgv: Can't check signature: No public key
    dpkg-source: warning: failed to verify signature on ./ghostwriter_1.8.0-2.dsc
    dpkg-source: info: extracting ghostwriter in ghostwriter-1.8.0
    dpkg-source: info: unpacking ghostwriter_1.8.0.orig.tar.gz
    dpkg-source: info: unpacking ghostwriter_1.8.0-2.debian.tar.xz
    dpkg-source: info: using patch list from debian/patches/series
    dpkg-source: info: applying fix-ime.patch
    dpkg-source: info: applying appstream-metadata-location.patch
    I: using fakeroot in build.
    I: Installing the build-deps
    [...]
    
    [...]
    dpkg-deb: building package 'ghostwriter' in '../ghostwriter_1.8.0-2_amd64.deb'.
    dpkg-deb: building package 'ghostwriter-dbgsym' in '../ghostwriter-dbgsym_1.8.0-2_amd64.deb'.
    dpkg-genbuildinfo
    dpkg-genchanges  >../ghostwriter_1.8.0-2_amd64.changes
    dpkg-genchanges: info: not including original source code in upload
    dpkg-source --after-build .
    dpkg-source: info: using options from ghostwriter-1.8.0/debian/source/options: --extend-diff-ignore=^\.gitlab-ci.yml$
    dpkg-buildpackage: info: binary and diff upload (original source NOT included)
    I: copying local configuration
    I: Copying back the cached apt archive contents
    I: unmounting dev/ptmx filesystem
    I: unmounting dev/pts filesystem
    I: unmounting dev/shm filesystem
    I: unmounting proc filesystem
    I: unmounting sys filesystem
    I: cleaning the build env 
    I: removing directory /var/cache/pbuilder/build/16027 and its subdirectories
    I: Current time: Tue Nov 19 23:39:48 CET 2019
    I: pbuilder-time-stamp: 1574203188

Les paquets sont faits et sont disponibles dans */var/cache/pbuilder/result/*:

    $ ls /var/cache/pbuilder/result/
    ghostwriter_1.8.0-2_amd64.buildinfo  ghostwriter_1.8.0-2.debian.tar.xz
    ghostwriter_1.8.0-2_amd64.changes    ghostwriter_1.8.0-2.dsc
    ghostwriter_1.8.0-2_amd64.deb        ghostwriter-dbgsym_1.8.0-2_amd64.deb

### PDEBUILD:

La même chose que plus haut mais cette fois avec `pdebuild`, je vais changer le debian/changelog et donner une version mineur avec l'outil `dch`, une petite modification pour qu'on différencie les paquets fait par les deux méthodes, je rajoute à ceux que je vais recompiler via `pdebuild` un *pdb* dans sa version:

    $ cd ghostwriter-1.8.0/
    $ dch -r
    
    ghostwriter (1.8.0-2+pdb1) stable; urgency=medium

      * Upload to stable. 
      * rebuild with pdebuild.     

    -- Sebastien CHAVAUX <mailXXXXXXXX>  Wed, 20 Nov 2019 00:18:21 +0100

Maintenant je lance la compilation:

    $ pdebuild
    dpkg-checkbuilddeps: error: Unmet build dependencies: libqt5svg5-dev qtmultimedia5-dev qtwebengine5-dev libhunspell-dev pkg-config qttools5-dev-tools
    W: Unmet build-dependency in source
    dpkg-source: info: using options from ghostwriter-1.8.0/debian/source/options: --extend-diff-ignore=^\.gitlab-ci.yml$
    dh clean
    debian/rules override_dh_auto_clean
    make[1]: Entering directory '/home/sebastien/Paquets/DEB/ghostwriter-test/ghostwriter-1.8.0'
    rm -f ./Makefile ./.qmake.stash
    rm -f ./debian/substvars
    rm -rf ./build/
    make[1]: Leaving directory '/home/sebastien/Paquets/DEB/ghostwriter-test/ghostwriter-1.8.0'
    dh_clean
    dpkg-source: info: using options from ghostwriter-1.8.0/debian/source/options: --extend-diff-ignore=^\.gitlab-ci.yml$
    dpkg-source: info: using source format '3.0 (quilt)'
    dpkg-source: info: building ghostwriter using existing ./ghostwriter_1.8.0.orig.tar.gz
    dpkg-source: info: using patch list from debian/patches/series
    dpkg-source: info: building ghostwriter in ghostwriter_1.8.0-2+pdb1.debian.tar.xz
    dpkg-source: info: building ghostwriter in ghostwriter_1.8.0-2+pdb1.dsc
    I: Generating source changes file for original dsc
    dpkg-genchanges: info: not including original source code in upload
    dpkg-source: info: using options from ghostwriter-1.8.0/debian/source/options: --extend-diff-ignore=^\.gitlab-ci.yml$
    I: pbuilder: network access will be disabled during build
    I: Current time: Wed Nov 20 00:28:44 CET 2019
    I: pbuilder-time-stamp: 1574206124
    I: Building the build Environment
    I: extracting base tarball [/var/cache/pbuilder/base.tgz]
    I: copying local configuration
    I: mounting /proc filesystem
    I: mounting /sys filesystem
    I: creating /{dev,run}/shm
    I: mounting /dev/pts filesystem
    I: redirecting /dev/ptmx to /dev/pts/ptmx
    I: policy-rc.d already exists
    I: Obtaining the cached apt archive contents
    I: Copying source file
    I: copying [../ghostwriter_1.8.0-2+pdb1.dsc]
    I: copying [../ghostwriter_1.8.0.orig.tar.gz]
    I: copying [../ghostwriter_1.8.0-2+pdb1.debian.tar.xz]
    I: Extracting source
    [...]

    [...]
    dpkg-deb: building package 'ghostwriter' in '../ghostwriter_1.8.0-2+pdb1_amd64.deb'.
    dpkg-deb: building package 'ghostwriter-dbgsym' in '../ghostwriter-dbgsym_1.8.0-2+pdb1_amd64.deb'.
    dpkg-genbuildinfo
    dpkg-genchanges  >../ghostwriter_1.8.0-2+pdb1_amd64.changes
    dpkg-genchanges: info: not including original source code in upload
    dpkg-source --after-build .
    dpkg-source: info: using options from ghostwriter-1.8.0/debian/source/options: --extend-diff-ignore=^\.gitlab-ci.yml$
    dpkg-buildpackage: info: binary and diff upload (original source NOT included)
    I: copying local configuration
    I: Copying back the cached apt archive contents
    I: unmounting dev/ptmx filesystem
    I: unmounting dev/pts filesystem
    I: unmounting dev/shm filesystem
    I: unmounting proc filesystem
    I: unmounting sys filesystem
    I: cleaning the build env 
    I: removing directory /var/cache/pbuilder/build/32118 and its subdirectories
    I: Current time: Wed Nov 20 00:24:24 CET 2019
    I: pbuilder-time-stamp: 1574205864
    signfile dsc /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1.dsc BE45AEFE4EB7156EED8CE1BF0C5C55889E178585
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'

    fixup_buildinfo /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1.dsc /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1_amd64.buildinfo
    signfile buildinfo /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1_amd64.buildinfo BE45AEFE4EB7156EED8CE1BF0C5C55889E178585
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'

    fixup_changes dsc /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1.dsc /var/cache/pbuilder/result//ghostwriter_1.8.0-2+pdb1_amd64.changes
    fixup_changes buildinfo /var/cache/pbuilder/result/ghostwriter_1.8.0-2+pdb1_amd64.buildinfo /var/cache/pbuilder/result//ghostwriter_1.8.0-2+pdb1_amd64.changes
    signfile changes /var/cache/pbuilder/result//ghostwriter_1.8.0-2+pdb1_amd64.changes BE45AEFE4EB7156EED8CE1BF0C5C55889E178585
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'
    gpg: WARNING: unsafe permissions on homedir '/home/sebastien/.gnupg'

    Successfully signed dsc, buildinfo, changes files

Les paquets sont faits et sont disponibles dans */var/cache/pbuilder/result/* avec leurs versions *1.8.0-2+pdb1* en plus de ceux précédemment faits:

    $ ls /var/cache/pbuilder/result/
    ghostwriter_1.8.0-2_amd64.buildinfo       ghostwriter_1.8.0-2+pdb1_amd64.changes
    ghostwriter_1.8.0-2_amd64.changes         ghostwriter_1.8.0-2+pdb1_amd64.deb
    ghostwriter_1.8.0-2_amd64.deb             ghostwriter_1.8.0-2+pdb1.debian.tar.xz
    ghostwriter_1.8.0-2.debian.tar.xz         ghostwriter_1.8.0-2+pdb1.dsc
    ghostwriter_1.8.0-2.dsc                   ghostwriter-dbgsym_1.8.0-2_amd64.deb
    ghostwriter_1.8.0-2+pdb1_amd64.buildinfo  ghostwriter-dbgsym_1.8.0-2+pdb1_amd64.deb    

Si vous avez des compléments d'informations, de configurations ou toutes autres suggestions, laissez-les moi en commentaire dans [le forum](https://passiongnulinux.tuxfamily.org/forum/) et je les intégrerai dans une édition.
