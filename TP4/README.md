# TP4 : Real services

Dans ce TP4, on va s'approcher de plus en plus vers de la gestion de serveur, commen on le fait dans le monde réel.

Le but de ce TP :

➜ **monter un serveur de stockage** VM `storage.tp4.linux`

- le serveur de stockage possède une partition dédiée
- sur cette partition, plusieurs dossiers sont créés
- chaque dossier contient un site web
- ces dossiers sont partagés à travers le réseau pour rendre leur contenu disponible à notre serveur web

➜ **monter un serveur web** VM `web.tp4.linux`

- il accueillera deux sites web
- il ne sera (malheureusement) pas publié sur internet : c'est juste une VM
- les sites web sont stockés sur le serveur de stockage, le serveur web y accède à travers le réseau

---

➜ Plutôt que de monter des petits services de test, ou analyser les services déjà existants sur la machine, on va donc passer à l'étape supérieure et **monter des trucs vraiment utilisés dans le monde réel** 

Rien de sorcier cela dit, et **à la fin vous aurez appris à monter un petit serveur Web.** Ce serait exactement la même chose si vous voulez publier un site web, et que vous voulez gérer vous-mêmes le serveur Web.

**Le serveur de stockage** c'est pour rendre le truc un peu plus fun (oui j'ai osé dire *fun*), et voir un service de plus, qui est utilisé dans le monde réel. En plus, il est parfaitement adapté pour pratiquer et s'exercer sur le partitionnement de façon pertinente.

➜ On aura besoin de deux VMs dans ce TP : 🖥️ **VM `web.tp4.linux`** et 🖥️ **VM `storage.tp4.linux`**.

> Pour une meilleure lisibilité, j'ai éclaté le TP en 3 parties.

# Partie 1 : Partitionnement du serveur de stockage

> Cette partie est à réaliser sur 🖥️ **VM storage.tp4.linux**.

On va ajouter un disque dur à la VM, puis le partitionner, afin de créer un espace dédié qui accueillera nos sites web.

➜ **Ajouter deux disques durs de 2G à la VM**

- cela se fait via l'interface graphique de virtualbox
- il faut éteindre la VM pour ce faire

> [**Référez-vous au mémo LVM pour réaliser le reste de cette partie.**](../../../cours/memo/lvm.md)

**Le partitionnement est obligatoire pour que le disque soit utilisable.** Ici on va rester simple : une seule partition, qui prend toute la place offerte par le disque.

Comme vu en cours, le partitionnement dans les systèmes GNU/Linux s'effectue généralement à l'aide de *LVM*.

**Allons !**


🌞 **Partitionner le disque à l'aide de LVM**

- créer 2 *physical volumes (PV)* :
  - un PV ajouté pour chacun des disques
  - chaque PV fait donc 2G

  ```golang
  [vince@vm2 ~]$ sudo pvdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  "/dev/sdb" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               im3asG-6Qg9-O1Nf-eIot-2rTk-NMr1-RXtGM9

  "/dev/sdc" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  VG Name
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size
  ```
- créer un nouveau *volume group (VG)*
  - il devra s'appeler `storage`
  - il doit contenir les deux PVs créés à l'étape précédente

  ```golang
  [vince@vm2 ~]$ sudo vgdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  --- Volume group ---
  VG Name               storage
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               3.99 GiB
  PE Size               4.00 MiB
  Total PE              1022
  Alloc PE / Size       0 / 0
  Free  PE / Size       1022 / 3.99 GiB
  VG UUID               FvMj9x-CQfT-NRo0-u8OG-SSJz-CLTx-mJl45s
  ```

- créer un nouveau *logical volume (LV)* : ce sera la partition utilisable
  - elle doit être dans le VG `storage`
  - elle doit occuper tout l'espace libre

```golang
[vince@vm2 ~]$ sudo lvdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  --- Logical volume ---
  LV Path                /dev/storage/storage
  LV Name                storage
  VG Name                storage
  LV UUID                hH7KsP-BmUB-whcs-80HS-cG3D-dowO-CsUTm9
  LV Write Access        read/write
  LV Creation host, time vm2.tp4.b1, 2024-02-20 09:28:00 +0100
  LV Status              available
  # open                 0
  LV Size                3.99 GiB
  Current LE             1022
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
  ```


🌞 **Formater la partition**

- vous formaterez la partition en ext4 (avec une commande `mkfs`)
  - le chemin de la partition, vous pouvez le visualiser avec la commande `lvdisplay`
```golang
  [vince@vm2 ~]$ mkfs -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
mkfs.ext4: Permission denied while trying to determine filesystem size
[vince@vm2 ~]$ sudo !!
sudo mkfs -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1046528 4k blocks and 261632 inodes
Filesystem UUID: 85610353-103f-4031-8320-078a49aff167
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
``` 

  - pour rappel un *Logical Volume (LVM)* **C'EST** une partition

🌞 **Monter la partition**

- montage de la partition (avec la commande `mount`)
  - la partition doit être montée dans le dossier `/storage`

  - preuve avec une commande `df -h` que la partition est bien montée
```golang
  [vince@vm2 ~]$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
devtmpfs                     4.0M     0  4.0M   0% /dev
tmpfs                        386M     0  386M   0% /dev/shm
tmpfs                        155M  3.7M  151M   3% /run
/dev/mapper/rl-root          6.2G  1.2G  5.0G  20% /
/dev/sda1                   1014M  220M  795M  22% /boot
tmpfs                         78M     0   78M   0% /run/user/1000
/dev/mapper/storage-storage  3.9G   24K  3.7G   1% /mnt/storage
```

  - utilisez un `| grep` pour isoler les lignes intéressantes

```golang
[vince@vm2 ~]$ df -h | grep storage
/dev/mapper/storage-storage  3.9G   24K  3.7G   1% /mnt/storage
```
  - prouvez que vous pouvez lire et écrire des données sur cette partition

  ```golang
[vince@vm2 ~]$ cat /mnt/storage/fichier_vince
coucou
```
- définir un montage automatique de la partition (fichier `/etc/fstab`)
  - vous vérifierez que votre fichier `/etc/fstab` fonctionne correctement

```golang
  [vince@vm2 ~]$ sudo umount /mnt/storage
[vince@vm2 ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/storage does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
/mnt/storage             : successfully mounted
```

Ok ! Za, z'est fait. On a un espace de stockage dédié pour stocker nos sites web.

⭐**BONUS**

- utilisez une commande `dd` pour remplir complètement la nouvelle partition
- prouvez que la partition est remplie avec une commande `df`
- ajoutez un nouveau disque dur de 2G à la machine
- ajoutez ce new disque dur à la conf LVM
- agrandissez la partition pleine à l'aide du nouveau disque
- prouvez aavec un `df` que la partition a bien été agrandie

**Passons à [la partie 2 : installation du serveur de partage de fichiers](./../part2/README.md).**


# Partie 2 : Serveur de partage de fichiers

**Dans cette partie, le but sera de monter un serveur de stockage.** Un serveur de stockage, ici, désigne simplement un serveur qui partagera un dossier ou plusieurs aux autres machines de son réseau.

Ce dossier sera hébergé sur la partition dédiée sur la machine **`storage.tp4.linux`**.

Afin de partager le dossier, **nous allons mettre en place un serveur NFS** (pour Network File System), qui est prévu à cet effet. Comme d'habitude : c'est un programme qui écoute sur un port, et les clients qui s'y connectent avec un programme client adapté peuvent accéder à un ou plusieurs dossiers partagés.

Le **serveur NFS** sera **`storage.tp4.linux`** et le **client NFS** sera **`web.tp4.linux`**.

L'objectif :

- avoir deux dossiers sur **`storage.tp4.linux`** partagés
  - `/storage/site_web_1/`
  - `/storage/site_web_2/`
- la machine **`web.tp4.linux`** monte ces deux dossiers à travers le réseau
  - le dossier `/storage/site_web_1/` est monté dans `/var/www/site_web_1/`
  - le dossier `/storage/site_web_2/` est monté dans `/var/www/site_web_2/`

🌞 **Donnez les commandes réalisées sur le serveur NFS `storage.tp4.linux`**

```golang
[vince@vm2 ~]$ sudo dnf install nfs-utils
[vince@vm2 ~]$ sudo mkdir /storage/site_web_1 -p
[vince@vm2 ~]$ sudo mkdir /storage/site_web2 -p
[vince@vm2 ~]$ ls -dl /storage/site_web_1
drwxr-xr-x. 2 root root 6 Feb 20 10:44 /storage/site_web_1
[vince@vm2 ~]$ sudo chown nobody /storage/site_web_1
[vince@vm2 ~]$ ls -dl /storage/site_web_1
drwxr-xr-x. 2 nobody root 6 Feb 20 10:44 /storage/site_web_1
[vince@vm2 ~]$ ls -dl /storage/site_web_2
drwxr-xr-x. 2 root root 6 Feb 20 10:49 /storage/site_web_2
[vince@vm2 ~]$ sudo chown nobody /storage/site_web_2
[vince@vm2 ~]$ sudo dnf install nano
Last metadata expiration check: 1:19:16 ago on Tue 20 Feb 2024 09:31:50 AM CET.
Package nano-5.6.1-5.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[vince@vm2 ~]$ sudo nano /etc/exports
[vince@vm2 ~]$ sudo systemctl enable nfs-server
[sudo] password for vince:
[vince@vm2 ~]$ sudo systemctl status nfs-server
- contenu du fichier `/etc/exports` dans le compte-rendu notamment
[vince@vm2 ~]$ sudo firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs rpc-bind ssh
```
```golang
  [vince@vm2 ~]$ cat /etc/exports
/storage/site_web_1 10.4.1.3(rw,sync,no_subtree_check)
/storage/site_web_2 10.4.1.3(rw,sync,no_subtree_check)
```


🌞 **Donnez les commandes réalisées sur le client NFS `web.tp4.linux`**

```golang
[vince@web ~]$ sudo dnf install nfs-utils
[sudo] password for vince:
[vince@web ~]$ sudo mkdir -p /var/www/site_web_1
[sudo] password for vince:
[vince@web ~]$ sudo mkdir -p /var/www/site_web_2
[vince@web ~]$ sudo mount 10.4.1.4:/storage/site_web_1 /var/www/site_we
b_1
[vince@web ~]$ sudo mount 10.4.1.4:/storage/site_web_2 /var/www/site_we
b_2
[vince@web ~]$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
devtmpfs                      4.0M     0  4.0M   0% /dev
tmpfs                         386M     0  386M   0% /dev/shm
tmpfs                         155M  3.7M  151M   3% /run
/dev/mapper/rl-root           6.2G  1.2G  5.0G  20% /
/dev/sda1                    1014M  220M  795M  22% /boot
tmpfs                          78M     0   78M   0% /run/user/1000
10.4.1.4:/storage/site_web_1  6.2G  1.2G  5.0G  20% /var/www/site_web_1
10.4.1.4:/storage/site_web_2  6.2G  1.2G  5.0G  20% /var/www/site_web_2
[vince@web ~]$ sudo touch /var/www/site_web_1/fichier_test
[vince@web ~]$ ls -l /var/www/site_web_1/fichier_test
-rw-r--r--. 1 nobody nobody 0 Feb 20  2024 /var/www/site_web_1/fichier_test
```


- contenu du fichier `/etc/fstab` dans le compte-rendu notamment
```golang
[vince@web ~]$ cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Oct 23 09:14:19 2023
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=98eba154-d188-4a39-a469-80eecdcf345d /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0

10.4.1.4 : /storage/site_web_1 /var/www/site_web_1 nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```



> Je vous laisse vous inspirer de docs sur internet **[comme celle-ci](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-rocky-linux-9)** pour mettre en place un serveur NFS.

**Ok, on a fini avec la partie 2, let's head to [the part 3](./../part3/README.md).**




# Partie 3 : Serveur web

- [Partie 3 : Serveur web](#partie-3--serveur-web)
  - [1. Intro NGINX](#1-intro-nginx)
  - [2. Install](#2-install)
  - [3. Analyse](#3-analyse)
  - [4. Visite du service web](#4-visite-du-service-web)
  - [5. Modif de la conf du serveur web](#5-modif-de-la-conf-du-serveur-web)
  - [6. Deux sites web sur un seul serveur](#6-deux-sites-web-sur-un-seul-serveur)

## 1. Intro NGINX


**NGINX (prononcé "engine-X") est un serveur web.** C'est un outil de référence aujourd'hui, il est réputé pour ses performances et sa robustesse.

Un serveur web, c'est un programme qui écoute sur un port et qui attend des requêtes HTTP. Quand il reçoit une requête de la part d'un client, il renvoie une réponse HTTP qui contient le plus souvent de l'HTML, du CSS et du JS.

> Une requête HTTP c'est par exemple `GET /index.html` qui veut dire "donne moi le fichier `index.html` qui est stocké sur le serveur". Le serveur renverra alors le contenu de ce fichier `index.html`.

Ici on va pas DU TOUT s'attarder sur la partie dév web étou, une simple page HTML fera l'affaire.

Une fois le serveur web NGINX installé (grâce à un paquet), sont créés sur la machine :

- **un service** (un fichier `.service`)
  - on pourra interagir avec le service à l'aide de `systemctl`
- **des fichiers de conf**
  - comme d'hab c'est dans `/etc/` la conf
  - comme d'hab c'est bien rangé, donc la conf de NGINX c'est dans `/etc/nginx/`
  - question de simplicité en terme de nommage, le fichier de conf principal c'est `/etc/nginx/nginx.conf`
- **une racine web**
  - c'est un dossier dans lequel un site est stocké
  - c'est à dire là où se trouvent tous les fichiers PHP, HTML, CSS, JS, etc du site
  - ce dossier et tout son contenu doivent appartenir à l'utilisateur qui lance le service
- **des logs**
  - tant que le service a pas trop tourné c'est empty
  - les fichiers de logs sont dans `/var/log/`
  - comme d'hab c'est bien rangé donc c'est dans `/var/log/nginx/`
  - on peut aussi consulter certains logs avec `sudo journalctl -xe -u nginx`

> Chaque log est à sa place, on ne trouve pas la même chose dans chaque fichier ou la commande `journalctl`. La commande `journalctl` vous permettra de repérer les erreurs que vous glisser dans les fichiers de conf et qui empêche le démarrage correct de NGINX.

## 2. Install

🖥️ **VM web.tp4.linux**

🌞 **Installez NGINX**

- installez juste NGINX (avec un `dnf install`) et passez à la suite
- référez-vous à des docs en ligne si besoin

```golang
[vince@web ~]$ sudo dnf install nginx
```

## 3. Analyse

Avant de config des truks 2 ouf étou, on va lancer à l'aveugle et inspecter ce qu'il se passe, inspecter avec les outils qu'on connaît ce que fait NGINX à notre OS.

Commencez donc par démarrer le service NGINX :

```bash
$ sudo systemctl start nginx
$ sudo systemctl status nginx
```

🌞 **Analysez le service NGINX**

- avec une commande `ps`, déterminer sous quel utilisateur tourne le processus du service NGINX
  - utilisez un `| grep` pour isoler les lignes intéressantes
```golang
[vince@web ~]$ ps aux | grep nginx
root       12488  0.0  0.1  10108   956 ?        Ss   10:22   0:00 nginx: master process /usr/sbin/nginx
nginx      12489  0.0  0.6  13908  4960 ?        S    10:22   0:00 nginx: worker process
vince      12535  0.0  0.2   6408  2276 pts/0    S+   10:29   0:00 grep --color=auto nginx
```
- avec une commande `ss`, déterminer derrière quel port écoute actuellement le serveur web
  - utilisez un `| grep` pour isoler les lignes intéressantes
- en regardant la conf, déterminer dans quel dossier se trouve la racine web
  - utilisez un `| grep` pour isoler les lignes intéressantes
- inspectez les fichiers de la racine web, et vérifier qu'ils sont bien accessibles en lecture par l'utilisateur qui lance le processus
  - ça va se faire avec un `ls` et les options appropriées

## 4. Visite du service web

**Et ça serait bien d'accéder au service non ?** Genre c'est un serveur web. On veut voir un site web !

🌞 **Configurez le firewall pour autoriser le trafic vers le service NGINX**

- vous avez reperé avec `ss` dans la partie d'avant le port à ouvrir

🌞 **Accéder au site web**

- avec votre navigateur sur VOTRE PC
  - ouvrez le navigateur vers l'URL : `http://<IP_VM>:<PORT>`
- vous pouvez aussi effectuer des requêtes HTTP depuis le terminal, plutôt qu'avec un navigateur
  - ça se fait avec la commande `curl`
  - et c'est ça que je veux dans le compte-rendu, pas de screen du navigateur :)

> Si le port c'est 80, alors c'est la convention pour HTTP. Ainsi, il est inutile de le préciser dans l'URL, le navigateur le fait de lui-même. On peut juste saisir `http://<IP_VM>`.

🌞 **Vérifier les logs d'accès**

- trouvez le fichier qui contient les logs d'accès, dans le dossier `/var/log`
- les logs d'accès, c'est votre serveur web qui enregistre chaque requête qu'il a reçu
- c'est juste un fichier texte
- affichez les 3 dernières lignes des logs d'accès dans le contenu rendu, avec une commande `tail`

## 5. Modif de la conf du serveur web

🌞 **Changer le port d'écoute**

- une simple ligne à modifier, vous me la montrerez dans le compte rendu
  - faites écouter NGINX sur le port 8080
- redémarrer le service pour que le changement prenne effet
  - `sudo systemctl restart nginx`
  - vérifiez qu'il tourne toujours avec un ptit `systemctl status nginx`
- prouvez-moi que le changement a pris effet avec une commande `ss`
  - utilisez un `| grep` pour isoler les lignes intéressantes
- n'oubliez pas de fermer l'ancien port dans le firewall, et d'ouvrir le nouveau
- prouvez avec une commande `curl` sur votre machine que vous pouvez désormais visiter le port 8080

> Là c'est pas le port par convention, alors obligé de préciser le port quand on fait la requête avec le navigateur ou `curl` : `http://<IP_VM>:8080`.

---

🌞 **Changer l'utilisateur qui lance le service**

- pour ça, vous créerez vous-même un nouvel utilisateur sur le système : `web`
  - référez-vous au [mémo des commandes](../../cours/memos/commandes.md) pour la création d'utilisateur
  - l'utilisateur devra avoir un mot de passe, et un homedir défini explicitement à `/home/web`
- modifiez la conf de NGINX pour qu'il soit lancé avec votre nouvel utilisateur
  - utilisez `grep` pour me montrer dans le fichier de conf la ligne que vous avez modifié
- n'oubliez pas de redémarrer le service pour que le changement prenne effet
- vous prouverez avec une commande `ps` que le service tourne bien sous ce nouveau utilisateur
  - utilisez un `| grep` pour isoler les lignes intéressantes

---

**Il est temps d'utiliser ce qu'on a fait à la partie 2 !**

🌞 **Changer l'emplacement de la racine Web**

- configurez NGINX pour qu'il utilise une autre racine web que celle par défaut
  - avec un `nano` ou `vim`, créez un fichiez `/var/www/site_web_1/index.html` avec un contenu texte bidon
  - dans la conf de NGINX, configurez la racine Web sur `/var/www/site_web_1/`
  - vous me montrerez la conf effectuée dans le compte-rendu, avec un `grep`
- n'oubliez pas de redémarrer le service pour que le changement prenne effet
- prouvez avec un `curl` depuis votre hôte que vous accédez bien au nouveau site

> **Normalement le dossier `/var/www/site_web_1/` est un dossier créé à la Partie 2 du TP**, et qui se trouve en réalité sur le serveur `storage.tp4.linux`, notre serveur NFS.

![MAIS](../img/nop.png)

## 6. Deux sites web sur un seul serveur

Dans la conf NGINX, vous avez du repérer un bloc `server { }` (si c'est pas le cas, allez le repérer, la ligne qui définit la racine web est contenu dans le bloc `server { }`).

Un bloc `server { }` permet d'indiquer à NGINX de servir un site web donné.

Si on veut héberger plusieurs sites web, il faut donc déclarer plusieurs blocs `server { }`.

**Pour éviter que ce soit le GROS BORDEL dans le fichier de conf**, et se retrouver avec un fichier de 150000 lignes, on met chaque bloc `server` dans un fichier de conf dédié.

Et le fichier de conf principal contient une ligne qui inclut tous les fichiers de confs additionnels.

🌞 **Repérez dans le fichier de conf**

- la ligne qui inclut des fichiers additionels contenus dans un dossier nommé `conf.d`
- vous la mettrez en évidence avec un `grep`

> On trouve souvent ce mécanisme dans la conf sous Linux : un dossier qui porte un nom finissant par `.d` qui contient des fichiers de conf additionnels pour pas foutre le bordel dans le fichier de conf principal. On appelle ce dossier un dossier de *drop-in*.

🌞 **Créez le fichier de configuration pour le premier site**

- le bloc `server` du fichier de conf principal, vous le sortez
- et vous le mettez dans un fichier dédié
- ce fichier dédié doit se trouver dans le dossier `conf.d`
- ce fichier dédié doit porter un nom adéquat : `site_web_1.conf`

🌞 **Créez le fichier de configuration pour le deuxième site**

- un nouveau fichier dans le dossier `conf.d`
- il doit porter un nom adéquat : `site_web_2.conf`
- copiez-collez le bloc `server { }` de l'autre fichier de conf
- changez la racine web vers `/var/www/site_web_2/`
- et changez le port d'écoute pour 8888

> N'oubliez pas d'ouvrir le port 8888 dans le firewall. Vous pouvez constater si vous le souhaitez avec un `ss` que NGINX écoute bien sur ce nouveau port.

🌞 **Prouvez que les deux sites sont disponibles**

- depuis votre PC, deux commandes `curl`
- pour choisir quel site visitez, vous choisissez un port spécifique