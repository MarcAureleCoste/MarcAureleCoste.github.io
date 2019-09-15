Faire une installation minimal de Fedora avec X
###############################################

:date: 2019-02-12 20:35
:modified: 2019-09-15 20:35
:tags: Fedora, Installation, Configuration
:category: Linux
:slug: fedora-installation
:authors: Marc-Aurele Coste
:lang: fr

.. _tearing: https://en.wikipedia.org/wiki/Screen_tearing
.. _QTile: http://www.qtile.org/
.. _QTile Config: https://gist.github.com/MarcAureleCoste/02c18d3bad74c85b5bcd3e415f2e01a9



Dans cet article je vais détailler les étapes pour faire une installation minimale qui permet de booter et lancer une session X avec un window manager (ici QTile).
C'est un exercice assez simple, surtout avec Fedora qui dispose d'un installateur graphique facile à prendre en main.

Dans un deuxième temps on passera rapidement sur les quelques fichiers / configurations à faire pour avoir un système fonctionnel.

Média bootable
==============

On télécharge l'ISO que l'on prendra en version *netinstall*. Pour la création d'une clé USB bootable on utilise la commande suivante

.. code-block:: sh

    dd if=path/to/iso of=/path/to/device.

Il existe également des outils dédiés pour faire la même chose avec une interface graphique.

:Windows:
    `rufus <https://rufus.ie/>`_ | `unetbootin <https://unetbootin.github.io/>`_
:Linux:
    `unetbootin <https://unetbootin.github.io/>`_
:MacOS:
    `unetbootin <https://unetbootin.github.io/>`_

Installation
============

Bon ici, rien de compliqué. On boot sur la clé et on suit les étapes.

.. note:: On est sur une version *netinstall* donc il nous faut une connexion internet Wi-Fi ou filaire.

.. image:: {static}/static/images/fedora_installation/choose_language.png
    :alt: choose language screen
    :align: center
    :class: responsive-images

On choisit la langue, par défaut je me mets toujours en anglais mais après c'est une question de préférence. Ne pas oublier de choisir son clavier (en haut à droite).

Ici on arrive sur le menu principal de l'installation.

.. image:: {static}/static/images/fedora_installation/main.png
    :alt: main screen
    :align: center
    :class: responsive-images

On choisit où on veut installer la distribution (Installation Destination)

.. image:: {static}/static/images/fedora_installation/select_drive.png
    :alt: select installation destination screen
    :align: center
    :class: responsive-images

Pour finir on choisit le type d'installation (Sofware Selection). Ici on prend la *minimal install* et c'est tout.

.. image:: {static}/static/images/fedora_installation/software_selection.png
    :alt: select software screen
    :align: center
    :class: responsive-images

On peut maintenant lancer le processus d'installation avec le bouton *begin installation* qui devrait être disponible.

Installation en Wi-Fi
---------------------

Pour ceux qui ont fait une installation en Wi-Fi il vous faut installer les drivers de votre carte wi-fi et aussi ajouter à NetworkManager le support du wi-fi (et du bluetooth).

Pour cela on reboot à nouveau sur sa clé USB et on procède de la manière suivante.

.. code-block:: sh

    # Permet de récupérer le nom du volume groupe (ici fedora)
    vgscan
    # Permet d'activer le volume groupe
    vgchange -ay fedora
    # Permet de vérifier que le volume groupe est bien activé
    lvscan

    # On crée un dossier qui nous servira de point de montage et on monte le volume root (fedora-root)
    mkdir /mnt/fedora
    mount /dev/mapper/fedora-root /mnt/fedora
    cd /mnt/fedora

    # On monte les dossiers importants pour le fonctionnement du chroot
    mount -o bind /dev dev
    mount -o bind /proc proc
    mount -o bind /sys sys
    mount -t tmpfs tmpfs tmp

    # On chroot
    chroot /mnt/fedora

    # On installe les paquets nécessaires pour notre connexion Wi-Fi
    dnf install @"Hardware Support" @"Common NetworkManager Submodules" network-manager-applet

Configuration
=============

Fedora est installée !

Il faut maintenant régler quelques petits détails afin d'avoir une distribution prête à l'emploi. Il nous faut tout d'abord installer X ainsi qu'un window manager. Pour ma part ce sera QTile_.

Installation de X, du WM et quelques autres packets utiles.
-----------------------------------------------------------

On utilise DNF pour installer tout ce dont nous avons besoin.

.. code-block:: sh

    # groupinstall pour base-x
    sudo dnf groupinstall 'base-x'
    # install classique pour le reste
    sudo dnf install qtile rxvt-unicode-256color w3m feh compton

:qtile:
    Notre window manager.
:rxvt-unicode-256color:
    Notre terminal.
:w3m:
    Notre navigateur internet pour le moment. (Facultatif: on peut aussi installer directement Firefox si on veut. Moi, je préfère passer par les archives tar fournies par Mozilla.)
:feh:
    Va nous servir à définir une image en background.
:compton:
    Un compositor léger qui pemettra de supprimer le tearing_

RPMs fusion
-----------

Pour avoir accés à une liste plus complète de packets on installe les repos `RPM fusion <https://rpmfusion.org/>`_ *free* et *non free*

.. code-block:: sh

    sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

Finitions
---------

On commence par recupérer les fichiers `.xinitrc <https://gist.github.com/MarcAureleCoste/62094177c8f1c0239077b8cc541b427a>`_ et `.Xresources <https://gist.github.com/MarcAureleCoste/2472133ba1ddcb3406dc972fe291a5f1>`_ que l'on copie dans son **home**.

:.xinitrc:
    Sert lorsqu'on lance la session X pour paramétrer les programmes qui doivent être exécutés et quel window manager lancer.
:.Xresources:
    Permet de configurer certaines applications et notamment rxvt-unicode-256color que nous avons installé un peu avant.

Pour finir on récupère le `fichier de configuration <https://gist.github.com/MarcAureleCoste/02c18d3bad74c85b5bcd3e415f2e01a9>`_ de QTile et on le place dans le dossier suivant *~/.config/qtile/*.

On peut maintenant démarrer notre sessions X avec la commande

.. code-block:: sh

    startx

.. note:: Si vous n'avez pas encore le fichier **.xinitrc** dans votre *home* il faut passer en argument de *startx* le path absolue vers l'éxécutable de votre window manager (pour qtile */usr/bin/qtile*).

Voilà, l'installation de Fedora est finie et nous avons une installation minimale parfaitement fonctionnelle.
